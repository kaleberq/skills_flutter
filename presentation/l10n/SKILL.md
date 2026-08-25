---
name: l10n
description: >-
  Strings estáticas via ARB e AppLocalizations; acesso por extension l10n em
  BuildContext. Use ao adicionar copy de UI fixa, ARB, ou context.l10n.
  Copy A/B ou config remota: ModelsBuilder (skill screen-helpers). Extension:
  skill dart-extensions.
---

# l10n

Extension `l10n` no `BuildContext`: skill `dart-extensions`. Models de copy na tela: skill `screen-helpers`. Components: skill `ui-components`. Testes de widget: skill `widget-testing`.

## O que vai para ARB

| Copy | Onde |
| ---- | ---- |
| UI **estática** (título de app bar, hint fixo, erro genérico estável) | ARB → `AppLocalizations` → `context.l10n.*` |
| A/B, remote config, regra de feature | **ModelsBuilder** (keys) → `*TextsModel` |

Não hardcodar português (nem outro idioma) no component se o app já tem l10n para aquele texto.

MaterialApp: `localizationsDelegates` + `supportedLocales` em `lib/app` — skill `app-layer`.

## Acesso

```dart
Text(context.l10n.itemDetailTitle)
```

A extension encapsula `AppLocalizations.of(context)`. Sem `!` no call site. Implementação: skill `dart-extensions`.

ViewModel **não** recebe `BuildContext`. Copy de l10n na screen (ou builder que a screen chama com `l10n` já resolvido) → `*TextsModel`. Helper de tela não importa `material.dart` só para traduzir — a screen passa as strings.

## Testes

Screen / flow: **não** `find.text(l10n.itemDetailTitle)` nem o literal do ARB. Assert por **KeysEnum** — skill `widget-testing`.

Component burro: fixture em `*TextsModel`; `find.text` da fixture é OK no teste do **component**.

## Anti-patterns

- `'Título'` literal no component com l10n existente
- `AppLocalizations.of(context)!` espalhado (usar `context.l10n`)
- ViewModel importando l10n / `BuildContext`
- Screen test com `find.text` da string do ARB
- Meter copy de remote config no ARB (vai no ModelsBuilder)
