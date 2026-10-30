# Guía de agente — clovergy_coordination_hub

Repo del módulo de seguimiento y coordinación TE ↔ IT de Clovergy. Lee primero `README.md` y la
SPEC-002 (`docs/specs/`).

## Reglas

- **Spec primero.** Nada se construye sin la firma de G1 del PM en la SPEC-002; un cambio de contrato
  entra antes como delta en su registro de cambios.
- **CLOVEX, solo lectura.** Ningún conector crea, cambia ni borra nada en CLOVEX.
- **Datos de cliente, nunca en git.** Ni base de datos, ni capturas, ni exportaciones; en desarrollo,
  datos sintéticos.
- **Entrega:** rama `feat|fix|improve/CLV-XXX-descripcion`, PR que nombra su issue y validación del owner
  (`joaquimtrebol`), que es quien fusiona. Nada de push ni PR sin su permiso explícito.
- **Idioma:** UI y documentación en español; código, identificadores y ficheros en inglés. Se escribe
  «ticket». No se nombran cursos ni formaciones externas.
- **Repo ≠ id:** repo `clovergy_coordination_hub`, id de registro `clovergy-coordination-hub`. <!-- docs-check: ignore -->
