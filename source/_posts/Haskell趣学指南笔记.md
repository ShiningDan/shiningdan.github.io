---
title: Haskell趣学指南笔记
date: 2018-01-24 14:45:35
categories: coding
tags:
  - Haskell
---

本文为[《Haskell 趣学指南》](https://learnyoua.haskell.sg/content/zh-cn/ch01/introduction.html)笔记

<!--more-->

## 简介

Haskell 与其他语言不同，是一门纯粹函数式编程语言 (purely functional programming language)。

在纯粹函数式编程语言中，你不是像命令式语言那样命令电脑「要做什么」，而是通过用函数来**描述出问题「是什么」**。如「阶乘是指从1到某个数的乘积」，「一个串列中数字的和」是指把第一个数字跟剩余数字的和相加。

变量 (variable) 一旦被指定，就不可以更改了。所以说，在纯粹函数式编程语言中的**函数能做的唯一事情就是利用参数计算结果，不会产生所谓的"副作用 (side effect)"**。一开始会觉得这限制很大，不过这也是他的优点所在：若以同样的参数调用同一个函数两次，得到的结果一定是相同。

Haskell 是**惰性 (lazy) 的。也就是说若非特殊指明，函数在真正需要结果以前不会被求值**。

Haskell 是静态类型 (statically typed) 的。

## 开始使用 Haskell

Haskell 杜宇类型的要求很高：

`5+"llama"`，ghci 提示说 "llama" 并不是数值类型

运行 `True==5`，ghci 就会提示类型不匹配。

`+` 运算符要求两端都是数值
`==` 运算符仅对两个可比较的值可用。

在 Haskell 中，函数调用的形式是函数名，空格，空格分隔的参数表：

```
ghci> min 3.4 3.2
3.2
```

如果某函数有两个参数，也可以用中缀函数的形式调用它。取两个整数相除所得商的 div 函数, div 92 10 可得 9，但这种形式不容易理解：究竟是哪个数是除数，哪个数被除？使用中缀函数的形式就更清晰了：

```
92 `div` 10
```

在 Haskell 中，函数的调用使用空格，例如 `bar (bar 3)`

### 定义函数

```
doubleMe x = x + x
```

函数的声明与它的调用形式大致相同，都是先函数名，后跟由空格分隔的参数表。但在声明中一定要在 = 后面定义函数的行为。

```
doubleUs x y = x*2 + y*2
可以写成
doubleUs x y = doubleMe x + doubleMe y
```

这种情形在 Haskell 下边十分常见：编写一些简单的函数，然后将其组合，形成一个较为复杂的函数，这样可以减少重复工作。

Haskell 中的函数并没有顺序，所以先声明 `doubleUs` 还是先声明 `doubleMe` 都是同样的。

首字母大写的函数是不允许的

### `if` 语句

Haskell 中 `if` 语句的 `else` 部分是不可省略。

```
doubleSmallNumber x = if x > 100
                      then x
                      else  x*2
```

在命令式语言中，你可以通过 `if` 语句来跳过一段代码，而在 Haskell 中，每个函数和表达式都要返回一个结果。

Haskell 中的 `if` 语句的另一个特点就是它其实是个表达式，表达式就是返回一个值的一段代码：`5` 是个表达式，它返回 `5`；`4+8` 是个表达式；`x+y` 也是个表达式，它返回 `x+y` 的结果。正由于 `else` 是强制的，`if` 语句一定会返回某个值，所以说 `if` 语句也是个表达式。

### List 入门

List 是一种单类型的数据结构，可以用来存储多个类型相同的元素。我们可以在里面装一组数字或者一组字符，但不能把字符和数字装在一起。

```
ghci> let lostNumbers = [4,8,15,16,23,48]  
```

字符串实际上就是一组字符的 List，`"Hello"` 只是 `['h','e','l','l','o']` 的语法糖而已。所以我们可以使用处理 List 的函数来对字符串进行操作。 

 将两个 List 合并是很常见的操作，这可以通过 ++ 运算符实现。

```
ghci> [1,2,3,4] ++ [9,10,11,12]  
[1,2,3,4,9,10,11,12]  
ghci> "hello" ++ " " ++ "world"  
"hello world"  
ghci> ['w','o'] ++ ['o','t']  
"woot"
```

在使用 ++ 运算符处理长字符串时要格外小心(对长 List 也是同样)，Haskell 会遍历整个的 List(++ 符号左边的那个)

用 `:` 运算符往一个 List 前端插入元素会是更好的选择。`:` 运算符可以连接一个元素到一个 List 或者字符串之中，而 `++` 运算符则是连接两个 List。

```
ghci> 'A':" SMALL CAT"  
"A SMALL CAT"  
ghci> 5:[1,2,3,4,5] 
[5,1,2,3,4,5]
```

`[1,2,3]` 实际上是 `1:2:3:[]` 的语法糖。

若是要按照索引取得 List 中的元素，可以使用 `!!` 运算符，索引的下标为 0。

```
ghci> "Steve Buscemi" !! 6  
'B'  
ghci> [9.4,33.2,96.2,11.2,23.25] !! 1  
33.2
```

试图在一个只含有 4 个元素的 List 中取它的第 6 个元素，就会报错。

当 List 内装有可比较的元素时，使用 `>` 和 `>=` 可以比较 List 的大小。

还可以对 List 做啥？如下是几个常用的函数：

head 返回一个 List 的头部，也就是 List 的首个元素。
tail 返回一个 List 的尾部，也就是 List 除去头部之后的部分。
last 返回一个 List 的最后一个元素。
init 返回一个 List 除去最后一个元素的部分。
length 返回一个 List 的长度。
null 检查一个 List 是否为空：length 为 0
reverse 将一个 List 反转:
take 返回一个 List 的前几个元素，看：
drop 与 take 的用法大体相同，它会删除一个 List 中的前几个元素。
sum 返回一个 List 中所有元素的和。product 返回一个 List 中所有元素的积。
elem 判断一个元素是否在包含于一个 List，通常以中缀函数的形式调用它。

```
ghci> head [5,4,3,2,1] 
5
ghci> take 3 [5,4,3,2,1]  
[5,4,3]  
ghci> 4 `elem` [3,4,5,6]  
True  
ghci> 10 `elem` [3,4,5,6]  
False
```

### 使用 Range

Range 是构造 List 方法之一，而其中的值必须是可枚举的，像 1、2、3、4...字符同样也可以枚举，字母表就是 A..Z 所有字符的枚举。

```
ghci> [1..20]
[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
ghci> ['a'..'z']
"abcdefghijklmnopqrstuvwxyz"
ghci> ['K'..'Z']  
"KLMNOPQRSTUVWXYZ"
```

Range 的特点是他还允许你指定每一步该跨多远

```
ghci> [2,4..20]
[2,4,6,8,10,12,14,16,18,20]
```

另一方面就是仅凭前几项，数组的后项是不能确定的。要得到 20 到 1 的 List，`[20..1]` 是不可以的。必须得 `[20,19..1]`。

出于定义的原因，浮点数并不精确。若是使用浮点数的话，你就会得到如下的糟糕结果

```
ghci> [0.1, 0.3 .. 1]
[0.1,0.3,0.5,0.7,0.8999999999999999,1.0999999999999999]
```

由于 Haskell 是惰性的，它不会对无限长度的 List 求值，否则会没完没了的。它会等着，看你会从它那儿取多少。

```
ghci> take 10 (cycle [1,2,3])
[1,2,3,1,2,3,1,2,3,1]
ghci> take 12 (cycle "LOL ")
"LOL LOL LOL "
ghci> take 10 (repeat 5)
[5,5,5,5,5,5,5,5,5,5]
```

### List Comprehension

竖线左端的部分是输出函数

```
ghci> [x*2 | x <- [1..10], x*2 >= 12]
[12,14,16,18,20]
ghci> [ x | x <- [50..100], x `mod` 7 == 3]
[52,59,66,73,80,87,94]
```

从一个 List 中筛选出符合特定限制条件的操作也可以称为过滤 (filtering)

假如我们想要一个 comprehension，它能够使 List 中所有大于 10 的奇数变为 "BANG"，小于 10 的奇数变为 "BOOM"，其他则统统扔掉。方便重用起见，我们将这个 comprehension 置于一个函数之中。

```
boomBangs xs = [ if x < 10 then "BOOM!" else "BANG!" | x <- xs, odd x]
```

除了多个限制条件之外，从多个 List 中取元素也是可以的。

```
ghci> [ x*y | x <-[2,5,10], y <- [8,10,11], x*y > 50]
[55,80,100,110]
```

让我们编写自己的 length 函数吧！就叫做 length'!

```
length' xs = sum [1 | _ <- xs]
```

### Tuple

一组数字的 List 就是一组数字，它们的类型相同，且不关心其中包含元素的数量。而 Tuple 则要求你对需要组合的数据的数目非常的明确，**它的类型取决于其中项的数目与其各自的类型**。Tuple 中的项由括号括起，并由逗号隔开。

另外的不同之处就是 Tuple 中的项不必为同一类型，在 Tuple 里可以存入多态别项的组合。

使用 Tuple 前应当事先明确一条数据中应该由多少个项。每个不同长度的 Tuple 都是独立的类型，所以你就不可以写个函数来给它追加元素。而唯一能做的，就是通过函数来给一个 List 追加序对，三元组或是四元组等内容。

fst 返回一个序对的首项。
snd 返回序对的尾项。
zip 它可以用来生成一组序对 (Pair) 的 List。

```
ghci> fst ("Wow", False)
"Wow"

ghci> zip [1,2,3,4,5] [5,5,5,5,5]
[(1,5),(2,5),(3,5),(4,5),(5,5)]

ghci> zip [5,3,2,6,2,7,2,5,4,6,6] ["im","a","turtle"]
[(5,"im"),(3,"a"),(2,"turtle")]

ghci> let rightTriangles' = [ (a,b,c) | c <- [1..10], b <- [1..c], a <- [1..b], a^2 + b^2 == c^2, a+b+c == 24]
ghci> rightTriangles'
[(6,8,10)]
```

## 类型和类型声明

Haskell 是 Static Type，这表示在编译时期每个表达式的类型都已经确定下来，这提高了代码的安全性。

与 Java 和 Pascal 不同，Haskell 支持类型推导。写下一个数字，你就没必要另告诉 Haskell 说"它是个数字"，它自己能推导出来。

使用 `:t` 命令后跟任何可用的表达式，即可得到该表达式的类型

```
ghci> :t 'a'  
'a' :: Char  
ghci> :t True  
True :: Bool  
ghci> :t "HELLO!"  
"HELLO!" :: [Char]  
ghci> :t (True, 'a')  
(True, 'a') :: (Bool, Char)  
ghci> :t 4 == 5  
4 == 5 :: Bool
```

函数也有类型。编写函数时，给它一个明确的类型声明是个好习惯

```
removeNonUppercase :: [Char] -> [Char]  
removeNonUppercase st = [ c | c <- st, c `elem` ['A'..'Z']]
```

要是多个参数的函数该怎样，参数之间由 -> 分隔，而与回传值之间并无特殊差异。回传值是最后一项，参数就是前三项。

```
addThree :: Int -> Int -> Int -> Int  
addThree x y z = x + y + z
```
如下是几个常见的类型：

* Int 表示整数。Int 是有界的，也就是说它由上限和下限。对 32 位的机器而言，上限一般是 2147483647，下限是 -2147483648。
* Integer 表示...厄...也是整数，但它是无界的。它的效率不如 Int 高。
* Float 表示单精度的浮点数。
* Double 表示双精度的浮点数。
* Bool 表示布尔值，它只有两种值：True 和 False。
* Char 表示一个字符。一个字符由单引号括起，一组字符的 List 即为字符串。
* Tuple 的类型取决于它的长度及其中项的类型。注意，空 Tuple 同样也是个类型，它只有一种值：`()`。

### Typeclasses入门

`==` 函数的类型声明是怎样的？

```
ghci> :t (==)  
(==) :: (Eq a) => a -> a -> Bool
```

`=>` 符号。它左边的部分叫做类型约束。我们可以这样阅读这段类型声明："相等函数取两个相同类型的值作为参数并回传一个布尔值，而这两个参数的类型同在 Eq 类之中(即类型约束)"

`Eq` 这一 Typeclass 提供了判断相等性的接口，凡是可比较相等性的类型必属于 `Eq` class。

`elem` 函数的类型为: `(Eq a)=>a->[a]->Bool`。这是它在检测值是否存在于一个 List 时使用到了`==` 的缘故。

* `Eq` 包含可判断相等性的类型。提供实现的函数是 `==` 和 `/=`。
* `Ord` 包含可比较大小的类型。除了函数以外，我们目前所谈到的所有类型都属于 Ord 类。Ord 包中包含了`<, >, <=, >=` 之类用于比较大小的函数。
* `Show` 的成员为可用字符串表示的类型。最常用的函数表示 `show`。它可以取任一Show的成员类型并将其转为字符串。
* `Read` 是与 `Show` 相反的 Typeclass。`read` 函数可以将一个字符串转为 `Read` 的某成员类型。

看一下 `read` 函数的类型声明吧：

```
ghci> :t read  
read :: (Read a) => String -> a
```

它的回传值属于 ReadTypeclass，但我们若用不到这个值，它就永远都不会得知该表达式的类型。所以我们需要在一个表达式后跟:: 的类型注释


```
ghci> show 3  
"3"  
ghci> read "8.2" + 3.8  
12.0  
ghci> read "5" :: Int  
5  
```

* `Enum` 的成员都是连续的类型 -- 也就是可枚举。Enum 类存在的主要好处就在于我们可以在 `Range` 中用到它的成员类型：每个值都有后继子 (successer) 和前置子 (predecesor)，分别可以通过 `succ` 函数和 `pred` 函数得到。该 Typeclass 包含的类型有：`(), Bool, Char, Ordering, Int, Integer, Float 和 Double`。
* `Bounded` 的成员都有一个上限和下限。
* `Num` 是表示数字的 Typeclass，它的成员类型都具有数字的特征。
* `Integral` 同样是表示数字的 Typeclass。`Num` 包含所有的数字：实数和整数。而 `Integral` 仅包含整数，其中的成员类型有 `Int` 和 `Integer`。
* `Floating` 仅包含浮点类型：`Float` 和 `Double`。

## 函数的语法

### 模式匹配 (Pattern matching)

模式匹配通过检查数据的特定结构来检查其是否匹配，并按模式从中取得数据。如果没有匹配项，则调用最后一个非匹配的表达式。

检查我们传给它的数字是不是 7：

```
lucky :: (Integral a) => a -> String  
lucky 7 = "LUCKY NUMBER SEVEN!"  
lucky x = "Sorry, you're out of luck, pal!"
```

阶乘函数：

```
factorial :: (Integral a) => a -> a  
factorial 0 = 1  
factorial n = n * factorial (n - 1)
```

在 List Comprehension 中也能用模式匹配：

```
ghci> let xs = [(1,3), (4,3), (2,4), (5,3), (5,6), (3,1)]  
ghci> [a+b | (a,b) <- xs]  
[4,7,6,8,11,4]
```

一旦模式匹配失败，它就简单挪到下个元素。

对 List 本身也可以使用模式匹配。你可以用 `[]` 或 `:` 来匹配它。

实现个我们自己的 `head` 函数。

```
head' :: [a] -> a  
head' [] = error "Can't call head on an empty list, dummy!" 
head' (x:_) = x
```

获得前几项元素：

```
tell :: (Show a) => [a] -> String  
tell [] = "The list is empty"  
tell (x:[]) = "The list has one element: " ++ show x  
tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y  
tell (x:y:_) = "This list is long. The first two elements are: " ++ show x ++ " and " ++ show y
```

`as` 模式，就是将一个名字和 `@` 置于模式前，可以在按模式分割什么东西时仍保留对其整体的引用。

```
capital :: String -> String  
capital "" = "Empty string, whoops!"  
capital all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]

ghci> capital "Dracula"  
"The first letter of Dracula is D"
```

### 什么是 Guards

模式用来检查一个值是否合适并从中取值，而 guard 则用来检查一个值的某项属性是否为真。

guard 由跟在函数名及参数后面的竖线标志，通常他们都是靠右一个缩进排成一列。一个 guard 就是一个布尔表达式，如果为真，就使用其对应的函数体。如果为假，就送去见下一个 guard，如之继续。

最后的那个 guard 往往都是 `otherwise`，它的定义就是简单一个 `otherwise = True` ，捕获一切。

```
bmiTell :: (RealFloat a) => a -> a -> String  
bmiTell weight height  
    | weight / height ^ 2 <= 18.5 = "You're underweight"  
    | weight / height ^ 2 <= 25.0 = "You're normal"  
    | weight / height ^ 2 <= 30.0 = "You're fat"  
    | otherwise                 = "You're a whale"
```

实现我们自己的 `compare` 函数：

```
myCompare :: (Ord a) => a -> a -> Ordering  
a `myCompare` b  
    | a > b     = GT  
    | a == b    = EQ  
    | otherwise = LT
```

### 关键字 Where

给它一个名字来代替这三个表达式会更好些，函数在 where 绑定中定义的名字只对本函数可见

```
bmiTell :: (RealFloat a) => a -> a -> String  
bmiTell weight height  
    | bmi <= skinny = "You're underweight"  
    | bmi <= normal = "You're normal"  
    | bmi <= fat = "You're fat"  
    | otherwise                 = "You're a whale"
    where bmi = weight / height ^ 2
    skinny = 18.5  
    normal = 25.0  
    fat = 30.0
```

where 绑定也可以使用模式匹配

```
...  
where bmi = weight / height ^ 2  
      (skinny, normal, fat) = (18.5, 25.0, 30.0)
```

搞个计算一组 bmi 的函数：

```
calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
calcBmis xs = [bmi w h | (w, h) <- xs] 
    where bmi weight height = weight / height ^ 2
```

使用 let

```
calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2]
```

### 关键字 Let

`let` 绑定与 where 绑定很相似。`where` 绑定是在函数底部定义名字，对包括所有 guard 在内的整个函数可见。`let` 绑定则是个表达式，允许你在任何位置定义局部变量，而对不同的 guard 不可见。

**`let` 绑定本身是个表达式，而 `where` 绑定则是个语法结构。因而 `let` 可以随处安放**

`let` 的格式为 `let [bindings] in [expressions]`。在 `let` 中绑定的名字仅对 `in` 部分可见。

```
cylinder :: (RealFloat a) => a -> a -> a  
cylinder r h = 
    let sideArea = 2 * pi * r * h  
        topArea = pi * r ^2  
    in  sideArea + 2 * topArea
```

let 已经这么好了，还要 where 干嘛呢？嗯，let 是个表达式，定义域限制的相当小，因此不能在多个 guard 中使用。一些朋友更喜欢 where，因为它是跟在函数体后面，把主函数体距离类型声明近一些会更易读。

### case 语法

`case` 语句。就是取一个变量，按照对变量的判断选择对应的代码块。其中可能会存在一个万能匹配以处理未预料的情况。
 
模式匹配本质上不过就是 `case` 语句的语法糖而已。这两段代码就是完全等价的：

```
head' :: [a] -> a  
head' [] = error "No head for empty lists!"  
head' (x:_) = x
```

```
head' :: [a] -> a  
head' xs = case xs of [] -> error "No head for empty lists!"
                      (x:_) -> x
```

看得出，case表达式的语法十分简单：

```
case expression of pattern -> result  
                   pattern -> result  
                   pattern -> result  
                   ...
```

函数参数的模式匹配只能在定义函数时使用，而 case 表达式可以用在任何地方。

```
describeList :: [a] -> String  
describeList xs = "The list is " ++ case xs of [] -> "empty."  
                                               [x] -> "a singleton list."   
                                               xs -> "a longer list."
```

## 递归

递归在 Haskell 中非常重要。命令式语言要求你提供求解的步骤，Haskell 则倾向于让你提供问题的描述。这便是 Haskell 没有 `while` 或 `for` 循环的原因，递归是我们的替代方案。

### 实作 Maximum

maximum 函数取一组可排序的 List（属于 Ord Typeclass） 做参数，并回传其中的最大值。

现在看看递归的思路是如何：

```
maximum' :: (Ord a) => [a] -> a  
maximum' [] = error "maximum of empty list"  
maximum' [x] = x  
maximum' (x:xs) = max x (maximum' xs)
```

## 高端函数

Haskell 中的函数可以接受函数作为参数也可以返回函数作为结果，这样的函数就被称作高端函数。

本质上，Haskell 的所有函数都只有一个参数，所有多个参数的函数都是 Curried functions。

看看 `max` 函数的类型: `max :: (Ord a) => a -> a -> a`。 也可以写作: `max :: (Ord a) => a -> (a -> a)`。 可以读作 `max` 取一个参数 `a`，并回传一个函数(就是那个 `->`)，这个函数取一个 `a` 类型的参数，回传一个 `a`。 这便是为何只用箭头来分隔参数和回传值类型。

```
applyTwice :: (a -> a) -> a -> a  
applyTwice f x = f (f x)
```

使用：

```
ghci> applyTwice (+3) 10  
16  
ghci> applyTwice (++ " HAHA") "HEY"  
"HEY HAHA HAHA"  
ghci> applyTwice ("HAHA " ++) "HEY"  
"HAHA HAHA HEY"  
ghci> applyTwice (multThree 2 2) 9  
144  
ghci> applyTwice (3:) [1]  
[3,3,1]
```

`zipWith`，它取一个函数和两个 List 做参数，并把两个 List 交到一起(使相应的元素去调用该函数)。 如下就是我们的实现:

```
zipWith' :: (a -> b -> c) -> [a] -> [b] -> [c]  
zipWith' _ [] _ = []  
zipWith' _ _ [] = []  
zipWith' f (x:xs) (y:ys) = f x y : zipWith' f xs ys

ghci> zipWith' (+) [4,2,5,6] [2,6,2,3]  
[6,8,7,9]  
ghci> zipWith' max [6,3,2,1] [7,3,1,5]  
[7,3,2,5]  
ghci> zipWith' (++) ["foo "，"bar "，"baz "] ["fighters"，"hoppers"，"aldrin"]  
["foo fighters","bar hoppers","baz aldrin"]  
ghci> zipWith' (*) (replicate 5 2) [1..]  
[2,4,6,8,10]  
ghci> zipWith' (zipWith' (*)) [[1,2,3],[3,5,6],[2,3,4]] [[3,2,2],[3,4,5],[5,4,3]]  
[[3,4,6],[9,20,30],[10,12,12]]
```

### map 与 filter

```
map :: (a -> b) -> [a] -> [b]  
map _ [] = []  
map f (x:xs) = f x : map f xs

ghci> map (++ "!") ["BIFF"，"BANG"，"POW"]  
["BIFF!","BANG!","POW!"]  
```

filter 函数取一个限制条件和一个 List，回传该 List 中所有符合该条件的元素。

```
filter :: (a -> Bool) -> [a] -> [a]  
filter _ [] = []  
filter p (x:xs)   
    | p x       = x : filter p xs  
    | otherwise = filter p xs
    
ghci> filter (>3) [1,5,3,2,1,6,4,3,2,1]  
[5,6,4]  
```

出所有小于 10000 且为奇的平方的和

```
ghci> sum (takeWhile (<10000) (filter odd (map (^2) [1..])))
166650
```

得先提下 `takeWhile` 函数，它取一个限制条件和 List 作参数，然后从头开始遍历这一 List，并回传符合限制条件的元素。 而一旦遇到不符合条件的元素，它就停止了。

### lambda

lambda 就是匿名函数。有些时候我们需要传给高端函数一个函数，而这函数我们只会用这一次，这就弄个特定功能的 lambda。

编写 lambda，就写个 \，后面是用空格分隔的参数，-> 后面就是函数体。

```
numLongChains :: Int  
numLongChains = length (filter (\xs -> length xs > 15) (map chain [1..100]))
```

### 关键字 fold

我们会发现处理 List 的许多函数都有固定的模式，通常我们会将边界条件设置为空 List，再引入 (x:xs) 模式，对单个元素和余下的 List 做些事情。这一模式是如此常见，因此 Haskell 引入了一组函数来使之简化，也就是 fold。它们与map有点像，只是它们回传的是单个值。

一个 fold 取一个二元函数，一个初始值(我喜欢管它叫累加值)和一个需要折叠的 List。

```
sum' :: (Num a) => [a] -> a  
sum' xs = foldl (\acc x -> acc + x) 0 xs

ghci> sum' [3,5,2,1]  
11
```

### 有$的函数调用

```
($) :: (a -> b) -> a -> b  
f $ x = f x
```

普通的函数调用符有最高的优先级，而 `$` 的优先级则最低。用空格的函数调用符是左结合的，如 `f a b c` 与 `((f a) b) c` 等价，而 `$` 则是右结合的。

`sum (map sqrt [1..130])`，由于低优先级的 $，我们可以将其改为 `sum $ map sqrt [1..130]`，可以减少我们代码中括号的数目，可以把$看作是在右面写一对括号的等价形式。

### 函数组合

```
(.) :: (b -> c) -> (a -> b) -> a -> c  
f . g = \x -> f (g x)
```
 
 f 的参数类型必须与 g 的回传类型相同
 
 函数组合的用处之一就是生成新函数，并传递给其它函数。
 
 
 ```
 ghci> map (\x -> negate (abs x)) [5,-3,-6,7,-3,2,-19,24]  
[-5,-3,-6,-7,-3,-2,-19,-24]
 ```
 
 用函数组合，我们可以将代码改为：
 
 ```
 ghci> map (negate . abs) [5,-3,-6,7,-3,2,-19,24]  
[-5,-3,-6,-7,-3,-2,-19,-24]
 ```
 
## 模块 (Modules)

### 装载模块

Haskell 中的模块是含有一组相关的函数，类型和类型类的组合。而 Haskell 进程的本质便是从主模块中引用其它模块并调用其中的函数来运行操作。

在 Haskell中，装载模块的语法为 import，这必须得在函数的定义之前，所以一般都是将它置于代码的顶部。

避免命名冲突有个方法，便是 qualified import

```
import qualified Data.Map
```

这样一来，再调用 Data.Map 中的 filter 函数，就必须得 `Data.Map.filter`

检索函数或搜索函数字置就用 [Hoogle](http://www.Haskell.org/hoogle/)，

### Data.List

显而易见，Data.List 是关于 List 操作的模块，它提供了一组非常有用的 List 处理函数。

### Data.Char

Data.Char 模块包含了一组用于处理字符的函数。由于字符串的本质就是一组字符的 List，所以往往会在 filter 或是 map 字符串时用到它.

### Data.Map

关联列表(也叫做字典)是按照键值对排列而没有特定顺序的一种 List。

如下便是个表示电话号码的关联列表:

```
phoneBook = [("betty","555-2938") ,
             ("bonnie","452-2928") ,
             ("patsy","493-2928") ,
             ("lucille","205-2928") ,
             ("wendy","939-8282") ,
             ("penny","853-2492") ]
```

对关联列表最常见的操作就是按键索值

```
findKey :: (Eq k) => k -> [(k,v)] -> Maybe v 
findKey key [] = Nothing
findKey key ((k,v):xs) = 
     if key == k then 
         v 
     else 
         findKey key xs
```

### Data.Set

fromList 函数同你想的一样，它取一个 List 作参数并将其转为一个集合

### 创建自己的模块

```
module Geometry  
( sphereVolume  
，sphereArea  
，cubeVolume  
，cubeArea  
，cuboidArea  
，cuboidVolume  
) where  

sphereVolume :: Float -> Float  
sphereVolume radius = (4.0 / 3.0) * pi * (radius ^ 3)  

...
```

## 构造我们自己的 Types 和 Typeclasses

不过在 Haskell 中该如何构造自己的类型呢？好问题，一种方法是使用 data 关键字。首先我们来看看 Bool 在标准函数库中的定义：

```
data Bool = False | True
```

data 表示我们要定义一个新的类型。= 的左端标明类型的名称即 Bool，= 的右端就是值构造子 (Value Constructor)，它们明确了该类型可能的值。

**值构造子的本质是个函数，可以返回一个类型的值**

```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

函数计算图形面积

```
surface :: Shape -> Float
surface (Circle _ _ r) = pi * r ^ 2
surface (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

当我们往控制台输出值的时候，Haskell 会先调用 show 函数得到这个值的字符串表示才会输出。因此要让我们的 Shape 类型成为 Show 类型类的成员。

```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float deriving (Show)
```

我们若要取一组不同半径的同心圆

```
ghci> map (Circle 10 20) [4,5,6,6]
[Circle 10.0 20.0 4.0,Circle 10.0 20.0 5.0,Circle 10.0 20.0 6.0,Circle 10.0 20.0 6.0]
```

### Record Syntax

```
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     , height :: Float
                     , phoneNumber :: String
                     , flavor :: String
                     } deriving (Show)
```

### 类型构造子

类型构造子可以取类型作参数，产生新的类型。

```
data Maybe a = Nothing | Just a
```

在它的值不是 Nothing 时，它的类型构造子可以搞出 Maybe Int，Maybe String 等等诸多态别。但只一个 Maybe 是不行的，因为它不是类型，而是类型构造子。要成为真正的类型，必须得把它需要的类型参数全部填满。

### 生成类型类 instance

我们先了解下 Haskell 是如何自动生成这几个类型类的 instance，Eq, Ord, Enum, Bounded, Show, Read。

## 输入与输出

在 Haskell 中，一个函数不能改变状态，像是改变一个变量的内容。在 Haskell 中函数唯一可以做的事是根据我们给定的参数来算出结果。如果我们用同样的参数调用两次同一个函数，它会回传相同的结果。

```
import Data.Char

main = do
    putStrLn "What's your first name?"
    firstName <- getLine
    putStrLn "What's your last name?"
    lastName <- getLine
    let bigFirstName = map toUpper firstName
        bigLastName = map toUpper lastName
    putStrLn $ "hey " ++ bigFirstName ++ " " ++ bigLastName ++ ", how are you?"
```

它看起来就像一个命令式的程序。我们写了一个 do 并且接着一连串指令，就像写个命令式程序一般

getLine 的型态是 IO String。你不能串接一个字符串跟 I/O action。我们必须先把 String 的值从 I/O action 中取出，而唯一可行的方法就是在 I/O action 中使用 `name <- getLine`

在 Haskell 中，他的意义则是利用某个 pure value 造出 I/O action。用之前盒子的比喻来说，就是将一个 value 装进箱子里面。产生出的 I/O action 并没有作任何事，只不过将 value 包起来而已。所以在 I/O 的情况下来说，return "haha" 的型态是 IO String

getChar 是一个读取单一字符的 I/O action。getLine 是一个读取一行的 I/O action。getContents 是一个从标准输入读取直到 end-of-file 字符的 I/O action。

### 乱数

在 System.Random 模块中。他包含所有满足我们需求的函数。

### Exceptions

Haskell 有一个很棒的型态系统。Algebraic data types 允许像是 Maybe 或 Either 这种型态，我们能用这些型态来代表一些可能有或没有的结果。

## 函数式地思考来解决问题

[什么是函数式编程思维？ - 用心阁的回答 - 知乎](https://www.zhihu.com/question/28292740/answer/40336090)







