# 01. Visión, alcance y decisiones

## 1. Qué significa SDD en esta propuesta

Se interpreta SDD como **Specification-Driven Development**: la especificación versionada define el alcance, los comportamientos esperados y las restricciones. El diseño, las tareas, el código, las pruebas y la documentación deben poder relacionarse con ella.

La cadena obligatoria será:

```text
Fuente local/Jira → SPEC → REQ → AC/escenario → diseño/ADR → TASK → test → cambio → review → PR → cierre
```

Un requisito sin una forma de comprobarlo está incompleto. Una tarea sin requisito o habilitador técnico justificado es alcance añadido. Una prueba verde que no demuestra el criterio de aceptación no cierra ese criterio.

**Gherkin no es un modelo de lenguaje.** Es una sintaxis para describir comportamientos mediante escenarios. Codex o Claude pueden producir esos escenarios. Los requisitos técnicos adicionales —rendimiento, compatibilidad, dependencias, concurrencia, observabilidad— necesitan también condiciones medibles y un método de verificación. [Referencia de Gherkin](https://cucumber.io/docs/gherkin/reference/).

## 2. Producto y límites

| Capacidad | MVP | Evolución |
|---|---|---|
| CLI Go y archivos locales | Sí | Consola interactiva y panel opcional |
| Adaptador Codex | Sí | Claude Code con los mismos contratos |
| Un repositorio por SPEC | Sí | Coordinación explícita entre varios repositorios |
| Fuente de ticket local | Primero | Jira Cloud; Data Center como adaptador distinto |
| GitHub PR hacia develop | Sí, tras flujo local | Webhooks y GitHub App para equipos |
| Grafo de C# | Piloto y modo degradado explícito | Dependencias entre servicios y otros lenguajes |
| TDD y revisión | Sí | Análisis más costoso según riesgo |
| Merge/despliegue automático | Fuera del MVP | Política separada con permisos propios |
| Idéntico código entre modelos | No garantizable | Evaluación de equivalencia funcional |

El alcance es el ciclo de ingeniería hasta integración y actualización del conocimiento. Despliegue, aprobación de producción, seguimiento de incidentes y gestión comercial requieren contratos adicionales; no se darán por resueltos por cerrar una SPEC.

## 3. Decisiones arquitectónicas iniciales

| ID | Decisión propuesta | Motivo y coste |
|---|---|---|
| ADR-SUP-001 | Orquestador en Go con adaptadores | Las reglas de negocio del flujo no dependen del proveedor |
| ADR-SUP-002 | CLI de agentes como primera integración | Reutiliza sus herramientas; requiere fijar y probar versiones |
| ADR-SUP-003 | Sesión nueva por fase y tarea reanudada | Elimina dependencia del chat; exige buen empaquetado de contexto |
| ADR-SUP-004 | Markdown + JSON + diario SQLite | Legibilidad, validación y recuperación; exige reglas claras de autoridad |
| ADR-SUP-005 | Grafo como índice derivado | Nunca sustituye al código ni a la especificación aprobada |
| ADR-SUP-006 | Escritura de código exclusiva de develop | Review detecta y describe; develop corrige y vuelve a probar |
| ADR-SUP-007 | Control de cambios por hashes y revisión | Evita aprobar resultados de una versión anterior |
| ADR-SUP-008 | Perfil DevelApi versionado | Estandariza sin imponer una arquitectura ajena al repositorio |
| ADR-SUP-009 | Review independiente y cierre en dos momentos | Evita que la documentación posterior invalide una aprobación sin advertirlo |

Estos ADR están propuestos, no aceptados mediante una decisión real del equipo. Para implementarlos, convertir cada fila en un ADR del propio producto con contexto, alternativas y consecuencias.

## 4. Cómo conseguir resultados similares

La unidad reproducible no es solo `SPEC-001`. Es:

```text
source hash + revision SPEC + commit del repo + cambios locales admitidos
+ versiones de perfil, plantillas, prompts, herramientas y proveedor
+ configuración de modelo + selección de contexto + respuestas aprobadas
```

El mismo identificador apuntando a un Jira que ha cambiado ya no es la misma entrada. Personas con distintos permisos pueden ver tickets o código diferentes: no se debe ocultar esa diferencia bajo la etiqueta «mismo prompt».

Recomendaciones concretas:

1. El binario asigna IDs estables, ordena secciones y genera Markdown desde datos validados. El modelo no inventa nombres de carpetas.
2. Cada comando tiene un esquema de entrada y salida; campos desconocidos, referencias rotas y resultados incompletos bloquean la publicación.
3. Se fijan versiones en un archivo de bloqueo. No elegir automáticamente «el último modelo» o «la última plantilla» en cada ejecución.
4. Se controla la carga de instrucciones locales, memoria automática, plugins y herramientas. Registrar la configuración efectiva y el hash de las instrucciones aplicables.
5. Se explicitan los supuestos de bajo impacto. Las preguntas que cambien contratos, permisos, datos o comportamiento quedan bloqueantes hasta una respuesta persistida.
6. La recuperación de contexto aplica reglas de orden y presupuesto; se guardan las referencias elegidas. No usar búsquedas arbitrarias distintas para cada persona sin registrarlas.
7. Las comprobaciones decisivas son ejecutables: esquemas, referencias, build, tests, formato, política de archivos, CI y trazabilidad.
8. Cambiar de Codex a Claude requiere superar evaluaciones de contrato; no reescribir la especificación para acomodar al nuevo modelo.

Los parámetros que reducen variación solo se fijan cuando el modelo los admite. No se presupone soporte de temperatura o semilla ni se promete determinismo matemático.

## 5. Estándares y excepciones

Orden de autoridad propuesto: políticas organizativas y permisos efectivos → configuración aprobada del repositorio → perfil .NET fijado → especificación aceptada → diseño → tarea → material recuperado. Una instrucción dentro de un ticket o un comentario de código es material de trabajo, no una autorización para cambiar las reglas del ejecutor.

Una excepción necesita `rule_id`, razón, alcance, responsable y revisión de SPEC. Nunca convertir un fallo en éxito suprimiendo una regla silenciosamente. La deuda previa se registra como baseline; no obliga a reformatear todo el repositorio en cada feature.

## 6. Indicadores de éxito

| Indicador | Objetivo inicial propuesto |
|---|---|
| Artefactos con esquema válido | 100 % antes de avanzar |
| Requisitos aceptados con prueba o evidencia prevista | 100 % |
| Criterios cerrados con evidencia realmente ejecutada | 100 % |
| Escrituras de código fuera de develop | 0 |
| PR duplicadas al repetir review | 0 |
| Recuperaciones de interrupción sin perder tarea confirmada | 100 % en pruebas de fallos |
| Cierre sobre SHA distinto al aprobado | 0 |
| Similitud de requisitos identificados en casos evaluados | Objetivo ≥ 0,95 F1 frente a referencia curada |
| Coste de contexto | Medir por SPEC, incluyendo indexación y reintentos |

Son criterios de aceptación del producto propuesto, no resultados ya medidos.
