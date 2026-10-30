# clovergy_coordination_hub

Módulo de seguimiento y coordinación entre Transición Energética (TE) e IT de la plataforma Clovergy:
entradas (formulario, OneNote, feedback), tickets de CLOVEX en solo lectura, reuniones, acuerdos,
decisiones y un panel de análisis dinámico por rol.

## Ficha

| | |
|---|---|
| Id de registro | `clovergy-coordination-hub` |
| Capa | `APPS` |
| Estado | Especificación: SPEC-002 en «propuesta», pendiente de la firma de G1 del PM |
| Owner | `joaquimtrebol` |
| Spec | [`docs/specs/SPEC-002-modulo-seguimiento-coordinacion.md`](docs/specs/SPEC-002-modulo-seguimiento-coordinacion.md) |
| PRD | `clovergy_pm/docs/prd/gestor-incidencias-monitorizacion.md` (H9–H12) |

## Ámbito

- **Es:** la superficie de trabajo del seguimiento TE ↔ IT (fase 1, autónoma y «en sombra» sobre el Excel
  semanal) y, en fase 2, un módulo integrable en CLOVEX.
- **No es:** el plan de trabajo de IT (CLOVEX), el registro de incidencias operativas del cliente (portal
  Clovergy) ni el backlog del portfolio (`clovergy_pm`). No escribe nunca en CLOVEX.

## Estado

Todavía no hay código. Nada se construye antes de la firma de G1 de la SPEC-002.

## Datos

Los datos de cliente no se versionan: ni bases de datos, ni capturas, ni exportaciones. En desarrollo,
solo datos sintéticos.
