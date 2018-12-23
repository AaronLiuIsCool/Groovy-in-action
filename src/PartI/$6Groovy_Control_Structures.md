# Chapter 6. Groovy control structures
This chapter covers

Groovy truth
Conditionals and branching
Looping
Exception handling
The pursuit of truth and beauty is a sphere of activity in which we are permitted to remain children all our lives.

Albert Einstein

At the hardware level, computer systems use simple arithmetic and logical operations, such as jumping to a new location if a memory value equals zero. Any complex flow of logic that a computer is executing can always be expressed in terms of these simple operations. Fortunately, languages like Java raise the abstraction level available in programs you write so that you can express the flow of logic in terms of higher-level constructs—for example, looping through all of the elements in an array or processing characters until you reach the end of a file.

In this chapter, we explore the constructs Groovy provides to describe logic flow in ways that are even simpler and more expressive than Java. Before we look at the constructs themselves, however, we have to examine Groovy’s answer to that age-old philosophical question: What is truth?[1]

1 Groovy has no opinion as to what beauty is. We’re sure that if it did, however, it would involve expressive minimalism. Closures too, probably.

6.1. GROOVY TRUTH
To understand how Groovy handles control structures such as if and while, you need to know how it evaluates expressions, which need to have Boolean results. Many of the control structures we examine in this chapter rely on the result of a Boolean test—an expression that’s first evaluated and then considered as being either true or false. The outcome of this affects which path is then followed in the code. In Java, the consideration involved is usually trivial, because Java requires the expression to be one resulting in the primitive boolean type to start with. Groovy is more relaxed about this, allowing simpler code at the slight expense of language simplicity. We’ll examine Groovy’s rules for Boolean tests and give some advice to avoid falling into an age-old trap.

6.1.1. Evaluating Boolean tests
The expression of a Boolean test can be of any (nonvoid) type. It can apply to any object. Groovy decides whether to consider the expression as being true or false by applying the rules shown in table 6.1, based on the result’s runtime type. The rules are applied in the order given, and once a rule matches, it completely determines the result.[2]

2 It would be rare to encounter a situation where more than one rule matched, but you never know when someone will subclass java.lang.Number and implement java.util.Map at the same time.

Table 6.1. Sequence of rules used to evaluate a Boolean test
Runtime type

Evaluation criterion required for truth

Boolean	Corresponding Boolean value is true
Matcher	Matcher has a match
Collection	Collection is nonempty
Map	Map is nonempty
String, GString	String is nonempty
Number, Character	Value is nonzero
None of the above	Object reference is non-null
The following listing shows these rules in action, using the Boolean negation operator ! to assert that expressions that ought to evaluate to false really do so.

Listing 6.1. Example Boolean test evaluations


These rules can make testing for “truth” simpler and easier to read. But they come with a price, as you’re about to find out.

6.1.2. Assignments within Boolean tests
Before we get into the meat of the chapter, we have a warning to point out. Just like Java, Groovy allows the expression used for a Boolean test to be an assignment—and the value of an assignment expression is the value assigned. Unlike Java, the type of a Boolean test isn’t restricted to booleans, which means that a problem you might have thought was ancient history reappears, albeit in an alleviated manner. Namely, an equality operator == incorrectly entered as an assignment operator = is valid code with a drastically different effect than the intended one. Groovy shields you from falling into this trap for the most common appearance of this error: when it’s used as a top-level expression in an if statement. But it can still arise in less usual cases.

The following listing leads you through some typical variations of this topic.

Listing 6.2. What happens when == is mistyped as =


The equality comparison  is fine and would be allowable in Java. In , an equality comparison was intended, but one of the equal signs was left out. This raises a Groovy compiler error, because an assignment isn’t allowed as a top-level expression in an if test.

Boolean tests can be nested inside expressions in arbitrary depth; the simplest one is shown at , where extra parentheses around the assignment make it a subexpression, and therefore the assignment becomes compliant with the Groovy language. The value 3 will be assigned to x, and x will be tested for truth. Because 3 is considered true, the value 3 gets printed. This use of parentheses to please the compiler can even be used as a trick to spare an extra line of assignment. The unusual appearance of the extra parentheses then serves as a warning sign for the reader.

The restriction of assignments from being used in top-level Boolean expressions applies only to if and not to other control structures such as while. Doing assignment and testing in one expression are often used with while in the style shown at . This style tends to appear with classic uses like processing tokens retrieved from a parser or reading data from a stream. Although this is convenient, it leaves us with the potential coding pitfall shown at , where x is assigned the value 2 and the loop would never stop if there weren’t a break statement.[3]

3 Remember that the code in this book has been executed. If we didn’t have the break statement, the book would have taken literally forever to produce.

This potential cause of bugs has given rise to the idiom in other languages (such as C and C++, which suffer from the same problem to a worse degree) of putting constants on the left side of the equality operator when you wish to perform a comparison with one. Such a construct is sometimes called a “Yoda conditional” since in the Star Wars motion picture the Yoda character talks in this swapped fashion like “difficult to see the future is.” Following this style the last while statement in the previous listing (still with a typo) becomes



This would raise an error, as you can’t assign a value to a constant. We’re back to safety—so long as constants are involved. Unfortunately, not only does this fail when both sides of the comparison are variables, it also reduces readability. Whether it’s a natural occurrence, a quirk of human languages, or conditioning, most people find while (x==3) significantly simpler to read than while (3==x). Although neither is going to cause confusion, the latter tends to slow people down or interrupt their train of thought. In this book, we’ve favored readability over safety—but our situation is somewhat different than that of normal development. You’ll have to decide for yourself which convention suits you and your team better.

Now that we’ve examined which expressions Groovy will consider to be true and false, we can start looking at the control structures themselves.

6.2. CONDITIONAL EXECUTION STRUCTURES
Our first set of control structures deals with conditional execution. They all evaluate a Boolean test and make a choice about what to do next based on whether the result was true or false. None of these structures should come as a new experience to any Java developer, but, of course, Groovy adds twists of its own. We’ll cover if statements, the conditional operator, switch statements, and assertions.

6.2.1. The humble if statement
Our first two structures act exactly the same way in Groovy as they do in Java, apart from the evaluation of the Boolean test itself. We start with if and if else statements.

Just as in Java, the Boolean test expression must be enclosed in parentheses. The conditional block is normally enclosed in curly braces. These braces are optional if the block consists of only one statement.[4]

4 Even though the braces are optional, many coding conventions insist on them to avoid errors that can occur through careless modification when they’re not used.

A special application of the “no braces needed for single statements” rule is the sequential use of else if. In this case, the logical indentation of the code is often flattened—that is, all else if lines have the same indentation although their meaning is nested. The indentation makes no difference to Groovy and is only of aesthetic relevance.

Listing 6.3 gives some examples, using assert true to show the blocks of code that will be executed, and assert false to show the blocks that won’t be.

There should be no surprises in the listing, although it might still look slightly odd to you that non-Boolean expressions such as strings and lists can be used for Boolean tests. Don’t worry—it becomes natural over time.

Listing 6.3. The if statement in action
if (true)       assert true
else            assert false
if (1) {
    assert true
} else {
    assert false
}
if ('nonempty') assert true
else if (['x']) assert false
else            assert false
if (0)          assert false
else if ([])    assert false
else            assert true
6.2.2. The conditional ?: operator and Elvis
Groovy also supports the ternary conditional operator ?: for small inline tests, as shown in listing 6.4. This operator returns the object that results from evaluating the expression left or right of the colon, depending on the test before the question mark. If the first expression evaluates to true, the middle expression is evaluated. Otherwise, the last expression is evaluated. Just as in Java, whichever of the last two expressions isn’t used as the result isn’t evaluated.

Listing 6.4. The conditional operator
def result = (1==1) ? 'ok' : 'failed'
assert result == 'ok'
result = 'some string' ? 10 : ['x']
assert result == 10
Again, notice how the Boolean test (the first expression) can be of any type. Also note that because everything is an object in Groovy, the middle and last expressions can be of radically different types.

Groovy comes with another interesting shortcut for the case that the test expression is to be used as the result value when true. Consider this piece of code:

def argument = "given"
def standard = "default"
def result   = argument ? argument : standard
Groovy allows you to abbreviate the third line as

def value = argument ?: standard
Not only is this version shorter but it also evaluates the argument only once. When reading the ?: operator like an emoticon you can guess why we call it the Elvis operator.

Opinions about the ternary conditional operator vary wildly. Some people find it extremely convenient and use it often. Others find it too Perl-ish. You may well find that you use it less often in Groovy because there are features that make its typical applications obsolete—for example, GStrings (covered in section 3.4.2) allow dynamic creation of strings that would be constructed in Java using the ternary operator.

So far, so Java-like. Things change significantly when we consider switch statements.

6.2.3. The switch statement and the in operator
On a recent train ride, I (Dierk) spoke with a teammate about Groovy, mentioning the oh-so-cool switch capabilities. He wouldn’t even let me get started, waving his hands and saying, “I never use switch!” I was put off at first, because I lost my momentum in the discussion; but after more thought, I agreed that I don’t use it either—in Java.

The switch statement in Java is quite restrictive. Originally you could only switch on an int type, with byte, char, and short automatically being promoted to int. As of Java 5, enum types can also be switched on, due to some compiler trickery, and as of Java 7, additional trickery with string hash codes lets you use strings too. But even with these extensions, it’s still restrictive. Its applicability is bound to either low-level tasks or some kind of dispatching on a type code. In object-oriented languages, the use of type codes is considered smelly.[5]

5 See “Replace Conditional with Polymorphism” in chapter 9 of Refactoring by Martin Fowler (Addison-Wesley, 2000).

The switch structure
The general appearance of the switch construct, shown in the following listing, is just like in Java, and its logic is identical in the sense that the handling logic falls through to the next case unless it’s exited explicitly. We’ll explore exiting options in section 6.4.

Listing 6.5. General switch appearance is like Java or C


Although the fall through is supported in Groovy, there are few cases where this feature really enhances the readability of the code. It usually does more harm than good (this applies to Java, too). As a general rule, putting a break at the end of each case is good style.

switch with classifiers
You’ve seen the Groovy switch used for classification in section 3.5.5 and when working through the datatypes. A classifier is eligible as a switch case if it implements the isCase method. In other words, a Groovy switch like

switch (candidate) {
    case classifier1  : handle1()      ; break
    case classifier2  : handle2()      ; break
    default           : handleDefault()
}
is roughly equivalent (besides the fall through and exit handling) to

if      (classifier1.isCase(candidate)) handle1()
else if (classifier2.isCase(candidate)) handle2()
else     handleDefault()
This allows expressive classifications and even some unconventional uses with mixed classifiers. Unlike Java’s constant cases, the candidate may match more than one classifier. This means that the order of cases is important in Groovy, whereas it doesn’t affect behavior in Java. The next listing gives an example of multiple types of classifiers. After having checked that our number 10 isn’t zero, isn’t in range 0..9, isn’t in list [8,9,11], isn’t of type Float, and isn’t an integral multiple of 3, we finally find it to be made of two characters.

Listing 6.6. Advanced switch and mixed classifiers


The new feature  is that we can classify by type. Float is of type java.lang.Class, and the GDK enhances Class by adding an isCase method that tests the candidate with isInstance.

The isCase method on closures  passes the candidate into the closure and returns the result of the closure call coerced to a Boolean.

The final classification  as a two-digit number works because ~/../ is a Pattern and the isCase method on patterns applies its test to the toString value of the argument.

To leverage the power of the switch construct, it’s essential to know the available isCase implementations. It’s not possible to provide an exhaustive list, because any custom type in your code or in a library can implement it. Table 6.2 has the list of known implementations in the GDK.

Table 6.2. Standard implementations of isCase for switch, grep, and in
Class

a.isCase(b) implemented as

Object	a.equals(b)
Class	a.isInstance(b)
Collection	a.contains(b)
Range	a.contains(b)
Pattern	a.matcher(b.toString()).matches()
String	(a==null && b==null) || a.equals(b)
Closure	a.call(b)
Note
The isCase method is also used with grep on collections such that collection.grep(classifier) returns a collection of all items that are a case of that classifier.

The in operator
The isCase logic is actually used three times: for switch cases for grep classification and for the in operator as used for conditionals like the following assertion:

def okValues = [1, 2, 3]
def value    = 2
assert value in okValues
Using the Groovy switch in the sense of a classifier is a big step forward. It adds much to the readability of the code. The reader sees a simple classification instead of a tangled, nested construction of if statements. Again, you’re able to reveal what the code does rather than how it does it.

As pointed out in section 4.1.2, the switch classification on ranges is particularly convenient for modeling business rules that tend to prefer discrete classification to continuous functions. The resulting code reads almost like a specification.

Look actively through your code for places to implement isCase. A characteristic sign of looming classifiers is lengthy else if constructions.

Advanced topic
It’s possible to overload the isCase method to support different kinds of classification logic depending on the candidate type. If you provide both methods, isCase(String candidate) and isCase(Integer candidate), then switch ('1') can behave differently than switch(1) with your object as the classifier.

Our next topic, assertions, may not look particularly important at first glance. But although assertions don’t change the business capabilities of the code, they do make the code more robust in production. Moreover, they do something even better: enhance the development team’s confidence in their code, as well as their ability to remain agile during additional enhancements and ongoing maintenance.

6.2.4. Sanity checking with assertions
This book contains several hundred assertion statements—and indeed, you’ve already seen a number of them. Now it’s time to go into extra detail. We’ll look at producing meaningful error messages from failed assertions, reflect over reasonable uses of this keyword, and show how to use it for inline unit tests. We’ll also quickly compare the Groovy solution to Java’s assert keyword and assertions as used in unit test cases.

Producing informative failure messages
When an assertion fails, it produces a stack trace and a message. Put the code

a = 1
assert a==2
in a file called FailingAssert.groovy, and let it run via

> groovy FailingAssert.groovy
It’s expected to fail, and it does so with the message

Assertion failed:

assert a==2
       ||
       |false
       1
       at FailingAssert.run(FailingAssert.groovy:2)
       at FailingAssert.main(FailingAssert.groovy)
You see that on failure, the assertion prints out the failed expression and the value of all subexpressions plus the stack trace.

This is a lot of information, and it’s sufficient to locate and understand the error in most cases, but not always. Let’s try another example that tries to protect a file reading code from being executed if the file doesn’t exist or cannot be read (Perl programmers will see the analogy to or die):

input = new File('no such file')
assert  input.exists()
assert  input.canRead()
println input.text
This produces the output

Caught: java.lang.AssertionError: Expression: input.exists()
  ...
which isn’t very informative. The missing information here is what the bad filename was. To this end, assertions can be instrumented with a trailing message:

input = new File('no such file')
assert input.exists()  , "cannot find '$input.name'"
assert input.canRead() , "cannot read '$input.canonicalPath'"
println input.text
This produces the following:

... cannot find 'no such file'. Expression: input.exists()
which is the information we need. But this special case also reveals the sometimes unnecessary use of assertions, because in this case we could easily leave the assertions out:

input = new File('no such file')
println input.text
The result is the following sufficient error message:

FileNotFoundException: no such file (The system cannot find the file
specified)
This leads to the following best practices with assertions:

Before writing an assertion, let your code fail, and see whether any other thrown exception is good enough.
When writing an assertion, let it fail the first time, and see whether the failure message is sufficient. If not, add a message. Let it fail again to verify that the message is now good enough.
If you feel you need an assertion to clarify or protect your code, add it regardless of the previous rules.
If you feel you need a message to clarify the meaning or purpose of your assertion, add it regardless of the previous rules.
Ensure code with inline unit tests
Finally, there’s a potentially controversial use of assertions as unit tests that lives right inside production code and gets executed with it. The following listing shows this strategy with a nontrivial regular expression that extracts a hostname from a URL. The pattern is first constructed and then applied to some assertions before being put to action. We also implement a simple method assertHost for easy asserting of a match grouping.[6]

6 Please note that we use regexes here only to show the value of assertions. If we really set out to find the hostname of a URL, we’d use candidate.toURL().host.

Listing 6.7. Using assertions for inline unit tests


Imagine finding a Groovy script such as this sitting on a production filesystem and let’s assume you want to understand it. If you’re very lucky, the script might be under version control and have a test harness that’s run against it regularly. But if that isn’t the case, or if the preceding example assertions were perhaps included as comments, then a reader of the script cannot really be sure that it works as expected. In such circumstances, the value of inline assertions becomes obvious.

Some may fear a bad impact on performance when doing this style of inline unit tests. The best answer is to use a profiler and investigate where performance is really relevant. Our assertions in listing 6.7 run in a few milliseconds and shouldn’t normally be an issue. When performance is important, one possibility would be to put inline unit tests where they’re executed only once per loaded class: in a static initializer. You’ll need to decide for yourself whether inline unit tests suit your scenarios, but we strongly recommend them as a technique to keep in mind and apply on a case-by-case basis.

Relationships to other assertions
Java has had an assert keyword since JDK 1.4. It differs from Groovy assertions in that it has a slightly different syntax (a colon instead of a comma to separate the Boolean test from the message) and it can be enabled and disabled. Java’s assertion feature isn’t as powerful, because it works only on a Java Boolean test, whereas the Groovy assert keyword takes a full Groovy conditional (see section 6.1).

The JDK documentation has a long chapter on assertions that discusses the disabling feature for assertions and its impact on compiling, starting the virtual machine, and resulting design issues. Although this is fine and the design rationale behind Java assertions is clear, we feel the disabling feature is the biggest stumbling block for using assertions in Java. You can never be sure that your assertions are really executed.

Some people claim that for performance reasons, assertions should be disabled in production, after the code has been tested with assertions enabled. On this issue, Bertrand Meyer,[7] the father of design by contract, pointed out that it’s like learning to swim with a swimming belt and taking it off when leaving the pool and getting in the ocean. In Groovy, your assertions are always executed.

7 See Object-Oriented Software Construction, 2nd ed., by Bertrand Meyer (Prentice-Hall, 1997).

Assertions also play a central role in unit tests. Groovy comes with a bundled version of JUnit, the leading unit test framework for Java. JUnit makes a lot of specialized assertions available to its TestCases. Groovy adds even more of them, as you’ll see in chapter 17. The information that Groovy provides when assertions fail makes them very convenient when writing unit tests, because it relieves the tester from writing lots of messages.

Assertions can make a big difference to your personal programming style and even more to the culture of a development team, regardless of whether they’re used inline or in separate unit tests. Asserting your assumptions not only makes your code more reliable, it also makes it easier to understand and easier to work with.

That’s it for conditional execution structures. They’re the basis for any kind of logical branching and a prerequisite to allow looping—the language feature that makes your computer do all the repetitive work for you. The next two sections cover the looping structures while and for.

6.3. LOOPING
The structures you’ve seen so far have evaluated a Boolean test once and changed the path of execution once based on the result of the condition. Looping, on the other hand, repeats the execution of a block of code multiple times. The loops available in Groovy are while and for, both of which we cover here.

6.3.1. Looping with while
The while construct is like its Java counterpart. The only difference is the one you’ve seen already—the power of Groovy Boolean test expressions. To summarize, the Boolean test is evaluated, and if it’s true, the body of the loop is then executed. The test is then reevaluated, and so forth. Only when the test becomes false does control proceed past the while loop. The next listing shows an example that removes all entries from a list. We visited this problem in chapter 3, where you discovered that you can’t use each for that purpose. The second example adds the values again in a one-liner body without the optional braces.

Listing 6.8. Example while loops
def list = [1,2,3]
while (list) {
    list.remove(0)
}
assert list == []

while (list.size() < 3) list << list.size()+1
assert list == [1,2,3]
Again, there should be no surprises in this code, with the exception of using just list as the Boolean test in the first loop.

Note that there are no do {} while(condition) or repeat {} until (condition) loops in Groovy. Of course with closures you could write your own do-while or repeat-until control structures with only some minor restrictions and differences compared to a language-supported equivalent. We discuss some of these differences in the next section. In chapter 19, we look at a WhenUntilTransform which even removes some of the limitations.

6.3.2. Looping with for
Considering it’s probably the most commonly used type of loop, the traditional for loop in Java is relatively hard to use, when you examine it closely. Through familiarity, people who have used a language with a similar structure (and there are many such languages) grow to find it easy to use, but that is solely due to frequent use, not to good design. Although the nature of the traditional for loop is powerful, it’s rarely used in a way that can’t be more simply expressed in terms of iterating through a collection-like data structure. Although supporting most forms of the for loop that Java supports, Groovy embraces this simplicity and strongly encourages for loops following this structure:

for (variable in iterable) { body }
where variable may optionally have a declared type. The Groovy for loop iterates over iterable. Frequently used iterables are ranges, collections, maps, arrays, iterators, and enumerations. In fact, any object can be an iterable. Groovy applies the same logic as for object iteration, described in chapter 12.

Braces around the body are optional if it consists of only one statement. The following listing shows some of the possible combinations.

Listing 6.9. Multiple for loop examples




The first example  uses explicit typing for s and no braces with a loop body of a single statement. The looping is done on a range of strings.

The usual for loop appearance when working on a collection is shown in . Recall that thanks to autoboxing, this also works for arrays.

Groovy also supports Java for loops style  and the more recent iterable variants either on the index  or the string value itself .

Looping on a half-exclusive integer range  is a slight improvement over the traditional Java for loop style  or an equivalent to the Java iterable index style .

The final example  illustrates the typical Groovy style recommended when working on strings. It’s more Groovy to treat a string as a collection of characters.

Using the for loop with object iteration as described in section 12.1.3 provides some very powerful combinations. You can use it to print a file line-by-line via

def file = new File('myFileName.txt')
for (line in file) println line
or to print all one-digit matches of a regular expression:

def matcher = '12xy3'=~/\d/
for (match in matcher) println match
If the container object is null, no iteration will occur:

for (x in null) println 'This will not be printed!'
If Groovy cannot make the container object iterable by any means, the fallback solution is to do an iteration that contains only the container object itself:

for (x in new Object()) println "Printed once for object $x"
Object iteration makes the Groovy for loop a sophisticated control structure. It’s a valid counterpart to using methods that iterate over an object with closures, such as using Collection’s each method.

The main difference is that the body of a for loop isn’t a closure! That means this body is a block:

for (x in 0..9) { println x }
whereas this body is a closure:

(0..9).each { println it }
Even though they look similar, they’re very different in construction.

A closure is an object of its own and has all the features that you saw in chapter 5. It can be constructed in a different place and passed to the each method. The body of the for loop, in contrast, is directly generated as bytecode at its point of appearance. No special scoping rules apply.

This distinction is even more important when it comes to managing exit handling from the body. The next section shows why.

6.4. EXITING BLOCKS AND METHODS
Although it’s nice to have code that reads as a simple list of instructions with no jumping around, it’s often vital that control is passed from the current block or method to the enclosing block or calling method—or sometimes even further up the call stack. Just like in Java, Groovy allows this to happen in an expected, orderly fashion with return, break, and continue statements, and in emergency situations with exceptions. Let’s take a closer look.

6.4.1. Normal termination: return/break/continue
The general logic of return, break, and continue is similar to Java. One difference is the return keyword is optional for the last expression in a method or closure. If it’s omitted, the return value is that of the last expression. Methods with explicit return type void don’t return a value; closures always return a value.[8]

8 But what if the last evaluated expression of a closure is a void method call? In this case, the closure returns null.

The following listing shows how the current loop is cut short with continue and prematurely ended with break. Like Java, there’s an optional label.

Listing 6.10. Simple break and continue


In classic programming style, the use of break and continue is sometimes considered smelly. But it can be useful for controlling the workflow in services that run in an endless loop. Similarly, returning from multiple points in the method is frowned upon in some circles, but other people find it can greatly increase the readability of methods that might be able to return a result early. We encourage you to figure out what you find most readable and discuss it with whoever else is going to be reading your code—consistency is as important as anything else.

As a final note on return handling, remember that closures, when used with iteration methods like each, have a different meaning of return than the control structures while and for, as explained in section 5.6.

6.4.2. Exceptions: throw/try-catch-finally
Exception handling in Groovy is similar to Java and follows the same logic. Just as in Java, you can specify a complete try-catch-finally sequence of blocks, or just try-catch, or just try-finally. Note that unlike various other control structures, braces are required around the block bodies whether or not they contain more than one statement. The main difference between Java and Groovy in terms of exceptions is that declarations of exceptions in the method signature are optional, even for checked exceptions. The next listing shows the usual behavior.

Listing 6.11. Exception handling in Groovy
def myMethod() {
    throw new IllegalArgumentException()
}

def log = []
try {
    myMethod()
} catch (Exception e) {
    log << e.toString()
} finally {
    log << 'finally'
}
assert log.size() == 2
There are no compile-time or runtime warnings from Groovy when checked exceptions aren’t declared. When a checked exception isn’t handled, it’s propagated up the execution stack like a RuntimeException.

Java 7 introduced a multi-catch syntax. Groovy also supports this as this code shows:

try {
  if (Math.random() < 0.5) 1 / 0
  else null.hashCode()
} catch (ArithmeticException | NullPointerException exception) {
  println exception.class.name
}
Note
Java 7 introduced a try-with-resources mechanism. At the time of writing, Groovy doesn’t support that syntax. try-with-resources isn’t needed in Groovy, where we have full closure support. A future version of Groovy may support the Java 7 notation to ease cut-and-paste compatibility between the two languages, but even if it does, we’d encourage you to consider the closure variants for managing resources, which are cleaner and more powerful.

We cover integration between Java and Groovy in more detail in chapter 16; but it’s worthwhile noting an issue relating to exceptions here. When using a Groovy class from Java, you need to be careful. The Groovy methods will not declare that they throw any checked exceptions unless you’ve explicitly added the declaration, even though they might throw checked exceptions at runtime. Unfortunately, the Java compiler attempts to be clever and will complain if you try to catch a checked exception in Java when it believes there’s no way that the exception can be thrown. If you run into this and need to explicitly catch a checked exception generated in Groovy code, you may need to add a throws declaration to the Groovy code, just to keep javac happy.

6.5. SUMMARY
This was our tour through Groovy’s control structures: conditionally executing code, looping, and exiting blocks and methods early. It wasn’t too surprising because everything turned out to be like Java, enriched with a bit of Groovy flavor. The only structural difference was the for loop. Exception handling is very similar to Java, except without the requirement to declare checked exceptions.[9]

9 Checked exceptions are regarded by many as an experiment that was worth performing but that proved not to be as useful as had been hoped.

Groovy’s handling of Boolean tests is consistently available both in conditional execution structures and in loops. We examined the differences between Java and Groovy in determining when a Boolean test is considered to be true. This is a crucial area to understand, because idiomatic Groovy will often use tests that aren’t simple Boolean expressions.

The switch keyword and its use as a general classifier bring a new object-oriented quality to conditionals. The interplay with the isCase method allows objects to control how they’re treated not only inside switch but also for the grep method on lists and the in operator in Boolean expressions. You get three for one. Although the use of switch is often discouraged in object-oriented languages, the new power given to it by Groovy gives it a new sense of purpose.

In the overall picture, assertions find their place as the bread-and-butter tool for the mindful developer. They belong in the toolbox of every programmer who cares about their craft.

With what you learned in the tour, you have all the means to do any kind of procedural programming. But certainly, you have higher goals and want to master object-oriented programming. The next chapter will teach you how.

Recommended Playlists  History Topics Tutorials Settings Support Get the App Sign Out
© 2018 Safari. Terms of Service / Privacy Policy