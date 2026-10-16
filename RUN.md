# 🚦 RUN — continuar el desarrollo de Superharness

> 📝 **Estado:** READY · 📅 **Última actualización:** 2026-10-16 · 🎨 **Perfil visual:** `markdown-mermaid@1.0.0`
> 🎯 **Proyecto:** `F:\Proyectos\superharness` · 📋 **Plan:** [checklist y fichas](super-sdd-plan/14-plan-de-accion.md) · 🧾 **Registro:** [.execution/LOG.md](.execution/LOG.md)

<a id="entrada"></a>

## 🚪 Instrucción de entrada para una IA nueva

Lee este archivo completo antes de editar. Reanuda la tarea activa; si no hay una, toma la siguiente pendiente cuyas dependencias estén completas. Abre únicamente su ficha, referencias necesarias y código relacionado. Implementa y verifica esa tarea, guarda evidencia y actualiza el checkpoint antes de terminar la sesión. No necesitas el historial del chat.

**Este archivo permite arrancar; no sustituye leer el código ni la evidencia.** Ningún Markdown ejecuta procesos por sí mismo. Hasta construir la integración, el usuario pide continuar en el chat y la IA sigue este protocolo manual.

Prompt reutilizable:

```text
Lee F:\Proyectos\superharness\RUN.md y continúa con la siguiente tarea habilitada.
Comprueba primero el estado real del repositorio. Implementa y valida la tarea,
registra la evidencia y actualiza RUN.md y el checklist antes de terminar.
Si quedas bloqueado, guarda el checkpoint exacto y no marques la tarea como completa.
```

Por defecto, «la siguiente tarea» significa una tarea principal hasta terminarla o registrar un bloqueo real. Solo recorrer varias si el usuario pide continuar con un lote o hasta un hito. No detener una tarea por haber completado su primer subpaso.

<a id="estado"></a>

## 📍 Estado operativo actual

```yaml
run_schema: 1
project_id: superharness
status: READY
active_task: null
next_task: SH-001
checkpoint: "Diseño general disponible; iniciar contratos del motor."
last_completed_task: null
last_session: PLAN-20261016-001
baseline_head_observed: "5d90f43"
branch_at_resume: "Consultar Git; no asumir que sigue igual."
development_started: false
blocking_questions: []
environment_notes:
  - "Go no se encontró en PATH en esta sesión; comprobar instalación en SH-002."
  - "Codex y Git sí se localizaron; disponibilidad no equivale a autenticación probada."
working_tree_notes:
  - "Existen cambios documentales previos sin commit; preservarlos."
  - "No hay go.mod ni binario Superharness en la raíz inspeccionada."
```

Este bloque es un cursor escrito por la IA, no una prueba de finalización. READY indica trabajo seleccionable, IN_PROGRESS una tarea activa, BLOCKED una tarea sin avance posible y COMPLETED el plan íntegramente verificado. Esos estados pertenecen a este bootstrap y no sustituyen los estados de SPEC del producto.

<a id="siguiente"></a>

## ▶️ Siguiente tarea: SH-001 — contratos y decisiones de arranque

**Objetivo inmediato:** convertir el diseño actual en contratos inequívocos para implementar Go.

**Entradas mínimas:** [comandos](super-sdd-plan/02-comandos-y-ciclo.md), [arquitectura](super-sdd-plan/03-arquitectura-go-y-estado.md), [contratos](super-sdd-plan/07-contratos-y-plantillas.md), [separación de repositorios](super-sdd-plan/11-repositorio-de-conocimiento.md) y [chat](super-sdd-plan/13-interfaz-codex-y-despacho.md). Consultar [visión](super-sdd-plan/01-vision-y-decisiones.md) para resolver decisiones. La [ficha SH-001](super-sdd-plan/14-plan-de-accion.md#sh-001) define su aceptación.

**Pasos concretos:**

1. Consultar Git y preservar modificaciones existentes. Crear ID de sesión y registrar inicio en el LOG; reservar cooperativamente SH-001 actualizando active_task.
2. Extraer campos, estados, transiciones, errores y versiones comunes; identificar contradicciones sin inventar requisitos.
3. Crear bajo `.execution/contracts/` los contratos de bootstrap y ejemplos válidos/inválidos. Documentar en un README con iconos la semántica de cada campo y cómo verificarlo.
4. Registrar decisiones sobre versión del protocolo, persistencia, identidad de solución, publicación y scopes. Detallar en un ADR cualquier cambio al plan.
5. Revisar consistencia con los ocho comandos, el alta única, los repositorios separados y el contexto nuevo. Esta revisión documental no acredita tests ejecutables.
6. Guardar evidencia de revisión, marcar SH-001 y actualizar este bloque para SH-002, incluyendo los nuevos archivos que debe leer la IA.

**Fuera de esta tarea:** implementar fases, instalar skills, conectar Jira/GitHub o modificar DevelApi. La ausencia de Go no bloquea estos contratos documentales.

**Resultado esperado:** contratos revisables y decisiones explícitas; siguiente tarea de entorno preparada. No afirmar que la aplicación ya ejecuta esos contratos.

<a id="invariantes"></a>

## 🧭 Decisiones que se conservan entre sesiones

- Interfaz principal: /super-* desde el chat; skill delgada → Go → trabajador en sesión nueva. La aplicación está pendiente de implementación.
- Init una vez por solución; close publica documentación y grafo para la próxima feature.
- Análisis, SPEC y documentación del proyecto analizado van a su Git externo. Ejemplo: develapi-superharness; carpetas gestionadas de primer nivel con punto.
- La herramienta usa SDD y TDD; solo su fase develop podrá modificar código del producto analizado. Review crea/verifica PR hacia develop; close no autoriza un merge.
- Codex primero, Claude con el mismo contrato. El modelo no decide saltar gates.
- Todos los Markdown generados, incluido este plan, siguen el [perfil visual](super-sdd-plan/12-estandar-visual-markdown.md). Mermaid donde ayuda a explicar.
- Los documentos actuales son un plan manual de construcción. Cuando Superharness se autoanalice, sus SPEC y conocimiento generado también irán a un repo externo.
- El objetivo de consistencia se mide por contratos, requisitos y comportamiento; no se promete texto idéntico.

<a id="protocolo"></a>

## 🔄 Protocolo obligatorio de continuación

| Momento | Acción |
|---|---|
| Antes de tocar archivos | Leer instrucciones aplicables; confirmar raíz, branch, HEAD, status/diff y permisos. No descartar ni incluir cambios ajenos. |
| Selección | Consultar checklist y última evidencia. Si existe active_task, reanudarla. Si no, elegir el ID numérico menor pendiente con dependencias completas. |
| Discrepancia | El cursor next_task es orientativo: si apunta a una completada o bloqueada por dependencias, recalcular y registrar la corrección. Una casilla sin evidencia válida se trata como pendiente de reconciliación. |
| Inicio | Crear sesión `DEV-AAAAMMDD-HHMMSS-<sufijo>`, registrar agente/branch/HEAD y cambios previos; poner IN_PROGRESS y active_task. |
| Diseño | Leer ficha y referencias; resolver decisiones rutinarias con evidencia. Si una decisión cambia alcance o necesita datos externos, registrar pregunta concreta. |
| Implementación | Ciclos pequeños, TDD para comportamiento y permisos mínimos. No invocar fases aún inexistentes ni generar resultados ficticios. |
| Checkpoint | Actualizar tras una decisión relevante, prueba importante, tarea parcial o interrupción prevista; indicar archivo/símbolo y siguiente acción exacta. |
| Terminación | Verificar todos los criterios; registrar evidencia primero, después marcar checklist, después actualizar RUN y su bloque «Siguiente tarea». |
| Cierre de sesión | Informar tarea terminada o bloqueada, validación real y siguiente ID. Persistir antes de depender de una respuesta del usuario. |

**Fuente de verdad:** las casillas del [plan](super-sdd-plan/14-plan-de-accion.md#checklist) representan terminación; el [LOG](.execution/LOG.md) justifica cada cambio; este archivo indica dónde retomar. No duplicar listas de tareas completadas ni porcentajes manuales en otros archivos.

**Concurrencia:** este procedimiento manual admite una sola IA escritora. active_task es una reserva cooperativa, no un lock atómico. Si hay evidencia de otro escritor activo, coordinar antes de editar; no robar su tarea por antigüedad. El motor implementará locks reales en SH-004. Un valor IN_PROGRESS tras una interrupción no demuestra que siga vivo: comprobar sesión/procesos/estado antes de recuperar.

**Reanudación tras corte:** inspeccionar diff y evidencia; verificar si los pasos ya están aplicados, no repetir patches a ciegas. Si el LOG registra finalización pero la casilla sigue abierta, contrastar archivos/tests antes de reconciliar. Si la casilla está marcada sin evidencia suficiente, dejarla pendiente y anotar por qué. Volver a ejecutar los checks afectados si cambió su snapshot.

**Git:** no depender de una rama concreta ni cambiarla con trabajo ajeno pendiente. Si hace falta crear una rama, prefijo `codex/` salvo instrucción explícita distinta. Un commit puede agrupar código, evidencia y checkpoint de una tarea, pero ninguna casilla concede autorización para push/merge. Registrar commits existentes después de crearlos; no intentar incluir el hash del propio commit en su contenido.

<a id="evidencia"></a>

## 🧪 Evidencia y criterios para marcar una casilla

Registrar en el LOG:

- ID de tarea/sesión, estado anterior y nuevo, fecha, responsable y alcance.
- Archivos/símbolos relevantes, decisiones y criterios de aceptación comprobados.
- Comando exacto, cwd, exit code y resumen de cada verificación. Logs extensos bajo `.execution/evidence/<session-id>/`, saneados antes de versionar.
- Snapshot probado: commit si contiene los cambios; si hay cambios sin commit, HEAD base y hashes de los archivos modificados/no rastreados relevantes. Un HEAD solo no identifica un árbol sucio.
- RED/GREEN/REFACTOR cuando aplica; si no aplica, motivo concreto. «No ejecutado» es distinto de «pasó».
- Pendientes, bloqueo y siguiente acción. Enlaces a evidencia duradera, no solo a salida transitoria del chat.

No marcar done por haber escrito código, por un test con cero casos, por recibir una respuesta JSON válida o por ver un check remoto pendiente. Las pruebas de tareas anteriores solo siguen siendo válidas para el snapshot y alcance que acreditan.

Si una dependencia cambia e invalida trabajo aceptado, reabrir las casillas afectadas y registrar el motivo; no borrar la evidencia histórica. Pruebas diferidas obligatorias mantienen la tarea abierta.

<a id="checkpoint"></a>

## 💾 Checkpoint para reemplazar al terminar cada sesión

| Campo | Valor actual |
|---|---|
| Tarea en curso | Ninguna |
| Último paso comprobado | Plan operativo creado; implementación aún no iniciada |
| Archivos de trabajo | RUN.md, plan de acción y LOG; más documentación previa sin commit |
| Pruebas de aplicación | No ejecutadas: todavía no existe aplicación |
| Decisiones recientes | Cursor + checklist + evidencia; una IA escritora y una tarea por petición «siguiente» |
| Bloqueo real | Ninguno para SH-001; Go pendiente de comprobar en SH-002 |
| Siguiente acción exacta | Abrir ficha SH-001, consultar Git y extraer contratos del diseño |
| Qué no repetir | No reiniciar el plan ni tratar la documentación previa como trabajo ajeno descartable |

El próximo agente debe reemplazar estos valores y el detalle de «Siguiente tarea»; no limitarse a cambiar next_task. Mantener las reglas estables y llevar historia extensa al LOG.

<a id="limites"></a>

## ⚠️ Límites del mecanismo inicial

Un archivo permite recuperar contexto guardado, no recuerdos perdidos antes de escribirlo. Por eso se registran checkpoints durante el trabajo. No hay scheduler ni avance automático mientras no exista una ejecución activa: el usuario inicia la continuación desde el chat.

Al implementar el estado persistente de Superharness, migrar este seguimiento de forma explícita: importar tareas/recibos, comprobar equivalencia y dejar RUN como entrada a una vista del estado. No mantener dos motores de estado independientes.
