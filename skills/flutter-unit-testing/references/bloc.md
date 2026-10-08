# Testing Bloc / Cubit

Use `bloc_test` if it's in `dev_dependencies` (strongly preferred). If it isn't, ask the user before adding it, or fall back to the stream-based pattern at the bottom.

```yaml
dev_dependencies:
  bloc_test: ^9.1.0
```

## States need equality

`expect:` compares emitted states with `==`. States should extend `Equatable` or be `freezed`. If they don't, use matchers in `expect:` (`isA<UserLoaded>().having(...)`) instead of instances.

## Cubit

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  late MockUserRepository repo;

  setUp(() => repo = MockUserRepository());

  group('UserCubit', () {
    test('initial state is UserInitial', () {
      expect(UserCubit(repo).state, const UserInitial());
    });

    blocTest<UserCubit, UserState>(
      'emits [loading, loaded] when load succeeds',
      setUp: () => when(() => repo.fetchUser('1'))
          .thenAnswer((_) async => const User(id: '1')),
      build: () => UserCubit(repo),
      act: (cubit) => cubit.load('1'),
      expect: () => const [
        UserLoading(),
        UserLoaded(User(id: '1')),
      ],
      verify: (_) => verify(() => repo.fetchUser('1')).called(1),
    );

    blocTest<UserCubit, UserState>(
      'emits [loading, failure] when load throws',
      setUp: () => when(() => repo.fetchUser(any())).thenThrow(Exception('x')),
      build: () => UserCubit(repo),
      act: (cubit) => cubit.load('1'),
      expect: () => [
        const UserLoading(),
        isA<UserFailure>(),
      ],
    );
  });
}
```

## Bloc (events)

Same as Cubit, but `act` adds events:

```dart
blocTest<CounterBloc, int>(
  'emits [1] when Increment is added',
  build: CounterBloc.new,
  act: (bloc) => bloc.add(Increment()),
  expect: () => [1],
);
```

## Useful blocTest parameters

- `seed: () => SomeState()` — start from a non-initial state.
- `skip: 1` — ignore the first N emitted states.
- `wait: const Duration(milliseconds: 300)` — for debounced/throttled event transformers. Prefer this over real delays elsewhere.
- `errors: () => [isA<Exception>()]` — when the bloc calls `addError`.
- `verify: (bloc) { ... }` — interaction checks after `act`.

## Mocking a bloc used by another class

```dart
class MockAuthBloc extends MockBloc<AuthEvent, AuthState> implements AuthBloc {}
// or MockCubit<State> for cubits

whenListen(
  authBloc,
  Stream.fromIterable([const Authenticated(user)]),
  initialState: const Unauthenticated(),
);
```

## Without bloc_test

```dart
test('emits loading then loaded', () async {
  final cubit = UserCubit(repo);
  addTearDown(cubit.close);

  final states = expectLater(
    cubit.stream,
    emitsInOrder([const UserLoading(), const UserLoaded(User(id: '1'))]),
  );
  await cubit.load('1');
  await states;
});
```

Set up `expectLater` **before** acting — `stream` is broadcast and won't replay earlier states.