---
title: "Praticalist's Guide to Monads"
date: 2025-12-24
tags: 
  - posts
layout: post.njk
---

Every single Haskell blogger writes a monad explination. Usually it involves
multiple white paper release, a degree in expert math and no code written.

Personally, I think its a futile effort to understand what monads are in a 
mathamatical level. You see, you don't need math knowledge to know how to use 
a monad. In fact, using them is quite simple.

In the following guide, we will use Scala for explination instead of Haskell. Why? 
Because its frankly a pain to explain Haskell syntax. We will use something 
more in line with what we are all familiar with: object oriented programming.

## So, what is a monad?

Instead of bringing out the math book, lets look at monad as a trait (or 
abstract class) that implements some methods. Anything that implements 
the trait is a monad. In my opinion, the three most important parts of a monad 
is `pure`, `map` and `flatMap`. Usually there is a lot more helper methods
but we don't care about them.

```scala
// F[_] means a generic higher kind type, 
// which is any type that stores a generic type 
// for example List[Int]
trait Monad[F[_]]:
  def pure[A](a: A): F[A]
  // A => F[B] means a function that takes A and returns F[B]
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
  def map[A, B](fa: F[A])(f: A => B): F[B] =
    flatMap(fa)(a => pure(f(a)))
```

Don't worry if you don't understand what the generic types mean. Just 
know that a monad:
- Can be made with `pure(value)`, which returns a monad storing `value`
- Can be `flatMap(fn)`ed, where the inner value of the monad is unwrapped and feed 
into `fn` (the `fn` returns a new monad). The return of the `fn` shall be the return 
of the `flatMap(fn)` itself.
- Can be `map(fn)`ed to apply the function `fn` to the value that the monad 
stores, the output of `fn` is wrapped back in the same monad and returned.

By this point you should have realized it's just a container type that stores data.
Why make it complicated?

If you don't however, look at the following examples.

## Example: identity

Let's take a look at the most simple monad out there, the identity:

```scala
// just a simple container of a value
case class Id[A](value: A)

object Identity extends Monad[Id]:
  override def pure[A](a: A): Id[A] = Id(a)
  override def flatMap[A, B](fa: Id[A])(f: A => Id[B]): Id[B] =
    f(fa.value)
```

Let's try using it:
```scala
def double(n: Int) = Id(n * 2)

@main def core() =
  val a = Identity.pure(12) // Id(12)
  val b = Identity.map(a)(input => input + 1) // now Id(13)
  val c = Identity.flatMap(b)(double) // now Id(26)
```

Also note that the identity monad is absolutely useless. You won't be using
it to do anything than demonstrate the concept.

Side note: many existing monad libaries allows you use this syntax to 
chain `map`s and `flatMap`s:
```scala
Identity.pure(12)
  .map(input => input + 1)
  .flatMap(double)
```
It requires more code to achieve so we are just going to stick with the 
boilerplate-y but simple implementation.

## Example: option

Identity monads are useless, but option monads are not. Let's look at the  
implementation first.

```scala
// + means covariant, explaining it takes too much time so
// assume +T is just a simple generic type
enum Option[+T]:
  case Some(value: T)
  case None

object Opt extends Monad[Option]:
  override def pure[A](a: A): Option[A] = Option.Some(a)
  override def flatMap[A, B](fa: Option[A])(f: A => Option[B]): Option[B] =
    fa match
      case Option.Some(value) =>
        f(value)
      case Option.None =>
        Option.None
```

The gist is, in a chain of `map`s and `flatMap`s, whenever one link of the chain 
returns a `None`, the entire chain returns a `None`, just like an early return. 
You then handle the error.

You can use it just like an identity monad:
```scala
def double(n: Int) = Option.Some(n * 2)

@main def core() =
  val a = Opt.pure(12)
  val b = Opt.flatMap(a)(double)
  val c = Opt.map(b)(input => input + 1) // Some(25)
```

However, the power comes from when something goes wrong:
```scala
def double(n: Int): Option[Int] = Option.None

@main def core() =
  val a = Opt.pure(12)
  val b = Opt.flatMap(a)(double) // None
  val c = Opt.map(b)(input => input + 1) // None
```

Let's look at a pratical usecase. Supppose you are 
writing a server in functional programming style. 

1. The client send you some api calls. 
1. You extract the body field of the request, 
1. you look up the username in the body by querying a database
1. You send the user's public info back to the client. 

Notice how the middle 2 steps may yield you a `nil`. You can't pass `nil`
to another function, so you are either going to do an early return 
(functional programming doesn't allow that) or you add tons of boilerplate 
error handling functions.

Instead of setting your hair on fire, we can use option monads.

```scala
def getBody(req: Request): Option[Body] = ...
def getUser(username: String): Option[User] = ...

// assuming we already implmeneted the chain syntax thing
Handler.onReq(req =>
  val result = Opt.pure(req)
    .flatMap(getBody)
    .flatMap(getUser)

  result match 
    case Option.Some(value) =>
      OKResponse(userInfo = value)
    case Option.None =>
      FailResponse("body or username does not exist")
)
```

Sometimes when you want to inject error information into 
an option monad, you can also use a result monad.
```scala
enum Result[+T, +E]:
  case Ok(value: T)
  case Err(err: E)

// insert monad implementation
```
This one works the same way as an option monad, when a link 
in the `map` & `flatMap` chain returns an `Err`, the entire chain returns
said `Err` and you can now extract the error info from 
the `Err` thing and do error handling.

## Finale: IO 

We have talked about the normal use of monads, now let's look at 
something that is quite different: IO monads.

For those who wants completely pure functional programming, IO monads
are used to achieve side effects in a pure codebase (IO operations like 
printing lines or querying an external database). How do we do that?

Unfortunately, we won't implement IO monads here, they are too complex 
to write in a guide like this, so just assume we implemented it.

Of course, you can use it like an identity monad:
```scala
def double(a: Int) = IO.pure(a * 2)

IO.pure(12)
  .map(input => input + 1)
  .flatMap(double) // IO(26)
```

But it can also be used like this:
```scala
def queryDB(query: Query): IO[String] = ...
def printLn(msg: String): IO[Unit] = ...

IO.pure("John")
  .map(a => Query(username = a))
  .flatMap(queryDB)
  .flatMap(printLn) // IO[Unit]
```

The intention of this code snippet is quite simple: we first make an IO
monad storing `"John"`, then we turn `"John"` into a `Query`, we then 
feed `Query` into `queryDB`, and finally we print the return of the 
query.

Note that instead of doing anything, it returns an `IO[Unit]`. The 
`IO[Unit]` does nothing, but it stores crucial informations that 
routhly goes like this: 
1. Start as `"John"`
1. Turn it into a `Query`
1. Perform side effect of querying the database
1. Print the result of the query

As you can see, it stores the instructions on what the computer should do.

Then the `IO[Unit]` gets returned from the main function and ran by the 
runtime. So the gist is, instead of doing the side effect immediately, 
your """pure""" codebase is just constructing a long step by step instruction 
telling the computer what it should do, then returning it to the runtime so the 
stuff actually gets ran. It is pure because you never ever did any side effect,
all you do is construct data (in this case the step by step instructions) 
and return the data. (The runtime is of course not pure, in a way doing the 
pure functional programming is just lying to yourself that your code is pure, 
it never is)

## Epilogue

Hope this helps with understanding how to use a monad. Now I will disappear to 
hide from the angry Haskell users out there trying to get me. Good luck and enjoy 
your now """pure""" and monad filled codebase.
