# Testing Provider / ChangeNotifier

With `provider`, the logic lives in `ChangeNotifier` (or `ValueNotifier`) classes. Unit-test those classes directly — no `ChangeNotifierProvider` or widgets needed.

## Basic pattern

```dart
void main() {
  late MockCartRepository repo;
  late CartModel model;

  setUp(() {
    repo = MockCartRepository();
    model = CartModel(repo);
  });

  tearDown(() => model.dispose());

  group('CartModel', () {
    test('starts empty', () {
      expect(model.items, isEmpty);
      expect(model.total, 0);
    });

    test('add puts item in cart and notifies listeners', () {
      var notifications = 0;
      model.addListener(() => notifications++);

      model.add(const Item(id: 'a', price: 10));

      expect(model.items, [const Item(id: 'a', price: 10)]);
      expect(model.total, 10);
      expect(notifications, 1);
    });
  });
}
```

Counting notifications matters: it catches both missing `notifyListeners()` calls (UI won't update) and redundant ones (extra rebuilds).

## Async methods with loading flags

Record the state seen by each notification to check the full sequence:

```dart
test('load sets isLoading true then false and fills items', () async {
  when(() => repo.fetch()).thenAnswer((_) async => [item]);

  final loadingStates = <bool>[];
  model.addListener(() => loadingStates.add(model.isLoading));

  await model.load();

  expect(loadingStates, [true, false]);
  expect(model.items, [item]);
  expect(model.error, isNull);
});

test('load stores error and clears loading when repository throws', () async {
  when(() => repo.fetch()).thenThrow(Exception('offline'));

  await model.load();

  expect(model.isLoading, isFalse);
  expect(model.error, isNotNull);
});
```

## Gotchas

- Calling `notifyListeners()` after `dispose()` throws. If the model can finish an async call after disposal, add a test that disposes mid-call and confirms nothing throws (if the code doesn't guard against it, report it as a bug).
- `ProxyProvider` / `context.read` wiring is widget-level — out of scope for unit tests. Test the classes being wired instead.
- For `ValueNotifier<T>`, assert on `.value` and use the same listener-count approach.