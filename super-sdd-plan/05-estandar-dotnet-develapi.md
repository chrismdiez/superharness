# 05. Estándar .NET basado en DevelApi

## 1. Alcance y grado de certeza

Se inspeccionaron nombres de proyectos y archivos, configuraciones y una muestra de código en `F:\Proyectos\_Develapi` el 16 de octubre de 2026. Se localizaron **32 `.csproj`, todos con `TargetFramework=net10.0`, y 431 `.cs` excluyendo `bin/obj`**. No se ejecutó build ni tests. Git no permitió acceder a esos directorios desde esta sesión, por lo que no hay un commit de referencia verificado.

Etiquetas usadas: **O** = observado en archivos; **P** = propuesta para formalizar o mejorar; **V** = validar antes de convertir en obligación global. Este estándar es un perfil inicial, no una afirmación de que todo el repositorio ya lo cumple. Las [evidencias](10-evidencias-y-fuentes.md) permiten contrastarlo.

## 2. Estructura de soluciones

**O:** tres soluciones `.slnx` localizadas: Core, Gateway y Library. Core contiene varios proyectos de paquetes compartidos dentro de su solución. Las carpetas `develapi-endpoints` y `develapi-web` también existen, pero este perfil se centra en .NET.

Estructura de aplicaciones observada:

```text
develapi-<aplicacion>/
  DevelApi.<Aplicacion>.slnx
  src/
    DevelApi.<Aplicacion>.Api/
    DevelApi.<Aplicacion>.Common/
    DevelApi.<Aplicacion>.Core/
    DevelApi.<Aplicacion>.Dtos/
    DevelApi.<Aplicacion>.UseCases/
    DevelApi.<Aplicacion>.Services/
    DevelApi.<Aplicacion>.Events/
    DevelApi.<Aplicacion>.Middleware/
    DevelApi.<Aplicacion>.IoC/
    <proyectos de persistencia según perfil>
```

| Responsabilidad | Convención a conservar |
|---|---|
| Api | Host, endpoints, registro de rutas y adaptación HTTP |
| Core | Entidades, enums, value objects, interfaces de servicios y contratos por caso de uso |
| Dtos | Request/Response y validadores FluentValidation por entidad/operación |
| UseCases | Orquestación de negocio mediante `IInteractorService<TRequest,TResponse>` |
| EF.Repositories | Implementación de Commands/Queries sobre contextos EF |
| EF.DataContexts | Contextos, configuración de persistencia y guardas |
| Events | Handlers de eventos |
| Services | Implementaciones de servicios transversales o externos |
| IoC | Composición y métodos de extensión de registro |
| Common | Constantes y utilidades compartidas de la aplicación |

**O:** Gateway usa `Persistence.EF.Repositories`, `Persistence.EF.DataContexts` y `Persistence.IoC`; Library usa `EF.Repositories` y `EF.DataContexts`, sin el segmento `Persistence`. Gateway sitúa repositorios de casos de uso bajo `UseCases/`; Library los sitúa directamente bajo `Book/`.

**P:** mantener dos variantes del mismo perfil (`gateway`, `library`). No renombrar Library para que parezca Gateway durante una feature. Para aplicaciones nuevas, elegir una variante en un ADR antes de generar archivos. No imponer carpetas genéricas `Domain/Application/Infrastructure` ajenas al patrón existente.

**O:** `develapi-core` distribuye proyectos de capacidades, por ejemplo `DevelApi.Core.Email.Abstractions` y `DevelApi.Core.Email.Application`. No imponerle el árbol de una API.

## 3. Dependencias entre capas

En los proyectos inspeccionados, Gateway Core referencia Common; UseCases referencia Core, Dtos y Events. Library UseCases referencia Common, Core, Dtos y Events. Library Dtos referencia Common y Core. Api referencia IoC. Esto se debe respetar al extraer el baseline; no describir el estado actual como una arquitectura con dominio completamente independiente si las referencias dicen otra cosa.

**P:** prohibir ciclos; prohibir acceso directo de UseCases a un DbContext o repositorio EF concreto; usar contratos del Core. Las dependencias existentes son baseline, no autorización para nuevas dependencias arbitrarias. Tests de arquitectura deben controlar el grafo `.csproj` y usos de namespaces con excepciones explícitas.

Flujo observado y a conservar:

```text
Endpoint → IController<TRequest,TResponse> → IInputPort / Interactor
         → IInteractorService.Validate → IInteractorService.Execute
         → IOutputPort / Presenter
```

No añadir MediatR, otro mediator o un bus nuevo sin necesidad y ADR. La separación Commands/Queries observada no demuestra event sourcing ni CQRS distribuido completo.

## 4. Carpetas y nombres

| Elemento | Regla | Ejemplo |
|---|---|---|
| Proyecto/namespace | PascalCase por segmentos | `DevelApi.Library.UseCases` |
| Caso de uso | `<Entidad>/<Operación>` | `Book/Create/` |
| Servicio | `<Entidad><Operación>Service.cs` | `BookCreateService.cs` |
| DTO de entrada/salida | `<Entidad><Operación>Request/Response.cs` | `BookCreateRequest.cs` |
| Validador | `<Entidad><Operación>RequestValidator.cs` | `UserCreateRequestValidator.cs` |
| Contrato | Prefijo `I` | `IBookCreateCommands.cs` |
| Persistencia | `<Entidad><Operación>Commands/Queries.cs` | `BookGetQueries.cs` |
| Especificación de consulta | Nombre del criterio + `Specification` | `BookCreateByIsbnSpecification.cs` |
| Handler | Evento + `EventHandler` | `UserCreatedEventHandler.cs` |
| Extensiones | Nombre descriptivo + `Extensions` | `BookGetExtensions.cs` |
| Tests propuestos | Tipo + `Tests.cs` | `BookCreateServiceTests.cs` |

**O:** hay variantes singulares `UserGetExtension.cs` y plurales. **P:** preferir `Extensions` para archivos nuevos y conservar nombres existentes salvo refactor planificado.

**O:** identificadores y comentarios son mayoritariamente ingleses; hay excepciones como `GenerarPassword` y nombres históricos como `UserSuscriptions`. **P:** inglés en código nuevo; no corregir nombres públicos históricos si implica romper contratos fuera de alcance. Documentación de SDD en español; un idioma fijado por repositorio.

Rutas para un caso de uso nuevo de Library:

```text
src/DevelApi.Library.Dtos/Book/<Operacion>/
src/DevelApi.Library.Core/UseCases/Book/<Operacion>/Commands/
src/DevelApi.Library.Core/UseCases/Book/<Operacion>/Queries/
src/DevelApi.Library.Core/UseCases/Book/<Operacion>/Specifications/
src/DevelApi.Library.UseCases/Book/<Operacion>/
src/DevelApi.Library.EF.Repositories/Book/<Operacion>/
```

Crear solo los componentes necesarios. Un GET no necesita obligatoriamente Commands ni un evento.

## 5. Formato y estilo C#

**O:** la muestra usa namespaces de archivo (`namespace X;`), `using` después del namespace, llaves Allman, cuatro espacios, tipos y miembros públicos PascalCase, parámetros/locales/campos privados camelCase sin `_`, campos `readonly` para dependencias y acceso con `this.`. Usa constructores explícitos e inyección por constructor.

**P:** fijar esas convenciones para archivos nuevos. Orden de miembros: campos, constructores, propiedades, métodos públicos, métodos privados. Orden de `using` consistente por grupos según configuración; no asumir que la muestra ya es uniforme. Métodos pequeños con una responsabilidad; evitar regiones y comentarios que solo repitan el nombre.

**O:** los contratos actuales usan `Validate`, `Execute`, `Create`, `SaveChanges` sin sufijo `Async`. **P:** respetar esos nombres al implementar interfaces; para nuevos contratos asíncronos independientes se recomienda `Async`. La excepción se codifica por símbolo/contrato, no mediante renombrado masivo.

**O:** los `.csproj` usan tabulaciones en la muestra, mientras C# usa espacios. El `.editorconfig` localizado en Gateway solo define `*.md` como UTF-8 con BOM y CRLF. No hay evidencia de una regla C# central completa.

Propuesta de sección C# a incorporar mediante una tarea de develop, tras medir el impacto:

```ini
[*.cs]
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true
csharp_style_namespace_declarations = file_scoped:warning
csharp_using_directive_placement = inside_namespace:warning
csharp_new_line_before_open_brace = all
dotnet_style_qualification_for_field = true:warning
dotnet_style_qualification_for_property = true:warning
dotnet_style_qualification_for_method = true:warning
dotnet_style_qualification_for_event = true:warning

[*.csproj]
indent_style = tab
```

No fijar aquí una codificación o EOL C# como hecho observado. Medirla y resolverla en la adopción del perfil. Para Markdown dentro de Gateway conservar su regla existente; para este plan independiente se usa UTF-8. Validar las opciones contra el SDK/analyzers fijados, sin activar todas las advertencias como errores antes de conocer la deuda.

## 6. Comentarios y documentación XML

**O:** cabecera `<copyright ... company="PlaceholderCompany">` y XML `summary`, `param`, `returns`, `typeparam`. Los proyectos de la muestra generan documentación XML e incluyen StyleCop.Analyzers 1.1.118 y `stylecop.json`.

**P:** conservar XML en tipos y contratos públicos, constructores y miembros requeridos por el perfil. Describir significado, unidades, nulabilidad, errores y efectos; evitar `Task.` o `Result.` como documentación suficiente de un contrato nuevo.

`PlaceholderCompany` aparece como configuración real, pero no es el nombre corporativo confirmado. El perfil tendrá `company_name` configurable; no inventar un titular ni sustituir todas las cabeceras automáticamente.

La muestra contiene descripciones copiadas: `BookEndpoints` dice «User endpoints» y `BookGetQueries` dice «User Create Queries». Son incidencias a evitar, no convenciones que se deban reproducir. La validez de XML se comprueba automáticamente; la correspondencia semántica del comentario requiere revisión.

## 7. Validación, persistencia y errores

**O:** FluentValidation reside junto al Request; UseCases llama al validador y lanza `RequestValidationException`. Reglas de negocio producen excepciones compartidas y constantes de error. Commands usa contexto de escritura y Queries contexto de lectura; la muestra usa `AsNoTracking`, especificaciones y `DataContextGuards.SaveChanges`.

**P:** validar forma y valores del request en el validador, reglas de negocio en el caso de uso y restricciones de integridad en la base cuando sean necesarias. Una consulta previa no garantiza unicidad concurrente. Diseñar el tratamiento de colisiones y la transacción antes de escribir código.

No forzar `AsSplitQuery` en todas las consultas ni copiar una opción sin analizar su finalidad. Propagar CancellationToken donde los contratos lo admitan; ampliar contratos comunes requiere impacto y ADR si afecta consumidores. Evitar `.Result`, `.Wait()` y excepciones genéricas sin semántica.

**O:** se usan `Id` interno, `PublicId` GUID, `Guid.CreateVersion7()` y fechas UTC en servicios examinados. **P:** conservar el patrón donde aplique y permitir reloj inyectable para nuevos comportamientos dependientes del tiempo; es una mejora, no una convención ya verificada en todo el repo.

## 8. Logging, eventos y seguridad

**O:** `IAppLogger<T>`, mensajes `IN ->` y `OUT <-`, y handlers de eventos. **P:** mantener compatibilidad, usar propiedades estructuradas y no registrar credenciales, tokens, hashes sensibles ni contenido personal innecesario. Evitar logs de entrada/salida redundantes cuando no aporten observabilidad.

Un servicio inspeccionado dispara trabajo con `Task.Run` sin esperar en el flujo principal. No convertirlo en patrón de entrega garantizada. Si un requisito exige entrega fiable de eventos, diseñar outbox/cola y pruebas específicas. Tampoco copiar atributos de autorización comentados como política de acceso de endpoints nuevos: la SPEC debe declarar autenticación, autorización y aislamiento de usuario/tenant aplicables.

## 9. TDD y pruebas: estándar propuesto

No se encontraron proyectos con `IsTestProject` ni referencias a xUnit, NUnit, MSTest o Microsoft.NET.Test.Sdk en los `.csproj` inspeccionados. Esto justifica crear infraestructura de pruebas antes del primer ciclo TDD; no demuestra ausencia absoluta de pruebas externas.

```text
tests/
  DevelApi.<Aplicacion>.UnitTests/
  DevelApi.<Aplicacion>.IntegrationTests/
  DevelApi.<Aplicacion>.ArchitectureTests/
```

**P:** elegir xUnit para el piloto si no hay otro framework establecido, fijando versiones y validando .NET 10. Nombres `Method_Condition_ExpectedBehavior`, patrón Arrange/Act/Assert, casos deterministas, datos sintéticos y sin dependencia del orden. Unit tests de validadores/casos de uso; integración para EF, HTTP, DI, autenticación y transacciones según impacto.

Gherkin puede mapearse a tests C# mediante IDs, sin framework BDD en la primera entrega. Si se quiere ejecutar `.feature` directamente, seleccionar runner compatible en un ADR y probarlo. No llamar «escenario ejecutado» a un archivo `.feature` que nadie ejecuta.

## 10. Enforcement progresivo

1. Medir compilación, warnings, formato y estructura existentes.
2. Fijar SDK en `global.json`, paquetes, perfil y comandos; son incorporaciones propuestas, no archivos encontrados.
3. Aplicar reglas al código nuevo/tocado, con baseline explícito para deuda previa.
4. Añadir tests de arquitectura y trazabilidad.
5. Elevar severidad por grupos de reglas al estabilizar la base.

Comandos de calidad previstos, adaptados al runner y solución reales:

```text
dotnet restore <solucion>
dotnet build <solucion> --no-restore
dotnet test <proyecto-o-solucion-de-tests> --no-build
dotnet format <solucion> --verify-no-changes --no-restore
```

El runner debe comprobar que se descubrieron y ejecutaron tests; cero tests no es éxito. `--no-build` exige haber construido esos proyectos con la misma configuración. `dotnet format --verify-no-changes` verifica sin aplicar formato; el arreglo se realiza en develop. [Documentación de dotnet format](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-format).

## 11. Perfil compartido

Publicar `dotnet-develapi@0.1.0` con reglas, ejemplos de referencia, validadores y variantes Gateway/Library. Cada regla tendrá ID, origen O/P, severidad, scope, detector y excepción permitida. Ejemplos: `DOTNET-LAYOUT-001`, `DOTNET-NAMING-001`, `DOTNET-DOC-001`, `DOTNET-TDD-001`.

Una SPEC fija el hash del perfil al iniciarse. Cambiarlo requiere migración consciente, sin alterar silenciosamente la definición de terminado a mitad de una implementación.
