# Swift iOS Interview Questions And Answers

1. Что такое вывод типов (type inference) ?

[Возможность компилятора][type_inference] самому логически вывести тип значения у выражения:

```swift
// константа `myStr` имеет тип данных `String`
let myStr = "Hello, World!"

// явно выводим тип данных
let myStr: String = "Hello, World!"
```

2. Что такое Generics (обобщенное программирование) ?

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

3. Что такое Протоколы (protocols) ?

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

> ⚠️ Важно: Протокол не содержит реализаций (implementation) свойств и методов. Проперти не содержит данных, а тело метода пустое. Тип, который наследуется от протокола, должен реализовывать все проперти и методы, которые присутствуют в данном протоколе.


<!-- 4. Что такое Tuples (тупл, кортёж) ? -->



<!-- Link's section  -->
[type_inference]: https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html#ID322
[generics]: https://docs.swift.org/swift-book/LanguageGuide/Generics.html
[protocols]: https://docs.swift.org/swift-book/LanguageGuide/Protocols.html
