# Mocking reference

Pick the section matching the project's library (see SKILL.md step 2). Don't mix both libraries in one test file.

## mocktail

```yaml
dev_dependencies:
  mocktail: ^1.0.0
```

```dart
import 'package:mocktail/mocktail.dart';

class MockUserApi extends Mock implements UserApi {}
class FakeUser extends Fake implements User {}

void main() {
  // Needed once per type used with any()/captureAny() for non-primitive args.
  setUpAll(() {
    registerFallbackValue(FakeUser());
  });

  late MockUserApi api;
  setUp(() => api = MockUserApi());

  test('example', () async {
    // Stub: note the closure syntax
    when(() => api.getUser(any())).thenAnswer((_) async => const User(id: '1'));
    when(() => api.isOnline).thenReturn(true);                 // sync getter
    when(() => api.save(any())).thenThrow(ApiException(500));  // throw
    when(() => api.watch()).thenAnswer((_) => Stream.value(1)); // stream

    // ... act ...

    // Verify
    verify(() => api.getUser('1')).called(1);
    verifyNever(() => api.save(any()));
    verifyNoMoreInteractions(api);

    // Capture arguments
    final captured = verify(() => api.save(captureAny())).captured;
    expect((captured.single as User).id, '1');

    // Named args
    when(() => api.search(query: any(named: 'query'))).thenAnswer((_) async => []);
  });
}
```

Gotchas:
- Use `thenAnswer` (not `thenReturn`) for `Future` and `Stream` return values.
- Unstubbed methods returning non-nullable types throw — stub every call the code path makes, including `void`-returning async methods: `when(() => api.log(any())).thenAnswer((_) async {});`.
- `any()` on a custom type without `registerFallbackValue` throws a clear error — add the fallback in `setUpAll`.

## mockito

```yaml
dev_dependencies:
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

```dart
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

import 'user_repository_test.mocks.dart';

@GenerateNiceMocks([MockSpec<UserApi>()])
void main() {
  late MockUserApi api;
  setUp(() => api = MockUserApi());

  test('example', () async {
    when(api.getUser(any)).thenAnswer((_) async => const User(id: '1'));
    when(api.isOnline).thenReturn(true);
    when(api.save(any)).thenThrow(ApiException(500));
    when(api.search(query: anyNamed('query'))).thenAnswer((_) async => []);

    // ... act ...

    verify(api.getUser('1')).called(1);
    verifyNever(api.save(any));

    final captured = verify(api.save(captureAny)).captured;
    expect((captured.single as User).id, '1');
  });
}
```

Then run: `dart run build_runner build --delete-conflicting-outputs`

Gotchas:
- No closures: `when(api.getUser(any))`, not `when(() => ...)`.
- `any` is a getter, not a function call. Named args use `anyNamed('name')`.
- Match the project's existing annotation (`@GenerateMocks` vs `@GenerateNiceMocks`). Nice mocks return defaults for unstubbed calls; strict ones throw.
- The `.mocks.dart` file must be regenerated whenever the mocked class's signature changes.

## Fakes (no library)

For simple dependencies, a hand-written fake is often clearer and avoids a dependency:

```dart
class InMemoryUserStore implements UserStore {
  final _users = <String, User>{};
  @override
  Future<User?> read(String id) async => _users[id];
  @override
  Future<void> write(User user) async => _users[user.id] = user;
}
```

Prefer fakes when the test cares about *state* (what ended up stored), mocks when it cares about *interaction* (was the network called, with what).