### Атрибут @_assemblyVision

[Данный атрибут](https://github.com/apple/swift/blob/main/docs/ReferenceGuides/UnderscoredAttributes.md#_assemblyvision) показывает заметки компилятора для функции/метода, указывая где на исходном уровне после оптимизации находятся различные runtime вызовы и факторы, влияющие на производительность. Так же, затраты и выгоды при `inline`, где происходят `ARC` операции, каким образом генерируются generics и какие вызовы девиртуализируются.

Для использования применим атрибут к методу и соберем проект в релизной сборке:

```swift
@_assemblyVision
func loadModel(model: SDModel, computeUnit: ComputeUnits, reduceMemory: Bool) async throws {
    let fm = FileManager. default

    // A Pure call. Always profitable to inline "default argument 0 of Foundation.URL.path (percentEncoded:)"
    guard fm.fileExists(atPath: model.url.path()) else { ... }
    ...
}
```

![Assembly Vision](https://global.discourse-cdn.com/swift/original/3X/b/5/b5d913e55449f99da9c658b8c27a54a8ef8a3fb0.jpeg)

> Убедитесь, что вы отредактировали вашу схему (scheme) и в `Build Configuration` выбрали `Release`.
