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

Telas MVVM: skill `screen-mvvm-architecture` (espelhamento e cobertura). Imagens do DS: fake em `FLUTTER_TEST` quando o pacote já fizer isso.

## Por quê

Keys são estáveis. Texto e ícone mudam com tradução, A/B e design.

## Keys enum

`IKeyEnum` em `lib/presentation/common/enums/key_enum_interface.dart` (ou pasta equivalente de UI compartilhada — não em `lib/data`).

```dart
enum ExampleCardKeysEnum implements IKeyEnum {
  action(key: Key('example_card_action'));

  @override
  final Key key;

  const ExampleCardKeysEnum({required this.key});
}
```

No widget:

```dart
Container(
  key: ExampleCardKeysEnum.action.key,
  child: ...,
)
```

No teste:

```dart
// GOOD
expect(find.byKey(ExampleCardKeysEnum.action.key), findsOneWidget);
expect(find.byKey(ExampleCardKeysEnum.action.key), findsNothing);

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

## Regras extras

- Um arquivo de teste por component (`components/{x}_test.dart`)
- Component: `MaterialApp` + callbacks capturados + `makeTestable()`
- ViewModel: `ProviderContainer` + overrides + mockito
- Analytics: `verify` no `MockEventProvider`
- Imagens de rede: `mockNetworkImagesFor()`
- Tipagem explícita também nos testes
