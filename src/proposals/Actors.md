## Введение

`Concurrency` призвана обеспечить безопасную модель программирования, которая статически обнаруживает гонки данных (data races) и другие классические ошибки.
Данное введение знакомит со способом объявления concurrent tasks и предоставляет `data-race` безопасность для функций и замыканий.
Такая модель подходит для ряда шаблонов проектирования, включая `parallel maps` и concurrent callback patterns, но модель ограниченна работой с состоянием, захваченным в замыканиях.

Существуют классы предоставляющие механизм для объявления неизменяемого (mutable) состояния, доступные по всей программе.
Однако, возникают трудности при правильном использовании классов в concurrent программах, поскольку для предотвращения data races нужна ручная синхронизация.

Мы хотим предоставить возможность использовать изменяемое состояние предоставляя статическое обнаружение data races и других классических concurrency ошибок.
Модель акторов идеально подходит для задачи, описанной выше.
Каждый актор защищает свои собственные данные посредством изоляции данных, гарантируя, что только один поток будет иметь доступ к этим данным в определенный момент времени, даже если многие клиенты одновременно делают запросы к актору.
Акторы обеспечивают те же свойства безопасности гонки и памяти, что и structured concurrency, но при этом предоставляют привычные возможности абстракции и повторного использования, которыми обладают другие типы данных.

### Actors
Actor — reference type, который защищает доступ к своему изменяемому состоянию и вводит новое ключевое слово: `actor`.
Актор имеет инициализатор, методы, свойства и сабскрипты. Может быть расширен, подписываться на протоколы и поддерживает generic.
Основное различие — актор защищает своё состояние от data races. Такую защиту обеспечивает компилятор, на этапе статической проверки, с помощью набора ограничений на использование акторов и их членов экземпляра, которые в совокупности называются изоляцией акторов.

### Actor isolation

Изоляция акторов - это то, как акторы защищают свое изменяемое состояние.
Для акторов основным механизмом такой защиты является разрешение доступа к хранимым свойствам экземпляра только через self.

Ссылка на изолированное объявление актора из вне этого актора называются **cross-actor reference**.
Такие ссылки возможны в двух случаях:

1. cross-actor ссылка на неизменяемое состояние разрешена из любого места в том же модуле, где объявлен актор, поскольку после инициализации это состояние никогда не может быть изменено (ни внутри актора, ни вне его), поэтому data races отсутствуют по определению.
Ссылка на `other.accountNumber` разрешена на основе этого правила, потому что `accountNumber` объявлен с помощью `let` и имеет `value-semantic` тип данных `Int`.

2. Вторым способом для `cross-actor` ссылки — выполняется с помощью вызова асинхронной функции.
Такие вызовы асинхронной функций превращаются в «сообщения», запрашивающие у актора выполнение задачи, когда это можно сделать безопасно.
Данные сообщения хранятся в «почтовом ящике» (`mailbox`) актора и вызов асинхронной функции может быть приостановлен до тех пор, пока актор не сможет обработать соответствующее сообщение в своем почтовом ящике.
> ⚠️ Актор обрабатывает сообщения в своем почтовом ящике по одному за раз, поэтому у актора никогда не будет двух задач, выполняющих изолированный от актора код.
Это гарантирует отсутствие `data races` на изолированном изменяемом состоянии, потому `concurrency` отсутствует в любом коде, который может получить доступ к изолированному состоянию актора.

Например, если бы мы хотели внести депозит на банковский счет, мы могли бы сделать вызов метода `deposit(amount:)` на другом акторе, и данный вызов стал бы «сообщением», помещенным в «почтовый ящик» актора и вызов приостановился.
При обработки сообщений, будет вызов функции в области изоляции актора при условии, что другой код не выполняется в области этого же актора.

> ⚠️ Синхронный метод актора можно вызвать через `self.`, но `cross-actor` вызов для этого метода требует асинхронного вызова, т.е. использовать `await`.

### Cross-actor references and `Sendable`

`SE-0302` знакомит вас с протоколом `Sendable`.
Типы данных, наследующихся от `Sendable`, безопасны для общего выполнения в `concurrently` коде.
Существуют типы, которые работают вместе с `Sendable`: value-semantic типы — `Int`, `String`, value-semantic коллекции — `[String]` или `[Int: String]`, неизменяемые классы, классы которые выполняют свою собственную синхронизацию внутри (например, concurrent hash table).

Актор защищает свое изменяемое состояние, поэтому инстансы могут свободно использоваться в конкурентной среде и сам по себе актор поддерживает внутреннюю синхронизацию. 
> ⚠️ Каждый актор неявно соответствует протоколу `Sendable`

Все `cross-actor` ссылки работают с value типами.
Наш `BankAccount` содержит список владельцев, где каждый владелец является классом `Person`:

```swift
class Person {
  var name: String
  let birthDate: Date
}

actor BankAccount {
  // ...
  var owners: [Person]

  func primaryOwner() -> Person? { return owners.first }
}
```
Функцию `primaryOwner()` можно вызвать из другого актора, поэтому инстанс `Person` можно изменить откуда угодно:

```swift
if let primary = await account.primaryOwner() {
  primary.name = "The Honorable " + primary.name  // problem: concurrent mutation of actor-isolated state
}
```

Каждый не изменяемый доступ проблематичен, поскольку имя может быть изменено внутри актора в тоже время, когда исходный вызов пытается получить доступ.
Для предотвращения одновременного доступа к изолированному актору, необходимо чтобы все типы соответствовали `Sendable`.
Таким образом можно гарантировать, что никакие `cross-actor` ссылки не выйдут за изолированное состояние.
Вызов `account.primaryOwner()` выдаст ошибку:

`error: cannot call function returning non-Sendable type 'Person?' across actors`

Функцию `primaryOwner()` можно использовать в изолированной области видимости актора:

```swift
extension BankAccount {
  func primaryOwnerName() -> String? {
    return primaryOwner()?.name
  }
}
```

Функция `primaryOwnerName()` безопасна для асинхронного вызова, поскольку `String` и `Optional<String>` соответствуют `Sendable`.


