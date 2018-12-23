# Chapter 9. Compile-time metaprogramming and AST transformations
This chapter covers

Removing redundancy and verbosity with Groovy’s metaprogramming annotations
Writing your own compiler extensions using the AST transformations feature
Compile-time metaprogramming testing, tools, and pitfalls
It is my firm belief that all successful languages are grown and not merely designed from first principles.

Bjarne Stroustrup, The Design and Evolution of C++

The previous chapter covered dynamic programming with Groovy, where the behavior of a type or even an individual object can change while the program is executing. You don’t always need the behavior to vary that dynamically though—sometimes you want only to be able to apply common patterns in an expressive and efficient manner once and for all when the class is compiled.

This chapter covers compile-time metaprogramming or AST transformations. You’ll learn a bit about the concept and its importance. Then we’ll explore most of the transformations Groovy ships with, such as @ToString, @EqualsAndHashCode, and @Lazy, and show you how these keep your code lean and clean. Next, we’ll dive into more details about how AST transformations work and ways to create AST data structures. Then you’ll write your own local and global AST transformations before we look at tools available for viewing and testing AST data structures. As a final step, you’ll see some of the common mistakes and limitations encountered with compile-time metaprogramming.

9.1. A BRIEF HISTORY
The term compile-time metaprogramming has only recently entered the vocabulary of mainstream Groovy developers and some of the more daring Java developers. But Java has had a long history of code generation: tools and frameworks that automatically create code in the hopes of reducing development time. In the good-old days, when CORBA services were the standard remoting technology, it was common to have Java source code automatically generated as part of your build process. More modern applications still do similar things. The common wsdl2java and wsimport applications read WSDL (Web Service Definition Language) interface documents and produce source code for projects using web services. This approach is so common that Maven even has a convention for dealing with the files: put them all in a folder called generated.

9.1.1. Generating bytecode, not source code
The technologies listed so far share a common trait: they all generate source code as part of the build process. Like many other modern languages, Groovy takes a different approach to code generation. Instead of writing out source code that the standard compiler can later read and convert to bytecode, Groovy lets you, the programmer, get involved in the compilation process.

How are getters and setters generated?
In Groovy there’s no need to write getters and setters for fields: they’ll be generated for you. This occurs without a separate source-code file listing these getters and setters hidden on the disk somewhere. The Groovy compiler is smart enough to just read your source and write out the correct class definition in the .class file. These changes are all visible from Java or other languages calling your code, because they’re part of the compilation process. As far as anything looking at the class is concerned, the getters and setters exist as if they’d been handwritten.

From the very beginning, Groovy has made life easier for programmers by manipulating what gets written into the final JVM .class file. The difficulty was that if you wanted a new feature in the language, then you needed to download the Groovy source code and write the feature yourself. But this all changed with the 1.6 release and has evolved further in later releases.

9.1.2. Putting the power of code generation in the hands of developers
AST transformations have been part of Groovy since version 1.6. The AST part of this is an abstract syntax tree—a representation of code as data. This feature allows you to modify the code being generated without ever needing a source-code representation. For example, you can add new methods and fields to a class, or add code into the method bodies. Although no source code is generated, the bytecode is present in the final class file in an entirely ordinary way. This is important because it means Java objects calling your Groovy objects will see the new code, which isn’t the case for changes made through runtime metaprogramming.

Compile-time metaprogramming is an exciting area of the language. There are many new libraries and frameworks for Groovy that generate verbose, boilerplate code directly into the .class files instead of forcing all the users to write extra source code. Code generation is no longer limited to those brave developers willing to download and build the source code for the Groovy compiler: it’s available to anyone using Groovy. If you have a great idea for a new language feature then it’s possible to write it today as a library. This powerful technique creates a living language, where you’re allowed to extend the language in the direction best suited to your project. Many of Groovy’s features are implemented on top of the AST transformation framework. For example, the @Delegate, @Immutable, and @Log annotations all hook into the compiler and affect the final .class file. @Bindable is the secret to UI property binding in the Griffon framework (or writing Groovy Swing in general). The Spock and GContracts libraries both leverage AST transforms, providing useful and productivity-boosting results. Compile-time metaprogramming is used by these libraries to produce more readable tests and more correct runtime behavior.

Before you start writing your own transformations, we’ll look at a few annotations that ship with Groovy, so you can get a feel for what’s possible. It’s worth bearing in mind any repetitive coding tasks you’ve recently had to perform; if they sound like the kind of work that these annotations help with, you may well be able to eliminate them soon.

9.2. MAKING GROOVY CLEANER AND LEANER
Groovy ships with many AST transformations that you can use today to get rid of those annoying bits of repetitive code in your classes. When applied properly, the annotations described here make your code less verbose, so that the bulk of the code expresses meaningful business logic to the reader instead of meaningful code templates to the compiler. AST transformations cover a wide range of functionality, from generating standard toString() methods, to easing object delegation, to cleaning up Java synchronization constructs, and more. You don’t need to know anything about compilers or Groovy internals before using the annotations described in this section: just annotate a class or method and watch your standard code templates disappear.

For the purposes of this section, we’ve divided the existing AST transformations into six categories:

Code-generation transformations
Class design and design pattern annotations
Logging improvements
Declarative concurrency
Easier cloning and externalizing
Scripting support
Let’s start by looking at some annotations that write code into your class so that you don’t have to.

9.2.1. Code-generation transformations
AST transformations often focus on automating the repetitive task of writing common methods like equals(Object), hashCode(), and constructors that generate the code for you so that you don’t have to write it yourself. The built-in annotations in this category are @ToString, @EqualsAndHashCode, @TupleConstructor, @Lazy, @IndexedProperty, @InheritConstructors, @Builder, and @Sortable. See also @Newify in section 19.8.

@groovy.transform.ToString
Annotating a class with the @ToString annotation gives that class a standard toString() method. @ToString prints out the class name and, by default, all of the field values, as you can see in the simple example in the following listing.

Listing 9.1. Using @ToString to generate a toString() method
import groovy.transform.ToString

@ToString
class Detective {
    String firstName, lastName
}

def sherlock = new Detective(firstName: 'Sherlock', lastName: 'Holmes')
assert sherlock.toString() == 'Detective(Sherlock, Holmes)'
You can also control the information that toString() displays with various annotation parameters. An example including property names and eliding null values is shown in the following listing.

Listing 9.2. Using @ToString with annotation parameters
import groovy.transform.ToString

@ToString(includeNames = true, ignoreNulls = true)
class Sleuth {
  String firstName, lastName
}

def nancy = new Sleuth(firstName: 'Nancy', lastName: 'Drew')
assert nancy.toString() == 'Sleuth(firstName:Nancy, lastName:Drew)'
nancy.lastName = null
assert nancy.toString() == 'Sleuth(firstName:Nancy)'
Other annotation parameters let you exclude certain properties, include fields, exclude the package name from the class name, include properties from superclasses, and cache the produced value (useful for immutable objects) if you wish. A full description of the available parameters for @ToString appears in appendix E.

You might wonder what the generated toString() method looks like. Remember we said that no source code is produced, so there’s no code to show you directly, but we can show you the equivalent code (and later you’ll see the tools that allow you to do this yourself) for the Sleuth class’s toString() method, which would look something like this:

String toString() {
  def _result = new StringBuilder()
  def $toStringFirst = true
  _result.append('Sleuth(')

  def firstName = InvokerHelper.getProperty(this, 'firstName')
  if (firstName != null) {
    if ($toStringFirst) {
      $toStringFirst = false
    } else {
      _result.append(', ')
    }
    _result.append('firstName:')
    if (firstName.is(this)) {
      _result.append('(this)')
    } else {
      _result.append(InvokerHelper.toString(firstName))
    }
  }

  def lastName = InvokerHelper.getProperty(this, 'lastName')
  // ... ditto of above if statement but for lastName

  _result.append(')')
  return _result.toString()
}
Don’t be too concerned about the details in this equivalent code listing. After our tour of Groovy’s built-in transformations, you’ll get to see more details about such generated code. Most of the code should look similar to what you might write by hand, but for the curious, we’ll point out that there are some calls of InvokerHelper utility methods that you can safely ignore if you haven’t come across them before. They ensure that the property values printed will be correct even if dynamic changes have been made, and these values will be output using Groovy’s standard formatting mechanisms.

That’s it for the @ToString transform. Next we’ll look at another boilerplate-saving transformation for some of the other methods from Java’s Object class.

@groovy.transform.EqualsAndHashCode
Implementing the equals() and hashCode() methods correctly is repetitive and error-prone. Luckily the @EqualsAndHashCode annotation does it for you. The generated equals() method obeys the contract of Object.equals(), and the generated hashCode() produces an appropriate hash value using a standard algorithm factoring in the constituent fields. The following listing shows using @EqualsAndHashCode in action on an Actor class.

Listing 9.3. @EqualsAndHashCode generates equals() and hashCode() methods
import groovy.transform.EqualsAndHashCode

@EqualsAndHashCode
class Actor {
    String firstName, lastName
}
def magneto = new Actor(firstName:'Ian', lastName: 'McKellen')
def gandalf = new Actor(firstName:'Ian', lastName: 'McKellen')
assert magneto == gandalf
You can customize the equals() and hashCode() methods created using annotation parameters. With these, you can easily exclude certain properties from the calculation, include fields in the calculation, or cache the calculated values (appropriate if you have an immutable class). A full description of the available parameters for @EqualsAndHashCode appears in appendix E.

@groovy.transform.TupleConstructor
Groovy has a flexible syntax for creating objects, such as named arguments and with blocks. But sometimes you want the object constructor to take all of the fields explicitly, especially when you’re creating the Groovy object from Java code. The @TupleConstructor annotation adds this constructor onto the object, as you can see in the following listing.

Listing 9.4. Using @TupleConstructor to generate Java-style constructors
import groovy.transform.TupleConstructor

@TupleConstructor
class Athlete {
    String firstName, lastName
}
def a1 = new Athlete('Michael', 'Jordan')
def a2 = new Athlete('Michael')
assert a1.firstName == a2.firstName
By default, the overloaded constructors use the declaration order of the properties to determine the order of the parameters (and if the includeFields annotation parameter is enabled, then the fields will follow the properties, again in declaration order). In addition, each constructor argument is defined with Java’s default value for the argument type, allowing you to leave off parameters from the right if you plan to set those values later or you’re coding a scenario where the default suffices. There are numerous other ways to fine-tune the exact behavior in a very flexible way using annotation parameters. These parameters let you include or exclude certain properties, include fields, or interact with the superclass properties in various ways. Appendix E provides a full explanation of the available annotation parameters.

The @ToString, @EqualsAndHashCode, and @TupleConstructor annotations are so useful that in many cases you may want to use all three annotations together. In section 9.2.2 we’ll review @Canonical and @Immutable, which allow you to do just that. But first, let’s look at other very handy boilerplate-saving annotations.

@groovy.transform.Lazy
Lazy instantiation is a common idiom in Java. If a field is expensive to create, such as a database connection, then the field is initialized to null, and the actual connection is created only the first time that field is used. Typical in this idiom is a null check and instantiation within a getter method. But not only is this boilerplate code, there are numerous tricky scenarios, such as correctly handling creation in a multithreaded environment, which are error-prone. The @Lazy field annotation correctly delays field instantiation until the time when that field is first used and correctly handles numerous tricky special cases. An example illustrating this concept is shown in the following listing.

Listing 9.5. Using @Lazy to delay property instantiation


We first create a Resource class with inbuilt instance-tracking counters . This class and the counters are simply to help us understand what @Lazy is doing for us. In practice, this would be your database (or other expensive) resource. We now declare a ResourceMain class with four Resource properties. Normally you might have just one resource property but we want to illustrate available options.

The first property, res1 , is a normal Groovy property. The backing field for that property will point to an eagerly created Resource instance created when the ResourceMain instance is created. The second property, res2 , is lazily created. When the ResourceMain instance is created, the backing field for res2 will remain null. In the getter method for res2, a check is made to see if the backing field is null, and if so, a new Resource is created.

For the third property, res3 , we indicate that we want a static singleton Resource. The static modifier on the field denotes this case and the compiler knows to adopt the thread-safe lazy initialization holder class idiom.[1] This property also illustrates special syntax supported by the @Lazy annotation. If you have complex initialization logic where you can’t use the normal syntax for defining an initial value (for example, you might need try-catch logic), then you declare that logic as if it were a Closure but follow it by a matching pair of round brackets, as if you were following Groovy’s normal convention for calling a closure. We didn’t strictly need anything complex for our example but it does illustrate the idea.

1 Described in item 71 of Effective Java, 2nd ed., by Joshua Bloch (Addison-Wesley, 2008).

For an efficient thread-safe lazy instance field in Java, we might be tempted to use the double-checked locking idiom.[2] A correct implementation of double-checked locking would have a volatile private field and a synchronized getter. We use the volatile modifier to indicate this case as shown for res4 . Groovy automatically provides a correct implementation of the idiom. There are two other salient points to note about the declaration for res4.

2 Explained in section 16.2 of Java Concurrency in Practice, by Brian Goetz et al. (Addison-Wesley, 2006).

First, no initial value is given. In this case, the default no-arg constructor for the property’s type will be called. If the type of the field is abstract or it doesn’t have a no-arg constructor, you’ll receive a runtime error. For clarity, and to avoid a missing constructor exception, you might consider always supplying an initial value.

Second, we made use of the optional soft parameter annotation. This determines if the field should be a SoftReference, and therefore eligible for garbage collection. By default, the field isn’t a soft reference. A typical use case for making a soft reference would be if a resource was only rarely used but consumed significant memory. Allowing it to be garbage collected and recreated if needed again might be a prudent use of the memory footprint. We’d recommend avoiding premature optimization and only using this option if performance tests indicated a memory issue.

The remainder of the example checks Resource statistics to confirm the expected behavior. We first check  that creating our ResourceMain instance causes only one Resource to be created (the eager one). After attempting to use all of our resources, we note they’re all created . As a final check, we can see that when using the soft reference, a Resource is returned from the res4 getter  but the internal backing field is a soft reference to the Resource instance . We’re using Groovy’s dump() method to display some of the Resource internals but we don’t need to worry about the details here.

The use of lazy idioms and soft references are advanced topics and the code to do them correctly is notoriously error-prone. Using @Lazy with its optional parameters and modifier keywords is an easy way to leverage the advantages of deferred initialization but also make sure your code is correct.

@groovy.transform.IndexedProperty
Groovy automatically provides getters and setters for properties. This follows Java’s conventions for JavaBeans. What you may not know is that additional JavaBean conventions exist for dealing with array properties. In addition to providing setters and getters for the whole array, there are extra getter and setter methods that take an index value and work on just one member of the array. As you can see in listing 9.6, Groovy doesn’t automatically provide these extra methods because Groovy’s array-like notation is even easier to use . But if you’re creating classes that need to be accessed from Java or by JavaBean-aware tools, then you can use the @IndexedProperty annotation to have these methods added automatically. This works for both arrays and lists (and anything else supporting Groovy’s subscript (getAt/putAt) operator). You can see @IndexedProperty in use in the following listing.

Listing 9.6. Using @IndexedProperty to generate index-based setters and getters


The generated indexed setter and getter can be used to set a specific element  and read it  respectively.

@groovy.transform.InheritConstructors
The @InheritConstructors annotation removes the boilerplate of writing matching constructors for a superclass. Suppose you wanted to write your own custom PrintWriter-like class. The java.io.PrintWriter class has eight constructors, and your subclass should probably provide the same set of creation options. @InheritConstructors to the rescue. The annotation creates matching constructors for every superclass constructor. You can see two of them in use in the following listing.

Listing 9.7. Using @InheritConstructors to automatically generate constructors


The important code as far as this annotation is concerned happens when we use the two constructors, but for completeness, the remainder of the example uses the custom print writers to write content to the two output files that are then compared and deleted.

You can still write your own constructors, of course. If there’s a conflict with a superclass constructor then @InheritConstructors is smart enough to back off and not overwrite your implementation. A word of warning, however: think about your subclass when using this annotation. If your subclass introduces required properties, then it’s often best to make those properties required in a constructor and not implement too many of the superclass constructors. Plus, some Groovy features rely on the availability of a constructor without parameters, so having one on your class is typically a good idea; bear this in mind if you inherit constructors from a class without a no-arg constructor.

You can fine-tune whether annotations on a parent constructor are copied into your constructors using annotation parameters. A full description of the available parameters for @InheritConstructors appears in appendix E.

@groovy.transform.Sortable
The @Sortable annotation, shown in the following listing, removes the boilerplate of writing the implementation code for the methods of the Comparable and Comparator interfaces.

Listing 9.8. Using @Sortable to generate Comparable/Comparator methods


Using @Sortable is easy. Just include the annotation on your class definition. In this example we chose to make sorting based on the last name and, if those are equal, then based on the initial. We used the optional includes annotation parameter  to achieve that behavior. @Sortable adds Comparable<Politician> to the list of implemented interfaces of our Politician class and adds a compareTo method containing the appropriate comparison logic. Calling sort() is enough to invoke that logic. In our example, we then take the politician’s initials as our final result . In addition, @Sortable adds comparators for each of our included properties. Under the covers it creates an appropriate Comparator class containing a compare method but we don’t need to worry about the details because it also provides a method to gain access to singleton instances of those classes. In our example, we access the comparator for the initial property  and use one of the sort methods available for comparators .

In addition to the optional includes annotation parameter, there’s an excludes parameter. This is handy if you have many properties and want to exclude one or two from affecting the sort behavior. Appendix E provides a full explanation of the available annotation parameters.

@groovy.transform.Builder
The @Builder annotation removes the boilerplate of writing instance-building code. Given Groovy’s built-in support for compact instance creation, you might ask why such code is needed. Consider the following Chemist class:

class Chemist {
    String first, last
    int born
}
You can create a new instance easily, like this:

def c = new Chemist(first: "Marie", last: "Curie", born: 1867)
But what if you misspell one of the properties or supply the wrong type? You might have an IDE that’s powerful enough to give you a warning, but otherwise, the first indication you’ll have that something is wrong is when you receive a MissingPropertyException or some kind of CastException at runtime. Similarly, if you need to create instances from Java, Groovy’s conventions don’t make things easier for you. In these scenarios, you might find the @Builder annotation exactly meets your needs. Here’s an example of the @Builder annotation in use.

Listing 9.9. Using @Builder to make building classes easier


We access a builder  and use it to create our Chemist instance . Each of the methods has a typed parameter that allows better Java integration and the potential for increased IDE completion and checking.

Because one size doesn’t fit all when building, the @Builder annotation allows the building process to be customized by supplying alternative strategy classes. Groovy comes with four built-in strategies (in the groovy.transform.builder package) to cover some fairly common scenarios, but feel free to provide your own if you have different requirements. Table 9.1 summarizes the built-in @Builder strategies. If, like in listing 9.9, no strategy is specified, the DefaultStrategy is used. Consult the GroovyDoc (http://docs.groovy-lang.org/latest/html/gapi/groovy/transform/builder/Builder.html) for these annotations for more details.

Table 9.1. Built-in @Builder strategies
Strategy

Description

DefaultStrategy	Creates a nested helper class for instance creation. Each method in the helper class returns the helper until finally a build() method is called, which returns a created instance.
SimpleStrategy	Creates chainable setters, where each setter returns the object itself after updating the appropriate property.
ExternalStrategy	Allows you to annotate an explicit builder class while leaving some builder class being built untouched. This is appropriate when you want to create a builder for a class you don’t have control over such as from a library or another team in your organization.
InitializerStrategy	Creates a nested helper class for instance creation that when used with @CompileStatic allows type-safe object creation. Compatible with @Immutable.
The @Builder annotation provides numerous annotation parameters to allow further customization. In addition to specifying the strategy, you can include or exclude properties as well as rename various generated methods or classes. Note that not all strategies support all annotation parameters; consult the GroovyDoc for each strategy for further details. Appendix E provides a full explanation of the available annotation parameters.

That’s the last of the code-generation annotations in Groovy. Next up on the tour are annotations that help you maintain a better designed and more object-oriented system.

9.2.2. Class design and design pattern annotations
Some transformations focus on implementing common design patterns or best-practice idioms. The goal is to make the right design decisions also the easiest ones to implement. The annotations in this category are @Canonical, @Immutable, @Delegate, @Singleton, @Memoized, and @TailRecursive, and their goal is to bring clarity of design intent and correctness of implementation when using these design patterns. See also @Category in section 8.4.7.

@groovy.transform.Canonical
@ToString, @EqualsAndHashCode, and @TupleConstructor are commonly used together to create standard, or canonical, objects. Groovy provides @Canonical to make this a little easier. @Canonical is the combination of all three of these transformations. As you can see in the following listing, a canonical object has tuple constructors, equals() and hashCode() implementations, and a standard toString() representation.

Listing 9.10. @Canonical generates equals(), hashCode(), toString(), and constructors


The @Canonical annotation takes optional parameters to include or exclude properties from the constructor and method implementations that it creates. If you wish to have more fine-grained control over the transformation’s behavior, you can override its defaults by using one of the constituent annotations in conjunction with @Canonical. If you want to use @Canonical but customize the @ToString behavior, then annotate the class with both @Canonical and @ToString. The @ToString definition and parameters takes precedence over @Canonical. And just what exactly is a sensible default? A complete listing of the default values for @Canonical is given in appendix E.

@groovy.transform.Immutable
Immutable types (such as String) permit no changes in state: when an instance has been created it can never be altered. The main advantage of immutability is that the object is side-effect free and thread safe. There’s almost no way to change an immutable object from within a method or any way to abuse an immutable object across threads (without resorting to using reflection, that is). Also, there’s never a need to make a defensive copy of an immutable object, or worry about what other objects may have references to your internal state. Working with immutable objects is highly recommended on the Java platform. Groovy provides the groovy.transform.Immutable transformation to help you easily create immutable objects, as shown in the following listing.

Listing 9.11. Using @Immutable to mark fields final and suppress setter methods


The Genius class has quite a lot of generated code. It’s absent from the source but we can see it once we start using the class:

A Map-based constructor 
A tuple constructor 
A getter for each property, for example 
An appropriate toString method 
Appropriate equals and hashCode methods 
The @Immutable annotation is very intelligent about which properties to handle. All properties in an @Immutable class must also be marked Immutable, or be of a known immutable type such as a primitive type, String, Color, or URI. Known “effectively immutable” fields are also handled. Dates, arrays, and other cloneable objects are defensively copied in the constructor and getters so that state cannot be changed, and List, Map, and Collection classes are converted to Immutable objects in the constructor. Also, the @Immutable annotation shares similar behavior to @ToString and @EqualsAndHashCode: your class receives a nicely formatted toString() method and correct equals(Object) and hashCode() implementations. @Immutable uses sensible defaults for generating the toString(), equals(Object), and hashCode() methods, and they are fully described in appendix E.

Warning!
Earlier versions of the Groovy codebase contained two @Immutable annotations: groovy.lang.Immutable and groovy.transform.Immutable. The one in the groovy.lang package is deprecated. Please only use the new one in groovy.transform.

@groovy.lang.Delegate
In Java, one of easiest ways to reuse existing code is with a parent class, but just because it’s easy doesn’t mean that it’s the best approach. If you genuinely have a pure is-a relationship between two classes, then inheritance might be appropriate, but in many cases when constructing the new class you want to make modifications to the behavior of some of the methods. In those cases, consider delegation. A delegate is a has-a relationship between two classes. Typically, one class will contain a reference to another and then also share some of the API with that class. The following example might illustrate this better. First, let’s look at how we might be tempted to use inheritance to create a NoisySet class that prints some output whenever an item is added to the set. A naïve attempt might assume that a NoisySet is-a HashSet and might look something like this:

class NoisySet extends HashSet {
    @Override
    boolean add(item) {
        println "adding $item"
        super.add(item)
    }

    @Override
    boolean addAll(Collection items) {
        items.each { println "adding $it" }
        super.addAll(items)
    }
}
This approach is broken. Any items added using addAll will be printed out twice because under the covers addAll calls add within HashSet. For this simple case, we could remove the println statements in addAll but it would be a brittle solution—if the HashSet implementation changed in the future, we might no longer be printing out all the items! The solution is to use delegation. It’s a well-known and relatively straightforward design pattern but involves quite a bit of boilerplate code. For our example, it would look something like this:

class NoisySet implements Set {
    private Set delegate = new HashSet()

    @Override
    boolean add(item) {
        println "adding $item"
        delegate.add(item)
    }

    @Override
    boolean addAll(Collection items) {
        items.each { println "adding $it" }
        delegate.addAll(items)
    }

    @Override
    boolean isEmpty() {
        return delegate.isEmpty()
    }

    @Override
    boolean contains(Object o) {
        return delegate.contains(o)
    }

    // ... ditto for size, iterator, toArray, remove,
    // containsAll, retainAll, removeAll, clear ...
}
In this approach, our NoisySet has-a HashSet. We simply define a private delegate field. Then for each method in HashSet (or more accurately for each method in the Set interface) we provide an implementation that calls through to the delegate. The add and addAll methods will also contain the println statements required by our noisy set.

What is wrong with this implementation? Strictly speaking nothing, but it does suffer from the problem that if the base class ever changed, we’d need to add delegate methods. Also, the intent isn’t very clear. Intermixed with the two methods we actually changed are another 10 boilerplate methods that increase the maintenance footprint of the class and provide noise when trying to understand what the class is doing. Okay, we’re implementing a noisy set but we want its design to be noise free! Let’s consider an alternative implementation of NoisySet using the @Delegate annotation as shown in the following listing.

Listing 9.12. Using @Delegate
class NoisySet {
  @Delegate
  Set delegate = new HashSet()

  @Override
  boolean add(item) {
    println "adding $item"
    delegate.add(item)
  }

  @Override
  boolean addAll(Collection items) {
    items.each { println "adding $it" }
    delegate.addAll(items)
  }
}

Set ns = new NoisySet()
ns.add(1)
ns.addAll([2, 3])
assert ns.size() == 3
This example has a much clearer intent. We’re using delegation but changing two of the methods. The @Delegate transformation adds all of the public instance methods from the delegate onto your class, and automatically calls the delegate when those methods are invoked. This is how NoisySet can implement Set yet only declare two methods instead of every method on the Set interface. By default, the owning class is also made to implement all of the interfaces defined by the delegate as well, so you don’t even need to explicitly implement the Set interface if you don’t want.

There could be a conflict between the owner class and one of the delegate methods, or if multiple delegates are in use, between two methods with the same signature coming from different delegates. In that case, the first existing method is used. The delegate fields are processed in the order they appear in the class. If any method signature matches an existing signature, that method is skipped. If that isn’t what you require, you can fine-tune the behavior using annotation parameters that are described in appendix E.

@groovy.lang.Singleton
The Singleton pattern is intended to ensure that only one instance of a class exists within your system at a time. A standard way to achieve this behavior is for a class to have a private static reference to this instance, a private constructor so that it cannot be instantiated outside the class, and a public static method to access the single instance. If you were to manually implement a singleton in Groovy it would look something like this:

class Zeus {
    static final Zeus instance = new Zeus()
    private Zeus() { }
}

assert Zeus.instance
This isn’t too much code to write, particularly as the code generation in Groovy already supplies the accessor method, but it can be simplified using the @Singleton annotation. The obvious advantage is less code, but it also means that invoking the private constructor results in an exception, as shown in the following listing.

Listing 9.13. Using @Singleton to enforce a single instance of an object
import static groovy.test.GroovyAssert.shouldFail

@Singleton class Zeus { }

assert Zeus.instance
def ex = shouldFail(RuntimeException) { new Zeus() }
assert ex.message ==
    "Can't instantiate singleton Zeus. Use Zeus.instance"
An additional advantage is that you can use the @Lazy annotation parameter to properly generate a lazily instantiated instance, which marks the instance variable as volatile and correctly performs double-checked locking in the instantiation method. Appendix E describes the available annotation parameters for @Singleton.

Note
The Singleton pattern can be useful, but it’s considered by some to be an anti-pattern. Certainly in Java, a singleton offers no layers of abstraction: it’s a concrete type and cannot be extended or easily mocked or changed. The story is a little better in Groovy but there are often better approaches. Also, when using singletons, improper serialization or multiple classloaders can result in two instances of the singleton object, and there are also thread safety complications. Singletons are useful, but be aware of the downsides.

@groovy.transform.Memoized
For pure functional methods that always return the same result given the same inputs it can be efficient to cache the results, especially if calculating the result is quite complex or time-consuming. You could do this manually yourself but the groovy.transform.Memoized transformation can do it automatically for you, as shown in the following listing.

Listing 9.14. Using @Memoized to cache method results


All you need to do is annotate the method or methods you want to enable. In our case that’s the sum method . To show that caching is actually taking place we’ll also log the parameters each time a calculation occurs . We’ll call the sum with the parameters 3 and 4 twice ,  but those parameters will appear in the log only once .

The annotation has a few parameters for tweaking the caching behavior. See appendix E for details. Also, remember that if it’s a closure you want to memoize and not a class method, then see Closure’s memoize method discussed in chapter 5.

@groovy.transform.TailRecursive
When implementing algorithms in a functional style, recursion is often used in preference to imperative loops. If you were implementing a utility class with a function that returns the items from a list in reverse order (and ignoring that such a function already exists), you might use code such as this:

class ListUtil {
    static List reverse(List list) {
        if (list.isEmpty()) list
        else reverse(list.tail()) + list.head()
    }
}

assert ListUtil.reverse(['a', 'b', 'c']) == ['c', 'b', 'a']
This code works as it should and is reasonably elegant, though it could possibly be slower than its imperative equivalent. (We value correctness and clarity ahead of speed but sometimes need a little bit of speed too.) More importantly, for large lists, the code is subject to a stack overflow error. Some languages might try to automatically optimize such code to make it as fast as equivalent imperative code or to unravel the recursion to potentially avoid the stack overflow problem. Current versions of Groovy don’t promise such optimizations but do put you in control of optimizing a subset of recursive functions known as tail-recursive functions.

The previous example, while recursive, wasn’t tail-recursive. When calling back to itself, the last thing a tail-recursive function must do is call itself and nothing else. In our reverse code we reverse the tail (recursively) but then append the head. So, the first thing we need to do is rewrite the code to be tail-recursive. For our case, we introduce an additional parameter that stores the reversed list so far. Once we have a tail-recursive function we can then add the @TailRecursive annotation. Groovy will unravel the code and replace it with equivalent iterative code. Let’s look at this in more detail in the following listing.

Listing 9.15. Using @TailRecursive to optimize tail calls


Note
Before Groovy 2.3 introduced the @TailRecursive annotation, the way to avoid stack overflow when using recursion with closures was to use the Closure.trampoline() method (available since Groovy 1.8). This method wraps the closure into a TrampolineClosure, which, instead of doing a recursive call to the closure, returns a new closure, which is called during the next step of the computation. This turns a recursive execution into a sequential one, thus helping avoiding the stack overflow, albeit at some performance cost.

That’s the end of the discussion of the class design and design pattern annotations. Before moving on to concurrency and scripting annotations, let’s see some of the annotation-based logging improvements in Groovy.

9.2.3. Logging improvements
There’s still a surprising amount of debate about the best way of logging errors and informative messages from Java, and new logging frameworks are still in development. The @Log family of annotations exists to simplify correct logging idioms from Groovy code. The family includes @Log, @Log4j, @Log4j2, @Slf4j, and @Commons.

The annotation does more than just create a logger for you. To understand the power of @Log, consider the following listing and ask yourself if the runLongDatabaseQuery() method will be executed.

Listing 9.16. Using @Log to inject a Logger object into an object


From a Java background, the obvious answer would be yes, the runLongDatabaseQuery() method will be executed, because in Java, method arguments are always evaluated before the method is called. There’s no way to avoid this. In Groovy, the answer is maybe: it depends on whether the FINE log level is enabled.

The @Log annotation first creates a logger based on the name of your class. It then wraps any logging method with a conditional checking whether that level is enabled before trying to execute the logging line. The result is equivalent to wrapping the logging call in a if (logger.isEnabled(LogLevel.FINE)) condition. The arguments to the method may never be evaluated depending on the logging configuration. The transformation is smart too; no check is made if the parameter is a constant such as a simple string or integer. This improves the readability significantly—there’s no more need to include manual checks everywhere for the sake of performance. Groovy does the correct thing by default.

The @Log family of annotations takes one optional parameter: the name of the log variable. By default the log variable is called log, but you can change it to whatever you want. If you don’t like how Groovy initializes the Logger object based on the current class name, then add your own logger field and update the annotation to refer to the field name. Five major logging frameworks are covered by Groovy, and each has its own annotation. The five annotations are detailed in table 9.2 (they’re all in the groovy.util.logging package).

Table 9.2. Five @Log annotations
Name

Description

@Log	Injects a static final java.util.logging.Logger into your class and initializes it using Logger.getLogger(class.name).
@Commons	Injects an Apache Commons logger as a static final org.apache.commons.logging.Log into your class and initializes it using LogFactory.getLog(class).
@Log4j	Injects a Log4j logger as a static final org.apache.log4j.Logger into your class and initializes it using Logger.getLogger(class).
@Log4j2	Injects a Log4j2 logger as a static final org.apache.log4j.Logger into your class and initializes it using Logger.getLogger(class).
@Slf4j	Injects an Slf4j logger as a static final org.slf4j.Logger into your class and initializes it using org.slf4j.LoggerFactory.getLogger(class). The LogBack framework uses SLF4J as the underlying logger, so LogBack users should use @Slf4j.
It doesn’t stop there though, because the @Log feature is extensible. You can use your own company’s logger as well, as long as you implement one interface to define your new annotation. This extension mechanism is how the standard five @Log annotations are implemented, so there are five good examples in the Groovy codebase. To implement the interface you need to define a new Logger object and instantiate it, determine if a method should be wrapped in a conditional check, and then wrap the log call in a guard. Writing the AST for this isn’t hard, but you’ll need to understand the rest of the chapter before tackling the problem.

Next we’ll look at declarative concurrency. Groovy provides annotations to declare how your code is locked during multithreaded access instead of writing the code that performs low-level locking.

9.2.4. Declarative concurrency
Synchronization and access to a mutable state is hard to get right. Proper synchronization can leave your little branch of business logic hidden, surrounded by a forest of lock acquire and lock release code. The concurrency-related annotations aim to remedy this problem: @Synchronized, @WithReadLock, and @WithWriteLock.

@groovy.transform.Synchronized
Code that’s accessed from several threads at once often needs to be synchronized to avoid common concurrency problems. One problem with this is that correct concurrent code is hard: it’s all too easy to introduce one problem when trying to solve another. The easiest solution for Java developers is to add the synchronized keyword to the method declaration. This is another instance where the easiest solution isn’t the best solution.

Avoid low-level synchronization
Java contains many fine primitives for working with concurrent code, such as the synchronized keyword and the contents of the java.util.concurrent package. But these are mostly primitives and not abstractions. The tools are low level and meant to serve as a foundation. GPars is a framework for parallelization that’s built on top of these primitives. It provides many abstractions that shield you from low-level coordination tasks. GPars is described fully in chapter 18.

The problem with method-level synchronization is that it’s very coarse-grained and it’s also part of the public API of the object. You’re effectively locking on a publicly accessible reference: the this reference. Some secure coding standards ban method-level synchronization or synchronization on the this reference because an attacker who has a reference to your object can interfere with your synchronization by synchronizing on it. It’s best to declare a local, private lock and expose that lock to subclasses if classes need to coordinate locking. Doing this correctly is easy with the @Synchronized annotation, as seen in the following listing.

Listing 9.17. Declarative synchronization with @Synchronized


This annotation injects a lock object into your class. The object is a zero-length Object array so that your class remains serializable (which an Object instance isn’t). And any method marked with the annotation has a synchronized block around it but without method synchronization. If you want to limit the scope of your synchronized block, then provide a name for the lock using the default annotation parameter and write the synchronized block yourself when needed, as shown in the following listing.

Listing 9.18. Mixing @Synchronized with custom synchronized block


Synchronization is a low-level, primitive operation. Java has higher-level locking mechanisms as well, and the following two annotations help make them easy to use.

@groovy.transform.WithReadLock and @groovy.transform.WithWriteLock
Java 5 included the java.util.concurrent.locks.ReentrantReadWriteLock class as a tool to use when you need more control over locking than simply using synchronized blocks. A ReentrantReadWriteLock can guard against either read access or write access, where many readers are allowed concurrently, but only one writer is allowed. Although this is a very useful concurrency abstraction, acquiring and releasing a lock correctly is cumbersome, as you can see in this code snippet:

import java.util.concurrent.locks.ReentrantReadWriteLock

class PhoneBook3 {
    private final phoneNumbers = [:]
    final private lock = new ReentrantReadWriteLock()

    def getNumber(key) {
        lock.readLock().lock()
        try {
            phoneNumbers[key]
        } finally {
            lock.readLock().unlock()
        }
    }

    def addNumber(key, value) {
        lock.writeLock().lock()
        try {
            phoneNumbers[key] = value
        } finally {
            lock.writeLock().unlock()
        }
    }
}
Phew, that’s quite a bit of code. It does do the right thing: reading data is guarded with a read lock and writing data is guarded with a write lock. But the code is much simpler when you use the @WithReadLock and @WithWriteLock annotations instead as shown here:

import groovy.transform.*

class PhoneBook3 {
    private final phoneNumbers = [:]

    @WithReadLock
    def getNumber(key) {
        phoneNumbers[key]
    }

    @WithWriteLock
    def addNumber(key, value) {
        phoneNumbers[key] = value
    }
}
This time the logic of the class stands out instead of being drowned in a sea of tryfinally blocks, and you’ll never forget to release a lock. Similar to @Synchronized, these annotations take a parameter for the lock name, and that lock will be used if it exists in the class. You might wonder how to test this class. It isn’t that hard, but if you want to actually ensure that the read and write locks are working correctly, it can be a bit of work. What we’ll do is add some println debugging lines[3] and sleep calls[4] into the preceding example and then start off a bunch of interleaving threads that will read and write phone numbers concurrently. The complete example is shown in the following listing.

3 We don’t recommend using println statements in multithreaded code as a general rule, but we’ll get away with it in this simple example.

4 If we didn’t add some sleep calls, things would happen so quickly that you’ll likely not get any concurrency.

Listing 9.19. Using @WithReadLock and @WithWriteLock for efficient concurrency




The exact output will vary depending on your machine speed and language versions, but you should see something like this:

Reading started for Number2
Reading started for Number3
Reading done for Number2
Reading done for Number3
Writing started for Number3
Writing done for Number3
Reading started for Number4
Reading done for Number4
Writing started for Number4
Writing done for Number4
Reading started for Number5
Reading started for Number6
Reading done for Number6
Reading done for Number5
The important thing should be that multiple reads should be happening concurrently, but when any thread is writing, no other reading or writing should be taking place.

These examples all show how annotations for AST transformations work. There are other ways to be thread-safe as well. You could use an appropriate collection type from the java.util.concurrent package such as CopyOnWriteArrayList or ConcurrentHashMap, or you could use immutable objects or persistent data structures. The value in the Groovy annotation approach is that synchronization and safety are declarative. You don’t explain how the synchronization works, you just declare that it exists and let Groovy do the rest.

In general, declarative solutions offer good abstractions, where you don’t need to see the details and can focus on the more important parts of the code instead of the low-level mechanics. The same is true for other areas where you traditionally end up with a lot of boilerplate code to write. Each individual bit of boilerplate is simple enough, but after you’ve written it enough times you’re bound to make a subtle mistake—and it really impacts the readability of the class. The same idea extends to other operations you might wish to perform on your objects, too.

9.2.5. Easier cloning and externalizing
Implementing Cloneable and Externalizable correctly isn’t always simple. The @AutoClone annotation can give you a reasonable and configurable cloning strategy by adding just the annotation. In a similar vein, @AutoExternalize makes implementing Externalizable simpler by correctly creating default read and write methods.

@groovy.transform.AutoClone
Classes that implement Cloneable should provide a public clone method that creates a copy of the class. At its simplest, the @AutoClone annotation causes your class to implement Cloneable and provides a default and simple clone method implementation. But because one size doesn’t fit all when it comes to cloning, the @AutoClone annotation supports several slightly different styles of cloning. We’ll look at these styles shortly, but let’s first look at the annotation in action as shown in the following listing.

Listing 9.20. @AutoClone provides cloning capability
import groovy.transform.AutoClone

@AutoClone
class Chef1 {
    String name
    List<String> recipes
    Date born
}

def name = 'Heston Blumenthal'
def recipes = ['Snail porridge', 'Bacon & egg ice cream']
def born = Date.parse('yyyy-MM-dd', '1966-05-27')
def c1 = new Chef1(name: name, recipes: recipes, born: born)
def c2 = c1.clone()
assert c2.recipes == recipes
Under the covers, your class will be augmented to look something like this:

class Chef1 implements Cloneable {
    ...
    Chef1 clone() throws CloneNotSupportedException {
        Chef1 _result = (Chef1) super.clone()
        if (recipes instanceof Cloneable) {
            _result.recipes = (List<String>) recipes.clone()
        }
        _result.born = (Date) born.clone()
        return _result
    }
}
The superclass clone() method is invoked, followed by invoking clone() on each Cloneable field or property in the class. If a field or property isn’t Cloneable then it’s simply copied in a bitwise fashion. If some properties don’t support cloning, then a CloneNotSupportedException is thrown. You might wonder about the check for cloning recipes. Its type is List, which isn’t Cloneable though many list implementations including Groovy’s default list type (ArrayList) are, and so in our case recipes will be (shallow) cloned. Deep copies are left to the end user (you) to implement. That was simple but doesn’t cover a range of cloning scenarios. For a wider range of scenarios you need to select the appropriate cloning style. The available options are listed in table 9.3.

Table 9.3. Four @AutoClone styles
Name

Description

CLONE	Adds a clone() method to your class. The clone() method will call super.clone() before calling clone() on each Cloneable property of the class. Doesn’t provide deep cloning. Not suitable if you have final properties. This is the default cloning style if no style attribute is provided.
SIMPLE	Adds a clone() method to your class that calls the no-arg constructor then copies each property calling clone() for each Cloneable property. Handles inheritance hierarchies. Not suitable if you have final properties. Doesn’t provide deep cloning.
COPY_CONSTRUCTOR	Adds a copy constructor, which takes your class as its parameter, and a clone() method to your class. The copy constructor method copies each property calling clone() for each Cloneable property. The clone() method creates a new instance making use of the copy constructor. Suitable if you have final properties. Handles inheritance hierarchies. Doesn’t provide deep cloning.
SERIALIZATION	Adds a clone() method to your class that uses serialization to copy your class. Suitable if your class already implements the Serializable or Externalizable interface. Automatically performs deep cloning. Not as time or memory efficient. Not suitable if you have final properties.
So, using the SIMPLE style, your augmented class will have this form:

class Chef1 implements Cloneable {
    ...
    protected void cloneOrCopyMembers(Chef1 other) {
        other.name = name
        if (recipes instanceof Cloneable) {
            other.recipes = (List<String>) recipes.clone()
        } else {
            other.recipes = recipes
        }
        other.born = (Date) born.clone()
    }

    Chef1 clone() throws CloneNotSupportedException {
        Chef1 _result = new Chef1()
        this.cloneOrCopyMembers(_result)
        return _result
    }
}
And, with the SERIALIZATION style, your class would need to implement Serializable (or Externalizable) and the generated method would look like this:

Object clone() throws CloneNotSupportedException {
    def baos = new ByteArrayOutputStream()
    baos.withObjectOutputStream{ it.writeObject(this) }
    def bais = new ByteArrayInputStream(baos.toByteArray())
    bais.withObjectInputStream(getClass().classLoader){ it.readObject() }
}
Another popular cloning approach is to use the COPY_CONSTRUCTOR style. As shown in table 9.3, it handles both final properties and inheritance hierarchies. The following listing illustrates these features.

Listing 9.21. Using the COPY_CONSTRUCTOR style with @AutoClone
import groovy.transform.*
import static groovy.transform.AutoCloneStyle.*

@TupleConstructor
@AutoClone(style=COPY_CONSTRUCTOR)
class Person {
    final String name
    final Date born
}

@TupleConstructor(includeSuperProperties=true,
        callSuper=true)
@AutoClone(style=COPY_CONSTRUCTOR)
class Chef2 extends Person {
    final List<String> recipes
}

def name = 'Jamie Oliver'
def recipes = ['Lentil Soup', 'Crispy Duck']
def born = Date.parse('yyyy-MM-dd', '1975-05-27')
def c1 = new Chef2(name, born, recipes)
def c2 = c1.clone()
assert c2.name == name
assert c2.born == born
assert c2.recipes == recipes
The added methods generated for the Chef2 class look roughly like this:

protected Chef2(Chef2 other) {
    super(other)
    if (other.recipes instanceof Cloneable) {
        this.recipes = (List<String>) other.recipes.clone()
    } else {
        this.recipes = other.recipes
    }
}

public Chef2 clone() throws CloneNotSupportedException {
    new Chef2(this)
}
You can use several annotation parameters to fine tune @AutoClone, and these parameters are described in appendix E. With these annotation parameters and @AutoClone’s supported styles, many of your cloning scenarios should be covered. But for more complex objects, it’s often best to write your own clone method so that you can have complete control.

@groovy.transform.AutoExternalize
The Externalizable interface is similar to Serializable in that it’s used to persist objects into a binary form. Externalizable was added to the JDK after Serializable. The new interface gives you more control over the persisted form than Serializable does, and it doesn’t use reflection, which at one time was a performance bottleneck. Some performance-sensitive applications prefer using Externalizable.

A class marked @AutoExternalize automatically implements the Externalizable interface, gaining two new method implementations: readExternal(ObjectInput) and writeExternal(ObjectOutput). An example of its usage is in the following listing.

Listing 9.22. Using @AutoExternalize for easier serialization
import groovy.transform.*

@AutoExternalize
@ToString
class Composer {
    String name
    int born
    boolean married
}

def c = new Composer(name: 'Wolfgang Amadeus Mozart',
        born: 1756, married: true)

def baos = new ByteArrayOutputStream()
baos.withObjectOutputStream{ os -> os.writeObject(c) }
def bais = new ByteArrayInputStream(baos.toByteArray())
def loader = getClass().classLoader
def result
bais.withObjectInputStream(loader) {
    result = it.readObject().toString()
}
assert result == 'Composer(Wolfgang Amadeus Mozart, 1756, true)'
The generated methods look something like this:

class Composer implements Externalizable {
    ...
    void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name)
        out.writeInt(born)
        out.writeBoolean(married)
    }

    void readExternal(ObjectInput oin) {
        name = (String) oin.readObject()
        born = oin.readInt()
        married = oin.readBoolean()
    }
}
You can fine-tune the @AutoExternalize behavior using the annotation parameters described in appendix E. Note that if you look into the source code for @AutoExternalize it’s defined as an annotation alias combining the @ExternalizeMethods and @ExternalizeVerifier annotations. This split of functionality into two places can be considered an internal implementation detail. It does, however, allow the two bits of functionality to be run at different compiler phases. You might consider this technique when writing your own AST transformations.

That’s all for the cloning and externalizing annotations. The next set of annotations we’ll discuss exist to make using Groovy as a scripting language safe and secure.

9.2.6. Scripting support
You saw in section 2.3 that scripting is an integral part of the Groovy language. It should come as no surprise that some AST transforms have been developed to make your scripting even more productive. These range from transforms that give you some control over the created script class or its components to ones designed to assist with security and robustness. The transforms that fall into this category include @Field, @BaseScript, @TimedInterrupt, @ThreadInterrupt, and @ConditionalInterrupt. You should also check out the GroovyDoc for @SourceURI, which gives you a hook back to a script’s source.

Security and robustness are important aspects of modern software. Groovy makes it easy to run scripts submitted by your users (as well as your own scripts), but this can be a security hole that needs to be shielded not only against unauthorized access but also against accidental programming errors. No one wants a set of long-running scripts to cause a denial of service. These scripting annotations automatically add safety hooks into scripts so that they time out, respect a thread interrupt, or otherwise behave correctly. They’re designed to be added automatically to scripts executing in GroovyShell or another evaluator, but you can also use them yourself on your own scripts and classes.

@groovy.transform.TimedInterrupt
Annotating a class with @TimedInterrupt sets a maximum time the script or instances of the class are allowed to exist. If the maximum time is exceeded then a TimeoutException is thrown. This annotation is designed to guard against runaway processes, infinite loops, or a maliciously long-running user script.

When annotated, the object instance marks the instantiation time in the constructor. If this instance later detects that the maximum runtime is exceeded then it throws an exception. Checks are made at the beginning of every method call, the first line of every closure, and within every iteration of a for or while loop. If the object sits idle and is never invoked, then no exception is thrown regardless of how much time passes. The following listing is a simple example of its use.

Listing 9.23. Using @TimedInterrupt to guard against slow scripts


Annotation parameters you can use to fine-tune the behavior are described in appendix E.

@groovy.transform.ThreadInterrupt
For timely responsiveness, long-running user scripts should periodically check the Thread.currentThread().isInterrupted() status and throw an InterruptedException when an interrupt is detected. But in practice, scripts are almost never written this way. An easy way to properly respect the interrupted flag is to use the @ThreadInterrupt annotation. When this annotation is present, your script or class will automatically check the isInterrupted() flag and throw an InterruptedException if the thread is interrupted. These checks occur at the start of every method call, at the start of every closure, and within every iteration of a loop. An example is shown in the following listing.

Listing 9.24. Using @ThreadInterrupt to detect interruptions


Similar to @TimedInterrupt, there are some parameters you can use to tweak the behavior of @ThreadInterrupt that are detailed in appendix E.

@groovy.transform.ConditionalInterrupt
The last annotation in the Interrupt family is @ConditionalInterrupt. This annotation allows you to specify your own custom interrupt logic to be woven into a class. Like the others, the interrupt check occurs at the start of every method, the start of every closure, and each loop iteration.

The way you specify the conditional interrupt is within a closure annotation parameter. You can reference any variable that’s in scope within this closure. For scripts, general script variables are in scope, and for classes, instance fields are in scope. The following listing shows a script that executes some work 1,000 times or until 10 exceptions have been thrown, whichever comes sooner.

Listing 9.25. Using @ConditionalInterrupt to set an automatic error threshold
import groovy.transform.ConditionalInterrupt

@ConditionalInterrupt({ count <= 5 })
class BlastOff3 {
    def log = []
    def count = 10

    def countdown() {
        while (count != 0) {
            log << count
            count--
        }
        log << 'ignition'
    }
}

def b = new BlastOff3()
try {
    b.countdown()
} catch (InterruptedException ignore) {
    b.log << 'aborted'
}
assert b.log.join(' ') == '10 9 8 7 6 aborted'
Parameters you can use to tweak the functionality of @ConditionalInterrupt are described in appendix E.

@groovy.transform.Field
Section 7.2 gave details about how Groovy “wraps” script files into a script class and automatically provides main and run methods. The code inside your script ends up being placed inside the run method, which makes it a local variable declaration. Suppose you have a script containing the lines

def x = 4
println x
then the generated script file looks like this:



As you can see, the variable x is a local variable definition within the run method. It wouldn’t be visible to other methods or be available across multiple calls of the run method. For most scripts this is exactly what you want. Annotating a variable with @Field promotes it to a field within your script class. Let’s look at this in action in the following listing.

Listing 9.26. Using @Field for class-level instance variables in a script


The equivalent generated code would look like this:



@groovy.transform.BaseScript
Annotating a script with @BaseScript lets you customize a script’s parent class. Suppose you wanted all your scripts to save all printed lines to a log. You could add boilerplate code into each script to achieve this. But imagine you later wanted to alter your logging approach. Your maintenance burden would be quite high as you’d need to refactor each script. Let’s look at an alternative approach using @BaseScript. For the purposes of this example, we’ll use a very simple logging mechanism; we’ll keep only a list of printed strings.

Listing 9.27. Using @BaseScript to customize a script’s parent class
@BaseScript(LoggingScript)
import groovy.transform.BaseScript

abstract class LoggingScript extends Script {
  def log = []
  void println(args) {
    log << args
    System.out.println args
  }
}

println 'hello'
println 3 * 5

assert log.join(' ') == 'hello 15'
For this example, we placed the LoggingScript base class right into our script, but obviously to share this across scripts you’d normally want to place that into its own separate source file. Now, our generated script will extend from LoggingScript instead of the normal Script class and so it will contain the log field and augmented println method. You can see another example of @BaseScript in section 19.2.2.

9.2.7. More transformations
There are other transformations as well, but they’re covered elsewhere in the book. @PackageScope is discussed in chapter 7 and @Category and @Mixin are covered in chapter 8. See chapter 11 for a discussion of @Bindable, @Vetoable, and @ListenerList. @Newify is covered in section 19.8.

That’s the end of our tour of the AST transformations that come with Groovy. You can use these annotations today without knowing much more. But you don’t have to be satisfied with just what Groovy gives you. You’re free to write your own annotations as well. The rest of this chapter delves into the task of implementing your own annotations using AST transformations. We’re going to discuss local and global transformations, writing your own AST, testing your work, and the known limitations.

So why exactly would you want to write your own AST transformation? There are some good reasons to use compile-time metaprogramming instead of runtime. If you want Java to see the changes you make to a Groovy class, then use an AST transformation to write your changes directly into the produced class file. The code-generation transformations are good examples of doing this. If you need to avoid evaluating a method parameter before a method is invoked, then use an AST transform to avoid or wrap the call, as the @Log transformation does. And as we’ll see later, you may also find compile-time metaprogramming a good fit for advanced or fine-grained control over DSLs. Lastly, if you want to do something wildly different, like change the semantics of the language, then your best approach is an AST transform. Let’s get into deeper explanations and in-depth examples.

Throughout this chapter, you’ve heard the term abstract syntax tree several times, usually abbreviated as AST, but we haven’t looked at what it really means. It’s time to take a deep dive into AST and the Groovy compiler.

9.3. EXPLORING AST
To use AST transformations, you don’t necessarily need to understand all the details of their inner workings, but to write your own transformations, it’s important to have at least a little bit of knowledge about how the compiler works and which data structures it uses. In this section you’ll learn about ASTs, see some of the AST visualization tools available for the platform, and understand the basics of the Groovy compiler.

An AST is a representation of your program in tree form. The tree has nodes that can have leaves and branches, and there’s a single root node. Many compilers, not just Groovy, create an AST as a step toward a compiled program. In general and simplified terms, running a Groovy script is a multistep process, as shown in figure 9.1.

Figure 9.1. General process of compiling and running a Groovy script


First, the Groovy compiler reads the source file and checks it for basic forms of validity. Then the source code is converted into an AST, which is eventually converted to bytecode. Finally, the JVM loads the class and executes it. The AST is where all of the interesting language stuff happens. For example, adding getters and setters for properties happens in AST, and giving a script a main() method happens there. If you want to write a language feature, then you’ll quite possibly be working with the AST. Let’s look at simple AST examples to help you understand it better. Figure 9.2 shows the tree representing the expression 1 + 1, the simplest nontrivial example.

Figure 9.2. An AST for the expression 1 + 1


The plus operation is a binary one: it has two operands, a left and a right. When this program is executed, a plus operation is executed and (hopefully) the result is 2. Figure 9.3 shows a slightly more advanced example: the expression 1 + 2 + 3.

Figure 9.3. An AST for the expression 1 + 2 + 3


It’s no accident that the branch 1 + 2 forms the leftmost branch. Addition is associated left to right, and to compute the answer to 1 + 2 + 3 you must first compute 1 + 2. Only then will you be able to evaluate 3 + 3 and see the result as 6. Figure 9.4 shows a more realistic example, the groovy script assert 1 + 1 == 2.

Figure 9.4. An AST for the expression assert 1 + 1 == 2


In Groovy terms, this script creates an AssertStatement, which has a BooleanExpression, which in turn has a BinaryExpression. It’s this BinaryExpression that holds the == equals operator. Entire programs are easily represented in tree form, and the tree can be analyzed, navigated, and transformed as part of the compilation.

Each language (or compiler, really) has its own tree structure, and it’s up to the AST implementors to determine the exact structure. In Groovy, each element of the tree is an instance of the class ASTNode, and there’s a subclass for everything in the language: BooleanExpression, ForStatement, WhileStatement, ClosureExpression, to name a few. There are over 75 subclasses of ASTNode, and having a good IDE to help you navigate the class hierarchy is highly recommended. Point your IDE to the Groovy sources and you should be fine; some IDEs will automatically download them for you.

Homogeneous versus heterogeneous AST
Heterogeneous AST is the term for having one subclass of ASTNode for each language element. The tree is populated with many different types, and analyzing the tree means reading the type of the tree leaves, not just reading the leaf properties. The javac compiler from Oracle also uses a heterogeneous AST. The advantage is that it’s easy to store and retrieve node-specific data from the AST leaves. Other languages use a homogeneous AST, where every node in the tree is the same type. Imagine if all the objects in the entire tree were of the concrete type ASTNode. The main advantage in this approach is that tree visitors are trivial to write, but static analysis is more difficult. The book Language Implementation Patterns by Terence Parr (Pragmatic Bookshelf, 2010) offers excellent in-depth coverage of ASTs.

9.3.1. Tools of the trade
At this point you no doubt have many questions about Groovy, ASTs, and class files. Groovy compiles to Java class files, so to analyze bytecode of a .class file you can always use the javap application from the JDK. But that’s a very low-level approach, which can be time-consuming and frustrating, particularly if you’re trying to examine code of any significant size. Luckily, there are many tools at your disposal if you’re interested in digging a little deeper into how things work. If you plan on working with compile-time metaprogramming then each of the tools listed here will be an invaluable asset in your toolbox.

Groovy Console’s AST browser and source viewer is GroovyConsole, which is included in the Groovy installation and contains a tool that lets you view and analyze the AST of a Groovy script. Once you have GroovyConsole open, you can analyze any script using the menu item Script> View AST or the Ctrl-T shortcut (Cmd-T on Macs). The window that opens is called the AST browser. The AST browser has three parts: the tree view, the property table, and the decompiled source view, as shown in figure 9.5.

Figure 9.5. Groovy’s standard AST browser showing the AST data structures (top left), the properties (top right), and the equivalent source code (bottom)


The tree view displays the AST of your script using a standard tree widget. You can expand and navigate nodes, and otherwise explore the AST. As you click a tree node, the property table on the right lists all of the properties of the node, and the main GroovyConsole window highlights the source code corresponding to that AST node. The decompiled source view, along the bottom of the window, displays the AST rendered as Groovy source code. The generated source code is perhaps the easiest way to understand what the AST contains because it’s much easier to read source code than a tree component.

Throughout this chapter we’ve shown snippets of equivalent generated source code for several of the transforms. Now would be a great time to try examples within the GroovyConsole’s AST browser and see the equivalent generated code for yourself. You should change the Phase dropdown list between the various phases and refresh the generated code to see how the output changes during the compilation process. Setting the phase to Canonicalization is often useful, because most of the transforms have been invoked by then but other details added later don’t clutter up the class file.

The decompiled source viewer is one of the best features to learn and understand how Groovy works. Even if you’re not attempting to write AST transformations, it can be a real learning experience to see how different pieces of Groovy source code get transformed into the final output.

9.3.2. Other tools
The last important tools are a good decompiler and debugger. A decompiler will reverse-engineer the source code out of a class file, and the results can be quite amazing. Any Java decompiler should work perfectly well with Groovy, and there are many open source and free ones to choose from. JD-GUI is also nice. It’s free for noncommercial use but isn’t open source. Also a good IDE and debugger aid greatly when exploring the large ASTNode class hierarchy. When there are over 75 subclasses to navigate, it’s important to be able to quickly find the source code you need and view the current state of instances. There are several open source and free options available for IDEs with great Groovy support.

You’ve seen a lot already. We’re nearly ready to create our own transformations. But first, we’ll examine a few approaches to writing ASTs, including using the AstBuilder.

9.4. AST BY EXAMPLE: CREATING ASTS
This section examines creating ASTs in more depth, first by building an AST manually, and then using the three different approaches offered by AstBuilder. It’s hard to make a general comparison between the options. Each approach has advantages and disadvantages and should be used in different scenarios. The examples in this section all produce the same AST: a return statement that returns a new instance of java.util.Date. This is the same thing as the source code return new Date(), except that it’s the AST and not the actual source. The examples start at the lowest level possible, working directly with ASTNode instances, and ascend toward a higher level, where you can write code that’s automatically converted into an AST. Let’s see it in action.

9.4.1. Creating by hand
The most basic approach is to directly manipulate and construct the concrete classes. The main disadvantages are verbosity, complexity, and a lack of abstraction. As you can see in the following listing, the code to produce just a return statement can become quite large.

Listing 9.28. Creating AST objects by hand
import org.codehaus.groovy.ast.ClassHelper
import org.codehaus.groovy.ast.expr.*
import org.codehaus.groovy.ast.stmt.ReturnStatement

def ast = new ReturnStatement(
    new ConstructorCallExpression(
        ClassHelper.make(Date),
        ArgumentListExpression.EMPTY_ARGUMENTS
    )
)

assert ast instanceof ReturnStatement
For simple problems that approach suffices. An IDE gives you code completion and some type checking, and should allow you to navigate to the source code of the ASTNode class hierarchy, which helps a lot with the learning process. Also, there are no limitations on the AST you can produce. Any tree whatsoever can be created by directly using the classes like this, which isn’t the case for some of the other approaches. Also, it’s quite easy to merge in information from the calling context.

There are some disadvantages to this approach, and for larger, more real-world examples this technique becomes a burden. First, the code to create an AST quickly becomes large and doesn’t really resemble the source code it’s trying to model. For big examples it’s difficult to read the source code you write and mentally map it into the code it’s meant to produce. Second, you need to manage things like VariableScope and tying the nodes together yourself. For example, many tasks involve two steps, such as creating a MethodNode and then adding it to the parent class. To be effective you’ll need to learn a large part of the API. Third, this approach offers no abstraction layer over the raw AST.

The lack of abstraction can be seen in the example. For example, a ConstructorCallExpression accepts a ClassNode and an Expression as arguments (they’re used as the constructor type and arguments). To write this AST by hand you need to know that an empty argument list is ArgumentListExpression.EMPTY_ARGUMENTS and not null. Also, you need to know that the best type to use for the constructor arguments is an ArgumentListExpression object. You can use other types but they probably aren’t what you intend. Lastly and most importantly, a ClassNode should be made by calling the ClassHelper.make() method.

Creating a ClassNode with ClassHelper
The ClassHelper class contains logic for creating and caching a ClassNode correctly, which isn’t exactly a simple process. If you need a ClassNode object, then always create it through ClassHelper, passing either a Class reference or a String representing a fully qualified class name. The String parameter is useful when you don’t want a compile-time dependency on the target class.

Before looking at alternatives, it’s worth pointing out that there are some utility classes that reduce the burden of using the AST nodes directly. The main class is GeneralUtils in the org.codehaus.groovy.ast.tools package. Using static imports from this class allows you to improve the previous listing to look like the following.

Listing 9.29. Using the GeneralUtils helper class
import org.codehaus.groovy.ast.stmt.ReturnStatement
import static org.codehaus.groovy.ast.ClassHelper.make
import static org.codehaus.groovy.ast.tools.GeneralUtils.*

def ast = returnS(ctorX(make(Date)))
assert ast instanceof ReturnStatement
This is an improvement but this helper class doesn’t cover all of the ASTNode classes and you still need to know most of the implementation details of the ASTNode classes. A good abstraction should allow you to create ASTNode types without knowing all of this low-level information. Luckily, Groovy provides the AstBuilder.

9.4.2. AstBuilder.buildFromSpec
This approach provides a light DSL over the ASTNode class hierarchy. It should look similar to listing 9.28 but it’s slightly cleaner as can be seen in the following listing.

Listing 9.30. Creating AST objects using buildFromSpec
import org.codehaus.groovy.ast.builder.AstBuilder
import org.codehaus.groovy.ast.stmt.ReturnStatement

def ast = new AstBuilder().buildFromSpec {
   returnStatement {
       constructorCall(Date) {
           argumentList {}
       }
   }
}

assert ast[0] instanceof ReturnStatement
The AstBuilder is a convenient shortcut for writing shorter, more concise AST. A shorthand notation exists for every ASTNode type, and much of the API is simplified. For instance, you can work directly with Class objects instead of ClassNode objects, and scopes are largely handled for you. Similar to the by-hand approach, there are no limitations on the AST you create and it’s easy to merge in code and parameters from the surrounding context. The best documentation for buildFromSpec is the unit test, which shows the correct use of every single node type.

AstBuilder.buildFromSpec helps eliminate verbosity and some complexity. It almost matches the flexibility of calling the constructors by hand, suffering only from the fact that passing and referencing Class literals means that Class must be present at compile time (a limitation the manual approach doesn’t share). The buildFromSpec API currently doesn’t allow you to use ClassHelper in all cases—class literals are sometimes required. But the main disadvantage is that the DSL offers little in terms of abstraction. To effectively write a new AST you’ll still need to know a lot about what AST you want to produce. You don’t need to worry about scopes, but you’ll need to know that a ReturnStatement requires an Expression and a ConstructorCallExpression requires an ArgumentListExpression. The next two alternatives offer better abstraction but with some loss in flexibility.

9.4.3. AstBuilder.buildFromString
The AstBuilder object has a buildFromString method that converts Groovy source code into the corresponding AST. By default it compiles the code to the class-generation phase and returns only the AST for the enclosed script, not any classes defined within the script. Of course, both of these behaviors can be changed by passing different arguments to the method. This approach allows you to create an AST without knowing anything about the underlying object hierarchy: at this point we have a genuine abstraction over the AST classes. The following listing shows the buildFromString approach in action.

Listing 9.31. Creating AST objects using buildFromString
import org.codehaus.groovy.ast.builder.AstBuilder
import org.codehaus.groovy.ast.stmt.BlockStatement
import org.codehaus.groovy.ast.stmt.ReturnStatement

def ast = new AstBuilder().buildFromString('new Date()')
assert ast[0] instanceof BlockStatement
assert ast[0].statements[0] instanceof ReturnStatement
The only knowledge required is that a script is a BlockStatement and that BlockStatement has a ReturnStatement in its statement list. As you can see, this is a terse mechanism for AST creation, and the intent of the produced code is clear. This is the preferred approach when accepting and compiling user input, because it can usually be converted into a String.

The main limitation is flexibility. How exactly do you merge in code or variables from the calling context? You need to resort to String concatenation as shown in the following listing, which creates a method returning twice π.

Listing 9.32. Trying to mix dynamic code with buildFromString
import org.codehaus.groovy.ast.builder.AstBuilder
import org.codehaus.groovy.control.CompilePhase
import org.codehaus.groovy.ast.*

def approxPI = 3.14G
def ast = new AstBuilder().buildFromString(
    CompilePhase.CLASS_GENERATION,
    false,
    'static double getTwoPI() { def pi = ' + approxPI + '; pi * 2 }'
)

assert ast[1] instanceof ClassNode
def method = ast[1].methods.find { it.name == 'getTwoPI' }
assert method instanceof MethodNode
That’s a little complicated! Pushing compile-time data, such as the approximate value of π in the preceding example, into the AST requires String concatenation and potentially escaping, and getting the MethodNode requires searching through all the methods defined on the class and pulling it out by name. And we haven’t even looked into how you might access other ASTNode implementation details such as VariableScope objects.

For more advanced examples it might be too difficult to manage this complexity. You can use the buildFromString method for this type of task, but it’s fraught with difficulties. This is an abstraction over ASTNodes that doesn’t easily allow you to dive deeper into the code when the need arises. Finally, synthesizing some types of structural nodes is difficult.

The buildFromString method and the next approach are great for creating method bodies, expressions, or statements. But if you’re dealing with structures like ClassNodes, MethodNodes, or FieldNodes, then it’s easier to use buildFromSpec or create the nodes by hand.

9.4.4. AstBuilder.buildFromCode
The last approach is possibly the most interesting. Using the buildFromCode method you can specify your source code directly as source code, and the builder turns it into AST. This is similar to the buildFromString approach except that the input isn’t a String, it’s just code.

Listing 9.33. Creating AST objects using buildFromCode
import org.codehaus.groovy.ast.builder.AstBuilder
import org.codehaus.groovy.ast.stmt.ReturnStatement

def ast = new AstBuilder().buildFromCode {
  new Date()
}
assert ast[0].statements[0] instanceof ReturnStatement
This is quite simple, and it reads like code because it is code! The advantage is that the Groovy compiler and IDEs will highlight syntax correctly, do code completion, and generally validate your input. But its strength is also its weakness. The main disadvantage is that the Groovy compiler will validate your input. For instance, you cannot declare a new class within a closure body, so declaring a new class (or method) using buildFromCode isn’t allowed. Also, there’s no way to bind in data from the enclosing context. The new Date() expression here is only executed at runtime. The scope at compile time is different from the scope at runtime, so any variables in scope at compile time won’t be available when the code is executed. There’s no way to write the getTwoPI() method using this approach. When it’s appropriate, this is the most elegant solution, and offers the best abstraction level—but there’s a price to pay in flexibility.

There are many different scenarios where ASTs can be useful, and there are several APIs to help you build them. There’s no one right way to create an AST. In general, our advice is the same for most things in life: start simple, stay simple. If you think the AstBuilder simplifies your implementation, then by all means use it. But if you find yourself fighting against it, or spending too much time figuring out how it works, then just go the simple route and write the AST by hand. There’s a lot to learn with compile-time metaprogramming, and your time is probably better spent writing a few more tests than trimming down your AST generation by a few more lines of code.

If the caveats on AstBuilder use leave you feeling a little underwhelmed, stay tuned for Groovy macros that aim to remove some of the limitations of AstBuilder. Groovy macros are scheduled in Groovy’s roadmap for version 2.5.

You’ve seen plenty of built-in AST transforms and you now understand a lot more about the AST data structures. Let’s put your newfound knowledge to use and let you create some of your own AST transforms!

9.5. AST BY EXAMPLE: LOCAL TRANSFORMATIONS
All of the examples presented so far, such as @ToString and @Canonical, are known as local transformations. A local transformation relies on annotations to rewrite Groovy code. There are other forms of transformations as well; however a local transformation has the advantage of being the easiest to write: Groovy takes care of instantiating and invoking your transformation correctly, as well as making sure to avoid calling it when it’s not needed. Features written as local transformations modify the class generated by Groovy and are activated by annotating existing code structures, such as a method or a class.

Let’s start your exploration with a simple example of a local transformation. To demonstrate a local transformation, you’re going to create a method annotation that marks a method as being a Java main method. The transformation will add a main method that can be a public entry point to run the class, and that main method will create an instance of your class and call its annotated method. What you want to end up with is the ability to write code that looks like this:

class Greeter {
    @Main
    def greet() {
        println "Hello from the greet() method!"
    }
}
There’s no point in running this code at this point, because the @Main annotation doesn’t exist yet, but after just a few more steps, you’ll have written this annotation and also a transformation that creates a main method on the Greeter class. This main method will create an instance of the Greeter class and then invoke the greet() method on it. After your transformation runs, the source equivalent of the modified AST tree will be

class Greeter {
  def greet() {
    println "Hello from the greet() method!"
  }

  public static void main(String[] args) {
    new Greeter().greet()
  }
}
and the output from invoking this class as a console application will be

Hello from the greet() method!
From the sample use you can glean some information about the objects involved. You need to define an annotation called @Main, and that must trigger the AST transformation to create the main method. There isn’t much more to it than that. Creating and invoking the object is all done internally by Groovy. Figure 9.6 shows the classes involved with a local AST transformation.

Figure 9.6. Classes involved with the @Main local AST transformation


You could define the @Main annotation using Groovy or as a standard Java annotation; there’s no Groovy magic involved. If you chose Groovy you’d follow normal conventions and place it in a file called Main.groovy. Here’s what it would look like (a full listing is coming shortly):

@Retention(RetentionPolicy.SOURCE)
@Target([ElementType.METHOD])
@GroovyASTTransformationClass(classes = [MainTransformation])
@interface Main {}
The retention policy should be SOURCE, meaning that the compiler doesn’t carry the presence of this annotation through to the final class file. The target element type specifies what the annotation can be applied to—so in this case, you’re going to use METHOD. If you’ve written normal Java annotations before you’ll have come across these concepts.

The use of the GroovyASTTransformationClass annotation is special to the Groovy compiler. It specifies the class (or classes) implementing the logic of the AST transformation, and is how the compiler binds the pieces together. In this case, that’s the MainTransformation class, which you’ll write next. Before seeing all of the details though, let’s have a look at a class skeleton and sketch out what the transformation should do.

Following normal naming conventions, you’d place your code in a file called MainTransformation.groovy and that file would have the following form:

@GroovyASTTransformation(phase = CompilePhase.INSTRUCTION_SELECTION)
class MainTransformation implements ASTTransformation {
  void visit(ASTNode[] astNodes, SourceUnit sourceUnit) {
    // perform any checks
    // construct appropriate main method
    // add main method to class
  }
}
There are two important things to note about this example:

The class is annotated with @GroovyASTTransformation, which informs the Groovy compiler of the phase in which you want the transformation to be invoked. This is a required annotation for a transformation and must be Semantic Analysis or later. It can’t be any earlier than that because your original annotation wouldn’t be loaded at that point. There’s a bit of a chicken and egg problem with trying to go earlier.
Your transformation class must implement the ASTTransformation interface, which has a single method, called visit. The visit method takes two parameters. For simple transformations, you only need to use the ASTNode[] parameter. Element 0 contains the annotation that triggered the transformation and element 1 contains the ASTNode that was annotated.
Your MainTransformation class will be instantiated and invoked by Groovy when the @Main annotation is encountered. The code inside the visit method must accomplish these steps:


1.  Perform any checks.

2.  Find the method that was annotated with @Main (in this case greet()).

3.  Get a reference to the enclosing class (Greeter).

4.  Create a synthetic public static void main method and instantiate the Greeter instance within it.

5.  Invoke the greet() method on the Greeter instance.

6.  Add the new method onto the Greeter class.

There’s one more requirement that we haven’t mentioned until now. Before compiling your Greeter class, compiled versions of the classes for @Main and MainTransformation must be on the classpath. There are several ways to achieve this. If you’re compiling by hand from the command line or using your IDE, ensure that those files are compiled first. If you have a build tool like Gradle you can configure your build file so that compilation of your transformation classes occurs before compilation of classes that use those transformations. For the purposes of this chapter, place everything in the one source file but use a new GroovyShell to compile your Greeter class after everything else has been compiled.

The complete example is shown in the following listing.

Listing 9.34. Implementing the ASTTransformation for the @Main annotation




The visit method is where the interesting action happens. It shows how to create ASTNode objects. You can call constructors directly, or use the GeneralUtils helper class as shown here, or you’re also free to use the AstBuilder as discussed in section 9.4. The code sample does some error checking on the input, because in production code it’s often best to state your assumptions with a few assertions or guard clauses.

It may not be obvious, but there are a few assumptions made even in this small example. The enclosing class must have a no-argument constructor (because you call new Greeter()) that creates the object in a usable state. The example also doesn’t cater to annotating multiple methods, or annotating a static method, or handling an existing main method. You can obviously extend this example in numerous ways if you’re feeling adventurous. Let’s look at a few extensions now.

Listing 9.35. Implementing an enhanced @Main2 transformation




Several edge cases are now handled. We now check for two types of error. First, if the class containing the method being annotated (the Greeter class in this example) doesn’t have a no-arg constructor we stop compilation with an error message. Second, if an existing main method is found we’ll also treat that as an error unless the annotation’s merge attribute has been set to true, in which case we’ll add the calling code to the existing main method. We’ll also cater to static methods annotated with @Main2 calling that static method from inside the main method instead of creating a new instance. Finally, there’s an implementation detail we need to handle. For the case of adding code to an existing main method, we’ll account for main methods containing just a single statement as well as ones that contain a block of statements.

The edge cases of writing transformations can sometimes be challenging. Writing an AST transformation forces you to sit and think about just what could happen in the language, and you’ll come away from the experience with a much better understanding of Groovy.

When you write an AST transformation, you’ll need to decide which compiler phase to target. The choice depends on what you’re trying to do to the AST. A full description of the different compiler phases, as well as some hints for choosing which phase to target, appears in appendix F.

Local transformations require an annotation, which isn’t particularly limiting when you consider that Groovy annotations are more flexible than Java’s. They can appear in more places within a source file than in Java, including import statements. When in doubt choose a local transformation because it’s the easiest to write. You can always refactor to a global transformation or hard-code a transformation into a classloader later.

9.6. AST BY EXAMPLE: GLOBAL TRANSFORMATIONS
Global transformations are similar to local transformations except that no annotation is required to wire-in a visitor. Instead of having the end user specify when your transformation is applied, global transformations are simply applied to every single source unit in the compilation. Global transformations can also be applied to any phase in the compilation, even those before semantic analysis. With this flexibility comes a performance penalty. All compilations will take longer, even if your transformation isn’t used. For this reason you should use global transformations with reticence and consider implementing global transformations in Java or using @CompileStatic for the performance benefits.

Global transformations are specified in JAR file metadata. To deploy a global transformation it must be packaged into a JAR file, and the META-INF metadata must specify the fully qualified path of your transformation class. Let’s see this in action with an example. Imagine a transformation that adds a static method to every class that returns the date and time of the compilation as a String. You could use it from any class or script as follows:

println 'script compiled at: ' + compiledTime
class MyClass { }
println 'script class compiled at: ' + MyClass.compiledTime
Don’t try to run this as is just yet because we haven’t created the global transformation yet. We’ll have a test coming up shortly that will give you a proper chance to see this transform in action. Also, remember that free-standing scripts without classes still get generated into a Script subclass during compilation, so adding a getCompiledTime() method to every Class in the SourceUnit should be enough to accomplish this feature. For this transformation we’re going to add a public static method called getCompiledTime to every class in the SourceUnit, and it will simply return the date of compilation as a String.

In local transformations we manipulated the supplied ASTNode[] to find the context in which we were invoked. For global transformations this array holds little of interest. Instead we need to query the SourceUnit to find our source AST. It contains all the classes that were defined in the file along with the script, which itself is a class of type Script.

The following listing shows the implementation of our global transformation.

Listing 9.36. Adding a new method to a class using a global AST transformation
package regina

import org.codehaus.groovy.ast.*
import org.codehaus.groovy.transform.*
import org.codehaus.groovy.control.*
import org.codehaus.groovy.ast.builder.AstBuilder
import static groovyjarjarasm.asm.Opcodes.*

@GroovyASTTransformation(phase=CompilePhase.CONVERSION)
class CompiledAtASTTransformation implements ASTTransformation {

  private static final compileTime = new Date().toString()

  void visit(ASTNode[] astNodes, SourceUnit sourceUnit) {
    List classes = sourceUnit.ast?.classes
    classes.each { ClassNode clazz ->
      clazz.addMethod(makeMethod())
    }
  }

  MethodNode makeMethod() {
    def ast = new AstBuilder().buildFromSpec {
      method('getCompiledTime', ACC_PUBLIC | ACC_STATIC, String) {
        parameters {}
        exceptions {}
        block {
          returnStatement {
            constant(compileTime)
          }
        }
        annotations {}
      }
    }
    ast[0]
  }
}
For simplicity, the error checking was left out of the example; in a real transformation you’d apply similar guard clauses to the ones we used in the @Main example from listing 9.34. Now all we need to do is tell the compiler about our transformation so it can be applied appropriately.

The first requirement is that the transformation class or classes plus a particularly named metadata configuration file must be on the classpath. A common way to do this is to create a JAR file containing all of the necessary pieces. The JAR file must contain your AST transformation classes as well as a special file named org.codehaus.groovy .transform.ASTTransformation in the META-INF/services directory. By convention, the services directory is the standard place for putting configuration metadata files like the one we need to create. The configuration file is a simple text file, and each line is a fully qualified class name of an AST transformation. So, for our case we need the single-line regina.CompiledAtASTTransformation.

The full contents of our JAR file, including classes and services, can be seen in figure 9.7.

Figure 9.7. Contents of JAR file containing CompiledAtASTTransformation


The name of the JAR file doesn’t matter. As long as it’s on the classpath during compilation, the Groovy compiler will read the configuration file and apply any transformations listed within it. To make things easy, the sample code for this book has a little Gradle build file to generate the JAR for you. Just run gradlew jar at the command line to create the JAR in your build/libs folder. Feel free to create the JAR manually or using your own tool of choice.

Once we have our global transform on our Groovy classpath, we can test it by compiling any class or script. The following listing shows how.

Listing 9.37. Example showing use of getCompiledTime() on a class
package regina

class CompiledAtASTTransformationTest extends GroovyTestCase {
  // matches format: EEE MMM dd HH:mm:ss zzz yyyy
  static DATE_FMT = /\w{3} \w{3} \d\d \d\d:\d\d:\d\d \S{3,9} \d{4}/

  @Override
  protected void setUp() throws Exception {
    super.setUp()
  }

  void testShouldApplyToThisTest() {
    assert compiledTime.toString() =~ DATE_FMT
  }

  void testShouldApplyToScriptAndScriptClasses() {
    assertScript '''
      import static regina.CompiledAtASTTransformationTest.*
      assert compiledTime.toString() =~ DATE_FMT
      class MyClass { }
      assert MyClass.compiledTime.toString() =~ DATE_FMT
    '''
  }
}
The fact that the global transformation will typically be packaged in a JAR file has an effect on your project structure. If you want to use the transformation within your project, then the transformation JAR must be built before the compilation of the rest of your project. For most build tools and IDEs this means creating a separate project for the transformation, possibly along with a separate build script. Again, to make your life simple, the sample source code uses Gradle to make it easy for you to run the preceding test without you having to go to such trouble and without the global transform impacting any other Groovy work you might be doing. Simply run gradlew test from the command line to run the test.

There’s one more feature of global AST transformations that DSL writers find useful. You can specify a file extension to which the Groovy compiler will automatically apply your transformation. The mechanism to define a file extension is similar to defining the transformation. Simply write the file extensions (without any wildcards) into a file called org.codehaus.groovy.source.Extensions and include it in your JAR next to the org.codehaus.groovy.transform.ASTTransformation file. Each line of the file should list a file extension without any sort of wildcards attached. Your final JAR file needs to contain both the ASTTransformation file and the extensions configuration file, plus any required .class files.

Error reporting
Errors can be reported using the addError and addException methods on SourceUnit; however, it’s much better to use an ErrorHandler, which can be retrieved from the SourceUnit with the getErrorHandler() method. This object collects all of the error messages during the compile and has a broader API. There are methods to add an error and fail the build, add an error but continue, add warnings, and more. And please, for the sake of your users, always add error messages that contain a good description of what happened, the conditions that caused the error, and the line number where the error occurred. If your users really wanted cryptic or bizarre compilation failures, they’d be using C++.

There’s a lot to learn with compile-time metaprogramming, and your time is probably better spent writing a few more tests than trimming down your AST generation by a few more lines of code. That brings us to our next topic: testing.

9.7. TESTING AST TRANSFORMATIONS
Test, test, test. It’s hard to overtest an AST transformation because source code can come in a dizzying array of variations, each exposing unique edge cases. Also, during upgrades between Groovy versions you need a good regression test. Luckily, transformations are easy to test. Local transformations are the most testable. Typically, you create a nested class within your test case that contains your annotation and then test against that class. Consider the test for @WithReadLock in the following listing.

Listing 9.38. Possible unit test for the @WithReadLock transformation


This test case asserts that WithReadLock on a method creates a private final instance field called $reentrantlock with type ReentrantReadWriteLock. This is a simple and readable approach, and it works especially well within an IDE where the class can be verified and easily seen in searches. But there are two disadvantages. One, with a lot of tests come a lot of nested classes, and this can clutter up your namespace. Your build will create many, many extra classes that don’t have meaning outside of a specific test method. And two, there’s no way to debug the AST transformation in an IDE because by the time the test runs the class is already compiled. To work around these issues you can use a GroovyClassLoader to compile the class. The same test is presented in the following listing but this time using a GroovyClassLoader.

Listing 9.39. Improved unit test for the @WithReadLock transformation
import java.util.concurrent.locks.ReentrantReadWriteLock
import static java.lang.reflect.Modifier.*

class ReadWriteLockTestClassLoader extends GroovyTestCase {

  public void testLockFieldDefaultsForReadLock() {
    def tester = new GroovyClassLoader().parseClass('''
      class MyClass {
        @groovy.transform.WithReadLock
        public void readerMethod1() { }
      }
    ''')

    def field = tester.getDeclaredField('$reentrantlock')
    assert isPrivate(field.modifiers)
    assert !isTransient(field.modifiers)
    assert isFinal(field.modifiers)
    assert !isStatic(field.modifiers)
    assert field.type == ReentrantReadWriteLock
  }
}
With this approach debugger breakpoints should be hit when compiling MyClass, making development and troubleshooting much easier. Also, all class definitions are local to the test method, so the test isn’t polluted with dozens of private classes and there are never naming conflicts. But the class definition within a string makes it more difficult to find uses of your annotation. If you need to create instances or run a script as setup, you may want to use GroovyShell instead of GroovyClassLoader, as shown in the following listing.

Listing 9.40. An AST transformation unit test based on GroovyShell
import java.lang.reflect.Modifier

class ReadWriteLockTestGroovyShell extends GroovyTestCase {

  public void testLockFieldDefaultsForReadLock() {
    def tester = new GroovyShell().evaluate('''
      import groovy.transform.WithReadLock
      class MyClass {
        @WithReadLock
        public void readerMethod1() { }
      }
      new MyClass()
    ''')

    def field = tester.getClass().getDeclaredField('$reentrantlock')
    assert Modifier.isPrivate(field.modifiers)
    // and more assertions...
  }
}
Notice how this script returns a new instance of MyClass. GroovyClassLoader and GroovyShell are similar, and which you use is largely a matter of preference. One tip: try to leave your assertions out of the string script. The more you can leave out of the string the better, because tools will have a much easier time understanding and supporting your code.

Global transformations are a little harder to test because they must generally be packaged and on the classpath before your test is compiled. To make testing global transformations easier, Groovy contains a class created specifically for testing called TransformTestHelper. You configure the object with a transformation and a compiler phase in which the transform should run, and then ask it to compile a file or string into a class you can test against. The following listing shows an example of TransformTestHelper.

Listing 9.41. Using TransformTestHelper for testing transformations
import org.codehaus.groovy.tools.ast.TransformTestHelper
import static groovy.test.GroovyAssert.shouldFail
import static org.codehaus.groovy.control.CompilePhase.*

def DATE_FMT = /\w{3} \w{3} \d\d \d\d:\d\d:\d\d \S{3,9} \d{4}/

def folder = new File('src/main/groovy/regina')
def source = new File(folder, 'CompiledAtASTTransformation.groovy')
def transform = getClass().classLoader.parseClass(source).newInstance()

def helper = new TransformTestHelper(transform, PARSING)
def clazz = helper.parse(' class MyClass {} ')
shouldFail(MissingMethodException) {
  clazz.getCompileTime()
}

helper = new TransformTestHelper(transform, CONVERSION)
clazz = helper.parse(' class MyClass {} ')
assert clazz.getCompiledTime()
assert clazz.getCompiledTime() =~ DATE_FMT
Finally, Groovy comes with an AST transformation aimed at testing other AST transformations: @groovy.transform.ASTTest. While the tests we’ve shown you so far are testing the behavior of the AST transformation once it’s been applied, testing an AST transformation properly requires you to perform assertions on the AST itself. For example, each AST node contains a map of custom metadata, called nodeMetaData, where the writer of an AST transformation is allowed to store information. This feature is also used internally by the compiler. The type checker, written in the form of an AST transformation, also stores inferred types in the form of node metadata. The problem is that this information isn’t available in the class file or at runtime. This means that TransformTestHelper wouldn’t be able to access it because the information is lost.

This means that when using TransformTestHelper, you’re testing the result of the AST transformation, but you’re not testing the transformation itself. @ASTTest will let you do that. It’s an annotation that you can put on any AST node that accepts annotations, and that requires two arguments: a compile phase and a code block. For example, you can write:

@ASTTest(phase=CompilePhase.SEMANTIC_ANALYSIS, value={
   assert node instanceof DeclarationExpression
})
def name = 'Testing an AST transformation'
As you can see, the code block, in the form of a closure, has access to a special variable called node, which corresponds to the annotated node. This allows you to write custom assertions on the AST itself! Writing CompilePhase.SEMANTIC_ANALYSIS means that the assertion will be executed after the semantic analysis phase has completed.

Because not all AST nodes accept annotations, in addition to the node variable, the code block gives access to a helper method called lookup. The role of this method is to search for a specific AST node starting from the annotated node. Let’s imagine that you want to perform an assertion on a for loop. As you cannot annotate the for loop directly, we’ll use the lookup method to find it, as shown in the following listing.

Listing 9.42. Using @ASTTest with a lookup function


This technique, which uses a label as a marker, allows you to reach any AST node that is inside the scope of a node that can be annotated. Using @ASTTest, you’re now capable of testing the transformation during the compilation itself, which is a big plus over runtime checks.

Between GroovyShell, GroovyClassLoader, @ASTTest, and TransformTestHelper, there are quite a few options for testing. The hard part of testing isn’t overall test coverage, but covering the edge cases. For instance, do your tests cover code that’s written as a script and code that’s written as a class? How about a mixture of both? Does it cover inner classes and anonymous classes? Have you considered what happens with internal naming conflicts? How does it run when other transformations are present? Properly testing transformations is a fun challenge. There are many opportunities to learn from your own experience but also the experience of others. The Groovy source code contains many unit tests for transformations. Doing a little code archeology now is time well spent, especially if it avoids a future late-night support call.

9.8. LIMITATIONS
Congratulations! You’ve almost finished your training in compile-time metaprogramming. Consider yourself armed and dangerous, both to others and yourself. It may be tempting to write a new language feature, but be careful. There are limitations and drawbacks to mucking about with the Groovy compiler. This section contains the bare minimum set of limitations you should know before embarking on your journey.

9.8.1. It’s early binding
Groovy’s power comes from late binding. Methods can be added to classes at runtime. Method overloading is resolved at runtime. Missing method exceptions can be caught and handled at runtime. In contrast, all the AST transformation work occurs at compile time, making it less flexible than dynamic metaprogramming. It can be very useful, but in general runs against the spirit of the dynamic parts of Groovy. If you can find a runtime solution then use it. Sometimes the best answer to the question “When should I use compile-time meta-programming?” is “Only when you have to.”

9.8.2. It’s fragile
The syntax of Groovy and the GDK classes (anything in the groovy.* package) forms a public contract that’s guaranteed to be backwards compatible between releases. You may have noticed from the import statements in the code examples that most of the AST-related code is in the org.codehaus.groovy packages. The backwards compatibility promises are weaker here. As new features are added to the language, there may be instances where breaking changes are introduced to the AST node hierarchy.

9.8.3. It adds complexity
When you use compile-time metaprogramming you’re basically adding a feature to the language. If you add too many features, your users will drown in complexity. If you provide too many similar features, users may be confused about the best way to use an object. Language designers talk about orthogonality and composition: features should be independent of one another and be able to be used together without conflicts. Complexity lies at the intersection of overlapping language features. Consider how Java generics, autoboxing, and primitive types intersect. There are many edge cases where unboxing a Boolean into a boolean throws a NullPointerException. Or a List can hold all objects except primitives. When several features come together edge cases occur, and sharp edges are dangerous. Be sparing in your cleverness.

9.8.4. Its syntax is fixed
AST transformations can change the meaning, or semantics, of code. For instance, Spock repurposes the logical OR operator (|) and the break/continue label to have a special meaning in test specifications. But Spock doesn’t introduce any new syntax. The syntax of Groovy is fixed by the parser. Invalid Groovy won’t parse, and AST transformations will not be invoked for it. You can change the semantics of the language, but you cannot change the syntax...except that you actually can if you’re determined enough. Under the covers, Groovy uses ANTLR as a parser, and it’s possible to write an ANTLR plugin for Groovy. That’s a topic that deserves its own book, but information about how to do it can be found online.

9.8.5. It’s not typed
Most interesting AST transformations rely on knowing the type of a variable. To Groovy, almost everything is an object. It’s surprisingly difficult to determine the type of an instance and impossible to determine at compile time exactly where a method call will dispatch. For instance, metaclass additions are rarely known at compile time yet affect method dispatch. You can try to keep track of this information yourself, but as soon as a closure is declared, or a second thread is run, then the variable may no longer be what you think it is. This is acceptable for some tools like IDE integration or static analysis that read and make suggestions based on AST. But if you’re rewriting AST and generating new bytecode, then guessing the type of an instance and getting it wrong can have disastrous effects on a program, especially if it fails to compile because of your mistake. Be careful what you think you know about types. It’s easy to make a guess, and it’s easy to be wrong.

9.8.6. It’s unhygienic
It’s possible, using an AST transformation, to introduce a field, class, method, or variable that conflicts with an existing one. If you add a method called getCompiledTime(), you need to consider the possibility that the target class already has that method. The term for a compile-time metaprogramming system that allows naming conflicts is unhygienic. It’s not really a term of derision, but it’s obvious that the term was coined by users with a hygienic language. It’s not an insurmountable problem, and you should carefully select names for synthetic variables and private fields and methods. The $ symbol is typically used in the identifier name because this symbol is rarely used in user-written code. For example, @WithReadLock generates a field called $reentrantlock. You can still have a conflict, but it should be rare. Choose your names carefully.

And with that the compile-time metaprogramming training is complete. It’s time to go out into the world and write some interesting code.

9.9. NEXT STEPS
If you have an idea you want to implement, then the next steps are fairly obvious. Take a look at the templates in the Groovy source distribution, set up a project, and write some code. But before going too far we recommend talking about the idea on the Groovy mailing list. The community takes an active role helping people refine their ideas and make decisions about implementation. You may find you’re talked into using runtime metaprogramming instead.

If you don’t have a specific idea but want to learn more, then writing a static analysis rule for CodeNarc is an excellent way to get started. It’s a small and well-contained project, the community is friendly, and you should be able to make a contribution within an hour or two. There’s a create-rule script that comes with the project that creates all the needed files and unit test templates for new rules. CodeNarc is based on AST visitors, and the various types of visitors are fully described in appendix G.

For bookworms, there are other resources available. One of the authors keeps an active blog that includes several articles related to Groovy and compile-time metaprogramming. But other languages are worth investigating as well. Java, C#, Clojure, and JRuby also expose their ASTs to programmers or allow you to work directly with expression trees, and their documentation is easily found with a search engine. Of the languages listed, Clojure has the best support with a feature called Macros, which is arguably a more advanced and powerful form of the AST transformations described in this chapter. For an overview of language implementation in general, we definitely recommend Language Implementation Patterns by Terence Parr (Pragmatic Brookshelf, 2010). For an overview and classic reference for compile-time metaprogramming in Lisp, read Paul Graham’s On Lisp, which is freely available for download at www.paulgraham.com/onlisp.html.

9.10. SUMMARY
Compile-time metaprogramming is a new, exciting, and growing area of the Groovy ecosystem. Many of the new Groovy features use compile-time metaprogramming to eliminate redundant or verbose code. Use annotations like @Canonical, @Lazy, and @InheritConstructors to remove this unnecessary code from your source files yet still have it visible from Java. Use @Delegate, @Immutable, and @Singleton for an easy path to correct object design. The annotations @Log, @Commons, @Log4j, and @Slf4j streamline declaring and using loggers. Declarative concurrency constructs like @Synchronized clean up multithreaded code considerably, @AutoClone and @AutoExternalize help make externalizing and cloning simple, and scripting has become much safer with @TimedInterrupt, @ThreadInterrupt, and @ConditionalInterrupt.

Writing your own transformation involves either a local AST transformation, which manipulates the AST when a certain annotation is discovered, or a global AST transformation, which is run for every compiled class. If your transformation needs to know information about the tree, then you’ll probably need a code visitor to walk the entire AST, visiting nodes as they’re found in the source code. If neither local nor global transformation is suitable for your scenario then you can always weave a visitor directly into a classloader without too much effort.

Groovy contains several alternatives for generating ASTs. Writing them by hand using the class constructors is always an option, but it’s worth learning the AstBuilder API as well. The buildFromSpec method offers a useful DSL over the constructors; buildFromString offers a useful abstraction over all the classes, and buildFromCode provides an intuitive and elegant way to convert source into an AST.

There are several standard tools for working with ASTs. The javap application displays the raw .class file output. Groovy Console’s AST browser is much more advanced, and shows the AST in tree form and also generated source-code form. A good Java decompiler is always a useful view of transformation results too, but perhaps the best tool is a large suite of unit tests. GroovyClassLoader, GroovyShell, and TransformTestHelper can all be used to test drive (or regression test) AST transformations.

Compile-time metaprogramming isn’t suitable for every scenario. In general, we prefer late binding and flexibility. But there are concrete advantages: Java integration, easy embedded language development, delayed evaluation, and the ability to change the semantics of the language. AST transformations are opening a whole new world of what’s possible with Groovy. After reading this chapter you should be better prepared to go into that world and see what new and useful tools you can create.

Recommended Playlists  History Topics Tutorials Settings Support Get the App Sign Out
© 2018 Safari. Terms of Service / Privacy Policy