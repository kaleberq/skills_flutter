---
name: flutter-workflow
description: >-
  Roda testes, analyze, format e codegen no projeto Flutter. Use ao validar
  mudanças, executar flutter test / flutter analyze / dart format, ou
  build_runner. Codegen detalhado: skill codegen. Segredos: nunca ler/commitar .env.
alwaysApply: true
---

# Flutter workflow

Validação local. Codegen (`json_serializable`, Chopper, rotas tipadas): skill `codegen`. Wiring: skill `app-layer`.

## Testes

```bash
flutter test
flutter test test/{feature}/path/to/file_test.dart
```

Espelho de pastas e fakes: skill `unit-testing`. Widgets/keys: skill `widget-testing`.

## Análise e format

```bash
flutter analyze
dart format lib test
```

Corrigir o que a tarefa introduziu. Não reformatar o repositório inteiro sem pedido.

## Codegen

Não editar `*.g.dart` / `*.chopper.dart` / `*.freezed.dart`. Gerar **só** o arquivo alvo:

```bash
dart run build_runner build --delete-conflicting-outputs --build-filter=lib/path/to/file.g.dart
```

`--build-filter` aponta para o **arquivo gerado**. Detalhe e quando rodar: skill `codegen`.

## Flavor e env

`--flavor` e `--dart-define-from-file` devem bater com a **tabela do próprio app** (README/`pubspec`/scripts). Não inventar hosts, pares homolog/prod nem nomes de flavor.

```bash
flutter run --flavor <flavor_do_app> --dart-define-from-file=<arquivo_do_app>
```

Nunca ler, copiar ou commitar `.env`, `.env.*` nem valores de secret. Credenciais só via env/arquivo local ignorado pelo git.

## Anti-patterns

- `build_runner` sem `--build-filter` em mudança de um DTO
- Editar `*.g.dart` à mão
- Inventar flavor/host que não está no app
- Commitar ou colar secret de `.env` em skill, comentário ou PR
