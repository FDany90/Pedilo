# PEDILO — Notas de investigación de mercado

> **Estado:** ✅ Borrador completo (v1.0) — listo como insumo de Docs 04 y 07
> **Fecha:** 2026-05-01
> **Propósito:** documentar pain points reales, barreras de adopción, sistemas existentes, posicionamiento y argumentos de venta para informar **Doc 04 (UI/UX)** y **Doc 07 (GTM)**.
> **Foco:** bares/cervecerías + restaurantes a la mesa, mercado primario Argentina, contraste USA/México.
> **Caveat:** estas notas se basan en research desk + análisis competitivo. **No reemplazan entrevistas con dueños reales** — eso queda para la fase de piloto. Aún así sirven para tomar decisiones informadas hoy.

---

## 1. Estado actual: qué sistemas usan los restaurantes hoy

### 1.1. Argentina — Sistemas de gestión completa (POS)

Los locales medianos/grandes (>15 mesas) en Argentina usualmente tienen UN sistema POS principal que cubre toda la operación. Los más populares:

| Sistema | Precio aprox. | Adopción | Qué cubre | Observación |
|---|---|---|---|---|
| **Fudo** | ARS 19.500 (Inicial) → ARS 60.000 (Pro) + add-ons | **+25.000 negocios AR** | POS completo: pedidos, mesas, cocina, delivery (PedidosYa/Rappi), pagos online (MP), facturación, reportes, app de mozos, **menú QR + autopedido**, monitor de cocina. **API pública disponible**. | El más reconocido en AR. Crecimiento fuerte post-COVID. **Su QR tiene autopedido funcional** — no es solo un menú estático. Pero su QR es 1 módulo de 20, no su foco principal. |
| **Maxirest** | Desde ARS 11.996 + IVA (no publicado, solo por demo) | 28.000 registrados / **8.000 DAU** | POS completo: gestión, delivery, take-away, **menú QR con carrito + identificación de comensal (lado mozo)**, monitor de cocina, mobile ordering, reservas, reportes en tiempo real, facturación electrónica gratis, integraciones con Waitry/OrderFast | **Quejas frecuentes**: UI anticuada, soporte lento, contratos difíciles de cancelar, mantenimiento caro. Mercado direccionable: hay clientes deseando irse pero "atrapados". |

**Pricing público:** Fudo lo publica, Maxirest no (requiere demo + comercial).

**Hardware:** ambos funcionan en cualquier dispositivo cloud (sin hardware propietario obligatorio), pero suelen vender impresoras térmicas y equipos como add-on.

**Modelo de venta:** ambos siguen **modelo de demo + comercial humano**. Ningún signup self-service inmediato.

**Insight clave 1:** los POS completos en Argentina **YA INCLUYEN menú QR como módulo dentro de su oferta**. PEDILO no compite contra "no tienen QR" — compite contra "el QR ya viene con su POS, pero como uno más entre 20 features, no es el corazón del producto".

**Insight clave 2:** el modelo "demo + capacitación + contrato" es estándar en este segmento. **PEDILO con modelo PedidosYa-style (signup → activo en 10 min) es estructuralmente distinto y difícil de imitar para ellos** — porque cambiarlo significa cambiar su modelo de negocio entero.

### 1.2. Argentina — Sistemas de menú QR independientes (sin POS)

Para locales chicos o que solo quieren un menú digital sin reemplazar lo que ya tienen:

| Sistema | Modelo | Observación |
|---|---|---|
| **MOZO Digital** (mozo.digital) | Suscripción mensual, foco en menú QR personalizado | Marca conocida, miles de restaurantes según su web. Activo en CABA, Córdoba, Bariloche, San Luis. Casos: La Liga Bar, Mandinga, Lo de Miguel, Lamaska, Errantes. |
| **SoyMenu** (soymenu.com.ar) | Tiers freemium con plan pago por features | Posicionado como solución simple. |
| **BuenaCarta** | Plan gratis disponible | Foco en bajo costo / gratis. |
| **Cartadigital.gratis** | Gratis | Producto muy básico, monetiza con upsells. |
| **Simplemenú** | Suscripción "muy barata" según prensa | Push de adopción en el segmento más bajo. |

**Insight clave:** el segmento "menú QR independiente" es **un mercado commoditizado** con varios jugadores gratuitos. **PEDILO no puede diferenciarse solo por "tener menú QR"** — eso ya es commodity. La diferenciación viene por la **experiencia social del comensal y el modelo de pagos** (Patrón B Marketplace).

### 1.3. Argentina — Otros sistemas usados en paralelo

- **Pagos:** MercadoPago domina (95%+ de pagos digitales locales). Naranja, GetNet, Modo siguen.
- **Reservas:** Restorando (mercado mediano-alto), OpenTable (alta gama).
- **Delivery agregado:** PedidosYa, Rappi (algunos locales). Convive con take-away propio.
- **Stock e insumos:** muchos siguen en planilla Excel, mensaje WhatsApp con proveedor, libreta. Solo locales grandes usan módulos del POS.
- **Caja:** controlador fiscal físico (Hasar, Epson, NCR) en locales registrados que facturan formalmente.

### 1.4. Latinoamérica — Meniu (NO competidor real en in-restaurant)

> **Veredicto definitivo (confirmado tras research, 2026-05-01):** Meniu **NO es competidor de PEDILO en el espacio in-restaurant live**.

**Por qué:** Meniu es un producto brasilero (4.000 restaurantes en LATAM, multi-país sin foco AR) cuyo modelo es **menú digital + WhatsApp ordering**. El comensal escanea QR, ve menú, arma pedido, y la app abre WhatsApp con un mensaje pre-armado al restaurante. **No tiene tiempo real nativo, ni cuenta compartida en mesa, ni identidad social del comensal.**

| Aspecto | Meniu |
|---|---|
| Pedido | Por **WhatsApp** (no nativo) |
| Cuenta compartida en mesa | ❌ |
| Identidad social del comensal | ❌ |
| Tiempo real a cocina | ❌ (depende de leer WhatsApp) |
| Tier free | 7-day trial |
| Foco | Delivery + take-away + menú |
| País | 🇧🇷 Brasil — sin presencia AR significativa |

**Lo bueno** que pudimos tomar como referencia (no como competidor):
- Setup self-service en <5 min.
- Plan único simple (no múltiples tiers complejos).
- Multi-idioma desde infraestructura.
- Marketing tipo "alternativa gratis a X" (estrategia SEO interesante).

**Lo que NO copiamos:**
- Pedido por WhatsApp (mata UX, no escala, pierde tiempo real).
- Modelo genérico multi-país (PEDILO se especializa AR primero).

### 1.5. México — Referencia regional

| Sistema | Características destacadas |
|---|---|
| **Toteat** | POS todo-en-uno (cloud, escalable). Maneja pedidos en mesa, barra, cocina, delivery. Inventario con recetas. Integraciones con Uber Eats, PedidosYa. Implementación en 48hs con capacitación. Dominante en México y expandiendo a la región. |
| **Boomerang** | No aparece dominante en search inicial — pendiente revisar. |

**Insight clave:** el mercado mexicano está **más consolidado y maduro** que el argentino. Toteat se posiciona como "todo en uno" — la dirección hacia donde van los POS modernos.

### 1.6. USA — Estado del arte global

- **Toast** y **Square for Restaurants** son los líderes. Cobran USD 70–150/mes con hardware incluido (tablets, terminales, impresoras) + comisión por transacción.
- **Modelo dominante:** subscripción + hardware + transactional fees (split de pago / comisión). Mercado mucho más sofisticado y caro.
- **Adopción:** según un estudio de Hospitality Tech, el 75% de restaurantes independientes en USA planeaba adoptar nueva tecnología en 2023. Tendencia sostenida.
- **Lección:** el modelo Toast/Square está demostrado en mercados desarrollados. **El modelo PEDILO + Patrón B (commission) replica esa estructura adaptada a precio AR.**

### 1.7. Gaps y oportunidades — donde PEDILO juega

Tras todo el análisis, **estas son las 6 cosas que ningún competidor argentino hace bien hoy**:

1. ⭐ **Cuenta compartida con identidad social del comensal** (nicknames + avatares + vista live del grupo). Maxirest tiene "identificación de comensal" pero solo del lado mozo (para tickets), NO la experiencia social entre comensales.
2. ⭐ **"Mesa Viva"** — vista live del pedido grupal donde cada comensal ve qué pidieron los demás en tiempo real.
3. ⭐ **División de cuenta automática desde la app** del comensal (sin pasar por mozo) — habilitada por Patrón B Marketplace.
4. ⭐ **Validación del mozo en primer pedido** (anti-abuso) — refuerza el rol del mozo en lugar de reemplazarlo.
5. ⭐ **Comisión transparente por transacción** con foco específico en QR-pago directo (Patrón B Marketplace). Los POS tradicionales cobran solo suscripción.
6. ⭐ **Onboarding self-service en <10 minutos** sin demo, sin comercial, sin contrato. Modelo PedidosYa-style — estructuralmente distinto al de Fudo/Maxirest.

---

## 2. Pain points reales

### 2.1. Pain points operativos (mozos, cocina, caja)

| Pain point | Severidad | Cómo PEDILO ayuda |
|---|---|---|
| **Falta de personal calificado** (cocineros, parrilleros, bartenders) | 🔴 Crítica AR | PEDILO no resuelve la contratación, pero **reduce la dependencia del mozo "pasamanos"** entre cocina y mesa. |
| **Alta rotación de staff** (3–6 meses promedio) | 🔴 Alta | UI ultra-simple + login PIN → **capacitación de 5 min**. Reemplazar a un mozo no requiere re-entrenamiento. |
| **Mozo "pasamanos"** (corre entre cocina, mesa, caja todo el tiempo) | 🟡 Media-alta | Pedidos van **directo del comensal a cocina** (con validación). Mozo coordina, no tipea. |
| **Errores en comandas tomadas a mano** | 🟡 Media | El comensal escribe su pedido — sin malinterpretación. |
| **Cocina con papel + ruidos + manchas** | 🟡 Media | Display digital (tablet) con estado de cada item. Opcional impresión térmica V1.x. |
| **Caja descoordinada con mesa** | 🟡 Media | Cajero ve pedidos en tiempo real → cobra rápido cuando llega la mesa. |

### 2.2. Pain points del dueño

| Pain point | Severidad | Cómo PEDILO ayuda |
|---|---|---|
| **Costos altos y rentabilidad cayendo** | 🔴 Crítica AR (post-COVID + inflación + caída facturación 2024-2025) | **Tier FREE** = arranque sin riesgo. Solo paga si genera valor. |
| **Falta de visibilidad de qué pasa en el local** | 🟡 Alta | Reportes (Premium) muestran qué se vende, qué demora, qué se cancela. |
| **Atrapamiento contractual con POS legacy** (caso Maxirest) | 🟡 Media | PEDILO **convive con el POS actual** — no obliga a migrar. Bajo riesgo de adopción. |
| **Capacitar staff es caro y se va al mes** | 🟡 Alta | UI obvia + PIN simple. Capacitación cero. |
| **Quiere modernizar pero no quiere pagar consultor** | 🟡 Media | Self-service total. **No hay comercial que llame.** |

### 2.3. Pain points del cliente final (comensal)

| Pain point | Severidad | Cómo PEDILO ayuda |
|---|---|---|
| **Mozo tarda en tomar el pedido en hora pico** | 🔴 Alta | Pide directo desde el celular. Sin esperar. |
| **Errores en su pedido** ("yo pedí sin cebolla") | 🟡 Media | El comensal escribe sus notas. Quedan registradas. |
| **No sabe cuánto va a tardar su plato** | 🟡 Media | Tiempo de espera estimado por plato (V1.x). |
| **Pide la cuenta y tarda eternidad** | 🔴 Alta | Botón "pedir cuenta". Cajero la prepara mientras llega. (V2: pago directo desde app.) |
| **Dividir la cuenta es complicado** | 🟡 Media | Identidad de comensal desde día 1 → división automática V2 (Patrón B). |
| **No sabe qué plato pedir / qué le pidieron sus amigos** | 🟢 Soft pero universal | **Mesa Viva** — vista live del grupo. Calificaciones + top platos + recomendados. |

---

## 3. Barreras de adopción de software nuevo

| Barrera | Severidad | Aplicación a PEDILO | Cómo lo mitigamos |
|---|---|---|---|
| **Costo upfront** | 🔴 Alta | Fudo/Maxirest cobran ARS 12K-20K/mes mínimo desde día 1 | **Tier free real** + suscripción mensual baja. Wedge clave. |
| **Resistencia del staff** (mozos viejos, cocineros) | 🔴 Alta | Tablet en cocina es nueva; mozos pueden resistir | Login PIN, UI ultra-minimal, **validación del mozo refuerza su rol** (no lo reemplaza) |
| **Curva de aprendizaje** | 🟡 Media | Cualquier sistema nuevo lleva tiempo | Onboarding guiado, UI obvia. Capacitación opcional como add-on |
| **Data security / desconfianza** | 🟡 Media | Casos de hackeos POS en USA generan ruido. AR confía más. | RLS, logs auditables, status page público |
| **Integración con sistemas existentes** | 🟡 Media (matizada) | Si ya tiene Fudo, ¿conviven o reemplazan? | **PEDILO convive** con POS actual. No reemplaza inventario, factura ni delivery del POS. (Camino B confirmado §4.) |
| **Dependencia de legacy** | 🟡 Media | Locales con controlador fiscal físico, planillas Excel | PEDILO no toca eso. Lo deja al POS. |
| **Conectividad inestable** (wifi malo) | 🟡 Media | Realidad AR en muchos locales | Fallback de polling. Banner "reconectando". |
| **Hardware extra** (tablets, impresoras) | 🟢 Baja | PEDILO no requiere hardware nuevo en MVP | Web responsive (PWA) en celulares y PCs que el local ya tiene |

---

## 4. Coexistencia vs reemplazo — Camino B confirmado

**Decisión estratégica clave:** PEDILO **NO compite con Fudo/Maxirest en backoffice** (inventario, facturación, delivery, controlador fiscal). PEDILO **es la capa de cliente** sobre lo que el local ya tiene (o sobre nada, si arranca desde cero).

```
┌────────────────────────────────────────┐
│  Comensal (mesa, celular)              │
│         ↓                              │
│  ┌─────────────────────┐               │
│  │     PEDILO          │ ← capa cliente│
│  │  (QR, mesa, pago)   │               │
│  └─────────────────────┘               │
│         ↓                              │
│  ┌─────────────────────────────────┐   │
│  │  POS existente (opcional)       │   │
│  │  • Fudo / Maxirest / o nada     │   │
│  │  • Inventario + factura AFIP    │   │
│  │  • Delivery PedidosYa/Rappi     │   │
│  │  • Controlador fiscal           │   │
│  └─────────────────────────────────┘   │
└────────────────────────────────────────┘
```

**Posicionamiento por segmento:**

| Segmento | Pitch específico |
|---|---|
| **🟢 Locales sin nada digital** (libreta, planilla) | *"Modernizá la experiencia de tu cliente sin cambiar tu forma de operar. Gratis para empezar."* |
| **🟢 Locales nuevos / recién abren** | *"Arrancá con todo el toolkit moderno. Sin invertir miles de pesos en sistemas viejos."* |
| **🟢 Bares / cervecerías de ambiente joven** | *"Experiencia social premium para tus clientes. Cuenta compartida real, división de cuenta, agilidad."* |
| **🟡 Locales con MOZO Digital / SoyMenu** | *"Más que un menú QR. Mesa viva, identidad social y pago integrado."* |
| **🟡 Locales con Fudo/Maxirest pero módulo QR no usado** | *"Tu POS sigue funcionando. PEDILO te suma una capa de comensal premium que tu POS no hace bien."* |
| **🔴 Locales con Fudo/Maxirest + módulo QR activo y satisfecho** | NO target. Pelea perdida. |

**TAM accesible para PEDILO en AR (estimado):**
- Total establecimientos gastronómicos AR: 80.000–120.000.
- Fudo + Maxirest combinados: ~50.000 (con superposición). Asumamos 35.000 únicos activos.
- **Locales SIN Fudo ni Maxirest: 45.000–85.000.** **Ese es el target principal de PEDILO.**
- A esto se suman algunos locales con Fudo/Maxirest cuyo módulo QR no se usa o quieren mejor UX → **agregamos 5.000–10.000 más.**
- **TAM accesible total: 50.000–95.000 locales.**

> **Si PEDILO captura 1% de su TAM accesible en 3 años, son 500–950 locales. Coincide con la proyección del modelo de negocio.**

---

## 5. Posicionamiento y argumentos de venta

### 5.1. Oración del producto

> **"PEDILO es la interfaz que conecta al comensal con el staff del local — mozo, caja, cocina y bar — sin reemplazar el POS contable que el dueño ya use (o sin pedirle uno si no tiene)."**

Variante corta para landing:

> **"Sumá un canal de mesa autoservicio. No cambia nada de lo que ya hacés."**

### 5.2. Argumentos para el dueño

1. **Cero riesgo de probar.** Tier FREE permanente. Sin contrato. Sin demo. Te creás cuenta, subís el menú, te llevás los QRs en 10 minutos.
2. **Convive con tu sistema actual.** No tenés que migrar de Fudo/Maxirest si lo tenés. PEDILO no maneja inventario ni factura — lo deja al POS.
3. **Menos pasamanos.** Tu mozo coordina, no tipea. Pedidos van directo del comensal a cocina (con validación del mozo en primer pedido para evitar pedidos fantasma).
4. **Experiencia premium para tu cliente.** Cuenta compartida real con identidad social, división de cuenta, todo en su celular sin descargar app.
5. **Pago directo desde la mesa** (V2). Sin esperar al mozo. Tu caja recibe la confirmación al instante.
6. **Reducís la rotación de staff** indirectamente: como PEDILO es muy fácil de usar, capacitar a un mozo nuevo lleva 5 min.

### 5.3. Argumentos para el mozo

1. **Menos corridas.** Te enterás del pedido al instante sin tener que ir a la mesa.
2. **Refuerza tu rol.** Vos validás los pedidos antes de que vayan a cocina (anti-abuso).
3. **Menos errores.** El comensal escribe su pedido. Si hay un error, es de él.
4. **Login en 2 segundos.** Tu nombre y tu PIN. No mails, no contraseñas largas.
5. **Vos seguís manejando la propina** (PEDILO no la toca). Y eventualmente la app sugiere propina al comensal (V2 con pago).

### 5.4. Argumentos para el comensal final

1. **Pedís sin esperar al mozo.** Especialmente útil en hora pico.
2. **Ves qué pidió tu mesa en tiempo real.** "Mesa Viva".
3. **Sin descargar app, sin registrarte.** Escaneás y listo.
4. **Pagás cuando querés** (V2). División automática si pediste con amigos.
5. **Calificás los platos.** Y ves qué pidieron los más recomendados.

### 5.5. Comparativas head-to-head para pitch

| | Fudo/Maxirest | MOZO/SoyMenu | **PEDILO** |
|---|:-:|:-:|:-:|
| Free para arrancar | ❌ | 🟡 | ✅ |
| Setup en <10 min sin comercial | ❌ | ✅ | ✅ |
| Mesa viva (live group order) | ❌ | ❌ | ✅ |
| Cuenta compartida + identidad social | 🟡 (lado mozo) | ❌ | ✅ |
| División de cuenta automática | ❌ | ❌ | ✅ V2 |
| Validación del mozo (anti-abuso) | ❌ | ❌ | ✅ |
| POS completo (inventario, factura, delivery) | ✅ | ❌ | ❌ (lo deja al POS del local) |

---

## 6. Implicancias para PEDILO — cambios de scope MVP

### 6.1. Sumar al MVP (features extras)

Tras la conversación, sumamos al MVP estos features (estimación: ~30 hs de implementación adicional):

| # | Feature | Por qué entra |
|---|---|---|
| F1 | **Calificación promedio del plato** (★★★★ + cant. votos) | Commodity — sin esto parece anticuado. Alta percepción de valor a costo bajo. |
| F3 | **Top 5 platos más pedidos esta semana** | Trivial (un query). Ayuda al descubrimiento. |
| F4 | **Filtro "platos rápidos"** (<15 min, basado en histórico real) | Diferenciador para almuerzo de oficina. |
| F5 | **Filtro "mejor calificados"** | Sale gratis con F1. |
| F7 | **Recomendado del chef** (toggle admin) | Trivial. Ayuda a empujar platos clave. |
| F11 | **Chip "para compartir / individual"** | Visual barato, gran valor para mesa. |
| C1+ | **Llamar al mozo con motivo** (agua, ayuda, cuenta, queja) | Refuerza el llamado existente. |
| **MV** | ⭐ **"Mesa Viva"** (vista live compartida del grupo: avatares, items por comensal, status en tiempo real) | **Diferenciador estrella.** Refuerza el sabor social que ningún competidor tiene. |

### 6.2. Sacar del MVP (lo cubre POS existente o es scope creep)

| Feature | Razón |
|---|---|
| Reportes avanzados (ventas del día, productos más pedidos) | Cualquier POS lo hace. PEDILO solo necesita "lo que pasó dentro de PEDILO". |
| Performance de mozos / tiempos de plato | POS lo cubre. Premium V1.x. |
| Stock numérico real / recetas / insumos | Confirmado fuera (Doc 01). |
| Facturación AFIP | Confirmado fuera. |
| Multi-sucursal real | Confirmado fuera. |

### 6.3. Mantener rol cajero en MVP (cambio respecto a propuesta inicial)

> **Decisión revisada (2026-05-01):** se mantiene el rol cajero en MVP. Razón: en la realidad operativa argentina, el cajero es **quien tiene la PC fija en el local** y es el coordinador. Sin acceso al sistema, el mozo se vuelve pasamanos forzado entre cocina y caja. PEDILO debe enviarle al cajero la info de pedidos en curso y mesas con cuenta abierta.

### 6.4. "Modo Social" como feature Premium futuro (V1.x / V2)

> *Cross-mesa social: comensales de mesas distintas pueden ver e interactuar entre sí, opt-in del local + opt-in del comensal.*

**Idea original de Dani (2026-05-01):** dejar que comensales autoricen compartir info entre mesas — ver qué pidió "Juan de la mesa 8", mandar saludos, socializar. Especialmente para bares/cervecerías de ambiente joven.

**Veredicto del análisis:**

- ✅ **Es un diferenciador único genuino.** Ningún competidor lo tiene.
- ⚠️ **Es de alto riesgo si se diseña mal:** harassment (acoso), privacidad (Ley 25.326), cultura del local (no toda mesa quiere ser interpelada), moderación (incidentes y reportes), implementación compleja (60–100 hs).
- ❌ **NO va en MVP.**
- 🔬 **Premium feature opt-in** del local, V1.x o V2, con safeguards estrictos:
  - Doble opt-in: local activa "Modo Social" + comensal acepta participar al escanear.
  - Solo "saludos públicos" visibles a la mesa entera (no chats privados al inicio).
  - Bloqueo en 1 tap, lista negra de palabras, reportes con consecuencia.
  - Kill switch global.
  - Lanzamiento gradual en 2-3 locales piloto antes de generalizar.

**Posicionamiento de Modo Social cuando llegue:** feature destacado para cervecerías, after-office, bares jóvenes. Locales que lo activan pagan más (subraya tier Premium).

### 6.5. Reservas online — confirmado fuera del MVP

Aunque Meniu y otros lo incluyen, PEDILO no entra ahí en MVP. Posible Premium V1.x si los pilotos lo piden con fuerza.

---

## 7. Lecciones de competidores

### 7.1. Lo que TOMAMOS como referencia (lo bueno de cada uno)

| De | Tomamos |
|---|---|
| **Fudo** | API pública para integraciones futuras, marketing claro, cobertura nacional. |
| **Maxirest** | Cuidado con el lock-in contractual: PEDILO debe ser cancelable en 1 click. |
| **Toteat (México)** | Onboarding rápido (48hs con capacitación) — PEDILO supera con <10 min self-service. |
| **Toast / Square (USA)** | Modelo subscripción + transactional fee es el estado del arte. PEDILO replica adaptado. |
| **Meniu** | Setup self-service <5 min, plan único simple, multi-idioma desde infra. Marketing "alternativa gratis a X" como estrategia SEO. |
| **MOZO Digital** | Foco AR, casos de éxito locales reconocibles. |

### 7.2. Lo que EVITAMOS (errores de competidores)

| De | Evitamos |
|---|---|
| **Fudo / Maxirest** | Modelo demo + comercial humano + contrato. PEDILO va self-service. |
| **Maxirest** | UI anticuada, contratos difíciles de cancelar, mantenimiento abusivo. |
| **Meniu** | WhatsApp ordering (mata UX, no escala). PEDILO va con pedidos nativos en tiempo real. |
| **Cualquiera** | Cobrar desde día 1 sin tier free. PEDILO va con FREE permanente. |

### 7.3. La "irreproducible advantage" de PEDILO

Lo que ningún competidor puede copiar fácil **sin cambiar su modelo de negocio entero**:

1. **Modelo PedidosYa-style** (signup → activo en 10 min sin comercial). Para que Fudo/Maxirest lo copien tienen que despedir su fuerza comercial.
2. **Tier free real** (no trial). Para que Fudo/Maxirest lo copien tienen que canibalizar su revenue.
3. **Foco mono-segmento** (capa cliente, no POS completo). Pueden agregar features pero no pueden "des-ser" un POS completo.
4. **Sabor argentino + UX moderna** desde cero, sin legacy de UI vieja.
5. **Patrón B Marketplace nativo** desde el diseño. Fudo/Maxirest no lo tienen.

---

## 8. Próximas acciones (insumo para Docs 04 y 07)

### Para Doc 04 (UI/UX y Wireframes en Figma)

- Diseñar **"Mesa Viva"** como pantalla central del comensal (avatares + items live + status).
- Wireframes con sabor moderno argentino (no genérico multi-país).
- UX optimizada para que el setup self-service sea ≤10 min. Métrica visible.
- Login PIN para staff con teclado numérico grande.
- Filtros del menú (rápidos, mejor calificados, recomendados del chef).

### Para Doc 07 (Modelo de Negocio y GTM)

- **Pitch principal:** "interfaz comensal-mozo-caja, no reemplaza tu POS".
- **Comparativa head-to-head** lista para landing.
- **Estrategia SEO**: páginas tipo "alternativa moderna a Fudo / Maxirest / MOZO".
- **Foco geográfico inicial**: Buenos Aires, después Córdoba/Rosario.
- **Boca a boca + visita en frío** como canales primarios. NO Google/Meta Ads en MVP.
- **Programa de referidos** activable desde día 1.

### Pendientes para validar con piloto

- Confirmar que el pitch "convive con tu POS" cierra para dueños con Fudo/Maxirest activos.
- Medir tiempo real de setup self-service (objetivo ≤10 min).
- Probar adopción del staff sin capacitación formal.
- Validar conversión free → Premium en cohortes reales.
- Decidir scope final del Modo Social tras observar comportamiento social de comensales.

---

## Fuentes consultadas

- [Software para Restaurantes Argentina 2026 — sistemasdefacturacionygestion.com.ar](https://sistemasdefacturacionygestion.com.ar/restaurantes/)
- [Fudo Software Pricing — Capterra](https://www.capterra.com/p/241757/FUDO/)
- [Fudo precios oficial — fu.do](https://fu.do/es-ar/precios/)
- [Funcionalidades Fudo — fu.do](https://fu.do/es-ar/funcionalidades/)
- [Fudo Autoatención QR — soporte.fu.do](https://soporte.fu.do/es/articles/12531993-autoatencion-qr-en-mesas-configuracion-y-uso)
- [10 mejores Software para Restaurante Argentina — comparasoftware.com.ar](https://www.comparasoftware.com.ar/software-para-restaurantes)
- [Maxirest oficial — maxirest.com.ar](https://maxirest.com.ar/)
- [Maxirest identificación de comensal — ayuda.maxirest.com](https://ayuda.maxirest.com/identificacion-de-comensal)
- [Maxirest opiniones — comparasoftware.com.ar](https://www.comparasoftware.com.ar/maxirest)
- [Maxirest crecimiento — InfoGastronómica](https://www.infogastronomica.com.ar/creo-un-software-para-restaurantes-en-mar-de-ajo-y-hoy-tiene-10-000-usuarios-como-nacio-y-crecio-maxirest/)
- [Software gastronómico que crece en Argentina — Infobae](https://www.infobae.com/economia/2021/05/15/crearon-el-software-que-gestiona-los-restaurantes-mas-top-de-argentina-y-a-pesar-de-la-pandemia-no-paran-de-crecer-van-a-facturar-300-millones-en-2021/)
- [Mozo Digital — mozo.digital](https://mozo.digital/)
- [Simplemenú apuesta cartas digitales — infonegocios.info](https://infonegocios.info/nota-principal/mozo-me-trae-el-qr-simplemenu-apuesta-por-las-cartas-digitales-de-bares-y-restaurantes-y-a-un-precio-de-no-creer)
- [SoyMenu precios — soymenu.com.ar](https://soymenu.com.ar/precios)
- [Meniu home español — meniuapp.com/es](https://meniuapp.com/es)
- [Meniu pricing — meniuapp.com/variant](https://meniuapp.com/variant)
- [Meniu QR — meniuapp.com/qr](https://meniuapp.com/qr)
- [Meniu Google Play](https://play.google.com/store/apps/details?id=com.meniuapp.mobile&hl=en_US)
- [10 Best Meniu Alternatives — Menubly](https://www.menubly.com/blog/best-meniu-alternatives/)
- [Falta de personal en gastronomía — La Capital](https://www.lacapital.com.ar/la-ciudad/gastronomicos-problemas-la-falta-personal-idoneo-n10040824.html)
- [Reverso del boom gastronómico — La Nación](https://www.lanacion.com.ar/economia/clientes-que-comparten-plato-empleados-sin-preparacion-y-rentabilidad-dispar-el-reverso-del-boom-nid14072023/)
- [Toteat México — toteat.com](https://toteat.com/es-mx/precios)
- [Restaurant SaaS Adoption Resistance — Cloud Awards](https://www.cloud-awards.com/why-restaurant-saas-adoption-still-faces-resistance-in-2025)
- [Restaurant Technology Industry Statistics — Restroworks](https://www.restroworks.com/blog/restaurant-technology-industry-statistics/)
- [Independent restaurants adopting tech — Hospitality Tech](https://hospitalitytech.com/study-75-independent-restaurants-plan-adopt-new-technology-2023)
- [BistroSoft (alternativo argentino)](https://bistrosoft.com/mx/)
- [NúcleoIT Gastronómico (alternativo argentino)](https://nucleoit.com.ar/Software/Gastronomico)
- [Tango Restô Axoft (alternativo gama media-alta)](https://www.axoft.com/tango/software-para-gastronomia-restaurant/)

---

> **Próxima revisión recomendada:** después de los primeros 3 pilotos reales, para confirmar/ajustar pain points, posicionamiento y argumentos de venta con data del campo.
