---
name: dart-extensions
description: >-
  Cria Dart extensions em lib/extensions/: sugar em tipos existentes (String,
  DateTime, BuildContext, enums), sem regra de negócio de feature. Use ao
  adicionar ou alterar extension methods, l10n no context, ou helpers de tipo.
---

# Extensions

Local: `lib/extensions/`. Models e regras de feature: skill `domain-layer`. UI de tela: skills em `presentation/`. ARB e `context.l10n`: skill `l10n`.

Extension **enriquece um tipo**. Não vira service, repository nem ViewModel.

```text
lib/extensions/
├── string/
│   └── string_formatting.dart
├── date_time/
│   └── date_time_formatting.dart
└── build_context/
    └── l10n_extension.dart
```

Agrupar por **tipo estendido**, não por feature.

## Regras

- Um conceito por arquivo. Nome: `{tipo}_{papel}.dart` ou `{tipo}_extension.dart`.
- Métodos **puros** quando possível (mesmo input → mesmo output).
- Sem I/O, HTTP, storage, Riverpod, analytics.
- Sem import de `lib/data` nem de screens.
- Flutter só quando o tipo exige (`BuildContext`, `Color`, etc.).
- Lógica de negócio que precisa de vários campos do model → **método/getter no domain model**, não extension.

## Exemplos

```dart
extension StringFormatting on String {
  String get capitalized {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }
}

extension DateTimeFormatting on DateTime {
  String toIsoDate() =>
      '${year.toString().padLeft(4, '0')}-${month.toString().padLeft(2, '0')}-${day.toString().padLeft(2, '0')}';
}

extension L10nContext on BuildContext {
  AppLocalizations get l10n {
    final AppLocalizations? localizations = AppLocalizations.of(this);
    if (localizations == null) {
      throw StateError('AppLocalizations not found');
    }
    return localizations;
  }
}
```

Enum: extension no **próximo estado** ou label estável só se não couber no próprio enum.

```dart
extension ItemStatusNext on ItemStatus {
  ItemStatus get next => switch (this) {
        ItemStatus.draft => ItemStatus.active,
        ItemStatus.active => ItemStatus.closed,
        ItemStatus.closed => ItemStatus.closed,
      };
}
```

## Anti-patterns

- `ItemRepositoryX on ItemRepository` com fetch
- Extension que monta `Widget`
- Extension em DTO (`ItemDto`) — mapeamento fica no repository em data
- Pasta `extensions/` com classes que não são `extension`
- Copiar regra de feature (preço, permissão, jornada) para extension em vez do model
