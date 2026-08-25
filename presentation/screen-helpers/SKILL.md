---
name: screen-helpers
description: >-
  Extrai lógica de tela MVVM para helpers em helpers/: DataLoader,
  ModelsBuilder, Mapper, Filter, Calculator, FormValidation e
  AnalyticsTracker. Use ao criar ou refatorar helpers de screen, quando o
  ViewModel busca API, monta copy/labels, mapeia cards, calcula, valida
  formulário ou dispara analytics de ação.
---

# Helpers de tela MVVM

Tela: skill `screen-mvvm-architecture`. Testes de helper/VM: skill `unit-testing`. Helpers são **plain Dart** (sem Riverpod, sem widgets). ViewModel **orquestra**; screen só conecta callbacks.

```text
{screen}/helpers/
├── index.dart
├── {screen}_data_loader.dart
├── {screen}_models_builder.dart
├── {screen}_mapper.dart              # opcional
├── {screen}_filter.dart              # opcional
├── {screen}_calculator.dart          # opcional
├── {screen}_form_validation.dart     # opcional
└── {screen}_analytics_tracker.dart
```

Exportar tudo em `helpers/index.dart`. Um `{helper}_test.dart` por arquivo com lógica.

## Quando criar

| Helper | Criar quando | Não fazer |
| ------ | ------------ | --------- |
| **DataLoader** | Fetch via `I*Repository` | Mutar state; `ref.*`; widgets |
| **ModelsBuilder** | Copy → `*TextsModel`; payload de request → `*DataModel` | Widgets; `find.text` de label na screen |
| **Mapper** | Domain → `*CardModel` / row (listas) | State; Riverpod |
| **Filter** | Busca, filtro, ordenação pura | I/O |
| **Calculator** | Contas de formulário / totais | Side effects |
| **FormValidation** | Validators reutilizáveis | UI |
| **AnalyticsTracker** | Tap, submit, toggle | Page view (fica na route); Riverpod na classe |

Não criar helper “utils” genérico. Se a lógica só existe numa tela, o helper é **dessa** tela.

## Regras

- Tipagem explícita.
- Deps no **construtor** (repository, `ICopySource`, config remota, mappers).
- ViewModel instancia DataLoader / ModelsBuilder / Mapper no `build()` do Notifier e injeta deps com `ref.read`.
- **AnalyticsTracker** instancia na **screen** com `EventProvider` — não no ViewModel.
- Sem `material.dart`. Sem import de DS (tokens só na screen/components). Sem `AppLocalizations`.
- Copy: skill `l10n` (`ICopySource`). Stub por **CopyKey** nos testes do ModelsBuilder; screen não usa `find.text` de copy.

## Tela de detalhe de entidade (padrão)

Tela que busca um registro por id, mostra ficha e confirma seleção — o ViewModel **não** chama repository direto nem monta título/subtitle.

1. **DataLoader** — fetch → `AsyncValue<ItemModel>`
2. **ModelsBuilder** — títulos/copy a partir de regra de feature ou config remota (nunca string literal no VM)
3. **State** — `AsyncValue<ItemModel> entity`
4. **ViewModel** — `loading` → loader → `copyWith`; getter `isConfirmEnabled` (ex.: não confirmar enquanto `entity.isLoading`)
5. **Screen** — bootstrap `postFrameCallback`; `entity.when(data/error/loading)`; confirmar: tracker → notifier da jornada → `pop`
6. **AnalyticsTracker** — evento de confirmar / cancelar na screen

```dart
class ItemDetailDataLoader {
  final IItemRepository _repository;

  const ItemDetailDataLoader({required IItemRepository repository})
      : _repository = repository;

  Future<AsyncValue<ItemModel>> fetchById({
    required int id,
    required String apiUrl,
    required int scopeId,
  }) {
    return AsyncValue.guard(
      () => _repository.getById(id: id, url: apiUrl, scopeId: scopeId),
    );
  }
}

class ItemDetailModelsBuilder {
  final ICopySource _copy;
  final RuleChecker _rules;
  final int _scopeId;

  const ItemDetailModelsBuilder({
    required ICopySource copy,
    required RuleChecker rules,
    required int scopeId,
  }) : _copy = copy,
       _rules = rules,
       _scopeId = scopeId;

  ItemDetailHeaderModel buildHeader() {
    final bool useAlternateCopy = _rules.hasRule(
      rule: FeatureRule.showAlternateHeader,
      scopeId: _scopeId,
    );
    return ItemDetailHeaderModel(
      title: _copy.text(
        useAlternateCopy ? CopyKey.itemDetailTitleAlt : CopyKey.itemDetailTitle,
      ),
      subtitle: _copy.text(
        useAlternateCopy
            ? CopyKey.itemDetailSubtitleAlt
            : CopyKey.itemDetailSubtitle,
      ),
    );
  }
}
```

Títulos vêm de `ICopySource` (ARB, config remota ou API por trás da interface) — não hardcoded no ViewModel nem no component. Screen **não** chama `l10n`.

ViewModel:

```dart
Future<void> load({required int id, required String apiUrl, required int scopeId}) async {
  state = state.copyWith(entity: const AsyncValue.loading());
  final AsyncValue<ItemModel> result = await _dataLoader.fetchById(
    id: id,
    apiUrl: apiUrl,
    scopeId: scopeId,
  );
  state = state.copyWith(entity: result);
}

bool get isConfirmEnabled => !state.entity.isLoading;
```

## DataLoader

```dart
Future<AsyncValue<T>> fetchX(...) => AsyncValue.guard(() => _repository.getX(...));
```

`try/catch` → `AsyncValue.error` só se houver transformação antes do return.

## ModelsBuilder

- Resolve copy via **`ICopySource`** (skill `l10n`). Config remota/API só na **impl** de `ICopySource`, não no builder.
- Devolve `*TextsModel` / `*DataModel` imutáveis para a screen passar ao component (`data` pode ser `null`).
- Regras de feature / feature flag aqui, não no widget.

## Mapper / Filter / Calculator / Validation

Funções ou classe com métodos estáticos/puros quando não há deps. Com config remota, instanciar como o ModelsBuilder.

## AnalyticsTracker

```dart
class ItemDetailAnalyticsTracker {
  final EventProvider _eventProvider;

  const ItemDetailAnalyticsTracker({required EventProvider eventProvider})
      : _eventProvider = eventProvider;

  void trackConfirm({required int itemId}) {
    _eventProvider.trackAction(
      eventId: EventTypeEnum.itemDetailConfirm,
      eventName: 'item_detail_confirm',
      params: {'itemId': itemId},
    );
  }
}
```

Screen: `tracker.trackConfirm(...)` → `notifier.selectEntity(...)` → `context.pop()`.

## Anti-patterns

- `repository.getX()` no ViewModel ou na screen
- Título/subtitle/string de botão no ViewModel, na screen (`context.l10n`) ou no component
- Helper com `ref.watch` / `ConsumerWidget`
- Analytics no ViewModel
- Um `helpers.dart` global da feature
- DataLoader retornando `Widget` ou model de UI montado com TextStyle
