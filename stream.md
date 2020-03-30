流是一个很有意思的概念。

在介绍流之前，需要了解表达式的两种不同求值规则。

- 应用序求值(application order)
- 正则序求值(normal order)

简单点来说，应用序求值就是先求解各子表达式，再求解表达式本身。
而正则序求值就是按需求值，需要计算子表达式的时候再去求值。

例如，对于这样一个表示式
```lisp
(+ (* 3 2) (- 5 4))
```

应用序的求值过程是
```lisp
(+ (* 3 2) (- 5 4))
(+ 6 1)
7
```
而
```lisp
(+ (* 3 2) (- 5 4))
(+ 6 (- 5 4))
(+ 6 1)
7
```
为什么需要正则序求值呢？可以考虑这样一个除法问题

```
(define (div m n)
  (if (= n 0)
      m
      (/ m n)))

(div 3 0)
```
在应用序求值下，m 和 (/ m n)会先计算，这在n为0的时候将无法得出结果。在正则序下，
m会得到计算返回结果，而(/ m n)根本不会得到计算。

应用序会我们提供了一种表达"无限"的能力。例如下面的fibs就生成了整个斐波那契数列，而不是像之前仅仅计算了其中一项。

```lisp
(define (delay proc) (lambda () proc))
(define (force proc) (proc))

(define (cons-stream x y) (cons x (delay y)))
(define (car-stream s) (car s))
(define (cdr-stream s) (force (cdr s)))

(define (fibgen a b)
    (cons-stream a (fibgen b (+ a b))))

(define fibs
  (fibgen 0 1))

(define (stream-ref s n)
  (if (= n 0)
      (car s)
      (stream-ref (cdr-stream s) (- n 1))))

(stream-ref fibs 5)
```
从cons-stream、car-stream和cdr-stream上看，流和普通的序对的差别在于，流的尾项(cdr)是被延迟求值了的(delay)，
只有当真正访问时，值才会被强制计算(force)。也就是说，流是由数据和promise构成的，只有当被访问的时候，promise才会
被求值，从而用有限的内存营造了“无限”的假象。

流的另一个强大之处在于，将数据的生成和数据的处理解耦了。我们可以将流作为数据的发生器，而后经过一个个模块的处理，最后
得到想要的数据，就如同信号处理一般，中间的处理模块可以任意地增加和修改，而不用担心对整个系统的影响。

