----
title: Функциональные цепочки
prevChapter: /ru/about-functions/higher-order-functions.html
nextChapter: /ru/about-functions/functions-and-operators.html
----

В отношении функций часто можно сказать: "Один в поле не воин". В этой главе мы рассмотрим два удобных способа комбинирования функций. Сразу скажу, что комбинированию функций в Haskell уделено очень большое значение.

## Пример с URL

Известно, что вид URL обязан соответствовать особым правилам. Но в реальной жизни это не всегда так, поэтому иногда URL необходимо приводить к правильному виду. Вот как это может выглядеть:

```haskell
module Main where

import Data.Char
import Data.List

addPrefix :: String -> String
addPrefix url =
    if prefix `isPrefixOf` url then url else prefix ++ url
    where prefix = "http://"

encodeAllSpaces = replace " " "%20"  -- Заменяем все пробелы на %20.

makeItLowerCase = map toLower  -- Переводим символы строки в нижний регистр.      

main =
    putStrLn (addPrefix (encodeAllSpaces (makeItLowerCase url)))
    where url = "www.SITE.com/test me/Start page"
```

Вывод будет таким:

```bash
http://www.site.com/test%20me/start%20page
```

Вы, вероятно, обратили внимание на две необычные строки:

```haskell
encodeAllSpaces = replace " " "%20"

makeItLowerCase = map toLower
```

Что это? Вроде бы похоже на функции, определённые без объявления, но где же тут аргументы? А они здесь не нужны. Чтобы стало понятнее, напишем одну из них с аргументом:

```haskell
makeItLowerCase url = map toLower url
```

Это - полная форма записи функции. Но мы можем сократить её, убрав аргумент. Почему? Потому что такой инструкцией:

```haskell
makeItLowerCase = map toLower
```

мы объявляем: "Всё, теперь `makeItLowerCase` - это псевдоним для записи `map toLower`. Поэтому везде, где мы напишем `makeItLowerCase arg`, мы будем подразумевать запись `map toLower arg`". То есть `makeItLowerCase` просто заменяется записью `map toLower`.

Итак, у нас имеются три функции, каждая из которых делает с нашим URL простую исправительную операцию: `makeItLowerCase` переводит все символы в нижний регистр, `encodeAllSpaces` заменяет пробелы строкой `"%20"`, `addPrefix` добавляет префикс, если таковой отсутствует. И мы строим из этих трёх функций цепочку, на входе которой - неправильный URL, а на выходе - исправленный URL. Рассмотрим эту цепочку поближе:

```haskell
addPrefix (encodeAllSpaces (makeItLowerCase url))
```

Каждая из функций принимает на вход URL и возвращает уже обработанный ею URL, поступающий на вход следующей функции.

И всё бы хорошо, но есть в такой цепочке один минус - многовато круглых скобок (да простят меня программисты Lisp). Проблема усугубилась бы, если бы функций-исправителей было не три, а больше. Существует два способа сделать такую цепочку красивее.

## Композиция функций

В Haskell существует особая операция - композиция функций (function composition). Выглядит эта операция как точка. Её назначение - компоновать функции в цепочку. Вот так:

```haskell
(addPrefix . encodeAllSpaces . makeItLowerCase) url
```

Наши три функции объединяются в цепочку, на вход которой подаётся неправильный `url`. Результат такой композиции будет точно таким же, как если бы мы написали так:

```haskell
addPrefix (encodeAllSpaces (makeItLowerCase url))
```

Можно сказать, что композиция создала стек из наших трёх функций: перечислены они слева направо, а вызываться будут справа налево. Таким образом, строка `url` как бы едет по конвейеру, заезжая в него с правого края и выезжая с левого. А чтобы не запутаться с порядком, мысленно подставьте на место точки фразу "будет вызвана после":

```Haskell
(addPrefix "будет вызвана после" encodeAllSpaces "будет вызвана после" makeItLowerCase) url
```

## Функция применения

Есть ещё одна особая операция, подобная композиции, а именно применение функций (function application). Иногда её называют "функцией аппликации". Выглядит она как значок доллара. Её назначение такое же - компоновать функции в цепочку. Вот так:

```haskell
addPrefix $ encodeAllSpaces $ makeItLowerCase url
```

Здесь мы обошлись вообще без скобок. И такое написание также аналогично исходному:

```haskell
addPrefix (encodeAllSpaces (makeItLowerCase url))
```

Здесь тоже получился стек из функций: перечислены слева направо, а вызываются справа налево. Такой вызов справа налево называют ещё правоассоциативным (right-associative).

## Вместе

Вероятно, вам интересно, а в чём же разница между этими двумя способами? Ведь и первый и второй предназначены для комбинирования функций.

Главное различие состоит в том, что конструкция применения позволяет объединять не только функции, но также функцию с её аргументом:

```haskell
main = print $ "Hi master!"
```

Композиция не позволила бы нам проделать такой фокус. Но вы спросите, зачем это нужно?

Рассмотрим вот такой вызов:

```haskell
main = print ("Hi master '" ++ name ++ "', have a nice day!")
```

Функция `print` готова работать исключительно с одним аргументом, поэтому три литерала, объединяющиеся в один, необходимо взять в скобки. Однако значительно удобнее написать так:

```haskell
main = print    $           "Hi master '" ++ name ++ "', have a nice day!"
       функция  применение  ------- выражение, порождающее аргумент ------
```

Мы избавились от скобок, объединив функцию и её аргумент в маленькую цепочку. Именно благодаря такому свойству композиция и применение часто используются вместе:

```haskell
addPrefix . encodeAllSpaces . makeItLowerCase $ url
```

Точка объединяет функции, а доллар привязывает их комбинацию к единственному аргументу. Кстати, в реальных Haskell-проектах чаще всего используется именно такая "бесскобочная" форма вызова. И в последующих главах мы будет делать так же.

## В сухом остатке

* Композиция функций компонует функции в цепочку.
* Применение функций также компонует функции в цепочку, а также позволяет объединять функцию с её аргументом.
* Совместное использование применения и композиции упрощает код, избавляя нас от лишних круглых скобок.


