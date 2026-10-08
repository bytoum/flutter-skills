# Testing Retrofit (retrofit.dart + Dio) code

Retrofit generates a `_RestClient` implementation (`*.g.dart`) from an annotated abstract class. Don't write tests against the `.g.dart` file directly — test through the abstract interface. There are three layers, each tested differently:

| Layer | What to test | How |
|---|---|---|
| **Repository / service that uses the client** | Mapping results, error handling, caching | Mock the abstract `RestClient` (most tests go here) |
| **The Retrofit client itself** | Correct method, path, query, headers, body, JSON parsing | Real generated client + fake Dio adapter |
| **JSON models** (`json_serializable` / `freezed`) | `fromJson` / `toJson`, field names, nullability | Plain unit tests with fixture JSON |

## Before running tests

Retrofit and json_serializable need generated code. If `*.g.dart` files are missing or stale, tests won't compile:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Run this after changing any `@RestApi` class or model, and before `flutter test`.

## Example client

```dart
@RestApi(baseUrl: 'https://api.example.com')
abstract class RestClient {
  factory RestClient(Dio dio, {String? baseUrl}) = _RestClient;

  @GET('/users/{id}')
  Future<UserDto> getUser(@Path('id') String id);

  @GET('/users')
  Future<List<UserDto>> getUsers(@Query('page') int page);

  @POST('/users')
  Future<UserDto> createUser(@Body() CreateUserRequest body);
}
```

## 1. Repository tests — mock the client

Because `RestClient` is abstract, it mocks cleanly with either library.

```dart
// mocktail
class MockRestClient extends Mock implements RestClient {}

// mockito
@GenerateNiceMocks([MockSpec<RestClient>()])
```

```dart
void main() {
  late MockRestClient client;
  late UserRepository repository;

  setUp(() {
    client = MockRestClient();
    repository = UserRepository(client);
  });

  group('UserRepository.fetchUser', () {
    test('maps UserDto to User', () async {
      when(() => client.getUser('1'))
          .thenAnswer((_) async => const UserDto(id: '1', fullName: 'Ada'));

      final user = await repository.fetchUser('1');

      expect(user, const User(id: '1', name: 'Ada'));
    });

    test('throws UserNotFoundException on 404', () async {
      when(() => client.getUser(any())).thenThrow(
        dioError(statusCode: 404, path: '/users/1'),
      );

      expect(
        () => repository.fetchUser('1'),
        throwsA(isA<UserNotFoundException>()),
      );
    });

    test('throws NetworkException on connection error', () async {
      when(() => client.getUser(any())).thenThrow(
        DioException(
          requestOptions: RequestOptions(path: '/users/1'),
          type: DioExceptionType.connectionError,
        ),
      );

      expect(
        () => repository.fetchUser('1'),
        throwsA(isA<NetworkException>()),
      );
    });
  });
}

/// Test helper: builds a DioException that looks like an HTTP error response.
DioException dioError({required int statusCode, String path = '/'}) {
  final options = RequestOptions(path: path);
  return DioException(
    requestOptions: options,
    response: Response(requestOptions: options, statusCode: statusCode),
    type: DioExceptionType.badResponse,
  );
}
```

Put `dioError` in a shared helper (e.g. `test/helpers/dio_helpers.dart`) if several test files need it.

Cover for each repository method: success mapping, each HTTP status the code handles (400, 401, 404, 500…), timeouts/connection errors, and malformed/empty payloads if the code handles them.

If a method returns `HttpResponse<T>` (to read headers/status), stub it with:

```dart
HttpResponse(dto, Response(requestOptions: RequestOptions(), statusCode: 200))
```

## 2. Client tests — real generated client, fake transport

These check that the annotations are right: the generated code calls the correct URL with the correct query/body and parses the response. Use whichever option the project already has.

### Option A: `http_mock_adapter` (if in dev_dependencies)

```dart
import 'package:http_mock_adapter/http_mock_adapter.dart';

void main() {
  late Dio dio;
  late DioAdapter adapter;
  late RestClient client;

  setUp(() {
    dio = Dio(BaseOptions(baseUrl: 'https://api.example.com'));
    adapter = DioAdapter(dio: dio);
    client = RestClient(dio, baseUrl: 'https://api.example.com');
  });

  test('getUser calls GET /users/{id} and parses body', () async {
    adapter.onGet(
      '/users/1',
      (server) => server.reply(200, {'id': '1', 'full_name': 'Ada'}),
    );

    final user = await client.getUser('1');

    expect(user, const UserDto(id: '1', fullName: 'Ada'));
  });

  test('getUsers sends page as query parameter', () async {
    adapter.onGet(
      '/users',
      (server) => server.reply(200, []),
      queryParameters: {'page': 2},
    );

    expect(await client.getUsers(2), isEmpty);
  });

  test('createUser POSTs the JSON body', () async {
    adapter.onPost(
      '/users',
      (server) => server.reply(201, {'id': '9', 'full_name': 'Bo'}),
      data: {'full_name': 'Bo'},
    );

    final user = await client.createUser(const CreateUserRequest(fullName: 'Bo'));
    expect(user.id, '9');
  });

  test('throws DioException on 500', () async {
    adapter.onGet('/users/1', (server) => server.reply(500, {'error': 'boom'}));

    await expectLater(client.getUser('1'), throwsA(isA<DioException>()));
  });
}
```

An unmatched route (wrong path/query/body) makes `http_mock_adapter` throw, so the test fails if the annotations are wrong.

### Option B: hand-written fake adapter (no extra dependency)

Use this when `http_mock_adapter` isn't installed and the user doesn't want to add it. It also lets you assert on the captured request directly.

```dart
import 'dart:convert';
import 'dart:typed_data';
import 'package:dio/dio.dart';

class FakeAdapter implements HttpClientAdapter {
  FakeAdapter(this.statusCode, this.body);
  final int statusCode;
  final Object? body;
  final requests = <RequestOptions>[];

  @override
  Future<ResponseBody> fetch(
    RequestOptions options,
    Stream<Uint8List>? requestStream,
    Future<void>? cancelFuture,
  ) async {
    requests.add(options);
    return ResponseBody.fromString(
      jsonEncode(body),
      statusCode,
      headers: {Headers.contentTypeHeader: [Headers.jsonContentType]},
    );
  }

  @override
  void close({bool force = false}) {}
}

test('getUsers sends GET /users?page=2', () async {
  final adapter = FakeAdapter(200, []);
  final dio = Dio()..httpClientAdapter = adapter;
  final client = RestClient(dio, baseUrl: 'https://api.example.com');

  await client.getUsers(2);

  final req = adapter.requests.single;
  expect(req.method, 'GET');
  expect(req.path, '/users');
  expect(req.queryParameters, {'page': 2});
});
```

## 3. Model (DTO) tests

The most common real bug with generated models is a wrong JSON key (`full_name` vs `fullName`) or a nullability mismatch. Test with JSON that looks like the real API response.

```dart
group('UserDto', () {
  test('fromJson reads snake_case keys', () {
    final dto = UserDto.fromJson({'id': '1', 'full_name': 'Ada'});
    expect(dto, const UserDto(id: '1', fullName: 'Ada'));
  });

  test('fromJson handles missing optional avatar', () {
    final dto = UserDto.fromJson({'id': '1', 'full_name': 'Ada'});
    expect(dto.avatarUrl, isNull);
  });

  test('toJson round-trips', () {
    const dto = UserDto(id: '1', fullName: 'Ada');
    expect(UserDto.fromJson(dto.toJson()), dto);
  });
});
```

For large payloads, keep fixture files in `test/fixtures/` (e.g. `user.json`) and load them with `File('test/fixtures/user.json').readAsStringSync()`. Use real (anonymised) responses when possible.

For more model cases (defaults, unknown enums, converters, `explicitToJson`, freezed unions and custom getters), see `references/freezed-json.md`.

## Gotchas

- **Never call the real API** in unit tests. Always mock the client or fake the Dio adapter.
- **Interceptors** (auth tokens, logging, retry) are plain classes — unit-test them separately by calling `onRequest` / `onError` with a mock `RequestInterceptorHandler` / `ErrorInterceptorHandler`.
- If the client is built with a `baseUrl` from config, pass an explicit `baseUrl` in tests so they don't depend on environment.
- `DioException` (Dio 5) replaced `DioError` (Dio 4). Check the Dio version in `pubspec.yaml` and use the matching class.