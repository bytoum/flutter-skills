---
name: flutter-unit-testing
description: Write, fix, and run unit tests for Flutter and Dart code — services, repositories, models, use cases, Retrofit/Dio API clients, freezed and json_serializable models, classes wired with get_it/injectable, and state classes (Bloc/Cubit, Riverpod notifiers, Provider/ChangeNotifier). Use this skill whenever the user asks to add tests, improve coverage, test a class or function, mock a dependency or API, fix a failing `flutter test`, or check that Dart logic works, even if they don't say "unit test". Not for widget tests, golden tests, or integration tests.
---

# Flutter Unit Testing

This skill covers **unit tests**: tests of Dart logic that run with `flutter test` without pumping widgets. The goal is tests that are fast, deterministic, readable, and that would actually fail if the behavior broke.

If the user asks for widget, golden, or integration tests, say this skill is for unit tests and handle that request with general knowledge instead.

## Workflow

### 1. Learn the project before writing anything

Read `pubspec.yaml` and look at 1–2 existing files under `test/`. You are looking for:

- **Mocking library** in `dev_dependencies`: `mocktail`, `mockito`, both, or neither.
- **State management**: `flutter_bloc`/`bloc` (+ `bloc_test`), `flutter_riverpod`/`riverpod`/`hooks_riverpod`, `provider`.
- **Networking / code generation**: `retrofit` + `dio`, `json_serializable`, `freezed`, `http_mock_adapter`, `build_runner`.
- **Dependency injection**: `get_it`, `injectable`. Note whether classes take dependencies via constructor or call the locator (`GetIt.I<T>()` / `getIt<T>()`) directly.
- **Helpers already present**: `fake_async`, `clock`, custom fakes, `test/helpers/`, shared fixtures.
- **Conventions** in existing tests: folder layout, naming of `group`/`test` descriptions, `setUp` style, whether they use fakes or mocks. Match them — consistency with the codebase matters more than any style preference in this skill.

### 2. Choose the mocking approach (align with the project)

| Found in pubspec | Do this |
|---|---|
| `mocktail` only | Use mocktail. |
| `mockito` only | Use mockito with generated mocks. |
| Both | Follow what the nearest existing tests for that feature use. |
| Neither | **Stop and ask the user** which they want (mocktail = no code generation, simpler; mockito = generated, type-checked mocks). Don't add a dependency on your own. |

Then read `references/mocking.md` for the exact syntax of the chosen library.

Also ask yourself whether a mock is needed at all. A hand-written fake (e.g. an in-memory repository) is often clearer than a mock for simple dependencies, and pure functions/models need no doubles. Mock at the boundaries: network clients, databases, platform channels, storage, clocks, and other repositories/services.

### 3. Read the unit under test and list behaviors

Open the source file and write down (mentally or in a short comment to the user) what needs covering:

- Happy path for each public method
- Edge cases: empty lists, null/optional values, boundaries, zero, duplicates
- Error paths: thrown exceptions, failed futures, error results/states
- State transitions (for Bloc/Riverpod/ChangeNotifier): initial state, each event/method, loading → success / loading → failure
- Interactions that matter: that a dependency was called with the right arguments, or *not* called (e.g. cache hit skips network)

Test behavior through the public API. Don't test private methods directly or assert on implementation details that could change without behavior changing.

### 4. Load the matching guide

- Bloc / Cubit → `references/bloc.md`
- Riverpod → `references/riverpod.md`
- Provider / ChangeNotifier → `references/provider.md`
- Retrofit `@RestApi` clients, repositories that call them, Dio interceptors → `references/retrofit.md`
- `freezed` / `json_serializable` models: JSON mapping, defaults, enums, converters, unions, custom getters → `references/freezed-json.md`
- Anything registered in or resolved from `get_it` / `injectable` → `references/get-it.md`
- Plain Dart services, repositories, models, utils → the patterns below are enough.

A class can need more than one guide (e.g. a Cubit that calls a Retrofit-backed repository: use `bloc.md` for the Cubit and mock the repository; test the repository separately with `retrofit.md`).

### 5. Write the test file

- **Location**: mirror `lib/` under `test/`. `lib/features/auth/data/auth_repository.dart` → `test/features/auth/data/auth_repository_test.dart`. The file name must end in `_test.dart` or `flutter test` won't pick it up.
- **Imports**: use `package:<app_name>/...` imports (app name from `pubspec.yaml`), and `package:flutter_test/flutter_test.dart` (or `package:test/test.dart` for pure-Dart packages — match the project).
- **Structure**: one top-level `group` named after the class, nested `group`s per method, then `test`s. Descriptions should read as sentences: `'returns cached user when cache is fresh'`.
- **Arrange / Act / Assert**: keep the three phases visually distinct in each test.
- **Fresh state per test**: create the subject and its doubles in `setUp`, never share mutable objects across tests. Use `addTearDown` for cleanup (closing streams, disposing containers).

Template:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/features/user/data/user_repository.dart';
// + mocking import per references/mocking.md

void main() {
  late MockUserApi api;
  late UserRepository repository;

  setUp(() {
    api = MockUserApi();
    repository = UserRepository(api: api);
  });

  group('UserRepository', () {
    group('fetchUser', () {
      test('returns user when api succeeds', () async {
        // Arrange
        // stub api.getUser('42') to return a User

        // Act
        final result = await repository.fetchUser('42');

        // Assert
        expect(result, const User(id: '42', name: 'Ada'));
        // verify api.getUser('42') called once
      });

      test('throws UserNotFoundException when api returns 404', () async {
        // stub api to throw ApiException(statusCode: 404)

        expect(
          () => repository.fetchUser('42'),
          throwsA(isA<UserNotFoundException>()),
        );
      });
    });
  });
}
```

### 6. Core assertion patterns

```dart
// Values and collections
expect(value, equals(3));
expect(list, containsAll([1, 2]));
expect(list, isEmpty);
expect(user, isA<AdminUser>().having((u) => u.level, 'level', 2));

// Exceptions — pass a closure, don't call the function first
expect(() => parse('x'), throwsA(isA<FormatException>()));
expect(() => parse('x'), throwsFormatException);

// Futures
await expectLater(service.load(), completion(isNotEmpty));
await expectLater(service.load(), throwsA(isA<NetworkException>()));

// Streams
await expectLater(
  service.watch(),
  emitsInOrder([1, 2, emitsDone]),
);
expect(stream, emitsError(isA<TimeoutException>()));
```

Equality: `expect` on custom classes needs `==`/`hashCode` (e.g. via `equatable` or `freezed`). If the model lacks equality, assert on fields with `.having(...)` rather than adding equality to production code just for tests — unless the user agrees.

### 7. Time, randomness, and async

Unit tests must not depend on the real clock, real delays, or real network.

- **Timers / `Future.delayed` / debounce**: use `fake_async`:
  ```dart
  test('debounces search by 300ms', () {
    fakeAsync((async) {
      cubit.onQueryChanged('fl');
      async.elapse(const Duration(milliseconds: 299));
      verifyNever(/* api.search call */);
      async.elapse(const Duration(milliseconds: 1));
      // verify api.search called once
    });
  });
  ```
- **`DateTime.now()`**: inject a clock (`package:clock` with `withClock(Clock.fixed(...), () {...})`) or a `DateTime Function()` parameter. If the code calls `DateTime.now()` directly and can't be controlled, tell the user and suggest the small refactor.
- **Randomness**: inject a seeded `Random`.
- Always `await` async calls in tests, and make the test body `async`. A missing `await` makes a test pass vacuously.

### 8. Run and fix

```bash
flutter test test/path/to/file_test.dart          # one file
flutter test --plain-name "fetchUser"              # tests matching a name
flutter test                                       # whole suite
flutter test --coverage                            # writes coverage/lcov.info
```

If the project uses code generation (mockito, Retrofit, json_serializable, freezed, riverpod_generator), run `dart run build_runner build --delete-conflicting-outputs` first. Missing or stale `*.g.dart` / `*.mocks.dart` / `*.freezed.dart` files are the most common reason new tests fail to compile.

When a test fails:
- Read the failure carefully. Distinguish "the test is wrong" (bad stub, missing `await`, wrong expectation) from "the code is wrong".
- **Never weaken an assertion just to make it pass.** Only change production code if the user asked you to.
- **Never skip, disable, or delete a failing test** to get a green run. That means no `skip:`, `markTestSkipped`, commented-out tests, `@Skip`, or `--exclude-tags`/`--plain-name` filters used to hide it. A skipped test reads as a passing suite and buries the problem.
- If you cannot fix a failure by correcting the test itself (the test is the wrong part), **stop**. Do not continue writing more tests or work around it. Leave the failing test in place and report to the user:
  1. **Which test failed**: file, test name, and the expected vs actual output.
  2. **Why**: your diagnosis, and whether it is a production bug, a missing dependency or codegen problem, an environment limit, or something you could not determine.
  3. **Options to resolve**, each with its trade-off, for example: fix the production code (show the change), adjust the expectation if the current behavior is actually intended, add or change a dependency, or refactor for testability. Mark the one you recommend.
  Then wait for the user's choice.
- The same applies when tests cannot compile or run (a dependency that won't resolve, failing codegen, an incompatible SDK): stop, give the reason and the options, and don't substitute a different testing approach without the user's agreement.
- If you can't run `flutter test` in the current environment, say so clearly and tell the user the exact command to run.

### 9. Report back

Briefly tell the user: which file(s) you created, what behaviors are covered (a short list), the test run result (counts of passed and failed; a report must never list skipped tests as a way of passing), and anything you couldn't test cleanly and why (e.g. hard-coded `DateTime.now()`, static singletons, untestable platform calls) with a suggested refactor.

## Things that make tests bad (avoid)

- Tests that only re-state the implementation (`verify` every single call with no outcome assertion).
- Over-mocking: mocking value objects, models, or the class under test itself.
- Shared mutable state between tests, or tests that depend on run order.
- `sleep`/real `Future.delayed` waits to "let things settle".
- One giant test that checks ten behaviors — split it so a failure name tells you what broke.
- Snapshotting whole objects when only one field matters — assert what the test is about.