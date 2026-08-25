---
name: domain-exceptions
description: >-
  Define exceptions de domain (sealed/base) e o contrato de mapeamento para
  mensagem de UI. Use ao criar tipos de erro de negócio/rede, mapear HTTP no
  adapter, ou exibir falha na screen. Mapeamento HTTP: skill error-mapping.
  Sem catch em components burros.
alwaysApply: true
---

# Exceptions de domain

HTTP → exception: skill `error-mapping`. UI: skills `mvvm-architecture` e `screen-mvvm-architecture`. Sem Flutter nestes tipos.

## Tipos

Hierarquia **sealed** ou classe base no domain. Sem `DioException`, status HTTP cru ou JSON de envelope na presentation.

```dart
sealed class AppException implements Exception {
  const AppException();
}

final class NetworkException extends AppException {
  const NetworkException();
}

final class UnauthorizedException extends AppException {
  const UnauthorizedException();
}

final class NotFoundException extends AppException {
  const NotFoundException();
}

final class ParseException extends AppException {
  const ParseException();
}

final class UnknownException extends AppException {
  const UnknownException();
}
```

Nomes e granularidade seguem o **app**. Não vazar tipo do client HTTP.

## Onde mapeia

O **adapter** `ApiClient` (`IApiClient`) traduz falha de rede/HTTP/parse → `AppException`. Datasource e repository **deixam subir** a exception de domain (ou rethrow). Skill `error-mapping`.

## Presentation

ViewModel / DataLoader: `AsyncValue.guard` ou `catch` **no VM/helper**, não no component burro.

Mensagem ao usuário: **helper de mapping** (switch na sealed class → string já resolvida no `*TextsModel` / `ICopySource`), não `e.toString()`, body HTTP nem `AppLocalizations` no VM.

```dart
String messageFor(AppException error, ItemDetailTextsModel texts) {
  return switch (error) {
    NetworkException() => texts.networkError,
    UnauthorizedException() => texts.unauthorizedError,
    NotFoundException() => texts.notFoundError,
    ParseException() => texts.parseError,
    UnknownException() => texts.genericError,
  };
}
```

Component burro recebe `String` já resolvida (ou `data == null` + estado de erro na screen). **Não** faz `try/catch`.

## Crash

Falhas inesperadas: reportar no **serviço de crash da app** (comentário/chamada genérica). Não amarrar SDK de um vendor neste skill.

```dart
} catch (e, st) {
  // Reportar o erro no serviço de crash da app.
  state = state.copyWith(errorMessage: messageFor(_asAppException(e), texts));
}
```

## Anti-patterns

- `on DioException` na screen ou no ViewModel
- `catch` em component burro
- Mostrar `exception.toString()` / JSON cru
- Exception de domain com `Widget` ou `BuildContext`
