---
name: unit-testing
description: >-
  Testes de unidade de ViewModel, helpers, domain e repository com fakes
  (implements IItemRepository), ProviderContainer + overrides e Mockito em
  EventProvider. Use ao criar *_view_model_test, helper_test ou teste de
  repository. Não é teste de widget (skill widget-testing).
alwaysApply: true
---

# Unit testing

Widgets, keys, `pumpWidget`: skill `widget-testing`. Tela MVVM: skill `screen-mvvm-architecture`. Helpers: skill `screen-helpers`. Domain: skill `domain-layer`. HTTP fake: skill `error-mapping`.

## O que entra aqui

| Sujeito | Fake / mock |
| ------- | ----------- |
| ViewModel | `ProviderContainer` + `overrides`; fake de `IItemRepository` / use case / `ICopySource` |
| DataLoader, ModelsBuilder, Mapper, Filter | Construtor com fakes (`FakeCopySource`); sem `pumpWidget` |
| Domain / use case | `implements IItemRepository` |
| Repository | `FakeApiClient` / fake de datasource — exceptions de **domain** |
| AnalyticsTracker | Mockito `MockEventProvider` + `verify` |

Não usar `tester.pump` / `find.byKey` neste skill.

## Espelho de pastas

```text
lib/presentation/features/{feature}/screens/{screen}/
    ↔
test/{feature}/...

lib/domain/...
    ↔
test/{feature}/domain/...   # ou test/unit/domain/ — seguir o app
```

Um `{arquivo}_test.dart` por unidade com lógica.

## Fake de repository

```dart
class FakeItemRepository implements IItemRepository {
  FakeItemRepository({this.items = const []});

  final List<ItemModel> items;

  @override
  Future<List<ItemModel>> getItems() async => items;
}

class FakeCopySource implements ICopySource {
  @override
  String text(CopyKey key) => key.name;
}
```

Tipagem **explícita** também no teste (`final List<ItemModel> items = ...`).

## ViewModel — ProviderContainer

```dart
final ProviderContainer container = ProviderContainer(
  overrides: [
    itemRepositoryProvider.overrideWithValue(FakeItemRepository(items: <ItemModel>[item])),
  ],
);
addTearDown(container.dispose);

final ItemDetailState state = container.read(itemDetailViewModelProvider(itemId));
```

Não instanciar `ItemRepository` real nem `Dio` no teste de VM.

## EventProvider

```dart
final MockEventProvider eventProvider = MockEventProvider();
verify(
  eventProvider.trackAction(
    eventId: EventTypeEnum.itemDetailConfirm,
    eventName: 'item_detail_confirm',
    params: {'itemId': itemId},
  ),
).called(1);
```

Tracker é classe plain Dart — teste **unit** do tracker, não da screen.

## Anti-patterns

- `flutter_test` binding / `MaterialApp` no teste de ViewModel
- `implements ItemRepository` (classe de data) em vez de `IItemRepository`
- `find.text` / `find.byKey` (isso é widget test)
- `final x = container.read(...)` sem tipo
- Fake que lança `DioException`
