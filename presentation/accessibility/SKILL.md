---
name: accessibility
description: >-
  Semantics, excludeFromSemantics em decorativo, contraste via tokens do DS e
  labels em shimmer quando o leitor de tela precisaria. Use ao criar/alterar
  components, keys, skeleton de loading ou cores. Keys: skill widget-testing.
  Components: skill ui-components.
---

# Acessibilidade

Components burros: skill `ui-components`. Keys para teste: skill `widget-testing`. Copy: skill `l10n` / `screen-helpers`.

## Semantics

- Conteúdo **acionável** (botão, link, campo): `Semantics` / widget já semântico (`TextButton`, `TextField`) + **key** (`IKeyEnum`)
- **Decorativo** (ícone duplicando texto, divisor): `excludeFromSemantics: true` no `Image`/`Icon` redundante
- Não aninhar `Semantics` que anunciam o mesmo rótulo duas vezes

```dart
Icon(Icons.chevron_right, excludeFromSemantics: true)
```

## Shimmer / loading

`data == null` → shimmer (skill `ui-components`). Evitar região vazia **sem** nome para o leitor de tela se aquele slot for o conteúdo principal:

```dart
Semantics(
  label: texts.loadingLabel,
  child: Shimmer.rectangular(
    key: ExampleSectionKeysEnum.shimmer.key,
    height: 64,
    width: double.infinity,
  ),
)
```

`loadingLabel` vem de `*TextsModel` (l10n ou ModelsBuilder), não literal no component. Se o chrome (título de `texts`) já descreve a seção, não duplicar.

## Contraste e cor

Cores e tipografia: **tokens do design system**, não `Color(0xFF...)` solto no component. Contraste segue o DS; não “corrigir” com hex paralelo.

## Keys

Keys não substituem semantics, mas não conflitam: `key` no widget visível (incluindo shimmer). Testes usam `find.byKey` — skill `widget-testing`.

## Anti-patterns

- Ícone + texto com o mesmo anúncio duplicado
- Shimmer full-screen sem label quando não há título visível
- `color: Color(0xFF000000)` para “acessibilidade”
- `IgnorePointer` + área clicável invisível sem semantics
- Component catch de erro só para não “quebrar” o leitor — erro é da screen/VM
