# PEDILO

> Plataforma SaaS multi-tenant de menú QR + pedidos en mesa para restaurantes y bares de Argentina.

**Posicionamiento:** PEDILO es la interfaz que conecta al comensal con el staff del local — mozo, caja, cocina y bar — sin reemplazar el POS contable que el dueño ya use (o sin pedirle uno si no tiene).

---

## 📚 Documentación

Todo el proyecto está en fase de planificación. **Sin código todavía.**

La documentación viva está en [`docs/`](docs/). Empezá por el índice:

→ **[docs/00-indice.md](docs/00-indice.md)**

### Documentos principales

- [01 — Visión y Alcance del MVP](docs/01-vision-y-alcance-mvp.md) ✅ Aprobado v1.3
- [02 — Arquitectura y Stack](docs/02-arquitectura-y-stack.md) ✅ Aprobado v0.4
- 03 — Modelo de Datos (pendiente)
- 04 — UI/UX y Wireframes (pendiente)
- 05 — Plan de Desarrollo por Fases (pendiente)
- 06 — Operación y Soporte (pendiente, post-piloto)
- 07 — Modelo de Negocio y GTM (pendiente)

### Documentos auxiliares

- [Glosario de siglas](docs/glosario-siglas.md)
- [Notas de modelo de negocio](docs/notas-modelo-de-negocio.md)
- [Notas de investigación de mercado](docs/notas-investigacion-mercado.md)

---

## 🛠️ Stack previsto

- **Frontend + Backend:** Next.js 15 (App Router) + React + TypeScript
- **Base de datos:** PostgreSQL en Supabase (con RLS para multi-tenancy)
- **Auth:** Supabase Auth (owner) + capa custom username+PIN (staff)
- **Tiempo real:** Supabase Realtime
- **Hosting:** Vercel
- **Pagos (V2):** MercadoPago

Ver [docs/02-arquitectura-y-stack.md](docs/02-arquitectura-y-stack.md) para el detalle.

---

## 📅 Estado del proyecto

**Fase:** Planificación + diseño.
**Próximo hito:** redactar Doc 03 (Modelo de Datos).
**Timeline MVP en piloto:** 4–5 meses desde inicio del desarrollo.
