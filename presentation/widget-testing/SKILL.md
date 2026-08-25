---
name: widget-testing
description: >-
  Escreve testes de widget com KeysEnum (IKeyEnum) para visibilidade e tap,
  e stub de config remota por key — sem find.text de copy configurável em
  screen/flow tests. Use ao criar ou alterar *_test.dart de widgets, components,
  screens ou flows.
---

# Widget testing

Convenção: assert de show/hide e tap via **keys de enum**, não por texto ou ícone.

Telas MVVM: skill `screen-mvvm-architecture`. Components: skill `ui-components`. ViewModel / helper / repository **sem** `pumpWidget`: skill `unit-testing`. Copy ARB: skill `l10n` (screen test não usa `find.text` de l10n). Imagens do DS: fake em `FLUTTER_TEST` quando o pacote já fizer isso.

## Por quê

Keys são estáveis. Texto e ícone mudam com tradução, A/B e design.

## Keys enum

`IKeyEnum` em `lib/presentation/common/enums/key_enum_interface.dart` (ou pasta equivalente de UI compartilhada — não em `lib/data`).

```dart
enum ExampleSectionKeysEnum implements IKeyEnum {
  action(key: Key('example_section_action')),
  shimmer(key: Key('example_section_shimmer')),
  value(key: Key('example_section_value'));

  @override
  final Key key;

  const ExampleSectionKeysEnum({required this.key});
}
```

No widget:

```dart
Container(
  key: ExampleSectionKeysEnum.action.key,
  child: ...,
)
```

No teste:

```dart
// GOOD
expect(find.byKey(ExampleSectionKeysEnum.action.key), findsOneWidget);
expect(find.byKey(ExampleSectionKeysEnum.action.key), findsNothing);

// loading — data null
expect(find.byKey(ExampleSectionKeysEnum.shimmer.key), findsOneWidget);

// BAD — frágil
expect(find.text('Título configurável'), findsOneWidget);
expect(find.byIcon(Icons.add), findsNothing);
```

## Config remota

Não usar `find.text` com copy de config remota em **screen ou flow tests**.

| Camada | Copy de config remota | Comportamento / visibilidade |
| ------ | --------------------- | ---------------------------- |
| `*_models_builder_test.dart` | ✅ assert no model; stub por **key** | — |
| `*_screen_test.dart`, flow | ❌ `find.text('label de config')` | ✅ stub da config remota por **key** + `find.byKey` |
| Component burro + fixture | ✅ `find.text` para **dado de negócio** do model | ✅ KeysEnum para tap/show/hide |

```dart
when(remoteSettings.getString('example_section_enabled')).thenReturn('true');
expect(find.byKey(ExampleSectionKeysEnum.group.key), findsOneWidget);
```

## Unit vs widget

| | Widget test (este skill) | Unit test (skill `unit-testing`) |
| - | ------------------------ | -------------------------------- |
| Sujeito | Screen, component, flow | ViewModel, helper, domain, repository |
| API | `tester.pumpWidget`, `find.byKey` | `ProviderContainer`, fakes `implements I*` |
| Copy l10n / config | Sem `find.text` na screen | Assert no model / estado |

ViewModel e AnalyticsTracker: detalhe na skill `unit-testing` (`ProviderContainer`, `MockEventProvider`). Aqui só se o teste **renderiza**.

## Regras extras

- Um arquivo de teste por component (`components/{x}_test.dart`)
- Component: `MaterialApp` + callbacks capturados + `makeTestable()`
- Imagens de rede: `mockNetworkImagesFor()`
- Tipagem explícita também nos testes
