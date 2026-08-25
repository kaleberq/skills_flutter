---
name: data-layer
description: >-
  Implementa a camada data em Clean Architecture: DTOs, HTTP via port
  IApiClient (adapter sobre Chopper ou Dio), Hive/socket, services e
  repositories. Use ao criar/alterar API, DTOs, serializers, Chopper, Dio,
  adapter HTTP, IApiClient, repositories, services, channels ou storage.
alwaysApply: true
---

# Data layer

Local: `lib/data/`. Contratos: skill `domain-layer`. Wiring Riverpod: skill `app-layer`. Codegen: skill `codegen` (`build_runner` + `--build-filter`). HTTP → exception de domain: skill `error-mapping`.

```text
lib/data/
├── dtos/              # toJson/fromJson — sem regra de negócio
├── datasources/       # I/O cru (HTTP Chopper ou Dio, Hive, socket)
├── services/          # I{Name}Service — fala DTO, não domain model
├── repositories/      # I{Feature}Repository — orquestra + mapeia DTO → model
└── channels/          # I{Name}Channel — MethodChannel / plataforma
```

HTTP entra atrás de **port + adapter** (`IApiClient`). Chopper e Dio ficam só no adapter. Domain, datasources, repositories e ViewModel **não** importam `chopper` nem `dio`.

## Regras

- Toda classe concreta aqui **implementa** uma interface em `domain/interfaces`.
- Sem widgets / detalhes de UI.
- Presentation **não** importa data (só domain).
- Service conhece **DTO**; repository conhece **domain model** e faz o map.
- Datasource conhece protocolo/I/O (JSON, `Response`, socket) — não regra de negócio.

## Fluxo

```text
Datasource / HTTP / Channel
        ↓ DTO (fromJson)
Service (I*Service)          ← opcional; infra (rede, cache)
        ↓ DTO
Repository (I*Repository)    ← map DTO → domain model
        ↓ Model
ViewModel / DataLoader
```

## DTO (transporte)

Só serialização. Sem getters de negócio.

```dart
class ItemDto {
  final int id;
  final String name;
  final int? relatedId;

  const ItemDto({required this.id, required this.name, this.relatedId});

  factory ItemDto.fromJson(Map<String, dynamic> json) { /* ... */ }
  Map<String, dynamic> toJson() { /* ... */ }
}
```

Chopper ou Dio: DTO com `fromJson`/`toJson` (`@JsonSerializable` quando houver codegen). `const`, `@JsonKey` se o nome diferir. Nunca editar `*.g.dart`.

Sufixo: **`Dto`**. Código novo não usa `Entity` como modelo de negócio — domínio é o model em `lib/domain`. Envelope de API sem model de negócio continua `Dto`.

## Repository (implementação)

Injeta datasource que usa `IApiClient` (não Dio/Chopper). Nunca retorna DTO. Um repository por feature.

```dart
class ItemRepository implements IItemRepository {
  final ItemRemoteDataSource _remote;

  const ItemRepository({required ItemRemoteDataSource remote})
      : _remote = remote;

  @override
  Future<List<ItemModel>> getItems() async {
    final List<ItemDto> dtos = await _remote.getItems();
    return dtos.map(_toModel).toList();
  }

  ItemModel _toModel(ItemDto dto) => ItemModel(
        id: dto.id,
        title: dto.name,
        relatedId: dto.relatedId,
      );
}
```

## Service vs datasource vs channel

| Peça | Faz |
| ---- | --- |
| Datasource | I/O cru, buffer, protocolo |
| Service | Implementa `I*Service`; DTO in/out |
| Channel | Implementa `I*Channel`; MethodChannel |
| Repository | Implementa `I*Repository`; map + orquestração |

## HTTP — port + adapter (`IApiClient`)

Desacoplar o client HTTP com **Adapter** (Clean/Hexagonal: **port + adapter**).

| Peça | Onde | Papel |
| ---- | ---- | ----- |
| **Port** `IApiClient` | `domain/interfaces/services/` | Contrato: `get`/`post`/`patch`/`delete` → `Map<String, dynamic>?`. Sem Dio/Chopper. |
| **Adapter** `ApiClient` | `data/services/` | Implementa `IApiClient`; encapsula Dio **ou** Chopper; mapeia erro de rede para exception de domain. |
| Datasource / repository | `data/` | Depende de `IApiClient`, nunca do client concreto. |

```dart
abstract interface class IApiClient {
  Future<Map<String, dynamic>?> get(String path, {String? accessToken});
  Future<Map<String, dynamic>?> post(
    String path, {
    required Map<String, dynamic> body,
    String? accessToken,
  });
  Future<Map<String, dynamic>?> patch(
    String path, {
    required Map<String, dynamic> body,
    String? accessToken,
  });
  Future<void> delete(String path, {String? accessToken});
}
```

- Um adapter por implementação (`ApiClient` com Dio, outro com Chopper se precisar).
- Interceptors, timeout, `baseUrl` e headers ficam no adapter.
- Não vazar `DioException` / `Response` do Chopper para cima — mapear para exception de domain (skill `error-mapping`).
- Testes: `FakeApiClient implements IApiClient` e lança exceptions de **domain**, não Dio.

Chopper e Dio são válidos **dentro do adapter**. Datasource:

```dart
class ItemRemoteDataSource {
  final IApiClient _apiClient;
  const ItemRemoteDataSource({required IApiClient apiClient})
      : _apiClient = apiClient;

  Future<List<ItemDto>> getItems() async {
    final Map<String, dynamic>? json = await _apiClient.get('/items');
    final List<dynamic> raw = json?['data'] as List<dynamic>? ?? [];
    return raw
        .map((e) => ItemDto.fromJson(e as Map<String, dynamic>))
        .toList();
  }
}
```

### Adapter com Dio

```dart
class ApiClient implements IApiClient {
  final Dio _dio;
  ApiClient({Dio? dio}) : _dio = dio ?? Dio(BaseOptions(baseUrl: baseUrl));

  @override
  Future<Map<String, dynamic>?> get(String path, {String? accessToken}) async {
    try {
      final response = await _dio.get<dynamic>(path, options: Options(headers: _auth(accessToken)));
      return response.data is Map<String, dynamic> ? response.data as Map<String, dynamic> : null;
    } on DioException catch (error) {
      throw _mapError(error);
    }
  }
}
```

### Adapter com Chopper

Chopper (`@ChopperApi`, `*.chopper.dart`) vive só no adapter. Registrar `fromJson` no mapa da feature **e** no converter do client. `build_runner` no arquivo gerado (`--build-filter`). O restante da app continua falando `IApiClient`.

## Storage local

São datasources atrás de interface de domain (`ICacheService`, etc.).

| Tech | Uso |
| ---- | --- |
| Store local (ex.: Hive) | Persistência via interface de domain |
| SharedPreferences | Preferências simples, atrás da mesma interface quando fizer sentido |

## Wiring

Providers `Provider<I*>` e `GoRouter` ficam em `lib/app/` — skill `app-layer`. Data só **implementa**; não registra o grafo global.

Compor implementações **fora** do domain.

## Testes

Fake da **interface** (`class FakeNetworkService implements INetworkService`). DTO: round-trip `toJson`/`fromJson`.

## Anti-patterns (quebra SOLID)

| Evitar | Princípio |
| ------ | --------- |
| `Dio` / Chopper no datasource, repository, ViewModel ou screen | **D** — depender da concretude, não de `IApiClient` |
| Vazar `DioException`, `Response` Chopper ou o client HTTP para cima | **L** + **D** — contrato do port não é honrado |
| `IAppRepository` / um repository para várias features | **I** + **S** |
| Repository concreto em `lib/domain/` | **D** |
| DTO com regra de negócio; domain model com `fromJson` | **S** |
| Adapter (`ApiClient`) misturando parse de feature + I/O HTTP | **S** — I/O no adapter; DTO no datasource; model no repository |
| Screen/ViewModel importar `IApiClient`, Chopper ou Dio | **D** — presentation fala com `I*Repository` |
| `Provider<ItemRepository>` tipado na impl, não em `IItemRepository` | **D** |
| UI em `lib/data` | **S** |

## Chopper — registro (só Chopper)

```dart
const itemDtoSerializers = {
  ItemDto: ItemDto.fromJson,
};
```

Feature nova: criar o mapa **e** incluir no converter do client Chopper.

Testes: DTO round-trip em `test/unit/data/dtos/`; repository/datasource com `FakeApiClient implements IApiClient`; ViewModel com fake de `I*Repository` (não Chopper/Dio real).

Código novo usa sufixo `Dto`. `Entity` existente = transporte. Não migrar todos os repositories do domain numa PR de um endpoint.
