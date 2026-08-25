---
name: codegen
description: >-
  Roda build_runner para json_serializable (DTO), Chopper, anotações Riverpod
  e go_router typed routes. Use ao criar/alterar DTO, serviço Chopper, rotas
  tipadas ou arquivos *.g.dart/*.freezed.dart. Nunca editar gerados à mão.
alwaysApply: true
---

# Codegen

Comandos gerais: skill `flutter-workflow`. DTO, Chopper e registro de serializers: skill `data-layer`. Rotas tipadas: skill `presentation-ui`.

## Quando rodar

| Origem | Gera | Rodar quando |
| ------ | ---- | ------------ |
| `@JsonSerializable` no DTO | `*.g.dart` | Novo DTO ou campo JSON |
| Chopper (`@ChopperApi`) | `*.chopper.dart` | Novo endpoint / método |
| go_router typed (`GoRouteData`) | `*.g.dart` de rotas | Nova `TypedGoRoute` |
| Anotação Riverpod (se o app usar) | `*.g.dart` | Novo `@riverpod` |

Não rodar o builder “por garantia” em pasta inteira se um `--build-filter` resolve.

## Comando

```bash
dart run build_runner build --delete-conflicting-outputs --build-filter=lib/data/dtos/item_dto.g.dart
```

O filtro é o **arquivo gerado**, não o fonte. Vários arquivos: um `--build-filter` por output, ou o glob que o app já documentar.

## Nunca editar gerados

`*.g.dart`, `*.freezed.dart`, `*.chopper.dart` são output. Ajuste vai no fonte (`item_dto.dart`, `@ChopperApi`, `*_route.dart`) e no runner.

## Chopper — serializers

Feature nova: mapa `fromJson` **e** inclusão no converter do client. Skill `data-layer` (seção Chopper).

```dart
const itemDtoSerializers = {
  ItemDto: ItemDto.fromJson,
};
```

## Anti-patterns

- Commit de `*.g.dart` inconsistente com o fonte (gerar de novo em vez de editar o `.g.dart`)
- `build_runner watch` na CI ou em tarefa pontual de um DTO
- Esquecer o registro Chopper e só gerar o DTO
