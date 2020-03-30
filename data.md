## 复数的表示
我们知道复数有直角坐标和极坐标两种表示方法，现在假定已有直角和极坐标表示下的构造
make-from-real-imag、make-from-mag-angle和对应的选择函数real-part、imag-part、magnitude、angle。
那么复数的运算可以按下面描述:
```lisp
(define (add z1 z2)
   (make-from-real-imag (+ (real-part z1) (real-part z2))
                        (+ (imag-part z1) (imag-part z2))))
(define (sub z1 z2)
   (make-from-real-imag (- (real-part z1) (real-part z2))
                        (- (imag-part z1) (imag-part z2))))
(define (mul z1 z2)
   (make-from-mag-angle (* (magnitude z1) (magnitude z2))
                        (+ (angle z1) (angle z2))))
(define (div z1 z2)
   (make-from-mag-angle (/ (magnitude z1) (magnitude z2))
                        (- (angle z1) (angle z2))))
```
这样就将复数的使用和实现分离开了，使用者只需要关心如何使用，而不用操心具体实现。

下面给出了复数的两种实现，一种基于直角坐标，一种基于极坐标。
rectangle
```lisp
(define (make-from-real-imag x y) (cons x y))
(define (real-part z) (car z))
(define (imag-part z) (cdr z))
(define (make-from-mag-angle r a)
  (make-from-real-imag (* r (cos a)) (* r (sin a))))
(define (magnitude z)
  (define (square x) (* x x))
  (sqrt (+ (square (real-part z)) (square (imag-part z)))))
(define (angle z)
  (atan (real-part z) (imag-part z)))
```

polar
```lisp
(define (make-from-mag-angle r a) (cons r a))
(define (magnitude z) (car z))
(define (angle z) (cdr z))
(define (make-from-real-imag x y)
  (define (square x) (* x x))
  (cons (sqrt (+ (square x) (square y))) (atan y x)))
(define (real-part z) (* (magnitude z) (cos (anlge z))))
(define (imag-part z) (* (magnitude z) (sin (anlge z))))
```

那么该如何让两种表示方式在同一个系统里共存呢？

- 基于类型的分配
简单地来时，基于类型的分配就是是数据带上类型，根据数据的类型选择不同的操作
```lisp
(define (attach tag content) (cons tag content))
(define (tag x)(car x))
(define (content x) (cdr x))

(define (rectangle? complex) (eq? (tag complex) 'rectangle))
(define (polar? complex) (eq? (tag complex) 'polar))

(define (real-part z)
  (cond  (rectangle? z) (real-part-rectangle z)
         (polar? z) (real-part-polar z))
         (error "wrong type"))
```
这是最自然的实现，但上面的实现有两个问题，一是使用者需要知道所有的类型，二是实现者需要更改自己的过程名以避免冲突。

- 数据导向
数据导向实际上维护这样的一个操作表， 按照数据类型和过程名去表格里查找对应的过程，这样不同的实现方式相互独立，且添加
新的实现方式后对现有系统毫无影响。

-------------------------------------------------------
    op      |    Polar          |   Rectangle
-------------------------------------------------------
real-part   | real-part-polar   |  real-part-rectangle
imag-part   | imag-part-polar   |  imag-part-rectangle
magnitude   | magnitude-polar   |  magnitude-rectangle
angle       | angle-polar       |  angle-rectangle
-------------------------------------------------------


```lisp
;;; Table definition and operation
(define (assoc key records)
  (cond ((null? records) false)
	((equal? key (caar records)) (car records))
	(else
	    (assoc key (cdr records)))))

(define (make-table)
  (let ((local-table (list '*table*)))
    (define (lookup key-1 key-2)
      (let ((subtable (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record (assoc key-2 (cdr subtable))))
              (if record
                  (cdr record)
                  false))
            false)))
    (define (insert! key-1 key-2 value)
      (let ((subtable (assoc key-1 (cdr local-table))))
        (if subtable
            (let ((record (assoc key-2 (cdr subtable))))
              (if record
                  (set-cdr! record value)
                  (set-cdr! subtable
                            (cons (cons key-2 value)
                                  (cdr subtable)))))
            (set-cdr! local-table
                      (cons (list key-1
                                  (cons key-2 value))
                            (cdr local-table)))))
      'ok)    
    (define (dispatch m)
      (cond ((eq? m 'lookup-proc) lookup)
            ((eq? m 'insert-proc!) insert!)
            (else (error "Unknown operation -- TABLE" m))))
    dispatch))

(define operation-table (make-table))
(define get (operation-table 'lookup-proc))
(define put (operation-table 'insert-proc!))

;;; Polar implement
(define (install-complex-polar)
  (define (make-from-mag-angle r a) (cons r a))
  (define (magnitude z) (car z))
  (define (angle z) (cdr z))
  (define (make-from-real-imag x y)
    (define (square x) (* x x))
    (cons (sqrt (+ (square x) (square y))) (atan y x)))
  (define (real-part z) (* (magnitude z) (cos (angle z))))
  (define (imag-part z) (* (magnitude z) (sin (angle z))))

  (put 'polar 'make-from-mag-angle
       (lambda (x y) (cons 'polar (make-from-mag-angle x y))))
  (put 'polar 'magnitude magnitude)
  (put 'polar 'angle angle)
  (put 'polar 'make-from-real-imag
       (lambda (x y) (cons 'polar (make-from-real-imag x y))))
  (put 'polar 'real-part real-part)
  (put 'polar 'imag-part imag-part)
  'done)

;;; Rectangle implement
(define (install-complex-rectangle)
  (define (make-from-real-imag x y) (cons x y))
  (define (real-part z) (car z))
  (define (imag-part z) (cdr z))
  (define (make-from-mag-angle r a)
    (make-from-real-imag (* r (cos a)) (* r (sin a))))
  (define (magnitude z)
    (define (square x) (* x x))
    (sqrt (+ (square (real-part z)) (square (imag-part z)))))
  (define (angle z)
    (atan (real-part z) (imag-part z)))

  (put 'rectangle 'make-from-mag-angle
       (lambda (x y) (cons 'rectangle (make-from-mag-angle x y))))
  (put 'rectangle 'magnitude magnitude)
  (put 'rectangle 'angle angle)
  (put 'rectangle 'make-from-real-imag
       (lambda (x y) (cons 'rectangle (make-from-real-imag x y))))
  (put 'rectangle 'real-part real-part)
  (put 'rectangle 'imag-part imag-part)
  'done)

;;;Extract tag and content 
(define (tag z) (car z))
(define (content z) (cdr z))

;;; Upper interface
(define (make-from-real-imag x y)
  ((get 'rectangle 'make-from-real-imag) x y))
(define (make-from-mag-angle r a)
  ((get 'polar 'make-from-mag-angle) r a))
(define (real-part z)
   ((get (tag z) 'real-part) (content z)))
(define (imag-part z)
   ((get (tag z) 'imag-part) (content z)))
(define (magnitude z)
   ((get (tag z) 'magnitude) (content z)))
(define (angle z)
   ((get (tag z) 'angle) (content z)))

;;;Install packages
(install-complex-polar)
(install-complex-rectangle)

;;; Examples
(define z1 (make-from-real-imag 3 4))
(define z2 (make-from-mag-angle 5 (atan 4 3)))
(magnitude z1)
(real-part z2)
```

- 消息传递
事实上，还有另外组织方式，更具模块化。
```lisp
(define (make-from-real-imag x y)
  (define (square x) (* x x))
  (define (dispatch op)
    (cond ((eq? op 'real-part) x)
          ((eq? op 'imag-part) y)
          ((eq? op 'magnitude)
           (sqrt (+ (square x) (square y))))
          ((eq? op 'angle) (atan y x))
          (else
           (error "wrong operation" op))))
  dispatch)
```
在这种实现下，复数的使用如下
```lisp
(define z (make-from-real-imag 3 4))
(z 'magnitude)
```
或者我们可以定义一层语法糖，这样就完全掩盖了z是用过程实现的这一点。
```lisp
(define (magnitude z) (z 'magnitude))
```
从这里可以看到，数据本身也可以是过程，使用者完全不会察觉到其中的差异，并且在实现过程中还可以看到现代程序语言中的包管理和面向对象的影子，

