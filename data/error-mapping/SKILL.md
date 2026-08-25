---
name: error-mapping
description: >-
  Mapeia HTTP, rede e parse para exceptions de domain no adapter IApiClient.
  Use ao implementar ApiClient, tratar DioException/status, ou escrever
  FakeApiClient. Datasource/repository não vazam DioException. Tipos: skill
  domain-exceptions.
---

# Error mapping (HTTP → domain)

Port `IApiClient` e adapter: skill `data-layer`. Tipos: skill `domain-exceptions`.

O mapeamento vive **dentro do adapter**. Acima só existe exception de domain.

## Onde

| Camada | Faz |
| ------ | --- |
| Adapter `ApiClient` | `on DioException` / falha Chopper / JSON inválido → `AppException` |
| Datasource | Parse DTO; se o client já mapeou, não engolir e relançar `DioException` |
| Repository | Orquestra; não importa `dio` / `chopper` |
| Fake em teste | `implements IApiClient` e **lança** `NetworkException` / `NotFoundException` / etc. |

## Adapter

```dart
AppException mapHttpError({required int? statusCode, required bool isOffline}) {
  if (isOffline) return const NetworkException();
  return switch (statusCode) {
    401 || 403 => const UnauthorizedException(),
    404 => const NotFoundException(),
    _ => const UnknownException(),
  };
}

// no get/post:
} on DioException catch (error) {
  throw mapHttpError(
    statusCode: error.response?.statusCode,
    isOffline: error.type == DioExceptionType.connectionError,
  );
}
```

Parse do envelope no datasource:

```dart
try {
  return ItemDto.fromJson(json);
} on Object {
  throw const ParseException();
}
```

Não deixar `FormatException` / `TypeError` vazar para o VM.

## Testes

```dart
class FakeApiClient implements IApiClient {
  @override
  Future<Map<String, dynamic>?> get(String path, {String? accessToken}) {
    throw const NotFoundException();
  }
  // ...
}
```

Assert no repository/DataLoader: tipo de **domain**, não `DioException`.

## Anti-patterns

- `DioException` no datasource, repository, ViewModel ou teste de VM
- Mapear status HTTP na screen (`if (status == 404)`)
- FakeApiClient que lança `DioException`
- Swallow (`catch (_) {}`) no adapter sem traduzir
