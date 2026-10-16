# 06. Integraciones con agentes, Jira y GitHub

## 1. Regla general

Las integraciones son adaptadores del motor. El LLM no dispone de credenciales para mutar Jira o GitHub directamente: propone resultados estructurados y Go ejecuta las operaciones permitidas. Así se puede probar idempotencia, auditar acciones y mantener las mismas reglas con Codex y Claude.

El modo inicial de conectores será lectura y simulación de escritura. Al configurar el producto se autorizarán explícitamente las operaciones deseadas —crear PR, actualizar el comentario de progreso, transicionar tickets— y esa política persistirá. No pedir confirmación repetida para cada operación ya autorizada.

## 2. Codex

Usar ejecución no interactiva: `codex exec --json` permite consumir eventos y `--output-schema` restringe la respuesta final. `--output-last-message` puede guardarla aparte. El modo de lectura es el predeterminado y `--sandbox workspace-write` habilita escritura donde corresponda. Crear una ejecución nueva por fase; `--ephemeral` controla la conservación de la sesión del proveedor, no sustituye el diario de Super. La autenticación CLI y el uso de API tienen condiciones propias. [Modo no interactivo de Codex](https://learn.chatgpt.com/docs/non-interactive-mode).

Forma orientativa de invocación para una fase documental:

```text
codex exec --json --sandbox read-only --output-schema <schema> -
```

Go envía el prompt por stdin, captura eventos y valida la salida. Para develop selecciona escritura en un checkout aislado. `doctor` comprobará versión, flags, autenticación, modelo y política efectiva. No usar flags antiguos por memoria ni ampliar permisos automáticamente si un comando falla.

Para accesos desde chat, Codex descubre skills de repositorio en `.agents/skills`. El empaquetado debe adaptarse a la superficie concreta; no se garantiza un slash arbitrario solo por crear un archivo Markdown. [Skills de Codex](https://learn.chatgpt.com/docs/build-skills).

## 3. Claude Code

El adaptador puede usar `claude -p`, `--output-format json` y `--json-schema`; debe leer el resultado estructurado y los metadatos, no confundirlos con la respuesta textual. También existe salida de eventos. Cada fase comienza sin `--continue` ni `--resume`. [Ejecución programática de Claude Code](https://code.claude.com/docs/en/headless).

Las skills de proyecto en `.claude/skills/<nombre>/SKILL.md` pueden exponerse como slash commands. Un wrapper delega en la CLI Super; no reproduce su máquina de estados en el prompt. [Skills de Claude Code](https://code.claude.com/docs/en/skills).

El aislamiento de instrucciones de usuario, herramientas y memoria se comprobará por versión. El modo `--bare` documentado puede ayudar a controlar la carga automática, pero cambia la autenticación y los componentes cargados; no se activará suponiendo que conserva el login de suscripción. El contrato de Super debe funcionar con el modo elegido y probado.

## 4. La experiencia /super-* y el clear

Hay tres capas diferentes:

| Capa | Garantía |
|---|---|
| `super requirements SPEC-001` | Contrato portable y determinista de despacho |
| `/super-requeriments SPEC-001` en `super shell` | Alias idéntico en una consola controlada por la aplicación |
| Skill/comando dentro de Codex o Claude | Acceso dependiente de la superficie y versión del proveedor |

La consola propia ofrece exactamente la sintaxis solicitada. Un chat existente puede conservar su historial visual, pero solo transmite el ID de SPEC y parámetros explícitos al orquestador. El agente de trabajo siempre arranca aparte. Pedir al modelo que escriba `/clear` no constituye una garantía programática.

La documentación de Codex describe `/new` para iniciar otro chat; esa función de interfaz es distinta del proceso nuevo que garantiza Super. [Comandos de Codex](https://learn.chatgpt.com/docs/developer-commands?surface=cli).

## 5. Capacidades y compatibilidad

`doctor` generará un manifiesto con versión de CLI, autenticación disponible, modelos accesibles, schemas soportados, eventos, herramientas, límites, cancelación y sandbox. La interfaz común se limita a capacidades probadas; un proveedor que no pueda aplicar una política de fase debe rechazarla.

Guardar `requested_model` y `resolved_model` si se puede conocer. No cambiar de modelo al agotar cuota sin registrarlo. Un fallback requiere una opción explícita de configuración y un run nuevo. Las pruebas de aceptación comparan comportamiento y contratos; una sesión Claude no debe intentar interpretar el formato interno de sesión de Codex.

## 6. Jira

Se asume Jira Cloud para el primer adaptador. Jira Data Center necesita configuración y validación específicas. No se necesita automatizar el navegador ni «leer el panel» visualmente: integrar incidencias, campos y estados mediante API.

Jira Cloud REST v3 utiliza Atlassian Document Format en campos como descripción y comentarios. El importador conservará el original y una normalización legible. La documentación distingue OAuth 2.0 para integraciones y autenticación apropiada para scripts personales. [REST API v3 de Jira](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/).

Contrato propuesto:

| Operación | Momento | Control |
|---|---|---|
| Leer ticket y campos configurados | new/refresh | Snapshot, ID, fecha y hash |
| Consultar estados y transiciones | Configuración/sync | IDs descubiertos; no nombres universales |
| Actualizar comentario propio de progreso | Fases confirmadas | Un comentario con marcador estable, sin duplicados |
| Enlazar SPEC/PR | review | Guardar IDs y URL verificados |
| Transicionar a trabajo/revisión | Según política | Verificar transición permitida y estado actual |
| Transicionar a terminado | close final | Merge, docs y verificación confirmados |

`jira.fields.acceptance_criteria` será un mapeo configurable: no asumir un `customfield_*` universal. Persistir campos usados, adjuntos admitidos y enlaces; limitar tamaño y no descargar URLs arbitrarias incluidas en el ticket.

Si cambia el ticket tras requirements, mostrar diff y crear revisión explícita. No reemplazar requisitos aceptados durante develop. Si la edición local y Jira divergen, pedir una resolución persistida; no aplicar «última escritura gana» a ciegas.

## 7. GitHub

MVP: repositorio Git existente y acceso autenticado; API REST para PR o un adaptador `gh` probado. Para uso en equipo, evaluar GitHub App con permisos ajustados y tokens de instalación. El adaptador no registra secretos en comandos ni artefactos.

La API de PR permite gestionar base, head, título, cuerpo y modo draft. Super fijará `base=develop`, verificará que la rama exista y leerá el recurso tras crearlo. [Pull requests en GitHub REST](https://docs.github.com/en/rest/pulls/pulls).

Contenido de PR generado desde manifiestos:

```markdown
# SPEC-001: <resultado de la entrega>

## Problema y comportamiento resultante
<Qué cambia para el usuario y por qué>

## Trazabilidad
SPEC: <ruta o enlace> · Jira: <enlace verificado>
Requisitos: REQ-001, REQ-002 · ADR: ADR-001 o «no requiere nuevo ADR»

## Validación
<Comandos, resultados, referencia de evidencia y commit evaluado>

## Riesgos y operación
<Migración, compatibilidad, rollback y limitaciones reales>
```

La revisión remota necesita checks del commit correcto, reglas de rama, aprobaciones válidas y ausencia de conflictos. `unknown` o un check aún pendiente no equivalen a verde. Guardar `reviewed_head_sha`, `base_sha`, `checks_sha` y `merged_commit_sha`; distinguir squash/rebase/merge y verificar la relación resultante por API/Git, sin exigir igualdad entre head y commit de merge.

El orquestador puede publicar hallazgos como comentario/check si está autorizado, pero no convierte su propia evaluación en aprobación humana. Tampoco debe crear un comentario nuevo por cada consulta de estado sin cambios.

## 8. Permisos por fase

| Fase | Código | Documentación | Jira | GitHub |
|---|---|---|---|---|
| init | Lectura | Perfil/inventario | Lectura opcional | Diagnóstico opcional |
| new | Lectura | Análisis | Leer fuente | Lectura opcional |
| requirements/design/task | Lectura | Artefactos de fase | Progreso si configurado | Lectura opcional |
| develop | Escritura aislada | Evidencia | Progreso si configurado | Push de rama de SPEC si configurado |
| review | Lectura; runner aislado | Informe | Enlace/progreso | Crear/actualizar PR, comprobar checks |
| close | Lectura | Documentación de proyecto | Transición final configurada | Consultar integración; PR documental si necesaria |

El orquestador puede hacer commits documentales y push sin modificar código de producto. Los cambios de configuración que afectan build/runtime/CI cuentan como código y solo se preparan en develop.

## 9. Robustez operativa

Timeouts, reintentos con backoff y respeto de límites del servicio. Reintentar lecturas; antes de repetir escrituras, reconciliar por identificador o marcador. No asumir un header de idempotencia que el endpoint no documente.

Guardar credenciales en el almacén del sistema o gestor de secretos; configuración solo con referencias. Si se añaden webhooks, validar firma, deduplicar entregas, manejar orden distinto y consultar estado actual antes de transicionar. En el MVP bastan consultas al ejecutar cada comando; no hace falta un servicio siempre encendido.

No se han probado cuentas, permisos, protecciones de rama ni el workflow real de tu Jira. La primera configuración debe descubrirlos y producir un diagnóstico reproducible.
