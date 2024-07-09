---
title: Monad
categories: Knowledge
date: 2022-04-14
---

*A monad is just a monoid in the category of endofunctors, what's the problem?*

<!--more-->

[通过Scala理解什么是Monad](https://my.oschina.net/guanxun/blog/475527)
[Applicative Monad](https://cloud.tencent.com/developer/article/1371274)

A monda M is a parametric type M[T] with two operations, flatMap and unit.
```scala
extension [T, U](m: M[T])
    def flatMap(F: tT => M[U]): M[U]

def unit[T](x: T): M[T]

extension [A, B, C](f: A => B)
    @infix def andThen(g: B => C): A => C = 
        x => g(f(x))

m.map(f) == m.flatMap(x => unit(f(x)))
```
### Group

### Monoid

### Monad's laws
**Associativity**
```scala
monad.flatMap(f).flatMap(g) == monad.flatMap(v => f(v).flatMap(g))
```

**Left unit**
```scala
unit(x).flatMap(f) == f(x)
```

**Left unit**
```scala
monad.flatMap(unit) == monad
```

### Functor

### Endofunctor



