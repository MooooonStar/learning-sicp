## 符号求导

symbol就是形如 'x 的表达式，若(define x 3)， x只是3的别名。而'x则表示了x这个符号本身。
symbol其实是(quote x)的语法糖，有了symbol之后，我们就可以做一些有意思的事情了，比如符号求导。

```lisp
(define (deri exp var)
  (cond ((contant? exp) 0)
        ((same-variable? exp var) 1)
        ((sum-exp? exp)
          (make-sum (deri (addend exp) var) (deri (augend exp) var)))
        ((product-exp? exp)
         (let ((u (addend exp)) (v (augend exp)))
              (make-sum (make-product u (deri v var)) (make-product v (deri u var)))))))

(define (addend exp) (cadr exp))
(define (augend exp) (caddr exp))
(define (contant? exp) (number? exp))
(define (same-variable? exp var) (and (symbol? exp) (eq? exp var)))

(define (sum-exp? exp) (eq? (car exp) '+))
(define (product-exp? exp) (eq? (car exp) '*))
(define (make-sum u v) (list '+ u v))
(define (make-product u v) (list '* u v))
```

这里为简单起见，仅添加了4条规则，即常数的导数为0，变量本身的导数为1，和式求导和乘积求导按微积分给定的规则。

表达式就按照lisp语法构造，只是前面多了符号'，例如'(+ 2 x)表示2+x, 于是加数和被加数分别为(car (cdr exp))
和(car (cdr (cdr exp))),可以分别被简写为(cadr exp)和(caddr exp), 即将多个car和cdr操作压缩成一个。

下面我们就可以进行求导了。
(deri '(* x x) 'x) 结果为
(mcons '+ (mcons (mcons '* (mcons 'x (mcons 1 '()))) (mcons (mcons '* (mcons 'x (mcons 1 '()))) '())))
即'(+ (* x 1) （* x 1)， 很显然表达式未进行化简。

可以加入如下几条规则对表达式进行简单地化简
```lisp
(define (make-sum u v)
  (cond ((and (number? u) (number? v)) (+ u v))
        ((and (number? u) (= u 0)) v)
        ((and (number? v) (= v 0)) u)
        ((same-variable? u v) (list '* 2 u))
        (else (list '+ u v))))
(define (make-product u v)
  (cond ((and (number? u) (number? v)) (* u v))
        ((and (number? u) (= u 0)) 0)
        ((and (number? u) (= u 1)) v)
        ((and (number? v) (= v 0)) 0)
        ((and (number? v) (= v 1)) u)
        (else (list '* u v))))
```
再次对(deri '(* x x) 'x) 结果为 (mcons '* (mcons 2 (mcons 'x '())))。显然上面的求导程序还很粗糙，但是已经可以让
我们感受到，引入symbol后lisp的强大了。
