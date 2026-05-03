# PEDILO — Índice de Documentación

> **Plan total:** 7 documentos formales + artefactos auxiliares.
> **Última actualización:** 2026-05-01

---

## 📚 Documentos formales

| # | Documento | Archivo | Estado | Cuándo se redacta |
|---|---|---|---|---|
| **01** | **Visión y Alcance del MVP** | [01-vision-y-alcance-mvp.md](01-vision-y-alcance-mvp.md) | ✅ Aprobado v1.1 | Hecho |
| **02** | **Arquitectura y Stack** | [02-arquitectura-y-stack.md](02-arquitectura-y-stack.md) | ✅ Aprobado v0.4 | Hecho |
| **03** | **Modelo de Datos** (entidades, relaciones, schemas SQL, RLS policies) | `03-modelo-de-datos.md` | ⏳ Pendiente | Después de aprobar Doc 02 |
| **04** | **UI/UX y Wireframes en Figma** | `04-ui-ux-wireframes.md` | ⏳ Pendiente | Después de Doc 03 |
| **05** | **Plan de Desarrollo por Fases** (hitos, sprints, criterios de hecho) | `05-plan-de-desarrollo.md` | ⏳ Pendiente | Después de Doc 04 |
| **06** | **Operación y Soporte Productivo** (SLAs, runbooks, on-call, status page, comunicación de incidentes) | `06-operacion-y-soporte.md` | ⏳ Pendiente | **Cerca del piloto** (no antes) |
| **07** | **Modelo de Negocio y Go-to-Market** (tiers, pricing, proyección revenue, canales de venta) | `07-modelo-de-negocio.md` | ⏳ Pendiente | En paralelo con Doc 04/05 (no bloquea desarrollo) |

---

## 🛠️ Artefactos auxiliares

| Archivo | Propósito | Estado |
|---|---|---|
| [glosario-siglas.md](glosario-siglas.md) | Glosario de siglas usadas en toda la documentación, agrupadas por temática | ✅ Vivo |
| [notas-modelo-de-negocio.md](notas-modelo-de-negocio.md) | Notas preliminares de pricing, mercado, proyección y GTM. **Insumo del futuro Doc 07** | ✅ Borrador |
| [notas-investigacion-mercado.md](notas-investigacion-mercado.md) | Análisis competitivo (Fudo, Maxirest, MOZO, Meniu, Toteat, Toast), pain points, barreras de adopción, posicionamiento "interfaz comensal-mozo-caja", argumentos de venta, scope MVP refinado, Modo Social como feature Premium futuro. **Insumo de Docs 04 (UI/UX) y 07 (GTM)** | ✅ v1.0 |
| `../CLAUDE.md` (raíz del repo) | Bootstrap para que cualquier sesión nueva de IA arranque con contexto. Apunta a este índice. | ⏳ **Opcional.** Solo se crea si: (1) el proyecto se inicializa en Git, (2) Dani trabaja desde otra máquina, o (3) suma colaboradores. Mientras trabaje solo en esta PC, la memoria + el índice alcanzan. |
| `runbooks/` | Carpeta con runbooks vivos (problemas conocidos, procedimientos de migración, etc.) | ⏳ Se va llenando con la operación real |
| `postmortems/` | Postmortems de incidentes pasados con cultura sin culpa | ⏳ Cuando haya operación real |
| `../README.md` (raíz del repo) | Descripción técnica del repositorio (cómo levantar el proyecto local, scripts, etc.) | ⏳ Cuando arranque el código |

---

## 🗺️ Orden y dependencias

```
Doc 01 ✅ Visión y Alcance
    ↓
Doc 02 📝 Arquitectura y Stack
    ↓
Doc 03 ⏳ Modelo de Datos (depende de Doc 02)
    ↓
Doc 04 ⏳ UI/UX en Figma (depende de Doc 03 — pantallas reflejan los datos)
    ↓
Doc 05 ⏳ Plan de Fases (depende de todo lo anterior)
    ↓
        🚀 ARRANCA DESARROLLO DEL MVP
    ↓
CLAUDE.md (bootstrap del repo, foto final antes de codear)

   ─── EN PARALELO al desarrollo ───
Doc 07 ⏳ Modelo de Negocio: borrador antes del piloto
Doc 06 ⏳ Operación: redactar semanas antes del piloto
```

---

## 📐 Convenciones de los documentos

- **Numeración con dos dígitos** (`00-`, `01-`, etc.) — `00` reservado para este índice.
- **Header estándar:** Estado, Versión, Fecha, Próximo paso, Historial de cambios.
- **Estados posibles:** `Borrador para revisión` / `En revisión` / `Aprobado`.
- **Versiones:** semver simplificado (`v0.1`, `v0.2`, `v1.0` al aprobar).
- **Comentarios de Dani durante revisión** con la convención `> 💬 **Dani:** ...`. Yo los respondo y, una vez resueltos, marco con `> ✅ **Resuelto:** ...`.
- **Idioma:** español rioplatense.
- **Code snippets, schemas y nombres técnicos:** en inglés (`tenant_id`, `payment_mode`, etc.).

---

## 🚦 Para arrancar una sesión nueva de IA con contexto

Mientras `CLAUDE.md` no exista todavía, podés arrancar una sesión nueva diciéndole al asistente:

> "Estoy trabajando en PEDILO. Leé `docs/00-indice.md` y luego `docs/01-vision-y-alcance-mvp.md` antes de avanzar. Después decime en qué punto del proyecto estamos."

Cuando esté `CLAUDE.md` en la raíz del repo, esto se hace solo.
