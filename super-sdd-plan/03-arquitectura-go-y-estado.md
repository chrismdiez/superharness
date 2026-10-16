# 03. Arquitectura Go, estado y recuperación

## 1. Componentes

```mermaid
flowchart TD
    CLI[CLI y consola Super] --> Engine[Motor de workflow Go]
    Engine --> Contracts[Esquemas y validadores]
    Engine --> State[Diario SQLite y almacenamiento de artefactos]
    Engine --> Context[Constructor de contexto]
    Context --> Graph[Adaptador del grafo]
    Context --> Files[Fuentes y documentos versionados]
    Engine --> Providers[Adaptadores de agentes]
    Providers --> Codex[Codex CLI]
    Providers --> Claude[Claude Code CLI]
    Engine --> Runner[Runner aislado de build y tests]
    Engine --> Git[Git y worktrees]
    Engine --> Outbox[Operaciones remotas pendientes]
    Outbox --> Jira[Jira]
    Outbox --> GitHub[GitHub]
```

El núcleo de dominio solo conoce SPEC, fases, tareas, evidencias y transiciones. No conoce flags concretos de Codex, JSON de Jira ni llamadas de GitHub. Esa separación facilita probarlo sin gastar tokens ni conectar cuentas.

Estructura propuesta:

```text
cmd/super/main.go
internal/
  cli/                    # argumentos, consola, códigos de salida
  domain/                 # Spec, Task, Run, Evidence, Finding
  workflow/               # transiciones y ejecución de fases
  contracts/              # JSON Schema y validadores semánticos
  artifacts/              # manifiestos, hashes, publicación atómica
  state/                  # SQLite, locks, journal, migraciones
  context/                # selección y presupuesto de contexto
  graph/                  # interfaz y consultas normalizadas
  providers/codex/        # procesos, eventos y capacidades
  providers/claude/
  integrations/jira/
  integrations/github/
  git/                    # snapshots, ramas, worktrees
  runner/                 # procesos, permisos, logs, resultados
  policy/                 # scopes de escritura y gates
  telemetry/              # costes, duración y métricas
assets/
  schemas/
  templates/
  prompts/
  profiles/dotnet-develapi/
tests/
  fixtures/
  integration/
  evals/
```

Preferir biblioteca estándar para procesos, HTTP, JSON y hashing; seleccionar bibliotecas de CLI, SQLite y JSON Schema en una tarea dedicada y fijarlas en `go.mod`/`go.sum`. No se fija aquí una dependencia sin probar distribución Windows/Linux y compatibilidad. El binario será Go; el indexador externo, Codex, Claude y .NET siguen siendo dependencias operativas. «Hecho en Go» no implica que toda la cadena sea un único ejecutable sin runtimes.

## 2. Interfaces propuestas

Pseudocódigo Go: los tipos de dominio se completarán durante implementación.

```go
type AgentProvider interface {
    Capabilities(ctx context.Context) (Capabilities, error)
    Run(ctx context.Context, request RunRequest, sink EventSink) (RunResult, error)
}

type CodeGraph interface {
    Index(ctx context.Context, snapshot Snapshot) (GraphManifest, error)
    Query(ctx context.Context, query GraphQuery) (GraphResult, error)
    Resolve(ctx context.Context, ref CodeReference) (ResolvedReference, error)
}

type IssueTracker interface {
    Read(ctx context.Context, ref SourceRef) (SourceSnapshot, error)
    Apply(ctx context.Context, operation RemoteOperation) (RemoteReceipt, error)
}

type PullRequestHost interface {
    Ensure(ctx context.Context, request PullRequestRequest) (PullRequest, error)
    Inspect(ctx context.Context, id PullRequestID) (PullRequestStatus, error)
}
```

`RunRequest` contiene: fase, SPEC/revisión, modelo fijado, contexto renderizado, schema de salida, directorio aislado, capacidades permitidas, límites y rutas de staging. `RunResult` contiene: versión real del proveedor, ID de sesión, artefactos propuestos, preguntas, uso si está disponible y estado de terminación. Un exit code 0 no sustituye la validación de la respuesta.

La cancelación necesita terminar el árbol de procesos, no solo el proceso padre: estrategia específica para Windows y Unix, timeout, drenaje de stdout/stderr y registro de interrupción. Usar argumentos separados con `exec.CommandContext`; no construir comandos shell concatenando texto del ticket.

## 3. Quién es la fuente de verdad

| Información | Fuente de verdad | Copias/derivados |
|---|---|---|
| Código integrado | Git remoto y commit verificado | Checkout y grafo |
| Requisitos y decisiones aceptadas | Artefactos versionados y manifiesto de revisión | Paquete de contexto |
| Ticket en el momento del análisis | Snapshot de fuente con hash | `source.md` normalizado |
| Estado de ejecución y operaciones pendientes | Diario transaccional SQLite | `state.json`, consola y handoff |
| Estado remoto de PR/Jira | Consulta al servicio | Recibos locales con fecha |
| Relaciones de código | Código/compilación del snapshot | Base de grafo reconstruible |
| Texto generado de requisitos | JSON aceptado | Markdown renderizado |

Los Markdown que son vistas generadas llevan esa indicación. Los documentos editables, como la fuente y respuestas humanas, tienen un proceso de importación: parsear, validar y crear una revisión. No se mantienen JSON y Markdown editables de forma independiente sin reconciliarlos.

SQLite no se versiona ni se comparte en una unidad de red entre desarrolladores. Para pasar trabajo a otra máquina se exporta un bundle con manifiestos, estado, evidencias relevantes y hashes. El receptor verifica el bundle y el Git correspondiente antes de reconstruir su diario local. Un clone con solo Markdown puede mostrar planificación; no puede certificar tests que no están presentes o enlazados a una evidencia accesible de CI.

## 4. Almacenamiento

```text
.super/
  config.yaml                  # configuración del repo, sin secretos
  toolchain.lock.json           # versiones y hashes de plantillas/perfil
  runtime/                     # ignorado por Git
    state.db
    runs/<run-id>/
      input-manifest.json
      context.md
      events.jsonl
      result.json
      evidence/
      staging/
  cache/graph/                  # ignorado por Git, reconstruible
docs/sdd/
  project/                     # conocimiento aceptado del proyecto
  specs/SPEC-001/               # artefactos compartidos
```

No guardar prompts completos con datos sensibles en Git. Los logs privados tienen redacción, retención configurable y permisos de archivo restringidos. La evidencia pública contiene resultados y referencias necesarias, no tokens de acceso ni datos reales de usuarios.

Tablas mínimas: `specs`, `spec_revisions`, `phase_runs`, `task_runs`, `artifacts`, `evidence`, `approvals`, `events`, `locks`, `outbox`, `remote_receipts`. Restricciones únicas sobre `(repo_id, spec_id)`, `run_id`, clave de idempotencia y revisión del agregado.

## 5. Persistencia sin transacciones ficticias

Un commit SQLite y un rename de archivos no forman una única transacción. Protocolo propuesto:

1. Crear directorio de run y escribir resultados en staging, sin sustituir los aceptados.
2. Verificar schemas, hashes, referencias, scopes y gates.
3. Registrar en SQLite una publicación `PREPARED` con lista y hashes de todos los archivos.
4. Publicar un directorio inmutable `revisions/<revision-id>/` mediante rename en el mismo volumen. Verificar la estrategia en Windows; evitar sustituir archivos que otra herramienta tiene abiertos.
5. Marcar `COMMITTED` y actualizar puntero a revisión actual en SQLite.
6. Regenerar vistas legibles y `state.json` por escritura temporal y reemplazo. Son reconstruibles.

Al arrancar, buscar publicaciones preparadas. Si los archivos completos coinciden, terminar el commit; si no, mantener la última revisión válida y registrar recuperación. Nunca avanzar el estado solo porque un `.md` exista.

Para operaciones remotas se usa outbox: registrar intención, efectuar operación idempotente, guardar recibo y reconciliar. Si GitHub creó la PR pero la respuesta se perdió, buscarla antes de reintentar. No prometer una transacción distribuida entre Git, SQLite, Jira y GitHub.

## 6. Contexto limpio de verdad

Cada invocación hace lo siguiente:

```text
validar estado → congelar inputs → recuperar referencias → construir context.md
→ lanzar proceso/sesión nueva → validar output → guardar handoff y evidencias
```

No usar `resume`, `continue` ni un ID de sesión anterior para enlazar fases. Un `/clear` visual dentro de una aplicación no prueba aislamiento del ejecutor. La UI puede conservar mensajes; la garantía se aplica al proceso que realiza la fase. Los comandos de chat solo despachan parámetros y presentan el resultado.

El paquete contiene reglas aplicables, objetivo de fase, artefactos aceptados, preguntas resueltas, decisiones relevantes, referencias verificadas y evidencia resumida. No contiene el historial de razonamiento del modelo. `handoff.md` guarda hechos y decisiones, no razonamiento privado.

Las instrucciones de repositorio deben seguir aplicándose. Para reproducibilidad se registran y fijan; no se eluden políticas organizativas para conseguir una configuración idéntica. Cambios de configuración personal no registrados invalidan la etiqueta de ejecución reproducible.

## 7. Invalidación y concurrencia

Cada artefacto declara `inputs[]` con hash y versión. El motor mantiene un DAG de dependencias. Un cambio de requisito invalida descendientes; un cambio de estilo invalida comprobaciones de estilo; una actualización del grafo obliga a revalidar referencias afectadas, no necesariamente a reescribir toda la SPEC.

El bloqueo local se toma por `(repo, SPEC)` y el escritor de código por worktree. Contiene propietario, run, heartbeat y vencimiento; recuperar un lock vencido requiere comprobar que el proceso anterior no sigue activo. Actualizaciones de estado usan compare-and-swap sobre revisión.

Dos máquinas no quedan coordinadas por SQLite local. MVP: una rama/SPEC tiene un propietario y se rechazan pushes no fast-forward o divergencias. Evolución: coordinador remoto con leases y operaciones condicionales. Nunca usar force-push automático para resolver concurrencia.

## 8. Garantía de escritura exclusiva

Un prompt que diga «no edites» no es un control suficiente. Requirements, design y task trabajan con snapshot de código de solo lectura; el modelo devuelve JSON y el orquestador publica los documentos. Review ejecuta tests en checkout desechable con runner controlado: los tests pueden generar `bin/obj` y salidas temporales, pero ningún cambio del producto se publica.

Develop trabaja en checkout dedicado con rutas autorizadas. Init y close disponen solo de escritura documental y de configuración propia. Denegar escapes de ruta, symlinks/junctions fuera del scope y acceso de escritura a credenciales, remotos o worktrees ajenos. Si el proveedor no permite aplicar la política necesaria en el host, usar aislamiento externo o rechazar el modo; no fingir aislamiento.

Toda fase compara el árbol antes y después, incluyendo archivos no seguidos y archivos de build/configuración. Las evidencias y el runtime quedan fuera del hash del árbol del producto para evitar que escribir un log invalide su propia evidencia. Se guardan por separado `product_tree_hash`, `artifact_manifest_hash` y `full_commit_sha`.

## 9. Casos de recuperación obligatorios

| Incidente | Comportamiento |
|---|---|
| Cierre durante RED | Conservar test y logs; revalidar árbol y repetir RED al continuar |
| Límite de contexto | Checkpoint y nuevo proceso con tarea pendiente; no truncar requisitos esenciales |
| Fallo de proveedor | Marcar run fallido; no cambiar de modelo silenciosamente |
| Archivo editado manualmente | Detectar hash distinto e importar como revisión o bloquear |
| PR creada y respuesta perdida | Reconciliar por rama/base/marcador antes de crear otra |
| Jira inaccesible tras merge | `SYNC_PENDING`; no afirmar cierre remoto |
| Evidencias ausentes al clonar | Reejecutar validaciones o recuperar bundle/CI; no confiar en un checkbox |
| Cambio de HEAD tras review | Invalidar aprobación y checks asociados; volver a review |

Las migraciones de schemas y SQLite se versionan y se prueban con copia previa. Un binario antiguo debe rechazar un schema futuro que no entienda.
