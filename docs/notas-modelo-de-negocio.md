# PEDILO — Notas preliminares de Modelo de Negocio y Go-to-Market

> **Estado:** 📋 Borrador de notas / pre-Doc 07
> **Fecha:** 2026-05-01
> **Propósito:** preservar la conversación inicial sobre pricing, mercado, proyección de ingresos y canales de venta. Este archivo es **insumo del futuro Documento 07: Modelo de Negocio y Go-to-Market**, que se redactará formalmente más adelante.

---

## 1. Estructura de tiers — esquemas evaluados

Se evaluaron 3 esquemas de tiers para el modelo Freemium:

### Esquema A — 2 tiers (Free + Premium) ⭐ recomendado para MVP

| | **Free** | **Premium** |
|---|---|---|
| Mesas | hasta 5 | ilimitadas |
| Productos | hasta 30 | ilimitados |
| Staff | hasta 3 usuarios | ilimitados |
| Pedidos/mes | hasta 200 | ilimitados |
| Soporte | comunidad / FAQ | WhatsApp + email |
| Pago online | ❌ | ✅ |
| Reportes y analítica | ❌ | ✅ |
| Reseñas/calificaciones | ❌ | ✅ |
| Branding propio en menú | ❌ (logo PEDILO visible) | ✅ |
| Multi-sucursal | ❌ | ✅ (cuando esté) |

### Esquema B — 3 tiers (Free + Starter + Premium)

Suma un tier intermedio "Starter" (~USD 12/mes, sin pago online ni reportes pesados). Más matiz pero más complejo de comunicar.

### Esquema C — Pago por uso (transaccional puro)

% por pedido procesado en lugar de mensualidad. Descartado: ingresos impredecibles, no sostiene costos fijos.

**Decisión preliminar: Esquema A en MVP.** Si más adelante se pierden clientes que querían "algo en el medio", se evalúa sumar Starter (Esquema B).

---

## 2. Precio y moneda

**Decisión preliminar:**
- **Moneda:** USD con cobro automático en ARS al tipo de cambio del día. Estándar de SaaS B2B en Argentina, protege contra inflación.
- **Premium mensual:** **USD 25/mes** (a confirmar tras piloto).
- **Premium anual:** USD 250/año (2 meses gratis = -16%) para incentivar compromiso.

**Anclajes de mercado (Argentina, 2026):**

| Producto | Precio mensual aprox. |
|---|---|
| MOZO (AR) | USD 15–35 según features |
| Yuoosh (AR) | USD 20–40 |
| MenuDigital (AR) | USD 10–20 (más simple) |
| Toast (USA, comparable global) | USD 70–150 (otro mercado) |

> Sweet spot probable para PEDILO: USD 19–29/mes Premium.

---

## 3. Tamaño del mercado

### Buenos Aires Capital (CABA)

| Categoría | Aprox. |
|---|---|
| Establecimientos gastronómicos totales registrados | 25.000–30.000 |
| Activos con servicio de mesa (excluye delivery puro) | 10.000–15.000 |
| **Target real para PEDILO** (medianos, tech-receptivos, PYMEs) | 3.000–5.000 |

### Argentina total

| Categoría | Aprox. |
|---|---|
| Establecimientos gastronómicos | 80.000–120.000 |
| **TAM** (todos los locales que conceptualmente podrían usar PEDILO) | 30.000–40.000 |
| **SAM** (al alcance comercial real, urbanos, conectados) | 15.000–20.000 |
| **SOM realista a 3–5 años** (1–3% del SAM) | **200–600 locales pagos** |

---

## 4. Proyección de ingresos — escenarios

**Supuestos base:**
- ARPU = USD 32 (mix Premium USD 25 + Pro USD 49 al 70/30%)
- Conversión free → paid = 10%
- Patrón B (commission) eventual: 1% sobre 30% de pedidos pagados online × USD 25 ticket promedio

### Tabla reconciliada (la versión correcta tras revisión del 2026-05-01)

| Locales totales | Activos en pico (~30%) | Paid (10%) | MRR sin commission | MRR con commission Patrón B | Costo infra | Margen aprox. |
|---|---|---|---|---|---|---|
| 100 | ~30 | 10 | USD 320 | USD 320–800 | USD 25–45 | USD 275–755 |
| 300 | ~100 | 30 | USD 960 | USD 1.500–2.500 | USD 100–200 | USD 800–2.300 |
| 500 | ~150 | 50 | USD 1.600 | USD 3.000–5.000 | USD 200–400 | USD 1.400–4.600 |
| 1.000 | ~300 | 100 | USD 3.200 | USD 8.000–15.000 | USD 500–1.000 | USD 2.700–14.000 |
| 1.500 | ~500 | 150 | USD 4.800 | USD 15.000–27.000 | USD 1.000–2.000 | USD 3.800–25.000 |

### Lecturas honestas

- **MVP a 12 meses (50–200 locales totales):** infra USD 25–45/mes, holgado. Revenue magro pero esperable en validación.
- **Año 2 (200–500 locales totales):** infra empieza a sentirse, margen lo cubre cómodo.
- **Año 3+ (500–1.000 locales totales):** si el Patrón B (commission) está activo, MRR escala bien. Sin commission, sigue siendo side income.
- **El número MRR depende dramáticamente de si activás Patrón B.** Sin commission, PEDILO es side income decente. Con commission, escala bien.

### Hitos de revenue

- **USD 1.000–2.000 MRR:** sale de "side project hobby". Probablemente con ~400–800 locales totales.
- **USD 5.000+ MRR:** considerar full time.
- **USD 10.000+ MRR:** primera contratación posible.

---

## 5. Palancas de optimización de revenue (sin cambiar el producto)

Identificadas 7 palancas para subir el techo:

1. **Tier "Pro" arriba de Premium** (USD 49–69) con multi-sucursal, reportes avanzados, integración impresora, soporte prioritario, branding 100% custom. **Si 30% de paid van a Pro, ARPU sube de USD 25 a USD 32 (+28% MRR).**
2. **% por transacción en pago online** (V2 Patrón B): 0,5–1% sobre el ticket procesado, encima de comisión MP. **La palanca más potente** — puede 5x el MRR.
3. **Cobrar por sucursal:** 1 cuenta con 3 sucursales paga 3× Premium.
4. **Setup fee único** USD 49–99 al onboarding.
5. **Add-ons facturados aparte:** impresión QRs premium, diseño custom de menú, capacitación staff, reseller de impresoras térmicas.
6. **Plan anual con descuento** (USD 250/año = -17%). Cash upfront + reduce churn.
7. **Foco en segmento alto** (medianos-altos con 25–60 mesas) en vez de competir en mercado masivo barato.

---

## 6. Estrategia de venta / promoción

Canales evaluados para gastronomía en Argentina, ordenados por efectividad real:

| Canal | Costo | Velocidad | ROI |
|---|---|---|---|
| **Boca a boca** entre dueños de bares | $0 | Lenta (3-12 meses) | 🟢 Altísimo. **Canal #1 en gastronomía** |
| **Visita en frío** (caminar barrios y demostrar en tablet) | Tiempo | Media | 🟢 Funciona si hay piel para vender |
| **Programa de referidos** (1 mes gratis por local que traigas) | Bajo | Media | 🟢 Activa boca a boca |
| **Partnerships** (asociaciones gastronómicas AHRCC/FEHGRA, distribuidores de bebidas, contadores especializados) | Tiempo de relacionamiento | Lenta | 🟢 Acceso a docenas de locales |
| **Instagram orgánico** (posts mostrando la app en uso real) | Tiempo | Lenta | 🟡 Útil para credibilidad |
| **Google Ads / Meta Ads** | USD 100-500/mes | Rápida | 🟡 Caro para B2B, ROI dudoso al inicio |
| **Influencers gastronómicos** | USD 200-1000/post | Rápida | 🔴 Caro y difícil de medir |
| **Cold email** | $0 | Lenta | 🟡 Bajo conversion |
| **Eventos / ferias gastronómicas** (ej: ExpoFood) | USD 500-2000/stand | Media | 🟢 Bueno para credibilidad y volumen de leads |

### Plan recomendado primeros 3-6 meses post-lanzamiento

1. **Conseguir 3–5 pilotos gratis** con dueños conocidos para generar casos de éxito y testimonios.
2. **Visitas en frío estructuradas:** 5 visitas/semana con demo en tablet. ~30 visitas/mes → razonable esperar 2–4 conversiones a Free.
3. **Programa de referidos** desde el día 1: "tráeme un local, te regalo 1 mes Premium".
4. **Instagram orgánico** mostrando capturas reales del producto en uso — para credibilidad.
5. **Partnership clave a explorar:** distribuidores de bebidas (Quilmes, CCU). Visitan locales todas las semanas.
6. **NO en MVP:** Google Ads, Meta Ads, influencers. Caros y de bajo ROI con marca desconocida.

---

## 7. Pendientes para el Documento 07 formal

- [ ] Definir features finales por tier free / premium / pro.
- [ ] Política de período de prueba (¿14 días Premium gratis?).
- [ ] Política de cancelación / downgrade.
- [ ] CAC objetivo y payback period.
- [ ] LTV proyectado y churn esperado.
- [ ] Plan de pricing review (ej: revisar precios cada 6 meses con inflación USD).
- [ ] Estrategia de branding y posicionamiento.
- [ ] Calendario de lanzamiento.
- [ ] Plantillas de pitch para visitas en frío.
- [ ] Análisis competitivo detallado (MOZO, Yuoosh, MenuDigital, etc.).

---

> **Notas operativas:**
> - Estas notas se redactaron en sesión del 2026-05-01 a pedido de Dani. No reemplazan el Doc 07, que se hará formalmente en paralelo con Docs 04/05.
> - Cualquier número de mercado puede estar desactualizado — verificar fuentes (AHRCC, INDEC, Cámara de Comercio) cuando se redacte el Doc 07.
