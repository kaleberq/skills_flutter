---
name: index-barrels
description: >-
  Exporta pastas de tela via index.dart (components, models, helpers,
  enums/keys) sem vazar data nem criar ciclo. Use ao criar pasta de screen
  MVVM, adicionar arquivo nessas pastas, ou ajustar barrels em presentation.
---

# Index barrels

Tela MVVM: skill `screen-mvvm-architecture`. Composição do app: skill `app-layer`. Domain/data **não** precisam de barrel para a presentation importar contratos — presentation importa **interfaces e models de domain**.

## Onde

Cada pasta da screen exporta via `index.dart`:

```text
{screen}/
├── components/index.dart
├── models/index.dart
├── helpers/index.dart
└── enums/keys/index.dart
```

A screen / ViewModel importam o barrel da pasta, não dezenas de arquivos soltos (exceto o próprio `{screen}_screen.dart` / `_view_model.dart`).

```dart
export 'item_header.dart';
export 'item_row.dart';
```

Arquivo novo na pasta → **uma linha** no `index.dart`.

## O que não exportar

| OK no barrel de presentation | Não |
| --------------------------- | --- |
| Components, UI models, helpers, keys da **própria** screen | `ItemRepository`, `ItemDto`, `ApiClient`, Chopper/Dio |
| Enums `IKeyEnum` da tela | Implementação em `lib/data` |
| Re-export de model de **domain** só se a pasta de models da tela realmente o usa como API pública da pasta | Barrel de `presentation` reexportando `lib/data` |

Presentation não importa data. Barrel de `components/` não puxa repository.

## Ciclos

- `a/index.dart` não exporta `b/` se `b/index.dart` exporta `a/`
- Helper não importa o barrel de `components/` se o component importa o barrel de `helpers/`
- Preferir import do **arquivo** quando o barrel criaria ciclo

Domain e data: barrels opcionais **dentro** da camada; não usar barrel de presentation em `lib/domain`.

## Anti-patterns

- `export '../../data/item_repository.dart';` em `presentation/.../index.dart`
- Barrel da feature inteira misturando screens, data e domain
- `index.dart` que exporta implementação HTTP
- Esquecer o export e importar path relativo inconsistente na mesma pasta
