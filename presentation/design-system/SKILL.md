---
name: design-system
description: >-
  Reusa o design system já existente no projeto: tokens e widgets do pacote DS,
  closest match a partir do Figma, sem UI paralela. Use ao montar layout,
  estilizar component, aplicar tema ou traduzir um frame do Figma em código.
alwaysApply: true
---

# Design system

O app **já tem** um pacote DS. A tarefa é **usar o que existe**, não reinventar. Components burros: skill `ui-components`. Tela: skill `screen-mvvm-architecture`. Tema/bootstrap: skill `app-layer`. Contraste/semantics: skill `accessibility`.

## Regras

- Seguir o DS **existente no projeto** (widgets + tokens). Não criar botão, cor, radius ou tipografia paralelos.
- Figma / spec: referência visual. No código: **closest match** no pacote DS — não pixel-perfect com `Container` + hex.
- Tokens (`spacing`, `radius`, `typography`, `color`, `shadow`) só em `components/` e no `build` da screen.
- ViewModel, helpers, domain e data **não** importam o pacote DS.
- Scaffold / app bar / botão primário / shimmer / input: widget do DS **antes** de Material cru (`Scaffold`, `ElevatedButton`, `CircularProgressIndicator`).
- Tema (`ThemeData` do DS) em `lib/app`. Screen não redefine `Theme` local só para “ficar parecido”.

## Figma → código

1. Identificar o componente no DS (botão, chip, list tile, shimmer…).
2. Se existir equivalente: usar. Mapear padding/cor para **tokens**, não literais.
3. Se não existir: compor com tokens + widgets do DS (não copiar o frame em `BoxDecoration` solto).
4. Componente novo no **pacote DS** só com acordo explícito — não nascer na feature.

Tamanho de **imagem** pode ser literal quando o DS não expõe token para aquilo.

## Tokens (não literais)

```dart
// GOOD
Padding(
  padding: EdgeInsets.all(DsSpacing.medium),
  child: Text(texts.title, style: DsTypography.body),
)

// BAD
Padding(
  padding: const EdgeInsets.fromLTRB(13, 7, 13, 7),
  child: Text(texts.title, style: TextStyle(fontSize: 13, color: Color(0xFF222222))),
)
```

Nomes (`DsSpacing`, `DsTypography`) são ilustrativos — usar os do **pacote DS do app**.

## Onde entra cada coisa

| Peça | Onde |
| ---- | ---- |
| `ThemeData` / `DSTheme` | `lib/app` |
| Scaffold, slivers, estrutura da page | screen (`build`) |
| Card, row, botão, shimmer, campo | `components/` |
| Copy | `*TextsModel` (`ICopySource`) — não no DS |
| Cor “de estado” (erro, sucesso) | token do DS, não hex |

## Anti-patterns

- Segunda paleta / `TextStyle` na feature “porque no Figma está assim”
- `Scaffold` / `AppBar` / `ElevatedButton` Material quando o DS tem equivalente
- Importar DS no ViewModel ou no ModelsBuilder
- Copiar o frame do Figma como árvore de `Container` sem tokens
- Criar widget de design genérico em `presentation/common` em vez de usar (ou estender) o pacote DS
