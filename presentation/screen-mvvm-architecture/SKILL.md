---
name: screen-mvvm-architecture
description: >-
  Implementa telas MVVM em lib/presentation/features/: screen smart, view_model,
  state, helpers (data_loader, models_builder, analytics), models de UI, components
  burros, enums/keys,   design system (Figma MCP → tokens/componentes do DS) e TDD com testes
  obrigatórios. Use ao criar ou refatorar telas MVVM em qualquer feature do app.
---

# Arquitetura de Tela MVVM

Padrão para **telas novas** em `lib/presentation/features/`. Complementa `mvvm-architecture` e `presentation-ui`. Em conflito com regras genéricas de MVVM, **este padrão de tela prevalece**.

Design System: tokens e widgets do pacote DS do app. Helpers: skill `screen-helpers`. Components: skill `ui-components`. Testes de widget: skill `widget-testing`. VM/helpers/domain: skill `unit-testing`. Use cases: skill `use-cases`. Copy ARB: skill `l10n`. Semantics: skill `accessibility`. Barrels: skill `index-barrels`.

## Quick reference

```text
Repository → DataLoader → ViewModel → State → Screen → Components (burros)
```

- **Screen** — único lugar com Riverpod, analytics de ação, navegação
- **ViewModel** — orquestra helpers; sem Widget/analytics; **não** depende de outro ViewModel
- **Components** — `*TextsModel` + `*DataModel?` + callbacks; `data == null` → shimmer; zero `ref.*`
- **Testes** — obrigatórios; espelhar lib em `test/{feature}/view/{screen}/`

Jornada multi-tela: a **screen** pode `watch` mais de um provider. Um ViewModel **não** recebe nem lê outro ViewModel — lógica compartilhada vai para serviço / use case (skills `domain-layer`, `use-cases`).

## Estrutura de pastas

```text
lib/presentation/features/{feature}/screens/[{journey}/]{screen_name}/
├── {screen_name}_screen.dart
├── {screen_name}_view_model.dart
├── {screen_name}_state.dart
├── {screen_name}_route.dart          # preferir co-locado
├── {screen_name}_actions.dart        # opcional: part of screen
├── components/  models/  helpers/  enums/keys/
└── */index.dart
```

## Fluxo nova tela

1. `{screen}_screen`, `_view_model`, `_state`, `_route`
2. Helpers: data_loader, models_builder, analytics (+ mapper/filter/calculator se preciso)
3. Components burros + keys enum (`IKeyEnum`) + models
4. AnalyticsTracker (ações) + page view na route
5. Testes TDD por camada
6. `flutter test test/{feature}/view/{screen}/` verde

## Regras críticas

| Tópico | Regra |
| ------ | ----- |
| Tipagem | Explícita — nunca `final x = ...` sem tipo |
| Provider | `NotifierProvider.autoDispose` + `.family` quando há params de rota |
| State | `@immutable`, `final`, `const`, `copyWith`; async → `AsyncValue<T>` |
| ViewModel | Helpers no `build()`; `AsyncValue.guard`; reportar erros async; sem `material.dart`; sem depender de outro ViewModel |
| Screen | Tracker de ação; bootstrap `postFrameCallback`; `ref.listen` erros; controllers + dispose |
| Components | Burros — `texts` + `data?`; `data == null` → shimmer no component; sem Riverpod, config remota, analytics, navegação, regra de negócio |
| Analytics | Page view na **route**; ações no **tracker da screen** (não no ViewModel) |
| DS | Figma MCP → closest match nos tokens/componentes do DS; tokens só em `components/` e `build` da screen |
| Testes de copy | Stub por **key**; assert por **KeysEnum** — não `find.text` de label de config |
| Testes | Obrigatório: um `{component}_test.dart` por component |

## Analytics

Ações do usuário ficam no `*AnalyticsTracker` instanciado na **screen**. Page view continua na **route**. Não disparar analytics no ViewModel.

## Helpers — quando criar

Contratos e tela de detalhe (fetch + copy + confirmar): skill `screen-helpers`.

| Helper | Criar quando |
| ------ | ------------ |
| `*_data_loader.dart` | Fetch de API/repository |
| `*_models_builder.dart` | Config remota, regras de copy, models compostos |
| `*_mapper.dart` | Domain → card/row models (listas) |
| `*_filter.dart` | Filtro, busca, ordenação |
| `*_calculator.dart` | Cálculos numéricos |
| `*_form_validation.dart` | Validadores reutilizáveis |
| `*_analytics_tracker.dart` | Eventos de ação da tela |

## Checklist — nova tela

### Estrutura
- [ ] `{screen}_screen`, `{screen}_view_model`, `{screen}_state`, route registrada
- [ ] `helpers/`, `models/`, `components/`, `enums/keys/` com `index.dart`

### ViewModel
- [ ] `NotifierProvider.autoDispose.family`
- [ ] Helpers no `build()`; `ref.onDispose` para timers
- [ ] `AsyncValue.guard` para async; reportar erros
- [ ] Sem `material.dart`; tipagem explícita; não injeta outro ViewModel

### Screen
- [ ] AnalyticsTracker para ações; page view na route
- [ ] Bootstrap postFrameCallback; `ref.listen` erros; `ref.select` quando útil
- [ ] Controllers com dispose; components: `texts` + `data?` + callbacks

### Components
- [ ] Burros: `*TextsModel` + `*DataModel?`; shimmer se `data == null`; sem Riverpod/config remota/analytics/navegação/regra de negócio
- [ ] Keys enum (`IKeyEnum`); `{component}_test.dart` obrigatório

### Design System
- [ ] Figma MCP como referência; closest match no DS do app
- [ ] Tokens DS em components/ e build da screen
- [ ] Tamanhos de imagem podem ser literais

### Testes
- [ ] `flutter test test/{feature}/view/{screen_name}/` passando
- [ ] Teste por helper, component e models com lógica

## Anti-patterns

- Component smart (`ConsumerWidget`, `ref.*`, repository, analytics, config remota)
- Navegação (`push`/`go`) ou regra de negócio em component
- ViewModel retornando `Widget`, importando `material.dart`, disparando analytics ou dependendo de outro ViewModel
- Função / método `_buildX` que retorna `Widget` — extrair component
- Operador `!` em nullable (`widget.data!`) — usar variável local + `if (x != null)`
- Repository ou montagem de labels de config remota na screen
- Labels de config remota hardcoded no component
- Component com um único model misturando copy e payload de request
- Loading da request na screen (spinner) em vez de `data: null` + shimmer no component
- State mutável; page view na screen em vez da route
- Tela sem testes espelhados
- `find.text` com copy de config remota em screen/flow tests
- Widget legado smart em `components/` replicado em telas novas

## Provider e State

```dart
final exampleViewModelProvider = NotifierProvider.autoDispose
    .family<ExampleViewModel, ExampleState, int>(ExampleViewModel.new);

final exampleViewModelProvider = NotifierProvider.autoDispose
    .family<ExampleViewModel, ExampleState, ExampleFamily>(ExampleViewModel.new);
```

- Async → `AsyncValue<T>`; sync → tipos planos
- Helpers no `build()` do Notifier; update só via `state = state.copyWith(...)`
- `ref.onDispose` para timers

```dart
AsyncValue<Data> get screenReadyState {
  for (final AsyncValue<dynamic> dep in _asyncDependencies) {
    if (dep case AsyncError(:final error, :final stackTrace)) {
      return AsyncValue.error(error, stackTrace);
    }
  }
  if (_asyncDependencies.any((dep) => dep is AsyncLoading)) {
    return const AsyncValue.loading();
  }
  return primaryData;
}

State copyWith({Field? field, bool clearField = false}) => State(
  field: clearField ? null : (field ?? this.field),
);
```

## ViewModel e DataLoader

- Fetch: `copyWith(field: const AsyncValue.loading())` → loader → `copyWith(field: result)`
- Async: `AsyncValue.guard`; reportar erros (serviço de crash da app)
- Getters de UI model via `ModelsBuilder` — screen não monta strings de config remota
- DataLoader: plain Dart, deps no construtor, retorna `AsyncValue` / `Future<AsyncValue>`
- Config remota: ViewModel lê o provider no `build()` e injeta nos helpers; flags nunca no component
- Não injetar outro ViewModel; compartilhado → `I*Service` / use case

## Screen

1. Providers como getters (family / route args)
2. `AnalyticsTracker` com `EventProvider`
3. `initState` + `postFrameCallback` (bootstrap)
4. `ref.listen` erros; `ref.watch(...select(...))` quando útil
5. ViewModel getters → `*TextsModel` / `*DataModel?` → components burros
6. Callbacks: tracker → notifier → navegação
7. Sem `_buildX()` que retorna `Widget`; sem `!` em nullable

Controllers, `FocusNode` e `TextEditingController` na screen: **não esquecer `dispose`**. Handlers longos: `part '{screen}_actions.dart'`.

## Components

Contrato completo (dois models, shimmer): skill `ui-components`.

```dart
class ExampleSection extends StatelessWidget {
  final ExampleSectionTextsModel texts;
  final ExampleSectionDataModel? data;
  final VoidCallback onTap;
  const ExampleSection({
    super.key,
    required this.texts,
    this.data,
    required this.onTap,
  });
}
```

`data == null` = loading da request → shimmer do DS **dentro** do component. Screen só passa `null`. Sem `data!`. Promoção de tipo:

```dart
final ExampleSectionDataModel? data = this.data;
if (data != null) {
  // usa data
}
```

Keys: `IKeyEnum` em `enums/keys/` (incluir `shimmer`). StatefulWidget só para UI efêmera (debounce, accordion, animação). Não extrair UI em funções que retornam `Widget`.

## Testes (TDD)

```text
lib/presentation/features/{feature}/screens/{screen}/
    ↔
test/{feature}/view/{screen}/
```

Cobertura: state, view_model, screen, cada helper, cada component, tracker, models com lógica, flows críticos.

Ordem: helpers puros → ModelsBuilder/Mapper/DataLoader → ViewModel + tracker → components → screen/flow.

Config remota: stub por key; screen/flow assert com `find.byKey`, não `find.text` de label configurável.

```dart
Widget makeTestable({
  ExampleSectionTextsModel? texts,
  ExampleSectionDataModel? data,
}) => MaterialApp(
  home: Scaffold(
    body: ExampleSection(
      texts: texts ?? _defaultTexts,
      data: data,
      onTap: () => tapped = true,
    ),
  ),
);
```

ViewModel: `ProviderContainer` + overrides. Analytics: `verify` no `MockEventProvider`. Imagens: `mockNetworkImagesFor()`.

