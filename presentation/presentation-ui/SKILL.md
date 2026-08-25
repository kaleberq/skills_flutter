---
name: presentation-ui
description: >-
  Implementa navegação go_router (typed routes + page view), analytics
  (EventProvider, EventTypeEnum, ações vs page view) e layout (scaffold do DS,
  Slivers, AsyncValue.when, design system). Use ao criar rotas, disparar
  eventos, ou montar layout de screens.
---

# Presentation & UI

Navegação, analytics e layout. Telas novas: skill `screen-mvvm-architecture`. Agregar rotas e `GoRouter`: skill `app-layer`. Tokens e widgets do DS: skill `design-system`.

## Navigation

`go_router`: respeitar a versão pinada no `pubspec`; não upgradar sem alinhamento.

### Typed routes (preferido em telas novas)

`TypedGoRoute` + `GoRouteData`. Se a tela está sob árvore string-based, typed route não se aplica — usar legado só para navegar até a rota existente.

**Route class** (`*_route.dart` ao lado da screen):

```dart
@immutable
class ExampleDetailRoute extends GoRouteData {
  final int itemId;
  const ExampleDetailRoute({required this.itemId});

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return Consumer(
      builder: (context, ref, child) {
        ref.read(eventProvider).trackPageView(
          journey: ExampleJourney.item,
        );
        return ExampleDetailsScreen(itemId: itemId);
      },
    );
  }
}
```

**Registro** no agregador em `lib/app/router/` (ex.: `{feature}_routes.dart`):

```dart
const List<TypedGoRoute> exampleTypeSafeRoutes = [
  TypedGoRoute<ExampleDetailRoute>(path: 'item/:itemId'),
];
```

Importar o route file no agregador de rotas — necessário para codegen mesmo se não for óbvio. Rodar build_runner no arquivo gerado.

**Navegação:**

```dart
ExampleDetailRoute(itemId: 42).push(context);
ExampleDetailRoute(itemId: 42).go(context);
```

Regras:

- `build` da route envolve a screen em `Consumer`
- Page view dispara **dentro** de `build` / `builder`
- Preferir route **co-locada** com a screen; journey compartilhada é OK quando a árvore já existe

### Legacy routes (somente screens existentes)

Não usar em telas novas. Ao navegar para legado, preferir `pushNamed`/`goNamed`:

```dart
context.pushNamed(
  ExampleJourney.item.name,
  pathParameters: {'id': itemId},
);
```

## Analytics

Tudo passa por `EventProvider` (`ref.read(eventProvider)`). Destinos de analytics ficam encapsulados no provider — a tela não chama SDKs direto.

### Novos eventos

Entrada no enum de eventos da app (ex.: `lib/presentation/common/events/event_type_enum.dart`). Nome curto e estável.

### Page view — na route

```dart
final EventProvider analytics = ref.read(eventProvider);
analytics.trackPageView(journey: ExampleJourney.list);
```

### Ações do usuário — na screen (tracker)

Em telas MVVM novas: classe `*AnalyticsTracker` plain Dart com `EventProvider`, instanciada na **screen**. Não no ViewModel.

```dart
ref.read(eventProvider).trackAction(
  eventId: EventTypeEnum.exampleItemTap,
  eventName: 'example_item_tap',
  params: {'itemId': itemId},
);
```

`eventName` deve bater com o nome do `EventTypeEnum`.

**Padrão:** page view na route, ações no tracker da screen — não no ViewModel.

## Layout

Contrato de tokens/widgets: skill `design-system`.

- Scaffold do **design system** (ou wrapper do app) no lugar de `Scaffold` cru
- `CustomScrollView` + Slivers em listas complexas
- `Flexible` / `Expanded` em vez de tamanhos fixos
- Layouts que wrapam texto
- Async UI com `AsyncValue.when`
- Widgets do DS antes de UI customizada
