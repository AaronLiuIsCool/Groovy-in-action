# Chapter 7. Object orientation, Groovy style
This chapter covers

Defining classes and scripts
Object-oriented features: inheritance, interfaces, multimethods, and traits
Working with GroovyBeans
Advanced syntax features: GPath, spread operators, and command chains
Any intelligent fool can make things bigger, more complex, and more violent. It takes a touch of genius—and a lot of courage—to move in the opposite direction.

Albert Einstein

There’s a common misconception about scripting languages. Because a scripting language might support a less rigid approach to typing and provide some initially surprising syntax shorthands, it may be perceived as a nice new toy for hackers rather than a language suitable for serious OOP. This reputation stems from the time when scripting was done in terms of shell scripts or early versions of Perl, where the lack of encapsulation and other object-oriented features sometimes led to poor code management, frequent code duplication, and obscure hidden bugs. It wasn’t helped by languages that combined notations from several existing sources as part of their heritage.

Over time, the scripting landscape has changed dramatically. Perl has added support for object orientation, Python has extended its object-oriented support, and, more recently, even JavaScript can be generated from more strictly typed languages like TypeScript and PureScript.

Groovy extends the reach of Java by making it scriptable, but it also provides new language constructs to better reveal the intent of the developer. You’ve already seen that Groovy provides reference types in cases where Java uses nonobject primitive types, introduces ranges and closures as first-class objects, and has many shorthand notations for working with collections of objects. But these enhancements are just scratching the surface. Groovy allows you to not just write code but to design it and keep this design visible.

In this chapter, we’ll take you on a journey. We begin in familiar territory, with classes, objects, constructors, references, and so forth. Every so often, there’s something a bit different, a little tweak of Grooviness. By the end of the chapter, you’ll see code that reads so much like plain English that it could have been mistaken for a comment. Welcome to the Groovy world.

7.1. DEFINING CLASSES AND SCRIPTS
Class definition in Groovy is almost identical to Java; classes are declared using the class keyword and may contain fields, constructors, initializers, and methods.[1] Methods and constructors may themselves use local variables as part of their implementation code. Scripts are different—offering additional flexibility but with some restrictions too. They may contain code, variable definitions, and method definitions, as well as class definitions. We’ll describe how all of these members are declared and cover a previously unseen operator on the way.

1 Interfaces are also like their Java counterparts, but we’ll hold off discussing those further until section 7.3.2.

7.1.1. Defining fields and local variables
In its simplest terms, a variable is a name associated with a slot of memory that can hold a value. Just as in Java, Groovy has local variables, which are scoped within the method they’re part of, and fields, which are associated with classes or instances of those classes. Fields and local variables are declared in much the same way, so we cover them together.

Declaring variables
Fields and local variables must be declared before first use (except for a special case involving scripts, which we discuss later). This helps to enforce scoping rules and protects the programmer from accidental misspellings. The declaration always involves specifying a name, and may optionally include a type, modifiers, and assignment of an initial value. Once declared, variables are referenced by their name.

Scripts allow the use of undeclared variables, in which case these variables are assumed to come from the script’s binding and are added to the binding if they’re not yet there. The binding is a data store that enables transfer of variables to and from the caller of a script. Section 16.2.2 has more details about this mechanism.

Groovy uses Java’s modifiers—the keywords private, protected, and public for modifying visibility[2]; final for disallowing reassignment; and static to denote class variables. A nonstatic field is also known as an instance variable. These modifiers all have the same meaning as in Java.

2 Java’s default package-wide visibility is supported via the @PackageScope annotation.

The default visibility for fields has a special meaning in Groovy. When no visibility modifier is attached to a field declaration, a property is generated for the respective name. You’ll learn more about properties in section 7.4 when we present GroovyBeans.

Defining the type of a variable is optional. But the identifier must not stand alone in the declaration. When no type and modifier are given, the def keyword must be used as a replacement, effectively indicating that the field or variable can be assigned an object of any type at runtime.

The following listing depicts the general appearance of field and variable declarations with optional assignment and using a comma-separated list of identifiers to declare multiple references at once.

Listing 7.1. Variable declaration examples


Assignments to typed references must conform to the type. You saw in chapter 3 that Groovy provides autoboxing and coercion when it makes sense. All other cases are type-breaking assignments and lead to a ClassCastException at runtime, as can be seen in the following listing.[3]

3 The shouldFail method as used in this example checks that a ClassCastException occurs. More details can be found in section 17.3.

Listing 7.2. Variable declaration examples
final String PI = '3.14'
assert PI.class.name == 'java.lang.String'
assert PI.size() == 4
GroovyAssert.shouldFail(ClassCastException){
    Float areaOfCircleRadiusOne = PI
}
As previously discussed, variables can be referred to by name in the same way as in Java—but Groovy provides a few more interesting possibilities.

Referencing and dereferencing fields
In addition to referring to fields by name with the obj.fieldname[4] syntax, they can also be referenced with the subscript operator, as shown in the next listing. This allows you to access fields using a dynamically determined name.

4 This notation can also appear in the form of obj.@fieldname, as you’ll see in section 7.4.2.

Listing 7.3. Referencing fields with the subscript operator
class Counter {
    public count = 0
}

def counter = new Counter()

counter.count = 1
assert counter.count == 1

def fieldName = 'count'
counter[fieldName] = 2
assert counter['count'] == 2
Accessing fields in such a dynamic way is part of the bigger picture of dynamic execution that we’ll analyze in the course of this chapter.

If you worked through the Groovy datatype descriptions, your next question will probably be: Can I override the subscript operator? Sure you can, and you’ll extend but not override the general field-access mechanism that way. But you can do even better and extend the field-access operator!

Listing 7.4 shows how to do that. To extend both set and get access, provide the following methods

Object get (String name)
void   set (String name, Object value)
There’s no restriction on what you do inside these methods; get can return artificial values, effectively pretending that your class has the requested field. In listing 7.4, the same value is always returned, regardless of which field value is requested. The set method is used for counting the write attempts.

Listing 7.4. Extending the general field-access mechanism
class PretendFieldCounter {
    public count = 0

    Object get (String name) {
        return 'pretend value'
    }
    void set (String name, Object value) {
        count++
    }
}

def pretender = new PretendFieldCounter()

assert pretender.isNoField == 'pretend value'
assert pretender.count     == 0

pretender.isNoFieldEither  = 'just to increase counter'

assert pretender.count     == 1
With the count field, you can see that it looks like the get/set methods aren’t used if the requested field is present. This is true for our special case. In section 7.4 you’ll see the full set of rules that produces this effect.

Generally speaking, overriding the get method means to override the dot-fieldname operator. Overriding the set method overrides the field-assignment operator.

For the Geeks
What about a statement of the form x.y.z=something? This is equivalent to getX().getY().setZ(something).

Referencing fields is also connected to the topic of properties, which we’ll explore in section 7.4, where we’ll discuss the need for the additional obj.@fieldname syntax.

7.1.2. Methods and parameters
Method declarations follow the same concepts you’ve seen for variables: the usual Java modifiers can be used; declaring a return type is optional; and, if no modifiers or return type are supplied, the def keyword fills the hole. When the def keyword is used, the return type is deemed to be unrestricted (although it can still have no return type, the equivalent of a void method). In this case, under the covers, the return type will be java.lang.Object. The default visibility of methods is public.

The following listing shows the typical cases in a self-describing manner.

Listing 7.5. Declaring methods


The main method  has interesting twists. First, the public modifier can be omitted because it’s the default. Second, args usually has to be of type String[] to make the main method the one to start the class execution. Thanks to Groovy’s method dispatch, it works anyway, although args is now implicitly of static type java.lang.Object. Third, because return types aren’t used for the dispatch, we can further omit the void declaration.

So, the Java declaration

public static void main (String[] args)
boils down to this in Groovy:

static main (args)
Note
The Java compiler fails on missing return statements when a return type is declared for the method. In Groovy, return statements are optional, therefore it’s impossible for the compiler to detect “accidentally” missing returns.

The main(args) example illustrates that declaring explicit parameter types is optional. When type declarations are omitted, Object is used. Multiple parameters can be used in sequence, delimited by commas. The following listing shows that explicit and omitted parameter types can also be mixed.

Listing 7.6. Declaring parameter lists
class ClassWithTypedAndUntypedMethodParams {
  static void main(args) {
    assert 'untyped' == method(1)
    assert 'typed' == method('whatever')
    assert 'two args' == method(1, 2)
  }

  static method(arg) {
    return 'untyped'
  }

  static method(String arg) {
    return 'typed'
  }

  static method(arg1, Number arg2) {
    return 'two args'
  }
}
In the examples so far, all method calls have involved positional parameters, where the meaning of each argument is determined from its position in the parameter list. This is easy to understand and convenient for the simple cases you’ve seen, but suffers from a number of drawbacks for more complex scenarios:

You must remember the exact sequence of the parameters, which gets increasingly difficult with the length of the parameter list. We recommend a coding style that encourages small numbers of parameters, but this isn’t always possible.
If it makes sense to call the method with different information for alternative use scenarios, different methods must be constructed to handle these alternatives. This can quickly become cumbersome and lead to a proliferation of methods, especially where some parameters are optional. It’s especially difficult if many of the optional parameters have the same type. Fortunately, Groovy comes to the rescue with using maps as named parameters.
Note
Whenever we talk about named parameters, we mean keys of a map that are used as an argument in method or constructor calls. From a programmer’s perspective, this looks pretty much like native support for named parameters, but it isn’t. This trick is needed because the JVM doesn’t support storing parameter names in the bytecode.[5]

5 This isn’t strictly true. Some APIs define their own @ParameterName annotations to store such information and Java 8 can optionally do so. It would be more accurate to say there is no universally adopted approach that is guaranteed to be enabled.

The following listing illustrates Groovy method definitions and calls supporting positional and named parameters, parameter lists of variable length, and optional parameters with default values. The example provides four alternative summing mechanisms, each highlighting different approaches for defining the method call parameters.

Listing 7.7. Advanced parameter uses




All four alternatives have their pros and cons. In , sumWithDefaults, we have the most obvious declaration of the arguments expected for the method call. It meets the needs of the sample script—being able to add two or three numbers together—but we’re limited to as many arguments as we have declared parameters.

Using lists as shown in  is easy in Groovy, because in the method call, the arguments only have to be placed in brackets. We can also support argument lists of arbitrary length. But it’s not as obvious what the individual list entries should mean. Therefore, this alternative is best suited when all arguments have the same meaning, as they do here where they’re used for adding. Refer to section 4.2.3 for details about the List.inject method.

The sumWithOptionals method  can be called with two or more parameters. To declare such a method, define the last argument as an array. Groovy’s dynamic method dispatch bundles excessive arguments into that array.

Named arguments can be supported by using a map as in . It’s good practice to reset any missing values to a default before working with them. This also better reveals what keys will be used in the method body, because this isn’t obvious from the method declaration.

When designing your methods, you have to choose one of the alternatives. You may wish to formalize your choice within a project or incorporate the Groovy coding style.

Note
There are more ways of implementing parameter lists of variable length. You can use varargs with the method(args...) or method(Type[] args) notation or even hook into Groovy’s method dispatch by overriding the invokeMethod(name, params[]) that every GroovyObject provides. You’ll learn more about these hooks in section 7.6.2.

Advanced naming
When calling a method on an object reference, you should usually follow this format:

objectReference.methodName()
This format imposes the Java restrictions for method names; for example, they may not contain special characters such as minus (-) or dot (.). But Groovy allows you to use these characters in method names if you put quotes around the name:

objectReference.'my.methodName'()
This feature supports scenarios where the method name of a call becomes part of the functionality. You won’t normally use this feature directly, but it is used under the covers by other parts of Groovy. You’ll see this in action in chapters 8 and 10.

For the Geeks
Where there’s a string, you can generally also use a GString. So how about obj."${var}"()? Yes, this is also possible, and the GString will be resolved to determine the name of the method that’s called on the object!

That’s it for the basics of class members. Before we leave this topic, though, there’s one convenient operator we should introduce while we’re thinking about referring to members via references.

7.1.3. Safe dereferencing with the ?. operator
When a reference doesn’t point to any specific object, its value is null. When calling a method or accessing a field on a null reference, a NullPointerException (NPE) is thrown. This is useful to protect code from working on undefined preconditions, but it can easily get in the way of “best-effort” code that should be executed for valid references and just be silent otherwise.

Listing 7.8 shows several alternative approaches to protect code from NPEs. As an example, we wish to access a deeply nested entry within a hierarchy of maps, which results in a path expression—a dotted concatenation of references that’s typically cumbersome to protect from NPEs. We can use explicit if checks or use the try-catch mechanism. Groovy provides the additional ?. operator for safe dereferencing. When the reference before that operator is a null reference, the evaluation of the current expression stops, and null is returned.

Listing 7.8. Protecting from NullPointerExceptions using the ?. operator


In comparison, using the safe dereferencing operator in  is the most elegant and expressive solution.

Note that  is more compact than its Java equivalent, which would need three additional nullity checks. It works because the expression is evaluated from left to right, and the && operator stops evaluation with the first operand that evaluates to false. This is known as short-circuit evaluation.

Alternative  is a bit verbose and doesn’t allow fine-grained control to protect only selective parts of the path expression. It also abuses the exception-handling mechanism. Exceptions weren’t designed for this kind of situation, which is easily avoided by verifying that the references are non-null before dereferencing them. Causing an exception and then catching it is the equivalent of steering a car by installing big bumpers and bouncing off buildings.

Some software engineers like to think about code in terms of cyclomatic complexity (http://en.wikipedia.org/wiki/Cyclomatic_complexity), which in short describes code complexity by analyzing alternative pathways through the code. The safe dereferencing operator merges alternative pathways and therefore reduces complexity when compared to its alternatives; essentially, the metric indicates that the code will be easier to understand and simpler to verify as correct.

7.1.4. Constructors
Objects are instantiated from their classes via constructors. If no constructor is given, an implicit constructor without arguments is supplied by the compiler. This appears to be exactly like in Java, but because this is Groovy, it should not be surprising that additional features are available.

In section 7.1.2, we examined the merits of named parameters versus positional ones, as well as the need for optional parameters. The same arguments applicable to method calls are relevant for constructors, too, so Groovy provides the same convenience mechanisms. We’ll first look at constructors with positional parameters, and then we’ll examine named parameters.

Positional parameters
Until now, we’ve only used implicit constructors. The following listing introduces the first explicit one. Notice that just like all other methods, the constructor is public by default. We can call the constructor in three different ways: the usual Java way, with enforced type coercion by using the as keyword, or with implicit type coercion.

Listing 7.9. Calling constructors with positional parameters


The coercion in  and  may be surprising. When Groovy sees the need to coerce a list to some other type, it tries to call the type’s constructor with all arguments supplied by the list, in list order. This need for coercion can be enforced with the as keyword or arise from assignments to explicitly typed references. The latter of these is called implicit construction, which we’ll cover shortly.

Named parameters
Named parameters in constructors are handy. One use case that crops up frequently is creating immutable classes that have some parameters that are optional. Using positional parameters would quickly become cumbersome because you’d need to have constructors allowing for all combinations of the optional parameters.

As an example, suppose in listing 7.9 that VendorWithCtor should be immutable and name and product can be optional. We’d need four[6] constructors: an empty one, one to set name, one to set product, and one to set both attributes. To make things worse, we couldn’t have a constructor with only one argument, because we couldn’t distinguish whether to set the name or the product attribute (they’re both strings). We’d need an artificial extra argument for distinction, or we’d need to strongly type the parameters.

6 In general, 2n constructors are needed, where n is the number of optional attributes.

But don’t panic: Groovy’s special way of supporting named parameters comes to the rescue again.

The following listing shows how to use named parameters with a simplified version of the VendorWithCtor class. It relies on the implicit default constructor. Could that be any easier?

Listing 7.10. Calling constructors with named parameters
class SimpleVendor {
    String name, product
}

new SimpleVendor()
new SimpleVendor(name: 'Canoo')
new SimpleVendor(product: 'ULC')
new SimpleVendor(name: 'Canoo', product: 'ULC')

def vendor = new SimpleVendor(name: 'Canoo')
assert 'Canoo' == vendor.name
The listing illustrates how flexible named parameters are for your constructors. In cases where you don’t want this flexibility and want to lock down all of your parameters, define your desired constructor explicitly; the implicit constructor with named parameters will no longer be available.

Coming back to how we started this section, the empty default constructor invocation new SimpleVendor() appears in a new light. Although it looks exactly like its Java equivalent, it’s a special case of the default constructor with named parameters that happen to be called without any being supplied.

Implicit constructors
Finally, there’s a way to call a constructor implicitly by simply providing the constructor arguments as a list. That means that instead of calling the Dimension(width, height) constructor explicitly, for example, you can use

java.awt.Dimension area

area = [200, 100]

assert area.width  == 200
assert area.height == 100
Of course, Groovy must know which constructor to call, and therefore implicit constructors are solely available for assignment to statically typed references where the type provides the respective constructor. They don’t work for abstract classes or even interfaces.

Implicit constructors are often used with builders, as you’ll see in the SwingBuilder example in section 11.6.

That’s it for the usual class members. This is a solid basis we can build upon. But we’re not yet in the penthouse; we’ve four more levels to go. Next, we’ll walk through the topic of how to organize classes and scripts before reaching the level of advanced object-oriented features. The next floor after that is named GroovyBeans and deals with simple object-oriented information about objects. At this level, we can play with Groovy’s power features. Finally, we’ll visit the highest level, where we look at advanced syntax features.

7.2. ORGANIZING CLASSES AND SCRIPTS
In section 2.4.1, you saw that Groovy classes are Java classes at the bytecode level, and consequently, Groovy objects are Java objects in memory. At the source-code level, Groovy class and object handling is for all practical purposes a superset of the Java syntax. We’ll examine the organization of classes and source files, and the relationships between the two. We’ll also consider Groovy’s use of packages and type aliasing, as well as demystify where Groovy can load classes from its classpath.

7.2.1. File to class relationship
The relationship between files and class declarations isn’t as fixed as in Java. Groovy files can contain any number of public class declarations according to the following rules:

If a Groovy file contains no class declaration, it’s handled as a script; that is, it’s transparently wrapped into a class of type Script. This automatically generated class has the same name as the source script filename[7] (without the extension). The content of the file is wrapped into a run method, and an additional main method is constructed for easily starting the script.
7 Because the class has no package name, it’s implicitly placed in the default package.

If a Groovy file contains exactly one class declaration with the same name as the file (without the extension), then there’s the same one-to-one relationship as in Java.
A Groovy file may contain multiple class declarations of any visibility, and there’s no enforced rule that any of them must match the filename. The groovyc compiler happily creates *.class files for all declared classes in such a file. If you wish to invoke your script directly—for example, using groovy on the command line or within an IDE—then the first class within your file should have a main method.[8]
8 Strictly speaking, you can alternatively extend GroovyTestCase or implement the Runnable interface.

A Groovy file may mix class declarations and scripting code. In this case, the scripting code will become the main class to be executed, so don’t declare a class yourself having the same name as the source filename.
When not compiling explicitly, Groovy finds a class by matching its name to a corresponding *.groovy source file. At this point, naming becomes important. Groovy only finds classes where the class name matches the source filename. When such a file is found, all declared classes in that file are parsed and become known to Groovy.

The following listing shows a sample script with two simple classes, Vendor and Address. For the moment, they have no methods, only public fields.

Listing 7.11. Multiple class declarations in one file
class Vendor {
    public String     name
    public String     product
    public Address    address = new Address()
}

class Address  {
    public String     street, town, state
    public int        zip
}

def canoo = new Vendor()
canoo.name            = 'Canoo Engineering AG'
canoo.product         = 'UltraLightClient (ULC)'
canoo.address.street  = 'Kirschgartenst. 7'
canoo.address.zip     =  4051
canoo.address.town    = 'Basel'
canoo.address.state   = 'Switzerland'

assert canoo.dump()         =~ /ULC/
assert canoo.address.dump() =~ /Basel/
Vendor and Address are simple data storage classes. They’re roughly equivalent to struct in C or record in Pascal. We’ll soon explore more elegant ways of defining such classes.

The previous example illustrates a convenient convention supported by Groovy’s source file to class mapping rules, which we discussed earlier. This convention allows small helper classes that are used only with the current main class or current script to be declared within the same source file. Compare this with Java, which allows you to use nested classes to introduce locally used classes without cluttering up your public class namespace or making navigation of the codebase more difficult by requiring a proliferation of source-code files. Although it isn’t exactly the same, this convention has similar benefits for Groovy developers.

7.2.2. Organizing classes in packages
Groovy follows Java’s approach of organizing files in packages of hierarchical structure. The package structure is used to find the corresponding class files in the filesystem’s directories.

Because *.groovy source files aren’t necessarily compiled to *.class files, there’s also a need to look up *.groovy files. When doing so, the same strategy is used: the compiler looks for a Groovy class Vendor in the business package in the file business/Vendor.groovy.

In listing 7.12, we separate the Vendor and Address classes from the script code, as shown in listing 7.11, and move them to the business package.

Classpath
The lookup has to start somewhere, and Java uses its classpath for this purpose. The classpath is a list of possible starting points for the lookup of *.class files. Groovy reuses the classpath for looking up *.groovy files.

When looking for a given class, if Groovy finds both a *.class and a *.groovy file, it uses whichever is newer; that is, it’ll recompile source files into *.class files if they’ve changed since the previous class file was compiled.[9]

9 Whether classes are checked for runtime updates can be controlled by the CompilerConfiguration, which obeys the system property groovy.recompile by default. See the API documentation for details.

Packages
Exactly like in Java, Groovy classes must specify their package before the class definition. When no package declaration is given, the default package is assumed.

The following listing shows the file business/Vendor.groovy, which has a package statement as its first line.

Listing 7.12. Vendor and Address classes moved to the business package
package business

class Vendor {
    public String  name
    public String  product
    public Address address = new Address()
}

class Address  {
    public String  street, town, state
    public int     zip
}
To reference Vendor in the business package, you can either use business.Vendor within the code or use import statements for abbreviation.

Imports
Groovy follows Java’s notion of allowing import statements before any class declaration to abbreviate class references.

Note
Please keep in mind that unlike in some other scripting languages, an import statement has nothing to do with literal inclusion of the imported class or file. It merely informs the compiler how to resolve references.

The following listing shows the use of an import statement, with the .* notation advising the compiler to try resolving all unknown class references against all classes in the business package.

Listing 7.13. Using import to access Vendor in the business package
import business.*

def canoo = new Vendor()
canoo.name          = 'Canoo Engineering AG'
canoo.product       = 'UltraLightClient (ULC)'

assert canoo.dump() =~ /ULC/
Default import statements
By default, Groovy imports six packages and two classes, making it seem like every Groovy code program contains the following initial statements:

import java.lang.*
import java.util.*
import java.io.*
import java.net.*
import groovy.lang.*
import groovy.util.*
import java.math.BigInteger
import java.math.BigDecimal
Type aliasing
An import statement has another nice twist: together with the as keyword, it can be used for type aliasing. Whereas a normal import statement allows a fully qualified class to be referred to by its base name, a type alias allows a fully qualified class to be referred to by a name of your choosing. This feature resolves naming conflicts and supports local changes or bug fixes to a third-party library.

Consider the following library class:

package thirdparty

class MathLib {
  Integer twice(Integer value) {
    return value * 3         // intentionally wrong!
  }
  Integer half(Integer value) {
    return value / 2
  }
}
Note its obvious error[10] (although in general it might not be an error but just a locally desired modification). Suppose now that we have existing code that uses that library:

10 Where are the library author’s unit tests?

assert 10 == new MathLib().twice(5)
We can use a type alias to rename the old library and then use an inheritance to make a fix. No change is required to the original code that was using the library, as you can see in the following listing.

Listing 7.14. Using import as for local library modifications


Now, suppose that we have the following additional math library that we need to use:

package thirdparty2

class MathLib {
    Integer increment(Integer value) {
        return value + 1
    }
}
Although it has a different package, it has the same name as the previous library. Without aliasing, we have to fully qualify one or both of the libraries within our code. With aliasing, we can avoid this in an elegant way and improve communication by better indicating intent within our program about the role of the third-party library’s code, as shown in the next listing.

Listing 7.15. Using import as for avoiding name clashes
import thirdparty.MathLib as TwiceHalfMathLib
import thirdparty2.MathLib as IncMathLib
def math1 = new TwiceHalfMathLib()
def math2 = new IncMathLib()
assert 3 == math1.half(math2.increment(5))
If we later find a math package with both increment and twice/half functionality, we can refer to that new library twice and keep our more meaningful names.

You should consider using aliases within your own program, even when using simple built-in types. If you’re developing an adventure game, for example, you might alias Map to SatchelContents. (Here we mean java.util.Map and not TreasureMap, which our adventure game might allow us to place within the satchel!) This doesn’t provide the strong typing that defining a separate SatchelContents class would give, but it does greatly improve the human understandability of the code.

7.2.3. Further classpath considerations
Finding classes in *.class and *.groovy files is an important part of working with Groovy, and unfortunately a likely source of problems.

If you installed the JDK including the documentation, you’ll find the classpath explanation under %JAVA_HOME%/docs/technotes/tools/windows/classpath.html under Windows, or under a similar directory for Linux and Solaris. Everything the documentation says equally applies to Groovy.

A number of contributors can influence the effective classpath in use. The overview in table 7.1 may serve as a reference when you’re looking for a possible “bad guy” that’s messing up your classpath.

Table 7.1. Forming the classpath
Origin

Definition

Purpose and Use

JDK/JRE	%JAVA_HOME%/lib %JAVA_HOME%/lib/ext	Boot classpath for the Java runtime environment and its extensions
OS setting	CLASSPATH variable	Provides general default settings
Command shell	CLASSPATH variable	Provides more specialized settings
Java	-cp --classpath option	Settings per runtime invocation
Groovy	%GROOVY_HOME%/lib	Groovy runtime environment
Groovy	-cp	Settings per groovy execution call
Groovy	.	Groovy classpath defaults to the current directory
Groovy defines its classpath in a special configuration file under %GROOVY_HOME%/conf. Looking at the file groovy-starter.conf reveals the following lines (beside others):

# Load required libraries
load ${groovy.home}/lib/*.jar
# load user specific libraries
# load ${user.home}/.groovy/lib/*
Uncommenting the last line by removing the leading hash sign enables a cool feature. In your personal home directory user.home, you can use a subdirectory .groovy/lib (note the leading dot!), where you can store any *.class or *.jar files that you want to have accessible whenever you work with Groovy.

If you have problems finding your user.home, open a command shell and execute

groovy -e "println System.properties.'user.home'"
Chances are, you’re in this directory by default anyway.

Chapter 16 goes through more advanced classpath issues that need to be respected when embedding Groovy in environments that manage their own class-loading infrastructure—for example, an application server.

You’re now able to use constructors in a number of different ways to make new instances of a class. Classes may reside in packages, and you’ve seen how to make them known via import statements. This wraps up our exploration of object basics. The next step is to explore more advanced object-oriented features.

7.3. ADVANCED OBJECT-ORIENTED FEATURES
Before beginning to embrace further parts of the Groovy libraries that make fundamental use of the object-oriented features we’ve been discussing, we first stop to briefly explore other object-oriented concepts that change once you enter the Groovy world. We’ll cover inheritance and interfaces, which will be familiar from Java, and multimethods, which will give you a taste of the dynamic object orientation coming later. Finally, we’ll look at Groovy’s support for traits that offer incredible flexibility when composing functionality.

7.3.1. Using inheritance
You’ve seen how to explicitly add your own fields, methods, and constructors into your class definitions. Inheritance allows you to implicitly add fields and methods from a base class. The mechanism is useful in a range of use cases. We leave it up to others[11] to describe its benefits and warn you about the potential overuse of this feature. We simply let you know that all the inheritance features of Java (including abstract classes) are available in Groovy and also work (almost seamlessly[12]) between Groovy and Java.

11 Rebecca Wirfs-Brock et al., Designing Object-Oriented Software (Prentice-Hall, 1990) is a good place to begin.

12 The only limitation that we’re aware of has to do with map-based constructors, which Groovy provides by default. These aren’t available directly in Java if you extend a Groovy class. They’re provided by Groovy as a runtime trick.

Groovy classes can extend Groovy and Java classes and interfaces alike. Java classes can also extend Groovy classes and interfaces. You need to compile your Java and Groovy classes in a particular order for this to work (see section 16.4.2 for more details). The only other thing you need to be aware of is that Groovy is more dynamic than Java when it selects which methods to invoke for you. This feature is known as multimethods and is discussed further in section 7.3.3.

7.3.2. Using interfaces
A frequently advocated style of Java programming involves using Java’s interface mechanism. Code written using this style refers to the dependent classes that it uses solely by interface. The dependent classes can be safely changed later without requiring changes to the original program. If a developer accidentally tries to change one of the classes for another that doesn’t comply with the interface, this discrepancy is detected at compile time. Groovy fully supports the Java interface mechanism.

Some[13] argue that interfaces alone aren’t strong enough, and design-by-contract is more important for achieving safe object substitution and allowing nonbreaking changes to your libraries. Judicious use of abstract methods and inheritance becomes just as important as using interfaces. Groovy’s support for Java’s abstract methods, its automatically enabled assert statement, and its built-in ready access to test methods mean that it’s ideally suited to also support this stricter approach.

13 See Bertrand Meyer, Object-oriented Software Construction, 2nd ed. (Prentice-Hall, 1997) and http://cafe.elharo.com/java/the-three-reasons-for-data-encapsulation/.

Still others argue that dynamic typing is the best approach, leading to much less typing and less scaffolding code without much reduced safety—which should be covered by tests in any case. The good news is that Groovy supports this style as well. To give you a flavor of how this would impact you in everyday coding, consider how you’d build a plug-in mechanism in Java and Groovy.

In Java, you’d normally write an interface for the plug-in mechanism and then an implementation class for each plug-in that implements that interface. In Groovy, dynamic typing allows you to more easily create and use implementations that meet a certain need. You’re likely to be able to create just two classes as part of developing two plug-in implementations. In general, you have a lot less scaffolding code and a lot less typing.

Implementing interfaces and SAM types
If you decide to make heavy use of interfaces, Groovy provides ways to make them more dynamic. If you have an interface, MyInterface, with a single method and a closure, myClosure, you can use the as keyword to coerce the closure to be of type MyInterface. In fact from Groovy 2.2, you don’t even need the as keyword. Groovy does implicit closure coercion into single abstract method types as shown in this example, where the addListener method would normally require an ActionListener:

import java.awt.event.ActionListener
listeners = []
def addListener(ActionListener al) { listeners << al }
addListener { println "I heard that!" }
listeners*.actionPerformed()
Alternatively, if you have an interface with several methods, you can create a map of closures keyed on the method names and coerce the map to your interface type. See the Groovy documentation for more details: http://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#closure-coercion.

In summary, if you’ve come from the Java world, you may be used to following a strict style of coding that strongly encourages interfaces. When using Groovy, you’re not compelled to stick with any one style. In many situations, you can minimize the amount of typing by making use of dynamic typing; and if you really need it, the full use of interfaces is available.

7.3.3. Multimethods
Remember that Groovy’s mechanics of method lookup take the dynamic type of method arguments into account, whereas Java relies on the static type. This Groovy feature is called multimethods.

The following listing shows two methods, both called oracle, that are distinguishable only by their argument types. They’re called two times with arguments of the same static type but different dynamic types.

Listing 7.16. Multimethods: method dispatch on runtime type


The x argument is of static type Object and of runtime type Integer. The y argument is of static type Object but of runtime type String. Both arguments are of the same static type, which would make the equivalent Java program dispatch both to oracle(Object). Because Groovy dispatches by the runtime type, the specialized implementation of oracle(String) is used in the second case.

With this capability in place, you can better avoid duplicated code by being able to override behavior more selectively. Consider the equals implementation in the following listing that overrides Object’s default equals method only for the argument type Equalizer.

Listing 7.17. Multimethods to selectively override equals
class Equalizer {
    boolean equals(Equalizer e){
        return true
    }
}

Object same  = new Equalizer()
Object other = new Object()

assert   new Equalizer().equals( same  )
assert ! new Equalizer().equals( other )
When an object of type Equalizer is passed to the equals method, the specialized implementation is chosen. When an arbitrary object is passed, the default implementation of its superclass Object.equals is called, which implements the equality check as a reference identity check.

The net effect is that the caller of the equals method can be fully unaware of the difference. From a caller’s perspective, it looks like equals(Equalizer) would override equals(Object), which would be impossible to do in Java. Instead, a Java programmer has to write it like this:

public class Equalizer {             // Java
    public boolean equals(Object obj)
    {
        if (obj == null)                 return false;
        if (!(obj instanceof Equalizer)) return false;
        Equalizer w = (Equalizer) obj;
        return true;                 // custom logic here
    }
}
This is unfortunate, because the logic of how to correctly override equals needs to be duplicated for every custom type in Java. This is another example where Java uses the static type Object and leaves the work of dynamic type resolution to the programmer.

Note
Wherever there’s a Java API that uses the static type Object, this code effectively loses the strength of static typing. You’ll inevitably find it used with typecasts, compromising compile-time type safety. This is why the Java type concept is called weak static typing: you lose the merits of static typing without getting the benefits of a dynamically typed language such as multimethods.

Groovy, in contrast, comes with a single and consistent implementation of dispatching methods by the dynamic types of their arguments.

That’s it for multimethods, but in terms of advanced object-oriented features, we’ve saved the best for last. One of the benefits of object-oriented languages is the flexibility in designing systems through composition and subtyping. Java’s decision to adopt single inheritance of implementation greatly simplified the language at the expense of making it more difficult to support certain kinds of reuse. We’ve all heard the mantra “prefer delegation over inheritance.” It’s arguable that this is a direct consequence of Java’s restrictions. A programmer might have the desire to share code capabilities within their classes without duplication, but given Java’s restrictions, they create inappropriate subtype relationships. Default methods in Java 8 interfaces lift this restriction somewhat but still don’t allow a full “design by capability” that includes state. Groovy introduces traits to support composition of capabilities in a very flexible way. We’ll cover traits next.

7.3.4. Using traits
Groovy traits support composition of capabilities. Capabilities that are designed to be shared are implemented in traits. Your classes can then implement those traits to indicate that they provide that capability.[14] They “inherit” the implementation from the trait but can override it if they wish. If this sounds like Java 8 default methods, you’re on the right track, but Groovy traits also support state. Let’s look at some examples.

14 Before Groovy 2.3 the way to share capabilities among classes was through the @Mixin annotation, which is now deprecated in favor of more powerful traits.

Assume we have to model a Book class. Books have an ISBN number. Books also have a title just like other types of publications but not all publications have an ISBN. So our domain is books that are subclasses of publications:

class Publication {
    String title
}

class Book extends Publication {
    String isbn
}
But in our application, we also need to save Book instances to a database. Saving has nothing to do with books or publications but is an independent capability that applies to all persistent entities.

The following listing solves this design task with Groovy traits in a very fine-grained manner such that one can mix-and-match the capabilities of having identifiers and versions for persistent entities.

Listing 7.18. Traits with inheritance


At  we make Publication an Entity. This is what we call the intrusive way of applying traits. There’s an even more flexible one: applying them nonintrusively at runtime. Publications stay totally agnostic of persistency:

class Publication {
    String title
}
We apply the trait later when needed with the enforced coercion through the as keyword:

Entity gina = new Book(title:"gina", isbn:"111111") as Entity
gina.id = 1
gina.version = 1
Note that gina is no longer of type Book as it was before. That’s the price we pay for flexibility. But this nonintrusive way of extending a class independent from its inheritance in a type-safe manner is a great way of developing incrementally.

That’s all you need to know to get started using traits. There are some more advanced details such as rules for dealing with conflicts when two or more traits define the same method, and rules for chaining traits and restrictions when combining traits with AST transformations. See the Groovy documentation on traits for more details.[15]

15 Groovy Language Documentation, 1.4.2, “Traits,” http://docs.groovy-lang.org/docs/groovy-latest/html/documentation/#_traits.

7.4. WORKING WITH GROOVYBEANS
The JavaBeans specification[16] was introduced with Java 1.1 to define a lightweight and generic software component model for Java. The component model builds on naming conventions and APIs that allow Java classes to expose their properties to other classes and tools. This greatly enhanced the ability to define and use reusable components and opened up the possibility of developing component-aware tools.

16 The JavaBeans spec, www.oracle.com/technetwork/java/javase/overview/spec-136004.html.

The first tools were mainly visually oriented, such as visual builders that retrieved and manipulated properties of visual components. Over time, the JavaBeans concept has been widely used and extended to a range of use cases including server-side components (in JavaServer Pages [JSP]), transactional behavior and persistence (Enterprise JavaBeans [EJB]), object-relational mapping (ORM) frameworks, and countless other frameworks and tools.

Groovy makes using JavaBeans (and therefore most of these other JavaBean-related frameworks) easier with special language support. This support covers three aspects: special Groovy syntax for creating JavaBean classes; mechanisms for easily accessing beans, regardless of whether they were declared in Groovy or Java; and support for JavaBean event handling. This section will examine each part of this language-level support, as well as cover the library support provided by the Expando class.

7.4.1. Declaring beans
JavaBeans are normal classes that follow certain naming conventions. For example, to make a String property myProp available in a JavaBean, the bean’s class must have public methods declared as String getMyProp and void setMyProp (String value). The JavaBean specification also strongly recommends that beans should be serializable so they can be persistent and provide a parameterless constructor to allow easy construction of objects from within tools. A typical Java implementation is

// Java
public class MyBean implements java.io.Serializable {
  private String myprop;
  public String getMyprop(){
    return myprop;
  }
  public void setMyprop(String value){
    myprop = value;
  }
}
The Groovy equivalent is

class MyBean implements Serializable {
  String myprop
}
The most obvious difference is size. One line of Groovy replaces seven lines of Java. But it’s not only about less typing, it’s also about self-documentation. In Groovy, it’s easier to assess which fields are considered exposed properties: all fields that are declared with default visibility. The three related pieces of information—the field and the two accessor methods—are kept together in one declaration. Changing the type or name of the property requires changing the code in only a single place.

Note
Older versions of Groovy used an @Property syntax for denoting properties. This was considered ugly and was removed in favor of handling properties as a “default visibility.”

Underneath the covers, Groovy provides public accessor methods similar to this Java code equivalent, but you don’t have to type them. Moreover, they’re generated only if they don’t already exist in the class. This allows you to override the standard accessors with either customized logic or constrained visibility. Groovy also provides a private backing field (again similar to the Java equivalent code). Note that the JavaBean specification cares only about the available accessor methods and doesn’t even require a backing field, but having one is an intuitive and simple way to implement the methods—so that’s what Groovy does.

Note
It’s important that Groovy constructs the accessor methods and adds them to the bytecode. This ensures that when using a MyBean in the Java world, the Groovy MyBean class is recognized as a proper JavaBean.

The following listing shows the declaration options for properties with optional typing and assignment. The rules are equivalent to those for fields (see section 7.2.1).

Listing 7.19. Declaring properties in GroovyBeans
class MyBean implements Serializable {
    def untyped
    String typed
    def item1, item2
    def assigned = 'default value'
}
def bean = new MyBean()
assert 'default value' == bean.getAssigned()
bean.setUntyped('some value')
assert 'some value' == bean.getUntyped()
bean = new MyBean(typed:'another value')
assert 'another value' == bean.getTyped()
Properties are sometimes called readable or writeable depending on whether the corresponding getter or setter method is available. Groovy properties are both readable and writeable, but you can always roll your own if you have special requirements. When the final keyword is used with a property declaration, the property will only be readable (no setter method is created and the backing field is final).

Writing GroovyBeans is a simple and elegant solution for fully compliant JavaBean support, with the option of specifying types as required.

7.4.2. Working with beans
The wide adoption of the JavaBeans concept in the world of Java has led to a common programming style where bean-style accessor methods are limited to simple access (costly operations are strictly avoided in these methods). These are the types of accessors generated for you by Groovy. If you have complex additional logic related to a property, you can always override the relevant getter or setter method, but you’re usually better off writing a separate business method for your advanced logic.

Accessor methods
Even for classes that don’t fully comply with the JavaBeans standard, you can usually assume that such an accessor method can be called without a big performance penalty or other harmful side effects. The characteristics of an accessor method are much like those of a direct field access (without breaking the uniform access principle[17]).

17 “Uniform Access Principle,” http://en.wikipedia.org/wiki/Uniform_access_principle.

Groovy supports this style at the language level according to the mapping of method calls shown in table 7.2.

Table 7.2. Groovy accessor method to property mappings
Java

Groovy

getPropertyname()	propertyname
setPropertyname(value)	propertyname = value
This mapping works regardless of whether it’s applied to a Groovy or plain old Java object (POJO), and it works for beans as well as for all other classes. The next listing shows this in a combination of bean-style and derived properties.

Listing 7.20. Calling accessors the Groovy way


Note how much the Groovy-style property access in  and  looks like direct field access, whereas  makes clear that there’s no field but only some derived value. From a caller’s point of view, the access is truly uniform.

Because field access and the accessor method shortcut have an identical syntax, it takes rules to choose one or the other.

Note
When both a field and the corresponding accessor method are accessible to the caller, the property reference is resolved as an accessor method call. If only one is accessible, that option is chosen.

That looks straightforward, and it is in the majority of cases. But there are some points to consider, as you’ll see next.

Field access with .@
Before we leave the topic of properties, we have one more example to explore. The following listing illustrates how you can provide your own accessor methods and also how to bypass the accessor mechanism. You can get directly to the field using the .@ (dot-at) operator when the need arises.

Listing 7.21. Advanced accessors with Groovy


Let’s start with what’s familiar: bean.value at  calls getValue and thus returns the doubled value. But wait—getValue calculates the result at  as value * 2. If value at this point was interpreted as a bean shortcut for getValue, we’d have an endless recursion.

A similar situation arises at , where the assignment this.value = would in bean terms be interpreted as this.setValue, which would also let us fall into endless looping. Therefore, the following rules have been set up.

Rules
Inside the lexical scope of a field, references to fieldname or this.fieldname are resolved as field access, not as property access. The same effect can be achieved from outside the scope using the reference.@fieldname syntax.

It needs to be mentioned that these rules can produce pathological corner cases with logical but surprising behavior, such as when using @ from a static context or with def x=this; x.@fieldname, and so on. We’ll not go into more details here, because such a design is discouraged. Decide whether to expose the state as a field, as a property, or via explicit accessor methods, but don’t mix these approaches. Keep the access uniform.

Bean-style event handling
Besides properties, JavaBeans can also be event sources that feed event listeners.[18] An event listener is an object with a callback method that gets called to notify the listener that an event was fired. An event object that further qualifies the event is passed as a parameter to the callback method.

18 See the JavaBeans Specification: http://www.oracle.com/technetwork/java/javase/overview/spec-136004.html.

The JDK is full of different types of event listeners. A simple event listener is the ActionListener on a button, which calls an actionPerformed(ActionEvent) method whenever the button is clicked. A more complex example is the VetoableChangeListener that allows listeners to throw a PropertyVetoException inside their vetoableChange(PropertyChangeEvent) method to roll back a change to a bean’s property. Other uses are multifold, and it’s impossible to provide an exhaustive list.

Groovy supports event listeners in a simple but powerful way. Suppose you need to create a Swing JButton with the label “Push me!” that prints the label to the console when it’s clicked. A Java implementation can use an anonymous inner class in the following way:

// Java
final JButton button = new JButton("Push me!");
button.addActionListener(new IActionListener(){
    public void actionPerformed(ActionEvent event){
        System.out.println(button.getText());
    }
});
The developer needs to know about the respective listener and event types (or interfaces), as well as about the registration and callback methods.

A Groovy programmer only has to attach a closure to the button as if it were a field named by the respective callback method:

button = new JButton('Push me!')
button.actionPerformed = { event ->
    println button.text
}
The event parameter is added only to show how we could get it when needed. In this example, it could have been omitted, because it’s not used inside the closure.

Note
Groovy uses bean introspection to determine whether a field setter refers to a callback method of a listener that’s supported by the bean. If so, a ClosureListener is transparently added that calls the closure when notified. A ClosureListener is a proxy implementation of the required listener interface.

Event handling is conceived as a JavaBeans standard. But you don’t need to somehow declare your object to be a bean before you can do any event handling. The dependency is the other way around: as soon as your object supports this style of event handling, it’s called a bean.

Although Groovy adds the ability to register event listeners easily as closures, the Java style of bean event handling remains fully intact. That means you can still use all available Java methods to get a list of all registered listeners, adding more of them, or removing them when they’re no longer needed.

7.4.3. Using bean methods for any object
Groovy doesn’t distinguish between beans and other kinds of object. It solely relies on the accessibility of the respective getter and setter methods.

The following listing shows how to use the getProperties method and thus the properties property (sorry for the tricky wording) to get a map of a bean’s properties. You can do so with any object you fancy.

Listing 7.22. GDK methods for bean properties
class ClassWithProperties {
    def       someProperty
    public    someField
    private   somePrivateField
}

def obj = new ClassWithProperties()

def store = []
obj.properties.each { property ->
    store += property.key
    store += property.value
}
assert store.contains('someProperty')
assert store.contains('someField')        == false
assert store.contains('somePrivateField') == false
assert store.contains('class')

assert obj.properties.size() == 2
In addition to the property that’s explicitly declared, you also see class and metaClass references. These are artifacts of the Groovy class generation.[19]

19 The class property stems from Java. But tools that use Java’s bean introspection often hide this property.

This was a taste of what will be explained in more detail in section 12.1.

7.4.4. Fields, accessors, maps, and Expando
In Groovy code, you’ll often find expressions such as object.name. Here’s what happens when Groovy resolves this reference:

If object refers to a map, object.name refers to the value corresponding to the name key that’s stored in the map. Otherwise, if name is a property of object, the property is referenced (with precedence of accessor methods over fields, as you saw in section 7.4.2).
Every Groovy object has the opportunity to implement its own getProperty (name) and setProperty(name, value) methods. When it does, these implementations are used to control the property access. Maps, for example, use this mechanism to expose keys as properties.
Field access can be intercepted by providing the object.get(name) method, as shown in section 7.1.1. This is a last resort as far as the Groovy runtime is concerned: it’s used only when there’s no appropriate JavaBeans property available and when getProperty isn’t implemented.
It’s worth noting that when name contains special characters that wouldn’t be valid for an identifier, it can be supplied in string delimiters—for example, object.'my-name'. You can also use a GString: def name = 'my-name'; object. "$name". As you saw in section 7.1.1 and we’ll further explore in section 12.1.1, there’s also a getAt implementation on Object that delegates to the property access so that you can access a property via object[name].

The rationale behind the admittedly nontrivial reference resolution is to allow dynamic state and behavior for Groovy objects. Groovy comes with an example of how useful this feature is: Expando. An Expando can be thought of as an expandable alternative to a bean, albeit one that can be used only within Groovy and not directly in Java. It supports the Groovy style of property access with a few extensions. Listing 7.23 shows how an Expando object can be expanded with properties by assignment, analogous to maps. The difference comes with assigning closures to a property. Those are executed when accessing the property, optionally taking parameters. In the example, the boxer fights back by returning multiple times what he has taken before.

Listing 7.23. An example using Expando
def boxer = new Expando()

assert null == boxer.takeThis

boxer.takeThis = 'ouch!'

assert 'ouch!' == boxer.takeThis

boxer.fightBack = {times -> delegate.takeThis * times  }

assert 'ouch!ouch!ouch!' == boxer.fightBack(3)
In a way, Expando’s ability to assign closures to properties and have property access calling the stored closures is like dynamically attaching methods to an object.

Maps and Expandos are extreme solutions when it comes to avoiding writing simple data structures as classes, because they don’t require any extra class to be written. In Groovy, accessing the keys of a map or the properties of an Expando doesn’t look different from accessing the properties of a full-blown JavaBean. This comes at a price: Expandos cannot be used as beans in the Java world and don’t support any kind of typing.

7.5. USING ADVANCED SYNTAX FEATURES
This section presents three power features that Groovy supports at the language level: GPath, the spread operator, and command chains.

We start by looking at GPaths. A GPath is a construction in Groovy code that powers object navigation. The name is chosen as an analogy to XPath, which is a standard for describing traversal of XML (and equivalent) documents. Just like an XPath, a GPath is aimed at expressiveness: realizing short, compact expressions that are still easy to read.

GPaths are almost entirely built on concepts that you’ve already seen: field access, shortened method calls, and the GDK methods added to Collection. They introduce only one new operator: the *. (spread-dot) operator. Let’s start working with it right away.

7.5.1. Querying objects with GPaths
We’ll explore Groovy by paving a path through the Reflection API. The goal is to get a sorted list of all getter methods for the current object. We’ll do so step by step, so please open a groovyConsole and follow along. You’ll try to get information about your current object, so type

this
and run the script (by pressing Ctrl-Enter). In the output pane, you’ll see something like

Script1@e7e8eb
which is the string representation of the current object. To get information about the class of this object, you could use this.getClass, but in Groovy you can type

this.class
which displays (after you run the script again)

class Script2
The class object reveals available methods with getMethods, so type

this.class.methods
which prints a long list of method object descriptions. This is too much information for the moment. You’re only interested in the method names. Each method object has a getName method, so call

this.class.methods.name
and get a list of method names, returned as a list of string objects. You can easily work on it, applying what you learned about strings, regular expressions, and lists. Because you’re only interested in getter methods and want to have them sorted, type

this.class.methods.name.grep(~/get.*/).sort()
and voilà, you’ll get the result

["getBinding", "getClass", "getMetaClass", "getProperty"]
Such an expression is called a GPath. One special thing about it is that you can call the name property on a list of method objects and receive a list of string objects—that is, the names.

The rule behind this is that

list.property
is equal to

list.collect{ item -> item?.property }
This is an abbreviation of the special case when properties are accessed on lists. The general case reads like

list*.member
where *. is the spread-dot operator and member can be a field access, a property access, or a method call. The spread-dot operator is needed whenever a method should be applied to all elements of the list rather than to the list itself. It is equivalent to

list.collect{ item -> item?.member }
To see GPath in action, we step into an example that’s reasonably close to reality. Suppose you’re processing invoices that consist of line items, where each line refers to the sold product and a multiplicity. A product has a price in dollars and a name. An invoice could look like table 7.3.

Table 7.3. Sample invoice
Name

Price in $

Count

Total

ULC	1,499	5	7,495
Visual editor	499	1	499
Figure 7.1 depicts the corresponding software model in a UML class diagram. The Invoice class aggregates multiple LineItems that in turn refer to a Product.

Figure 7.1. UML class diagram of an Invoice class that aggregates multiple instances of a LineItem class, which in turn aggregates exactly one instance of a Product class


The following listing is the Groovy implementation of this design. It defines the classes as GroovyBeans, constructs sample invoices with this structure, and uses GPath expressions to query the object graph in multiple ways.

Listing 7.24. Invoice example for GPath




The queries in the listing are fairly involved. The first  finds the total for each invoice, adding up all the line items. We then run a query  which finds all the names of products that have a line item with a total of over $7,000. The next query  finds the date of each invoice containing a purchase of the ULC product and turns it into a string.

Printing the full Java equivalent here would cover several pages and would not be a very exciting read. Java 8 improves the Java comparison to some degree, but the metrics still very much favor Groovy whether you measure lines of code (LOC), number of statements, or complexity in the sense of nesting depth.

Writing less code isn’t just an exercise for its own sake. It also means lower chances of making errors and thus less testing effort. Whereas some new developers think of a good day as one in which they’ve added lots of lines to the codebase, we consider a really good day as one in which we’ve added functionality but removed lines from the codebase.

In a lot of languages, less code comes at the expense of clarity. Not so in Groovy. The GPath example is the best proof. It’s much easier to read and understand than its Java counterpart. Even the complexity metrics are superior.

As a final observation, consider maintainability. Suppose your customer refines their requirements, and you need to change the lookup logic. How much effort does that take in Groovy as opposed to Java?

7.5.2. Injecting the spread operator
Groovy provides a * (spread) operator that’s connected to the spread-dot operator in that it deals with tearing a list apart. It can be seen as the reverse counterpart of the subscript operator that creates a list from a sequence of comma-separated objects. The spread operator distributes all items of a list to a receiver that can take this sequence. Such a receiver can be a method that takes a sequence of arguments or a list constructor.

What is this good for? Suppose you’ve a method that returns multiple results in a list, and your code needs to pass these results to a second method. The spread operator distributes the result values over the second method’s parameters:

def getList(){
    return [1,2,3]
}
def sum(a,b,c){
    return a + b + c
}
assert 6 == sum(*list)
This allows clever meshing of methods that return and receive multiple values while allowing the receiving method to declare each parameter separately.

The distribution with the spread operator also works on ranges and when distributing all items of a list into a second list:

def range = (1..3)
assert [0,1,2,3] == [0,*range]
The same trick can be applied to maps:

def map = [a:1,b:2]
assert [a:1, b:2, c:3] == [c:3, *:map]
The spread operator eliminates the need for boilerplate code that would otherwise be necessary to merge lists, ranges, and maps into the expected format. You’ll see this in action in section 10.3, where this operator helps implement a user command language for database access.

As shown in the previous assertions, the spread operator is used inside expressions, supporting a functional style of programming as opposed to a procedural style. In a procedural style, you’d introduce statements like list.addAll(otherlist).

Now comes a Groovy’s syntax feature that makes reading Groovy code like plain English.

7.5.3. Concise syntax with command chains
Much of the Groovy goodness comes from the combination of simple language features. Command chains are such a feature that’s based on a very simple idea: one can omit dots and parentheses in chain-of-method calls.

When a chain-of-method call looks like

link(producer).to(consumer)
then Groovy allows you to write this as

link producer to consumer
which reads like an English sentence and is immensely useful in DSLs.

Command chains are also possible with methods that have multiple arguments or that take an argument map. The following lines are equivalent:

move(10, forward).painting(color:blue)
move 10, forward painting color:blue
Note that this is a pure syntax feature and doesn’t require any special provision when defining the methods. It works with all methods that have at least one argument. Without an argument, the syntax would be ambiguous. A method without parentheses and not at least one argument is indistinguishable from property access. If that were allowed, the sequencing of method names and arguments would be destroyed with one exception: in the very last position of a method chain. And, in fact, this is the one and only position where a property access is allowed.

Groovy is often perceived as a scripting language for the JVM, and it is. Scripts cannot impose lots of ceremonial syntax to please the compiler. That would be too cumbersome for the script author. But making Java scriptable isn’t the most distinctive feature. Groovy syntax is made so that the programmer can always clearly reveal his intent.

7.6. SUMMARY
Congratulations on making it to the end of this chapter. If you’re new to dynamic languages, your head may be spinning right now—it’s been quite a journey!

The chapter started without too many surprises, showing the similarities between Java and Groovy in terms of defining and organizing classes. As we introduced named parameters for constructors and methods, optional parameters for methods, and dynamic field lookup with the subscript operator, as well as Groovy’s “load at runtime” abilities, it became obvious that Groovy has more spring in its step than Java.

Groovy’s handling of the JavaBeans conventions reinforced this, as we showed Groovy classes with JavaBean-style properties that were simpler and more expressive to both create and use than their Java equivalents. By the time you saw Groovy’s power features such as GPath, command chains, and traits, you could value the clarity of the syntax.

In retrospect, the dependencies and mutual support between these different aspects of the language become obvious: using the map datatype with default constructors, using the range datatype with the subscript operator, using operator overriding with the switch control structure, using closures for grepping through a list, using the list datatype in generic constructors, using bean properties with a fieldlike syntax, and so on. This seamless interplay not only gives Groovy its power but also makes it fun to use.

What’s perhaps most striking is the compactness of the Groovy code, while the readability is preserved if not enhanced. It has been reported[20] that developer productivity hasn’t improved much since the 1970s in terms of lines of code written per day. The boost in productivity comes from the fact that a single line of code nowadays expresses much more than in previous eras. Now, if a single line of Groovy can replace multiple lines of Java, we could start to see the next major boost in developer productivity.

20 The Journal of Defense Software Engineering, 8/2000, www.crosstalkonline.org/storage/issue-archives/2000/200008/200008-0-Issue.pdf, based on the work of Gerald M. Weinberg.

Descriptions of static languages can stop at this point. You learned the syntax and that’s it. Not so for Groovy. You’ve only swallowed the blue pill of the static parts of the language. Next comes the red pill for you: Groovy’s dynamic language features.

Recommended Playlists  History Topics Tutorials Settings Support Get the App Sign Out
© 2018 Safari. Terms of Service / Privacy Policy