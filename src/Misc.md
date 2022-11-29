Данный файл содержит список некоторых «скрытых» функций, которые можно найти в исходном коде компилятора.
Поисковый запрос на GitHub: `repo:apple/swift public func path:/^stdlib\/public\/core\//`

```swift
print(_canBeClass(MainActor.self))
print(_undefined("Unimplemented"))
print(_isStdlibDebugChecksEnabled())
print(_isDebugAssertConfiguration())
print(_isPowerOf2(4))
print(_SwiftStdlibVersion.current)
```

Для того, чтобы вывести имя типа по средствам языка Swift, вначале получим мангленный символ:

```swift
func get_type_by_name<T>(_ t: T.Type) -> Any.Type? {
  let mangledType: String? = _mangledTypeName(t)
  
  guard mangledType != nil else {
    return nil
  }
  let typeName: Any.Type? = _typeByName(mangledType!)

  return typeName
}
```

Выведем тип для `_SwiftStdlibVersion` и опционального Int8:

```swift
print(get_type_by_name(_SwiftStdlibVersion.self))
print(get_type_by_name(Optional<Int8>.self))
```

Более комплексный пример:

```swift
protocol P {}
struct A: P {}
struct B: P {}
struct MV<T:P> {}

print(get_type_by_name(MV<B>.self))
```
