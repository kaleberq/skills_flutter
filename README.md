# skills_flutter

Skills de Cursor para apps Flutter em **Clean Architecture + MVVM** (Riverpod). São genéricas: servem qualquer app com a mesma estrutura de pastas.

## Rules vs skills

| | **Rules** (`.cursor/rules/*.mdc`) | **Skills** (`SKILL.md` em pastas) |
|---|-----------------------------------|-------------------------------------|
| **O quê** | Convenções curtas, sempre válidas | Playbooks longos por tarefa/camada |
| **Quando entram** | `alwaysApply: true` ou `globs` por pasta | Agente descobre pela `description` ou você invoca |
| **Neste repo** | 4 rules em `.cursor/rules/` | 20 skills nas pastas abaixo |

**Instalação no app Flutter:**

1. Copiar `.cursor/rules/` → `.cursor/rules/` do projeto (convenções persistentes).
2. Copiar cada pasta de skill → `.cursor/skills/` do projeto (workflows sob demanda).

Cada skill é uma pasta com um `SKILL.md` (`name` + `description` no frontmatter).

## Camadas (`lib/`)

```text
lib/
├── main.dart
├── app/            # composition root
├── domain/         # contratos e models
├── data/           # impls, HTTP, DTO
├── extensions/     # sugar de tipos
└── presentation/   # telas MVVM
```

As skills seguem essa árvore.

## Índice

### app/

| Skill | Quando usar |
| ----- | ----------- |
| [app-layer](app/app-layer/SKILL.md) | Providers globais, `GoRouter`, `MaterialApp`, wiring `I*` |
| [flutter-workflow](app/flutter-workflow/SKILL.md) | `flutter test` / `analyze` / `format`; flavor só o do app |
| [codegen](app/codegen/SKILL.md) | `build_runner`, não editar `*.g.dart` |
| [index-barrels](app/index-barrels/SKILL.md) | `index.dart` em components/models/helpers/keys |

### domain/

| Skill | Quando usar |
| ----- | ----------- |
| [domain-layer](domain/domain-layer/SKILL.md) | Models, enums, `IRepository` / `IService` — sem Flutter |
| [use-cases](domain/use-cases/SKILL.md) | Lógica compartilhada; VM não depende de outro VM |
| [exceptions](domain/exceptions/SKILL.md) | Exceptions de domain; mensagem via copy, não HTTP cru |

### data/

| Skill | Quando usar |
| ----- | ----------- |
| [data-layer](data/data-layer/SKILL.md) | DTO, repository, `IApiClient` (Chopper/Dio só no adapter) |
| [error-mapping](data/error-mapping/SKILL.md) | Rede/HTTP/parse → exception de domain no adapter |

### extensions/

| Skill | Quando usar |
| ----- | ----------- |
| [dart-extensions](extensions/dart-extensions/SKILL.md) | Extension em `String` / `DateTime` / enum — sem regra de feature |

### presentation/

| Skill | Quando usar |
| ----- | ----------- |
| [mvvm-architecture](presentation/mvvm-architecture/SKILL.md) | Screen smart vs component burro; state; sem `!`; sem VM→VM |
| [screen-mvvm-architecture](presentation/screen-mvvm-architecture/SKILL.md) | Tela nova completa (pastas, checklist, TDD) |
| [screen-helpers](presentation/screen-helpers/SKILL.md) | DataLoader, ModelsBuilder, tracker, calculator… |
| [ui-components](presentation/ui-components/SKILL.md) | `*TextsModel` + `*DataModel?`; `data == null` → shimmer |
| [design-system](presentation/design-system/SKILL.md) | Reusar o DS **do projeto**; tokens; closest match no Figma |
| [presentation-ui](presentation/presentation-ui/SKILL.md) | go_router, analytics (`EventProvider`), layout |
| [l10n](presentation/l10n/SKILL.md) | Copy via `ICopySource`; tela/VM sem `AppLocalizations` |
| [widget-testing](presentation/widget-testing/SKILL.md) | KeysEnum; sem `find.text` de copy na screen |
| [unit-testing](presentation/unit-testing/SKILL.md) | VM/helpers/repo com fakes; não é widget test |
| [accessibility](presentation/accessibility/SKILL.md) | Semantics, contraste por tokens, shimmer com label |

## Rules (`.cursor/rules/`)

| Rule | Escopo |
| ---- | ------ |
| [flutter-architecture](.cursor/rules/flutter-architecture.mdc) | Sempre — camadas, MVVM, exemplos genéricos |
| [flutter-security](.cursor/rules/flutter-security.mdc) | Sempre — credenciais, `.env`, dados sensíveis |
| [flutter-presentation](.cursor/rules/flutter-presentation.mdc) | `lib/presentation/**` — copy, components, analytics, DS |
| [flutter-data-domain](.cursor/rules/flutter-data-domain.mdc) | `lib/{data,domain}/**` — HTTP port, erros, use cases |

As convenções estão nas rules; skills detalham o *como* implementar.

## Relação rápida

```text
ICopySource / I*Repository / I*UseCase
        → DataLoader / ModelsBuilder
        → ViewModel → State
        → Screen → Components (DS + texts + data?)
```
