SICP使用Lisp作为编程语言，推荐DrRacket作为解释器。下面用一个小例子来简单介绍下lisp。
```lisp
(define N 5)                             
(define (fact n) 
        (cond ((< n 2) 1)
            (else (* n (fact (- n 1))))))
(fact N)
```
第1行定义了一个常量N，值为10
第2-5行定义了一个过程fact,参数为n。过程定义的语法为(define (name arglist) body), 其中arglist可以为任意个参数
```
(define (sum x y) (+ x y))
```
cond是lisp中的一种条件判断，其语法为
```lisp
(cond (prediction1? expr1)
      (prediction2? expr2))
```
cond顺序判断各子句，如为真则返回对应结果。可以最后接else子句，
若前面都不为真，则返回else子句的结果。cond有一种简化形式if，
(if prediction? expr1 expr2)， 若条件为真，返回expr1的结果，否则返回expr2的结果。
我们熟悉的switch/case语句，和cond的功能类似。
第6行调用了过程fib，参数为N

可以看到，lisp一个显著的特点就是层层嵌套的括号，括号里是前序表达式，如(+ 3 4)表示3+4。这种语法具有很强的一致性和拓展性，
简单的表达式构成复杂表达式后，形式上没有任何变化，特别适合递归操作，比如(* 5 (+ 3 4))即为 5*(3+4)。再比如(add 3 4)和 (+ 3 4)，
从形式上看不出自定义过程和基本过程的任何区别。

define是lisp中一个相当强大的功能， 甚至允许用户改掉系统关键字的行为。比如cons将元素组合成一个序对，但我们可以将它改为求两个元素的和。
```
(define cons (lambda (x y) (+ x y)))
```
现在调用(cons 3 4)返回的就是两者之和7了，虽然不建议这么做，但语言确实允许。
式子中lambda表达式定义了一个过程，其语法形式为(lambda (arglist) body)，我们熟知的匿名函数
和它类似，而之前过程的定义只不过是(define name (lambda (arglist) body))的语法糖。甚至于define也是lambda表示式的一种语法糖，
比如(define N 5)，可以写为((lambda (N) 'done) 5)。熟悉node.js的可能此时会拍大腿惊呼，这不就是node.js中export的实现方式么！

另外一个不同在于，lisp中没有循环，比如for这种，这有点出乎人意料了。但lisp中可以用迭代来实现循环，比如上面的阶乘可以改写成如下形式
```
(define (fact n) 
  (define (iter n product)
    (if (= n 1)
        product
        (iter (- n 1) (* n product))))
  (iter n 1))
```
在fact内部定义了一个迭代过程iter，将最后的结果存储到product上。这是一种尾递归，实现了迭代，从语法层面上就去掉了循环。

一般而言，一门程序设计语言需要具备的三大要素。这在lisp中体现如下:
- 基本元素
 数字1.0，字符串"hello"，符号'A，运算符如+、-、*、/等
- 组合的方法
  cons, cons将元素组合成序对，car取第一个元素，cdr取剩下的元素
```lisp
  (define a (cons 3 4))
  (car a) --> 3
  (cdr a) --> 4
```
- 抽象的方法
  define等


简而言之，lisp是一个关键字很少，语法简单但表达能力极强的语言。很多现代语言中大略带点lisp的影子，要知道lisp可是几十年前的上古语言啊。这
一方面要佩服语言设计者的前瞻性，另一方面也放映出语言的相通性。