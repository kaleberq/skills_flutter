---
name: l10n
description: >-
  Copy via abstração ICopySource (não AppLocalizations na tela/VM). Use ao
  adicionar textos de UI, ARB, config remota de copy, ou ModelsBuilder de
  *TextsModel. Telas e ViewModel não acessam l10n concreto.
---

# Copy (abstração)

Helpers: skill `screen-helpers`. Components: skill `ui-components`. Wiring: skill `app-layer`. Extension `l10n` no `BuildContext` **não** é para screens — só o adapter que implementa `ICopySource`.

## Regras

- Telas **não** acessam `l10n` / `context.l10n` / `AppLocalizations` diretamente.
- Textos passam por uma **abstração** desacoplada da implementação (`ICopySource`).
- O ViewModel **não** depende de `AppLocalizations` nem de qualquer impl concreta de localização.
- A abstração permite trocar a fonte depois (config remota, API, ARB) **sem** mudar telas nem regra de negócio.
- Não criar abstração para texto que **não** precisa de desacoplamento. **Seguir esta regra** para textos usados pela lógica do ViewModel (incluindo ModelsBuilder) ou que possam mudar de fonte.
- Funcionalidade nova: dependência na **abstração**, nunca na implementação concreta.

## Contrato (domain)

```dart
enum CopyKey {
  itemDetailTitle,
  itemDetailTitleAlt,
  itemDetailSubtitle,
  itemDetailSubtitleAlt,
  itemDetailConfirm,
  networkError,
}

abstract interface class ICopySource {
  String text(CopyKey key);
}
```

`ICopySource` em `domain/interfaces/services/`. Keys estáveis (enum ou id). Sem Flutter no contrato.

## Implementação (data / app)

Uma classe concreta **por fonte**. Só ela importa `AppLocalizations` (ou o SDK de config remota / HTTP).

```dart
class L10nCopySource implements ICopySource {
  final AppLocalizations _localizations;

  const L10nCopySource(this._localizations);

  @override
  String text(CopyKey key) => switch (key) {
        CopyKey.itemDetailTitle => _localizations.itemDetailTitle,
        CopyKey.itemDetailTitleAlt => _localizations.itemDetailTitleAlt,
        CopyKey.itemDetailSubtitle => _localizations.itemDetailSubtitle,
        CopyKey.itemDetailSubtitleAlt => _localizations.itemDetailSubtitleAlt,
        CopyKey.itemDetailConfirm => _localizations.itemDetailConfirm,
        CopyKey.networkError => _localizations.networkError,
      };
}
```

Trocar fonte: outra impl (`RemoteCopySource`, `ApiCopySource`) + o mesmo `Provider<ICopySource>`. Screen e VM intactos.

`MaterialApp`: `localizationsDelegates` + `supportedLocales` em `lib/app` — só o composition root. Provider:

```dart
final copySourceProvider = Provider<ICopySource>((ref) {
  // constrói a impl atual (ARB, config remota, …)
  return L10nCopySource(/* localizations resolvidas no app */);
});
```

## Fluxo na tela

```text
ICopySource → ModelsBuilder → *TextsModel → Screen → Component
```

- ViewModel injeta `ICopySource` no ModelsBuilder (`ref.read(copySourceProvider)` no `build()` do Notifier).
- Screen só recebe `*TextsModel` + `*DataModel?`. Sem `context.l10n`.
- Component burro só lê `texts` — skill `ui-components`.

```dart
class ItemDetailModelsBuilder {
  final ICopySource _copy;

  const ItemDetailModelsBuilder({required ICopySource copy}) : _copy = copy;

  ItemDetailTextsModel buildTexts() => ItemDetailTextsModel(
        title: _copy.text(CopyKey.itemDetailTitle),
        confirmLabel: _copy.text(CopyKey.itemDetailConfirm),
      );
}
```

## O que não abstrair

Literais que **não** entram em VM / `*TextsModel` / mensagem de erro de negócio: logs, `eventName` de analytics, strings de teste interno. Não inventar `ICopySource` para isso.

## Testes

Fake: `class FakeCopySource implements ICopySource`. Stub por `CopyKey`. Screen/flow: **KeysEnum**, não `find.text` do ARB. Skill `widget-testing` / `unit-testing`.

## Anti-patterns

- `context.l10n` / `AppLocalizations.of` na screen ou no component
- ViewModel ou ModelsBuilder importando `AppLocalizations` / `flutter_gen/gen_l10n`
- `Provider<L10nCopySource>` em vez de `Provider<ICopySource>`
- Hardcode de copy de produto no VM ou no component quando o texto é de UI/negócio
- Nova feature dependendo da impl ARB em vez de `ICopySource`
