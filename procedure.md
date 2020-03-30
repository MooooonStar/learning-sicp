## 寻找函数的不动点
 数x称为f(x)的不动点，如果x满足方程f(x)=x。给定一个初始值x0，不断的调用f(x)，若值能不断收敛则找到了函数的不动点。
 ```lisp
(define (fixed-point f)
  (define (good-enough? x)
    (< (abs (- (f x) x)) 0.00000001))
  (define (improve x)
    (/ (+ (f x)  x) 2))
  (define (iter x)
    (if (good-enough? x)
        x
        (iter (improve x))))
  (iter 1.0))

(define (sqrt x)
  (fixed-point (lambda (y) (/ x y))))

(sqrt 2.0)
```
上面首先定义了求不动点的过程fixed-point，内部包含三个子过程good-enough?，improve，iter，其中
good-enough?判断值是否已经收敛，条件为|f(x) - x| < 0.00000001, improve按照y=(x + f(x))/2不断
改进x(这里不直接使用f(x)纯粹是出于数学上的考虑，这是一种平均阻尼的技术，有助于函数收敛)，
iter判断x是否满足条件，是返回，不是继续迭代直到收敛。fixed-point接收一个过程f，返回从1开始不断迭代后的结果。

因为x ^ 2 = y 可以写成x = f(x) =  y / x, 所以把sqrt定义为求 y/x的不动点。 lambda用于创建过程，语法为
(lambda (<formal-parameter>) <body>)。事实上，过程定义的形式(define (<name> <formal-parameter>) <body>)不过是(define name (lambda (<formal-parameter>) <body>))的语法糖，从而前面define的两种特殊形式可以规约为一种，即只有(define name value)。

最后我们调用了sqrt，给定参数2, 得到了结果1.4142135623746899。

事实上，可以将平均阻尼的思想表示为如下过程
(define (damp f)
    (lambda (x) (/ (+ x (f x)) 2.0)))
此过程接收一个过程f，返回一个求x和f(x)平均值的过程。

于是我们的求平方根过程可以改写为
```lisp
(define (fixed-point f)
  (define (good-enough? x)
    (< (abs (- (f x) x)) 0.00000001))
  (define (iter x)
    (if (good-enough? x)
        x
        (iter (f x))))
  (iter 1.0))

(define (damp f)
    (lambda (x) (/ (+ x (f x)) 2.0)))

(define (sqrt x)
  (fixed-point (damp (lambda (y) (/ x y)))))

(sqrt 2.0)
```

这里可以看到，引入lambda表达式之后，过程和数据的定义形式统一了起来，并且过程即可以作为参数传递，也可以作为值返回，
过程的表现形式和使用已经和数据几乎没有差别了。