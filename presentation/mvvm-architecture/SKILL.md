---
name: mvvm-architecture
description: >-
  Aplica MVVM com Riverpod 2 NotifierProvider em qualquer tela Flutter do app:
  screen smart vs components burros, State imutável, AsyncValue.guard,
  reporte de erros e tipagem explícita. Use em qualquer mudança de screen,
  ViewModel ou state. Telas novas completas: skill screen-mvvm-architecture prevalece.
---

# MVVM (base)

Todas as telas novas usam MVVM com Riverpod 2 `NotifierProvider`. Módulo completo de tela (helpers, DS, TDD): skill `screen-mvvm-architecture` — **prevalece** em conflito.

## Smart / dumb

**Screen (smart):** `ConsumerStatefulWidget` ou `ConsumerWidget`. Único UI com Riverpod.

- Lifecycle
- Watch/read do ViewModel
- Analytics de ação (em telas novas: via `*AnalyticsTracker`, não inline no VM)
- Delega renderização a components burros

**Components (dumb):** `StatelessWidget` ou `StatefulWidget` sem DI.

- Sem `ref.read` / `ref.watch`
- Dados via construtor: `*TextsModel` + `*DataModel?` + callbacks
- `data == null` → shimmer no component (skill `ui-components`)
- Só apresentação

Se precisa de provider, não é burro.

## Pastas por screen

```text
lib/presentation/features/{feature}/screens/{journey}/{screen_name}/
  {screen}_screen.dart
  {screen}_view_model.dart
  {screen}_state.dart
  {screen}_route.dart
  components/   # opcional
```

Telas novas: também `helpers/`, `models/`, `enums/keys/` — ver `screen-mvvm-architecture`.

## State

- `@immutable`, campos `final`, `const` constructor, `copyWith`
- `AsyncValue<T>` só para async; tipos planos para sync

## ViewModel

- `NotifierProvider.autoDispose` (sempre, salvo exceção explícita)
- `.family` quando há parâmetros
- Async: `AsyncValue.guard`
- Update só via `state = state.copyWith(...)`
- Sem UI / Flutter widgets; comunica só via state
- Reportar falhas async e estados inválidos no serviço de crash da app

```dart
} catch (e, st) {
  // Reportar o erro no serviço de crash da app.
  state = state.copyWith(errorMessage: 'User-facing error message');
}
```

## Tipagem explícita

Nunca `final x = ...` sem tipo. Vale para screens, VMs, state, models, repositories, mappers e testes.

```dart
final String name = user.name;
final List<ItemModel> items = response.data;
```

## Fluxo de dados

```text
Repository → ViewModel → State → Screen → Dumb Components
```

Dados não sobem. UI não muta domain models.

## Config remota — models de tela

Só quando necessário. Dados de apresentação, defaults no constructor, load no `initState` da screen. Telas novas: labels no `ModelsBuilder`, não na screen — `screen-mvvm-architecture`.
