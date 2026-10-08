# Testing with get_it (and injectable)

## Prefer constructor injection: then get_it is irrelevant

If the class receives its dependencies through its constructor, unit tests just construct it with mocks. No locator needed:

```dart
class UserRepository {
  UserRepository(this._api, this._cache);
  final UserApi _api;
  final UserCache _cache;
}

// test
repository = UserRepository(MockUserApi(), MockUserCache());
```

This is the case whenever the project uses `injectable` with `@injectable` / `@lazySingleton` on constructor-injected classes. The DI setup lives in `configureDependencies()`, and unit tests don't need to call it.

## When the class calls the locator directly

If code does `GetIt.I<UserApi>()` or `getIt<UserApi>()` inside methods, tests must register doubles in the locator, and reset it after every test so nothing leaks between tests:

```dart
final getIt = GetIt.instance; // or import the project's `getIt`

void main() {
  late MockUserApi api;

  setUp(() {
    api = MockUserApi();
    getIt.registerSingleton<UserApi>(api);
  });

  tearDown(() async {
    await getIt.reset();
  });

  test('loads user via located api', () async {
    when(() => api.getUser('1')).thenAnswer((_) async => dto);

    final user = await ProfileService().load('1');

    expect(user.id, '1');
  });
}
```

Notes:
- `reset()` is async. Always `await` it in `tearDown`.
- Register the abstract type the code asks for (`registerSingleton<UserApi>(mock)`), not the mock type. Otherwise the lookup fails.
- Use `registerFactory` if the code expects a fresh instance per lookup, and `registerSingletonAsync` + `await getIt.allReady()` if it awaits async singletons.
- Point out to the user that direct locator calls make classes harder to test, and suggest moving to constructor injection.

## Optional: DI graph smoke test (injectable)

This test checks that every registration resolves. It catches a missing `@injectable` or a cycle before runtime. Only add it if the user wants it:

```dart
test('all dependencies resolve', () async {
  await configureDependencies(environment: Environment.test);
  addTearDown(getIt.reset);

  expect(() => getIt<UserRepository>(), returnsNormally);
});
```

Register real network/storage modules under a `prod` environment and fakes under `test`, so this test stays offline. If the project's modules can't run offline, don't skip or drop this test. Stop and report why, with options (for example, register fakes for those modules under `test`, or leave the DI-graph test out with the user's agreement), then wait for the user's choice.