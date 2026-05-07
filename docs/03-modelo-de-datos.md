# PEDILO — Documento 03: Modelo de Datos

> **Estado:** Borrador para revisión
> **Versión:** 0.1
> **Fecha:** 2026-05-01
> **Próximo paso:** revisar, marcar correcciones, aprobar antes de pasar a Documento 04 (UI/UX y Wireframes)
>
> **Insumos:**
> - Doc 01 v1.3 (Visión y Alcance del MVP).
> - Doc 02 v0.4 (Arquitectura y Stack), particularmente decisiones D1–D18 y safeguards de Modo Social (§10.5).

---

## 1. Resumen ejecutivo

El modelo de datos de PEDILO es un **schema PostgreSQL** corriendo en Supabase, con **Row Level Security (RLS)** activado en TODAS las tablas con `tenant_id` para garantizar aislamiento multi-tenant.

**Cantidad de tablas en MVP:** ~16 tablas core + ~5 tablas Premium / V1.x preparadas.

**Convenciones globales:**
- Todos los IDs son **UUID v4** (`gen_random_uuid()`), no enteros autoincrementales.
- Todas las tablas tienen `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`.
- Las tablas mutables tienen `updated_at TIMESTAMPTZ` actualizado vía trigger.
- **Soft delete vía `deleted_at TIMESTAMPTZ NULL`** en tablas donde aplica (productos, mesas, staff). El borrado físico solo se hace por jobs de limpieza con retención configurable.
- Foreign keys con `ON DELETE CASCADE` cuando la relación es de propiedad estricta (ej: `order_items` cuando se borra el `order`).
- `ON DELETE SET NULL` cuando la relación es histórica y queremos preservar el registro padre.
- Naming: **snake_case** para tablas y columnas, **plural** para nombres de tabla.

---

## 2. Diagrama ERD (alto nivel)

```
                         ┌──────────────┐
                         │   tenants    │ ◄────── auth.users (Supabase)
                         │              │             │
                         │ + payment_   │             │ via tenant_members
                         │   config     │             ▼
                         │ + features   │       ┌──────────────────┐
                         │   flags      │       │ tenant_members   │
                         └──────┬───────┘       │ (owner, admin)   │
                                │               └──────────────────┘
                ┌───────────────┼───────────────┬───────────────┐
                ▼               ▼               ▼               ▼
         ┌──────────┐    ┌──────────┐   ┌────────────┐   ┌─────────────┐
         │ branches │    │  staff_  │   │ categories │   │  products   │
         │  (suc.)  │    │  users   │   └─────┬──────┘   │             │
         └────┬─────┘    │(PIN)     │         │          │ + ratings   │
              │          └──────────┘         │          │ + tags      │
              ▼                               ▼          │ + chef_rec  │
         ┌──────────┐                    ┌────────────┐  │ + serving_  │
         │  tables  │ ◄──── QR ─────────►│  products  │◄─┤   type      │
         │  (mesas) │                    └─────┬──────┘  └──────┬──────┘
         └────┬─────┘                          │                │
              │                                │                │
              ▼                                ▼                ▼
         ┌────────────────┐              ┌──────────────────────┐
         │ table_sessions │              │  product_ratings     │
         └────┬───────────┘              │ (device_id-based)    │
              │                          └──────────────────────┘
              ▼
         ┌──────────────┐               ┌──────────────────────┐
         │  comensales  │               │  product_stats       │
         │ (nickname +  │               │ (vista materializada)│
         │  device_id)  │               │ - top_orders_7d      │
         └────┬─────────┘               │ - avg_prep_time      │
              │                         │ - avg_rating         │
              ▼                         └──────────────────────┘
         ┌──────────┐
         │  orders  │
         └────┬─────┘
              │
              ▼
         ┌──────────────┐
         │ order_items  │ ◄──── trigger ────► realtime.broadcast
         │ + state      │
         │ + station    │
         └──────────────┘

         ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
         │ waiter_calls │    │   payments   │    │ forbidden_   │
         │              │    │  (vacío MVP) │    │   words      │
         └──────────────┘    └──────────────┘    └──────────────┘

  ─── Premium V1.x / V2 (preparadas pero vacías en MVP) ───
         ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
         │ social_optins   │  │ social_messages │  │ social_blocks   │
         └─────────────────┘  └─────────────────┘  └─────────────────┘
         ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
         │ social_reports  │  │ social_bans     │  │ comensal_history│
         └─────────────────┘  └─────────────────┘  └─────────────────┘
```

---

## 3. Convenciones y decisiones transversales

### 3.1. Multi-tenant via `tenant_id`

- **TODA tabla con datos del tenant lleva `tenant_id UUID NOT NULL`** referenciando a `tenants(id)`.
- Excepción: tablas globales (`auth.users` de Supabase, `forbidden_words` global, etc.) que documentamos explícitamente.
- `tenant_id` siempre es la **primera columna después del `id`** para legibilidad y consistencia de índices.

### 3.2. Timestamps

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()  -- actualizado vía trigger
deleted_at TIMESTAMPTZ NULL  -- soft delete cuando aplica
```

### 3.3. UUIDs

- `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`.
- `device_id` para comensales: TEXT (no UUID — viene de cliente, validado a UUID v4 pero almacenado flexible).

### 3.4. Soft delete

Aplica a tablas donde el borrado podría romper integridad histórica:
- `products`: si borrás un producto, los `order_items` históricos deben seguir referenciando el nombre/precio que tenían al momento.
- `tables` (mesas): para no perder histórico de pedidos.
- `staff_users`: para preservar audit log de logins.

**Patrón:** todas las queries de la app filtran `WHERE deleted_at IS NULL`. Triggers de soft delete settean el campo en lugar de DELETE.

### 3.5. Audit trail

Tablas críticas (orders, payments, staff_users) tienen `created_by` y `updated_by` apuntando a `auth.users(id)` o `staff_users(id)`. Esto se propaga automáticamente vía middleware Next.js que setea variables de sesión Postgres antes de cada query.

---

## 4. Tablas core (administración del tenant)

### 4.1. `tenants`

Cada restaurante en PEDILO es un tenant.

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Identidad
  slug TEXT NOT NULL UNIQUE,           -- "lapizzeria" → URL pedilo.com/r/lapizzeria
  name TEXT NOT NULL,                  -- "La Pizzería de Don Pepe"
  legal_name TEXT,                     -- razón social
  logo_url TEXT,                       -- Supabase Storage
  cuit TEXT,                           -- AFIP

  -- Plan y billing
  plan TEXT NOT NULL DEFAULT 'free',   -- 'free' | 'premium' | 'pro'
  plan_active_since TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  plan_renews_at TIMESTAMPTZ,
  trial_ends_at TIMESTAMPTZ,

  -- Pagos online (Doc 02 §12 — preparado para V2)
  payment_mode TEXT NOT NULL DEFAULT 'none',     -- 'none' | 'direct' | 'marketplace'
  mp_access_token TEXT,                          -- cifrado vía pgcrypto
  mp_user_id TEXT,
  commission_rate DECIMAL(5,4) NOT NULL DEFAULT 0,
  commission_active_since TIMESTAMPTZ,

  -- Feature flags por tenant (RA9)
  social_mode_enabled BOOLEAN NOT NULL DEFAULT false,
  kitchen_display_enabled BOOLEAN NOT NULL DEFAULT true,
  bar_display_enabled BOOLEAN NOT NULL DEFAULT true,

  -- Configuración general
  timezone TEXT NOT NULL DEFAULT 'America/Argentina/Buenos_Aires',
  currency TEXT NOT NULL DEFAULT 'ARS',
  language TEXT NOT NULL DEFAULT 'es-AR',

  -- Auditoria
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ,

  CONSTRAINT slug_format CHECK (slug ~ '^[a-z0-9][a-z0-9-]{2,49}$')
);

CREATE INDEX idx_tenants_slug ON tenants(slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenants_plan ON tenants(plan) WHERE deleted_at IS NULL;
```

**Notas:**
- `slug` es UNIQUE global (es la URL pública).
- `plan` se duplicará después en una tabla `tenant_subscriptions` cuando entre billing real (V2).
- `mp_access_token` se cifra con `pgcrypto` usando una clave de entorno; la app desencripta on-demand.

### 4.2. `tenant_members`

Relaciona usuarios autenticados (Supabase Auth) con tenants y su rol.

```sql
CREATE TABLE tenant_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL,                  -- 'owner' | 'admin'
  invited_by UUID REFERENCES auth.users(id),
  joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE (tenant_id, user_id)
);

CREATE INDEX idx_tenant_members_user ON tenant_members(user_id);
```

**Notas:**
- En MVP, casi siempre será 1 owner por tenant. Estructura preparada para más.
- El `role` aquí es del **owner/admin**, NO del staff (mozo/cajero/etc.) — eso vive en `staff_users`.

### 4.3. `branches` (sucursales)

```sql
CREATE TABLE branches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name TEXT NOT NULL,                  -- "Sucursal Palermo"
  address TEXT,
  phone TEXT,
  is_default BOOLEAN NOT NULL DEFAULT false,
  active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ,

  CONSTRAINT one_default_per_tenant UNIQUE (tenant_id, is_default) DEFERRABLE INITIALLY DEFERRED
);

CREATE INDEX idx_branches_tenant ON branches(tenant_id) WHERE deleted_at IS NULL;
```

**Notas:**
- En MVP cada tenant tiene **exactamente 1 sucursal** (creada automáticamente al onboarding). El modelo soporta N — solo se desbloquea como feature Premium.
- `is_default` con UNIQUE deferrable permite cambiar de "default" sin error transaccional.

### 4.4. `staff_users`

Login con username + PIN para mozo/cajero/cocina/bar.

```sql
CREATE TABLE staff_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  username TEXT NOT NULL,              -- "juan01"
  pin_hash TEXT NOT NULL,              -- bcrypt
  display_name TEXT NOT NULL,          -- "Juan", para mostrar en UI
  avatar_color TEXT,                   -- #hex, asignado al crear
  role TEXT NOT NULL,                  -- 'mozo' | 'cajero' | 'cocinero' | 'bartender'
  active BOOLEAN NOT NULL DEFAULT true,

  -- Anti-brute-force
  failed_attempts INT NOT NULL DEFAULT 0,
  locked_until TIMESTAMPTZ,
  pin_rotated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ,

  UNIQUE (tenant_id, username),
  CONSTRAINT valid_role CHECK (role IN ('mozo', 'cajero', 'cocinero', 'bartender'))
);

CREATE INDEX idx_staff_users_tenant ON staff_users(tenant_id) WHERE deleted_at IS NULL;
```

**Notas:**
- PIN se hashea con bcrypt (cost factor 10).
- Rotación obligatoria cada 90 días: app verifica `pin_rotated_at < NOW() - 90 days` y obliga reset al login.
- `active = false` desactiva el login sin borrar (para audit).

### 4.5. `staff_login_log`

Audit log de todos los logins de staff.

```sql
CREATE TABLE staff_login_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  staff_user_id UUID REFERENCES staff_users(id) ON DELETE SET NULL,
  attempted_username TEXT NOT NULL,    -- guarda intento aunque el user no exista
  success BOOLEAN NOT NULL,
  ip TEXT,
  user_agent TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_staff_login_log_tenant_time ON staff_login_log(tenant_id, created_at DESC);
```

---

## 5. Tablas operativas (mesa, sesión, comensal)

### 5.1. `tables` (mesas físicas del local)

```sql
CREATE TABLE tables (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  number TEXT NOT NULL,                -- "5", "T1", "A-3" — string para flexibilidad
  capacity INT NOT NULL DEFAULT 4,
  qr_token TEXT NOT NULL UNIQUE,       -- token corto único, embebido en URL del QR
  active BOOLEAN NOT NULL DEFAULT true,
  notes TEXT,                          -- "mesa cerca de la ventana"

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ,

  UNIQUE (branch_id, number)
);

CREATE INDEX idx_tables_tenant ON tables(tenant_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tables_qr_token ON tables(qr_token) WHERE deleted_at IS NULL;
```

**Notas sobre `qr_token`:**
- A pesar de que en Doc 01 §S9 confirmamos QR estático, igual usamos un token aleatorio único en lugar de un número predecible. URL: `pedilo.com/r/<slug>/m/<qr_token>`.
- Razón: si una mesa se elimina, su `qr_token` queda invalidado (otra no puede tener el mismo). Un QR robado no funciona después de eliminada.
- En MVP NO se rota el token. En V1.x con QR dinámico (Premium) se permitirá rotarlo desde el admin.

### 5.2. `table_sessions` (sesión activa de una mesa)

```sql
CREATE TABLE table_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  table_id UUID NOT NULL REFERENCES tables(id) ON DELETE CASCADE,

  status TEXT NOT NULL DEFAULT 'active',   -- 'active' | 'pending_validation' | 'closed' | 'closed_by_timeout'
  validated_by_staff_id UUID REFERENCES staff_users(id),
  validated_at TIMESTAMPTZ,

  total_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  paid_amount DECIMAL(12,2) NOT NULL DEFAULT 0,

  opened_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_activity_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  closed_at TIMESTAMPTZ,
  closed_by UUID REFERENCES staff_users(id),
  close_reason TEXT,                       -- 'paid' | 'timeout' | 'manual' | 'error'

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT one_active_session_per_table EXCLUDE USING gist (
    table_id WITH =
  ) WHERE (status IN ('active', 'pending_validation'))
);

CREATE INDEX idx_table_sessions_tenant_status ON table_sessions(tenant_id, status);
CREATE INDEX idx_table_sessions_table ON table_sessions(table_id);
CREATE INDEX idx_table_sessions_inactivity ON table_sessions(last_activity_at)
  WHERE status IN ('active', 'pending_validation');
```

**Notas:**
- El `EXCLUDE` constraint garantiza que **una mesa no pueda tener 2 sesiones activas a la vez**.
- El índice `idx_table_sessions_inactivity` es crítico para el cron job de timeout.
- `total_amount` y `paid_amount` se mantienen sincronizados vía trigger (más abajo).

### 5.3. `comensales`

Cada comensal anónimo dentro de una sesión de mesa.

```sql
CREATE TABLE comensales (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  table_session_id UUID NOT NULL REFERENCES table_sessions(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,             -- UUID del dispositivo persistido en cookie
  nickname TEXT,                       -- opcional
  avatar_color TEXT NOT NULL,          -- asignado por sistema al unirse
  joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  last_seen_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE (table_session_id, device_id)
);

CREATE INDEX idx_comensales_session ON comensales(table_session_id);
CREATE INDEX idx_comensales_device ON comensales(device_id);
```

**Notas:**
- `device_id` se genera client-side al primer escaneo y persiste en cookie/localStorage por 30 días.
- Un mismo `device_id` puede tener múltiples `comensales` históricos (uno por cada sesión de mesa donde participó).
- Para features de loyalty V1.x (D17), se queries por `device_id` cruzando todos sus `comensales` y sus `order_items`.

---

## 6. Tablas de menú

### 6.1. `categories`

```sql
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  name TEXT NOT NULL,                  -- "Entradas", "Pizzas"
  description TEXT,
  display_order INT NOT NULL DEFAULT 0,
  active BOOLEAN NOT NULL DEFAULT true,

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_categories_tenant_order ON categories(tenant_id, display_order)
  WHERE deleted_at IS NULL AND active = true;
```

### 6.2. `products`

```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,

  -- Identidad
  name TEXT NOT NULL,
  description TEXT,
  price DECIMAL(12,2) NOT NULL CHECK (price >= 0),
  photo_url TEXT,                      -- Supabase Storage

  -- Estación destino (ruteo automático)
  station TEXT NOT NULL DEFAULT 'kitchen',  -- 'kitchen' | 'bar' | 'other'

  -- Atributos del menú (D14)
  serving_type TEXT NOT NULL DEFAULT 'individual',  -- 'individual' | 'shared' | 'either'

  -- Tags dietéticos (Doc 01 §S2)
  tags TEXT[] DEFAULT '{}',            -- ['vegano', 'sin-tacc', 'picante']

  -- D13 — Recomendado del chef
  chef_recommended BOOLEAN NOT NULL DEFAULT false,

  -- Estado (Nivel A inventario)
  available BOOLEAN NOT NULL DEFAULT true,

  -- Orden visual
  display_order INT NOT NULL DEFAULT 0,

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ,

  CONSTRAINT valid_station CHECK (station IN ('kitchen', 'bar', 'other')),
  CONSTRAINT valid_serving CHECK (serving_type IN ('individual', 'shared', 'either'))
);

CREATE INDEX idx_products_category ON products(category_id, display_order)
  WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tenant_chef ON products(tenant_id) WHERE chef_recommended = true AND deleted_at IS NULL;
CREATE INDEX idx_products_tags ON products USING GIN(tags);
```

### 6.3. `product_ratings` (D11)

```sql
CREATE TABLE product_ratings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,             -- comensal anónimo
  table_session_id UUID REFERENCES table_sessions(id) ON DELETE SET NULL,
  comensal_id UUID REFERENCES comensales(id) ON DELETE SET NULL,

  rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  comment TEXT,                        -- opcional, max 200 chars (constraint en app)
  flagged BOOLEAN NOT NULL DEFAULT false,

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE (product_id, device_id)
);

CREATE INDEX idx_product_ratings_product ON product_ratings(product_id);
CREATE INDEX idx_product_ratings_device ON product_ratings(device_id);
```

### 6.4. `product_stats` (vista materializada — D12)

```sql
CREATE MATERIALIZED VIEW product_stats AS
SELECT
  p.id AS product_id,
  p.tenant_id,
  COUNT(DISTINCT oi.id) FILTER (
    WHERE oi.created_at > NOW() - INTERVAL '7 days'
  ) AS total_orders_7d,
  AVG(EXTRACT(EPOCH FROM (oi.delivered_at - oi.received_at)) / 60) FILTER (
    WHERE oi.delivered_at IS NOT NULL
  ) AS avg_prep_time_min,
  AVG(pr.rating) AS avg_rating,
  COUNT(pr.id) AS ratings_count
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.id AND oi.state != 'cancelado'
LEFT JOIN product_ratings pr ON pr.product_id = p.id
WHERE p.deleted_at IS NULL
GROUP BY p.id, p.tenant_id;

CREATE UNIQUE INDEX idx_product_stats_pk ON product_stats(product_id);
CREATE INDEX idx_product_stats_tenant_orders ON product_stats(tenant_id, total_orders_7d DESC);
CREATE INDEX idx_product_stats_tenant_rating ON product_stats(tenant_id, avg_rating DESC);
```

**Refresh:** vía `pg_cron` cada 1 hora con `REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats`. Trade-off de 1h de delay aceptable para "top 5", "mejor calificado" y "rápidos".

---

## 7. Tablas de pedidos

### 7.1. `orders`

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  table_session_id UUID NOT NULL REFERENCES table_sessions(id) ON DELETE CASCADE,
  comensal_id UUID NOT NULL REFERENCES comensales(id) ON DELETE RESTRICT,

  total_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  notes TEXT,                          -- nota a nivel pedido (raro, generalmente notes va por item)

  -- Validación del mozo (Doc 01 §5.4 — solo el primer pedido de la sesión)
  is_first_of_session BOOLEAN NOT NULL DEFAULT false,
  validated_by_staff_id UUID REFERENCES staff_users(id),
  validated_at TIMESTAMPTZ,

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_session ON orders(table_session_id);
CREATE INDEX idx_orders_tenant_time ON orders(tenant_id, created_at DESC);
CREATE INDEX idx_orders_pending_validation ON orders(tenant_id)
  WHERE is_first_of_session = true AND validated_at IS NULL;
```

### 7.2. `order_items` (el corazón del producto)

```sql
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  table_session_id UUID NOT NULL REFERENCES table_sessions(id) ON DELETE CASCADE,
  comensal_id UUID NOT NULL REFERENCES comensales(id) ON DELETE RESTRICT,

  -- Snapshot del producto al momento del pedido (no FK estricto, evita issues si el producto se borra)
  product_id UUID REFERENCES products(id) ON DELETE SET NULL,
  product_name_snapshot TEXT NOT NULL, -- copia del nombre al momento
  product_price_snapshot DECIMAL(12,2) NOT NULL,
  station TEXT NOT NULL,               -- 'kitchen' | 'bar' | 'other'

  quantity INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
  notes TEXT,                          -- "sin cebolla", "bien cocido"

  -- Estado del item (Doc 01 §5.5)
  state TEXT NOT NULL DEFAULT 'pendiente_validacion_mozo',
  -- 'pendiente_validacion_mozo' | 'recibido' | 'en_preparacion' | 'listo' | 'entregado' | 'cancelado'

  cancellation_reason TEXT,
  cancelled_by UUID REFERENCES staff_users(id),
  cancelled_at TIMESTAMPTZ,

  -- Timestamps de transición de estado (para métricas)
  received_at TIMESTAMPTZ,
  preparation_started_at TIMESTAMPTZ,
  ready_at TIMESTAMPTZ,
  delivered_at TIMESTAMPTZ,

  -- Routing override (Doc 02 §10.5 — modo "Mozo dicta")
  -- Si station = 'kitchen' pero kitchen_display_enabled = false en el tenant,
  -- el item igualmente entra a 'kitchen' como destino lógico, pero la UI lo
  -- enruta al panel del mozo en lugar del display de cocina.

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT valid_state CHECK (state IN (
    'pendiente_validacion_mozo', 'recibido', 'en_preparacion',
    'listo', 'entregado', 'cancelado'
  ))
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_session ON order_items(table_session_id);
CREATE INDEX idx_order_items_comensal ON order_items(comensal_id);
CREATE INDEX idx_order_items_station_state ON order_items(tenant_id, station, state)
  WHERE state IN ('recibido', 'en_preparacion', 'listo');
CREATE INDEX idx_order_items_active ON order_items(tenant_id, created_at DESC)
  WHERE state NOT IN ('entregado', 'cancelado');
```

**Notas críticas:**
- **Snapshot del producto:** guardamos `product_name_snapshot` y `product_price_snapshot` para que si el dueño edita o borra el producto, los pedidos históricos sigan teniendo sentido.
- **Cada item lleva `comensal_id`:** habilita división de cuenta en V2 (Patrón B Marketplace) sin migración (Doc 02 §12).
- **Estado de transición timestamped:** permite calcular tiempo en cada estado para métricas.

### 7.3. `waiter_calls` (D15)

```sql
CREATE TABLE waiter_calls (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  table_session_id UUID NOT NULL REFERENCES table_sessions(id) ON DELETE CASCADE,
  comensal_id UUID REFERENCES comensales(id) ON DELETE SET NULL,

  reason TEXT NOT NULL,                -- 'agua' | 'ayuda' | 'cuenta' | 'queja' | 'otro'
  message TEXT,                        -- opcional

  attended_at TIMESTAMPTZ,
  attended_by UUID REFERENCES staff_users(id),

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT valid_reason CHECK (reason IN ('agua', 'ayuda', 'cuenta', 'queja', 'otro'))
);

CREATE INDEX idx_waiter_calls_pending ON waiter_calls(tenant_id, created_at DESC)
  WHERE attended_at IS NULL;
```

---

## 8. Tablas de pagos (V2 — schema preparado, vacío en MVP)

### 8.1. `payments`

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  branch_id UUID NOT NULL REFERENCES branches(id) ON DELETE CASCADE,
  table_session_id UUID NOT NULL REFERENCES table_sessions(id),
  comensal_id UUID REFERENCES comensales(id),

  payment_mode TEXT NOT NULL,          -- 'direct' | 'marketplace'

  gross_amount DECIMAL(12,2) NOT NULL CHECK (gross_amount >= 0),
  commission_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
  net_to_tenant DECIMAL(12,2) NOT NULL,

  mp_payment_id TEXT,
  mp_status TEXT,                      -- 'approved' | 'rejected' | 'pending' | etc.
  mp_payer_email TEXT,

  status TEXT NOT NULL DEFAULT 'pending', -- 'pending' | 'approved' | 'rejected' | 'refunded'

  paid_at TIMESTAMPTZ,
  metadata JSONB,                      -- payload completo del webhook

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT valid_payment_mode CHECK (payment_mode IN ('direct', 'marketplace'))
);

CREATE INDEX idx_payments_session ON payments(table_session_id);
CREATE INDEX idx_payments_mp ON payments(mp_payment_id) WHERE mp_payment_id IS NOT NULL;
CREATE INDEX idx_payments_tenant_time ON payments(tenant_id, created_at DESC);
```

---

## 9. Tablas de moderación

### 9.1. `forbidden_words` (anti-abuso, Doc 02 §10.3)

```sql
CREATE TABLE forbidden_words (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,  -- NULL = global
  word TEXT NOT NULL,
  severity TEXT NOT NULL DEFAULT 'block', -- 'flag' | 'block'
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE (COALESCE(tenant_id::text, 'global'), word)
);

CREATE INDEX idx_forbidden_words_tenant ON forbidden_words(tenant_id);
```

**Notas:**
- Lista global mantenida por PEDILO + lista por tenant (admin del local agrega palabras propias).
- Aplicada en server-side al validar nicknames, notas de pedido, comments en ratings.

---

## 10. Tablas Premium V1.x / V2 (preparadas, vacías en MVP)

### 10.1. `comensal_history` (D17 — V1.x)

```sql
CREATE TABLE comensal_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,
  product_id UUID REFERENCES products(id) ON DELETE SET NULL,
  product_name_snapshot TEXT NOT NULL,
  ordered_at TIMESTAMPTZ NOT NULL,
  rating_given INT,
  visit_count_at_time INT NOT NULL,    -- "era la visita N en este local"

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_comensal_history_device_tenant ON comensal_history(device_id, tenant_id, ordered_at DESC);
```

**Notas:**
- En MVP, vacía. Se puebla retroactivamente si se activa el feature.
- En V1.x permite queries tipo "la última vez probaste X", "es tu 3ra visita".

### 10.2. Tablas `social_*` (D18 — V1.x/V2)

Schema completo definido en Doc 02 §10.5. Aquí se referencia para no duplicar:

- `social_optins` — opt-in del comensal por sesión.
- `social_messages` — saludos públicos cross-mesa.
- `social_blocks` — bloqueos device_id ↔ device_id.
- `social_reports` — reportes de mensajes.
- `social_bans` — bans temporales/permanentes por tenant.

---

## 11. Row Level Security (RLS) — la decisión más crítica

### 11.1. Estrategia general

- **RLS habilitado en TODA tabla con `tenant_id`.**
- Política base: el usuario solo accede a filas donde `tenant_id = (auth.jwt() ->> 'tenant_id')::uuid`.
- Excepciones documentadas: tablas globales (`forbidden_words` con `tenant_id = NULL`).

### 11.2. Custom claim en JWT

Supabase Auth Hook agrega `tenant_id` al JWT del usuario al iniciar sesión:

```sql
CREATE OR REPLACE FUNCTION public.custom_access_token_hook(event jsonb)
RETURNS jsonb LANGUAGE plpgsql STABLE AS $$
DECLARE
  claims jsonb;
  user_tenant_id uuid;
BEGIN
  SELECT tenant_id INTO user_tenant_id
  FROM tenant_members
  WHERE user_id = (event->>'user_id')::uuid
  LIMIT 1;

  claims := event->'claims';
  IF user_tenant_id IS NOT NULL THEN
    claims := jsonb_set(claims, '{tenant_id}', to_jsonb(user_tenant_id::text));
  END IF;

  event := jsonb_set(event, '{claims}', claims);
  RETURN event;
END;
$$;
```

### 11.3. Política de aislamiento estándar

Aplicada a TODAS las tablas con `tenant_id`:

```sql
-- Ejemplo en `orders`. Repetir para cada tabla.
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "tenant_isolation_select" ON orders
  FOR SELECT USING (
    tenant_id = (auth.jwt() ->> 'tenant_id')::uuid
  );

CREATE POLICY "tenant_isolation_insert" ON orders
  FOR INSERT WITH CHECK (
    tenant_id = (auth.jwt() ->> 'tenant_id')::uuid
  );

CREATE POLICY "tenant_isolation_update" ON orders
  FOR UPDATE USING (
    tenant_id = (auth.jwt() ->> 'tenant_id')::uuid
  ) WITH CHECK (
    tenant_id = (auth.jwt() ->> 'tenant_id')::uuid
  );

CREATE POLICY "tenant_isolation_delete" ON orders
  FOR DELETE USING (
    tenant_id = (auth.jwt() ->> 'tenant_id')::uuid
  );
```

### 11.4. Acceso del comensal anónimo (sin JWT de Supabase Auth)

El comensal NO está autenticado en Supabase Auth — usa `device_id` y session cookie firmada. Para que pueda leer el menú y crear pedidos, hay endpoints específicos del backend (Server Actions) que se ejecutan con el **service role key** (bypassa RLS) **pero validan manualmente** que:
1. El `tenant_id` del request coincide con el slug de la URL.
2. El `device_id` está autorizado para esa `table_session_id`.
3. El `comensal_id` pertenece al `device_id`.

**Esto significa:** la RLS protege contra usuarios autenticados maliciosos. Para clientes anónimos, la protección está en la capa de aplicación (middleware Next.js).

### 11.5. Acceso del staff (PIN)

El staff tampoco tiene JWT de Supabase Auth (usa cookie custom firmada). Misma estrategia: server actions validan la cookie y el `tenant_id`.

### 11.6. Tests de aislamiento OBLIGATORIOS

Antes del piloto, escribir tests automáticos que verifiquen:

- Usuario `tenant_A` no puede SELECT/INSERT/UPDATE/DELETE en filas de `tenant_B` en CADA tabla.
- Comensal con `device_id_X` no puede leer pedidos de comensales con `device_id_Y` en otra mesa.
- Staff de `tenant_A` no puede loguear con credenciales de staff de `tenant_B`.
- Tests de regresión en CI que corren en cada PR.

---

## 12. Triggers y funciones

### 12.1. Auto-update de `updated_at`

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Aplicar a cada tabla con updated_at
CREATE TRIGGER trg_tenants_updated_at BEFORE UPDATE ON tenants
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
-- (repetir para cada tabla mutable)
```

### 12.2. Notify Realtime al cambiar estado de item

```sql
CREATE OR REPLACE FUNCTION notify_order_item_change() RETURNS TRIGGER AS $$
BEGIN
  PERFORM realtime.broadcast(
    'tenant:' || NEW.tenant_id || ':orders:' || NEW.station,
    'item_state_changed',
    jsonb_build_object(
      'item_id', NEW.id,
      'state', NEW.state,
      'table_id', (SELECT table_id FROM table_sessions WHERE id = NEW.table_session_id)
    )
  );
  -- También al canal de la mesa específica
  PERFORM realtime.broadcast(
    'tenant:' || NEW.tenant_id || ':tables:' ||
      (SELECT table_id::text FROM table_sessions WHERE id = NEW.table_session_id),
    'item_state_changed',
    jsonb_build_object('item_id', NEW.id, 'state', NEW.state)
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_items_notify
  AFTER INSERT OR UPDATE OF state ON order_items
  FOR EACH ROW EXECUTE FUNCTION notify_order_item_change();
```

### 12.3. Mantener `total_amount` y `paid_amount` en `table_sessions`

```sql
CREATE OR REPLACE FUNCTION update_session_totals() RETURNS TRIGGER AS $$
BEGIN
  UPDATE table_sessions
  SET total_amount = (
    SELECT COALESCE(SUM(quantity * product_price_snapshot), 0)
    FROM order_items
    WHERE table_session_id = COALESCE(NEW.table_session_id, OLD.table_session_id)
      AND state != 'cancelado'
  ),
  last_activity_at = NOW()
  WHERE id = COALESCE(NEW.table_session_id, OLD.table_session_id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_items_update_session_total
  AFTER INSERT OR UPDATE OR DELETE ON order_items
  FOR EACH ROW EXECUTE FUNCTION update_session_totals();
```

### 12.4. Auto-cierre de sesiones por timeout (cron via `pg_cron` o Vercel Cron)

Job que corre cada 15 min:

```sql
-- Notificación a las 2 hs
UPDATE table_sessions SET last_notified_at = NOW()
WHERE status = 'active'
  AND last_activity_at < NOW() - INTERVAL '2 hours'
  AND last_notified_at IS NULL OR last_notified_at < last_activity_at;
-- (la notificación a Slack/email se dispara fuera de la DB)

-- Cierre forzado a las 6 hs
UPDATE table_sessions SET
  status = 'closed_by_timeout',
  closed_at = NOW(),
  close_reason = 'timeout'
WHERE status IN ('active', 'pending_validation')
  AND last_activity_at < NOW() - INTERVAL '6 hours';
```

---

## 13. Indexes y performance

Indexes ya definidos por tabla. Resumen de los más críticos:

| Índice | Por qué |
|---|---|
| `idx_table_sessions_inactivity` | Cron job de timeout, query frecuente |
| `idx_order_items_station_state` | Query del display de cocina/bar — items pendientes |
| `idx_order_items_active` | Panel del mozo — pedidos en curso |
| `idx_orders_pending_validation` | Notificación al mozo de mesas pendientes de validar |
| `idx_products_tags` (GIN) | Filtros dietéticos del menú |
| `idx_product_stats_tenant_orders` | Top 5 más pedidos |
| `idx_product_stats_tenant_rating` | Mejor calificados |
| `idx_tables_qr_token` | Lookup al escanear QR |
| `idx_tenants_slug` | Resolver `pedilo.com/r/<slug>` |

---

## 14. Migrations strategy

### 14.1. Herramientas

- **Supabase CLI** para gestión de migrations (`supabase db diff`, `supabase db push`).
- Archivos en `db/migrations/YYYYMMDDHHMMSS_descripcion.sql` versionados en Git.
- **Drizzle ORM** (Doc 02 D2) provee tipos TypeScript generados desde el schema, pero las migrations las gestiona Supabase.

### 14.2. Política de migraciones

- **Aditivas siempre que se pueda:** agregar columnas nullable o con DEFAULT. Nunca DROP COLUMN en el mismo deploy que la quita del código.
- **Backwards-compatible mínimo 1 deploy:** la app sigue funcionando con o sin la nueva columna durante el rollout.
- **Migraciones destructivas en deploy posterior**, cuando se confirma que no hay tráfico legacy.
- **Tests obligatorios** que corren las migrations en una DB ephemeral antes del deploy.

### 14.3. Migraciones futuras previstas

- **V1.x:** activar features de loyalty (poblar `comensal_history` retroactivamente).
- **V1.x:** activar Modo Social (llenar tablas `social_*`).
- **V2:** activar Patrón A de pagos (popular `payments`).
- **V2.x:** migrar a Patrón B Marketplace (no requiere cambio de schema, solo `payment_mode` a `marketplace`).

---

## 15. Seed data para desarrollo

Script `db/seed.sql`:

- 3 tenants demo (`la-pizzeria-demo`, `bar-demo`, `cafe-demo`).
- 1 owner por tenant (auth.users + tenant_members).
- 1 sucursal por tenant.
- 4 staff por tenant: 1 mozo, 1 cajero, 1 cocinero, 1 bartender. PIN inicial `1234` (rotación obligatoria al primer login).
- 5 mesas por tenant, números 1-5.
- 3 categorías por tenant (Entradas, Principales, Bebidas).
- ~20 productos por tenant con fotos placeholder.
- 5 productos con `chef_recommended = true`.
- 0 sesiones de mesa, 0 pedidos (se crean al usar el sistema).

**Reset rápido:**

```bash
supabase db reset
npm run seed
```

---

## 16. Decisiones pendientes del Doc 03

| # | Tema | Default propuesto |
|---|---|---|
| DD1 | **Cifrado de `mp_access_token`** | Postgres `pgcrypto` con clave maestra en env var `DB_ENCRYPTION_KEY`. Alternativa: Supabase Vault (más nuevo, más limpio, evaluar disponibilidad). |
| DD2 | **Estrategia de archivado de sesiones cerradas** | Conservar en mesa principal por 90 días, después mover a tabla `table_sessions_archive` vía job mensual. Reduce tamaño de queries activas. |
| DD3 | **Foreign key de `comensal_id` en `order_items`** | `ON DELETE RESTRICT` (no permitir borrar comensal si tiene items). Alternativa: `SET NULL` para permitir limpieza, perdiendo trazabilidad. **Recomiendo RESTRICT.** |
| DD4 | **Histórico de cambios de precio de productos** | NO en MVP. Se confía en `product_price_snapshot` por item. Tabla `product_price_history` solo si lo pide el piloto. |
| DD5 | **Idempotencia de pedidos** | Cada `order` con `idempotency_key` (UUID generado client-side) para evitar duplicados por reintentos de red. Sumar columna en `orders`. |
| DD6 | **Compresión / borrado de logs viejos** | `staff_login_log` y similares: rotación a 90 días vía cron. |

---

## 17. Riesgos del modelo de datos

| # | Riesgo | Mitigación |
|---|---|---|
| RD1 | **Costos de RLS en queries de gran cardinalidad** (Postgres aplica las políticas en cada row) | Indexes con `tenant_id` siempre primero. Revisar plans de queries lentos en piloto. |
| RD2 | **Vista materializada `product_stats` desactualizada** | Refresh cada 1 hora vía cron. Si se detecta lag, reducir intervalo o pasar a triggers incrementales. |
| RD3 | **Soft delete creciente impacta queries** | Indexes parciales con `WHERE deleted_at IS NULL`. Job de purga física mensual para tenants inactivos. |
| RD4 | **`table_sessions` con `EXCLUDE constraint`** podría fallar bajo concurrencia | Probarlo con tests de stress. Alternativa: lock pesimista en mesa al abrir sesión. |
| RD5 | **Crecimiento de `order_items`** (~1M rows/año a 100 locales activos) | Particionamiento por `created_at` mensual cuando supere 10M rows. Por ahora innecesario. |
| RD6 | **`mp_access_token` en plain text** si pgcrypto falla | Tests de seguridad antes de V2. Auditoría manual de cifrado. |

---

## 18. Pendientes antes de pasar al Documento 04 (UI/UX)

- [ ] **Vos:** revisar este documento, marcar correcciones / cambios.
- [ ] **Vos:** confirmar las **6 decisiones pendientes DD1–DD6**.
- [ ] **Vos:** confirmar los **6 riesgos RD1–RD6**.
- [ ] **Yo:** una vez aprobado, pasar a Documento 04 con wireframes Figma reflejando el modelo de datos (ej: pantalla "Mesa Viva" con avatares = entidades `comensales`, items = `order_items`).

---

## 19. Glosario específico del modelo de datos

- **RLS (Row Level Security):** mecanismo de Postgres que filtra filas según políticas, transparente para el query.
- **Soft delete:** marcar como borrado vía `deleted_at` en lugar de DELETE físico.
- **Materialized view:** vista cuyos resultados están pre-computados y persistidos. Más rápida pero requiere refresh.
- **JWT custom claim:** dato adicional embebido en el token de auth (ej: `tenant_id`).
- **Snapshot column:** columna que copia un valor de otra tabla en el momento de creación, para preservar histórico ante cambios futuros (ej: `product_name_snapshot`).
- **`pgcrypto`:** extensión de Postgres para cifrado simétrico.
- **`pg_cron`:** extensión que permite ejecutar SQL periódicamente desde Postgres.
- **Idempotency key:** clave que identifica una operación de forma única para evitar duplicados por reintentos.
- **Partial index:** índice que solo cubre las filas que cumplen una condición (ej: `WHERE deleted_at IS NULL`).
- **GIN index:** índice especializado para búsquedas en arrays/JSON.
