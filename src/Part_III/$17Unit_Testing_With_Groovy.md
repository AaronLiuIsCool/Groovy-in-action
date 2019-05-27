# Chapter 17. Unit testing with Groovy
This chapter covers
1. Unit testing Groovy and Java code
2. Incorporating code coverage tools
3. Integrating IDEs
4. Testing with Spock
5. Automating the build process
## The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong, it usually turns out to be impossible to get at or repair.
Douglas Adams

Developer unit testing has become a de facto standard in the Java community.[1] The confidence and structure that 
JUnit[2] and other testing frameworks bring to the development process are almost revolutionary. To those of us who 
were actively developing Java applications in the latter years of the 20th century, automated unit testing was 
almost unheard of. Yes, we wrote tests, but they were hardly automated or even a part of a standard build!

```
1 See Kevin Tate, Sustainable Software Development: An Agile Perspective (Addison Wesley Professional, 2005) 
and Greg Smith and Ahmed Sidky, Becoming Agile (Manning, 2009).
2 See Petar Tahchiev, et al., JUnit in Action, 2nd Ed. (Manning, 2010); J. B. Rainsberger, JUnit Recipes 
(Manning, 2004), and www.junit.org for more information.
``` 

Fast-forward to the present, and many people wouldn’t think of writing, let alone releasing, code without 
corresponding unit tests. We write tests all the time, and we expect everyone else on our teams to do the same. 
Moreover, momentum is growing behind the idea of writing code by always writing tests first. Although this isn’t 
universal, it’s another indicator that the recent growth in the importance of tests will continue.

We test at all levels, from unit testing to integration testing to system testing. It’s sometimes more fun to write 
the tests than the subject under test, because doing so improves not only the code itself, but also the design of 
the code. When tests are written often and continually, code has the benefit of being highly extensible, in addition 
to being obviously freer of defects and easier to repair when needed.

Combine this increased awareness of developer testing with Groovy, and you have a match made in heaven. With Groovy, 
tests can be written more quickly and easily. It gets even better when you combine the simplicity of unit testing 
in Groovy with normal Java. You can write Groovy tests for your Groovy-based systems and leverage the many Java 
libraries and test-extension packages. You can write Groovy tests for your Java-based systems and leverage 
Groovy’s enhanced syntax benefits and extended test functionality.

Groovy makes unit testing a breeze, whichever way you use it, mainly due to four key aspects:

* Groovy embeds JUnit, so you don’t have to set up a new dependency.
* Groovy has an enhanced test-case class, which adds a plethora of new assertion methods.
* Groovy has built-in mock, stub, and other dynamic class-creation facilities that simplify isolating a test class 
from its collaborators.
* Tests written in Groovy can be easily run from Gradle, Maven, or your favorite IDE.
* Our focus in this chapter is unit testing; however, many of the ideas can be extended to other kinds of 
testing as well. We’ll mention specific examples throughout the chapter.

## 17.1. GETTING STARTED
The section header implies that you have preparation to do before you can start your testing activities. But you don’t. 
There’s no external support to download or install. Groovy treats unit testing as a first-class developer duty and 
ships with everything you need for that purpose.

Even more important, it simplifies testing by making assertions part of the language,[3] automatically executing test 
cases by transparently invoking its TestRunner when needed, and providing the means to run suites of test cases easily, 
both from the command line and through integration with your IDE or build environment. This section shows you how 
simple it can be and introduces you to GroovyTestCase, the base class used for most unit testing in Groovy.

```3 Java also supports assertions at the language level but disables them by default.```

### 17.1.1. Writing tests is easy
Assume you have Groovy code that converts temperatures measured in Fahrenheit (F) to Celsius (C). To that end, you 
define a celsius method in a Converter class like so:
```
class Converter {
    static celsius (fahrenheit) { (fahrenheit - 32) * 5 / 9 }
}
```
Is this implementation correct? Probably, but you can’t be sure. You need to gain additional confidence in this 
method before the next non-U.S. traveler uses your method to understand the U.S. weather forecast.

A common approach with unit testing is to call the subject under test with static sample data that produces 
well-known results. That way, you can compare the calculated results against your expectations.

Choosing a good set of samples is key. As a rule of thumb, having a few typical cases and all the corner cases you 
can think of is a good choice.[4] Typical cases would be 68° F = 20° C for having a garden party or 95° F = 35° C for 
going to the beach. Corner cases would be 0° F, which is between -17° C and -18° C, the coldest temperature that 
Gabriel Daniel Fahrenheit could create with a mixture of ice and ordinary salt in 1714. Another corner case is when 
water freezes at 32° F = 0° C.

```4 Finding good test data is a science of its own and involves activities such as structural analysis of the 
parameter domain. For our purposes, we keep it simple. Refer to the background literature for more information.
```

Sound complicated? It isn’t. The following listing statically imports the method and then adds scripted unit tests 
using simple assertions that are built into the language itself.

Listing 17.1. Scripted unit tests for the Fahrenheit to Celsius conversion method
```
assert 20 == celsius(68)
```
Scripted tests of this kind are useful. As an example, look at this book: most listings contain such self-checking 
asserts to ensure the code works and to help reveal your expectations from the code at the same time. Most even 
work as inline tests where assertions live inside the code under test.

But what if a test would fail due to an implementation error? Groovy will report this back in a visual way. Say 
that instead of converting 95° F to 35° C, we incorrectly assume that the result should be 34° C. If we execute 
the following:
```
assert 34  == celsius(95)
```
Groovy will report us back the following assertion error:
```
assert 34  == celsius(95)
           |  |
           |  35
           false
```
Here you can see Groovy’s Power Assert at work. Power Assert, originally introduced in the Spock Testing Framework, 
provides a very concise and clear way of reporting errors by outputting the result of each invocation to the console. 
This makes it easier to understand which parts went right, and which parts wrong.

Whenever the environment of self-testing code changes, the inline tests assert that it is still working. Environmental 
changes can happen for a number of reasons: evaluating the script on a different machine, using an updated JDK or 
Groovy version, or running with different versions of packages that the script depends upon.

There are circumstances when tests cannot be inlined, such as due to performance requirements. Sometimes scripted 
tests aren’t convenient enough because they do not self-organize into trees of test suites. In such cases, it’s 
conventional to pack all the tests of a given script or class into a separate class residing in a separate file. 
This is where GroovyTestCase appears on stage.

### 17.1.2. GroovyTestCase: an introduction
Groovy bundles an extended JUnit class dubbed GroovyTestCase, which facilitates unit testing in a number of ways. 
It includes a host of new assert methods, and it also facilitates running Groovy scripts masquerading as test cases.

The added assertions are listed in table 17.1. We won’t go into the details of each method, mostly because they’re descriptively named. Where it’s not absolutely obvious what the meaning is, the description provided in the table should be sufficient. Even though we won’t discuss them explicitly, we’ll use them in the assertions elsewhere in this chapter, so you’ll see how useful they are.

Table 17.1. Enhanced assertions available in GroovyTestCase
Method

Description

void assertArrayEquals(Object[] expected, Object[] value)	Compares the contents and length of each array
void assertContains(char expected, char[] array)	Verifies that a given array of chars contains an expected value
void assertContains(int expected, int[] array)	Verifies that a given array of ints contains an expected value
void assertInspect(Object value, String expected)	Similar to the assertToString method, except that it calls the inspect method
void assertLength(int length, char[] array)	Convenience method for asserting the length of an array
void assertLength(int length, int[] array)	Convenience method for asserting the length of an array
void assertLength(int length, Object[] array)	Convenience method for asserting the length of an array
void assertScript(final String script)	Attempts to run the provided script
void assertToString(Object value, String expected)	Invokes the toString method on the provided object and compares the result with the expected string
void shouldFail(Closure code)	Verifies that the closure provided fails
void shouldFail(Class clazz, Closure code)	Verifies that the closure provided throws an exception of type clazz
void shouldFail(String scriptText)	Verifies that the provided script fails when executed
void shouldFail(Class clazz, String scriptText)	Verifies the provided script throws an exception of type clazz when executed
void shouldFailWithCause(Class clazz, Closure code)	Verifies that the closure provided fails and that a particular exception is the cause of the failure
In addition to the methods listed in the previous table, consider the convenient notYetImplemented method, which you can use to mark a test method as not implemented yet. Here is an example:

public void testNotImplementedYet() {
    if (GroovyTestCase.notYetImplemented(this)) return
    fail("will be implemented tomorrow")
}
In the previous example, the test case is marked as not yet implemented. If the test somehow passes, which was not yet expected, the test will fail with a descriptive error message.

Groovy doesn’t force you to extend GroovyTestCase, and you’re free to continue to extend the traditional TestCase class provided by JUnit.[5] Having said that, unless you need the functionality of a different subclass of TestCase, you have plenty of reasons to use GroovyTestCase and no reasons to specifically avoid it. Along with the assertions listed in table 17.1, it’s easier to work with GroovyTestCase than TestCase, as you’ll see in the next section.

5 These methods extend the 4.12 version of JUnit, which is bundled with Groovy.

17.1.3. Working with GroovyTestCase
To use Groovy’s enhanced TestCase class, extend it as follows:[6]

6 You don’t have to import it—it resides in one of the packages imported by default.

class SimpleUnitTest extends GroovyTestCase {
   void testSimple() {
     assertEquals("Groovy should add correctly", 2, 1 + 1)
   }
}
You can also use the JUnit 4 @Test annotation or TestNG’s equivalent to mark your test. In that case, you don’t have to extend from GroovyTestCase unless you want to use GroovyTestCase’s convenience methods, nor do you have to start your test method with the “test” prefix:

import org.junit.Test
import static org.junit.Assert.assertEquals
class SimpleUnitTest {
   @Test
   void shouldAdd() {
     assertEquals("Groovy should add correctly", 2, 1 + 1)
   }
}
Remember, you’re free to extend any TestCase class you choose, so long as it’s in your classpath. You can easily extend JUnit’s TestCase as follows:

import junit.framework.TestCase

class AnotherSimpleUnitTest extends TestCase {
   void testSimpleAgain() {
     assertEquals("Should subtract correctly too", 2, 3 - 1)
   }
}
Test cases can be run via the groovy command you’ve used previously for scripts and applications. For example, you can run the SimpleUnitTest script seen earlier, by typing the command groovy SimpleUnitTest:

> groovy SimpleUnitTest
.
Time: 0

OK (1 test)
If the output looks familiar, that’s probably because it is the standard JUnit output you’d expect to see if you ran a normal Java JUnit test using JUnit’s text-based test runner.

Now that you’ve got your feet wet, let’s go back and start again from scratch, this time testing a little more methodically.

## 17.2. UNIT TESTING GROOVY CODE
We’ve introduced Groovy’s testing capabilities, but we skipped over some of the details. We’ll now explore more of those details by going through a slightly larger Groovy application in need of testing. We’ll start with a new example and build up our test class, refactoring tests as we go, validating boundary data, testing that inputs aren’t inadvertently changed, even checking that the tests themselves haven’t been adversely changed.

Let’s imagine we’ve built a small counter class that determines how many numbers in a list are larger than a target threshold number. The Groovy code is fairly trivial but useful as our example class under test:

class Counter {
    int biggerThan(items, threshold) {
        items.grep{ it > threshold }.size()
    }
}
Testing this class is easy. First, we define our test case class, CounterTest, which extends GroovyTestCase:

class CounterTest extends GroovyTestCase {
    ...
}
Next, we follow the common unit-testing practice of writing a method to set up the variables we’ll need in the tests that follow:

class CounterTest extends GroovyTestCase {
    private counter
    void setUp() {
        counter = new Counter()
    }
    ...
}
We’re now in a position to write a test:

void testCounterWorks() {
  assertEquals(2, counter.biggerThan([5, 10, 15], 7))
}
We could continue adding tests in this way, but first let’s introduce constants that capture useful boundary case data and refactor out a helper method:

static final Integer[ ] NEG_NUMBERS   = [-2, -3, -4]
static final Integer[ ] POS_NUMBERS   = [ 4,  5,  6]
static final Integer[ ] MIXED_NUMBERS = [ 4, -6,  0]
private check(expectedCount, items, threshold) {
    assertEquals(expectedCount,
                 counter.biggerThan(items, threshold))
}
This lets us specify more tests in a compact form:

void testCountHowManyFromSampleNumbers () {
    check(2, NEG_NUMBERS, -4)
    check(2, POS_NUMBERS, 4)
    check(1, MIXED_NUMBERS, 0)
    ...
}
Once you’ve written sufficient tests to cover all the boundary cases you think are important (or to meet your project’s coverage requirements, discussed in section 17.7.1), you may think you’re finished, but you can do more. First, you can ensure that your method doesn’t change the input items. You can provide the correct answer but accidentally modify the input data and cause errors to occur elsewhere. Here’s one example of such a test:



You can add items[0] = 0 as the first line of the biggerThan method to show how this test would pick up an accidental bug in the code.

We now have sound tests in place, but we can be more paranoid about our test data and introduce a final test. Over time, we expect further developers to work on the code, and they’ll likely change the test constants. To ensure that our key cases remain covered, we can create a test that validates our assumptions about the data:



This ensures that our positive, negative, and mixed numbers retain the properties we intend.[7]

7 You could argue that we are being too paranoid here. Maybe, but it gives us a chance to show off a few more example test assertions.

Now for a neat bit of Groovy magic. It turns out that even though we set out to create a calculator for numbers, nothing in our original method was specific to numbers. We’ll add another test to illustrate this, using strings with their natural order:

void testCountHowManyFromSampleStrings() {
  check(2, ['Dog', 'Cat', 'Antelope'], 'Bird')
}
Putting this together results in the code in the following listing.

Listing 17.2. A complete test example, including implementation at the end




Looks familiar, doesn’t it? It’s darn close to normal JUnit test code, but with slight improvements thanks to Groovy’s extra assert methods, proper closure support, and more compact syntax. Groovy hasn’t made the code much shorter here, only a bit more convenient. As is often true, there’s more test code than production code (although in this case, the difference is more pronounced than usual).

It’s immediately obvious that Groovy code can test Groovy code, but it may not be as clear that you can test your existing Java using the benefits of GroovyTestCase, too. You’ll see this in action in the next section.

## 17.3. UNIT TESTING JAVA CODE
At this point in your career, you’ve probably coded more Java applications than Groovy ones. It stands to reason that one of the quickest ways to experience the pleasures of Groovy is to use this nifty language to test normal Java applications. As it turns out, this process is amazingly simple.

Using Groovy to test normal Java code involves three steps:


1.  Write your tests in Groovy.

2.  Ensure that the Java .class files you wish to test are on the classpath.

3.  Run your Groovy tests in the normal way (on the command line or via your IDE or favorite build environment).

That’s it most of the time. Of course, there are more complicated scenarios. If you’re running a complicated integration test and want to run your Groovy test code on a server, you can always run groovyc on your test code and then follow the same steps that you’d go through for a Java application.

Let’s explore this further by looking at an example. Rather than spend time describing a Java application that you may not have seen before, we’ll consider how to write tests for one of the Java collection classes: HashMap.

One of the first things you’d do if you wrote Java tests for HashMap is set up test fixtures. You do the same thing in Groovy, but you have Groovy’s convenient syntax to make your tests shorter and easier to understand. This is how we set up our test fixtures for an arbitrary key object and a sample map:

static final KEY = new Object()
static final MAP = [key1: new Object(), key2: new Object()]
One of the complicated things to test with Java-based tests is proper exception handling. Groovy’s built-in shouldFail assert method can be of great assistance for such tests. It’s part of HashMap’s expected behavior to disallow a null value when constructing the HashMap. Creating a HashMap by passing in a null value as in new HashMap(null) should lead to a NullPointerException. The shouldFail method asserts that this exception is thrown from within its closure:

void testHashMapRejectsNull() {
    shouldFail(NullPointerException) {
        new HashMap(null)
    }
}
If the attached closure fails to throw any exception (say you accidentally left out null as the parameter), the test would fail with a message like:

junit.framework.AssertionFailedError: testHashMapRejectsNull() should
have failed with an exception of type java.lang.NullPointerException
If the closure fails but with an incorrect exception, say you accidentally had -1 instead of null as the parameter, the test would fail with a message like:

testHashMapRejectsNull() should have failed with an exception of type
java.lang.NullPointerException, instead you got exception
java.lang.IllegalArgumentException: Illegal initial capacity: -1
The shouldFail method (inherited from GroovyTestCase) additionally returns the exception message so you can test that the correct message is generated by the exception, as in the following example:

void testBadInitialSize() {
    def msg = shouldFail(IllegalArgumentException) {
        new HashMap(-1)
    }
    assertEquals "Illegal initial capacity: -1", msg
}
If the incorrect exception message was returned (say you accidentally had -2 in the constructor call instead of -1), your test fails with a message similar to the following:

junit.framework.ComparisonFailure:
expected:<... initial capacity: -[1]>
but was:<... initial capacity: -[2]>
Groovy’s object-inspection methods (see section 9.1.1 for further details) also prove useful for writing our Groovy tests. Here is how you might use dump:

assert MAP.dump().contains('java.lang.Object')
We can put all this together into a complete test class as shown in the following listing.

Listing 17.3. Testing HashMap from Groovy




None of the behavior here is unexpected—after all, the classes we’re testing are familiar ones. Using shouldFail is more compact and readable than the equivalent in Java with a try/catch, which fails if it reaches the end of the try block. It’s also safer than the JUnit4 annotation for exception testing, which only checks whether anything in the method throws the desired exception, rather than the line of code we want to check.

The use of dump in this test isn’t as elegant as it tends to be in real testing. When you know the internal structure of the class, you can perform more useful tests against the introspected representation.

The final point we’ll mention about using Groovy to test your Java code is related to the agile software development practice of test-driven development (TDD).[8] Using this practice, code is developed by first writing a failing test and then writing production code to make that test pass, followed by refactoring and then repeating the process. Modern IDEs provide strong support for this practice, by offering to automatically create a nonexistent class mentioned in a test.

8 See Test-Driven Development: By Example by Kent Beck (Addison Wesley, 2002) and Test Driven Practical TDD and Acceptance TDD for Java Developers by Lasse Koskela (Manning, 2007).

You can still adopt TDD using a hybrid Groovy/Java environment. Current IDEs provide decent support to assist making this as streamlined as for pure Java environments.

Having considered individual test classes, you’ll now see how to run sets of tests together.

## 17.4. ORGANIZING YOUR TESTS
Until now, we’ve run our Groovy tests individually. For large systems, tests typically aren’t run individually but are grouped into test suites that are run together. Or sometimes you want to run the same test but with multiple (perhaps large) sets of data. We’ll look at ways to create suites, write parameterized or data-driven tests, and use property-based testing. These techniques let you scale up your testing ambitions. We’ll start with test suites.

### 17.4.1. Test suites
JUnit has built-in facilities for working with suites. These facilities allow you to add individual test cases (and other nested suites) to test suites. JUnit’s test runners know about suites and run all the tests they contain. Unfortunately, these facilities require you to manually add all of your tests to a suite and assume you’re using Java classes for your tests. We’ll look at ways of making life easier with Groovy.

Because grouping tests into suites is so important, numerous solutions have popped up in the Java world for automatically creating suites, but these too typically assume you’re using Java classes. The good news is that because Groovy classes compile to Java classes, you don’t have to abandon any of your current practices for grouping tests—as long as you’re willing to compile your Groovy files using groovyc first. The even better news is that solutions exist that allow you to work more naturally directly with your Groovy files.

First, we should mention GroovyTestSuite, which is a Java class. It allows you to invoke Groovy test scripts from the command line as follows:

> java groovy.util.GroovyTestSuite src/test/Foo.groovy
GroovyTestSuite, because it is a Java class, can be used with any conventional Java IDE or Java build environment for running JUnit tests. It allows you to add Groovy files into your test suites, as shown in the following listing. This creates a suite containing the two previous tests. You could also add Java tests to the same suite.

Listing 17.4. Adding Groovy scripts to a JUnit suite with GroovyTestSuite
import junit.framework.*
import junit.textui.TestRunner

static Test suite() {
  def suite = new TestSuite()
  def gts = new GroovyTestSuite()
  suite.addTestSuite(gts.compile("Listing_17_02_CounterTest.groovy"))
  suite.addTestSuite(gts.compile("Listing_17_03_HashMapTest.groovy"))
  return suite
}

TestRunner.run(suite())
We create a normal JUnit TestSuite and call GroovyTestSuite’s compile method to compile the Groovy source code so that TestSuite knows how to run it. We then use the normal JUnit console UI to run the tests. It isn’t aware that it’s running anything other than normal Java.

Next, we look at AllTestSuite, which can be thought of as an improved version of GroovyTestSuite. It allows you to specify a base directory and a filename pattern, and then it adds all the matching Groovy files to a suite. The following listing shows how you’d use it to run the same tests as we did in listing 17.4.

Listing 17.5. Adding Groovy scripts to a JUnit suite with AllTestSuite
def suite = AllTestSuite.suite(".", "Listing_17_*Counter*Test.groovy")
junit.textui.TestRunner.run(suite)
This time, we use the return value of the suite method directly, but if we want to add multiple directories or patterns, we can call suite multiple times, adding the tests to a suite before running them all together.

We’ll have more to say about grouping tests into suites and running test suites when we look at IDE, Gradle, and Maven integration later in this chapter, but first let’s look at scaling up your test input data.

### 17.4.2. Parameterized or data-driven testing
JUnit 4, TestNG, and Spock (which we’ll cover shortly) all provide facilities for data-driven tests. The following listing shows how to write such a test for JUnit 4.

Listing 17.6. Using Parameterized data with JUnit 4




JUnit uses the Parameterized test runner  to invoke data-driven tests. The @Parameter annotation  earmarks an array of test data. The test class constructor  is parameterized so that its parameters match one row of test data.

Our example used a hard-coded array of test values, but there’s no reason this couldn’t have come from a file, Excel spreadsheet, or database. TestNG and Spock also have similar capabilities. Before leaving this topic, we want to look at a technique you can use to write many fewer tests with potentially large sets of auto-generated test data.

### 17.4.3. Property-based testing
When applying agile developer practices, such as TDD, and working with imperative code, we often end up playing a little game. We try to work out the minimum tests we can write to steer the design of the production code being produced to have the desired “business” functionality but also to achieve 100% code coverage.[9] It makes perfect sense. Because we know a little bit about our implementation’s internal workings, we craft our tests to cover every branch—effectively validating the assumptions we make in each and every part of our implementation. An effective pair programmer will try to bring any hidden assumptions about the implementation to the surface. Best to deal with such assumptions up front and not when they become an issue in production.

9 For the impatient, you don’t have long to wait to learn more; we’ll cover that topic in the next section.

When working with functional languages this practice is rarely used, at least not in the same way. A slightly different slant is often taken on the testing process. Instead of trying to test assumptions about inner workings, the focus is about validating the external behavior of the code. Are there any hidden assumptions about its behavior that might cause unexpected results in the future? In this context, property-based testing as a concept has arisen and flourished.

With property-based testing, we try to establish expected properties of part of our system. We then hit it with a large amount of input data and see if those properties hold. As an example consider this code:

@Grab('net.java.quickcheck:quickcheck:0.6')
import static net.java.quickcheck.generator.PrimitiveGenerators.*
import static net.java.quickcheck.generator.CombinedGeneratorsIterables.*

for (words in someNonEmptyLists(strings())) {
    assert words*.size().sum() == words.sum().size()
}
Here, we use two inbuilt generators from the QuickCheck for Java library: one that produces arbitrary strings and another one that produces non-empty lists of items from another generator. Putting them together, we get lists of non-empty strings.

The property or invariant that we want to hold is that if we take any such list and concatenate all the strings and find the length of the concatenated string, then we should get the same value that we would obtain by finding the length of each individual string and summing the lengths together. This makes sure nothing is lost (or incorrectly added) when we concatenate strings together.

We can use the same library and apply that same concept to our temperature converter as shown in the following listing.

Listing 17.7. Property-based testing using QuickCheck for Java with Groovy


We use a generator, in this case for integers , to provide a random set of test values for the temperature in Fahrenheit. Then looping 100 times, we obtain the next temperature from the generator  and pass it through the converter.

In general at this point, we need a way to determine if the result we got was correct. We manually created the expected value when doing TDD, but here we won’t have that option. We could have an oracle of some kind, perhaps another algorithm we know produces the correct result but may be too slow to use in production, or perhaps a database of correct answers is available. But in general with property-based testing, we give up on the goal of trying to validate fixed values. Instead we’ll check that certain properties hold.

For our converter we’ll check two properties:

That the Celsius value is smaller than or equal to the Fahrenheit value  which holds true for temperatures above -40 degrees. That conveniently matches what our generator is producing.
That any Fahrenheit temperature and its converted Celsius value pass a little sanity check. We know the range of temperatures in which water is a liquid for both Fahrenheit and Celsius scales.[10] A Fahrenheit temperature corresponding to the liquid phase, once converted, should be in the liquid range we know for Celsius and vice versa .
10 We’re keeping the underlying science nice and simple for this example.

Each time we run the test, it uses 100 different random test values, so over time we’ll get increasingly good data coverage of our system.[11] While our water liquidity test might seem a little strange, it reveals the nature of property-based testing. It’s up to you to determine what properties are important in your system.

11 And if needed, many property-based libraries have ways to seed the randomness for repeatable tests.

We used simple inbuilt generators in our examples. In general, if you dive into property-based testing, you’ll likely create your own generators and combine your own generators with the provided ones to build composite generators. The end result of using property-based testing is that you’ll end up having much fewer data values encoded in your tests, which eases refactoring and maintenance.

This approach to testing can feel “harder” as long as its underlying functional thinking is unfamiliar. The benefit is that you’re led to detect behavioral characteristics of your system by making them explicit in your tests. Writing such tests is often an enlightening experience.

As you can see, property-based testing is a powerful technique available in our testing toolbox. Speaking of useful techniques, let’s examine a few more advanced techniques, which you’ll also want to stash away in a corner of your Groovy testing toolbox.

## 17.5. ADVANCED TESTING TECHNIQUES
Let’s switch into “Groovy expert mode” and look at advanced testing techniques. Several of the techniques will help you test hard-to-test systems by leveraging Groovy’s dynamic nature. You’ll also learn how to gain knowledge about your test coverage and about the performance of small parts of your system in isolation. Let’s start with exploring why systems can be hard to test.

Automated testing is easy if you develop your automated tests in close interplay with your production code, because you immediately design your system for testability. Unfortunately, this level of test awareness isn’t universal, and you’ll sometimes find yourself in the position where you have to write tests for code that already exists. This is when you need advanced testing techniques, the same way you’d need a more specialized tool than a dinner fork to efficiently extract a single strand of spaghetti from a bowl of pasta.

A number of bad programming habits can make testing difficult. One is writing incoherent classes and methods that do more than they should, resulting in overly long classes and methods. Even worse is code with many dependencies to other classes that we’ll call collaborators. Unit testing your subject under test (SUT) in its purist form means that you test it in isolation without the collaborators so you’re focused on finding errors in your code.[12]

12 Other kinds of integration tests should pick up errors that come from integrating your code with the collaborators.

The first set of advanced testing techniques we’re about to explore is mainly concerned with replacing such collaborators for the purpose of unit testing the SUT in isolation. To that end, we first show how you can employ Groovy’s core language features to provide “fake” collaborators. We then explore Groovy’s special support for so-called stubs and mocks, which allow flexible simulation of collaborator behavior, as well as let you specify exactly how the collaborators must be used. We finish our first wave of techniques by considering an approach that can be used when all else fails: using logs to test that your classes are behaving as you expect them to.

### 17.5.1. Testing made groovy
Once, I (Dierk) gave a lecture on unit testing where I asked the audience to challenge me with the most difficult testing problem they could think of, something they believed would be impossible to unit test. Their proposal was to test the load-balancer of a server farm. How could we test this in Groovy?

The core logic of a load balancer is to relay a received request to the machine in the server farm that currently has the lowest load. Suppose we already have collaborator classes that describe requests, machines, and the farm; a Groovy load balancer could have the following method:

def relay(request, farm) {
    farm.machines.sort { it.load }[0].send(request)
}
The method finds the machine with the lowest load by sorting all machines in the farm by the load property, taking the first one, and calling the send method on that machine object.

To unit test this logic, we need to somehow call the relay method to verify its behavior. We can do this only if we have request and farm objects, but we don’t want our test to depend on any of the production collaborator classes. Luckily, our Groovy solution doesn’t demand any specific types, and we can use any type we fancy.

What would be a good object to use for the farm parameter? Thanks to Groovy’s duck typing of the relay parameters, any object that we can ask for a machines property will do—a map for example. The machines property, in turn, needs to be something that can be sorted by a load property and understands the send(request) method. Listing 17.8 follows this route by testing the load balancer logic with a map-based farm of fake machines made using a FakeMachine class. Fake machines return a self-reference from their send method to allow subsequent asserts to verify that the send method was called on the expected machine.

Listing 17.8. Unit-testing a load balancer with Groovy collaborator replacements


Note that we don’t need to create a special stub for the request parameter. Because it’s relayed and no methods are ever called on it, null is fine.

The important point about the previous listing is that the load-balancing logic is tested in full isolation. No accidental change to any of the collaborator classes can possibly affect this test. When this test fails, we can be sure that the load-balancing logic and nothing else is in trouble.

### 17.5.2. Stubbing and mocking
Until now, our load balancer was fairly easy to test in isolation because we could feed all collaborator objects into the relay method. That wasn’t a real challenge. Things get more interesting when we need to replace objects that cannot be set from the outside.[13]

13 In UML terms: when the collaborator is composed, not aggregated.

Example problem: collaborator construction
Suppose our load balancer directly creates its collaborator farm object:

def relay(request) {
    new Farm().getMachines().sort { it.load }[0].send(request)
}
The Farm class looks like this:

class Farm {
    def getMachines() {
        /* some expensive code here */
    }
}
From an implementer’s perspective, such a solution could be justifiable for a number of reasons. Perhaps the Farm’s getMachines method provides support for finding all machines via a network scan and then caches that information. Anyway, we wouldn’t want to perform an expensive operation if we didn’t need it, so placing the new Farm().getMachines() statement within relay seems like the way to go. From a tester’s perspective, however, even allowing for potential caching, calling the real code is going to be too expensive an operation for a unit-test environment, where tests should execute in the blink of an eye if developers are expected to run them often. Also, we need to run our tests even when no real machines are available.

The implementation isn’t easily testable. We can’t use the fake implementation techniques in the way you saw earlier, because we have no way to sneak such a subclass into our subject under test. One common trick when testing would be to subclass Farm. That won’t help us here either, for the same reasons. Should we give up? No!

Stubbing out the collaborator
Groovy’s Meta-Object capabilities come to the rescue in the form of Groovy stubs. The trick provided by Groovy stubs is to intercept all method calls to instances of a given class (Farm in this case) and return a predefined result. Here’s how it works.

We first construct a stub object for calls to the Farm class:

import groovy.mock.interceptor.StubFor

def farmStub = new StubFor(Farm)
Next, we create two fake machines to help define our expectations from the stub:

def fakeOne = new Expando(load:10, send: { false } )
def fakeTwo = new Expando(load:5,  send: { true }  )
Then, we demand that when the getMachines method is called on our stub, our fake machines are returned. Registering this behavior is done by calling the respective method on the stub’s demand property and passing a closure argument to define the behavior:

farmStub.demand.getMachines { [fakeOne, fakeTwo] }
Finally, we pass our test code as a closure to the stub’s use method. This ensures that the stub is in charge when the test is executed: any call to any Farm object will be intercepted and handled by our stub. The full test scenario is given in the following listing.

Listing 17.9. Using Groovy stubs to test an otherwise untestable load balancer


Note that for the use of Groovy stubs, it makes no difference whether the collaborator class is written in Java or Groovy. The class under test, however, must be a Groovy class.

Stub expectations
Groovy stubs support a flexible specification of the demanded behavior. To demand calls to different methods, do so in sequence:

someStub.demand.methodOne { 1 }
someStub.demand.methodTwo { 2 }
When calls to the stubbed method should yield different results per call, add the respective demands in sequence:

someStub.demand.methodOne { 1 }
someStub.demand.methodOne { 2 }
You can provide a range to specify how often the demanded closure should apply; the default is (1..1):

someStub.demand.methodOne(0..35) { 1 }
Finally, it’s also possible to react to the method argument that the SUT passes to the collaborator’s method. Each argument of the method call is passed into the demand closure and can thus be evaluated inside it. Suppose you expect that the stubbed method is called only with even numbers, and you’d like to assert that invariant while testing. You can achieve this with

someStub.demand.methodOne {
    number -> assert 0 == number % 2
    return 1
}
Of course, you can also combine all these kinds of demand declarations, producing an elaborate specification of call sequences on the collaborator and returned values. The more elaborate that specification is, the more likely it is that you’ll want to also assert that all demanded method calls happened. For stubs, this isn’t asserted by default, but you can enforce this check by calling

someStub.expect.verify()
after the use closure.

Stubs use a LooseExpectation for verifying the demanded method calls. It’s called loose because it only verifies that all demanded methods were called, not whether they were called in the sequence of the specification.

Comparing stubs and mocks
Strict expectations are used with mocks. A mock object has all the behavior of a stub and more. The strict expectation of a mock verifies that all the demanded method calls happen in exactly the sequence of the specification. The first method call that breaks this sequence causes the test to fail immediately. Also, with mocks you have no need to explicitly call the verify method, because that happens by default when the use closure ends.

At first glance, it appears that mocks and stubs are almost the same thing, with mocks being a bit more rigorous. But a deep fundamental difference exists in the purpose behind their use:[14] Stubs enable your SUT to run in isolation and allow you to make assertions about state changes of the SUT. With mocks, the test focus moves to the interplay of the SUT and its collaborators. What gets asserted is whether the SUT follows a specified protocol when talking with the outside world. A protocol defines the rules that the SUT has to obey when calling the collaborator. Typical rules would be: the first method call must be init, the last method call must be close, and so on.

14 See www.martinfowler.com/articles/mocksArentStubs.html for more details.

Consider a new variant of our load balancer that uses a SortableFarm class, which provides a sort method to change its internal representation of machines such that any subsequent call to getMachines returns them sorted by load:

class SortableFarm extends Farm {
    def sort() {
        /* here the Farm would sort its machines by load */
    }
}
Our SUT now has to follow a certain protocol when using SortableFarm: first sort must be called, and then getMachines:

def relay(request) {
    def farm = new SortableFarm()
    farm.sort()
    farm.getMachines()[0].send(request)
}
Listing 17.10 uses a mock as constructed with the MockFor class to verify that our SUT exactly follows this protocol. Only the compliance to the protocol is tested and nothing else; for this special test, we don’t even verify that the call is relayed to the machine with the lowest load.

Listing 17.10. Using Groovy mock support to verify protocol compliance


If you’re unfamiliar with mock objects, protocol-based testing (also called interaction-based testing) will probably appear strange to you. In traditional testing, we tend to focus on state changes and return values rather than on the effects caused to collaborating objects. In particular cases, interactions with collaborators are implementation details and shouldn’t be tested. If they are part of the object’s guaranteed behavior, mock testing is appropriate.

Groovy’s clever way of providing stubs and mocks even for objects that cannot be passed to the SUT is a double-edged sword. Testing should lead you into a design of high coherence and low coupling. Without resorting to clever Java tricks, Java mocks work only if you can pass them to the SUT, forcing you to expose the collaborator, which usually leads to a more flexible design. Groovy has no such restriction, because you can more easily test even a rotten design. The implication is that Groovy won’t stop you from building a less-flexible design even when using the latest development practices.

Conversely, Java projects often suffer from the deadlock that appears when developers find large sections of untestable code. They cannot easily refactor such a section of code because it has no tests. They cannot easily write tests without refactoring the code to make it more testable. With Groovy’s built-in mocking facilities, you have a better chance of escaping this deadlock.

### 17.5.3. Using GroovyLogTestCase
Sometimes, even with stubs and mocks, testing a particular object can be difficult. The amount of work involved in setting up all the mocked interactions in a tricky scenario may outweigh the benefits of your testing efforts. To be realistic, if your system (and resulting tests) is that complex, perhaps you have a bug in your tests. In such cases, another useful feature provided by Groovy is GroovyLogTestCase. You saw in listing 17.2 that it was relatively easy to test the fictitious countHowManyBiggerThan calculator. Suppose, though, that it was much harder to test. We could resort to writing information to a log file, and then we could manually check the log file to see if it appears to contain the correct information. In these scenarios, GroovyLogTestCase can be extremely useful. Consider the following modified LoggingCounter:

import java.util.logging.*

class LoggingCounter {
    static final LOG = Logger.getLogger('LoggingCounter')
    def biggerThan(items, target) {
        def count = 0
        items.each{
            if (it > target) {
                count++
                LOG.finer "item was bigger - count this one"
            } else if (it == target) {
                LOG.finer "item was equal - don't count this one"
            } else {
                LOG.finer "item was smaller - don't count this one"
            }
        }
        return count
    }
}
Note that the calculator outputs log messages for each of three scenarios: the item being tested was smaller than, equal to, or bigger than the target value. We can now test this class with the assistance of GroovyLogTestCase, as shown in the following listing.

Listing 17.11. Using GroovyLogTestCase for tricky cases


If you look at the test data in the MIXED_NUMBERS list, you expect four entries to be bigger than -1, two to be smaller, and one to be the same. Log messages corresponding to these cases will be stored in the log variable thanks to the stringLog statement. Our test then uses regular expressions to ensure that the log contains the correct number of each kind of log message.

GroovyLogTestCase makes use of the Log String testing pattern[15] in a test scenario that would otherwise be cumbersome and error-prone to implement. It relieves you of the work of setting the appropriate log levels and registering string appenders for the SUT logger. After the test, it cleans up properly and restores the old logging configuration.

15 Described in chapter 27 of Test-Driven Development: By Example.

That finishes our wave of techniques for tackling hard-to-test systems. Up next we look at performance testing.

### 17.5.4. Unit testing performance
There may be many performance characteristics of your system that are important and worthy of being tested. But in the context of this chapter, we’ll limit ourselves to looking at one library that lets you perform simple load and performance tests at the unit test level. JUnitPerf is an extension framework for JUnit that offers the ability to ascertain fine-grained performance and scalability of your objects and its methods. For instance, JUnitPerf enables scenarios such as “the findTrades method must return a list of Trade objects within one second and the test fails if it performs too slowly (even if the test did return a valid list of Trade objects).” The framework also adds scalability via threading. Using this scenario, you can add the requirement that under a load of 100 invocations, the findTrades method must return a collection of Trade objects within one second.

Understandably, there are scenarios within Groovy where this type of framework could come in handy:

Testing the performance and scalability of Groovy applications
Testing the performance and scalability of normal Java code in tests written in Groovy
Using JUnitPerf with Groovy can be a little tricky. JUnitPerf is a decorator-based framework. It decorates test cases by individually wrapping them with a decorator. This is typically done within a suite method. Groovy’s GroovyTestSuite and AllTestSuite test runners, however, ignore suite definitions and provide alternative mechanisms for determining which tests to run.

To allow JUnitPerf to work with Groovy involves following a few simple steps. First, you need a way to select a single JUnit test that you want to decorate. If you look at JUnit’s TestCase class, you’ll notice that it provides a constructor that takes the name of a test method and allows a single test case to be selected. We can make use of this for JUnitPerf by declaring a constructor that takes a method name and have it call super(testName):

Listing_17_12_JUnitPerf(String testName) {
  super(testName)
}
Then we create a suite method that defines a test case using this constructor:

static Test suite() {
  def testCase = new Listing_17_12_JUnitPerf("testConverter")
  // decorate testCase and return decorated version
}
Now we can apply the appropriate decorators on the test case according to JUnitPerf’s documentation for load and stress testing scenarios:

def loadTest = new LoadTest(testCase, numUsers, stagger)
It sounds complicated, but really it’s the same steps you’d follow to use JUnitPerf in Java. As an example, the next listing utilizes JUnitPerf to test our temperature converter. It verifies that invoking testConverter 20 times in concurrent threads (with each thread staggered by 100 milliseconds) returns within 2100 milliseconds.

Listing 17.12. Using JUnitPerf decorators to perform load and time tests

When you run this program, you should see output indicating that the program is running your tests, followed by the time it took to complete the tests. Because there are 20 users starting 100 ms apart, we expect the test to run for at least 2 seconds. If the time is less than 2.1 seconds, then the test will be successful:

....................TimedTest (WAITING): LoadTest (NON-ATOMIC):
ThreadedTest: testConverter(Listing_17_12_JUnitPerf): 2014 ms
Time: 2.014
OK (20 tests)
If the test takes too long to run (suppose we expect it to complete in 2.01 seconds), it will fail:

There was 1 failure:
1) LoadTest (NON-ATOMIC): ThreadedTest:
testConverter(Listing_17_12_JUnitPerf)junit.framework.AssertionFailedError:
Maximum elapsed time exceeded! Expected 2010
ms, but was 2020ms.
The next time you need to figure out the performance of Groovy code or you want to test the performance and scalability of your Java application with Groovy, give JUnitPerf a try!

### 17.5.5. Code coverage with Groovy
Code-coverage tools are now a mainstream part of any serious Java engineer’s toolkit. They provide useful feedback on how well your testing efforts are going. To leverage any existing Java code coverage tool for Groovy, you need to compile your Groovy into bytecode and then run the tool as before.

If you’re interested in the coverage of your Groovy code and you try this technique with an older coverage tool, you’ll probably not have the ability to see reports indicating which lines of code were executed, because the tool or its reporting infrastructure doesn’t know about Groovy source files.

The good news is that efforts are being made to provide native Groovy support in code-coverage tools. One open source tool that has gained Groovy support is Cobertura (http://cobertura.sourceforge.net).

Cobertura works in a similar way to many other coverage tools for the JDK.[16] During the build process, it modifies our bytecode so that later when our code executes, it will write out information about which code paths have been executed. This information will be stored away in a form suitable for later processing by the coverage tool reporting.

16 But tools such as Clover actually hook into Groovy’s compiler phases.

Consider the following Groovy class:[17]

17 If we were trying to be really Groovy, we could write [a,b,c].sort()[-2..-1].sum(), but that would have made it harder to show some lines covered and some not!

class BiggestPairCalc {
  int sumBiggestPair(a, b, c) {
    def op1 = a
    def op2 = b
    if (c > a) {
      op1 = c
    } else if (c > b) {
      op2 = c
    }
    return op1 + op2
  }
}
Here’s a test for this code:

class BiggestPairCalcTest extends GroovyTestCase {
    void testSumBiggestPair() {
        def calc = new BiggestPairCalc()
        assertEquals(9, calc.sumBiggestPair(5, 4, 1))
    }
}
At this stage, we could run our test and make sure it passes. To get coverage, however, requires a few extra steps. We used a Gradle build file to capture these steps. The entire build file, build.gradle,[18] is fairly simple and looks like the following:

18 Within the cobertura directory of the book’s sample code.

plugins {
  id 'net.saliman.cobertura' version '2.2.6'
}

apply plugin: 'groovy'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.codehaus.groovy:groovy-all:2.4.0'
  testCompile 'junit:junit:4.12'
}
The Cobertura plugin makes numerous tasks available to Gradle. The one we want is gradle cobertura. This will compile our code, execute our tests, and generate a Cobertura report as shown in figure 17.1. Note that nothing special was required to get the Groovy coverage. All Java classes (if any) and Groovy classes in our project will be part of the coverage analysis.

Figure 17.1. Cobertura code-coverage summary report


If we go deeper into the report by clicking an appropriate link for one of our source files, we can see which lines are covered by tests. Figure 17.2 shows that lines 6 and 8 are not covered yet by tests and 5 and 7 are only partially covered.

Figure 17.2. Cobertura code-coverage file report showing partial coverage


Now that we can see where we’re missing coverage, we can add more tests to our test method:

assertEquals(15, calc.sumBiggestPair( 5, 9, 6))
assertEquals(16, calc.sumBiggestPair(10, 2, 6))
We can run the tests to make sure they all still work and then check the coverage again to see how our coverage is going. The result is shown in figure 17.3.

Figure 17.3. Cobertura code-coverage file report showing full coverage


Is the code correct? The tests all pass, and we have 100% coverage—that means we don’t have any bugs, right? For fun, let’s add one more test:

assertEquals(11, calc.sumBiggestPair(5, 2, 6))
If we run our tests again, they now fail! There was an error in our original algorithm. That was nothing to do with Groovy, but is a reminder that coverage is a necessary but not sufficient condition to show that you have all the tests that you need. We can fix the calculator as shown in figure 17.3.

int sumBiggestPair(int a, int b, int c) {
    int op1 = a
    int op2 = b
    if (c > [a,b].min()) {
        op1 = c
        op2 = [a,b].max()
    }
    return op1 + op2
}
Now we can run all our tests. They should all pass, and Cobertura should report 100% coverage.

You have seen that Groovy makes even advanced testing techniques easily available through core language features. The running theme of improving developer convenience with Groovy finds its logical continuation in the next section, where we integrate Groovy unit testing in Java IDEs.

## 17.6. IDE INTEGRATION
In section 1.6, you saw that several major Java IDEs (with the addition of plug-ins) have useful support for editing and running Groovy code. The same mechanisms are suitable for editing and running your Groovy tests. But the story doesn’t end there.

Java IDEs often have additional features to better support Java unit testing, such as enhanced test runners. Fortunately, you’ll see that many of these enhanced features can be leveraged for your Groovy unit testing. We explore how to use the two test suite classes you saw earlier within an IDE, before taking a brief look at how Groovy’s close relationship with Java allows it to be used with cutting-edge IDE testing features.

### 17.6.1. Using GroovyTestSuite
While editing a Groovy test file within your IDE, you can run it like any other Groovy file. Eclipse users with the Groovy plug-in installed might right-click, select Run As, and then select Groovy. IntelliJ IDEA users with the Groovy plug-in installed might press Ctrl-Shift-F10. In both cases, the corresponding tests within the current file would run. If your Groovy file was several assert statements in a script file, as in listing 17.1, then you wouldn’t see any output—this is expected because assert statements make noise only when something goes wrong. If you don’t want to run your tests individually or want additional feedback when running your tests, GroovyTestSuite may be what you’re after.

In section 17.4, you saw that GroovyTestSuite could be used to invoke a Groovy test from the command line. You also saw how it could be used to add Groovy files into a standard JUnit suite.[19] We now look at another way to use GroovyTestSuite: as part of an IDE run configuration. Figure 17.4 shows how to configure Eclipse to use GroovyTestSuite as part of a run configuration. Select Run -> Run, and create a new Java Application configuration. Set the Project to be your current project, and select groovy.util.GroovyTestSuite as the Main class.

19 Test suites remain an important concept you typically use in conjunction with other IDE integration.

Figure 17.4. Eclipse run configuration for Main tab using GroovyTestSuite


Next, click the Arguments tab; in the Program Arguments box, include the path to your Groovy script, as shown in figure 17.5.

Figure 17.5. Eclipse run configuration for the Arguments tab using GroovyTestSuite


When you run this configuration, you should see output similar to that shown in figure 17.6.

Figure 17.6. Eclipse GroovyTestSuite example run output


Users of JUnit’s text-based runner should now feel at home and will see a bit more feedback than the previously empty output.

### 17.6.2. Using AllTestSuite
JUnit’s green/red bar reporting mechanism found in graphical test runners can be addictive when you are “in the groove.” The default behavior of Groovy’s GroovyTestSuite, however, doesn’t easily fit into the graphical runner model, because those runners usually prefer to run normal Java classes, rather than Groovy files.

One strategy is to rely on groovyc to compile all test cases and then run them via a Java-aware GUI runner; however, that takes an extra step. It’s more fun to see the green bar immediately after coding! This is where AllTestSuite, which we discussed in section 17.7, really shines. In addition to its uses for organizing your tests into suites, AllTestSuite can also be used as part of configuring your test runs.

To configure Eclipse to use AllTestSuite, create a new JUnit run configuration, select your project, and set the Test class to groovy.util.AllTestSuite, as shown in figure 17.7.

Figure 17.7. Eclipse AllTestSuite run configuration Test tab


Then, in the Arguments tab, define two properties that tell AllTestSuite which Groovy tests to run. These properties need to be supplied as two VM Arguments. The properties need to be adjusted for your system but will look something like -Dgroovy.test.dir=. for the directory and -Dgroovy.test.pattern=Listing*Counter*.groovy for the filename pattern. Your configuration will be similar to that shown in figure 17.8.

Figure 17.8. Eclipse AllTestSuite run configuration Arguments tab


When you run this configuration, you should see the familiar green and red bars, as shown in figure 17.9. We don’t have time to illustrate how to set up other IDEs, but you’ll see IntelliJ IDEA screenshots when we cover Spock.

Figure 17.9. Eclipse AllTestSuite example test run output


Now that you’ve experienced how the IDE support for Groovy works, it’s time to explore one of Groovy’s most widely used testing framework—say hello to Spock.

17.7. TESTING WITH THE SPOCK FRAMEWORK
As you can see in the Groovy GDK chapter, Groovy gives us great support to more easily test our code using powerful constructs such as maps and Groovy’s dynamic nature. The flexibility of the Groovy language made things possible that are often hard to do in the Java programming language.

We can go further in this and create an even more readable and compact test. Meet Spock (www.spockframework.org), a testing and specification framework for Java and Groovy applications. Spock supports a diversity of testing styles and one of the most well-known approaches is a Behavior-Driven Development (BDD)[20] style approach, characterized by its Given-When-Then format. The following listing introduces the Spock Given-When-Then style.

20 For more information, see http://en.wikipedia.org/wiki/Behavior-driven_development.

Listing 17.13. A simple Spock specification
@Grab('org.spockframework:spock-core:1.0-groovy-2.4')
import spock.lang.Specification

class GivenWhenThenSpec extends Specification {

  def "test adding a new item to a set"() {
    given:
    def items = [4, 6, 3, 2] as Set

    when:
    items << 1

    then:
    items.size() == 5
  }
}
What’s most interesting to know is that the code in the previous listing is completely valid Groovy code and makes great use of the flexibility Groovy provides when writing software. Groovy labels are used to separate the Given-When-Then blocks. Also note that you can use spaces in the method name, a feature provided by Spock, which is implemented using an AST transformation.

In the setup part of the test, marked with the given label, we create a set of numbers. In the when part of the test we execute an action on our test subject. In our case, we add a number. Then, in the last part of the test, we check if the item has been added. Note that no assert statement is needed here to verify the result; all expressions in the then block are checked automatically for being true.

17.7.1. Testing with mocks
The test described in the previous section is a simple one. In the real world, the class you’re testing usually has dependencies on other classes that can make testing often a bit trickier. Luckily, Spock provides excellent mocking[21] support.

21 For more information, see “Mocks Aren’t Stubs,” http://martinfowler.com/articles/mocksArentStubs.html.

Consider a more complex example. Instead of testing java.util.Set’s, we’ll test the purchasing of movie tickets. It’s a fairly simple domain model with a MovieTheater and a Purchase class. In this example, we’ll mock out the MovieTheater and focus on the Purchase class.

Given those assumptions, let’s use an interface as follows for the MovieTheater:

interface MovieTheater {
  void purchaseTicket(name, number)
  boolean hasSeatsAvailable(name, number)
}
In a separate test, when testing the MovieTheater, we wouldn’t mock it out, but focus on the MovieTheater instead. In this case, the MovieTheater is a mock, and we’ll validate that the Purchase will call the right methods when buying a movie ticket.

What should happen when the fill method is called on an instance of the Purchase class is that we should check if the theater has availability and, if so, buy a ticket. A skeleton for such a class is as follows:

@TupleConstructor
class Purchase {
  def name, number, completed = false

  def fill(theater) {
    if (theater.hasSeatsAvailable(name, number)) {
      theater.purchaseTicket(name, number)
      completed = true
    }
  }
}
With these definitions in place, we can now write our test, as shown in the following listing.

Listing 17.14. Testing with mocks


In listing 17.14, you can see that the then block contains one assertion and one interaction. An interaction consists of four distinct parts: a cardinality, a target constraint, a method constraint, and an argument constraint:



Diving into all options here goes a bit beyond the scope of this chapter, but if you want to learn more about Spock interactions, you can do so in the excellent documentation online.[22]

22 For more information, see http://docs.spockframework.org/.

The next listing has a similar approach, but describes the case when all of the movies are sold out. There are no tickets for any movie. In the test, you see the usage of the Spock wildcard operator (_), which can be read as “any”. In our test we don’t care which movie is purchased; none are available, and because of this, no purchase can be completed.

Listing 17.15. Using wildcards to ignore arguments


To create a bit more flexibility, we can also use wildcard matchers in combination with Groovy closures. The closure can be used to check the argument passed to the method, failing the test if the argument doesn’t match. In listing 17.16, one such approach is demonstrated. Here we basically have the same scenario as listing 17.14 but this one runs as part of a special “couples night” scenario we’re trying to test. As part of this scenario, we might not care what movies are watched, only that the tickets are sold in pairs. We’ll ignore the first argument with a wildcard and check the second argument by injecting a closure  that will validate the argument passed to the purchaseTicket method, as seen in the following listing.

Listing 17.16. Argument checking with injected closures


### 17.7.2. Data-driven Spock tests
In section 17.4 we looked briefly at data-driven tests. Spock also supports that approach and in fact has a special notation for capturing such test case data elegantly, as can be seen in the following listing.

Listing 17.17. Data-driven testing
@Grab('org.spockframework:spock-core:1.0-groovy-2.4')
import spock.lang.*

import static Converter.celsius

class Listing_17_17_SpockDataDriven extends Specification {
  def "test temperature scenarios"() {
    expect:
    celsius(tempF) == tempC

    where:
    scenario                  | tempF || tempC
    'Freezing'                |    32 ||     0
    'Garden party conditions' |    68 ||    20
    'Beach conditions'        |    95 ||    35
    'Boiling'                 |   212 ||   100
  }
}
A little syntactic sugar is going on here. Note that the variables defined in the where clause are automatically available in the expect block. Also, there’s the use of the || symbol, which is used as a separator between input and output separators. Note that the || could have been replaced by the | symbol, but the || are used here to visually set them apart.

Reporting failures
But what if our implementation contains an error (we’ll momentarily alter one figure), and we want to get feedback about it? In that case, the following output is:

Condition not satisfied:

celsius(tempF) == tempC
|       |      |  |
35      95     |  34
            false
It’s a pretty clear explanation, but is it clear which iteration caused this error? Because we only have four iterations, it’s not that hard to find out that iteration three caused the error, and we can easily fix that. In other cases, for example when dealing with external data, or when we have lots of data, the solution might be less obvious. To handle this, we can use the @Unroll annotation.

@Unroll annotation
A method annotated with @Unroll will have its iterations reported independently. The test execution isn’t changed; the only thing that has changed is how Spock reports each test iteration. We can annotate the method in the following way:

@Unroll
def "test temperature scenarios"() { ... }
Executing the same test now using IntelliJ will produce the output shown in Figure 17.10.

Figure 17.10. IntelliJ IDEA Listing_17_17_SpockDataDriven test unrolled.


As you can see in the figure, each test is now reported separately. With a small change, we can do even better though:

@Unroll
def "Scenario #scenario: #tempFºF should convert to #tempCºC"() { ... }
When running the test now (again in IntelliJ), the report produced now looks as shown in Figure 17.11.

Figure 17.11. IntelliJ IDEA Listing_17_17_SpockDataDriven test unrolled with parameter values.


Now, the @Unroll annotation uses the method variables from the test. Using the placeholders, we can tell at a glance which iteration is causing the problem and what our actual expectation would be.

As demonstrated in this section, Spock provides a flexible way of writing tests and should be considered when writing code on a software project, be it in Java or in Groovy. Making sure your software works correctly is crucial and writing tests for it is one of the best ways to assert that your software behaves the way you want it to, and that it keeps doing that. To do so, we can make use of build automation, which is described in the next section.

## 17.8. BUILD AUTOMATION
We looked at how to run tests individually or in suites from the command line and using IDEs. For a team environment, however, the automated build environment should also run all the tests.[23] Two of the more popular build automation technologies in the Java world are Gradle and Maven. We’ll look briefly at how to integrate Groovy with each of these technologies.

23 See Pragmatic Project Automation: How to Build, Deploy, and Monitor Java Apps by Mike Clark (The Pragmatic Programmers, 2004) for more details on why this is important.

### 17.8.1. Build integration with Gradle
Gradle is a new and flexible build automation tool, capable of building, testing, publishing, and deploying software, much like Maven. Gradle accomplishes this by providing a declarative Groovy DSL and sensible defaults, making a build file not bigger than is needed, thus improving readability.

We’ll talk about Gradle again in chapter 20 to cover its general use for build automation. Here, we’ll focus on its use for integrating unit tests in the build.

Before we start, we need to install Gradle. This can be done in multiple ways, one of which is downloading Gradle from the Gradle website (www.gradle.org). An easier way is to use the GVM tool to manage the set of software installations, such as Groovy, Grails, Gradle, and more. The GVM can be downloaded from the GVM tool website (www.gvmtool.net) using a simple command:[24]

24 Windows users might want to use Cygwin or posh-gvm or download Gradle manually.

curl -s get.gvmtool.net | bash
Once GVM is installed, installing Gradle is as easy as typing:

gvm install gradle
This will download and install the newest version of Gradle, which at the moment of writing is version 2.2.1. Gradle uses a build file named build.gradle that contains all the instructions to create a correct build.

The basic build.gradle we’ll use is the following:

apply plugin: 'groovy'

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.4.0'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
This is the basic set of information needed to build our Groovy project. This build file adds the Groovy plugin, which makes is possible to mix Groovy and Java code by enabling joint compilation; adds the Groovy dependency to compile the Groovy files; and adds the latest version of the JUnit 4 dependency.

To create the project structure, we need to add a Gradle task, which will create the complete project structure for us. Add the following lines of code to the build.gradle file as shown in the following listing.

Listing 17.18. Adding a Gradle task
task initProject () << {
    if (hasProperty(initPlugins)) {
        initPlugins.split(',').each { plug ->
            project.apply {
                plugin(plug.trim())
            }
        }
    }

    project.sourceSets*.allSource.srcDirTrees.flatten().dir.each { dir ->
        dir.mkdirs()
    }
}
Now we can easily create a complete project structure without having to create it by hand. Executing the Gradle initProject task will create everything we need. We can execute the task by typing:

gradle initProject -PinitPlugins=groovy
The result is the following project structure:

.
├── build.gradle
└── src
    ├── main
    │   ├── groovy
    │   ├── java
    │   └── resources
    └── test
        ├── groovy
        ├── java
        └── resources
As you can see, Gradle created a directory structure well known by Maven users that contains source and test folders for Groovy as well as for Java. If you’re not going to use any Java source files in your project, you can of course remove those directories, but for now, we’ll leave them in.

The next task is creating the test and the SUT. For this, we’ll use a simple calculator. Place the Calculator.groovy with the following contents in the src/main/groovy directory, as shown in the next listing.

Listing 17.19. Using Calculator.groovy
class Calculator {
    def add(number1, number2) {
        return number1 + number2
    }
}
And its test class, CalculatorTest.groovy, in the src/test/groovy directory:

import org.junit.Test

class CalculatorTest {
    @Test
    void testAdd() {
        def calculator = new Calculator()
        assert 10 == calculator.add(3, 7)
    }
}
We’ve now created a simple test for our Calculator. By running the Gradle test task, Gradle will compile our source code and run the tests to validate whether the outcome is what we expect. Running the Gradle test task is done with the following command.

gradle test
This creates the following output:

:compileJava UP-TO-DATE
:compileGroovy
:processResources UP-TO-DATE
:classes
:instrument SKIPPED
:compileTestJava UP-TO-DATE
:compileTestGroovy
:processTestResources UP-TO-DATE
:testClasses
:test

BUILD SUCCESSFUL
As you can see here, everything built correctly, and we now have a successful build. To accomplish this, all we had to do was enable the Groovy plugin and provide the right dependencies, and we’re ready to integrate in our build. Can’t be much easier than that, can it?

Next up, we’ll integrate the build with Maven, another Open Source build tool.

### 17.8.2. Build integration with Maven
Apache Maven is a software project-management framework that can help you manage the many activities associated with producing a project’s deliverable artifacts. This may include acquiring your project’s dependent software, compiling your software, testing it, packaging it, and generating test and metrics reports. Two main versions of Maven are in use today: Maven 2 (versions 2.0 and above) and Maven 3 (versions 3.0 and above).

Maven supports the concept of plug-ins to perform many of the project lifecycle activities that it manages for you. For example, there are plug-ins to compile Java files, test them, package them up as jar files, and so forth. Because Groovy tests are easily compiled to normal Java bytecode, it should come as no surprise that you can leverage many of the existing Maven Java tasks to assist you. Plus, there are purpose-built Maven tasks for Groovy that you can utilize.

If you’re already a Maven user you can use the Groovy-Eclipse compiler.[25] Using the plugin you can compile your Java and Groovy projects. In the approach we’re going to use to ensure that our Groovy tests automatically run as part of our Maven build, we first need to compile the Groovy files down to bytecode. First we need to enable the Groovy compiler. We can do this by adding the following to the pom.xml, so that our Groovy sources are compiled, and can then be used in the test phase, as shown in the following listing.

25 See http://docs.groovy-lang.org/latest/html/documentation/tools-groovyeclipse.html for more details.

Listing 17.20. Enabling the Groovy compiler
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.2</version>
            <configuration>
                <compilerId>groovy-eclipse-compiler</compilerId>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>org.codehaus.groovy</groupId>
                    <artifactId>groovy-eclipse-compiler</artifactId>
                    <version>2.9.1-01</version>
                </dependency>
            </dependencies>
        </plugin>
        <plugin>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-eclipse-compiler</artifactId>
            <version>2.9.1-01</version>
            <extensions>true</extensions>
        </plugin>
    </plugins>
</build>
For the Maven groovy compiler to work, we need Groovy to be in our Java classpath. In Maven terms, we’ve introduced Groovy as a dependency, so we’ll also have to update Maven’s pom.xml file and add the Groovy (as well as the JUnit framework) dependency as shown in the following listing.

Listing 17.21. Updating Maven’s pom.xml file and adding a Groovy dependency
<dependencies>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>2.4.0</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
We are now ready to run our tests. This can be done from a DOS or UNIX command shell:

$> mvn test
The output should look something like this:

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Concurrency config is parallel='none', perCoreThreadCount=true, threadCount=2, useUnlimitedThreads=false
Running CalculatorTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.351 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------
[INFO] Total time: 2.794 s
[INFO] Finished at: 2015-02-06T05:16:34+10:00
[INFO] Final Memory: 10M/219M
[INFO] ------------------------------------------------------
Configuring Maven to run test cases in Groovy is fairly straightforward, and as you can see, plugging Groovy into your normal Java build processes is a cinch, whether they’re Gradle or Maven based.

## 17.9. SUMMARY
That wraps up our exploration of how Groovy adds immense value to your unit-testing activities.

In any serious project it’s recommended you use a build tool. Whether you want to use Gradle or Maven is up to you, but build tools provide great functionality for creating build artifacts, managing dependencies or deploying software to servers. Having a standard way of building software in a team is often crucial and should be done in a central and standard way.

We believe that unit testing is not only a worthwhile activity but also sometimes even more demanding and full of variations and engineering challenges than writing production code. Our experiences with Groovy are that it assists with meeting those demands and challenges. We hope you felt this too when we examined the benefits that Groovy brings to unit testing: the automatic availability of JUnit, the enhanced test case class with its additional assert methods, and the in-built support for mocks, stubs, and other dynamic classes.

Groovy’s integrated unit-test support lets you test Groovy and Java code alike. Our more detailed examination of how to unit test Groovy code with Groovy tests, how to test Java code with Groovy tests, and how to organize your tests into meaningful suites gave you the grounding to begin testing your own systems using Groovy.

Our investigation of advanced testing techniques led us to explore how to use stubs, mocks, and other dynamic classes such as maps and GroovyLogTestCase. With the help of these advanced features, it’s possible to test complex scenarios with minimal to moderate effort. Previously tricky scenarios can sometimes be tackled with much less work. This can often be the difference between justifying unit testing and it being too expensive. By augmenting these techniques with data-driven and property-based testing you have enormous flexibility and no reason not to have an appropriate testing regime and quality production code.

For sustainable software development with a high level of test coverage, unit testing must be both pleasant and efficient. What makes it pleasant is seamless integration into the developer’s IDE of choice to provide immediate feedback in develop/test/refactor cycles. What makes it efficient is the frequent unsupervised self-running execution of the test suite in an automated build process. In the Groovy world, both of these have excellent support.

Groovy gains much from its Java heritage. This was shown clearly when we looked at additional Java-level tool integration: in particular, one technology that enabled us to do code coverage and another that enabled us to do stress and performance testing. We examined only two tools, but hundreds are available for Java, and there are many yet-unexplored possibilities for leveraging them in Groovy.

To advocates of unit testing, Groovy can only be seen as a powerful and positive addition to the Java and Groovy developer’s toolkit. With Groovy, you can write your tests more quickly and easily. Just think, with all the time you’ll save by writing tests in Groovy, you can now go back to your customer and ask for more feature requests!
