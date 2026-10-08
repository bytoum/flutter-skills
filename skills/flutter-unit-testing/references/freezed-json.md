# Testing freezed and json_serializable models

## What to test (and what not to)

Generated code is already tested by its authors. Don't write tests that only prove `copyWith` or `==` work.

Test **your** decisions:
- JSON mapping: `@JsonKey(name:)`, defaults, nullable or missing fields, custom converters, enum values.
- Custom getters and methods added to the class.
- Union / sealed class behavior that your code switches on.
- Validation logic in factory constructors or `assert`s.

## JSON round-trips with fixtures

Store realistic API payloads under `test/fixtures/`. `flutter test` runs from the package root, so relative paths work:

```dart
import 'dart:convert';
import 'dart:io';

Map<String, dynamic> fixture(String name) =>
    jsonDecode(File('test/fixtures/$name').readAsStringSync())
        as Map<String, dynamic>;
```

```dart
group('UserDto JSON', () {
  test('fromJson reads snake_case keys', () {
    final dto = UserDto.fromJson(fixture('user.json'));

    expect(dto, const UserDto(id: '1', fullName: 'Ada', role: Role.admin));
  });

  test('toJson writes snake_case keys', () {
    const dto = UserDto(id: '1', fullName: 'Ada', role: Role.admin);

    expect(dto.toJson(), {'id': '1', 'full_name': 'Ada', 'role': 'admin'});
  });

  test('round-trip preserves all fields', () {
    final json = fixture('user.json');

    expect(UserDto.fromJson(json).toJson(), json);
  });

  test('uses default when optional field is missing', () {
    final dto = UserDto.fromJson({'id': '1', 'full_name': 'Ada'});

    expect(dto.role, Role.member); // @Default(Role.member)
  });

  test('unknown enum value falls back', () {
    final dto = UserDto.fromJson({'id': '1', 'full_name': 'A', 'role': 'owner'});

    expect(dto.role, Role.unknown); // @JsonKey(unknownEnumValue: Role.unknown)
  });

  test('throws when required field is missing', () {
    expect(
      () => UserDto.fromJson({'full_name': 'Ada'}),
      throwsA(isA<TypeError>()),
    );
  });
});
```

Notes:
- Nested objects: if `toJson()` returns nested model instances instead of maps, the round-trip equality fails. That means `explicitToJson: true` is missing. Report it rather than working around it in the test.
- Missing required fields usually throw `TypeError`, but `checked: true` makes them throw `CheckedFromJsonException`. Match whatever the project's config produces.
- Custom `JsonConverter`s (dates, money, colors) deserve their own small test group: `fromJson` and `toJson` with edge values (null, timezone, zero).

## freezed unions / sealed classes

```dart
@freezed
sealed class PaymentResult with _$PaymentResult {
  const factory PaymentResult.success(String receiptId) = PaymentSuccess;
  const factory PaymentResult.declined(String reason) = PaymentDeclined;
}
```

Assert on the variant type and its fields:

```dart
expect(result, isA<PaymentSuccess>().having((r) => r.receiptId, 'receiptId', 'r-1'));
expect(result, const PaymentResult.declined('insufficient_funds'));
```

Equality from freezed makes these model instances work directly in `expect`, `blocTest` `expect:` lists, and Riverpod `AsyncData(...)`.

## Custom getters and methods

They need a private constructor (`const User._();`). Test them like normal methods:

```dart
test('initials uses first letter of each name', () {
  expect(const User(firstName: 'Ada', lastName: 'Lovelace').initials, 'AL');
});
```

## Gotchas

- Check the freezed version in `pubspec.yaml`. freezed 3 drops `when`/`map` in favor of Dart 3 `switch` patterns, so match the project's style in any test helper code.
- Run `dart run build_runner build --delete-conflicting-outputs` if `.freezed.dart` or `.g.dart` files are missing or out of date. Compile errors mentioning `_$User` almost always mean stale generated code.