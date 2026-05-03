# PEDILO — Documento 02: Arquitectura y Stack

> **Estado:** Borrador para revisión
> **Versión:** 0.2
> **Fecha:** 2026-05-01
> **Próximo paso:** revisar, marcar correcciones, aprobar antes de pasar a Documento 03 (modelo de datos)
>
> **Historial de cambios:**
> - **v0.1 (2026-05-01):** borrador inicial (16 secciones).
> - **v0.2 (2026-05-01):** sección 12 expandida — pagos online y comisión PEDILO con Patrones A (V2 inicial) y B (Marketplace, V2.x objetivo). Schema y hooks listos desde MVP para evitar migración futura.
> - **v0.3 (2026-05-01):** sumada sección 17 — Escalabilidad, costos y triggers de migración. Niveles de carga, comparación de opciones cuando se rompe Realtime (Team Plan / Pusher / Self-hosted), reconciliación honesta revenue vs costos de infra, umbrales de alertas.

---

## 1. Resumen ejecutivo

PEDILO se construye como una **aplicación web full-stack en Next.js sobre Supabase**, desplegada en **Vercel**. La arquitectura prioriza:

- **Velocidad de desarrollo** (un solo dev, side project, 4–5 meses al MVP).
- **Costo bajo durante validación** (USD 0–45/mes hasta tener facturación).
- **Multi-tenancy seguro desde día 1** vía Postgres RLS.
- **Tiempo real** sin escribir WebSockets a mano.
- **Operabilidad** (un solo deploy, observabilidad clara, rollback simple).

**Stack final:**

| Capa | Tecnología |
|---|---|
| Frontend + Backend | **Next.js 15** (App Router) + **React 19** + **TypeScript** |
| Base de datos | **PostgreSQL** en **Supabase** |
| Auth (owner/admin) | **Supabase Auth** (email + password) |
| Auth (staff) | Capa custom **username + PIN** sobre Supabase |
| Tiempo real | **Supabase Realtime** (canales por tenant) |
| Storage (fotos) | **Supabase Storage** |
| Hosting | **Vercel** |
| Rate limiting | **Upstash Redis** |
| Error tracking | **Sentry** |
| Logs centralizados | **Better Stack** (cuando crezca) |
| Email transaccional | **Resend** |
| Pagos (V2) | **MercadoPago** |

---

## 2. Diagrama de arquitectura general

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENTES (browsers)                         │
├─────────────────────────────────────────────────────────────────────┤
│  Comensal      Mozo (tablet)   Cocinero (tablet)   Owner/Admin (PC) │
│  móvil         "/staff"        "/staff"            "/admin"         │
│  "/r/<slug>"                                                        │
└──────┬──────────────┬──────────────────┬──────────────────┬─────────┘
       │              │                  │                  │
       │     HTTPS / WebSocket           │                  │
       │              │                  │                  │
┌──────▼──────────────▼──────────────────▼──────────────────▼─────────┐
│                          VERCEL EDGE                                │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              Next.js 15 (App Router)                         │   │
│  │  ┌─────────────┬─────────────┬──────────────┬─────────────┐  │   │
│  │  │ (public)    │ (menu)      │ (staff)      │ (admin)     │  │   │
│  │  │ Landing     │ Cliente del │ Mozo, cocina,│ Owner del   │  │   │
│  │  │ Onboarding  │ comensal    │ bar, caja    │ local       │  │   │
│  │  └─────────────┴─────────────┴──────────────┴─────────────┘  │   │
│  │  Server Actions  +  Route Handlers  +  Middleware            │   │
│  └──────────────────────────────────────────────────────────────┘   │
└──────┬─────────────────────┬─────────────────────┬─────────┬────────┘
       │                     │                     │         │
       ▼                     ▼                     ▼         ▼
┌──────────────┐   ┌──────────────────┐   ┌─────────────┐  ┌─────────┐
│   SUPABASE   │   │  UPSTASH REDIS   │   │   SENTRY    │  │ RESEND  │
│              │   │  (rate limiting) │   │  (errores)  │  │ (email) │
│  ┌────────┐  │   └──────────────────┘   └─────────────┘  └─────────┘
│  │Postgres│  │
│  │  + RLS │  │   ┌──────────────────┐   ┌─────────────┐
│  └────────┘  │   │  BETTER STACK    │   │ MERCADOPAGO │
│  ┌────────┐  │   │  (logs/uptime)   │   │   (V2)      │
│  │  Auth  │  │   └──────────────────┘   └─────────────┘
│  └────────┘  │
│  ┌────────┐  │
│  │Realtime│◄─┼────── WebSocket subscriptions desde clientes
│  └────────┘  │       (canales por tenant)
│  ┌────────┐  │
│  │Storage │  │
│  └────────┘  │
└──────────────┘
```

**Flujos principales:**

- **Comensal escanea QR** → Vercel sirve página `/r/<slug>/m/<n>` (server-rendered con menú) → cliente subscribe a canal Realtime para estado de su pedido.
- **Comensal confirma pedido** → Server Action en Next.js → INSERT en Postgres (con RLS) → trigger publica evento Realtime → mozo recibe notificación → mozo valida (anti-abuso) → estado del pedido pasa a `recibido` → cocina/bar reciben item via Realtime.
- **Mozo cambia estado** → Server Action → UPDATE en Postgres → trigger Realtime → cliente del comensal y display de estación se actualizan en <2s.

---

## 3. Stack: alternativas evaluadas

Tres alternativas fueron consideradas. La A es la elegida.

### Alternativa A — Next.js + Supabase + Vercel ✅ **(elegida)**

**Pros:**
- Un solo proyecto, un solo deploy. Mínimo plumbing.
- Supabase resuelve auth, realtime, storage y RLS sin código custom.
- Vercel deploya Next.js en 1 click, con preview por PR.
- Costo USD 0–45/mes durante validación.
- Familiaridad parcial (React, TS, Postgres ya manejados).

**Contras:**
- Curva de aprendizaje de Next.js App Router (~15–25 hs).
- Vendor lock-in moderado con Supabase (Auth, Realtime, Storage).
- Vercel tiene límites en tier gratis (100 GB-h serverless/mes) que pueden quedar cortos al crecer.

**Mitigación del lock-in:** Postgres es estándar (migración trivial a Neon, Railway, RDS). Auth y Realtime sí son específicos de Supabase, pero su API es razonablemente reemplazable si hace falta.

### Alternativa B — Next.js + Postgres self-managed + Auth.js + Pusher

Stack: Next.js + Vercel + Neon (Postgres) + Auth.js (NextAuth) + Pusher para realtime + Cloudflare R2 para storage.

**Pros:**
- Sin lock-in con Supabase.
- Pusher tiene tier gratis generoso.
- Neon Postgres es excelente y barato.

**Contras:**
- Más servicios para configurar y mantener (5 vs 2).
- Auth.js es robusto pero requiere más código que Supabase Auth.
- Pusher cuesta más en alta concurrencia.
- **Suma estimada de 30–50 hs de plumbing extra.**

**Veredicto:** mejor portabilidad pero peor velocidad. Para un side project con 4–5 meses de plazo, la velocidad gana.

### Alternativa C — Backend separado (Fastify) + React SPA + Postgres

Stack clásico: backend Node/Fastify desplegado en Railway, frontend React SPA en Vercel, Postgres en Supabase o Railway.

**Pros:**
- Arquitectura conocida y "tradicional".
- Backend reusable si hace falta una app móvil nativa en V2.

**Contras:**
- Dos proyectos para mantener (frontend + backend).
- Sin SSR del menú público → peor SEO de las páginas de menú de cada local.
- Requiere infra extra (Railway USD 5–20/mes adicionales).
- **Suma estimada de 40–60 hs de plumbing extra.**

**Veredicto:** se descarta. El costo en velocidad no se justifica para este MVP.

---

## 4. Multi-tenancy

Esta es **la decisión más importante de seguridad del proyecto**. Un error acá filtra datos de un restaurante a otro y mata el producto.

### 4.1. Modelo elegido: shared DB + shared schema + RLS por `tenant_id`

Todas las tablas tienen una columna `tenant_id UUID NOT NULL` y políticas de **Row Level Security (RLS)** en Postgres que filtran automáticamente cualquier query al tenant del usuario autenticado.

```sql
-- Ejemplo de política
CREATE POLICY "tenant_isolation" ON orders
  FOR ALL
  USING (tenant_id = (auth.jwt() ->> 'tenant_id')::uuid);
```

**Cómo funciona en runtime:**

1. Usuario se autentica → Supabase Auth emite JWT que incluye `tenant_id` como custom claim.
2. Cualquier query del usuario hacia Postgres pasa por el JWT.
3. Postgres aplica la política RLS y solo devuelve filas con ese `tenant_id`.
4. **Imposible** que un dev se olvide del filtro: aunque escriba `SELECT * FROM orders` sin `WHERE`, RLS lo filtra.

### 4.2. Alternativas descartadas

| Modelo | Por qué se descartó |
|---|---|
| Schema per tenant | Complejidad de migraciones (×N schemas), Supabase no lo soporta bien. |
| DB per tenant | Insostenible en costo/operación con 100+ tenants. |
| RLS + sharding por región | Innecesario a esta escala. |

### 4.3. Tests obligatorios de aislamiento

Antes de pasar al piloto se escriben tests que verifiquen:

- Usuario de tenant A **no puede leer** filas de tenant B en ninguna tabla.
- Usuario de tenant A **no puede escribir/actualizar/borrar** filas de tenant B.
- Comensal anónimo de un local **solo puede leer/escribir** datos de su propia mesa.
- Test de regresión que se corre en cada PR.

### 4.4. Routing por tenant

- URL pública del local: `pedilo.com/r/<slug>` (ej: `pedilo.com/r/lapizzeria`).
- URL del menú escaneado: `pedilo.com/r/<slug>/m/<numero-mesa>`.
- Panel admin: `pedilo.com/admin` (autenticado, deduce el tenant del usuario).
- Panel staff: `pedilo.com/staff` (autenticado por PIN, dentro del contexto del tenant del local).

> **Decisión:** **path-based** (no subdominio por tenant) en MVP. Más simple. Si en el futuro queremos custom domains (`pedidos.lapizzeria.com.ar`), Vercel lo soporta sin cambios mayores.

---

## 5. Autenticación y autorización

Modelo híbrido confirmado en Doc 01 §5.2 / S11.

### 5.1. Owner / Admin del local — Supabase Auth

- Email + contraseña.
- Recuperación de password por email.
- Magic links opcionales.
- Tabla `users` (gestionada por Supabase) + tabla `tenant_members` que vincula `user_id` con `tenant_id` y rol (`owner`, `admin`).
- JWT incluye custom claim `tenant_id` (vía Supabase Auth Hook).

**Flujo de onboarding:**
1. Usuario va a `/signup` → completa email, password, datos del local.
2. Server Action crea: registro en `auth.users` + tenant nuevo + `tenant_members` linkeando ambos con rol `owner`.
3. Email de verificación (Supabase lo dispara solo).
4. Redirige a `/admin/onboarding` para completar setup inicial (mesas, primer producto).

### 5.2. Staff (mozo, cajero, cocinero, bartender) — username + PIN custom

Supabase Auth no soporta este flujo nativamente. Se construye una capa simple encima.

**Tabla `staff_users`:**

```sql
CREATE TABLE staff_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  username TEXT NOT NULL,
  pin_hash TEXT NOT NULL,           -- bcrypt
  display_name TEXT NOT NULL,        -- "Juan", "Ana"
  role TEXT NOT NULL,                -- 'mozo' | 'cajero' | 'cocinero' | 'bartender'
  active BOOLEAN NOT NULL DEFAULT true,
  failed_attempts INT NOT NULL DEFAULT 0,
  locked_until TIMESTAMPTZ,
  pin_rotated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (tenant_id, username)
);
```

**Flujo de login:**
1. Tablet del local va a `/r/<slug>/staff` → muestra lista de avatares (consulta a `staff_users` filtrada por tenant del slug).
2. Usuario elige su avatar → input numérico para PIN.
3. Server Action verifica PIN con bcrypt. Si OK, crea sesión custom (cookie HTTP-only firmada con secret).
4. Si falla, incrementa `failed_attempts`. A los 5 fallos, setea `locked_until = NOW() + 15 min` y notifica al admin.
5. Sesión válida por 12 hs (configurable). Renovación silenciosa si hay actividad.

**Cookie de sesión:** firmada con `JWT_SECRET`, contiene `staff_user_id`, `tenant_id`, `role`, `exp`. Middleware de Next.js la valida en cada request a `/staff/*`.

**Rotación obligatoria:** cada 90 días el sistema marca el PIN como vencido y obliga a cambiarlo en el próximo login.

### 5.3. Comensal — sesión anónima

- No hay login.
- Al escanear QR, se setea cookie `device_id` (UUID generado) si no existe.
- Server Action que confirma pedido valida que la mesa esté "activa" y que el `device_id` tenga sesión abierta en esa mesa.
- Cookie `device_id` persiste 30 días.

### 5.4. Autorización (RBAC simple)

Cada endpoint sensible se protege en middleware Next.js + en RLS Postgres.

**Roles y permisos básicos del MVP:**

| Acción | Comensal | Mozo | Cajero | Cocinero | Bartender | Admin/Owner |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| Ver menú del local | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Crear pedido en mesa | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Validar primer pedido de mesa | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Cambiar estado de item (cocina) | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| Cambiar estado de item (bar) | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Marcar item entregado | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| Cerrar cuenta de mesa | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ |
| Cancelar item | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Toggle disponible/agotado | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Crear/editar productos | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Crear/editar mesas | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Invitar staff / setear PINs | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Ver reportes (Premium) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 6. Tiempo real

### 6.1. Supabase Realtime — cómo funciona

Supabase Realtime expone tres mecanismos. Usamos los tres:

1. **Postgres Changes** — escuchar cambios (INSERT/UPDATE/DELETE) en tablas. Útil para que un display se actualice cuando cambia un pedido.
2. **Broadcast** — mensajes pub/sub arbitrarios entre clientes en un canal. Útil para notificaciones puntuales (ej: "mozo, te llaman de la mesa 3").
3. **Presence** — saber quién está conectado a un canal. Útil para "X comensales en la mesa".

### 6.2. Estrategia de canales (clave para seguridad multi-tenant)

**Patrón de naming:**

```
tenant:<tenant_id>:orders:kitchen     ← display de cocina
tenant:<tenant_id>:orders:bar         ← display de bar
tenant:<tenant_id>:orders:waiters     ← panel de mozos
tenant:<tenant_id>:tables:<table_id>  ← canal de una mesa específica
                                       (cliente del comensal, mozo de esa mesa)
```

**Suscripciones por rol:**

- Cocinero → `tenant:<tid>:orders:kitchen`.
- Bartender → `tenant:<tid>:orders:bar`.
- Mozo → `tenant:<tid>:orders:waiters` + `tenant:<tid>:tables:*` (todas las mesas que atiende).
- Comensal → `tenant:<tid>:tables:<table_id>` (solo la suya).
- Admin → todos los canales del tenant.

**Validación del lado servidor:** Supabase permite definir RLS también sobre los eventos Realtime, garantizando que un usuario malicioso no pueda subscribirse a canales de otro tenant.

### 6.3. Disparo de eventos

Postgres triggers que ejecutan `realtime.broadcast()` cuando cambia el estado de un item:

```sql
CREATE OR REPLACE FUNCTION notify_order_change() RETURNS TRIGGER AS $$
BEGIN
  PERFORM realtime.broadcast(
    'tenant:' || NEW.tenant_id || ':orders:' || NEW.station,
    'item_state_changed',
    jsonb_build_object('item_id', NEW.id, 'state', NEW.state, 'table_id', NEW.table_id)
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### 6.4. Performance objetivo

- **Latencia de propagación:** < 2 segundos del `UPDATE` al render en cliente.
- **Concurrencia objetivo MVP:** 200 conexiones simultáneas (suficiente para 20 locales × 10 dispositivos).
- **Concurrencia escalada:** 5.000+ con Supabase Pro.

### 6.5. Fallback ante caída de Realtime

Si la suscripción WebSocket cae, el cliente:
1. Muestra banner "reconectando…".
2. Intenta reconexión con backoff exponencial.
3. Como red de seguridad, hace **polling cada 30 segundos** mientras está sin Realtime.
4. Al reconectar, hace un fetch completo de estado para reconciliar.

---

## 7. Estructura del proyecto Next.js

App Router con **route groups** para separar por audiencia.

```
pedilo/
├── app/
│   ├── (public)/                  # No autenticado, público
│   │   ├── page.tsx               # Landing
│   │   ├── signup/                # Registro de owner
│   │   ├── login/                 # Login owner
│   │   └── pricing/
│   │
│   ├── (menu)/                    # Comensal anónimo
│   │   └── r/
│   │       └── [slug]/
│   │           ├── page.tsx       # Vista del menú del local (sin mesa)
│   │           └── m/
│   │               └── [tableNum]/
│   │                   ├── page.tsx       # Menú con sesión de mesa
│   │                   ├── cart/
│   │                   └── status/
│   │
│   ├── (staff)/                   # Mozo, cajero, cocina, bar
│   │   └── r/
│   │       └── [slug]/
│   │           └── staff/
│   │               ├── login/
│   │               ├── waiter/    # Panel mozo
│   │               ├── kitchen/   # Display cocina
│   │               ├── bar/       # Display bar
│   │               └── cashier/   # Caja
│   │
│   ├── (admin)/                   # Owner / admin del local
│   │   └── admin/
│   │       ├── layout.tsx         # Sidebar nav
│   │       ├── dashboard/
│   │       ├── tables/
│   │       ├── menu/
│   │       ├── products/
│   │       ├── staff/
│   │       ├── orders/
│   │       └── settings/
│   │
│   ├── api/                       # Route handlers (webhooks, etc.)
│   │   ├── webhooks/
│   │   │   └── mercadopago/       # V2
│   │   └── cron/
│   │       └── close-stale-tables/  # Job de cierre por timeout
│   │
│   ├── layout.tsx
│   └── globals.css
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts              # Cliente browser
│   │   ├── server.ts              # Cliente server-side
│   │   └── middleware.ts          # Cliente para middleware
│   ├── auth/
│   │   ├── owner.ts               # Helpers para auth owner
│   │   └── staff.ts               # Helpers para auth staff (PIN)
│   ├── db/
│   │   ├── queries.ts             # Queries reutilizables
│   │   └── mutations.ts
│   ├── realtime/
│   │   └── channels.ts            # Helpers de naming/subscribe
│   ├── ratelimit.ts               # Upstash wrappers
│   ├── logger.ts                  # Pino logger estructurado
│   └── utils/
│
├── components/
│   ├── ui/                        # Componentes base (shadcn/ui)
│   ├── menu/                      # Componentes específicos del menú
│   ├── orders/
│   ├── admin/
│   └── staff/
│
├── db/
│   ├── migrations/                # SQL migrations versionadas
│   └── seed.sql                   # Data de seed para dev
│
├── tests/
│   ├── e2e/                       # Playwright
│   ├── integration/               # Tests de RLS, multi-tenant
│   └── unit/                      # Vitest
│
├── public/
├── middleware.ts                  # Auth + tenant context
├── next.config.mjs
├── tailwind.config.ts
└── package.json
```

**Decisiones puntuales:**

- **shadcn/ui** como librería de componentes base (Tailwind + Radix). Es código que se copia al repo, no dependency. Customizable y sin lock-in.
- **Server Actions** para todas las mutaciones (formularios, acciones del usuario). Más simple que API Routes para CRUD.
- **Route Handlers** (`app/api/*`) solo para webhooks externos y cron jobs.
- **`middleware.ts`** valida sesión y setea contexto del tenant antes de cada request.

---

## 8. Entornos, CI/CD y despliegue

### 8.1. Tres entornos

| Entorno | Propósito | Costo |
|---|---|---|
| **dev (local)** | Desarrollo en tu máquina. Supabase corriendo en Docker local. | $0 |
| **staging** | Preview de PRs. Supabase project "staging" + Vercel preview. | $0 (free tier) |
| **prod** | Producción. Supabase Pro + Vercel Pro. | USD 25–45/mes |

### 8.2. Branching y deploys

- **`main`** = producción. Cada merge a main deploya automáticamente.
- **PRs contra main** generan preview URL en Vercel automáticamente.
- **Migrations de Postgres** se aplican vía `supabase db push` en CI antes del deploy.

### 8.3. CI/CD (GitHub Actions)

Pipeline en cada PR:

```yaml
- Lint (ESLint)
- Type check (tsc --noEmit)
- Unit tests (Vitest)
- Integration tests (RLS, auth) contra Supabase ephemeral
- E2E tests (Playwright) contra preview deploy
- Build (next build)
```

Pipeline en merge a main:

```yaml
- Re-run de PR pipeline
- Apply migrations (supabase db push)
- Deploy a Vercel (automático via Git integration)
- Smoke tests post-deploy
- Notificación a Slack / Discord
```

### 8.4. Rollback

- **Frontend/backend Next.js:** Vercel mantiene los últimos N deploys. Rollback en 1 click desde dashboard, o via CLI: `vercel rollback`. Tiempo: < 2 minutos.
- **Migraciones de DB:** cada migration tiene su `down`. `supabase db reset` revierte. **Pero ojo:** rollback de schema es peligroso si ya hay data. Política: las migraciones forward son **siempre aditivas** (agregar columnas nullable, nuevas tablas), nunca destructivas en el mismo deploy. Eliminaciones se hacen en deploys posteriores cuando se confirma que no hay tráfico legacy.

### 8.5. Secrets y variables de entorno

- **Vercel** maneja los secrets por entorno (dev, preview, prod).
- **Local:** archivo `.env.local` en gitignore. Template `.env.example` en repo.
- **Nunca** commitear secrets. Pre-commit hook con `detect-secrets`.

Lista de secrets necesarios:

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY     # solo server-side, nunca expone
DATABASE_URL                  # Postgres directo (migrations)
JWT_SECRET                    # firma de cookies de staff
UPSTASH_REDIS_REST_URL
UPSTASH_REDIS_REST_TOKEN
SENTRY_DSN
SENTRY_AUTH_TOKEN
RESEND_API_KEY
MERCADOPAGO_ACCESS_TOKEN      # V2
MERCADOPAGO_WEBHOOK_SECRET    # V2
```

---

## 9. Observabilidad

### 9.1. Logs estructurados

Todos los logs van como **JSON** vía **Pino**. Ejemplo:

```json
{
  "level": "info",
  "time": "2026-05-01T14:23:11.123Z",
  "tenant_id": "a7f3b2...",
  "user_id": "u1...",
  "request_id": "req_xY...",
  "route": "POST /admin/products",
  "msg": "Product created",
  "product_id": "p1..."
}
```

**Reglas:**
- Cada request tiene un `request_id` único (header `x-request-id` o generado).
- Cada log incluye `tenant_id` cuando aplica.
- Sin PII innecesaria (no loguear contraseñas, tokens, datos personales del comensal).

### 9.2. Error tracking — Sentry

- Sentry SDK en frontend y backend de Next.js (integración nativa).
- Cada error se envía con:
  - Stack trace completo.
  - Request ID, tenant ID, user ID.
  - Breadcrumbs (acciones previas del usuario).
  - Release tag (commit SHA).
- **Source maps** uploadeados en CI para que los stack traces sean legibles.
- Alertas configuradas:
  - Cualquier error nuevo (no visto antes) → email.
  - Error rate > 1% en últimas 5 min → Slack.
  - Error en endpoint crítico (confirmar pedido, login) → Slack inmediato.

### 9.3. Logs centralizados

- En MVP: logs de Vercel (CLI `vercel logs`) + Supabase logs alcanzan.
- Cuando supere ~50 locales activos: enviar logs a **Better Stack / Logtail** vía drain de Vercel. Tier free hasta ~3 GB/mes.

### 9.4. Uptime monitoring

- **Better Stack Uptime** o **UptimeRobot** (free).
- Endpoints monitorados cada minuto:
  - `/` (landing pública)
  - `/api/health` (health check propio que verifica DB + Redis)
- Alertas a email + push si baja > 1 min.

### 9.5. Métricas de negocio

- Dashboard interno simple en `/admin/system` (solo super-admin PEDILO).
- Queries directas a Postgres para:
  - Tenants activos hoy
  - Pedidos por hora
  - Errores activos
  - Uptime semanal
  - Conversión free → paid
- No requiere Datadog/Grafana en MVP. Postgres + página HTML alcanza.

### 9.6. Health check propio

```ts
// app/api/health/route.ts
GET /api/health → {
  status: "ok" | "degraded" | "down",
  checks: {
    database: { ok: true, latency_ms: 12 },
    realtime: { ok: true },
    redis: { ok: true, latency_ms: 5 },
  },
  version: "abc123",
  uptime_s: 4321
}
```

---

## 10. Anti-abuso y rate limiting

### 10.1. Stack: Upstash Redis

- Redis serverless con tier gratis (10k requests/día) suficiente para MVP.
- Librería `@upstash/ratelimit` con sliding window.

### 10.2. Endpoints con rate limit

| Endpoint | Límite | Clave |
|---|---|---|
| Confirmar pedido | 10/min | `device_id` + IP |
| Llamar al mozo | 1/2 min | `device_id` |
| Login owner | 5/15 min | email + IP |
| Login staff (intentos PIN) | 5/15 min | `staff_user_id` |
| Onboarding/signup | 3/hora | IP |
| Cualquier mutation autenticada | 60/min | `user_id` |

### 10.3. Otras mitigaciones

- **Validación del mozo en primer pedido** (Doc 01 §5.4) — bloquea pedidos fantasma desde casa.
- **Filtro de palabras en nicknames y notas** — lista negra simple en tabla `forbidden_words` editable por admin del local.
- **Captcha en signup** (`hCaptcha` o `Cloudflare Turnstile`, free).
- **Honeypot fields** en formularios públicos.
- **CSRF protection** — Next.js Server Actions lo manejan por default.

### 10.4. Detección de comportamiento sospechoso (post-MVP)

- Velocidad anormal de pedidos en una mesa.
- Mismo `device_id` activo en múltiples mesas simultáneas.
- Pedidos cancelados por cocina recurrentes desde un mismo dispositivo.

Por ahora se loguean como warnings; las acciones automáticas se evalúan en V1.x.

---

## 11. Estrategia de testing

### 11.1. Pirámide

```
        /\
       /  \  E2E (Playwright)
      /----\ ~10 tests críticos del happy path
     /------\
    /  Integ \ Tests de RLS, auth, multi-tenant
   /----------\ ~30-50 tests
  /            \
 /     Unit     \ Vitest
/________________\ ~100-200 tests de helpers, validators
```

### 11.2. Tests obligatorios antes del piloto

**Multi-tenancy (críticos):**
- ✅ Usuario tenant A no puede leer datos de tenant B en cada tabla con `tenant_id`.
- ✅ Usuario tenant A no puede mutar datos de tenant B.
- ✅ Comensal de mesa A no puede ver pedidos de mesa B.
- ✅ Staff de tenant A no puede loguear en tenant B.

**Auth:**
- Lockout tras 5 intentos fallidos de PIN.
- Recovery de password owner.
- Expiración de sesión staff.

**Flujos críticos (E2E):**
- Onboarding completo: signup → crear mesas → crear menú → primer pedido.
- Comensal escanea QR → pide → mozo valida → pasa a cocina → entrega.
- Cierre de cuenta por mozo.
- Cancelación de item.

**Performance:**
- Carga inicial de menú < 2s en 4G simulado.
- Latencia Realtime < 2s en condiciones normales.

### 11.3. Datos de prueba

- Seed script genera: 3 tenants demo, 5 mesas cada uno, 20 productos, 4 staff por tenant.
- Reset rápido para CI: `supabase db reset && npm run seed`.

---

## 12. Pagos online y comisión PEDILO (preparación V2)

> **MVP V1:** sin pago online. El cliente paga al mozo. Sin comisión.
> **V2:** pago online vía MercadoPago, sin comisión PEDILO al inicio (Patrón A).
> **V2.x (objetivo estratégico):** **Patrón B — Marketplace con split automático**. Es la palanca de revenue principal del negocio (ver charla del modelo de negocio del 2026-05-01).

La arquitectura del MVP **debe dejar lista la base de datos y los hooks** para soportar ambos patrones sin migración dolorosa.

### 12.1. Patrón A — Pago directo al tenant (V2 inicial)

Cuando entre el pago online, la primera versión usa este patrón:

```
Comensal paga $1.000
        ↓
[Cuenta MercadoPago del restaurante]
        ↓
$1.000 al restaurante (íntegro)
        ↓
PEDILO sigue cobrando solo la suscripción Premium mensual
```

- Cada tenant conecta **su propia cuenta de MP** vía OAuth (`MERCADOPAGO_ACCESS_TOKEN` se almacena por tenant cifrado).
- PEDILO **no maneja el dinero**, solo facilita el flujo.
- Sin obligaciones fiscales adicionales para PEDILO sobre las transacciones del comensal.
- Permite cobrar comisión variable **vía facturación mensual aparte**, pero no es automático ni recomendado en producción a escala.

**Por qué Patrón A primero:**
- Implementación rápida (no requiere registro especial en MP).
- Permite validar el feature de pago online con clientes reales sin trabar el desarrollo en aprobaciones de marketplace.
- Onboarding del restaurante simple: "conectá tu MP".

### 12.2. Patrón B — Marketplace / Split automático ⭐ (objetivo V2.x)

Patrón final hacia el que apunta el negocio:

```
Comensal paga $1.000
        ↓
[Marketplace de MP donde PEDILO está registrado como plataforma]
        ↓
Split automático en cada cobro
        ↓
$990 al restaurante  +  $10 a PEDILO  (ej: 1% de comisión)
```

**Pre-requisitos para activar Patrón B:**
- PEDILO registrado como **Aplicación Marketplace** en MercadoPago (formulario + revisión de MP, ~1–2 semanas).
- CUIT/datos fiscales de PEDILO en regla (Monotributo o SRL según volumen).
- Términos y condiciones de PEDILO actualizados con cláusula de plataforma intermediaria.
- Cumplimiento de requisitos AFIP correspondientes a plataformas de pago.

**Implementación técnica:**
- Mismo `MERCADOPAGO_ACCESS_TOKEN` por tenant (sigue siendo su cuenta), pero las preferencias de pago se crean indicando `marketplace_fee` con la comisión de PEDILO.
- MP hace el split automáticamente al cobrarse el pago.
- El webhook recibe ambos eventos: pago acreditado al tenant + comisión acreditada a PEDILO.
- Migración A→B es **transparente para el comensal**: ve el mismo flujo de pago.

### 12.3. Decisiones para el MVP (schema y hooks ya listos)

Aunque V1 no tiene pago online ni comisión, **dejamos preparado**:

**Campos por tenant** (en tabla `tenants`):

```sql
payment_mode TEXT NOT NULL DEFAULT 'none'      -- 'none' | 'direct' | 'marketplace'
mp_access_token TEXT                            -- cifrado, NULL en MVP
mp_user_id TEXT                                 -- ID de usuario en MP
commission_rate DECIMAL(5,4) NOT NULL DEFAULT 0 -- % de comisión PEDILO (0 = sin comisión)
commission_active_since TIMESTAMPTZ
```

**Tabla `payments`** (vacía en MVP, schema listo):

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  table_session_id UUID REFERENCES table_sessions(id),
  comensal_id UUID,                          -- quién pagó (puede ser parcial en split)
  payment_mode TEXT NOT NULL,                -- 'direct' | 'marketplace'
  gross_amount DECIMAL(12,2) NOT NULL,       -- total cobrado al cliente
  commission_amount DECIMAL(12,2) NOT NULL,  -- comisión PEDILO (0 en Patrón A)
  net_to_tenant DECIMAL(12,2) NOT NULL,      -- lo que recibe el restaurante
  mp_payment_id TEXT,                        -- ID de MP
  mp_status TEXT,                            -- 'approved' | 'rejected' | etc.
  status TEXT NOT NULL DEFAULT 'pending',
  paid_at TIMESTAMPTZ,
  metadata JSONB,                            -- payload completo de MP
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Cada `item` lleva `comensal_id`** desde día 1 → habilita "pagar mi parte" en V2 sin migración.

**Webhook handler** `/api/webhooks/mercadopago` con código stub que parsea ambos formatos de notificación (directo y marketplace) y guarda la transacción correspondiente.

**Variables de entorno** previstas para V2 (no usadas en MVP):

```
MERCADOPAGO_APP_ID                  # ID de PEDILO como app marketplace (Patrón B)
MERCADOPAGO_APP_SECRET              # secret de la app
MERCADOPAGO_WEBHOOK_SECRET          # validación de webhooks
MERCADOPAGO_DEFAULT_COMMISSION_RATE # comisión default si no hay override por tenant
```

### 12.4. Implicancias fiscales (a profundizar antes de V2)

| Patrón | PEDILO factura por | PEDILO declara |
|---|---|---|
| **A** (pago directo) | Suscripción Premium mensual al restaurante. Eventualmente comisión variable mensual aparte. | Solo lo que factura. No es responsable del dinero del comensal. |
| **B** (marketplace) | Suscripción + comisión automática en cada transacción. | Plataforma de pago: tiene obligaciones especiales con AFIP, retenciones, etc. |

**Pendiente antes de activar Patrón B:** consulta con contador especializado en plataformas digitales. Es una conversación de 1 hora que evita problemas de meses.

### 12.5. Decisiones tomadas que NO se cambian al pasar de A a B

Estos puntos se diseñan ahora y sirven a ambos patrones:

- ✅ Cada item con `comensal_id` (división de cuenta).
- ✅ Tabla `payments` con campos `gross_amount`, `commission_amount`, `net_to_tenant`.
- ✅ `payment_mode` y `commission_rate` por tenant (en V1 son `'none'` y `0`).
- ✅ Webhook handler con dispatcher que soporta ambos formatos.
- ✅ MP access token cifrado en DB (Postgres `pgcrypto` con clave en env var).

---

## 13. Subdominios / routing público

### 13.1. MVP: path-based

```
pedilo.com/                   ← landing
pedilo.com/signup
pedilo.com/login
pedilo.com/admin              ← admin (autenticado)
pedilo.com/r/lapizzeria       ← menú público del local (sin mesa)
pedilo.com/r/lapizzeria/m/3   ← menú con sesión de mesa 3
pedilo.com/r/lapizzeria/staff ← login de staff
status.pedilo.com             ← status page (Better Stack)
```

### 13.2. Post-MVP: custom domains opcionales

Cuando un local Premium quiera su propio dominio:

```
pedidos.lapizzeria.com.ar  →  CNAME a Vercel  →  resuelve al tenant "lapizzeria"
```

Vercel + Next.js soportan este patrón vía middleware que detecta el host header. **No requiere cambios al MVP**, solo activarlo cuando lo pidan.

---

## 14. Decisiones pendientes

| # | Tema | Default propuesto |
|---|---|---|
| D1 | **Librería de validación de schemas** | **Zod**. Estándar en TS, integra bien con Server Actions. |
| D2 | **ORM o queries directas** | **Drizzle ORM** sobre Supabase. Type-safe, liviano, no oculta SQL. Alternativa: Supabase JS client + tipos generados. |
| D3 | **Estado del cliente** | **Zustand** para state global mínimo (carrito, usuario), **TanStack Query** para data fetching/cache. |
| D4 | **Forms** | **React Hook Form** + Zod. |
| D5 | **Estilos** | **Tailwind CSS** + **shadcn/ui**. |
| D6 | **Subida de imágenes (productos)** | Cliente sube directo a Supabase Storage con presigned URL. Server valida tamaño/tipo. Resize on-the-fly via Supabase image transformations. |
| D7 | **Generación de QR** | Librería `qrcode` server-side. Genera PDF imprimible vía `pdfkit` con logo del local. |
| D8 | **Internacionalización** | NO en MVP (solo español). Reservar `next-intl` para post-MVP. |
| D9 | **Analytics de producto** | **PostHog** (free tier) o **Plausible**. Tracking de eventos clave (signup, primer pedido, etc.). Decidir cuál cerca del piloto. |
| D10 | **Job scheduler** (cron de timeouts) | **Vercel Cron** (incluido en plan Pro). Alternativa: pg_cron en Supabase. Decisión: Vercel Cron para MVP. |

---

## 15. Riesgos arquitectónicos

| # | Riesgo | Mitigación |
|---|---|---|
| RA1 | RLS mal configurado deja filtrar datos entre tenants | Tests de aislamiento obligatorios + revisión manual de cada policy + auditoría antes del piloto. |
| RA2 | Vercel/Supabase free tier se queda corto en pico inesperado | Límites monitoreados, alertas a 70% de uso, plan de upgrade pre-aprobado. |
| RA3 | Curva de App Router más larga de lo esperado | Prototipo descartable de 1–2 fines de semana antes del código real (R4 doc 01). |
| RA4 | Latencia Realtime > 2s en hora pico | Carga prevista bajo holgada al MVP. Re-evaluar a 50+ locales. |
| RA5 | Costo de Supabase Realtime escala mal con concurrencia alta | Monitorear conexiones simultáneas, considerar self-hosted Realtime si supera USD 200/mes. |
| RA6 | Lock-in con Supabase complica migración futura | Capa de abstracción mínima en `lib/supabase/*`. Postgres es portable; Auth y Realtime sí requieren reemplazo si migra. |
| RA7 | Vercel Serverless cold starts en endpoints poco usados | Endpoints críticos del cliente del comensal son SSR/CSR híbridos, no serverless puro. Evaluar Edge Runtime para handlers de pedido. |

---

## 16. Pendientes antes de pasar al Documento 03 (modelo de datos)

- [ ] **Vos:** revisar este documento, marcar correcciones / cambios.
- [ ] **Vos:** confirmar las **10 decisiones pendientes** (sección 14) o ajustarlas.
- [ ] **Vos:** confirmar los **7 riesgos arquitectónicos** (sección 15) o sumar otros.
- [ ] **Vos:** revisar los **triggers de migración de la sección 17** y ajustar umbrales si querés.
- [ ] **Yo:** una vez aprobado, redactar Documento 03 con el modelo de datos completo (entidades, relaciones, schemas SQL, RLS policies).

---

## 17. Escalabilidad, costos y triggers de migración

> *Esta sección documenta hasta dónde llega el stack actual, qué se rompe primero, y cuándo migrar a alternativas. Pensada para revisar 1, 2 o 3 años después cuando el negocio escale.*

### 17.1. Modelo de carga por local típico

Referencia para multiplicar al estimar carga total. Local "mediano" según Doc 01:

| Métrica | Valor estimado |
|---|---|
| Mesas activas en pico | 30 |
| Dispositivos comensales conectados (Realtime) en pico | 30–90 |
| Dispositivos staff conectados (Realtime) | 5–8 |
| **Conexiones Realtime simultáneas en pico por local** | **~30–100** |
| Pedidos por día | 200–400 |
| DB queries por pedido (INSERTs + UPDATEs + SELECTs) | 8–15 |
| Bandwidth por local/mes (menús con fotos + traffic cliente) | 200–500 MB |

### 17.2. Niveles de escala y bottleneck dominante

#### 🟢 Nivel 1 — hasta ~50 locales activos en pico simultáneo

| Métrica | Carga | Capacidad stack actual |
|---|---|---|
| Realtime concurrent | 1.500–5.000 | ❌ Excede los 500 de Supabase Pro |
| Pedidos/día agregados | 10K–20K | ✅ Postgres trivial |
| DB compute | ~10% shared "Small" | ✅ |
| Bandwidth/mes | 10–25 GB | ✅ Holgado dentro de 250 GB Pro |

**Primer bottleneck real: Realtime concurrent connections.** Saturás los 500 de Pro a partir de **~10 locales activos simultáneos** (no 50).

**Costo total mensual:** USD 45–120.

#### 🟡 Nivel 2 — 50–200 locales activos en pico

| Métrica | Carga | Capacidad |
|---|---|---|
| Realtime concurrent | 5K–20K | ❌ Crítico — necesita Team plan o self-hosted |
| Pedidos/día | 50K–200K | 🟡 Compute "Small" se siente |
| DB connections | 150–400 | ✅ Supavisor pooling lo maneja |
| Bandwidth/mes | 50–200 GB | 🟡 Cerca del límite Pro de 250 GB |

**Costo total mensual:** USD 200–800 (depende de cómo escalás Realtime — ver §17.4).

#### 🟠 Nivel 3 — 200–1.000 locales activos en pico

| Métrica | Carga | Capacidad |
|---|---|---|
| Realtime concurrent | 20K–100K | ❌ Self-hosted o servicio dedicado obligatorio |
| Pedidos/día | 200K–1M | 🟡 Compute "Large" o "XL" + posibles read replicas |
| DB queries/segundo en pico | 500–3.000 QPS | 🟡 Índices muy bien pensados + cache de queries comunes |
| Bandwidth/mes | 200 GB–1 TB | 🟡 Vercel bandwidth costs materiales |

**Costo total mensual:** USD 800–3.000.

#### 🔴 Nivel 4 — 1.000+ locales activos en pico

A esta escala, **replanteo arquitectónico**: Postgres self-managed con replicas, Realtime distribuido, CDN dedicado, sharding regional, observabilidad enterprise.

**Costo:** USD 3.000–15.000+/mes. **Pero a esta escala ya no sos un side project** — tenés equipo, hay devops dedicado, y ARR justifica las inversiones.

### 17.3. Vector de bottleneck por orden de aparición

```
Locales activos en pico simultáneo:

 0 ──── 10 ──── 30 ──── 80 ──── 200 ──── 500 ──── 1000 ──── 3000+
 │              │       │       │        │        │
 │              │       │       │        │        └─ Replanteo arquitectónico
 │              │       │       │        │
 │              │       │       │        └─ Self-hosted Realtime obligatorio
 │              │       │       │           DB Large + read replicas
 │              │       │       │
 │              │       │       └─ Team plan Supabase o self-hosted Realtime
 │              │       │          Compute upgrade DB
 │              │       │
 │              │       └─ ALERTA: Realtime cerca del límite Pro (500)
 │              │          Empezá a evaluar plan
 │              │
 │              └─ Holgado, monitorear bandwidth
 │
 └─ Stack Pro alcanza sin problema
```

**Orden de aparición:**
1. 🥇 **Realtime concurrent connections** (rompe a ~10 locales activos en pico)
2. 🥈 **DB compute** (rompe a ~100 locales activos)
3. 🥉 **Bandwidth** (rompe a ~200 locales activos)
4. 4️⃣ **Storage** (rompe a ~500 locales activos)

### 17.4. Opciones cuando se rompe Realtime (la decisión #1 al escalar)

Cuando Realtime de Supabase Pro se queda corto, hay 4 caminos posibles:

| Opción | Costo mensual @ 5K concurrent | Esfuerzo setup | Mantenimiento | Recomendación |
|---|---|---|---|---|
| **Supabase Team** | USD 599 | 0 hs | Cero | Si tenés flujo de caja sobrado y querés cero fricción operativa |
| **Pusher.com (managed)** | USD 49 | 2–4 hs | Cero | ⭐ **Mejor relación costo/esfuerzo** |
| **Ably (managed)** | USD 29–99 | 2–4 hs | Cero | Alternativa viable a Pusher |
| **Soketi self-hosted (Hetzner)** | USD 10–20 (VPS) | 8–16 hs setup + ~1 h/mes | Bajo-medio | Más barato, requiere DevOps básico |
| **Centrifugo self-hosted** | USD 15–30 (VPS) | 16–24 hs | Medio | Más robusto que Soketi para alta escala |
| **Self-host Supabase Realtime** | USD 20–40 (VPS) | 24+ hs | Alto | NO recomendado para solo dev |

**Recomendación según etapa:**

- **0–80 locales activos:** Supabase Pro alcanza, no migres preventivamente.
- **80–200 locales activos:** migrar a **Pusher.com** (managed, USD 49/mes). Migración compatible casi sin código nuevo si la capa `lib/realtime/channels.ts` está bien aislada (ver §7).
- **200–1.000 locales activos:** seguir en Pusher (sube a tier mayor, ~USD 99–249/mes) o migrar a **Soketi self-hosted** si querés ahorrar.
- **1.000+ locales activos:** Centrifugo cluster o solución enterprise (Ably Enterprise, Supabase Enterprise).

### 17.5. Otras palancas de optimización antes de migrar

Antes de pagar más, revisá si podés bajar la carga:

- **Reducir conexiones Realtime ociosas:** desconectar canales cuando un comensal se va (en `beforeunload` o tras 30 min sin actividad).
- **Polling fallback en lugar de WS para clientes con baja prioridad:** ej: comensales podrían ver estado de su pedido cada 10s vía polling en vez de WS persistente. Reduce 80% de las conexiones a costa de latencia.
- **Compresión y resize de imágenes** del menú (reduce bandwidth ×3–5).
- **Cache de menús públicos** vía Vercel Edge — el menú de un local cambia poco, cachealo 5 minutos.
- **Índices DB** afinados: cualquier query lento en producción es candidato.

Estas optimizaciones pueden **duplicar la capacidad efectiva del stack actual** sin gastar un dólar más.

### 17.6. Reconciliación honesta: revenue vs costos de infra

> *Esta tabla reconcilia lo que se conversó en la charla del modelo de negocio (2026-05-01) con los números de infra reales.*

Asumimos:
- ARPU = USD 32 (mix Premium USD 25 + Pro USD 49 al 70/30%)
- Conversión free → paid = 10%
- Patrón B (commission) eventual: 1% sobre 30% de pedidos pagados online × USD 25 ticket promedio

| Locales totales | Activos en pico (~30%) | Paid (10%) | MRR sin commission | MRR con commission Patrón B | Costo infra estimado | **Margen aprox.** |
|---|---|---|---|---|---|---|
| 100 | ~30 | 10 | USD 320 | USD 320–800 | USD 25–45 | USD 275–755 |
| 300 | ~100 | 30 | USD 960 | USD 1.500–2.500 | USD 100–200 | USD 800–2.300 |
| 500 | ~150 | 50 | USD 1.600 | USD 3.000–5.000 | USD 200–400 | USD 1.400–4.600 |
| 1.000 | ~300 | 100 | USD 3.200 | USD 8.000–15.000 | USD 500–1.000 | USD 2.700–14.000 |
| 1.500 | ~500 | 150 | USD 4.800 | USD 15.000–27.000 | USD 1.000–2.000 | USD 3.800–25.000 |

**Lecturas importantes:**

- **MVP a 12 meses (50–200 locales totales):** infra **USD 25–45/mes**, holgado. La presión de costos no es el tema todavía.
- **Año 2 (200–500 locales totales):** infra empieza a sentirse (USD 100–400/mes), pero el margen lo cubre cómodo.
- **Año 3+ (500–1.000 locales totales):** infra USD 500–1.000/mes. Si el Patrón B (commission) está activo, el MRR lo absorbe sin problema. Si seguís solo en suscripción, el margen se aprieta.
- **El número MRR depende dramáticamente de si activás Patrón B.** Sin commission, PEDILO es un side income decente. Con commission, escala bien.

### 17.7. Triggers de migración (umbrales para alertas)

Configurar desde el día 1 en Better Stack / dashboard interno:

| Métrica | 🟡 Advertencia | 🔴 Acción inmediata | Acción al cruzar 🔴 |
|---|---|---|---|
| Realtime concurrent connections | 350 (70% de 500) | 450 (90%) | Migrar a Pusher.com (Doc en `docs/runbooks/realtime-migration.md`) |
| DB CPU promedio (5 min) | 60% | 80% | Upgrade compute add-on |
| DB connections activas | 150 | 250 | Revisar pooling, queries lentas |
| Bandwidth mensual Vercel | 700 GB (70% Pro) | 900 GB (90%) | Bandwidth add-on o CDN |
| Storage Supabase | 70 GB (70%) | 90 GB (90%) | Storage add-on o limpieza |
| Vercel function compute | 700 GB-h (70%) | 900 GB-h (90%) | Optimizar funciones lentas / Edge runtime |
| p95 latency endpoint pedido | 1.5s | 3s | Investigar query lento, índices |
| Sentry error rate | 0.5% | 1% | Investigar y arreglar |
| Realtime event publish rate | 1.5K/s | 2.5K/s | Revisar si hay loops o suscripciones de más |

### 17.8. Ejercicio: ¿qué deploys hoy alcanzan para X locales?

Para tener intuición rápida, te dejo este "tablero":

| Configuración | Capacidad estimada | Costo/mes | Cuándo usarla |
|---|---|---|---|
| Supabase Pro + Vercel Pro (default propuesto) | ~30–80 locales activos en pico | USD 45 | MVP + año 1 |
| + Pusher.com Starter | ~200 activos en pico | USD 95 | Cuando rompa Realtime de Pro |
| + Pusher Pro tier + Supabase compute "Medium" | ~500 activos en pico | USD 250–400 | Año 2–3 |
| + Soketi self-hosted + Supabase compute "Large" + Vercel Enterprise | ~1.000 activos en pico | USD 800–1.500 | Año 3+ |
| Replanteo arquitectónico completo | 1.000+ activos en pico | USD 3.000+ | Cuando tu MRR justifique equipo |

### 17.9. Reglas operativas para el escalado

1. **No migres preventivamente.** Si el stack actual aguanta, no toques nada. Cada migración tiene riesgo.
2. **Migrá apenas el umbral 🟡 (70%) se sostenga 7 días.** Picos puntuales no justifican.
3. **Documentá cada migración** en `docs/runbooks/`. Tu yo-futuro va a agradecer saber por qué se movieron las piezas.
4. **Probá la migración en staging primero.** Siempre. Sin excepción.
5. **Tené plan de rollback** para cada migración. Si Pusher no anda, ¿cómo volvés a Supabase Realtime en 10 minutos?
6. **Revisá esta sección cada 6 meses.** Los precios y capacidades de Supabase, Vercel, Pusher cambian. Lo que dice esta tabla puede estar desactualizado en 2 años.

---

## Glosario técnico

- **App Router:** modelo de routing de Next.js 13+ basado en file-system + Server Components.
- **Server Actions:** funciones server-side llamables desde el cliente, alternativa a API routes para mutaciones.
- **RLS (Row Level Security):** mecanismo de Postgres que filtra filas según políticas, transparente para el query.
- **JWT (JSON Web Token):** token firmado que prueba la identidad. Lo emite Supabase Auth y viaja en cada request.
- **Realtime channel:** canal pub/sub al que clientes se subscriben para recibir eventos en tiempo real.
- **Multi-tenancy:** patrón donde múltiples clientes comparten una infraestructura con datos aislados lógicamente.
- **Tenant:** un cliente del SaaS (en PEDILO, un restaurante).
- **Edge Runtime:** entorno de ejecución de Vercel más liviano y rápido que Node, con limitaciones de APIs disponibles.
- **Cold start:** primera ejecución de una función serverless, más lenta porque hay que arrancar el contenedor.
- **CNAME:** registro DNS que apunta un dominio a otro.
- **Webhook:** endpoint que recibe notificaciones de un servicio externo (ej: MercadoPago avisa que un pago se acreditó).
- **Concurrent connections (Realtime):** cantidad de WebSockets abiertos simultáneamente con el servicio Realtime. Métrica clave para dimensionar.
- **Pusher / Ably / Soketi / Centrifugo:** servicios o productos que proveen pub/sub en tiempo real vía WebSockets. Alternativas a Supabase Realtime.
- **Self-hosted:** correr un servicio en tu propia infra en lugar de usar la versión SaaS managed.
- **Add-on (Supabase):** capacidad extra (compute, storage, bandwidth) que se contrata sobre el plan base.
- **Read replica:** copia de la DB principal usada solo para lecturas, descarga al primary.
- **CDN (Content Delivery Network):** red de servidores distribuidos geográficamente que cachean contenido estático cerca del usuario.
