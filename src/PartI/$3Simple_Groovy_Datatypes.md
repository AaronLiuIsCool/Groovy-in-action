# Chapter 3. Simple Groovy datatypes

This chapter covers

* Groovy’s approach to typing
* Operators as method implementations
* Strings, regular expressions, and numbers

```Do not worry about your difficulties in mathematics. I can assure you mine are still greater.```

Albert Einstein

Groovy supports a limited set of datatypes at the language level; that is, it offers constructs for literal declarations
and specialized operators. This set contains the simple datatypes for strings, regular expressions, and numbers, as well 
as the collective datatypes for ranges, lists, and maps. This chapter covers simple datatypes; the next chapter 
introduces collective datatypes.

Before we go into details, we’ll talk about Groovy’s general approach to typing. With this in mind, you can appreciate 
Groovy’s approach of treating everything as an object and all operators as method calls. You’ll see how this improves 
the level of object orientation in the language compared to Java’s division between primitive types and reference types.

We then describe the natively supported datatypes individually. By the end of this chapter, you’ll be able to 
confidently work with Groovy’s simple datatypes and have a whole new understanding of what happens when you write 1+1.

## 3.1. OBJECTS, OBJECTS EVERYWHERE
In Groovy, everything is an object. It is, after all, an object-oriented language. Groovy doesn’t have the slight 
“fudge factor” of Java, which is object-oriented apart from some built-in types. To explain the choices made by 
Groovy’s designers, we’ll first go over basics of Java’s type system. We’ll then explain how Groovy addresses the 
difficulties presented, and finally examine how Groovy and Java can still interoperate with ease due to automatic boxing 
and unboxing where necessary.

### 3.1.1. Java’s type system: primitives and references

Java distinguishes between primitive types (such as boolean, short, int, double, float, char, and byte) and 
reference types (such as Object and String). There’s a fixed set of primitive types, and these are the only types that 
have value semantics—where the value of a variable of that type is the actual number 
(or character, or true/false value). You cannot create your own value types in Java.

Reference types (everything apart from primitives) have reference semantics—the value of a variable of that type is 
only a reference to an object. Readers with a C/C++ background may wish to think of a reference as a pointer—it’s 
a similar concept. If you change the value of a reference type variable, it has no effect on the object it was 
previously referring to—you’re just making the variable refer to a different object or to no object at all. The 
reverse is true too: changing the contents of an object doesn’t affect the value of a variable referring to that object.

You cannot call methods on values of primitive types, and you cannot use them where Java expects objects of type 
java.lang.Object. For each primitive type, Java has a wrapper type—a reference type that stores a value of the primitive
type in an object. The wrapper for int, for example, is java.lang.Integer.

Conversely, operators such as * in 3 * 2 or a * b aren’t supported for arbitrary[1] reference types in Java, but only
for primitive types (with the notable exception of +, which is also supported for strings).

#### From Java 5 onward, the autoboxing feature may kick in to unbox the wrapper object to its primitive payload and apply the operator.
{Aaron notes: Above is an important design.}

The Groovy code in listing 3.1 calls methods on seemingly primitive types (first with a literal declaration and then 
on a variable), which isn’t allowed in Java where you need to explicitly create the integer wrapper to convince the 
compiler. While calling + on strings is allowed in Java, calling the - (minus) operator is not. Groovy allows both.

Listing 3.1. Groovy supports primitive methods and object operators
```
(60 * 60 * 24 * 365).toString(); // invalid Java

int secondsPerYear = 60 * 60 * 24 * 365;
secondsPerYear.toString(); // invalid Java

new Integer(secondsPerYear).toString();

assert "abc" - "a" == "bc" // invalid Java
```

### 3.1.2. Groovy’s answer: everything’s an object
To make Groovy fully object-oriented, and because at the JVM level Java doesn’t support object-oriented operations 
such as method calls on primitive types, the Groovy designers decided to do away with primitive types. When Groovy 
needs to store values that would have used Java’s primitive types, Groovy uses the wrapper classes already provided 
by the Java platform. Table 3.1 provides a complete list of these wrappers.

#### Table 3.1. Java’s primitive datatypes and their wrappers
Primitive type                      Wrapper type                        Description

byte	                            java.lang.Byte	                    8-bit signed integer
short	                            java.lang.Short	                    16-bit signed integer
int	                                java.lang.Integer	                32-bit signed integer
long	                            java.lang.Long	                    64-bit signed integer
float	                            java.lang.Float	                    Single-precision (32-bit) floating-point value
double	                            java.lang.Double	                Double-precision (64-bit) floating-point value
char	                            java.lang.Character	                16-bit Unicode character
boolean	                            java.lang.Boolean	                Boolean value (true or false)

Any time you see what looks like a primitive literal value (the number 5, for example, or the Boolean value true) 
in Groovy source code, that’s a reference to an instance of the appropriate wrapper class. For the sake of brevity and 
familiarity, Groovy allows you to declare variables as if they were primitive-type variables. Don’t be fooled—the type 
used is really the wrapper type. Strings and arrays aren’t listed in table 3.1 because they’re already reference types, 
not primitive types—no wrapper is needed.

While we have the Java primitives under the microscope, so to speak, it’s worth examining the numeric literal formats 
that Java and Groovy each use. They’re slightly different because Groovy allows instances of java.math.BigDecimal and 
java.math.BigInteger to be specified using literals in addition to the usual binary floating-point types. 
{Aaron notes: Above is an important design.}
Table 3.2 gives examples of each of the literal formats available for numeric types in Groovy.

#### Table 3.2. Numeric literals in Groovy
Type                                Example literals

java.lang.Integer[a]	            15, 0x1234ffff, 0b00110011, 100_000_000
java.lang.Long	                    100L, 200l[b]
java.lang.Float	                    1.23f, 4.56F
java.lang.Double	                1.23d, 4.56D
java.math.BigInteger	            123g, 456G
java.math.BigDecimal	            1.23, 4.56, 1.4E4, 2.8e4, 1.23g, 1.23G

a. Project Coin introduced binary literals and underscores within integer constants as part of Java 7. You can use these 
features within Groovy even using older versions of the JDK.

b. The use of lowercase l as a suffix indicating Long is discouraged, as it can look like a 1 (number one). There’s no 
difference between the uppercase and lowercase versions of any of the suffixes.

Notice how Groovy decides whether to use a BigInteger or a BigDecimal to hold a literal with a G suffix depending on the 
presence or absence of a decimal point. Notice also how BigDecimal is the default type of noninteger literals. 
BigDecimal will be used unless you specify a suffix to force the literal to be a Float or a Double.

### 3.1.3. Interoperating with Java: automatic boxing and unboxing
Converting a primitive value into an instance of a wrapper type is called boxing in Java and other languages that 
support the same notion. The reverse action—taking an instance of a wrapper and retrieving the primitive value—is called 
unboxing. Groovy performs these operations automatically for you where necessary. This is primarily the case when you 
call a Java method from Groovy. This automatic boxing and unboxing is known as autoboxing.
{Aaron notes: Above is an important design.}

You’ve seen that Groovy is designed to work well with Java, so what happens when a Java method takes primitive 
parameters or returns a primitive return type? How can you call that method from Groovy? Consider the existing method 
in the java.lang.String class: int indexOf (int ch). You can call this method from Groovy like this:
```
assert 'ABCDE'.indexOf(67) == 2
```
From Groovy’s point of view, you’re passing an Integer containing the value 67 (the Unicode value for the letter C), 
even though the method expects a parameter of primitive type int. Groovy takes care of the unboxing. The method returns 
a primitive type int that’s boxed into an Integer as soon as it enters the world of Groovy. That way, you can compare 
it to the Integer with value 2 back in the Groovy script.

#### Figure 3.1 shows the process of going from the Groovy world to the Java world and back.

#### Figure 3.1. Autoboxing in action: an Integer parameter is unboxed to an int for the Java method call, and an int return value is boxed into an Integer for use in Groovy.

All of this is transparent—you don’t need to do anything in the Groovy code to enable it. Now that you understand 
autoboxing, the question of how to apply operators to objects becomes interesting. We’ll explore this question next.

### 3.1.4. No intermediate unboxing
If in 1+1 both numbers are objects of type Integer, you may be wondering whether those Integer objects are unboxed to 
execute the plus operation on primitive types.

The answer is no: Groovy is more object-oriented than Java. It executes this expression as 1.plus(1), calling the plus() 
method of the first Integer object, and passing[2] the second Integer object as an argument. The method call returns an 
Integer object of value 2.
{Aaron notes: Above is an important design.}

#### 2 The phrase “passing an object” is short for “passing a reference to an object.” In Groovy and Java alike, only references are passed as arguments: objects themselves are never passed.

This is a powerful model. Calling methods on objects is what object-oriented languages should do. It opens the door 
for applying the full range of object-oriented capabilities to those operators.

Let’s summarize. No matter how literals (numbers, strings, and so forth) appear in Groovy code, they’re always objects.
Only at the border to Java are they boxed and unboxed. Operators are shorthand for method calls. Now that you’ve seen
how Groovy handles types when you tell it what to expect, let’s examine what it does when you don’t give it any type
information.

## 3.2. THE CONCEPT OF OPTIONAL TYPING
So far, we haven’t used any type declarations in the sample Groovy scripts—or have we? Well, we haven’t used them in the
way that you’re familiar with in Java. We assigned strings and numbers to variables and didn’t care about the type. 
Behind the scenes, Groovy implicitly assumes these variables to be of static type java.lang.Object. This section 
discusses what happens when a type is specified, and the pros and cons of doing it either way.

### 3.2.1. Assigning types
Groovy offers the choice of explicitly specifying variable types just as you do in Java. Table 3.3 gives examples of 
optional type declarations. It’s tricky—anything talking about a “type declaration” makes us think it’s a type being 
declared, not a variable. What we’re really talking about is “variable declarations using optional typing,” but that’s 
a mouthful. We’re normally an absolute stickler for getting terminology right, but if you’d like to fudge this slightly 
for the sake of more readable text, that’s fine.

#### Table 3.3. Example Groovy statements and the resulting runtime type
```
Statement                       Type of value                           Comment

def a = 1	                    java.lang.Integer	                    Implicit typing
def b = 1.0f	                java.lang.Float	 
int c = 1	                    java.lang.Integer	                    Explicit typing using the Java primitive type names
float d = 1	                    java.lang.Float	 
Integer e = 1	                java.lang.Integer	                    Explicit typing using reference type names
String f = '1'	                java.lang.String	 
```
The def keyword is used to indicate that no particular type is specified.

As we stated earlier, it doesn’t matter whether you declare a variable to be of type int or Integer. Groovy uses the
reference type (Integer) either way.

It’s important to understand that regardless of whether a variable’s type is explicitly declared, the system is type 
safe. Unlike untyped languages, Groovy doesn’t allow you to treat an object of one type as an instance of a different 
type without a well-defined conversion being available. You could never assign a java.util.Date to a reference of type 
java.lang.Number, in the hope that you’d end up with an object that you could use for calculation. That sort of behavior 
would be dangerous, which is why Groovy doesn’t allow it any more than Java does.
{Aaron notes: Above is an important design.}

### 3.2.2. Dynamic Groovy is type safe
We’ll first look at Groovy’s default dynamic behavior. It’s important to understand that even when using all of its 
dynamic capabilities, Groovy is providing full type safety at runtime. In chapter 10, we’ll explore how to make Groovy 
provide more type checking at compile time to match and even exceed the kind of checking you might expect from Java.
{Aaron notes: Above is an important design.}

The web is full of heated discussions of whether static or dynamic typing is “better” while it often remains unclear 
what either should actually mean. Static is often associated with the appearance of type markers in the code. For 
instance, code such as

```String greeting = readFromConsole()```
is often considered static because of the String type marker, while unmarked code like

```def greeting = readFromConsole()```
is usually deemed dynamic. But it isn’t as simple as that. In languages that support type inference[3] and have no 
dynamic behavior capabilities, it might be possible to fully statically type check the latter code, while in a fully 
dynamic language, it’s not possible at compile time to type check the former code even with type markers. This is 
because in general solely dynamic languages cannot tell what type the readFromConsole() method will eventually 
return,[4] so there’s little point in doing many of the traditional compile-time checks.[5]
```
3 And indeed Groovy, when using @TypeChecked or @CompileStatic, is one of them!

4 It may, for example, be intercepted, relayed, or replaced by a different method.

5 Groovy will still do some compile-time checks even when compiling dynamically. For instance, if you declare that a 
class implements an interface, Groovy requires that at compile time it contains the methods from the interface.
```
By default, Groovy is very much a dynamic language. You can safely leave out type markers (and also type casts) in most
scenarios and know that Groovy will do the appropriate runtime checks to ensure type safety when required. Because type
markers are optional in Groovy, that concept is often called optional typing. The types are still there, of course, but 
you can choose not to make them explicit in your code.

All of that may sound as if type markers are superfluous, but they play an important role not only at runtime—for the 
method dispatch as you’ll see in chapters 7 and 8—but also for our current concern: type-safe assignments.

Groovy uses type markers to enforce the Java type system at runtime. Yes, you’ve read this correctly: Groovy enforces 
the Java type system! But it only does so at runtime, where Java does so with a mixture of compile-time and runtime 
checks. Java enforces the type system to a large extent at compile time based on static information, which gives static 
typing its second meaning. The fact that Java does part of the work at runtime can easily be inferred from the fact that 
Java programs can still raise ClassCastExceptions and other runtime typing errors.

All this explains why the Groovy compiler[6] takes no issue with
```
6 Your IDE will present you a big warning, though. It can apply additional logic like data flow analysis and type 
inference to even discover more hidden assignment errors. It’s your responsibility as a developer on how to deal with 
these warnings.
```
```
Integer myInt = new Object()
println myInt
```
but when running the code, the cast from Object to Integer is enforced and you’ll see
```
org.codehaus.groovy.runtime.typehandling.GroovyCastException:
    Cannot cast object 'java.lang.Object@5b0bc6'
    with class 'java.lang.Object' to class 'java.lang.Integer'
```
In fact, this is the same effect you see if you write a typecast on the right-hand side of the assignment in Java.

Consider this Java code:
```
Integer myInt = (Integer) returnsObject(); // Java!
```
The Java compiler will check whether returnsObject() returns an object of a type that can sensibly be cast to Integer. 
Let’s assume that the declared return type is Object. That makes Object the compile-time type[7] of the returnsObject() 
reference. The hope is that at runtime it’ll yield an Integer, which becomes its runtime type.[8]
```
7 This is usually also called the static type but we avoid this term here to avoid further confusion.

8 Often called the dynamic type—a term we avoid for the same reason.
```
The Groovy code
```
Integer myInt = returnsObject()
```
is the exact equivalent of the preceding Java code as far as the type handling is concerned. The Groovy compiler inserts 
type-casting logic for you that makes sure that the right-hand side of an assignment is cast to the type of the 
left-hand side. Consequently, when using the dynamic programming style as in

```
def myInt = returnsObject()
```
you’d cast to Object because that’s assumed when def is used. But this can never have any effect because every object 
is at least of type Object and Groovy optimizes the cast away.

Declared types give you a number of benefits. They’re means of documentation and communication, but most of all, they enable you to reason about your code. Consider this code snippet:
```
Integer myInt = returnsObject()
println(++myInt)
```
The second line is guarded by the first line; there’s no way that it’d ever be called if myInt wasn’t of type Integer. 
You can reason that the ++ operator will be found and work as expected.

As a second example, consider a method definition with a parameter that bears a type marker:
```
def printNext(Integer myInt) {
    println(++myInt)
}
```
There’s no possible way that this method could ever be called with an argument that isn’t of type Integer! Even though 
the compiler accepts code like printNext(new Object()), this will never result in calling the previous method. And now 
to a common misconception.

Groovy types aren’t dynamic, they never change
{Aaron notes: Above is an important design.}
If we could make the ink blink, we would! The word “dynamic” doesn’t mean that the type of a reference, once declared, 
can ever change. Once you’ve declared Integer myInt, you cannot execute myInt = new Object(). This will throw a 
GroovyCastException. You can only assign a value, which Groovy can cast to an Integer. As you see, the phrase 
“dynamic typing” can be misleading and is best avoided.
{Aaron notes: Above is an important design.}

Type declarations and type casts also play an important role in the Groovy method dispatch that we’ll examine in more 
detail in chapters 7 and 8. Casts come with some additional logic to make development easier.

### 3.2.3. Let the casting work for you
To complete the picture, Groovy actually applies convenience logic when casting, which is mainly concerned with 
casting primitive types to their wrapper classes and vice versa, arrays to lists, characters to integers, Java’s type
widening for numeric types, applying the “Groovy truth” (see chapter 6) for casts to boolean, calling toString() for 
casts to string, and so on. The exhaustive list can be looked up in DefaultTypeTransformation.castToType.

Two notable features are baked into the Groovy type casting logic that may be surprising at first, but make for really 
elegant code: casting lists and maps to arbitrary classes. The following listing introduces these features by creating 
Point, Rectangle, and Dimension objects.

Listing 3.2. Casting lists and maps to arbitrary classes
```
import java.awt.*

Point topLeft = new Point(0, 0) // classic
Point botRight = [100, 100] // List cast
Point center = [x:50, y:50] // Map cast

assert botRight instanceof Point
assert center instanceof Point

def rect = new Rectangle()
rect.location = [0, 0] // Point
rect.size = [width:100, height:100] // Dimension
```
As you see from the listing, implicit runtime casting can lead to very readable code, especially in cases like property
assignments where Groovy knows that rect.size is of type java.awt.Dimension and can cast your list or map of constructor
arguments onto that. You don’t have to worry about it: Groovy infers the type for you.

We’ve seen the value of type markers and pervasive casting. But because Groovy offers optional typing, what is the use 
case for omitting type markers?

### 3.2.4. The case for optional typing
Omitting type markers isn’t only convenient for the lazy programmer who does adhoc scripting, but is also useful for 
relaying and duck typing. Suppose you get an object as the result of a method call, and you have to relay it as an 
argument to some other method call without doing anything with that object yourself:
```
def node = document.findMyNode()
log.info node
db.store node
```
In this case, you’re not interested in finding out what the heck the actual type and package name of that node are. 
You’re spared the work of looking them up, declaring the type, and importing the package. 
You also communicate: “That’s just something.”

The second use of unmarked typing is calling methods on objects that have no guaranteed type. This is often called duck 
typing, and it allows the implementation of generic functionality with high reusability.

Duck typing
As coined by the dynamic language community, “If it walks like a duck and quacks like a duck, it must be a duck.” Weakly 
typed languages usually let you call any method or access any property on an object, even if you don’t know at compile 
time or even at runtime that the object is of a known type that contains that method or property. This means you know 
the kind of objects you expect will have the relevant signature or property. It’s an assumption. If you can call the 
method or access the property, it must be the type you were expecting—hence, it’s a duck because it walks and quacks 
like a duck!

Duck typing implies that as long as an object has a certain set of method signatures, it’s interchangeable with any 
other object that has the same set of methods, regardless of whether the two have a related inheritance hierarchy.

For programmers with a strong Java background, it’s not uncommon to start programming Groovy almost entirely using type 
declarations, and gradually shift into a more dynamic mode over time. This is legitimate because it allows everybody to 
use what they’re confident with.

Rule of Thumb
Experienced Groovy programmers tend to follow this rule of thumb: as soon as you think about the type of a reference, 
declare it; if you’re thinking of it as “just an object,” leave the type out.
{Aaron notes: Above is an important design.}

Whether or not you declare your types, you’ll find that Groovy lets you do a lot more than you may expect. Let’s start 
by looking at the ability to override operators.

## 3.3. OVERRIDING OPERATORS
Overriding refers to the object-oriented concept of having types that specify behavior and subtypes that override this 
behavior to make it more specific. When a language bases its operators on method calls and allows these methods to be 
overridden, the approach is called operator overriding.

It’s more conventional to use the term operator overloading, which means almost the same thing. The difference is that 
overloading suggests, at least to many Java programmers, that you have multiple implementations of a method (and thus 
the associated operator) that differ only in their parameter types.

We’ll show you which operators can be overridden, show a full example of how overriding works in practice, and offer 
guidance on the decisions you need to make when operators work with multiple types.

### 3.3.1. Overview of overridable operators
As you saw in section 3.1.2, 1+1 is a convenient way of writing 1.plus(1). This is achieved by class Integer having an 
implementation of the plus method.

This convenient feature is also available for other operators. Table 3.4 shows an overview.
```
                                Table 3.4. Method-based operators
Operator        Name                Method                      Works with

a + b	        Plus	            a.plus(b)	                Number, String, StringBuffer, Collection, Map, Date, Duration
a – b	        Minus	            a.minus(b)	                Number, String, List, Set, Date, Duration
a * b	        Star	            a.multiply(b)	            Number, String, Collection
a / b	        Divide	            a.div(b)	                Number
a % b	        Modulo	            a.mod(b)	                Integral number
a++ 	        Postincrement       def v = a; a = a.next();    Iterator, Number, String, Date, 
++a             Preincrement	    v a = a.next(); a	        Range
a-- --a	        Postdecrement       def v = a; a = a.previous();
                Predecrement	    v a = a.previous(); a	    Iterator, Number, String, Date, Range
-a	            Unary minus	        a.unaryMinus()	            Number, ArrayList
+a	            Unary plus	        a.unaryPlus()	            Number, ArrayList
a ** b	        Power	            a.power(b)	                Number
a | b	        Numerical or	    a.or(b)	                    Number, Boolean, BitSet, Process
a & b	        Numerical and	    a.and(b)	                Number, Boolean, BitSet
a ^ b	        Numerical xor	    a.xor(b)	                Number, Boolean, BitSet
~a	            Bitwise complement	a.bitwiseNegate()	        Number, String (the latter returning a regular expression pattern)
a[b]	        Subscript	        a.getAt(b)	                Object, List, Map, CharSequence, Matcher, many more
a[b] = c	    Subscript assignment	a.putAt(b, c)	        Object, List, Map, StringBuffer, many more
a << b	        Left shift	        a.leftShift(b)	            Integral number, also used like “append” to StringBuffer, Writer, File, Socket, List
a >> b	        Right shift	        a.rightShift(b)	            Number
a >>> b	        Right shift         unsigned	                a.rightShiftUnsigned(b)	Number

switch(a)
{
case b:         Classification      b.isCase(a)	                Object, Class, Range, Collection, Pattern, Closure; also used with Collection c in c.grep(b), which returns all items of c where b.isCase (item)
}		
a in b	        Classification	    b.isCase(a)	See previous row

a == b	        Equals	            if (a implements Comparable) Object; consider hashCode()[a]
                                    { a.compareTo(b) == 0 } 
                                    else { a.equals(b) }
{Aaron notes: Above is an important design.} Any type a When overriding the equals method, Java strongly encourages the developer 
to also override the hashCode() method such that equal objects have the same hash code (whereas objects with the same 
hash code are not necessarily equal). See the Java API documentation of java.lang.Object#equals.    	

a != b	        Not equal	        !(a == b)	Object
a <=> b	        Spaceship	        a.compareTo(b)	java.lang.Comparable
a > b	        Greater than	    a.compareTo(b) > 0	 
a >= b	        Greater than or equal to	a.compareTo(b) >= 0	 
a < b	        Less than	        a.compareTo(b) < 0	 
a <= b	        Less than or equal to	a.compareTo(b) <= 0	 
a as type	    Enforced coercion	a.asType (typeClass)	
```
The case of equals
Nothing is easier than determining whether a == b is true, right? Well, only at first sight if you want this to be a 
useful equality check. If both are null, they should count as equal. If they reference the same object, they’re equal 
without the need for checking. In other words, a == a for all values of a.
{Aaron notes: Above is an important design.}
But there’s more! If a >= b and a <= b, then you can deduce that a == b, right? But this may impose a conflict if you 
have a Comparable object that doesn’t implement the equals method consistently. This is why Groovy only looks at the 
compareTo method for Comparable objects when doing the equality check and ignores the equals method in this case. You 
can find the full logic implemented in the Groovy runtime under DefaultTypeTransformation.compareEqual(a,b).
{Aaron notes: Above is an important design.}

You can easily use any of these operators with your own classes. Just implement the respective method. Unlike in Java, 
there’s no need to implement a specific interface.

Strictly speaking, Groovy has even more operators in addition to those in table 3.4, such as the dot operator for 
referencing fields and methods. Their behavior can also be overridden. They come into play in chapter 7.

This is all good in theory, but let’s see how it all works in practice.

### 3.3.2. Overridden operators in action
Listing 3.3 demonstrates an implementation of the == (equals) and + (plus) operators for a Money class. It’s an 
implementation of the Value Object[9] pattern. You allow values of the same currency to be summed, but don’t support 
multicurrency addition.

9 See a discussion of value objects at http://c2.com/cgi/wiki?ValueObject.

You implement equals indirectly by using the @Immutable annotation as introduced in section 2.3.4. Remember 
that == (or equals method) denotes object equality (equal values), not identity (same object instances).

Listing 3.3. Overriding the plus and equals operators
{Aaron notes: Above is an important design.}

Because every immutable object automatically gets a value-based implementation of equals, you get away with only a 
minimal declaration at . The use of this operator is shown at , where one dollar becomes equal to any other dollar. 
At , the + operator isn’t overridden in the strict sense of the word, because there’s no such operator in Money’s 
superclass (Object). In this case, operator implementing is the best wording. This is used at , where two Money 
objects are added.

We mentioned earlier in this section that Java programmers may already be familiar with method overloading. You can 
apply that concept even with operators by defining additional plus implementations. Let’s look at a possible overload 
for the + operator. In listing 3.3, Money can only be added via the plus method to other Money objects. However, you 
might also want to be able to add Money with code like this:

``` assert buck + 1 == new Money(2, 'USD') ```
We can provide the additional method as follows:
```
Money plus (Integer more) {
    return new Money(amount + more, currency)
}
```
which overloads the plus method with a second implementation that takes an Integer parameter. The Groovy method dispatch 
finds the right implementation at runtime.

Note
Our plus operation on the Money class returns Money objects in both cases. We describe this by saying that Money’s plus 
operation is closed under its type. Whatever operation you perform on an instance of Money, you end up with another 
instance of Money.
{Aaron notes: Above is an important design.}

This example leads to the general issue of how to deal with different parameter types when implementing an operator 
method. We’ll go through aspects of this issue in the next section.

### 3.3.3. Making coercion work for you
Implementing operators is straightforward when both operands are of the same type. Things get more complex with a 
mixture of types, say
```
1 + 1.0
```
This adds an Integer and a BigDecimal. What is the return type? Section 3.6 answers this question for the special case 
of numbers, but the issue is more general. One of the two arguments needs to be promoted to the more general type. 
This is called coercion.
{Aaron notes: Above is an important design.}

When implementing operators, there are three main issues to consider as part of coercion.

1. Supported argument types
You need to decide which argument types and values will be allowed. If an operator must take a potentially inappropriate 
type, throw an IllegalArgumentException where necessary. In the Money example, even though it makes sense to use Money 
as the parameter for the plus operator, you don’t allow different currencies to be added together.

2. Promoting more specific arguments
If the argument type is a more specific one than your own type, promote it to your type and return an object of your 
type. To see what this means, consider how you might implement the + operator if you were designing the BigDecimal 
class, and what you’d do for an Integer argument.

Integer is more specific than BigDecimal: every Integer value can be expressed as a BigDecimal, but the reverse isn’t 
true. So for the BigDecimal.plus(Integer) operator, you’d consider promoting the Integer to BigDecimal, performing the 
addition, and then returning another BigDecimal—even if the result could accurately be expressed as an Integer.

3. Handling more general arguments with double dispatch
If the argument type is more general, call its operator method with yourself (“this,” the current object) as an 
argument. Let it promote you. This is also called double dispatch,[10] and it helps to avoid duplicated, asymmetric, 
possibly inconsistent code. Let’s reverse the previous example and consider Integer.plus (BigDecimal operand).

*10 Double dispatch is usually used with overloaded methods: a.method(b) calls b.method(a) where method is overloaded 
*with method(TypeA) and method(TypeB).

We’d consider returning the result of the expression operand.plus(this), delegating the work to BigDecimal’s plus(Integer) method. The result would be a BigDecimal, which is reasonable—it’d be odd for 1+1.5 to return an Integer but 1.5+1 to return a BigDecimal.

Of course, this is only applicable for commutative[11] operators. Test rigorously, and don’t make the mistake of creating an endless cycle. If Integer’s plus is calling BigInteger’s plus, you better make sure that BigInteger’s plus doesn’t call back to Integer!

*11 An operator is commutative if the operands can be exchanged without changing the result of the operation. For 
*example, plus is usually required to be commutative (a + b == b + a) but minus is not (a – b != b - a).

4. Groovy’s conventional behavior
Groovy’s general strategy of coercion is to return the most general type. Other languages such as Ruby try to be smarter and return the least general type that can be used without losing information from range or precision. The Ruby way saves memory at the expense of processing time. It also requires that the language promotes a type to a more general one when the operation would generate an overflow of that type’s range. Otherwise, intermediary results in a complex calculation could truncate the result.

Now that you know how Groovy handles types in general, we can delve deeper into what it provides for each of the datatypes it supports at the language level. We begin with the type that’s probably used more than any other non-numeric type: the humble string.

## 3.4. WORKING WITH STRINGS
Considering how widely strings are used, many languages—including Java—provide few language features to make them easier to handle. Scripting languages tend to fare better in this regard than mainstream application languages, so Groovy takes on board some of those extra features. This section examines what’s available in Groovy and how to make the most of the extra abilities.

Groovy strings come in two flavors: plain strings and GStrings. Plain strings are instances of java.lang.String, and GStrings are instances of groovy.lang.GString. GStrings allow placeholder expressions to be resolved and evaluated at runtime. Many scripting languages have a similar feature, usually called string interpolation, but it’s more primitive than the GString feature of Groovy. Let’s start by looking at each flavor of string and how it appears in code.

### 3.4.1. Varieties of string literals
Java allows only one way of specifying string literals: placing text in quotes “like this.” If you want to embed dynamic values within the string, you have to either call a formatting method (made easier, but still far from simple, in Java 1.5) or concatenate each constituent part. If you specify a string with a lot of backslashes in it (such as a Windows filename or a regular expression), your code becomes hard to read, because you have to double the backslashes. If you want a lot of text spanning several lines in the source code, you have to make each line contain a complete string (or several complete strings).

Groovy recognizes that not every use of string literals is the same, so it offers a variety of options. These are summarized in table 3.5.

Table 3.5. Summary of the string literal styles available in Groovy
Start/end characters

Example

Placeholder resolved?

Backslash escapes?

Single quote	'hello Dierk'	No	Yes
Double quote	"hello $name"	Yes	Yes
Triple single quote (’ ’ ’)	' ' '========== Total: $0.02 ==========' ' '	No	Yes
Triple double quote (""")	"""first $line second $line third $line"""	Yes	Yes
Forward slash	/x(\d*)y/	Yes	Occasionally
Dollar slash	$/x(\d*)y/$	Yes	Occasionally
The aim of each form is to specify the text data you want with a minimum of fuss. Each of the forms has a single feature that distinguishes it from the others:

The single-quoted form never pays any attention to placeholders. This is closely equivalent to Java string literals.
The double-quoted form is the equivalent of the single-quoted form, except that if the text contains unescaped dollar signs, the dollar sign introduces a placeholder, and the string will be treated as a GString instead of a plain string. GStrings are covered in more detail in the next section.
The triple-quoted form (or multiline string literal) allows the literal to span several lines. New lines are always treated as \n regardless of the platform, but all other whitespace is preserved as it appears in the text file. Multiline string literals may also be GStrings, depending on whether single quotes or double quotes are used. Multiline string literals act similar to Ruby or Perl.
The slashy form of string literal is also multiline but allows strings with backslashes to be specified simply without having to escape all the backslashes. This is particularly useful with regular expressions, as you’ll see later. There are only a few exceptions and limitations. Slashes are escaped with a backslash. A backslash can’t appear as the last character of a slashy string. Dollar symbols that could introduce a placeholder but aren’t meant to also need to be escaped. If you want to create a string with a backslash followed by a u, the backslash needs to be escaped so as not to be interpreted as a Unicode character, which happens in the earliest stages of parsing.[12]
12 Escaping backslashes and dollars is slightly tricky in a slashy string and involves either using GString tricks or embedding Unicode escape sequences (for example, \u005C is the Unicode for a backslash). Here are three expressions involving slashy strings resulting in a string starting with a backslash followed by u2: /\u005Cu${1+1}/ or GString.EMPTY + '\\' + /u${1+1}/ or /${'\\'}u${1+1}/. A similar issue occurs if you want to use a dollar sign. This is a small (and rare) price to pay for the benefits available, however.

The dollar slashy form of string literal also allows strings with backslashes to be specified without having to escape all the backslashes. Only Unicode characters are escaped with a backslash. Dollar signs and slashes are escaped with a dollar sign. The other restrictions on backslashes you saw for normal slashy strings don’t apply.
As we hinted earlier, Groovy uses a similar mechanism to Java for specifying special characters, such as linefeeds and tabs. In addition to the Java escapes, dollar signs can be escaped in double-quoted GStrings to allow them to be placed directly in such strings without the compiler assuming you’re defining a GString placeholder. The full set of escaped characters is specified in table 3.6.

Table 3.6. Escaped characters as known to Groovy
Escaped special character

Meaning

\b	Backspace
\t	Tab
\r	Carriage return
\n	Linefeed
\f	Form feed
\\	Backslash
\$	Dollar sign
\uabcd	Unicode character u + abcd (where a, b, c, and d are hex digits)
\abc	Unicode character u + abc (where a, b, and c are octal digits, and b and c are optional)
\'	Single quote
\"	Double quote
Note that in a double-quoted string, single quotes don’t need to be escaped, and vice versa. In other words, 'I said, "Hi."' and "don't" both do what you hope they will. For the sake of consistency, both still can be escaped in each case. Likewise, dollar signs can be escaped in single-quoted strings, even though they don’t need to be. This makes it easier to switch between the forms.

Note that Java uses single quotes for character literals, but as you’ve seen, Groovy cannot do so because single quotes are already used to specify strings.

But you can achieve the same as in Java when providing the type explicitly:

char a = 'x'
or

Character b = 'x'
The java.lang.String 'x' is cast into a java.lang.Character. If you want to coerce a string into a character at other times, you can do so in either of the following ways:

'x' as char
or

'x'.toCharacter()
As a GDK goody, there are more to* methods to convert a string, such as toInteger, toLong, toFloat, and toDouble.

Whichever literal form is used, unless the compiler decides it’s a GString, it ends up as an instance of java.lang.String, just like Java string literals. So far, we’ve only teased you with allusions to what GStrings are capable of. Now it’s time to spill the beans.

### 3.4.2. Working with GStrings
GStrings are like strings with additional capabilities.[13] They’re literally declared in double quotes. What makes a double-quoted string literally a GString is the appearance of placeholders. Placeholders may appear in a full ${expression} syntax or an abbreviated $reference syntax. See the examples in the following listing.

13 groovy.lang.GString isn’t actually a subclass of java.lang.String, and couldn’t be, because String is final. But GStrings can usually be used as if they were strings—Groovy coerces them into strings when it needs to.

Listing 3.4. Working with GStrings


Within a GString, simple references to variables can be dereferenced with the dollar sign. This simplest form is shown at  , whereas  shows this being extended to use property accessors with the dot syntax. You’ll learn more about accessing properties in chapter 7.

The full syntax uses dollar signs and braces, as shown at . It allows arbitrary Groovy expressions within the braces. The braces denote a closure.

In real life, GStrings are handy in templating scenarios. A GString is used to create the string for an SQL query. Groovy provides even more sophisticated templating support, as shown in chapter 8. If you need a dollar character within a template (or any other GString use), you must escape it with a backslash as shown in .

Although GStrings behave like java.lang.String objects for all operations that a programmer is usually concerned with, they’re implemented differently to capture the fixed and dynamic parts (the so-called values) separately. This is revealed by the following code:

def me      = 'Tarzan'
def you     = 'Jane'
def line    = "me $me - you $you"
assert line == 'me Tarzan - you Jane'
assert line instanceof GString
assert line.strings[0] == 'me '
assert line.strings[1] == ' - you '
assert line.values[0]  == 'Tarzan'
assert line.values[1]  == 'Jane'
Placeholder evaluation time
Each placeholder inside a GString is evaluated at declaration time and the result is stored in the GString object. By the time the GString value is converted into a java.lang.String (by calling its toString method or casting it to a string), each value gets written[14] to the string. Because the logic of how to write a value can be elaborate for certain types (most notably closures), this behavior can be used in advanced ways that make the evaluation of such placeholders appear to be lazy. See section D.2 of the cheat sheet appendix for an example of this.

14 See Writer.write(Object) in section 12.2.3.

You’ve seen the Groovy language support for declaring strings. What follows is an introduction to the use of strings in the Groovy library. This will also give you a first impression of the seamless interplay of Java and Groovy. We start in typical Java style and gradually slip into Groovy mode, carefully watching each step.

### 3.4.3. From Java to Groovy
Now that you have your strings easily declared, you can have some fun with them. Because they’re objects of type java.lang.String, you can call String’s methods on them or pass them as parameters wherever a string is expected, such as for easy console output:

System.out.print("Hello Groovy!");
This line is equally valid Java and Groovy. You can also pass a literal Groovy string in single quotes:

System.out.print('Hello Groovy!');
Because this is such a common task, the GDK provides a shortened syntax:

print('Hello Groovy!');
You can drop parentheses and semicolons, because they’re optional and don’t help readability in this case. The resulting Groovy style boils down to

print 'Hello Groovy!'
Looking at this last line only, you cannot tell whether this is Groovy, Ruby, Perl, or one of several other line-oriented scripting languages. It may not look sophisticated, but it boils down the code to its essence.

The next listing presents more of the mix-and-match between core Java and additional GDK capabilities. How would you judge the signal-to-noise ratio of each line?

Listing 3.5. A miscellany of string operations
String greeting = 'Hello Groovy!'

assert greeting.startsWith('Hello')

assert greeting.getAt(0) == 'H'
assert greeting[0] == 'H'

assert greeting.indexOf('Groovy') >= 0
assert greeting.contains('Groovy')

assert greeting[6..11] == 'Groovy'
assert 'Hi' + greeting - 'Hello' == 'Hi Groovy!'

assert greeting.count('o') == 3

assert 'x'.padLeft(3)      == '  x'
assert 'x'.padRight(3,'_') == 'x__'
assert 'x'.center(3)       == ' x '
assert 'x' * 3             == 'xxx'
These self-explanatory examples give an impression of what’s possible with strings in Groovy. If you’ve ever worked with other scripting languages, you may notice that a useful piece of functionality is missing from listing 3.5: changing a string in place. Groovy cannot do so because it works on instances of java.lang.String and obeys Java’s invariant of strings being immutable.

Before you say “What a lame excuse!” here’s Groovy’s answer to changing strings: although you cannot work on String, you can still work on StringBuffer![15] On a StringBuffer, you can work with the << (left shift) operator for appending and the subscript operator for in-place assignments. Using the << operator on String returns a StringBuffer. Here’s the StringBuffer equivalent to listing 3.5:

15 Future versions may use a StringBuilder instead. StringBuilder was introduced in Java 1.5 to reduce the synchronization overhead of StringBuffers. Typically, StringBuffers are used only in a single thread and then discarded—but StringBuffer itself is thread-safe, at the expense of synchronizing each method call.



Note
Although the expression stringRef << string returns a StringBuffer, note that StringBuffer isn’t automatically assigned to the stringRef . When used on a String, it needs explicit assignment; on StringBuffer it doesn’t. With a StringBuffer, the data in the existing object is changed —with a String you can’t change the existing data, so you have to return a new object instead. You might also note that a greeting was explicitly typed. It’s effectively of type Object and can reference both String and StringBuffer values.

Throughout the next sections, you’ll gradually add to what you’ve learned about strings as you discover more language features. String has gained several new methods in the GDK. You’ve already seen a few of these, but you’ll see more as we talk about working with regular expressions and lists. The complete list of GDK methods on strings is listed in appendix C.

Working with strings is one of the most common tasks in programming, and for script programming in particular: reading text, writing text, cutting words, replacing phrases, analyzing content, search and replace—the list is amazingly long. Think about your own programming work. How much of it deals with strings?

Groovy supports you in these tasks with comprehensive string support, but that’s not the whole story. The next section introduces regular expressions, which cut through text like a chainsaw: difficult to operate but extremely powerful.

## 3.5. WORKING WITH REGULAR EXPRESSIONS
Once a programmer had a problem. He thought he could solve it with a regular expression. Now he had two problems.

Jamie Zawinski

Suppose you had to prepare a table of contents for this book. You’d need to collect all the headings like “3.5 Working with regular expressions”—paragraphs that start with a number or with a number, a dot, and another number. The rest of the paragraph would be the heading. This would be cumbersome to code naïvely: iterate over each character; check whether it’s a line start; if so, check whether it’s a digit; if so, check whether a dot and a digit follow. Puh—lots of rope, and we haven’t even covered numbers that have more than one digit.

Regular expressions come to the rescue. They allow you to declare such a pattern rather than programming it. Once you have the pattern, Groovy lets you work with it in numerous ways.

Regular expressions are prominent in scripting languages and have also been available in the Java library since JDK 1.4. Groovy relies on Java’s regex (regular expression) support and adds three operators for convenience:

The regex find operator, =~
The regex match operator, ==~
The regex pattern operator, ~string
An in-depth discussion about regular expressions is beyond the scope of this book. Our focus is on Groovy, not on regexes. We give the shortest possible introduction to make the examples comprehensible and provide you with a jumpstart. For additional information, see http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html.

Regular expressions are defined by patterns. A pattern can be anything from a simple character, fixed string, or something like a date format made up of digits and delimiters, up to descriptions of balanced parentheses in programming languages. Patterns are declared by a sequence of symbols. In fact, the pattern description is a language of its own. Some examples are shown in table 3.7. Note that these are the raw patterns, not how they’d appear in string literals. In other words, if you stored the pattern in a variable and printed it out, this is what you’d want to see. It’s important to make the distinction between the pattern itself and how it’s represented in code as a literal.

Table 3.7. Simple regular expression pattern examples
Pattern

Meaning

some text	Exactly “some text.”
some\s+text	The word “some” followed by one or more whitespace characters followed by the word “text.”
^\d+(\.\d+)? (.*)	Our introductory example: headings of level one or two denoted by a line ^ start, one or more digits, and an optional dot followed by more digits. Parentheses are used for grouping. The question mark makes the first group optional. The second group contains the title, made of a dot for any character and a star for any number of such characters.
\d\d/\d\d/\d\d\d\d	A date formatted as exactly two digits followed by a slash, two more digits followed by a slash, followed by exactly four digits.
A pattern like one of the examples in table 3.7 allows you to declare what you’re looking for, rather than having to program how to find something.

Next you’ll see how patterns appear as literals in code and what can be done with them. We’ll then revisit the initial example with a full solution, before examining some performance aspects of regular expressions and finally showing how they can be used for classification in switch statements and for collection filtering with the grep method.

### 3.5.1. Specifying patterns in string literals
How do you put the sequence of symbols that declares a pattern inside a string? In Java, this causes confusion. Patterns use lots of backslashes, and to get a backslash in a Java string literal, you need to double it. This leads to Java strings, which are very hard to read in terms of the raw pattern involved. It gets even worse if you need to match an actual backslash in your pattern—the pattern language escapes that with a backslash too, so the Java regex string literal needed to match a\b is "a\\\\b".

Groovy does much better. As you saw earlier, there’s the slashy form of string literal, which doesn’t require you to escape the backslash character and still works like a normal GString. The following listing shows how to declare patterns conveniently.

Listing 3.6. Regular expression GStrings
assert "abc" == /abc/
assert "\\d" == /\d/

def    reference = "hello"
assert reference == /$reference/
Note that you have the choice to declare patterns in either kind of string: literal string with single quotes, GString with double quotes, or slashy strings.

Tip
Sometimes the slashy syntax interferes with other valid Groovy expressions such as line comments or numerical expressions with multiple slashes for division. When in doubt, put parentheses around your pattern like (/pattern/). Parentheses force the parser to interpret the content as an expression.

Symbols
The key to using regular expressions is knowing the pattern symbols. For convenience, table 3.8 provides a short list of the most common ones. Put an earmark on this page so you can easily look up the table—you’ll use it a lot.

Table 3.8. Regular expression symbols (excerpt)
Symbol

Meaning

.	Any character
^	Start of line (or start of document, when in single-line mode)
$	End of line (or end of document, when in single-line mode)
\d	Digit character
\D	Any character except digits
\s	Whitespace character
\S	Any character except whitespace
\w	Word character
\W	Any character except word characters
\b	Word boundary
()	Grouping
( x | y )	x or y, as in (Groovy|Java|Ruby)
\1	Backmatch to group one; for example, find doubled characters with (.)\1
x *	Zero or more occurrences of x
x +	One or more occurrences of x
x ?	Zero or one occurrence of x
x { m , n }	At least m and at most n occurrences of x
x { m }	Exactly m occurrences of x
[a-f]	Character class containing the characters a, b, c, d, e, f
[^a]	Character class containing any character except a
(?is:x)	Switches mode when evaluating x; i turns on ignoreCase, s means single-line mode
Tip
Symbols tend to have the same first letter as what they represent; for example, digit, space, word, and boundary. Uppercase symbols define the complement; think of them as a warning sign for no.

More to consider:

Use grouping properly. The expanding operators such as * and + bind closely; ab+ matches abbbb. Use (ab)+ to match ababab.
In normal mode, the expanding operators are greedy, meaning they try to match the longest substring that matches the pattern. Add an additional question mark after the operator to put them into restrictive mode. You may be tempted to extract the href from an HTML anchor element with this regex: href="(.*)". But href= "(.*?)" is probably better. The first version matches until the last double quote in your text; the latter matches until the next double quote.[16]
16 This is only to explain the greedy behavior of regular expression, not to explain how HTML is parsed correctly, which would involve a lot of other topics such as ordering of attributes, spelling variants, and so forth.

This is only a brief description of the regex pattern format, but a complete specification comes with your JDK, as part of the Javadoc for java.util.regex.Pattern. It may change marginally between JDK versions; for JDK 7, it can be found online at http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html.

See the Javadoc to learn more about different evaluation modes, positive and negative look-ahead, back references, and posix characters.

It always helps to test your expressions before putting them into code. There are many online applications that allow interactive testing of regular expressions. You should be aware that not all regular expression pattern languages are exactly the same. You may get unexpected results if you take a regular expression designed for use in .NET and apply it in a Java or Groovy program. Although there aren’t many differences, the differences that do exist can be hard to spot. Even if you take a regular expression from a book or a website, you should still test that it works in your code.

Once you’ve declared the pattern you want, you need to tell Groovy how to apply it. We’ll explore a whole variety of uses.

### 3.5.2. Applying patterns
For a given string and pattern, Groovy supports the following tasks for regular expressions:

Tell whether the pattern fully matches the whole string.
Tell whether there’s an occurrence of the pattern in the string.
Count the occurrences.
Do something with each occurrence.
Replace all occurrences with some text.
Split the string into multiple strings by cutting at each occurrence.
Listing 3.7 puts patterns into action. Unlike most other examples, this listing contains some comments. This reflects real life and isn’t for illustrative purposes. The use of regexes is best accompanied by this kind of comment for all but the simplest patterns.

Listing 3.7. Regular expressions


 and  have an interesting twist. Although the regex find operator evaluates to a Matcher object, it can also be used as a Boolean conditional. We’ll explore how this is possible when examining the “Groovy Truth” in chapter 6.

Tip
To remember the difference between the =~ find operator and the ==~ match operator (it looks like a burning match), recall that match is more restrictive, because the pattern needs to cover the whole string. The demanded coverage is “longer” just like the operator itself.

See your Javadoc for more information about the java.util.regex.Matcher object, such as how to walk through all the matches and how to work with groupings within each match.

Common regex pitfalls
You don’t need to fall into the regex traps yourself. We’ve already done this for you and we (the authors) have learned the following:

When things get complex (note, this is when, not if), comment verbosely.
Use the slashy syntax instead of the regular string syntax, or you’ll get lost in a forest of backslashes.
Don’t let your pattern look like a toothpick puzzle. Build your pattern from subexpressions like WORD in listing 3.7.
Put your assumptions to the test. Write some assertions or unit tests to test your regex against static strings. Please don’t send us any more flowers for this advice; a tweet like “Assertions saved my life today! Thanks #ReGina.” will suffice.
### 3.5.3. Patterns in action
You’re now ready to do everything you wanted to do with regular expressions, except we haven’t covered “do something with each occurrence.” Something and each sounds like a cue for a closure to appear, and that’s the case here. String has a method called eachMatch that takes a regex as a parameter along with a closure that defines what to do on each match.

What is a match?
A match is the occurrence of a regular expression pattern in a string. It’s therefore a string: a substring of the original string. When the pattern contains groupings like in /begin(.*?)end/, you need to know more information: not just the string matching the whole pattern, but also what part of that string matched each group. Therefore, the match becomes a list of strings, containing the whole match at position 0 with group matches being available as match[n] where n is group number n. Groups are numbered by the sequence of their opening parentheses.

The match gets passed into the closure for further analysis. In the musical example in the next listing, each match is appended to a result string.

Listing 3.8. Working on each match of a pattern


There are two different ways to iterate through matches with identical behavior: use String.eachMatch(Pattern) , or use Matcher.each() , where the Matcher is the result of applying the regex find operator to a string and a pattern.  shows a special case for replacing each match with some dynamically derived content from the given closure. The variable it refers to the matching substring. The result is to replace “ain” with underscores, but only where it forms part of a rhyme.

To fully understand how the Groovy regular expression support works, we need to look at the java.util.regex.Matcher class. It’s a JDK class that encapsulates knowledge about:

How often and at what position a pattern matches.
The groupings for each match.
The GDK enhances the Matcher class with simplified array-like access to this information. In Groovy, you can think about a matcher as if it was a list of all its matches. This is what happens in the following example that matches all nonwhitespace characters:

def matcher = 'a b c' =~ /\S/

assert matcher[0] == 'a'
assert matcher[1..2] == ['b','c']
assert matcher.size() == 3
This use case comes with an interesting variant that uses Groovy’s parallel assignment feature that allows you to directly assign each match to its own reference.

def (a,b,c) = 'a b c' =~ /\S/

assert a == 'a'
assert b == 'b'
assert c == 'c'
It gets even more interesting with groupings in the match. If the pattern contains parentheses to define groups, then the result of asking for a particular match is an array of strings rather than a single one: the same behavior as we mentioned for eachMatch. Again, the first result (at index 0) is the match for the whole pattern. Consider this example, where each match finds pairs of strings that are separated by a colon. For later processing, the match is split into two groups, for the left and the right string:

def matcher = 'a:1 b:2 c:3' =~ /(\S+):(\S+)/

assert matcher.hasGroup()
assert matcher[0] == ['a:1', 'a', '1'] // 1st match
assert matcher[1][2] == '2' // 2nd match, 2nd group
In other words, what matcher[0] returns depends on whether the pattern contains groupings.

This also applies to the matcher’s each method, which comes with a convenient notation for groupings. When the processing closure defines multiple parameters, the list of groups is distributed over them:

def matcher = 'a:1 b:2 c:3' =~ /(\S+):(\S+)/
matcher.each { full, key, value ->
    assert full.size()  == 3
    assert key.size()   == 1 // a,b,c
    assert value.size() == 1 // 1,2,3
}
This matcher matches three times, passing the full match and the two groups into the closure on each match. The preceding code snippet enables you to assign meaningful names to the group matches. We decided to call them key and value, which much better reveals their intent than match[1] and match[2]would.

Our advice is to use group names whenever the group count is fixed. Groovy supports the spreading of match groups over closure parameters for all methods that pass a match into a closure. For example, you can use it with the String.eachMatch(regex){match->} method.

Implementation Detail
Groovy internally stores the most recently used matcher (per thread). It can be retrieved with the static property Matcher.lastMatcher. You can also set the index property of a matcher to make it look at the respective match with matcher.index = x. Both can be useful in some exotic corner cases. See Matcher’s API documentation for details.

We’ll revisit the Matcher class later in numerous places. It’s particularly interesting because it plays so well with Groovy’s approach of letting classes decide how to iterate over themselves and reusing that behavior pervasively.

Matcher and Pattern work in combination and are the key abstractions for regexes in Java and Groovy. You’ve seen Matcher, and we’ll have a closer look at the Pattern abstraction next.

### 3.5.4. Patterns and performance
Finally, let’s look at performance and the pattern operator ~string (note this is a tilde, not a minus sign). The pattern operator transforms a string into an object of type java.util.regex.Pattern. For a given string, this pattern object can be asked for a matcher object.

The rationale behind this construction is that patterns are internally backed by a finite-state machine that does all the high-performance magic. This machine is compiled when the pattern object is created. The more complicated the pattern, the longer the creation takes. In contrast, the matching process as performed by the machine is extremely fast.

The pattern operator allows you to split pattern-creation time from pattern-matching time, increasing performance by reusing the finite-state machine. The following listing shows a poor-man’s performance comparison of the two approaches. The precompiled pattern version is at least twice as fast (although these kinds of measurements can differ wildly).

Listing 3.9. Increasing performance with pattern reuse


To find words that start and end with the same character, the \1 backmatch is used to refer to that character. Its use is prepared by putting the word’s first character into a group, which happens to be group 1.

Note the difference in spelling in . This isn’t a =~ b but a= ~b. Tricky.

Use whitespace wisely
The observant reader may spot a language issue: What happens if you write a=~b without any whitespace? Is that the =~ find operator, or is it an assignment of the ~b pattern to a? For the human reader, it’s ambiguous. Not so for the Groovy parser. It’s greedy and will parse this as the find operator.

It goes without saying that being explicit with whitespace is good programming style, even when the meaning is unambiguous for the parser. Do it for the next human reader, which will probably be you.

Don’t forget that performance should usually come second to readability—at least to start with. If reusing a pattern means bending your code out of shape, you should ask yourself how critical the performance of that particular area is before making the change. Measure the performance in different situations with each version of the code, and balance ease of maintenance with speed and memory requirements.

### 3.5.5. Patterns for classification
Listing 3.10 completes your journey through the domain of patterns. The Pattern object, as returned from the pattern operator, implements an isCase(String) method that’s equivalent to a full match of that pattern with the string. This classification method is a prerequisite for using patterns conveniently with the in operator, the grep method, and in switch cases.

The example classifies words that consist of exactly four characters. The pattern, therefore, consists of the word character class \w followed by the {4} quantification.

Listing 3.10. Patterns for classification
def fourLetters = ~/\w{4}/

assert fourLetters.isCase('work')

assert 'love' in fourLetters

switch('beer'){
    case fourLetters: assert true; break
    default         : assert false
}

beasts = ['bear','wolf','tiger','regex']

assert beasts.grep(fourLetters) == ['bear','wolf']
Tip
Classifications read nicely with in, switch, and grep. It’s rare to call classifier.isCase(candidate) directly, but when you see such a call, it’s easiest to read it from right to left: “candidate is a case of classifier.”

Patterns are also prevalent in the Groovy library (see the GDK reference in appendix C). These methods give you the convenient choice between using either a string that describes the regular expression (conventionally this parameter is called regex), or supplying a pattern object instead (conventionally called pattern).

At times, regular expressions can be difficult beasts to tame, but mastering them adds a new quality to all text-manipulation tasks. Once you have a grip on them, you’ll hardly be able to imagine having programmed (some would say lived) without them. Writing this book without their help would have been very hard indeed. Groovy makes regular expressions easily accessible and straightforward to use.

This concludes our coverage of text-based types, but of course computers have always dealt with numbers as well as text. Working with numbers is easy in most programming languages, but that doesn’t mean there’s no room for improvement. Let’s see how Groovy goes the extra mile when it comes to numeric types.

## 3.6. WORKING WITH NUMBERS
We introduced the available numeric types and their declarations in section 3.1 and you’ve already seen that for decimal numbers, the default type is java.math.BigDecimal. This is a feature to get around the most common misconceptions about floating-point arithmetic. We’re going to look at which type is used where and what extra abilities have been provided for numbers in the GDK.

### 3.6.1. Coercion with numeric operators
It’s always important to understand what happens when you use one of the numeric operators.

Most of the rules for the addition, multiplication, and subtraction operators are the same as in Java, but there are some changes regarding floating-point behavior, and BigInteger and BigDecimal also need to be included. The rules are straightforward. The first rule to match the situation is used.

For the operations +, -, and *:

If either operand is a Float or a Double, the result is a Double. (In Java, when only Float operands are involved, the result is a Float too.)
Otherwise, if either operand is a BigDecimal, the result is a BigDecimal.
Otherwise, if either operand is a BigInteger, the result is a BigInteger.
Otherwise, if either operand is a Long, the result is a Long.
Otherwise, the result is an Integer.
Table 3.9 depicts the scheme for quick lookup. Types are abbreviated by uppercase letters.

Table 3.9. Numerical coercion
+ - *

B

S

I

C

L

BI

BD

F

D

Byte	I	I	I	I	L	BI	BD	D	D
Short	I	I	I	I	L	BI	BD	D	D
Integer	I	I	I	I	L	BI	BD	D	D
Character	I	I	I	I	L	BI	BD	D	D
Long	L	L	L	L	L	BI	BD	D	D
BigInteger	BI	BI	BI	BI	BI	BI	BD	D	D
BigDecimal	BD	BD	BD	BD	BD	BD	BD	D	D
Float	D	D	D	D	D	D	D	D	D
Double	D	D	D	D	D	D	D	D	D
Other aspects of coercion behavior include:

Like Java but unlike Ruby, no coercion takes place when the result of an operation exceeds the current range, except for the power operator.
For division, if any of the arguments is of type Float or Double, the result is of type Double; otherwise the result is of type BigDecimal with the maximum precision of both arguments, rounded half up. The result is normalized—that is, without trailing zeros.
Integer division (keeping the result as an integer) is achievable through explicit casting or by using the intdiv() method.
The shifting operators are implemented with bit-shifting semantics only for types Integer and Long but you can implement them for other types, too, through operator overriding. They don’t coerce to other types.
The power operator coerces to the next best type that can take the result in terms of range and precision, in the sequence Integer, Long, Double.
The equals operator coerces to the more general type before comparing.
Rules can be daunting without examples, so this behavior is demonstrated in table 3.10.

Table 3.10. Numerical expression examples
Expression

Result Type

Comments

1f*2f	Double	In Java, this would be Float.
(Byte)1+(Byte)2	Integer	As in Java, integer arithmetic is always performed in at least 32 bits.
1*2L	Long	 
1/2	BigDecimal (0.5)	In Java, the result would be the integer 0.
(int)(1/2)	Integer (0)	This is normal coercion of .BigDecimal to Integer.
1.intdiv(2)	Integer (0)	This is the equivalent of the Java 1/2.
Integer.MAX_VALUE+1	Integer	Non-power operators wrap without promoting the result type.
2**30	Integer	 
2**31	BigInteger	The power operator promotes where necessary.
2**3.5	Double	 
2G+1G	BigInteger	 
2.5G+1G	BigDecimal	 
1.5G==1.5F	Boolean (true)	Float is promoted to BigDecimal before comparison.
1.1G==1.1F	Boolean (false)	1.1 can’t be exactly represented as a Float (or indeed a Double), so when it’s promoted to BigDecimal, it isn’t equal to the exact BigDecimal 1.1G but rather 1.100000023841858G.
The only surprise is that there’s no surprise. In Java, results like in the fourth row are often surprising—for example, (1/2) is always 0 because when both operands of division are integers, only integer division is performed. To get 0.5 in Java, you need to write (1f/2).

This behavior is especially important when using Groovy to enhance your application with user-defined input. Suppose you allow superusers of your application to specify a formula that calculates an employee’s bonus, and a business analyst specifies it as businessDone * (1/3). With Java semantics, this will be a bad year for the poor employees.

### 3.6.2. GDK methods for numbers
The GDK defines all applicable methods from table 3.4 to implement overridable operators for numbers such as plus, minus, power, and so forth. They all work without surprises. In addition, the following methods fulfill their self-describing duty:

assert 1 == (-1).abs()
assert 2 == 2.5.toInteger()      // conversion
assert 2 == 2.5 as Integer       // enforced coercion
assert 2 == (int) 2.5            // cast
assert 3 == 2.5f.round()
assert 3.142 == Math.PI.round(3)
assert 4 == 4.5f.trunc()
assert 2.718 == Math.E.trunc(3)

assert '2.718'.isNumber()        // String methods
assert 5 == '5'.toInteger()
assert 5 == '5' as Integer
assert 53 == (int) '5'           // gotcha!
assert '6 times' == 6 + ' times' // Number + String
As you can see, there are various conversion possibilities: the toInteger() method (also available for Double, Float, and so on), enforced coercion with the as operator that calls the asType(class)method, and the humble cast.

Warning!
Don’t cast strings to numbers! In Groovy, you can cast a string of length 1 directly to a char. But char and int are essentially the same thing on the Java platform. This leads to the gotcha where '5' is cast to its Unicode value 53. Instead, use the type conversion methods.

More interestingly, the GDK also defines the methods times, upto, downto, and step. They all take a closure argument. Listing 3.11 shows these methods in action: times is just for repetition, upto is for walking a sequence of increasing numbers, downto is for decreasing numbers, and step is the general version that walks until the end value by successively adding a step width.

Listing 3.11. GDK methods on numbers


Calling methods on numbers can feel unfamiliar at first when you come from Java. Just remember that numbers are objects and you can treat them as such. As you’ve seen, numbers in Groovy work in a natural way and protect you against the most common errors with floating-point arithmetic. In most cases, there’s no need to remember all details of coercion. When the need arises, this section may serve as a reference.

The strategy of making objects available in unexpected places starts to become an ongoing theme. You’ve seen it with numbers, and section 4.1 will show the same principle applied to ranges.

## 3.7. SUMMARY
Contrary to popular belief, Groovy gives you the same type safety as Java, albeit at runtime instead of Java’s mix of compile time and runtime. This approach is a prerequisite to enable the awesome power of dynamic language features such as synthesized methods, flexible bindings for scripts, templates and closures, and all the other metaprogramming goodness that we’ll explore in the course of this book.

Making common activities more convenient is one of Groovy’s main promises. Consequently, Groovy promotes even the primitive data types to first-class objects and implements operators as method calls to make the benefits of object orientation ubiquitously available.

Developer convenience is further enhanced by allowing a variety of means for string literal declarations, whether through flexible GString declarations or with the slashy syntax for situations where extra escaping is undesirable, such as regular expression patterns. GStrings contribute to another of Groovy’s central pillars: concise and expressive code. This allows the reader a clearer insight into the runtime string value, without having to wade through reams of string concatenation or switch between format strings and the values replaced in them.

Regular expressions are well represented in Groovy, again confirming its comfortable place among other top-of-stack languages. Utilizing regular expressions is an everyday exercise, and a language that treated them as second-class citizens would be severely hampered. Groovy effortlessly combines Java’s libraries with language support, retaining the regular expression dialect familiar to Java programmers with the ease of use found in scripting.

The Groovy way of treating numbers with respect to type conversion and precision handling leads to intuitive use, even for nonprogrammers. This becomes particularly important when Groovy scripts are used for smart configurations of larger systems where business users may provide formulas—for example, to define share-valuation details.

Strings, regular expressions, and numbers alike profit from numerous methods that the GDK introduces on top of the JDK. A clear pattern has emerged already—Groovy is a language designed for the ease of those developing in it, concentrating on making repetitive tasks as simple as they can be without sacrificing the power of the Java platform.

You’ll soon see that this focus on ease of use extends far beyond the simple types that Java developers are used to having built-in language support for. The Groovy designers are well aware of other concepts that are rarely far from a programmer’s mind. The next chapter shows how intuitive operators, enhanced literals, and extra GDK methods are also available with Groovy’s collective datatypes: ranges, lists, and maps.