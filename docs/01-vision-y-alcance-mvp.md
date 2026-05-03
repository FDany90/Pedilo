# PEDILO — Documento 01: Visión y Alcance del MVP

> **Estado:** ✅ Aprobado
> **Versión:** 1.3
> **Fecha de última actualización:** 2026-05-01
> **Próximo paso:** redactar Documento 03 (modelo de datos)
>
> **Historial de cambios:**
> - **v0.1 (2026-05-01):** borrador inicial.
> - **v1.0 (2026-05-01):** revisión Dani — incorporados estados de pedido refinados, validación del mozo (anti-abuso), escalamiento de timeouts, modelo auth híbrido (email/PIN), nueva sección 11 de requisitos no funcionales, riesgos R8 y R9.
> - **v1.1 (2026-05-01):** sumada sección 11.6 — Operación y soporte productivo (12 requisitos operativos a alto nivel; detalle en futuro Doc 06).
> - **v1.2 (2026-05-01):** post-investigación de mercado — sumado posicionamiento canónico ("interfaz comensal-mozo-caja"), 8 features extras al MVP (Mesa Viva, calificaciones, top 5, filtros, recomendado del chef, chip compartir/individual, llamar al mozo con motivo), Modo Social como Premium con safeguards, riesgo R10 (competencia con módulos QR de Fudo/Maxirest). Insumo: `notas-investigacion-mercado.md` v1.0.
> - **v1.3 (2026-05-01):** resueltos comentarios de revisión de Dani — reformulado lenguaje cocina/bar (no es POS/KDS, es "pantalla de pedidos"), agregada sección de hardware mínimo del local con Modo "Mozo dicta" para locales sin inversión, sumado supuesto S12 (memoria del comensal por device_id en MVP, loyalty visible en V1.x, cuenta opcional en V2).

---

## 1. Visión

PEDILO es una **plataforma web SaaS multi-tenant** que permite a restaurantes y bares gestionar pedidos en mesa mediante códigos QR, sin que los clientes finales tengan que descargar nada ni registrarse.

**Posicionamiento canónico (oración del producto):**

> **PEDILO es la interfaz que conecta al comensal con el staff del local — mozo, caja, cocina y bar — sin reemplazar el POS contable que el dueño ya use (o sin pedirle uno si no tiene).**

**Modelo de adopción:** PedidosYa-style — signup self-service, sin demo, sin comercial, sin contrato. El restaurante se crea cuenta, sube el menú, descarga sus QRs, y está operando en menos de 10 minutos. **Tier free permanente** como wedge de adopción.

**Propuesta de valor:**

- Para el **dueño del local:** un panel único para gestionar mesas, menú, productos, staff y pedidos en tiempo real. Coexiste con su POS actual (Fudo, Maxirest, otro o ninguno) — no reemplaza inventario, facturación ni delivery.
- Para el **comensal:** abrir el menú escaneando un QR, pedir desde el celular, ver en tiempo real qué pidieron sus compañeros de mesa, llamar al mozo y, en el futuro, pagar online — sin instalar apps ni hacer cuenta.
- Para el **staff (mozos, cocina, bartender, cajero):** ver pedidos relevantes a su estación de trabajo, en tiempo real, sin tener que andar entre mesas y cocina.

---

## 2. Mercado objetivo

| Aspecto | Definición |
|---|---|
| Geografía MVP | **Argentina** únicamente |
| Idioma MVP | **Español** únicamente |
| Moneda MVP | **ARS** únicamente |
| Local típico apuntado | **Mediano** (15–40 mesas, 150–400 pedidos/día) |
| Volumen objetivo año 1 | ~20 locales en primeros meses → ~100 locales fin de año |
| Mercados futuros | LATAM, multi-idioma y multi-moneda (post-MVP) |

---

## 3. Modelo de negocio

- **Modelo:** SaaS multi-tenant (muchos restaurantes en una sola plataforma).
- **Monetización:** **Freemium** — tier gratis con límites + tiers pagos con features Premium.
- **Sin piloto comprometido aún.** Acción crítica antes de salir a vender masivo: conseguir 1–3 locales piloto reales.
- **Sin facturación AFIP en MVP:** el restaurante factura por su lado con el sistema que ya use.

> **🟡 Pendiente de definir** (no bloquea el MVP, pero hay que cerrarlo antes de cobrar):
> - Qué features cobra cada tier y a qué precio.
> - Período de prueba pago.
> - Política de cancelación / downgrade.

---

## 4. Actores del sistema

| Actor | Quién es | Cómo accede |
|---|---|---|
| **Owner / Admin del local** | Dueño o gerente | Login web → panel admin |
| **Mozo** | Personal de salón | Login web → panel mozo |
| **Cocinero** | Personal de cocina | Login web → **pantalla de pedidos para cocina** (no es un KDS / POS — es una página simple con items pendientes y botón "marcar listo") |
| **Bartender** | Personal de barra | Login web → **pantalla de pedidos para barra** (en pantalla fija de la barra — NO en celular personal, manos mojadas) |
| **Cajero** | Personal de caja | Login web → vista de cuentas y cierres |
| **Comensal** | Cliente final del local | **Anónimo**, escanea QR → menú |
| **Super-admin PEDILO** | Vos / equipo PEDILO | Panel interno para administrar tenants, planes, soporte |

---

## 5. Alcance del MVP — Features que SÍ entran

### 5.1. Onboarding y multi-tenancy

- Registro self-service del dueño del local.
- Creación de la cuenta del restaurante (tenant).
- Configuración inicial: nombre, datos de contacto, una sucursal.
- **Multi-sucursal:** el modelo de datos lo soporta desde día 1, pero el MVP limita a **1 sucursal por cuenta**.

### 5.2. Panel admin

- Gestión de **mesas** como **listado simple** (número, capacidad, estado: libre / ocupada / cuenta abierta). Sin plano visual del salón en MVP.
- Generación automática de **QR estático descargable** por mesa (URL fija del tipo `pedilo.com/r/<slug>/m/<n>`). Si la mesa se elimina, el QR queda inválido.
- Gestión de **menú** (categorías) y **productos** (nombre, descripción, precio, foto, estación de destino: cocina/bar/otra).
- Toggle **disponible/agotado** por producto (Nivel A de inventario; sin stock numérico ni recetas).
- Gestión de **staff y roles** con **modelo de auth híbrido**:
  - **Owner / Admin del local:** login con **email + contraseña** (cuenta única, recuperación por mail).
  - **Staff** (mozo, cajero, cocinero, bartender): el admin crea **username + PIN de 4 dígitos**, sin necesidad de email. Login en tablet del local: lista de avatares → tap → PIN. Mitigaciones: lockout tras 5 intentos fallidos (15 min), admin puede resetear PIN, audit log de logins, rotación obligatoria cada 90 días.
- Vista de **pedidos en curso** y **cuentas de mesa abiertas**.
- Cerrar / marcar como pagada una cuenta de mesa.

### 5.3. Portal del comensal (cliente final)

**Acceso e identidad:**
- Acceso **anónimo** vía escaneo de QR de mesa. Sin descargar app, sin registro.
- Opcional: ingresar **nickname** para identificarse en la mesa (sugerido). Avatar/color asignado por el sistema.
- Cada dispositivo recibe un **device_id** persistente (cookie/localStorage).
- **Memoria del comensal:** el sistema recuerda al comensal por `device_id` en silencio (no se le muestra nada al respecto en MVP). Las features de loyalty visibles ("la última vez probaste X", "es la 3ra vez que venís") son **V1.x**. Una **cuenta opcional** para historial cross-device queda como **V2 (post-piloto)**, nunca obligatoria. Ver supuesto S12.

**Vista del menú (mejoras añadidas en v1.2):**
- Vista del menú con categorías, productos, fotos, precios.
- ⭐ **Calificación promedio del plato** (★★★★ + cantidad de votos visibles).
- ⭐ **Top 5 platos más pedidos** esta semana — sección destacada al inicio.
- ⭐ **Filtros del menú:**
  - "Platos rápidos" (tiempo de preparación promedio < 15 min, basado en histórico real).
  - "Mejor calificados" (orden por rating).
  - Tags dietéticos: vegano, sin TACC, picante (3-5 tags pre-definidos por admin).
- ⭐ **"Recomendado del chef"** (badge visible en productos que el admin destaca).
- ⭐ **Chip "para compartir / individual"** en cada producto.
- Disponibilidad real (productos marcados "agotado" no aparecen).

**⭐ "Mesa Viva" — diferenciador estrella (nuevo en v1.2):**
- Vista compartida en tiempo real de la mesa: cada comensal ve qué pidió cada uno con avatar + nickname + items + status.
- Animación cuando alguien suma un item ("Andrea agregó: Milanesa").
- Status por comensal visible: "decidiendo...", "listo para confirmar", "pedido enviado".
- Es el corazón de la experiencia social del producto.

**Pedido y comunicación:**
- Carrito y confirmación de pedido.
- Notas por item ("sin cebolla", "bien cocido"). Texto libre.
- ⭐ **"Llamar al mozo" con motivo:** agua, ayuda, pedir cuenta, queja. (Permite priorizar al mozo).
- Vista del estado de SU pedido: "recibido" → "en preparación" → "listo" → "entregado".

### 5.4. Sesión de mesa

- Al escanear el QR se abre/se une a la **sesión activa** de esa mesa.
- Si no hay sesión activa, se crea una nueva (en estado **pendiente de validación**).
> - **Dani:**  Que pasa si se escanea el QR alguien de otra mesa y se quiere sumar a la sesion? Alguien tiene que autorizar? Si se requiere autorizacion para entrar en al sesion de la mesa. Hace la experiencia mucho mas comclpicada y agrega friccion?
- **Validación del mozo (anti-abuso):** la primera vez que un comensal **confirma un pedido** en una sesión nueva, el pedido queda en estado **`pendiente_validacion_mozo`**. El mozo recibe una notificación con la mesa, hora e items, y debe confirmar "OK, hay gente sentada". Recién entonces el pedido pasa a `recibido` y entra a la cocina/bar. Una vez validada la mesa, los siguientes pedidos de la misma sesión van directo sin re-validación.
- La sesión se **cierra cuando el mozo marca la cuenta como pagada** desde el panel admin/mozo.
- Cada item de pedido lleva: `nickname` + `device_id` + `comensal_id` (futuro pago dividido sin migrar).
- **Escalamiento de inactividad** (backup ante mozo olvidadizo):
  - **2 hs sin actividad** → notificación al mozo y al cajero ("Mesa N hace 2hs sin actividad, ¿sigue activa?").
  - **4 hs sin actividad** → notificación al admin del local.
  - **6 hs sin actividad** → cierre automático de la sesión, marcado con flag `cerrado_por_timeout` para auditoría posterior.

### 5.5. Pedidos y ruteo

- **Estados del item de pedido** (cada item avanza independientemente porque va a estaciones distintas):
  - `pendiente_validacion_mozo` (solo el primer pedido de una sesión nueva — ver 5.4)
  - `recibido` → `en_preparacion` → `listo` → `entregado`
  - `cancelado` (puede ocurrir desde cualquier estado anterior a `entregado` — ej: cocina sin insumos, cliente se arrepiente). Toda cancelación queda registrada con motivo y autor.
- **Estado del pedido** (agregado): se deriva del estado de sus items, no se almacena como columna.
- **Tiempo transcurrido visible** en UI: cada item muestra "hace X min en preparación". Se prefiere esto sobre un estado `demorado` explícito en MVP — si más adelante medimos que un estado dedicado agrega valor, se suma.
- **Ruteo automático** del item según su estación (cocina, bar, otra).
- **Display por estación**: cocina ve solo lo de cocina, bar ve solo lo de bar.
- **Panel del mozo**: ve todos los pedidos de las mesas que atiende, llamadas pendientes y cuentas abiertas.
- **Tiempo real**: cualquier cambio de estado se refleja en <1–2 segundos en todas las pantallas relevantes, **sin necesidad de refrescar**.
> - **Dani:**  Los pedidos van a tener como identificador la mesa no? Supongo que lo ahblaremos en el modelo de datos
### 5.6. Cosas que vamos a definir en el doc de arquitectura pero ya están confirmadas

- PWA (web responsive) para todos los actores.
- Sin requerimiento offline.
- Sin app nativa.

**Hardware mínimo del local:**
- **Mozo, cajero, owner:** usan dispositivos que **ya tienen** (celular propio, PC del local).
- **Cocina (opcional pero recomendado):** tablet/laptop/monitor fijo. Inversión típica: USD 50–100 (tablet usada).
- **Bar (opcional pero recomendado):** tablet/monitor fijo en la barra. **NO el celular del bartender** — manos mojadas, riesgos de daño y falsos toques. Inversión típica: USD 50–100.
- **Locales chicos con cocina + bar cercanos:** pueden usar **una sola pantalla compartida** si la disposición lo permite.
- **Sin requerimiento de hardware específico** ni propietario. Web responsive funciona en cualquier dispositivo con browser moderno.
- **Para locales que NO quieren invertir en hardware** existe el **Modo "Mozo dicta"**: el mozo recibe los pedidos en su celular y los lleva a cocina/bar manualmente. Acepta el pasamanos del mozo a cambio de cero inversión. Comunicar claramente en el onboarding como modo alternativo.

---

## 6. Alcance del MVP — Features que NO entran (Premium o Post-MVP)

### 6.1. Premium / V1.x posibles

- 💳 **Pago online** (MercadoPago) con división de cuenta (mi parte / todo / partes iguales). V2 inicial Patrón A → V2.x Patrón B Marketplace con comisión PEDILO.
- ⭐ **Reseñas escritas por plato** (calificaciones promedio sí entran al MVP, pero las reseñas escritas con texto, moderadas, son Premium).
- 📊 **Reportes y analítica avanzada**: ventas del día, productos más pedidos, performance por mozo, tiempos de preparación por plato (rápidos vs lentos), métricas de optimización, comentarios negativos filtrados, heatmap de pedidos.
- 🏢 **Multi-sucursal real** (>1 sucursal por cuenta).
- 🎫 **Descuentos / promos simples** ("20% OFF hoy") — *posible MVP liviano si entra en tiempo*.
- 🖨️ **Impresión física de comandas** (impresoras térmicas ESC/POS) — *a investigar costo de implementación*.
- 🗺️ **Plano visual del salón** ("layout editor"): canvas con mesas posicionadas, zonas (terraza/interior/VIP), vista del mozo en plano.
- 🔁 **QR dinámico** (URL redireccionable, analytics de escaneo, invalidación de QR robados).
- 🤝 **"Modo Social" — interacción cross-mesa** (V1.x / V2 con safeguards). Comensales de mesas distintas pueden verse e intercambiar saludos públicos opt-in. **No es chat privado.** Requiere doble opt-in (local activa "Modo Social" + comensal acepta participar al escanear). Safeguards obligatorios: bloqueo en 1 tap, lista negra de palabras, reportes con ban automático, kill switch global, lanzamiento gradual en 2-3 locales piloto. Posicionado como feature destacado para cervecerías, after-office y bares jóvenes. **Riesgo alto si se diseña mal — diseño detallado pospuesto a post-piloto cuando haya data real de comportamiento social.**
- 🎁 **Cortesía del chef** — el cocinero manda emoji/mensaje al entregar plato premium.
- 🎉 **Cumpleaños / brindis de mesa** — opcional, opt-in del comensal.
- 💬 **Mozo manda mensaje a la mesa** — "tu pedido se demora 5 min".
- 🎯 **Loyalty / reconocimiento de cliente recurrente** — basado en `device_id`: "es la 4ta vez que venís", "la última vez probaste X".
- 📍 **Mapa de locales con PEDILO** (descubrimiento por comensales).
- 🔗 **Sumarse a mesa por código/link** (un amigo que llega tarde se suma a la sesión activa).
- 🍷 **Maridajes / "se acompaña con..."**.
- 📞 **Tiempo de espera estimado por plato** (basado en mediana real, no auto-declarado).

### 6.2. Descartado del MVP (definitivo)

- 📲 Notificaciones por WhatsApp / SMS al cliente.
- 🧾 Integración con sistemas de cocina (KDS) externos.
- 📅 Reservas de mesa.
- 🛵 Take-away / delivery.
- 🌍 Multi-idioma / multi-moneda.
- 📦 Stock numérico real / recetas / insumos.
- 🧮 Facturación AFIP.

---

## 7. Decisiones técnicas (resumen ejecutivo)

> *Detalle completo y trade-offs van en el Documento 02 (arquitectura y stack).*

| Capa | Decisión | Justificación corta |
|---|---|---|
| Frontend | **Next.js (App Router) + React + TypeScript** | El usuario sabe React/TS; Next resuelve SSR del menú público (SEO) y simplifica el deploy. |
| Backend | **API Routes / Server Actions de Next.js** | Co-localizado con frontend, sin servidor separado para mantener. |
| Base de datos | **PostgreSQL en Supabase** | Familiaridad con Postgres + multi-tenancy via RLS. |
| Auth | **Supabase Auth** (owner/admin) + capa custom **username+PIN** (staff) | Modelo híbrido — ver sección 5.2. |
| Realtime | **Supabase Realtime** | Pedidos en cocina sin escribir WebSockets a mano. |
| Storage | **Supabase Storage** | Fotos de productos. |
| Hosting frontend/API | **Vercel** | Familiaridad, deploy en 1 click. |
| Pagos (V2) | **MercadoPago** | Estándar Argentina. |

**Costo estimado de infra durante validación: USD 0–45/mes** (cabe en budget de USD 100/mes con margen).

> ✅ **Resuelto:** observabilidad/logs y seguridad/abuso quedan cubiertos a alto nivel en la nueva **Sección 12: Requisitos no funcionales**. El detalle técnico va en el Documento 02 (arquitectura).

---

## 8. Restricciones operativas

| Restricción | Valor |
|---|---|
| Tamaño de equipo | 1 desarrollador (vos) |
| Modalidad | Side project después del trabajo |
| Dedicación | 8–16 hs/semana |
| Timeline MVP técnico | ~3 meses |
| Timeline MVP en piloto real | **4–5 meses** |
| Budget infra | USD 100/mes máximo durante validación |

---

## 9. Supuestos abiertos (proponer defaults para confirmar)

Estos son puntos que no quedaron explícitamente cerrados durante el descubrimiento. Propongo defaults razonables — **revisalos y decime si los aceptás o cambiás**:

| # | Supuesto | Default propuesto |
|---|---|---|
| S1 | **Fotos de productos en MVP** | ✅ SÍ. Sin foto la conversión cae fuerte en gastronomía. |
| S2 | **Atributos de producto** (vegano, sin TACC, picante, alérgenos) | 🟡 Versión simple en MVP: 3–5 tags pre-definidos opcionales. Sin alérgenos detallados ni calorías. |
| S3 | **Moneda** | ✅ ARS único en MVP. |
| S4 | **Notas del comensal al pedido** ("sin cebolla", "bien cocido") | ✅ SÍ. Campo de texto libre por item. Es de costo bajo y de alto valor. |
| S5 | **Modificadores de producto** (ej: tamaño S/M/L, agregar queso +$500) | 🟡 NO en MVP. Costo de UI alto, dejamos para V1.x. Si el restaurante tiene variantes, las carga como productos separados. |
| S6 | **Timeout de cierre de mesa** (backup ante mozo olvidadizo) | ✅ **Confirmado y refinado:** escalamiento 2h notif staff → 4h notif admin → 6h cierre automático. Detalle en sección 5.4. |
| S7 | **Rol "Manager" o "Encargado"** (entre Owner y Staff) | 🟡 NO en MVP. Owner + 4 roles operativos alcanzan. |
| S8 | **Vista del comensal del estado de su pedido** ("en preparación", "listo") | ✅ SÍ. Vital para la experiencia y reduce llamadas al mozo. |
| S9 | **Formato de QR** | ✅ **Estático** confirmado. URL fija `pedilo.com/r/<slug>/m/<n>`. Sin servicio de redirección. |
| S10 | **Vista de mesas en panel admin** | ✅ **Listado simple** confirmado. Plano visual del salón queda como Premium / V1.x. |
| S11 | **Modelo de auth para staff** (creación de usuarios por admin) | ✅ **Híbrido confirmado:** owner usa email+password (Supabase Auth); staff usa username+PIN gestionado por admin (sin email). Detalle en sección 5.2. |
| S12 | **Memoria del comensal entre visitas** (¿lo recordamos? ¿necesita cuenta?) | ✅ **Estrategia escalonada:** **MVP** = recuerdo silencioso por `device_id` (sin mostrar nada al usuario). **V1.x** = features visibles de loyalty ("la última vez probaste X", "es la 3ra vez que venís"). **V2 (post-piloto)** = cuenta opcional para historial cross-device, nunca obligatoria. Mantiene el wedge "zero friction" del modelo de adopción PedidosYa-style. |

> ✅ **Resuelto:** los 12 supuestos confirmados por Dani el 2026-05-01.

---

## 10. Riesgos identificados

| # | Riesgo | Mitigación propuesta |
|---|---|---|
| R1 | **Sin piloto comprometido** → riesgo de construir lo que nadie pide. | **Refinado:** primero **wireframes en Figma** (Documento 04), luego prototipo descartable de Next.js, en paralelo conseguir 2–3 dueños conocidos para mostrarles el Figma y escuchar feedback. Sin outreach masivo que retrase el desarrollo. |
| R2 | **Timeline ajustado** (140–220 hs estimadas para 96–192 hs disponibles). | **Aceptado por Dani:** sin restricción dura de meses ni horas si hace falta extender. Igual mantener lista de "features a recortar" como red de seguridad. |
| R3 | **Vendor lock-in con Supabase**. | Aceptado conscientemente. Costo de migración futura es asumible si el negocio funciona. |
| R4 | **Curva de Next.js** (15–25 hs estimadas). | **Confirmado:** prototipo descartable de 1–2 fines de semana antes del código real. |
| R5 | **Adopción del staff del local** (mozos viejos resisten cambio). | Login con username+PIN en lugar de email/password (S11) reduce fricción. UI ultra-simple para mozo. Considerar impresión física como puente para cocinas tradicionales (V1.x). |
| R6 | **Cierre incorrecto de sesiones de mesa** → cliente nuevo ve cuenta vieja. | Escalamiento de inactividad (sección 5.4) + visualización clara del estado de la mesa al escanear el QR. Probar exhaustivamente. |
| R7 | **Conectividad inestable en locales** (wifi malo, datos saturados). | Banner "reconectando" en UI, reintentos exponenciales, no perder pedidos en cola local del cliente. |
| R8 | **Abuso del sistema por extraños** (pedido fantasma desde casa, escanear QR de mesa ajena, spam "llamar al mozo", notas ofensivas, etc.) | **Validación del mozo en primer pedido** (sección 5.4) + rate limiting por device_id e IP + filtro de palabras en nicknames y notas + indicador "mesa ocupada" al escanear. Detalle completo en Documento 02. |
| R9 | **Observabilidad insuficiente en producción multi-tenant** — un local reporta un problema y no podemos diagnosticar. | Logs estructurados con `tenant_id` desde día 1 + Sentry para errors + Better Stack/Logtail para logs centralizados + uptime monitoring. Detalle en sección 12 y Documento 02. |
| R10 | **Competencia con módulos QR de POS dominantes** (Fudo +25K negocios, Maxirest +28K registrados): si el local ya tiene Fudo Premium con QR activo y satisfecho, NO va a cambiar a PEDILO. | **Camino B confirmado** (ver `notas-investigacion-mercado.md` §4): PEDILO **convive** con el POS, no lo reemplaza. Target principal: 50K-95K locales SIN Fudo/Maxirest activos. Diferenciadores estructurales irreproducibles: tier free real, modelo PedidosYa-style sin demo/comercial, Mesa Viva (vista live grupal), validación del mozo, Patrón B Marketplace. Análisis competitivo detallado en `notas-investigacion-mercado.md`. |

---

## 11. Requisitos no funcionales y temas transversales

> *Resumen a alto nivel. El detalle técnico de implementación va en el Documento 02 (arquitectura).*

### 11.1. Observabilidad y logs

- **Logs estructurados (JSON):** cada línea de log incluye `tenant_id`, `user_id`, `request_id` y nivel (`info` / `warn` / `error`). Esto permite filtrar por restaurante específico cuando uno reporte un problema.
- **Error tracking:** **Sentry** en frontend y backend (free tier: 5k errores/mes), con stack traces, breadcrumbs y release tracking.
- **Logs centralizados (cuando crezca):** **Better Stack / Logtail / Axiom** (free tier ~3GB/mes). Hasta entonces, los logs nativos de Vercel + Supabase alcanzan.
- **Alertas:** errores críticos disparan email / Slack via Sentry.
- **Uptime monitoring:** Better Stack o UptimeRobot (free tier).
- **Métricas de negocio** (pedidos por tenant, por estación, etc.): queries directas a Postgres + dashboard interno simple. No requiere herramienta dedicada en MVP.

**Costo estimado adicional: USD 0–10/mes** durante validación.

### 11.2. Seguridad y prevención de abuso

Vectores de ataque identificados (R8) y mitigaciones MVP:

| # | Ataque | Mitigación |
|---|---|---|
| A1 | Imprimir QR y pedir desde casa | **Validación del mozo** en primer pedido (sección 5.4). |
| A2 | Extraño escanea QR de mesa ajena y agrega items | Validación del mozo + nickname visible + notificación al resto de comensales cuando alguien nuevo se une. |
| A3 | Spam de "llamar al mozo" | Rate limit: 1 llamada cada 2 minutos por sesión. |
| A4 | Pedidos automatizados (bot) | Rate limit por `device_id` e IP en endpoint de pedido. |
| A5 | Notas/nickname ofensivos | Filtro simple de palabras (lista negra editable por admin) + reporte/baneo manual. |
| A6 | Cliente nuevo ve cuenta vieja | Estado claro de mesa al escanear: si la sesión está cerrada, abre nueva; visualización del estado de la mesa. |
| A7 | QR robado/distribuido en redes | Mitigación parcial vía validación del mozo. Mitigación completa requiere QR dinámico (Premium / V1.x). |
| A8 | Brute-force de PIN de staff | Lockout tras 5 intentos fallidos (15 min) + alerta al admin. |

### 11.3. Multi-tenancy y aislamiento de datos

- **Postgres Row Level Security (RLS)** activado en todas las tablas con `tenant_id`. Cada query en runtime se restringe automáticamente al tenant del usuario autenticado. Detalle en Documento 02.
- **Tests obligatorios** que verifiquen que un usuario de tenant A no puede leer/escribir datos de tenant B (la vulnerabilidad #1 más común en SaaS).

### 11.4. Performance objetivos (MVP)

- **Tiempo real de pedidos:** propagación a cocina/bar/mozo en **< 2 segundos** desde la confirmación del comensal.
- **Carga inicial del menú** (vista del comensal al escanear QR): **< 2 segundos** en 4G de Argentina.
- **Operaciones del staff** (cambiar estado de un item, cerrar cuenta): **< 500 ms** percibidos (con UI optimista y reconciliación).

### 11.5. Privacidad y datos personales

- Comensales son anónimos (nickname + device_id). No se almacena PII del cliente final más allá de eso.
- Owner y staff: email + datos de contacto del local. Cumplimiento básico Ley 25.326 (Protección de Datos Personales, Argentina).
- Política de privacidad y términos de servicio: pendiente redactar antes del piloto.

### 11.6. Operación y soporte productivo

> *Esta sección define los requisitos operativos a alto nivel. El playbook completo (procedimientos, plantillas, SLAs por tier, on-call) se documentará en el futuro **Documento 06: Operación y Soporte Productivo**, cerca del piloto, cuando ya se pueda definir contra clientes reales.*

**Requisitos operativos del MVP:**

| # | Requisito | Decisión MVP / a definir |
|---|---|---|
| O1 | **Canal de soporte al local** | MVP: WhatsApp Business + email (`soporte@pedilo.com`). Sin chat in-app ni teléfono 24/7. |
| O2 | **Tiempo de respuesta** | MVP: best-effort 24 hs hábiles. SLA formal por tier se define en Doc 06. |
| O3 | **Status page público** (`status.pedilo.com`) | MVP: usar **Better Stack Uptime** o **Instatus** (free tier) para publicar estado de servicios. Importante para que un dueño vea "no es solo a mí" cuando hay incidente. |
| O4 | **Proceso de incidente** | Playbook básico documentado: detectar → comunicar → diagnosticar → mitigar → postmortem. Detalle en Doc 06. |
| O5 | **Runbook de problemas conocidos** | Documento vivo (`docs/runbook.md`) que se va llenando con cada incidente: síntoma → causa → solución. Iniciar vacío y crecer con la operación. |
| O6 | **Backups de base de datos** | Supabase Pro hace backup diario automático con 7 días de retención. **Simulacro de restauración** obligatorio antes del primer piloto pago (verificar que el backup realmente funciona). RPO objetivo: 24 hs. RTO objetivo: 2 hs. |
| O7 | **Rollback rápido de deploys** | Vercel mantiene deploys anteriores y permite revertir en 1 click. Objetivo: rollback en **< 5 minutos** desde detección. |
| O8 | **Bug reporting in-app** | Botón "reportar problema" en panel admin y mozo que adjunta automáticamente: `tenant_id`, `user_id`, URL actual, último error de consola, timestamp. Reduce dramáticamente la calidad de los reportes. **Confirmar para MVP**. |
| O9 | **Métricas a vigilar diariamente** | Tablero interno con: errores nuevos en Sentry, uptime, pedidos por tenant, tiempo de respuesta P95, tenants activos hoy. **5 métricas máximo** — más se vuelve ruido. |
| O10 | **On-call / horario de atención** | MVP: vos sos el on-call. Alertas críticas (caída de servicio, error rate alto) llegan por push/email/Slack. Definir horarios de "guardia activa" cuando haya clientes pagando. |
| O11 | **Comunicación de incidentes** | MVP: post en status page + email/WhatsApp a clientes afectados. Plantillas de mensaje pre-redactadas (Doc 06). |
| O12 | **Postmortem** | Tras incidente con impacto a clientes: postmortem escrito en `docs/postmortems/YYYY-MM-DD.md` con timeline, causa raíz, acciones de mejora. Cultura sin culpa. |

**Pendientes para Documento 06 (operación detallada, cerca del piloto):**

- SLAs formales por tier (free vs Premium): uptime garantizado, tiempo de respuesta.
- Plantillas de comunicación de incidente.
- Procedimiento de onboarding de nuevo local (de venta a producción).
- Procedimiento de offboarding (cliente se da de baja).
- Política de retención de datos al dar de baja.
- Manual del staff PEDILO (cuando contrate gente).

---

## 12. Pendientes antes de pasar al Documento 02 (arquitectura)

- [x] **Vos:** revisar este documento, marcar correcciones / cambios. ✅ Hecho 2026-05-01.
- [x] **Vos:** confirmar o ajustar los supuestos abiertos (sección 9). ✅ Los 11 supuestos confirmados con refinamientos en S6 y nuevo S11.
- [x] **Vos:** decidir si aceptás los riesgos identificados o querés mitigaciones distintas (sección 10). ✅ Aceptados con refinamientos en R1, R2, R4 y nuevos R8 y R9.
- [ ] **Yo:** redactar Documento 02 con la arquitectura detallada, el stack final, decisiones de seguridad multi-tenant (RLS), estrategia de auth híbrida, observabilidad, y diagramas.

---

## Glosario rápido

- **Tenant / cuenta:** un restaurante o bar registrado en PEDILO.
- **Sucursal:** un local físico de un tenant (en MVP, máx. 1).
- **Mesa:** entidad física con un QR único.
- **Sesión de mesa:** período entre que un grupo se sienta y la cuenta se cierra como pagada.
- **Comensal:** persona individual dentro de una sesión de mesa, identificada por nickname + device_id.
- **Pedido:** conjunto de items que un comensal confirma.
- **Item:** un producto dentro de un pedido (con cantidad, notas, etc.).
- **Estación:** destino de un item para preparación (cocina, bar, otra).
- **Cuenta de mesa:** suma total de los pedidos de una sesión.
