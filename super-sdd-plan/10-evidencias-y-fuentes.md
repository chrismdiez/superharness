# 10. Evidencias, fuentes y límites de verificación

## 1. Qué se ha hecho

Inspección de lectura realizada el 16 de octubre de 2026 sobre `F:\Proyectos\_Develapi`. Se enumeraron soluciones, proyectos, archivos C#, posibles archivos de instrucciones, configuración de estilo y referencias a herramientas de tests. Se leyeron ejemplos de contratos, validadores, casos de uso, repositorios, entidades, DI y endpoints.

El plan se escribió inicialmente en `C:\Users\cmigu\Documents\New project\super-sdd-plan` y se copió al directorio del proyecto confirmado por el usuario: `F:\Proyectos\superharness\super-sdd-plan`. Esta última ubicación es la copia de trabajo vigente. No se ha implementado la CLI, instalado dependencias, modificado DevelApi, llamado a sus servicios ni creado recursos en Jira/GitHub.

## 2. Inventario observado

| Dato | Resultado | Límite |
|---|---|---|
| Carpetas principales | core, gateway, library, endpoints, web | No equivalen necesariamente a repositorios remotos verificados |
| Soluciones .NET localizadas | Core, Gateway, Library; formato `.slnx` | 3 soluciones |
| Proyectos `.csproj` | 32 | Excluidos `bin/obj` |
| Frameworks declarados | 32 proyectos `net10.0` | No compilados |
| Fuentes C# localizadas | 431 | Excluidos `bin/obj` |
| `.editorconfig` localizado | Gateway, reglas para Markdown | No define estilo C# completo |
| Tests convencionales | Sin `IsTestProject`, xUnit, NUnit, MSTest o Microsoft.NET.Test.Sdk en los proyectos inspeccionados | No descarta pruebas externas o no convencionales |
| AGENTS.md/CLAUDE.md | No localizados por las búsquedas realizadas en el árbol | No se afirma ausencia en otras ubicaciones del equipo |
| Git en F: | `git -C ...` devolvió Permission denied | Sin ramas, SHA, remotos ni estado de checkout verificados |

No se necesitan permisos adicionales para entregar este plan. Para implementar en el repositorio será necesario que el entorno permita trabajar en el checkout elegido.

## 3. Evidencias locales de las convenciones

| Evidencia | Qué respalda |
|---|---|
| [Gateway .editorconfig](F:/Proyectos/_Develapi/develapi-gateway/.editorconfig) | Markdown UTF-8 BOM y CRLF; alcance limitado |
| [Gateway UseCases csproj](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.UseCases/DevelApi.Gateway.UseCases.csproj) | net10, nullable, documentación XML, StyleCop y referencias |
| [Gateway Core csproj](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.Core/DevelApi.Gateway.Core.csproj) | Paquetes compartidos y referencia Common |
| [Library UseCases csproj](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.UseCases/DevelApi.Library.UseCases.csproj) | Variante de dependencias de Library |
| [StyleCop de UseCases](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.UseCases/stylecop.json) | CompanyName PlaceholderCompany |
| [Contrato IInteractorService](F:/Proyectos/_Develapi/develapi-core/src/DevelApi.Core.Abstractions/Interfaces/IInteractorService.cs) | Validate/Execute y CancellationToken |
| [Interactor compartido](F:/Proyectos/_Develapi/develapi-core/src/DevelApi.Core.DependencyInjection.Application/Implementations/Interactor.cs) | Secuencia validar, ejecutar y presentar |
| [UserCreateService](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.UseCases/User/Create/UserCreateService.cs:26) | Organización, inyección, comentarios y estilo |
| [UserCreateRequestValidator](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.Dtos/User/Create/UserCreateRequestValidator.cs:15) | FluentValidation en Dtos y constantes |
| [BookCreateService](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.UseCases/Book/Create/BookCreateService.cs:24) | Caso de uso, persistencia por interfaces y auditoría |
| [BookEndpoints](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.Api/Endpoints/BookEndpoints.cs:19) | Adaptación HTTP mediante IController |
| [BookGetQueries](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.EF.Repositories/Book/Get/BookGetQueries.cs) | Contexto de lectura y especificaciones |
| [UserCreateCommands](F:/Proyectos/_Develapi/develapi-gateway/src/DevelApi.Gateway.Persistence.EF.Repositories/UseCases/User/Create/UserCreateCommands.cs:18) | Variante Persistence y contexto de escritura |
| [Book](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.Core/Entities/Book.cs:14) | Entidad, IDs, nulabilidad y fechas |
| [DependencyContainer de Library](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.IoC/DependencyContainer.cs) | Composición y registro dinámico |
| [BookGetListRequestValidator](F:/Proyectos/_Develapi/develapi-library/src/DevelApi.Library.Dtos/Book/GetList/BookGetListRequestValidator.cs:18) | Page ≥ 1 y Limit entre 1 y 100 |

Las rutas son del equipo donde se realizó la inspección. Para hacer portable el perfil se deben almacenar también rutas relativas, repo ID y hashes durante init. Las líneas son ayudas de navegación, no identificadores permanentes.

## 4. Fuentes primarias consultadas

| Tema | Fuente | Uso en el plan |
|---|---|---|
| Codex no interactivo | [OpenAI: non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode) | Proceso por fase, eventos y respuesta estructurada |
| Codex skills | [OpenAI: build skills](https://learn.chatgpt.com/docs/build-skills) | Adaptadores de entrada y descubrimiento local |
| Codex comandos | [OpenAI: developer commands](https://learn.chatgpt.com/docs/developer-commands?surface=cli) | Distinguir interfaz y ejecución nueva |
| Claude programático | [Anthropic: headless](https://code.claude.com/docs/en/headless) | CLI y outputs del segundo adaptador |
| Claude skills | [Anthropic: skills](https://code.claude.com/docs/en/skills) | Slash commands y wrappers |
| Gherkin | [Cucumber: Gherkin reference](https://cucumber.io/docs/gherkin/reference/) | Escenarios observables y sintaxis |
| Grafo | [CodeGraphContext](https://github.com/CodeGraphContext/CodeGraphContext) | Candidato de indexación, pendiente de PoC |
| Análisis C# | [Microsoft: Roslyn workspace](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/work-with-workspace) | Alternativa de indexador semántico |
| Formato .NET | [Microsoft: dotnet format](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-format) | Comprobación de estilo |
| Jira | [Atlassian: REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/) | Importación y operaciones de incidencias |
| GitHub PR | [GitHub: pull requests](https://docs.github.com/en/rest/pulls/pulls) | Creación y consulta de PR |
| GitHub review | [GitHub: reviews](https://docs.github.com/en/rest/pulls/reviews) | Revisión asociada al commit |

La consulta documental valida capacidades generales, no su disponibilidad en tus cuentas ni la compatibilidad de una versión concreta instalada. Antes de implementar cada adaptador se fijarán versiones y tests de contrato. No se han usado cifras de precios ni prometido un modelo específico.

## 5. Decisiones que debe cerrar setup

| Decisión | Valor recomendado provisional | Consecuencia |
|---|---|---|
| Qué producto es «codegraph» | Evaluar CodeGraphContext | El adaptador permite reemplazarlo |
| Repo piloto | Library | Menor superficie para validar el flujo |
| Tipo de Jira | Cloud inicialmente | Data Center requiere otro adaptador |
| Destino de PR | develop, según petición | Fallar si no existe; no sustituir por main |
| Modelo Codex/Claude | Selección explícita disponible | Fijar ID/configuración y evaluar |
| SDK .NET | Mantener net10; fijar SDK concreto | Resolver compatibilidad con paquetes y herramientas |
| Framework de tests | xUnit si no hay otro establecido | Probar compatibilidad y fijar versión |
| Nombre de compañía | Configurado por el equipo | No inventar copyright |
| Aprobaciones | Según responsables y reglas de rama | Autoevaluación no sustituye aprobación requerida |
| Credenciales/hosting | Almacén de secretos; ejecución local inicial | Determina operaciones posibles |

No es necesario contestar todo antes de leer y revisar el plan. Estas decisiones forman parte de la fase 0 y del diagnóstico inicial, con valores explícitos y sin supuestos ocultos.
