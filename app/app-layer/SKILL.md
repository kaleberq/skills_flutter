---
name: app-layer
description: >-
  Composition root em lib/app/: wiring Riverpod (providers tipados em
  interfaces), agregação de rotas, tema e bootstrap. Use ao criar ou alterar
  providers globais, GoRouter do app, MaterialApp ou composição de
  implementações de data.
---

# App layer

Local: `lib/app/`. Domain: skill `domain-layer`. Implementações: skill `data-layer`. Telas e rotas de tela: skills em `presentation/`.

`lib/app` **compõe**. Não contém regra de negócio, DTO, fetch nem widgets de feature.

```text
lib/
├── main.dart                 # só bootstrap (binding, ProviderScope, runApp)
└── app/
    ├── app.dart              # MaterialApp.router (tema, l10n, router)
    ├── providers/            # Provider<I*> → impl em data
    ├── router/               # GoRouter + lista de rotas
    └── theme/                # opcional: ThemeData / cores de app (não widgets de tela)
```

## Regras

- Providers globais tipados na **interface** (`Provider<IItemRepository>`), nunca na classe concreta.
- Construção das impls: `ItemRepository(...)` importado de `lib/data` **só aqui** (e testes de wiring).
- Presentation **não** instancia repository/datasource/HTTP.
- Domain **não** importa `lib/app`.
- Rotas **de tela** ficam junto da screen (`*_route.dart`). `lib/app/router` só **agrega** e cria o `GoRouter`.
- `main.dart` não monta grafo de dependências — delega a `lib/app`.

## Wiring (Riverpod)

```dart
final apiClientProvider = Provider<IApiClient>((ref) => ApiClient());

final itemRemoteDataSourceProvider = Provider<ItemRemoteDataSource>((ref) {
  return ItemRemoteDataSource(apiClient: ref.watch(apiClientProvider));
});

final itemRepositoryProvider = Provider<IItemRepository>((ref) {
  return ItemRepository(remote: ref.watch(itemRemoteDataSourceProvider));
});
```

Infra de longa vida (`keepAlive` / sem `autoDispose`) só para clients, storage e session — não para ViewModel de tela.

## Router

```dart
List<RouteBase> get appRoutes => [
  // rotas geradas / TypedGoRoute das features
];

final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(routes: appRoutes);
});
```

Page view e `GoRouteData.build`: skill `presentation-ui`.

## Bootstrap

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ProviderScope(child: App()));
}

class App extends ConsumerWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final GoRouter router = ref.watch(routerProvider);
    return MaterialApp.router(
      theme: appTheme,
      routerConfig: router,
    );
  }
}
```

Tema: tokens do design system ou `lib/app/theme`. Sem screen, card ou bottom sheet aqui.

## Anti-patterns

- `Provider<ItemRepository>` (impl) em vez de `Provider<IItemRepository>`
- `Dio` / Chopper / DTO em `lib/app`
- Widget de feature em `lib/app`
- ViewModel / `NotifierProvider` de tela em `lib/app` (fica na pasta da screen)
- Import de `lib/app` em `lib/domain` ou em components burros
