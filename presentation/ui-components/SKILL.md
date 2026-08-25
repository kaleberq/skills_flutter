---
name: ui-components
description: >-
  Implementa components burros: model de textos (copy) + model de dados
  (request) nullable; data null → shimmer de loading no próprio component.
  Use ao criar ou alterar widgets em components/, *TextsModel, *DataModel
  ou estados de loading por seção.
---

# Components burros

Tela e ViewModel: skill `screen-mvvm-architecture`. Copy/labels: skill `screen-helpers` (ModelsBuilder) e skill `l10n` (ARB estático). Testes: skill `widget-testing`. Semantics e shimmer: skill `accessibility`.

Component **não** busca API, não lê Riverpod, não monta string de config remota. Só apresenta o que o construtor recebe.

## Dois models

| Model | Sufixo | Origem | Null |
| ----- | ------ | ------ | ---- |
| **Textos** | `*TextsModel` | Config remota / l10n via ModelsBuilder | Em geral **não** — copy já existe antes do fetch |
| **Dados** | `*DataModel` | Request, repository, cálculo a partir do domain | **Sim** — `null` = ainda carregando |

Não misturar copy e payload num único model. Callbacks no construtor (`VoidCallback`, `ValueChanged<T>`).

```dart
@immutable
class ExampleSectionTextsModel {
  final String title;
  final String actionLabel;

  const ExampleSectionTextsModel({
    required this.title,
    required this.actionLabel,
  });
}

@immutable
class ExampleSectionDataModel {
  final String valueLabel;
  final int count;

  const ExampleSectionDataModel({
    required this.valueLabel,
    required this.count,
  });
}
```

## Construtor

```dart
class ExampleSection extends StatelessWidget {
  final ExampleSectionTextsModel texts;
  final ExampleSectionDataModel? data;
  final VoidCallback? onAction;

  const ExampleSection({
    super.key,
    required this.texts,
    this.data,
    this.onAction,
  });
```

- `texts` obrigatório (títulos, botões, hints).
- `data` opcional. Screen passa `null` enquanto o request não resolveu (ou `AsyncValue` ainda é loading / sem value).
- Sem `data!`. Sem `ConsumerWidget` / `ref.*`.
- Sem funções que retornam `Widget` (`Widget _row()`). Trecho de UI → outro widget em `components/`.

## Loading = shimmer no component

Quem **usa** o component não monta um loader à parte. Passa `data: null`; o **próprio component** desenha o shimmer do design system no lugar dos valores.

Dois jeitos (os dois válidos):

**1. Bloco inteiro** — quando textos e dados precisam existir juntos para o layout fazer sentido:

```dart
@override
Widget build(BuildContext context) {
  final ExampleSectionDataModel? data = this.data;
  if (data == null) {
    return Shimmer.rectangular(
      key: ExampleSectionKeysEnum.shimmer.key,
      height: 64,
      width: double.infinity,
    );
  }
  return _body(texts: texts, data: data);
}
```

**2. Slot a slot** — chrome (card, título de `texts`) já visível; só os campos da request viram shimmer:

```dart
final ExampleSectionDataModel? data = this.data;
if (data != null)
  Text(data.valueLabel, key: ExampleSectionKeysEnum.value.key)
else
  const Shimmer.rectangular(
    key: ExampleSectionKeysEnum.shimmer.key,
    height: 16,
    width: 120,
  );
```

`Shimmer` é o widget de skeleton do DS do app (retângulo/círculo no tamanho do texto/slot). Não usar `CircularProgressIndicator` no lugar do conteúdo da seção.

## Screen (call site)

```dart
ExampleSection(
  texts: notifier.exampleSectionTexts,
  data: item == null ? null : notifier.buildExampleSectionData(item),
  onAction: () {
    tracker.trackAction(...);
    notifier.onAction();
  },
)
```

- `item == null` (ou `asyncItem.valueOrNull == null`) → `data: null` → shimmer.
- Não: `if (loading) return CircularProgressIndicator()` envolvendo o component.
- Não: esconder o component até o fetch acabar (some o layout).

## Nulidade — sem `!`

Não usar `!` em propriedades nullable (`if (widget.data!)`, `widget.data!.count`). Promover o tipo:

```dart
final ExampleSectionDataModel? data = widget.data;

if (data != null) {
  // usa data
}
```

## Sem funções que retornam Widget

`Widget _header()` na screen ou no component → extrair `ExampleHeader` (arquivo próprio). Facilita reuso, leitura, teste e manutenção.

## Regras

- Keys: `IKeyEnum` — incluir `shimmer` (e slots) para teste de loading.
- StatefulWidget só para UI efêmera (focus, debounce, accordion).
- Tokens do DS só no component (e no `build` da screen), não no ViewModel. Contraste: tokens, não hex — skill `accessibility`.
- Models `@immutable`, `const`, campos `final`.
- Textos vêm de `*TextsModel` (builder / `l10n` na screen). Sem literal de UI no component se houver l10n.

## Testes

```dart
// loading
ExampleSection(texts: _texts, data: null);
expect(find.byKey(ExampleSectionKeysEnum.shimmer.key), findsOneWidget);
expect(find.byKey(ExampleSectionKeysEnum.value.key), findsNothing);

// loaded
ExampleSection(texts: _texts, data: _data);
expect(find.byKey(ExampleSectionKeysEnum.shimmer.key), findsNothing);
expect(find.byKey(ExampleSectionKeysEnum.value.key), findsOneWidget);
```

Copy de `texts`: `find.text` no teste do **component** com fixture. Screen/flow: `find.byKey`, não `find.text` de config remota.

## Anti-patterns

- Um único `*Model` misturando label de config e campo de API
- `data` non-null obrigatório e loading na screen
- `data!` / `late` no model de request / `widget.valor!`
- `Widget _buildX(...)` em vez de um widget separado
- Shimmer na screen em volta do component em vez de `data: null`
- Component lendo repository, `AsyncValue` ou config remota
- Spinner no lugar de skeleton da seção
