## Введение

В современной разработке на языке Swift мы часто прибегаем к использованию асинхронной разработки используя замыкания и completion handlers, но с этими API трудно работать.
Особенно заметно, когда в коде содержится много асинхронных операций, также требуется обработать ошибки и тем самым усложняется control flow между вызовами.

Данная конструкция вводит модель `coroutine` корутин в Swift.
В функции можно использовать ключевое слово `async`, позволяя вам проектировать более комплексную/сложную логику, включая асинхронные операции, использовав обычный control flow механизм.
Компилятор отвечает за преобразование асинхронный функций в набор замыканий(appropriate set of closures) и машинное представление.

Данный файл описывает семантику и проблемы, но не concurrency.
Concurrency рассматривается в введении `structured concurrency`, which associates asynchronous functions with concurrently-executing tasks and provides APIs for creating, querying, and cancelling tasks.

## Мотивация: Completion handlers не удобны

Асинхронная разработка с помощью `explicit callbacks` (также completion handlers) содержит много проблем, описанных ниже.
Введение в асинхронные функции служит решением множества проблем. С `async` функциями можно писать код в обычном стиле.

### Проблема #1: Pyramid of doom (Пирамида судьбы)

Последовательность асинхронный выражений часто требует глубоко вложенных замыканий:

```swift
// Код уходит в глубину, вправо
func processImageData1(completionBlock: (_ result: Image) -> Void) {
    loadWebResource("dataprofile.txt") { dataResource in
        loadWebResource("imagedata.dat") { imageResource in
            decodeImage(dataResource, imageResource) { imageTmp in
                dewarpAndCleanupImage(imageTmp) { imageResult in
                    completionBlock(imageResult)
                }
            }
        }
    }
}

processImageData1 { image in
    display(image)
}
```

Такая пирамида судьбы значительно усложняет написание, чтение и отслеивание выполняемого кода. Допольнительно, используется стопка из замыканий.

### Проблема #2: Обработка ошибок

Callbacks делают очень сложной обработку ошибок. В Swift 2 появилась модель обработки ошибок для синхронного кода, но интерфейсы на основе callbacks не получают от нее никаких преимуществ:

```swift
// (2a) Using a `guard` statement for each callback:
func processImageData2a(completionBlock: (_ result: Image?, _ error: Error?) -> Void) {
    loadWebResource("dataprofile.txt") { dataResource, error in
        guard let dataResource = dataResource else {
            completionBlock(nil, error)
            return
        }
        loadWebResource("imagedata.dat") { imageResource, error in
            guard let imageResource = imageResource else {
                completionBlock(nil, error)
                return
            }
            decodeImage(dataResource, imageResource) { imageTmp, error in
                guard let imageTmp = imageTmp else {
                    completionBlock(nil, error)
                    return
                }
                dewarpAndCleanupImage(imageTmp) { imageResult, error in
                    guard let imageResult = imageResult else {
                        completionBlock(nil, error)
                        return
                    }
                    completionBlock(imageResult)
                }
            }
        }
    }
}

processImageData2a { image, error in
    guard let image = image else {
        display("No image today", error)
        return
    }
    display(image)
}
```

### Проблема #3: Выполнение условий сложно и чревато ошибками

Выполнять условия асинхронной функции - это большая проблема.
Например, предположим, что нам нужно «смахнуть» изображение после его получения, но иногда нам нужно сделать асинхронный вызов для декодирования изображения, прежде чем мы сможем выполнить свитчинг/условие.
Возможно, лучшим подходом к структурированию этой функции является запись кода `swizzling` в вспомогательное замыкание, которое условно перехватывается в обработчике завершения:


```swift
func processImageData3(recipient: Person, completionBlock: (_ result: Image) -> Void) {
    let swizzle: (_ contents: Image) -> Void = {
      // ... continuation closure that calls completionBlock eventually
    }
    if recipient.hasProfilePicture {
        swizzle(recipient.profilePicture)
    } else {
        decodeImage { image in
            swizzle(image)
        }
    }
}
```

### Проблема  #4: Легко сделать много ошибок

Можно легко выйти из функции раньше без правильного вызова  `completion handler`
Если мы забыли это сделать, то сложнее дебажить данную функцию:

```swift
func processImageData4a(completionBlock: (_ result: Image?, _ error: Error?) -> Void) {
    loadWebResource("dataprofile.txt") { dataResource, error in
        guard let dataResource = dataResource else {
            return // <- забыли вызвать блок
        }
        loadWebResource("imagedata.dat") { imageResource, error in
            guard let imageResource = imageResource else {
                return // <- забыли вызвать блок
            }
            ...
        }
    }
}
```

К счастью, вызов оператора `guard` в некоторой степени защищает от возвращаемого значения, но это не всегда актуально.

### Проблема #5: Большинство completion handlers не удобны, поэтому множество APIs являются синхронными

Трудно оценить количество, но некоторые разработчики считают, чтобы использование асинхронных API совмество с completion handlers значительно затруднительно, поэтому используют синхронный подход.
Это приводит к проблемам с производительностью, отзывчивостью и плавности UI.

## Решение: async/await

Асинхронные функции, известные как `async/await`, позволяют писать код в том же стиле, как и обычный синхронный код.
Более того, это сразу решает описанные проблемы выше, позволяя разработчикам использовать те же конструкции языка, доступные для синхронного кода.