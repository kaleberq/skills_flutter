---
name: mvvm-architecture
description: >-
  Aplica MVVM com Riverpod 2 NotifierProvider em qualquer tela Flutter do app:
  screen smart vs components burros, State imutável, AsyncValue.guard,
  reporte de erros e tipagem explícita. Use em qualquer mudança de screen,
  ViewModel ou state. Telas novas completas: skill screen-mvvm-architecture prevalece.
alwaysApply: true
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
- **Não** depende de outro ViewModel (não injeta, não `ref.read` de outro `*ViewModel`). Cada um é dono do próprio estado. Dado ou lógica compartilhada → **serviço / use case** no domain (skills `domain-layer`, `use-cases`)
- Reportar falhas async e estados inválidos no serviço de crash da app (skill `domain-exceptions`)

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

## Sem `Widget` em funções

Não criar `Widget _buildX(...)` / métodos que **retornam Widget**. Extrair para um `StatelessWidget` / component em `components/`. Reuso, leitura e teste ficam no widget, não em helpers da screen.

## Nulidade — sem `!`

Não usar o operador `!` em propriedades que podem ser nulas (`if (widget.value!)`, `data!.count`). Atribuir a uma variável local e validar:

```dart
final ExampleSectionDataModel? data = widget.data;

if (data != null) {
  // usa data com promoção de tipo
}
```

Evita *null check operator* em runtime e deixa o fluxo explícito.

## Config remota — models de tela

Só quando necessário. Dados de apresentação, defaults no constructor, load no `initState` da screen. Telas novas: labels no `ModelsBuilder` via `ICopySource`, não `context.l10n` — skills `screen-mvvm-architecture` e `l10n`. Testes de VM/helpers: skill `unit-testing`. Semantics/shimmer: skill `accessibility`.

## Dispose

Controllers, `FocusNode` e `TextEditingController` na **screen**: criar no `State` e **não esquecer `dispose`**. Timers no ViewModel: `ref.onDispose`.
