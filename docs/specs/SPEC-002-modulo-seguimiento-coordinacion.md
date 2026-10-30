---
id: SPEC-002
titulo: "Módulo de seguimiento y coordinación TE ↔ IT"
estado: propuesta   # borrador | propuesta | aprobada | implementada | obsoleta
fecha: 2026-10-30
prd: clovergy_pm/docs/prd/gestor-incidencias-monitorizacion.md (H9–H12)
issue: por abrir en clovergy_pm al firmar G1 (etiqueta project/clovergy-pm)
responsable_it: por nombrar (pregunta abierta P-03)
puerta: G1
---

# SPEC-002 — Módulo de seguimiento y coordinación TE ↔ IT

Grupo Trébol Energía · Transición Energética ↔ IT · Plataforma Clovergy
Versión 0.3 · 30 de octubre de 2026 · Punto 4 del encargo `00_PROMPT_seguimiento_coordinacion_IT_v1`

> **Qué es.** El contrato de lo que se construye y de cómo se comprueba: una aplicación ligera que reúne
> el seguimiento de incidencias (INC), feedback (FB), tickets de IT, reuniones, acuerdos y decisiones,
> con un panel de análisis dinámico. Responde «qué se construye y cómo se verifica»; el porqué está en el
> análisis `01_ANALISIS_entradas_roles_reunion_v1` y en el PRD.
>
> **Ciclo.** Explorar → proponer (este estado, `propuesta`) → **revisión del contrato por el PM (firma de
> G1, `aprobada`)** → aplicar → verificar → sincronizar esta spec con lo construido (`implementada`) →
> archivar. Nada se construye sin la firma de G1.
>
> **Dónde vivirá.** Cuando se firme, esta spec se copia a `docs/specs/` del repo que elija el PM (P-01) y
> sigue viva allí, revisada por PR. Esta copia de la carpeta del proyecto es la de revisión.

---

## 1. Resumen

Una aplicación web ligera, con la guía de diseño de la webapp de `clovergy_pm` (§3.1), que en su **fase 1 vive sola**: lee
las entradas (formulario, OneNote, CLOVEX, Excel de feedback) por conectores, guarda lo que es de TE
(triaje, seguimiento, notas, acuerdos, decisiones) y enseña un **panel de análisis dinámico** con filtros
cruzados y detalle, más un **modo reunión** donde varias personas anotan a la vez. En la **fase 2** se
integra en CLOVEX como un módulo más, por el medio que acuerde IT.

La fase 1 empieza **en sombra**: lee el Excel semanal tal como está y no lo sustituye hasta que se valide
con los cuatro roles (G2).

### No-alcance

- **No escribe en CLOVEX.** Ni tickets, ni estados, ni comentarios. El alta del ticket sigue siendo el
  correo al buzón de Desarrollo (DR-13 y DR-14). Si la fase 2 lo necesitara, se especifica como delta y lo
  decide IT.
- **No es el plan de trabajo de IT**: ni sprints, ni tareas, ni horas, que viven en CLOVEX (ADR-004).
- **No es el registro de incidencias operativas del cliente** (falta de datos, umbrales), que es la página
  de incidencias del portal Clovergy (PRD, D4). Este módulo sigue lo que va de TE a IT: bugs y mejoras de
  la plataforma y de sus módulos integrados, más el feedback que los origina.
- **No reconstruye la webapp de `clovergy_pm`**: la página `/clovex` sigue siendo la vista del portfolio en
  solo lectura; el módulo puede enlazarla, no la duplica.
- **No usa Power BI como pieza obligatoria.** Publica sus datos para quien quiera conectarlo, pero el panel
  es propio (§4.3); usar Power BI exigiría licencias que no se dan por supuestas.

## 2. Comportamiento

**Entradas**

1. Cada entrada (INC o FB) tiene un único ID (`INC-NN`, `FB-NNN`) que se conserva desde el Excel; el
   módulo no renumera.
2. Una respuesta del formulario crea una INC si su Id de Forms no se ha importado antes; si ya existe, no
   hace nada (idempotente).
3. Un punto del OneNote sin respuesta del formulario no crea una INC: aparece como «Pendiente de dar de
   alta» (DC-01).
4. Un ticket de CLOVEX de un producto del catálogo en alcance que no casa con ninguna entrada aparece en la
   bandeja **«Entradas por otra vía»** hasta que alguien elige *Alta como INC*, *Vincular a…* o *Sin INC*,
   con motivo.
5. Un ticket casa con una entrada por su número guardado o por el prefijo `[INC-NN]`/`[FB-NNN]` del asunto;
   cualquier otra coincidencia (palabras, producto) es solo una sugerencia «por confirmar».

**Relación con los tickets**

6. Una entrada puede tener varios tickets y un ticket varias entradas. Cada relación lleva tipo:
   *principal*, *ampliación*, *parecido (por confirmar)* o *duplicado*.
7. El estado en IT de una entrada es el de su ticket *principal*. Sin principal, no tiene estado en IT.
8. Un FB nunca tiene ticket propio: va en el ticket de su INC (decisión del PM del 29/10/2026).

**Estados y lectura**

9. La columna derivada **«Le toca a»** (PM/monitorización, IT, Bloqueado, Técnico TE, Nadie) se calcula
   siempre de los demás campos y nunca se teclea.
10. Lo que viene de CLOVEX manda sobre lo dicho en una reunión. Si no coincide, la entrada lleva el aviso
    «La reunión dijo X, CLOVEX dice Y».
11. Cada dato leído de una fuente guarda la fecha y hora de su lectura. Si la última lectura de CLOVEX
    tiene más de 7 días, todo lo que viene de CLOVEX se enseña marcado como desfasado. Si una fuente falla,
    se avisa; nunca se inventa ni se rellena un valor.
12. «Hecho» en CLOVEX pone la entrada en «Le toca a: Técnico TE» con 5 días laborables para validar. Un
    ticket parecido a una entrada validada en los 30 días siguientes la devuelve a «por validar».

**Reunión**

13. Una reunión tiene fecha, asistentes y una agenda con las secciones fijas del análisis (M16), generada
    a partir del estado del momento.
14. Durante la reunión, cualquier asistente con permiso anota sobre una fila de la agenda. Cada nota tiene
    tipo: *comentario*, *revisión*, *acuerdo*, *pregunta a IT* o *decisión*.
15. Un acuerdo tiene ID `ACU-AAAAMMDD-NN`, dueño y fecha; sale en todas las agendas siguientes hasta
    cerrarse. Uno vencido se marca en ámbar.
16. Una nota de tipo *decisión* crea o actualiza una DR-NN con su fecha y quién decidió.
17. Se puede anotar sobre algo sin entrada: un ticket sin INC o una decisión abierta tienen su propia fila
    en la agenda.
18. Al cerrar la reunión, el módulo genera el acta (Word, formato GTE) con la estructura M10 y su anexo
    Excel, sin enviarlos.

**Permisos (resumen; detalle en §12)**

19. «Enviar a IT (PM)» solo lo cambia el PM. Ningún proceso automático lo rellena.
20. El técnico de IT solo puede escribir las columnas de IT (prioridad IT, responsable IT, fecha objetivo)
    y notas; no cambia triaje ni prioridad de TE.

**Casos límite**

21. Dos personas cambian el mismo campo durante la reunión: gana la última y la anterior queda en el
    historial de la entrada, visible.
22. El Excel semanal cambia mientras el módulo está en sombra: la siguiente importación trae lo cambiado
    por cabecera y anota qué celdas difieren; nunca escribe en el Excel.
23. Un ticket desaparece de la lectura de CLOVEX (permiso, borrado): se conserva el último dato con la
    marca «no visto desde fecha».

## 3. Reglas de diseño

| Regla | Aplica | Cómo se cumple o por qué no |
|---|---|---|
| Una sola capa (`INGESTION → DATA → API → ANALYTICS → APPS`) | Sí | Capa `APPS` (y su API propia). No lee telemetría ni datos de lecturas de Clovergy |
| No duplicar lógica que ya existe en otra capa o repo | Sí | Las reglas de deducción de `completar` y la fórmula de «Paso a CLOVEX» se **portan** con las mismas pruebas, y el script queda como referencia hasta G2; el tablero de `/clovex` no se rehace |
| Nada fijado por cliente | Sí | Cuentas, productos de CLOVEX, rutas y equipo son tablas de configuración |
| UI con los tokens de diseño | Sí | Sigue la guía de diseño de la webapp de `clovergy_pm` (§3.1), decidido por el PM el 30/10/2026. Capa de tema con los tokens `clovergy-*`: si en la fase 2 hay que encajar en CLOVEX, se cambia el tema, no los componentes |
| Idioma | Sí | UI y documentos en español; código, identificadores y ficheros en inglés |
| Repo ≠ id | Sí | Si se crea repo: guion bajo en el repo, guion en el id de registro |
| Dependencias mínimas | Sí | Las de la webapp de `clovergy_pm` (Next.js, Tailwind, `lucide-react`) más una librería de gráficos y una base de datos ligera; cada una con versión y motivo en el README |
| Lo determinista va a prueba o CI | Sí | Reglas 1–23 con prueba (§8.3) |
| Solo lectura en CLOVEX | Sí | El conector usa el proxy `clovex-it-lectura` o, en fase 2, una cuenta de servicio de solo lectura |

### 3.1 Guía de diseño (la de la webapp de `clovergy_pm`)

Fuente: `clovergy_design/kit/GUIA-UXUI.md` y `tokens.json` (v2.0.0), tal como los aplica la webapp de
`clovergy_pm` (`webapp/tailwind.config.js`, `webapp/styles/globals.css`, sección «Diseño» de
`webapp/README.md`, rediseño de CLV-902). Se copian los tokens, no se inventan colores.

| Elemento | Regla |
|---|---|
| Marca | Barra superior teal `#00738c` de 70 px con el logo blanco oficial; salvia `#97a675` como acento |
| Superficies y texto | Fondo `#f8f8fb`, superficie `#ffffff`, texto `#495057`, texto secundario `#74788d`, borde `#dde4ec` |
| Estados | ok `#34c38f`, info `#50a5f1`, aviso `#f1b44c`, peligro `#f46a6a`, en insignias y avisos. «Le toca a» usa estos y los de marca, no colores propios |
| Tipografía | Poppins a 14 px para la interfaz; Roboto Mono con cifras tabulares para ids (INC-NN, tickets) y métricas; cargadas con `<link>`, nunca `@import` |
| Iconos | `lucide-react` a 16 px; sin emojis ni pictogramas |
| Navegación | Un solo menú lateral de 240 px por áreas (Panel, Bandeja, Lista, Reunión, Decisiones, Fuentes); por debajo de `lg` es un cajón que se cierra al navegar |
| Tablas | Dentro de tarjetas con scroll propio (`overflow-x-auto`); la página no desborda a 390 px |
| Accesibilidad | Foco visible teal, `aria-current` en la sección activa, `aria-label` en botones de solo icono, objetivos de 44 px en móvil, WCAG 2.1 AA |
| Comprobación | Una prueba como `scripts/webapp-interfaz.test.js` de `clovergy_pm` que verifica estas reglas en cada ejecución, y una revisión a 390 px |

## 4. Interfaz y contrato

### 4.1 Pantallas

| Pantalla | Para | Qué hace |
|---|---|---|
| **Panel** | Todos (corte por rol) | Cifras con la diferencia frente a la semana anterior, embudo por paso, «Le toca a», antigüedad y SLA, por ruta, módulo, cuenta y técnico, CLOVEX por etapa y QA. Filtros cruzados: un clic en un gráfico filtra los demás; doble clic baja a la lista |
| **Bandeja** | Responsable de monitorización | Entradas nuevas por triar y «Entradas por otra vía» por resolver |
| **Lista** | Todos | Tabla con las columnas de hoy, filtros guardados por rol (las cuatro vistas M9), exportación a Excel |
| **Ficha** | Todos | Una entrada: lo que anotó el técnico, triaje, tickets relacionados, historial, notas, capturas |
| **Reunión** | Asistentes | Agenda en secciones, anotación en vivo con tipo de nota, acuerdos abiertos, cierre que genera el acta |
| **Decisiones** | PM, coordinador | DR-NN y D-NN con dueño, fecha y lo que bloquean |
| **Fuentes** | PM | Última lectura de cada conector, errores, y botón de lectura manual |

Se prepara un mock navegable con datos sintéticos antes de G2; el enlace se añade aquí.

### 4.2 Modelo de datos (lógico)

| Entidad | Campos principales | Fuente maestra |
|---|---|---|
| `entry` | id (`INC-NN`/`FB-NNN`), tipo, origen (formulario, OneNote, feedback, otra vía), fecha, cuenta, técnico, módulo, ruta, situación, propuesta, impacto, alcance, capa, dónde, triaje, ID destino, prioridad PM, «Enviar a IT (PM)», técnico TE de seguimiento, estado TE, validación | Módulo (en sombra: el Excel) |
| `ticket` | número CLOVEX, etapa, estado QA, prioridad, producto, buzón, asunto, creado, actualizado, `read_at` | **CLOVEX** (solo lectura) |
| `entry_ticket` | entrada, ticket, tipo de relación, quién y cuándo la confirmó | Módulo |
| `meeting` | fecha, asistentes, estado (abierta, cerrada), acta generada | Módulo |
| `note` | reunión (opcional), entrada o ticket o decisión, tipo, texto, autor, fecha | Módulo |
| `agreement` | `ACU-AAAAMMDD-NN`, qué, dueño, fecha límite, estado, reunión de origen | Módulo |
| `decision` | DR-NN o D-NN, texto, de quién depende, qué bloquea, estado, acordado | Módulo |
| `source_read` | fuente, fecha y hora, resultado, nº de elementos, error | Módulo |
| `snapshot_weekly` | fecha, cifra, valor | Módulo (histórico M11) |

### 4.3 Contratos de entrada y salida

| Contrato | Sentido | Forma |
|---|---|---|
| **Excel semanal** (`Seguimiento_Revisiones_Semanales_TE.xlsx`) | Entra, en sombra | Lectura por cabecera de «Revisiones», «Reuniones», «Decisiones», «CLOVEX IT», «FB en IT»; nunca por posición |
| **Excel de feedback rev04** | Entra | Lectura por cabecera de «Feedback»; nunca se escribe |
| **Respuestas del formulario** | Entra | Excel de respuestas de Forms; en fase 2, si TI lo permite, Microsoft Graph |
| **OneNote** | Entra | El XML de página que ya deja `exportar_onenote.ps1` en `1_origen/` |
| **CLOVEX** | Entra | `clovex.json` + `clovex.local.json` del `clovex-snapshot.js` (fase 1); API con cuenta de servicio (fase 2, D5 del PRD) |
| **Agenda de la reunión** | Sale y entra | La misma tabla que `Agenda_reunion_IT_AAAA-MM-DD.xlsx` (análisis, M14), para que el script y el módulo convivan |
| **API REST** `/api/v1/` | Sale | Lectura de entradas, tickets, relaciones, reuniones, acuerdos, decisiones y cifras; escritura solo de lo que es del módulo (notas, triaje, relaciones, acuerdos), con el rol |
| **Servidor MCP** del módulo | Sale | Herramientas de lectura (`list_entries`, `get_entry`, `list_agreements`, `dashboard_figures`) y, con permiso, `add_note`; nunca toca CLOVEX |
| **Datos para análisis externo** | Sale | Exportación de las tablas en CSV o un endpoint de solo lectura que Power BI Desktop pueda leer, si se decide usarlo |
| **Acta y anexo** | Sale | Word con la plantilla corporativa y Excel; se generan, no se envían |

## 5. Integración con datos Clovergy

| Fuente | Sentido | Contrato | Autenticación | ¿Datos de cliente? |
|---|---|---|---|---|
| Tareas y tickets de IT (CLOVEX) | **lee** | MCP `clovex-it` vía el proxy `clovex-it-lectura` (fase 1); API de CLOVEX (fase 2) | Login de dispositivo personal (fase 1, ver riesgo); cuenta de servicio (fase 2) | Sí: asuntos con cuenta |
| Registro del portfolio y backlog (`clovergy_pm`) | lee | `registry/*.yaml` | — | No |
| Excel semanal y de feedback | lee | Cabeceras de §4.3 | Fichero local o SharePoint | Sí |
| API de lecturas, dispositivos, telemetría | **no aplica** | — | — | — |

- **Entidades y campos:** los de §4.2. No se usa ningún `variable_id`.
- **Dónde se guarda:** en fase 1, en un servidor interno de GTE o en el PC del PM durante la prueba; la base
  de datos y las capturas **nunca en git**. En desarrollo, solo datos sintéticos.
- **Calidad del dato:** fechas en hora de Madrid; duplicados resueltos por ID; huecos visibles como vacíos.
- **Si la fuente falla:** aviso en «Fuentes» y en la cabecera del Panel, y el último dato marcado como
  desfasado (regla 11).
- **¿Toca el esquema de datos de Clovergy?** No. Tiene esquema propio, que es el trabajo de más varianza
  de esta spec: se valida con el modelo de §4.2 antes de estimar.

## 6. Criterios de aceptación

- [ ] **AC-1** DADO el Excel semanal del 29/10, CUANDO se importa en sombra, ENTONCES el módulo tiene el
  mismo número de entradas, y las cifras del Panel coinciden con las del Panel del Excel una a una.
- [ ] **AC-2** DADO una respuesta del formulario ya importada, CUANDO se vuelve a importar, ENTONCES no se
  crea ninguna entrada nueva.
- [ ] **AC-3** DADO un ticket de un producto en alcance sin entrada, CUANDO se lee CLOVEX, ENTONCES aparece
  en la bandeja «Entradas por otra vía» y no sale de ella hasta tener una resolución con motivo.
- [ ] **AC-4** DADO INC-78 con 22451 como principal y 22351 como ampliado, CUANDO cambia la etapa de 22451
  en CLOVEX, ENTONCES el estado en IT de INC-78 cambia y el de 22351 no lo afecta.
- [ ] **AC-5** DADO cualquier entrada, CUANDO se abre, ENTONCES tiene exactamente un valor de «Le toca a»,
  derivado, y no se puede editar.
- [ ] **AC-6** DADO una lectura de CLOVEX de hace más de 7 días, CUANDO se abre el Panel, ENTONCES las cifras
  de CLOVEX salen marcadas como desfasadas con la fecha de la lectura.
- [ ] **AC-7** DADO una reunión abierta con dos asistentes, CUANDO cada uno añade una nota sobre filas
  distintas a la vez, ENTONCES las dos se guardan con su autor y se ven en la ficha de su entrada.
- [ ] **AC-8** DADO un acuerdo con fecha vencida, CUANDO se genera la agenda siguiente, ENTONCES aparece en
  «Acuerdos abiertos» en ámbar.
- [ ] **AC-9** DADO una nota de tipo *decisión*, CUANDO se guarda, ENTONCES existe o se actualiza su DR-NN.
- [ ] **AC-10** DADO un usuario con rol técnico de IT, CUANDO intenta cambiar «Enviar a IT (PM)» o el
  triaje, ENTONCES el módulo no lo permite.
- [ ] **AC-11** DADO cada uno de los cuatro roles, CUANDO abre el Panel con su vista, ENTONCES ve lo que le
  toca sin aplicar filtros a mano.
- [ ] **AC-12** DADO un clic en una barra del Panel (por ejemplo, la ruta «Ingesta y comunicaciones»),
  CUANDO se pulsa, ENTONCES los demás gráficos y cifras se filtran por esa ruta, y con doble clic se abre la
  lista de esas entradas.
- [ ] **AC-13** DADO una reunión cerrada, CUANDO se genera el acta, ENTONCES el Word tiene el formato GTE,
  la estructura M10 (resumen, acuerdos, fichas por ticket, anexo) y no se ha enviado nada.
- [ ] **AC-14** DADO el registro de actividad del conector de CLOVEX, CUANDO se revisa, ENTONCES no hay
  ninguna llamada de escritura.

## 7. Historias y tareas

| Historia | Criterios | Talla | Riesgo | Depende de | Definición de hecho |
|---|---|---|---|---|---|
| H-1 · Importar en sombra el Excel semanal y el de feedback | AC-1 | M | Medio (cabeceras cambiantes) | — | Prueba con copia del 29/10 en verde |
| H-2 · Modelo de datos y API de lectura | AC-1, AC-5 | M | **Alto** (esquema nuevo) | H-1 | Esquema revisado, pruebas en verde |
| H-3 · Conector de CLOVEX (lectura del snapshot) y relaciones N:M | AC-3, AC-4, AC-6, AC-14 | M | Medio | H-2 | Prueba con el snapshot del 29/10 |
| H-4 · Panel dinámico con filtros cruzados y vistas por rol | AC-6, AC-11, AC-12 | L | Medio | H-2 | Validado por un usuario de cada rol |
| H-5 · Bandeja de triaje y de «Entradas por otra vía» | AC-3 | S | Bajo | H-3 | Prueba y demo |
| H-6 · Modo reunión: agenda, notas tipadas, acuerdos y decisiones | AC-7, AC-8, AC-9 | L | Medio (concurrencia) | H-2 | Probado en una reunión real |
| H-7 · Acta y anexo con formato GTE | AC-13 | S | Bajo | H-6 | Acta revisada por el PM |
| H-8 · Roles y permisos | AC-10 | M | Medio (autenticación, P-04) | H-2 | Prueba por rol |
| H-9 · Importar formulario y OneNote | AC-2 | S | Bajo | H-2 | Prueba de idempotencia |
| H-10 · Tema y prueba de la guía de diseño (§3.1) | — | S | Bajo | H-4 | Prueba de interfaz en verde y revisión a 390 px |

Prioridad por valor y dependencia: H-1 → H-2 → H-3 → H-4 → H-6 → H-7 (lo que se enseña en G2); luego
H-5, H-8, H-9, H-10. Las tallas son una propuesta; las corrige quien construya.

## 8. Calidad y pruebas

### 8.1 Criterios de calidad

| Criterio | Umbral | Cómo se mide |
|---|---|---|
| Criterios de aceptación cubiertos por una prueba | 100 % o justificado | Matriz 8.3 |
| Casos límite de §2 (reglas 21–23) con prueba | Todos | Matriz 8.3 |
| Paridad con el Excel en sombra | 0 diferencias en las cifras del Panel | Prueba AC-1 |
| Accesibilidad | WCAG 2.1 AA en contraste y teclado | Revisión automática y manual |
| Rendimiento | Panel con 1.000 entradas y 500 tickets en < 2 s | Prueba con datos sintéticos |

### 8.2 Plan de pruebas

| Qué se prueba | Tipo | Datos | Quién valida |
|---|---|---|---|
| Reglas 1–23 | Unitaria | Sintéticos | Quien construye |
| Importación en sombra | Integración | Copia del libro del 29/10, en entorno interno | PM |
| Conector de CLOVEX | Integración | Snapshot del 29/10 | PM |
| Panel por rol | Manual | Copia anonimizada | Un usuario de cada rol (G2) |
| Modo reunión | Manual, en una reunión real | Copia | PM y técnico de IT |

### 8.3 Matriz de trazabilidad

| Historia | Criterio | Prueba | Código | Estado |
|---|---|---|---|---|
| H-1 | AC-1 | por escribir | — | No cubierto |
| H-2 | AC-5 | por escribir | — | No cubierto |
| H-3 | AC-3, AC-4, AC-6, AC-14 | por escribir | — | No cubierto |
| H-4 | AC-11, AC-12 | por escribir | — | No cubierto |
| H-6 | AC-7, AC-8, AC-9 | por escribir | — | No cubierto |
| H-7 | AC-13 | por escribir | — | No cubierto |
| H-8 | AC-10 | por escribir | — | No cubierto |
| H-9 | AC-2 | por escribir | — | No cubierto |

## 9. Auditorías de coherencia y validación

| Auditoría | Qué compara | Cómo | Cuándo |
|---|---|---|---|
| Spec ↔ código | Criterios frente a pruebas | Matriz 8.3 y revisión adversarial de solo lectura (la hace otra persona u otro agente) | Antes de G2 |
| Módulo ↔ Excel en sombra | Cifras y entradas | Prueba AC-1 en cada importación | Mientras dure la sombra |
| Módulo ↔ CLOVEX | Tickets vistos frente a la lectura | Conteo por etapa en «Fuentes» | En cada lectura |
| Docs ↔ implementación | Esta spec, el README y el flujo frente a lo construido | Revisión en cada PR | Cada PR que cambie comportamiento |
| Validación con usuario | El Panel y la reunión frente a las preguntas de cada rol | Reunión de IT con los cuatro roles, con acta | G2 |

## 10. Documentación

| Documento | Dónde | Estado |
|---|---|---|
| README con ficha, ámbito, arranque y dependencias | Raíz de `clovergy_coordination_hub` | Pendiente |
| `AGENTS.md` y `CLAUDE.md` que lo importa | Raíz del repo | Pendiente |
| Esta spec, viva | `docs/specs/` del repo | Propuesta (esta copia) |
| ADR de dónde vive el módulo y quién es el registro maestro | `docs/decisions/` | Pendiente de P-01 y DC-05 |
| Manual de uso de una página por rol | `docs/` | Pendiente |
| Registro de prompts del trabajo con agentes | `docs/prompts.md` | Pendiente |
| Actualización del flujo de revisiones semanales | Carpeta del proyecto | Al pasar a maestro |

## 11. Registro de cambios

| Fecha | Cambio (delta) | PR o issue |
|---|---|---|
| 30/10/2026 | v0.1 · propuesta inicial para G1 | — |
| 30/10/2026 | v0.2 · P-01 decidida por el PM: repo nuevo `clovergy_coordination_hub` | — |
| 30/10/2026 | v0.3 · P-02 decidida por el PM: guía de diseño de la webapp de `clovergy_pm` (nueva §3.1) | PR #1 |

## 12. Seguridad *(obligatoria antes de G3)*

- [ ] Sin secretos en el repo ni en el historial; la credencial de CLOVEX fuera del repo
- [ ] Autenticación con las cuentas de la empresa (P-04) y autorización por rol:

| Rol | Lee | Escribe |
|---|---|---|
| PM | Todo | Todo lo del módulo, incluido «Enviar a IT (PM)» |
| Coordinador de departamento | Todo | Decisiones y notas |
| Responsable de monitorización | Todo | Filtro, triaje, seguimiento, notas, acuerdos |
| Técnico de IT | Entradas trasladadas o en agenda, sus tickets | Columnas de IT, notas |
| Técnico de monitorización | Todo lo de monitorización | Sus altas, validación, notas |

- [ ] Sin datos de cliente en logs ni fuera de la infraestructura de GTE; los documentos internos sí pueden
  llevarlos
- [ ] Dependencias revisadas
- [ ] La lectura de CLOVEX con sesión personal (fase 1) se acepta solo mientras se aprueba la enmienda D-24
  del ADR-004; para el uso programado hace falta cuenta de servicio

## 13. Despliegue y operación *(obligatoria antes de G4)*

| Qué | Respuesta |
|---|---|
| Dónde vive | Fase 1: servidor interno de GTE o PC del PM durante la prueba (P-05). Fase 2: dentro de CLOVEX o a su lado, según IT |
| Quién lo opera | PM en fase 1; IT en fase 2 |
| Cómo se revierte | Mientras está en sombra, el Excel sigue siendo el registro: se deja de usar el módulo y no se pierde nada. Después de G2, exportación completa a Excel con las columnas de hoy |
| URL registrada en `registry/` | Al desplegar |

## 14. Preguntas abiertas

| ID | Pregunta | De quién | Qué bloquea | Recomendación |
|---|---|---|---|---|
| P-01 | ~~¿Dónde vive el código?~~ **Decidida el 30/10/2026 por el PM: repo nuevo**, con contrato pensado para encajar en CLOVEX en fase 2. Nombre decidido el mismo día: `clovergy_coordination_hub` (id de registro `clovergy-coordination-hub`). Su creación se pide aparte (no se crea nada sin permiso) | PM | — | — |
| P-02 | ~~¿Qué guía de diseño se sigue?~~ **Decidida el 30/10/2026 por el PM: la de la webapp de `clovergy_pm` (§3.1).** Para el encaje en CLOVEX de la fase 2 seguirá haciendo falta saber su tecnología (pregunta a IT) | PM | — | — |
| P-03 | ¿Quién de IT es el responsable de la spec y valida el encaje en CLOVEX? | Coordinador de IT | G3 | — |
| P-04 | ¿Autenticación con Microsoft Entra (cuentas de la empresa)? | TI | H-8 | Sí, si TI lo permite |
| P-05 | ¿Hay un servidor interno donde alojar la fase 1, o se prueba en el PC del PM? | TI | Despliegue | Probar en el PC hasta G2 |
| P-06 | ¿Se quiere Power BI además del panel propio, y hay licencias? | PM | Exportación para análisis externo | Panel propio; exportar para Power BI solo si ya hay licencias |
| DC-05 | ¿Quién es el registro maestro de INC y FB a medio plazo: este módulo o la página de incidencias del portal Clovergy (PRD, D4)? | PM | La salida de la sombra | Este módulo para lo que va a IT; el portal para las incidencias operativas del cliente, enlazadas por ID |
