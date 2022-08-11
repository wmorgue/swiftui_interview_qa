# Swift iOS Interview Questions & Answers

1. Что такое вывод типов (type inference)?

[Возможность компилятора][type_inference] самому логически вывести тип значения у выражения:

```swift
// константа `myStr` имеет тип данных `String`
let myStr = "Hello, World!"

// явно выводим тип данных
let myStr: String = "Hello, World!"
```

2. Что такое Generics (обобщенное программирование)?

[Парадигма программирования][generics], заключающаяся в таком описании данных и алгоритмов, которое можно применять к различным типам данных, не меняя само описание. Парадигма позволяет писать код, который избавляет от дублирования и выражает цели в ясном, абстрактном подходе:

```swift
// T — Generic тип
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
  (a, b) = (b, a)
}

var min = UInt8.min // 0
var max = UInt8.max // 255

swapTwoValues(&min, &max)

var first = "first"
var second = "second"

swapTwoValues(&first, &second)
```

3. Что такое Протоколы (protocols)?

[Протокол][protocols] - это набор методов, свойств (проперти) и других требований, которые соответствуют определенной задаче или функциональности. Протокол может быть принят классом, структурой или перечислением для реализации (implementation) этих требований.

```swift
protocol Human {
  var givenName: String { get }
}

struct Person: Human {
  var givenName: String
}

let nick = Person(givenName: "Nick") // nick.givenName это "Nick"
```

> ⚠️ Протокол не содержит реализаций (implementation) свойств и методов. Проперти не содержит данных, а тело метода пустое. Тип, который наследуется от протокола, должен реализовывать все проперти и методы, которые присутствуют в данном протоколе.


4. Что такое Tuples (тупл, кортёж)?

Список типов, который разделен запятыми и заключен в круглые скобки.

```swift
let coordinates: (Int, Int) = (2, 3)

var someTuple = (top: 10, bottom: 12)  // someTuple is of type (top: Int, bottom: Int)
someTuple = (top: 4, bottom: 42) // OK: names match
someTuple = (9, 99)              // OK: names are inferred
someTuple = (left: 5, right: 5)  // Error: names don't match
```

5. Что такое Optional? (опциональное значение)

Тип данных, который предоставляет обернутое значение или `nil` | `.none`, отсутсвие значения. Иными словами, значение может быть или не может. Можно записать как `Optional<Int>`, но предпочтительна более коротка форма `Int?`:

```swift
let shortForm: Int? = Int("42")
let longForm: Optional<Int> = Int("42")

let number: Int? = Optional.some(42)
let noNumber: Int? = Optional.none
print(noNumber == nil) // true
```

Развернуть (получить) значение можно с помощью:
- `Optional Binding` (if let, guard let, switch)

```swift
if let starPath = imagePaths["star"] {
    print("The star image is at '\(starPath)'")
} else {
    print("Couldn't find the star image")
}
```

- `Optional Chaining` используя опциональный постфикс оператор (postfix ?)

```swift
if imagePaths["star"]?.hasSuffix(".png") == true {
    print("The star image is in PNG format")
}
```

- `Nil-Coalescing Operator ??`, предоставляя дефолтное значение

```swift
let defaultImagePath = "/images/default.png"
let heartPath = imagePaths["heart"] ?? defaultImagePath
print(heartPath)
```

- `Unconditional (Force) Unwrapping !`

```swift
let number = Int("42")!
print(number)
```

> ⚠️ Принудительная развертка выражения со значением `nil` приведет к `runtime error`.


6. Что такое `Inout` и как его использовать?

По умолчанию все параметры в функциях являются неизменяемыми константами (`immutable let`). Это означает, что ты не можешь изменить значение параметра, т.е.  если ты захочешь изменить значение параметра функции в теле самой функции, то получишь ошибку компиляции. Если возникла такая необходимость, то используй ключевое слово `inout` между параметром и типом данных:

```swift
func reverseName(_ name: inout String) {
  name = "github.com/" + String(name.reversed())
}

var githubUser = "@wmorgue"
print(githubUser) // @wmorgue
reverseName(&githubUser) // github.com/eugromw@
print(githubUser)
```

> ⚠️ Ты не можешь установить дефолтное значение `_ name: inout String = "@wmorgue"`.
Когда передаешь значение в `inout` параметр, то используй амперсанд `&` перед выражением.


7. Разница между Self и self?

`Self` указывает на тип протокола, класса, структуры.

`self` указывает на значение, которое оно содержит.


<!-- Link's section  -->
[type_inference]: https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html#ID322
[generics]: https://docs.swift.org/swift-book/LanguageGuide/Generics.html
[protocols]: https://docs.swift.org/swift-book/LanguageGuide/Protocols.html