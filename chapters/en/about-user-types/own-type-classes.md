----
title: Собственные класы типов
prevChapter: /en/about-user-types/deriving.html
nextChapter: /en/about-user-types/newtype.html
----

До этого мы рассматривали лишь стандартные классы типов. А теперь поговорим о собственных классах типов.

## Перцы

Объявим класс типов `Pepper` (перец):

```haskell
type SHU = Integer  -- SHU (Scoville Heat Units), единица жгучести перца

class Pepper p where
    color :: p -> String
    pungency :: p -> SHU
```

У этого класса два метода, `color` (цвет) и `pungency` (жгучесть). Создадим два разных перца:

```haskell
data Poblano = Poblano  -- Распространён в национальных блюдах Мексики.

data TrinidadScorpion = TrinidadScorpion  -- Самый жгучий перец в мире.

instance Pepper Poblano where
    color Poblano = "green"
    pungency Poblano = 1500

instance Pepper TrinidadScorpion where
    color TrinidadScorpion = "red"
    pungency TrinidadScorpion = 855000
```

Теперь мы можем работать с этими перцами как обычно:

```haskell
main =
    putStrLn $ show (pungency trinidad) ++ ", " ++ color trinidad
    where trinidad = TrinidadScorpion
```

## Зачем они нужны

В самом деле, разве мы не можем использовать каждый из типов перца самостоятельно? Конечно можем. Главная цель определения собственного класса типов - указание контекста типов. Определим функцию, выводящую информацию о конкретном перце:

```haskell
pepperInfo :: Pepper p => p -> String
pepperInfo pepper = show (pungency pepper) ++ ", " ++ color pepper
```

Контекст полиморфного типа `p` говорит нам о том, что эта функция предназначена только для работы с перцами. Более того, сам класс тоже может иметь контекст типа:

```haskell
class Pepper p => Chili p where
    kind :: p -> String
```

Класс типов `Chili` объявлен с контекстом `Pepper`. Следовательно, занять место полиморфного типа `p` смогут лишь те типы, которые относятся к классу `Pepper`, и никакие другие. Это позволяет ограничить набор типов, которые смогут иметь отношение к классу `Chili`.

Разумеется, контекст может состоять и из нескольких классов. В этом случае они, как обычно, перечисляются в виде кортежа:

```haskell
class (Pepper p, Capsicum p) => Chili p where
    kind :: p -> String
```

## Константы

Добавим в наш класс константу:

```haskell
type SHU = Integer

class Pepper p where
    simple :: p  -- Это константное значение, а не функция.
    color :: p -> String
    pungency :: p -> SHU
    name :: p -> String

data Poblano = Poblano String  -- Унарный конструктор вместо нульарного.

instance Pepper Poblano where
    simple = Poblano "ancho"   -- Готовим простое значение.
    color (Poblano name) = "green"
    pungency (Poblano name) = 1500
    name (Poblano name) = name

main = putStrLn $ name (simple :: Poblano)  -- Обращаемся к значению.
```

`simple` - это константное значение, к которому можно обращаться напрямую. Мы говорим: "Каждый тип, относящийся к классу `Pepper`, обязан предоставлять константу `simple` своего собственного типа". Поэтому при определении экземпляра класса `Pepper` мы готовим это константное значение:

```haskell
simple = Poblano "ancho"
```

Поскольку теперь конструктор значения `Poblano` у нас унарный, он принимает строку (например, название перца или ассоциативного блюда). Здесь мы говорим: "Когда кто-нибудь обратится к константе `simple`, принадлежащей типу `Poblano`, он получит такое значение, как если бы мы написали просто `Poblanlo "ancho"`". Таким образом, эту константу можно рассматривать как конструктор по умолчанию для значения типа `Poblano`. И это весьма удобно: если для нашего типа имеет смысл некоторое умолчальное значение, лучше задавать его прямо внутри нашего типа.

## В сухом остатке

* Главная цель определения собственного класса типов - указание контекста типов.
* Класс типов может включать в себя как функции, так и константы.


