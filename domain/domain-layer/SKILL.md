---
name: domain-layer
description: >-
  Implementa a camada domain em Clean Architecture: models, enums, constants
  e interfaces (IRepository, IService, IChannel), sem Flutter e sem data.
  Use ao criar/alterar domain models, enums de negócio ou contratos de
  repository, service ou channel.
---

# Domain layer

Local: `lib/domain/`. Implementações: skill `data-layer`. UI models de tela: skill `screen-mvvm-architecture`.

```text
lib/domain/
├── constants/
├── enums/
├── interfaces/
│   ├── repositories/     # I{Feature}Repository
│   ├── services/         # I{Name}Service
│   └── channels/         # I{Name}Channel (plataforma)
└── models/
```

Em app com várias features, agrupar **dentro** dessas pastas (`models/{feature}/`, `interfaces/repositories/{feature}_repository_interface.dart`). Não criar repository concreto aqui.

## Regras

- **Sem Flutter.** Sem `material.dart`, widgets, `BuildContext`, cores, assets de UI.
- **Sem `lib/data`.** Domain não importa DTO, cliente HTTP, Hive, MethodChannel.
- **Sem `lib/presentation`.** Model de domínio ≠ model de tela.
- Contém só: regras de negócio, enums, constants, models, **contratos**.
- Interfaces descrevem o quê, não o como (sem JSON, socket, HTTP, path de arquivo).

## Interfaces

Toda implementação em `data` precisa de um contrato aqui. Prefixar com `I`.

| Tipo | Pasta | Nome |
| ---- | ----- | ---- |
| Repository | `interfaces/repositories/` | `IItemRepository` |
| Service | `interfaces/services/` | `INetworkService`, `IApiClient` (port HTTP) |
| Channel | `interfaces/channels/` | `IDeepLinkChannel` |

```dart
abstract class IItemRepository {
  Future<List<ItemModel>> getItems();
}

abstract interface class IImageEncoderService {
  Future<Uint8List?> encode(String payload);
}
```

- Um repository **por feature** — nunca `IAppRepository` genérico.
- Métodos retornam **domain models** (e tipos Dart/`typed_data`), nunca DTO.
- Presentation (ViewModel / DataLoader) depende da **interface**, não da classe em `data`.

## Models

- Plain Dart, `@immutable`, `const`, campos `final`
- **Sem** `@JsonSerializable` / `fromJson` / `toJson`
- Getters e helpers de negócio OK
- `copyWith` quando o state precisa atualizar o model
- **Não** `fromEntity` / `fromDto` no domain — mapping fica no **repository em data**

```dart
@immutable
class ItemModel {
  final int id;
  final String title;
  final int? relatedId;

  bool get hasRelated => relatedId != null;

  const ItemModel({
    required this.id,
    required this.title,
    this.relatedId,
  });
}
```

## Enums e constants

Enums de negócio e de protocolo (chaves de payload, tipos de mensagem) vivem no domain.

Constants de protocolo: `domain/constants/`.

Enum **não** carrega `Color`, `IconData` nem path de asset de UI.

## O que presentation pode importar

ViewModel / DataLoader: `IItemRepository`, `ItemModel`, enums de domain.

Não: `ItemRepository`, `ItemDto`, serviço HTTP concreto.

## Anti-patterns (quebra SOLID)

- Repository concreto em `lib/domain/` (**D**)
- `IAppRepository` genérico para várias features (**I**, **S**)
- Domain model importando `lib/data/dtos` ou `lib/presentation` (**D**, **S**)
- Interface que recebe/retorna DTO (**D**)
- `Provider<ItemRepository>` tipado na implementação em vez de `IItemRepository` (**D**)
- Enum de domain com `Color` / `IconData` / asset de UI (**S**)

## Dependências

```text
presentation  →  domain (interfaces + models)
data          →  domain (implements; mapeia DTO → model)
domain        →  Dart SDK apenas (typed_data ok)
```

ViewModels recebem `IItemRepository` no construtor. Testes: `class FakeItemRepository implements IItemRepository`.

Map DTO → model no repository em `lib/data`, não no domain:

```dart
ItemModel toModel(ItemDto dto) => ItemModel(
  id: dto.id,
  title: dto.name,
  relatedId: dto.relatedId,
);
```

Código novo: `IItemRepository` no domain, `ItemRepository implements IItemRepository` em data, `Provider<IItemRepository>(...)`. Migrar legado só quando a tarefa for migração explícita.
