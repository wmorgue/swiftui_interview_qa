Новые архитектуры в [Conditional Compilation Block](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/statements/#Conditional-Compilation-Block):

```swift
#if arch(x86_64) || arch(arm64) || arch(s390x) || arch(powerpc64) || arch(powerpc64le)
    ...
#elseif arch(i386) || arch(arm) || arch(arm64_32)
    ...
#else
    #error ("Some error")
#endif
```

WebAssembly: `#if os(WASI)`
Проверка на Foundation: `#if FOUNDATION_FRAMEWORK`

---

## Expression Macros

[Макросы позволяют](https://github.com/apple/swift-evolution/blob/main/proposals/0382-expression-macros.md) расширить возможности Swift, позволяя писать более выразительный код, библиотеки и избегать boilerplate.

Макрос объявляется с помощью `#` , как и функции могут принимать параметры и возвращать результат:

```swift
// вначале реализация, она скрыта

// присваивание макроса к его реализации
@freestanding(expression)
macro stringify<T>(_: T) -> (T, String) = #externalMacro(module: "ExampleMacros", type: "StringifyMacro")

// объявляем макрос
#stringify(x + y)

// a: Double = 1.0, b: Double = 2.0
let (a, b): (Double, String) = #stringify(1 + 2)
```

Атрибут `@freestanding(expression)` применяется только к макросам. Означает, что макрос является выражением макроса (macro is an expression macro).
Термин «freestanding» взят из [документа freestanding атрибута](https://github.com/apple/swift-evolution/blob/main/proposals/0397-freestanding-declaration-macros.md) и описывает использование макроса помеченным символом `#` .

## Attached Macros

[Прикрепленные макросы](https://github.com/apple/swift-evolution/blob/main/proposals/0389-attached-macros.md) называются так, потому что «прикрепляются» к конкретному объявлению. Реализуются с помощью синтаксиса кастомных атрибутов: `@AddCompletionHandler`.
У таких макрос могут быть роли, отвечающие за определенное поведение.

Объявляются с помощью ключевого слова `macro`, имеют проверку типа данных, что позволяет настраивать кастомное поведение:

```swift
// вначале реализация, далее присваивание и вызов.
// детали реализации скрыты

// присваиваем макрос к его реализации
@attached(peer, names: overloaded)
macro AddCompletionHandler(parameterName: String = "completionHandler")

// используем прикрепленный макрос
@AddCompletionHandler(parameterName: "onCompletion")
func fetchAvatar(_ username: String) async -> Image? { ... }
```


> ‼️ [Примеры с макросами](https://github.com/DougGregor/swift-macro-examples) доступны в репозитории.

## Package Manager Support for Custom Macros


[Данное введение добавляет поддержку макросов](https://github.com/apple/swift-evolution/blob/main/proposals/0394-swiftpm-expression-macros.md) в таргет `SwiftPM`:

```swift
import PackageDescription
import CompilerPluginSupport

let package = Package(
    name: "MacroPackage",
    dependencies: [
        .package(url: "https://github.com/apple/swift-syntax", from: "509.0.0"),
    ],
    targets: [
        .macro(name: "MacroImpl",
               dependencies: [
                   .product(name: "SwiftSyntaxMacros", package: "swift-syntax"),
                   .product(name: "SwiftCompilerPlugin", package: "swift-syntax")
               ]),
        .target(name: "MacroDef", dependencies: ["MacroImpl"]),
        .executableTarget(name: "MacroClient", dependencies: ["MacroDef"]),
        .testTarget(name: "MacroTests", dependencies: ["MacroImpl"]),
    ]
)
```

## Swift SDKs for Cross-Compilation

Появилась новая CLI команда `swift sdk` и [кросс компиляция для нескольких таргетов](https://github.com/apple/swift-evolution/blob/main/proposals/0387-cross-compilation-destinations.md).

##  `if` and `switch` expressions

Появилась [новая форма записи для `if` и `switch` паттернов](https://github.com/apple/swift-evolution/blob/main/proposals/0380-if-switch-expressions.md):

1. В качестве возвращаемого значения в функции, проперти и замыкании
2. Объявлять переменные
3. Присваивать значение переменной

```swift
let x: Double? = if p { nil } else { 2.0 }

let y: Float = switch x.value {
    case 0..<0x80: 1
    case 0x80..<0x0800: 2.0
    case 0x0800..<0x1_0000: 3.0
    default: 4.5
}

func test<T>(_: (Int?) -> T) {}

test { x -> Int? in
  guard let x { return nil }
  return x
}

// equivalent to let x = foo.map(process) ?? someDefaultValue
let x = if let foo { process(foo) } else { someDefaultValue }

let foo: String = do {
    try bar()
} catch {
    "Error \(error)"
}
```

## Custom Actor Executors

[Появился базовый механизм для настройки actor executors](https://github.com/apple/swift-evolution/blob/main/proposals/0392-custom-actor-executors.md).

Предоставляя экземпляр executors, акторы могут влиять на то, «где» они будут выполнять задачу, которую выполняют, сохраняя при этом изоляцию акторов, гарантированную моделью акторов.

Разработчикам предоставляется возможность реализовать простые `serial executors`, которые можно использовать с акторами, гарантируя выполнение кода на акторе с кастомным `executor`, который запускается в подходящем потоке или контексте.

## Noncopyable structs and enums

Все существующие типы данных в Swift являются **copyable**, что означает возможность создать несколько одинаковых, не уникальных и взаимозаменяемых типов данных.
Копируемые структуры и перечисления не лучшие кандидаты для уникальных ресурсов.
В свою очередь, классы являются уникальных ресурсом, поскольку объект имеет уникальную сущность после инициализации и только reference value могут быть копируемыми, но классы всегда требуют совместного владения ресурсом, что приводит к некоторым расходам, например: аллокации хипа и подсчета ссылок. Помимо расходов, совместных доступ усложняет использование и добавляет небезопасное использование.

На данный момент в Swift нет механизмов для создания уникального владения (unique ownership).

[Поэтому для структур и перечислений предлагается использовать объявлять тип данных как `noncopyable`, используя новый синтаксис ~Copyable.](https://github.com/apple/swift-evolution/blob/main/proposals/0390-noncopyable-structs-and-enums.md)
`Noncopyable` типы данных имеют уникальное владение (unique ownership) и никогда не могут быть копируемыми (за исключением неявного копирования).
Поскольку такие типы данных имееют уникальных идентификаторы, то можно использовать `deinit`, как в классах и акторах, которые автоматически выполняются в конце жизни инстанса.

Например:
```swift
struct FileDescriptor: ~Copyable {
  private var fd: Int32

  init(fd: Int32) { self.fd = fd }

  func write(buffer: Data) {
    buffer.withUnsafeBytes { 
      write(fd, $0.baseAddress!, $0.count)
    }
  }

  deinit {
    close(fd)
  }
}
```

Как и классы, инстансы данной структуры предоставляют доступ к управлению дескриптора файла,  автоматически закрывая его (`deinit`).
В отличии от класса, нет необходимости аллоцировать объект. Простая структура, содержащая ID дескриптора файла, должна хранится в стеке.

## Value and Type Parameter Packs для variadic generic programming

https://github.com/apple/swift-evolution/blob/main/proposals/0393-parameter-packs.md#motivation
