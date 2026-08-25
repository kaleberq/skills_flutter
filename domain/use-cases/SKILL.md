---
name: use-cases
description: >-
  Extrai lógica compartilhada entre ViewModels para use case ou I*Service no
  domain (impl em data): um método público, models de domain, sem Flutter.
  Use quando dois VMs compartilhariam fetch/regra, ou ao criar IGetItemUseCase
  / GetItem. ViewModel não depende de outro ViewModel.
alwaysApply: true
---

# Use cases

Camada e contratos: skill `domain-layer`. Implementações HTTP/repo: skill `data-layer`. MVVM: skill `mvvm-architecture`.

Quando **dois ViewModels** precisariam da mesma regra ou do mesmo dado: extrair **use case** ou `I*Service` no domain (implementação em data). ViewModel **não** injeta nem `ref.read` de outro ViewModel.

## Quando use case vs I*Service vs repository

| Peça | Usar quando |
| ---- | ----------- |
| `I*Repository` | Persistência / fetch da feature (`getItems`, `getById`) |
| `I*Service` | Infra ou regra reutilizável que não é “o repository da feature” (rede, cache, encoder) |
| Use case | Uma **ação de negócio** orquestrada (um fluxo), reusada por mais de um VM ou para deixar o VM fino |

Não criar use case que só encaminha um método do repository sem orquestração — o VM fala com `IItemRepository`. Use case entra quando há composição (vários repos, regra + fetch) ou o mesmo fluxo em telas distintas.

## Contrato

- **Um método público** por use case (`call` ou nome da ação)
- **Sem Flutter** (`material.dart`, `BuildContext`, widgets)
- Input/output = **domain models** (e tipos Dart), nunca DTO
- VM depende da **interface**, não da classe em data

```dart
abstract interface class IGetItemUseCase {
  Future<ItemModel> call({required int id});
}

class GetItem implements IGetItemUseCase {
  final IItemRepository _repository;

  const GetItem({required IItemRepository repository})
      : _repository = repository;

  @override
  Future<ItemModel> call({required int id}) {
    return _repository.getById(id: id);
  }
}
```

Pasta típica: `lib/domain/use_cases/` (ou `interfaces/` + impl em data se o use case precisar de I/O extra). Se a classe só orquestra repositories já existentes e não faz I/O próprio, pode viver no domain **sem** importar `lib/data`.

Wiring: `Provider<IGetItemUseCase>` em `lib/app` — skill `app-layer`.

## ViewModel

```dart
class ItemDetailViewModel extends AutoDisposeFamilyNotifier<ItemDetailState, int> {
  @override
  ItemDetailState build(int id) {
    _getItem = ref.read(getItemUseCaseProvider);
    return const ItemDetailState();
  }
}
```

Testes: `class FakeGetItem implements IGetItemUseCase` — skill `unit-testing`.

## Anti-patterns

- ViewModel A com `ref.read(otherViewModelProvider)`
- Use case “gordo” que vira **segundo repository** (CRUD completo, cache, HTTP)
- Use case que retorna DTO ou importa Chopper/Dio
- Vários métodos públicos no mesmo use case (`get`, `save`, `delete`) — parta em use cases ou use o repository
- Flutter / `Widget` / `Color` no use case
