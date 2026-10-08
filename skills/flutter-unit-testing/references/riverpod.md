# Testing Riverpod

Unit-test providers with a `ProviderContainer` — no widgets needed. Check the Riverpod major version in `pubspec.yaml` (2.x vs 3.x) and whether the project uses code generation (`riverpod_annotation`, `@riverpod`); match the existing style.

## Container helper

```dart
ProviderContainer createContainer({
  List<Override> overrides = const [],
}) {
  final container = ProviderContainer(overrides: overrides);
  addTearDown(container.dispose);
  return container;
}
```

On Riverpod 3.x, `ProviderContainer.test(overrides: [...])` does this for you (auto-disposes) — use it if available.

## Overriding dependencies

```dart
final repo = MockUserRepository();
final container = createContainer(overrides: [
  userRepositoryProvider.overrideWithValue(repo),
]);
```

Override the *dependency* providers, not the provider under test.

## Simple / Future providers

```dart
test('userProvider returns the user from the repository', () async {
  when(() => repo.fetchUser('1')).thenAnswer((_) async => const User(id: '1'));
  final container = createContainer(overrides: [
    userRepositoryProvider.overrideWithValue(repo),
  ]);

  final user = await container.read(userProvider('1').future);

  expect(user, const User(id: '1'));
});

test('exposes AsyncError when repository throws', () async {
  when(() => repo.fetchUser(any())).thenThrow(Exception('x'));
  final container = createContainer(overrides: [
    userRepositoryProvider.overrideWithValue(repo),
  ]);

  await expectLater(
    container.read(userProvider('1').future),
    throwsA(isA<Exception>()),
  );
  expect(container.read(userProvider('1')), isA<AsyncError<User>>());
});
```

## Notifier / AsyncNotifier

```dart
test('increment updates state', () {
  final container = createContainer();

  container.read(counterProvider.notifier).increment();

  expect(container.read(counterProvider), 1);
});
```

## Recording state transitions

Use `container.listen` with a listener to capture every emission (works with mocktail or a simple list):

```dart
test('emits loading then data when refresh is called', () async {
  final container = createContainer(overrides: [
    userRepositoryProvider.overrideWithValue(repo),
  ]);
  when(() => repo.fetchUsers()).thenAnswer((_) async => [user]);

  final states = <AsyncValue<List<User>>>[];
  container.listen(
    usersNotifierProvider,
    (_, next) => states.add(next),
    fireImmediately: true,
  );

  await container.read(usersNotifierProvider.notifier).refresh();

  expect(states.first, isA<AsyncLoading>());
  expect(states.last, AsyncData([user]));
});
```

## Gotchas

- `autoDispose` providers may be disposed between reads in a test. Keep them alive with `container.listen(provider, (_, __) {})` before reading.
- Always `await` the `.future` for async providers before asserting on data.
- Never reuse a container across tests.