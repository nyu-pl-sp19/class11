# Class 11

## Object-Oriented Programming

In object-oriented programming (OOP) languages, a program defines a
set of objects that interact with each other to accomplish a specific
task (i.e. perform a specific computation). An object consists of a
collection of named values, called *fields*, as well as functions,
called *methods*, that perform computations over the values stored in
the fields, potentially by calling methods or accessing fields of
other objects. The fields and methods of an object are also referred
to as the object's *members*. In many ways, objects are similar to
records in languages like OCaml and struct values in C.

Central to most object-oriented languages is the notion of a *class*,
which can be viewed as a template for constructing objects that share
the same interface (the names and types of the objects' members) and
behavior (the implementations of the objects' methods). Classes serve
many of the same purposes as modules in languages like OCaml: data
encapsulation, information hiding, decomposition, and polymorphism.

We revisit Scala to study the basic concepts of OOP languages as Scala
realizes many of these concepts in a fairly clean manner. However, we
will comment on some of the design choices made in Scala vis-a-vis
other OOP languages.

### Objects

Suppose we want to create a representation of a point in a
two-dimensional Cartesian space as a value in a Scala program. We
could do this by creating an object instance that stores the two
coordinates of the point, say as `Double` values in two fields called
`first` and `second`. Here is how to do this in Scala:

```scala
object PointObj {
  val first = 1.0
  
  val second = 2.0
}
``` 

When this code is executed at run-time, then an object with fields
`first` and `second` is allocated on the heap, and a global variable
`PointObj` stores a reference to that object. The fields of the
object are initialized to the values `1.0` and `2.0` respectively. We can
access, e.g., the field `first` of that object as follows:

```scala
scala> PointObj.first
res0: Double = 1.0
```

Objects like `PointObj` that are declared using the `object` keyword are
also called *singleton objects*.  What if we want to create many
objects like `PointObj` that only differ in the values stored in the
fields `first` and `second`? This is where classes come into play.

### Classes

Here is a Scala class describing general point objects:

```scala
class Point(fst: Double, snd: Double) {
  val first = fst
  
  val second = snd
}
```

In essence, the class `Point` is like our definition of `PointObj`
earlier, except that it abstracts from the concrete values that are
stored in the fields `first` and `second`. These concrete values `1.0`
and `2.0` used in `PointObj` are now replaced by the
parameters `fst` and `snd` of the class definition.

In general, the name of the class in a class definition is followed by
a list of class parameters. The parameter list implicitly defines a
*primary constructor* of that class with a corresponding list of
parameters. A constructor is a special function that constructs an
object instance from the definition of the class.

Here is how we can use the constructor to create a point object:

```scala
scala> val p = new Point(1.0, 2.0)
p: Point = Point@1458e1cc

scala> p.first
res0: Double = 1.0

scala> val q = new Point(2.0, 3.0)
q: Point = Point@1613af12

scala> q.second
res1: Double = 3.0
```

The expression `new Point(1.0, 2.0)` creates a new *instance* of class
`Point` by calling the constructor of the class. The constructor
creates the object on the heap, with the fields `first` and `second`
initialized to `1.0` and `2.0`, respectively. The value `p` is then
bound to the address pointing to that object. We can then access,
e.g., the field `first` of `p` by writing `p.first`.

Using class parameters to initialize fields is such a common idiom
that Scala provides syntactic sugar for this specific usage of class
parameters. For example, we can write the above class more compactly
as follows


```scala
class Point(val first: Double, val second: Double)
```

Note that the class parameters are now prefixed with the keyword
`val`. The meaning of these declarations is that `first` and `second`
are fields of the class and, at the same time, serve as class
parameters that will be used to initialize those fields.

By default, equality on objects is defined as *reference
equality*:

```scala
scala> val p1 = new Point(1.0, 2.0)
p1: Point = Point@1458e1cc

scala> val p2 = new Point(1.0, 2.0)
p2: Point = Point@1613af12

scala> p1 == p2
res0: Boolean = false

scala> val p3 = p1
p3: Point = Point@1458e1cc

scala: p3 == p1
res1: Boolean = true
```

The default implementation of object equality can be changed per
class. More on that later.

### Methods

The methods of the instances of a class are defined as functions in the
body of the class. For instance, suppose we want to add a method
`print` to our `Point` objects that allows us to pretty print point
objects on standard output. Here is how this would look like:

```scala
class Point(val first: Double, val second: Double) {
  def print(): Unit = {
    println("Point(" + first + ", " + second + ")")
  }
}
```

Calling a method `m` of an object `e` can be done using the syntax
`e.m(a1, ..., an)` where `a1` and `an` are the actual arguments passed
to the formal parameters of the method `m` of `e`. The object that the
expression `e` before the `.` in a method call evaluates to is called
the *receiver object* (or just receiver) of the method call and we
refer to `e` as the *receiver expression* of the method call
expression `e.m(a1, ..., an)`. 

Note that a method has access to the fields of the object on which it
is called. E.g. in our implementation of `print`, we refer to the
fields `first` and `second` of the instance. In general, when a method
call is executed on the receiver object `o` that `e` evaluates to,
then any reference to a field declared in the class where `m` was
defined (and to which `o` belongs) will refer to the specific values
of those fields associated with `o`.

Here is how this looks for our example:

```scala
scala> val p = new Point(1.0, 2.0)
p: Point = Point@1458e1cc

scala> p.print()
Point(1.0, 2.0)
res0: Unit = ()
```

Here, when `print` is called on `p`, then the fields `first` and
`second` in the body of `print` refer to the specific values `1.0` and
`2.0` stored in those fields for the instance `p`.

From a technical perspective, we can view methods as *global*
functions that take the instance on which they operate as an
additional parameter. For example, the method `print` of class `Point`
can be viewed as defining the following global function:

```scala
def print(this: Point): Unit = {
  println("Point(" + this.first + ", " + this.second + ")")
}
```

In this view, a method call `e.m(a1, ...., an)` translates to a call
of the corresponding global function `m(e, a1, ..., an)`. In our
concrete example, `p.print()` translates to `print(p)`.

The name `this` is actually a keyword in Scala. A usage of `this`
within the body of a method of a class will be bound to the receiver
of the method call. This can be useful to disambiguate, e.g., between
formal parameters of a method and the fields of the instance on which
that method is called:

```scala
class Point(val first: Double, val second: Double) {
  def foo(first: Double): Unit = {
    println(first) // prints the formal parameter first of the method foo
    println(this.first) // prints the field first of the instance on
                        // which foo is called
  }
}
```

If we have an occurrence of `this` outside of a method as in the
following example:

```scala
class A {
  val self = this
}
```

then this occurrence refers to the implicit `this` parameter of the
constructor of the class. That is, it denotes the instance obtained when
instantiating the class with `new A()`. In particular, the following
expression evaluates to `true`:

```scala
val a = new A()
a == a.self
```

### Accessibility Modifiers

By default, all members (i.e. fields and methods) of objects are
accessible by all other objects. Information hiding and encapsulation
can be realized by modifying the accessibility of class or object
members using so-called *accessibility modifiers*.

For instance, suppose that we have an alternative implementation of
our `Point` class where the two coordinates are mutable fields that
can be updated by certain methods provided by the class. In such a
scenario, we would not want to give all other objects direct access to
these mutable fields (as this would allow other objects to modify the
contents of these fields, which may break invariants that the point
objects make about the values of these fields). Here is how this can
be done:

```scala
class MutablePoint(private var first: Double, private var second: Double) {
  def getFirst: Double = first
  def getSecond: Double = second
}
```

The accessibility modifier `private` is added to the field
declarations. This modifier ensures that the two fields are only
accessible from within the class `MutablePoint`. That is, the methods
of two instances of `MutablePoints` can access each others fields
`first` and `second` directly. However, the methods of instances of
other classes cannot. In our code example, the class `MutablePoint`
still provides indirect read access to the fields via the *getter*
methods `getFirst` and `getSecond`. However, no instance of another
class can assign new values to these fields.

The most important access modifiers available in Scala are as follows:

* `public` (default): every other object has access to the member

* `private`: only instances of the current class have access to the member

* `private[this]`: each instance only has access to its own version of
  the member, but not the ones of other instances of the same class.
  
* `protected`: each instance of the current class as well as all its
  subclasses have access to the member (more on subclasses later).

### Secondary Constructors

Secondary constructors are defined like ordinary methods within the
class body using the dedicated keyword `this` for the name of the
constructor method. For example, suppose we want to add a secondary
no-argument constructor to our class `Point` that default initializes
the components of a point to `0.0`. Then we can do this as follows:

```scala
class Point(val first: Double, val second: Double) {
  ...
  def this() {
    this(0.0, 0.0)
  }
}
```

```scala
scala> val p = new Point()
p: Point = Point@1458e1cc
scala> p.first
res1: Double = 0.0
```

### Inheritance and Subtype Polymorphism

Class inheritance is a fundamental concept in almost all
object-oriented programming languages. It describes the ability to
have an object or class 'specialize' another one, inheriting parent
data and behavior. The subclass (i.e. the inheriting class) defines a
*subtype* of the type of its superclass (i.e. the parent class it
inherits from). Here, we think of the type of the class as its
interface defined by all its members and their signatures (i.e. the
fields of the class with their types as well as its methods with their
parameter and return types).

The subclass can only extend its superclass by adding new
members. However, it cannot remove members that exist in the
superclass. Since the only way to interact with an object is by
accessing or calling its members, any operation that can be performed
with an object of the superclass can also be performed with an object
of the subclass. This leads to the *substitution principle* of
object-oriented languages: objects that belong to the subtype can be
used whenever an object of the supertype is expected. That is, one can
think of the objects of the subtype as forming a subset of the objects
of the supertype. This feature is also referred to as *subtype
polymorphism*. For example, consider the following code snippet:

```scala
class A(val x: Int)
class B(x0: Int, val y: Int) extends A(x0)

def f(a: A): Int = a.x

f(new B(1, 2))
```

Class `B` extends class `A`, thus forming a subtype relationship
between the types of the two classes. In Scala, this subtype
relationship is expressed by the notation `B <: A`.

Since `B` is a subtype of `A`, it is OK to call the function `f`,
which expects an `A` with a `B` object instead. In particular, the
access to the field `x` of `a` in the body of `f` can be safely
executed on `B` instances because class `B` inherits all members of
class `A`, including field `x`.

#### Overriding Methods

A particular feature of class inheritance is the ability of the subclass
to modify the behavior of the methods it inherits from its superclass by
*overriding* those methods.

As a motivation, let's return to our example of the point class:

```scala
class Point(val first: Double, val second: Double) {
  def print(): Unit = {
    println("Point(" + first + ", " + second + ")")
  }
}
```

It appears as if the class `Point` does not extend any other
class. However, if a class does not explicitly extend another class,
then it extends the class `AnyRef` by default. That is, the above
class definition is actually interpreted as follows:

```scala
class Point(val first: Double, val second: Double) extends AnyRef {
  def print(): Unit = {
    println("Point(" + first + ", " + second + ")")
  }
}
```

The class `AnyRef` is a predefined class in Scala that provides
certain useful methods such as the method `equals` that determines a
default implementation for equality on objects (reference equality)
and the method `toString` that converts an object to a string
representation. The method `toString` is also used by the Scala REPL
to print object values. 

By default, the textual representation of objects consists of the name
of the object's class, followed by a unique object ID. We can modify
the way objects of a specific class are printed, by overriding the
`toString` method:

```scala
class Point(val first: Double, val second: Double) {
  override def toString(): String = "Point(" + first + ", " + second + ")"
  
  def print(): Unit = println(toString())
}
```

If we want to override a method in a Scala class, we have to
explicitly say so by using the `override` qualifier:

The pretty printer in the REPL will now use the new
`toString` method to print `Point` objects:

```scala
scala> val p = new Point(1.0,2.0)
p: Point = Point(1.0, 2.0)
```

The question is now, if we have a method call expression `e.m(a1, ...,
an)` in the program, which version of `m` is being called? To answer
this question we need to distinguish between the types inferred at
compile-time and those that are actually observed at run-time.

#### Static vs. Dynamic Types and Dynamic Dispatch

A superclass `A` is open for extension, i.e., it allows behavior to be
extended without modifying `A`'s source code by adding and overriding
methods in the subclasses of `A`. To understand the semantics of calls
to overridden methods, we have to understand the difference between
*static* and *dynamic* types.

The static type of an expression in a program is the type that the
compiler infers for that expression at compile-time. The static type
determines how we can interact with the result value of the expression
in the program (i.e. which of its fields and methods we can
access). If an expression `e` has static type `A`, then the compiler
will only allow us to access the members of type `A` on the result
value of `e`.

On the other hand, the dynamic type of an expression is the actual
type of the value obtained when the expression is evaluated at
run-time. For instance, consider the following code snippet:

```scala
class A(val x: Int) {
  def m(): Int = x
}
class B(x0: Int, val y: Int) extends A(x0) {
  override def m(): Int = x + y
}

def f(a: A) = a.m()

val a: A = new B(1, 2)
a.m()
```

The static type of `a` in the last line is `A`. The compiler infers
this type from the type annotation in the declaration of `a` on the
previous line. On the other hand, the dynamic type of `a` on the last
line is `B` since when `a` is evaluated at run-time, it refers to the
`B` instance created on the previous line. 

The behavior of a call to an overridden method such as `m` on the last
line is determined by the dynamic type of the receiver expression of
the method call. The call goes to the most recent implementation of
the method in the subtype hierarchy, starting from the dynamic type of
the receiver. Thus, in the example, the call `a.m()` on the last line
goes to `B.m` and not `A.m`. The last line therefore evaluates to `3`
and not `1`. This semantics of method calls is referred to as *dynamic
dispatch*. Methods that are dynamically dispatched are also called
*virtual methods*. In Scala, all public and protected methods of
classes are virtual by default whereas private methods are non-virtual.

Note that a receiver expression can have more than one dynamic
type. For instance, if we call the function `f` with an `A` instance,
the dynamic type of `a` in `f` for this call will be `A` and the call
`a.m()` in the body of `f` will go to `A.m`. On the other hand, if we
call `f` with a `B` instance, then the dynamic type of `a` in `f` for
this call will be `B` and the call to `a.m()` in the body of `f` will
go to `B.m`.

Scala's type system guarantees that if a program is well-typed, then
the dynamic types of an expression `e` in a program are always
subtypes of the static type of `e`. That is, the type system ensures
that any assumptions that the compiler makes about the members of `e`
based on the inferred static type are actually satisfied at
run-time. In particular, if you write `e.x.m(0)`, then the compiler will
ensure that the object that `e` evaluates to at run-time will indeed
have a field `x` and that this field denotes some other object 
that has a method `m` which takes a parameter of type `Int`.

### Singleton and Companion Objects

It is often useful to declare *factory methods*. Such methods are used
to simplify the construction of objects that involve complex
initialization code.  A good place to declare such factory methods is
the *companion object* of the class. The companion object of a class `C` is
a singleton object whose name is also `C`.  The companion object of `C` has access
to all private members of instances of `C`. 

```scala
class Point(val first: Int, val second: Int) {
  ...
}
object Point {
  def make(fst: Int, snd: Int) = new Point(fst, snd)
}
```

We can access members of companion objects as with any other singleton object:

```scala
scala> def p = Point.make(3.0, 4.0)
p: Point = Point(3.0, 4.0)
```

Note that the members of the companion object of class `C` are
conceptually equivalent to what is known as *static members* of class
`C` in languages like Java and C++.

### The `apply` Method

Methods with the name `apply` are treated specially by the Scala
compiler. For example, if we rename the factory method `make` in our
companion object for the `Point` class to `apply`

```scala
object Point {
  def apply(fst: Int, snd: Int) = new Point(fst, snd)
}
```

then we can call this method simply by referring to the `Point`
companion object, followed by the argument list of the call to `apply`
(omitting the method name `apply` in the call):

```scala
scala> def p = Point(3.0,4.0)
p: Point = Point(3.0, 4.0)
```

This is equivalent to the following explicit call to the
`apply` method:

```scala
scala> def p = Point.apply(3.0,4.0)
p: Pair = Point(3.0, 4.0)
```

The compiler automatically expands `Point(3.0,4.0)` to
`Point.apply(3.0, 4)`. That is, objects with an `apply` method can be
used as if they were functions. This feature is particularly useful to
enable concise calls to factory methods. In fact, factory methods for
the data structures in the Scala standard library are typically
implemented using `apply` methods in companion objects. We will see
later that `apply` methods also give us a nice way of realizing
higher-order functions in Scala.

### Implementing Subtype Polymorphism

In the following, we discuss how subtype polymorphism and dynamic
dispatch are implemented by compilers for OOP languages when they
compile OOP code to executable machine code or byte code.

The goal of this exercise is two-fold:

1. You will obtain a better understanding of what happens when an OOP
   program is executed and what is the performance overhead associated
   with using certain OOP features.
   
1. You will learn how to simulate OOP techniques in languages that do
   not support object-oriented programming directly.

Specifically, we cover two important concepts:

* Object data layout in memory

* Virtual method tables (aka *vtables*)

To understand how virtual method dispatch is implemented we need to
think about the data layout of objects in memory. Towards that end, we
will first understand inheritance and virtual methods by looking at
the data layout of objects and vtables. 

#### Object Data Layout in Memory

In the following, we will answer these questions:

* For any given object, how is the data organized in memory?

* How does the run-time find a particular data member of an instance?

* How do these things work when dealing with inheritance hierarchies?

* In particular, how is dynamic dispatch realized?

To get started, consider the following simple Scala classes:

```scala
class A(val x: Int, val y: Int)
class B(x1: Int, y1: Int, val z: Int) extends A(x1, y1)
```

Every instance of a class is represented by a contiguous block of
memory referred to as the object's data layout. The data layout
contains the instance-specific values for all the fields of the class
(including fields inherited from the superclass). The structure of the
data layouts of instances of the same class are identical. They only
differ in the specific values stored in the fields. 

What is most important for subtype polymorphism to work is that the
order in which the fields are stored in the data layout. If class `B`
extends class `A`, then the fields in `A`'s data layout must be
ordered such that all fields that are inherited from `A` **occur
before** the new fields added in `B`. Moreover, the order of the
inherited fields must **exactly match** the order of those fields in
the data layout of `A`.

Here is how this would look like for our example:

```
     A Instance:
   0┌─────────────┐
    │ value of x  │
   4├─────────────┤> members of A
    │ value of y  │
    └─────────────┘

     B Instance:
   0┌─────────────┐
    │ value of x  │
   4├─────────────┤> members of A
    │ value of y  │
   8├═════════════┤
    │ value of z  │  additional members of B
    └─────────────┘
```

Note that:

* Each data member can be accessed via a fixed offset from the base
  address of the data layout. The offset is determined by the number
  of bytes needed to represent a value of the type of that field
  (e.g. 4 bytes for `Int` values and 8 bytes for any type derived from
  `AnyRef` assuming a 64-bit architecture).

* Subclass objects have the same memory layout as superclass objects
  with additional space for the subclass fields that succeeds the space
  for the superclass fields.

* Objects of type `B` can be polymorphically operated on as if they
  were objects of type `A`, since the offsets of the subclass fields
  are the same. E.g.  the expression `a.y` where `a` has static type
  `A` would translate to:
  
  1. Take the address stored in the reference `a`, which points to the
     base of an object data layout.
  
  2. Add to it the offset of field `y` in the data layout of `A`,
     which is 4.
     
  3. Dereference the resulting address to retrieve the value of field
     `y` in the object referred to by `a`.

  These steps also work if the dynamic type of `a` is `B` since the
  relative offset of the entry for `y` in the `B` data layout is the
  same as in the `A` data layout.

* Private fields are also included in the data layout because they
  contain instance-specific data. When a class `B` extends a class `A`
  with a private field `x`, the field `x` must also be included in
  `B`'s data layout, even though `B` cannot access `x` directly. The
  reason for this is that private fields of `A` can still be accessed
  indirectly in a `B` instance by calling a public or protected method
  of `A` on it. Example:
  
  ```scala
  class A(private val x: Int) {
    def m(y: Int): Int = x + y
  }
  class B extends A(0) {
    ...
  }
  
  val b = new B
  b.m(1) // accesses field x on a B instance
  ```

**Question:** If we have polymorphic data structures of variable
sizes, how should we pass the data? **Answer:** by reference. Hence in
Scala (and Java), all objects are stored on the heap and passed by
reference.

Note that because the data layouts of subclass instances are
compatible with the data layout of superclass instances, there is no
need to check at run-time whether the actual dynamic type of an
instance is consistent with the expected static type. 

Now let's add some methods to `A` and `B`:
  
  ```scala
    class A(val x: Int, val y: Int) { 
      def m1() = { ... }
      def m2() = { ... }
    }
    class B(x1: Int, y1: Int, val z: Int) extends A(x1, y1) {
      override def m2() = { /* overriding A.m2 */ ... }
      def m3() = { ... }
    }
  ```
  
So should we take the same approach for methods as for data? That is,
for each method declared in a class, we could add an entry to the data
layout that stores a pointer to the implementation of that
method. When we override a method in a subclass, we simply change the
pointer at the appropriate entry in the subclass data layout to point
to the new implementation. This would give us the following
memory representation of an A and a B instance at run-time.

```
      A Instance:
    0┌─────────────┐
     │ value of x  │
    4├─────────────┤
     │ value of y  │
    8├─────────────┤                    ┌──────────────┐
     │ ptr. to m1  │───────────────────>│impl. of A.m1 │
   16├─────────────┤                    └──────────────┘
     │ ptr. to m2  │────────┐  ┌──────────────┐     ^
     └─────────────┘        └─>│impl. of A.m2 │     │
                               └──────────────┘     │
      B Instance:                                   │ 
    0┌─────────────┐                                │
     │ value of x  │                                │
    4├─────────────┤                                │
     │ value of y  │                                │
    8├─────────────┤                                │
     │ ptr. to m1  │────────────────────────────────┘ 
   16├─────────────┤           ┌─────────────┐
     │ ptr. to m2  │──────────>│impl. of B.m2│
   32├═════════════┤           └─────────────┘
     │ value of z  │
   36├─────────────┤           ┌─────────────┐
     │ ptr. to m3  │──────────>│impl. of B.m3│
     └─────────────┘           └─────────────┘
```

A call `a.m1()` would compile to

1. Take the base address of the data layout pointed to by `a`.

1. Add to the base address the relative offset of the pointer to `m1`
   in the data layout for `A`, which is 8.
   
1. Dereference to the resulting address to retrieve the pointer to the
   correct implementation of `m1`.
   
1. Call the method found at the retrieved address.

Again, this compiled code would work polymorphically for both `A` and
`B` instances and implement the dynamic dispatch correctly.

However, if we used this approach, then for every method added in a subclass
the size of each instance would grow by the size of one pointer. Consequently

* object creation would be slower, and

* memory consumption would be higher.

#### Virtual Method Tables (vtables)

We can avoid wasting space for each method in each object instance by
adding an extra level of indirection. When a class defines a virtual
method, the compiler adds a hidden member variable to the data layout
of that class.  This hidden member variable is called the *virtual
pointer* (aka *vpointer*).

The vpointer points to the *virtual method table* (aka *vtable*). Each
class has its own vtable which is shared by all instances of that
class. That is, there is exactly one vtable per class stored in
memory at run-time. The vtable is an contiguous memory block that
stores pointers to functions that implement the virtual methods of the
class.

As with the object data layout, the vtable of a subclass `B` has the
same layout as the vtable of its superclass `A`, with additional
entries for the subclass' virtual methods appended to it. Thus, the
relative offsets of the pointers to the shared methods from the start
of the vtable are the same in both `A`'s and `B`'s vtable.

The vtable of a subclass is created by copying the entries from the
vtable of the superclass and changing the pointers of overridden
methods to point to the new implementations. At instance creation (at
runtime) the vpointer of the instance will be set to point to the
right vtable of the instance's class.

For our example, we would get the following memory representation at run-time

```
      A Instance:                A vtable:
    0┌─────────────┐        ┌> 0┌────────────┐                    ┌─────────────┐
     │ vptr        │────────┘   │ ptr. to m1 │───────────────────>│impl. of A.m1│
    8├─────────────┤           8├────────────┤                    └─────────────┘
     │ value of x  │            │ ptr. to m2 │────────┐  ┌─────────────┐   ^
   12├─────────────┤            └────────────┘        └─>│impl. of A.m2│   │
     │ value of y  │                                     └─────────────┘   │
     └─────────────┘                                                       │
                                                                           │
      B Instance:                B vtable:                                 │
    0┌─────────────┐        ┌> 0┌────────────┐                             │
     │ vptr        │────────┘   │ ptr. to m1 │─────────────────────────────┘ 
    8├─────────────┤           8├────────────┤           ┌─────────────┐
     │ value of x  │            │ ptr. to m2 │──────────>│impl. of B.m2│
   12├─────────────┤          16├════════════┤           └─────────────┘ 
     │ value of y  │            │ ptr. to m3 │────────┐  ┌─────────────┐
   16├═════════════┤            └────────────┘        └─>│impl. of B.m3│
     │ value of z  │                                     └─────────────┘
     └─────────────┘
```

Note that at run-time, the vpointer of all `A` instances will point to
the same vtable data structure for class `A`, and similarly for `B`.

When a virtual method call expression `e.m()` is executed, the
low-level code first evaluates `e` to obtain the object instance `o`
on which the call is executed. It then looks up the vtable of the
`o`'s dynamic type via the vpointer, and then looks up the method's
implementation for that type via the corresponding pointer in the
vtable. The offset for the vtable lookup is calculated at compile-time
based on the static type of `e`. The two pointer lookups together
realize the dynamic dispatch.

Virtual methods thus add some runtime overhead:

* The data layout of each object grows by the size of one pointer (to
  store the vpointer).

* Each virtual method call involves a constant overhead of two pointer
  lookups compared to a regular function call (first, to retrieve the
  address of the correct vtable, second to retrieve the address of the
  correct method implementation to which the call should be
  dispatched).

* Due to the additional indirection of calling methods via pointers,
  the compiler also has fewer opportunities for applying static code
  optimizations such a inlining function calls. Though this is
  mitigated by just-in-time optimization techniques in modern
  run-time environments like the Java Virtual Machine.

Note that private methods are not included in vtables because they are
not dynamically dispatched.
