# Chapter 4. Collective Groovy datatypes

This chapter covers

* Understanding Groovy’s collective datatypes: ranges, lists, and maps
* How to declare them
* Operators and library methods for these types
* How to use them in action

#### The intuitive mind is a sacred gift and the rational mind is a faithful servant. We have created a society that honors the servant and has forgotten the gift.

Albert Einstein

The nice thing about computers is that they never get tired of repeatedly doing the same task. This is probably the 
singlemost important quality that justifies letting them take part in our lives. Searching through countless files 
or web pages, downloading emails every 10 minutes, looking up all values of a stock symbol for the last quarter to 
paint a nice graph—these are only a few examples in which a computer needs to repeatedly process an item of a data 
collection. It’s no wonder that a great deal of programming work is about collections.

Because collections are so prominent in programming, Groovy alleviates the tedium of using them by directly supporting 
datatypes of a collective nature: ranges, lists, and maps. Just as with simple datatypes, Groovy’s support for 
collective datatypes encompasses new lightweight means for literal declaration, specialized operators, and numerous 
GDK enhancements.

The notation that Groovy uses to set its collective datatypes into action will be new to Java programmers, but as 
you’ll see, it’s easy to understand and remember. You’ll pick it up so quickly that you’ll hardly be able to imagine 
a time when you were new to the concept.

Despite the new notation possibilities, lists and maps have the very same semantics as in Java. This situation is 
slightly different for ranges, because they don’t have a direct equivalent in Java. So let’s start our tour with 
that topic.

## 4.1. WORKING WITH RANGES
Think about how often you’ve written a loop like this:

for (int i=0; i<upperBound; i++){
   // do something with i
}

Most of us have done this thousands of times. It’s so common that it is second nature. Does the code tell you what 
it does or how it does it?

After inspecting the variable, the conditional, and the incrementation, you see that it’s an iteration starting at zero 
and not reaching the upper bound, assuming there are no side effects on i in the loop body. You have to go through the 
description of how the code works to find out what it does.

Next, consider how often you’ve written a conditional like this:

if (x >= 0 && x <= upperBound) {
    // do something with x
}

The same thing applies here: you have to inspect how the code works to understand what it does. Variable x must be 
between zero and an upper bound for further processing. It’s easy to overlook that the upper bound is now inclusive.

We’re not saying that we make mistakes using this syntax on a regular basis. We’re not saying that you can’t get used 
to (or indeed haven’t gotten used to) the C-style for loop, as countless programmers have over the years. What we’re 
saying is that it’s harder than it needs to be; and, more important, it’s less expressive than it could be. Can you 
understand it? Absolutely. Then again, you could understand this chapter if it were written entirely in capital 
letters—that doesn’t make it a good idea, though.

Groovy allows you to reveal the meaning of such code pieces by providing the concept of a range. A range has a left 
bound and a right bound. You can do something for each element of a range, effectively iterating through it. You can 
determine whether a candidate element falls inside a range. In other words, a range is an interval plus a strategy 
for how to move through it.

By introducing the new concept of ranges, Groovy extends your means of expressing your intentions in the code.

We’ll show you how to specify ranges, how the fact that they’re objects makes them ubiquitously applicable, how to use 
custom objects as bounds, and how they’re typically used in the GDK.

### 4.1.1. Specifying ranges
Ranges are specified using the double-dot range operator (..) between the left and right bounds. This operator has a 
low precedence, so you often need to enclose the declaration in parentheses. Ranges can also be declared using their 
respective constructors.

The ..< range operator specifies a half-exclusive range—that is, the value on the right isn’t part of the range:

left..right
(left..right)
(left..<right)
Ranges usually have a lower left bound and a higher right bound. When this is switched it’s called a reverse range. Ranges can also be any combination of the types we’ve described. The following listing shows these combinations and how ranges can have bounds other than integers, such as dates and strings. Groovy supports ranges at the language level with the special for-in-range loop.

Listing 4.1. Range declarations




Note that we assign a range to a variable . In other words, the variable holds a reference to an object of type groovy.lang.Range. We’ll examine this feature further and see what consequences it implies.

Date objects can be used in ranges  because the GDK adds the previous and next methods to date, which increases or decreases the date by one day.

Note
The GDK also adds minus and plus operators to java.util.Date, which increases or decreases the date by so many days.

The String methods previous and next are added by the GDK to make strings usable for ranges . The last character in the string is incremented or decremented, and overflow or underflow is handled by appending a new character or deleting the last character.

You can walk through a range with the each method, which presents the current value to the given closure with each step . If the range is reversed, you walk through the range backward. If the range is half-exclusive, the walking stops before reaching the right bound.

### 4.1.2. Ranges are objects
Because every range is an object, you can pass a range around and call its methods. The most prominent methods are each, which executes a specified closure for each element in the range, and contains, which specifies whether or not a value is within a range.

Being first-class objects, ranges can also participate in the game of operator overriding (see section 3.3) by providing an implementation of the isCase method, with the same meaning as contains. That way, you can use ranges as grep filters and as switch cases. This is shown in the following listing.

Listing 4.2. Ranges are objects


The use with the grep method  is a good example for passing around range objects: the midage range gets passed as a parameter to the grep method.

Classification through ranges  is what you’ll often find in the business world: interest rates for different ranges of allocated assets, transaction fees based on volume ranges, and salary bonuses based on ranges of business done. Although technical people prefer using functions, business people tend to use ranges. When you’re modeling the business world in software, classification by ranges can be very handy.

### 4.1.3. Ranges in action
Listing 4.1 made use of date and string ranges. In fact, any datatype can be used with ranges, provided that both of the following are true:

The type implements next and previous; that is, it overrides the ++ and -- operators.
The type implements java.lang.Comparable; that is, it implements compareTo, effectively overriding the <=> (spaceship) operator.
As an example, listing 4.3 implements a class Weekday that represents a day of the week. From the perspective of the code that uses the class, a Weekday has a value 'Sun' through 'Sat'. Internally, it’s just an index between 0 and 6. A little list maps indexes to weekday name abbreviations.

We implement next and previous to return the respective new Weekday object. compareTo simply compares the indexes.

With this preparation, we can construct a range of working days and work our way through it, reporting the work done until we reach the well-deserved weekend. Oh, and our boss wants to assess the weekly work report. A final assertion does this on his behalf.

Listing 4.3. Custom ranges: weekdays




This code can be placed inside one script file,[1] even though it contains both a class declaration and script code. The Weekday class is like an inner class to the script.

1 But don’t call it Weekday.groovy, otherwise two clashing Weekday.class files will be produced, one for the script and one for the inner class.

The implementation of previous  is a bit unconventional. Although next uses the modulo operator in a conventional way to jump from Saturday (index 6) to Sunday (index 0), the opposite direction simply decreases the index. The index –1 is used for looking up the previous weekday name, and DAYS[-1] references the last entry of the days list, as you’ll see in the next section. We construct a new Weekday('Sat'), and the constructor normalizes the index to 6.

Compared to the Java alternatives, ranges have proven to be a flexible solution. for loops and conditionals aren’t objects, they cannot be reused, and they cannot be passed around, but ranges can. Ranges let you focus on what the code does, rather than how it does it. This is a pure declaration of your intent, as opposed to fiddling with indexes and boundary conditions.

Using custom ranges is the next step forward. Look actively through your code for possible applications. Ranges slumber everywhere, and bringing them to life can significantly improve the expressiveness of your code. With a bit of practice, you may find ranges where you never thought possible. This is a sure sign that new language concepts can change your perception of the world.

You’ll shortly refer to your newly acquired knowledge about ranges when we explore the subscript operator on lists, the built-in datatype that we’re going to cover next.

## 4.2. WORKING WITH LISTS
In a recent Java project, we had to write a method that takes a Java array and adds an element to it. This seemed like a trivial task, but we forgot how awkward Java programming could be. (We’re spoiled from too much Groovy programming.) Java arrays cannot be changed in length, so you cannot add elements easily. One way is to convert the array to a java.util.List, add the element, and convert back. A second way is to construct a new array of size+1, copy the old values over, and set the new element to the last index position. Either way takes some lines of code.

But Java arrays also have their benefits in terms of language support. They work with the subscript operator to easily retrieve elements of an array by index like myarray[index], or store elements at an index position with myarray[index] = newElement.

We’ll demonstrate how Groovy lists give you the best of both approaches, extending the features for smart operator implementations, method overloading, and using lists as Booleans. With Groovy lists, you’ll also discover new ways of leveraging the power of the Java Collections API.

### 4.2.1. Specifying lists
Listing 4.4 shows various ways of specifying lists. The primary way is with square brackets around a sequence of items, delimited with commas:

[item1, item2, item3]
The sequence can be empty to declare an empty list. Lists are by default of type java.util.ArrayList and can also be declared explicitly by calling the respective constructor. The resulting list can still be used with the subscript operator. In fact, this works with any type of list, as shown next with type java.util.LinkedList.

Lists can be created and initialized at the same time by calling toList on ranges.

Listing 4.4. Specifying lists


We use the addAll(Collection) method from java.util.List  to easily fill the lists. As an alternative, the collection to fill from can be passed right into the constructor, as done here with LinkedList.

For the sake of completeness, we need to add that lists can also be constructed by passing a Java array to Groovy. Such an array is subject to autoboxing—a list will be automatically generated from the array with its elements being autoboxed.

The GDK extends all arrays, collection objects, and strings with a toList method that returns a newly generated list of the contained elements. Strings are handled like lists of characters.

### 4.2.2. Using list operators
Lists implement some of the operators that you saw in section 3.3. Listing 4.4 contains two of them: the getAt and putAt methods to implement the subscript operator. But this is a simple use that works with a mere index argument. There’s much more to the list operators than that.

Subscript operator
The GDK overloads the getAt method with range and collection arguments to access a range or a collection of indexes. This is demonstrated in the next listing.

The same strategy is applied to putAt, which is overloaded with a Range argument, assigning a list of values to a whole sublist.

Listing 4.5. Accessing parts of a list with an overloaded subscript operator


Subscript assignments with ranges don’t need to be of identical size. When the assigned list of values is smaller than the range or even empty, the list shrinks . When the assigned list of values is bigger, the list grows .

Ranges used within subscript assignments are a convenience feature to access Java’s excellent sublist support for lists. See also the Javadoc for java.util.List#sublist.

In addition to positive index values, lists can be subscripted with negative indexes that count from the end of the list backward. Figure 4.1 shows how positive and negative indexes map to an example list [0,1,2,3,4].

Figure 4.1. Positive and negative indexes of a list of length 5, with “in bounds” and “out of bounds” classification for indexes


Consequently, you get the last entry of a nonempty list with list[-1] and the next-to-last with list[-2]. Negative indexes can also be used in ranges, so list[-3..-1] gives you the last three entries. When using a reversed range, the resulting list is reversed as well, so list[4..0] is [4,3,2,1,0]. In this case, the result is a new list object rather than a sublist in the sense of the JDK. Even mixtures of positive and negative indexes are possible, such as list[1..-2], to cut away the first entry and the last entry.

Avoid negative indexes with half-exclusive ranges
Ranges in List’s subscript operator are IntRanges. Exclusive IntRanges are mapped to inclusive ones at construction time, before the subscript operator comes into play and can map negative indexes to positive ones. This can lead to surprises when mixing positive left and negative right bounds with exclusiveness; for example, IntRange (0..<-2) gets mapped to (0..-1), such that list[0..<-2] is effectively list[0..-1].

Although this is stable and works predictably, it may be confusing for the readers of your code, who may expect it to work like list[0..-3]. For this reason, this situation should be avoided for the sake of clarity.

Adding and removing items
Although the subscript operator can be used to change any individual element of a list, there are also operators available to change the contents of the list in a more drastic way: plus(Object), plus(Collection), leftShift(Object), minus(Collection), and multiply. The following listing shows them in action. The plus method is overloaded to distinguish between adding an element and adding all elements of a collection. The minus method only works with collection parameters.

Listing 4.6. List operators involved in adding and removing items


While we’re talking about operators, it’s worth noting that we’ve used the == operator on lists, happily assuming that it does what we expect. Now you see how it works: the equals method on lists tests that two collections have equal elements. See the Javadoc of java.util.List#equals for details.

Control structures
Groovy lists are more than flexible storage places. They also play a major role in organizing the execution flow of Groovy programs. The following listing shows the use of lists in Groovy’s if, switch, and for control structures.

Listing 4.7. Lists taking part in control structures


In  and , you see the trick that you already know from patterns and ranges: implementing isCase and getting a grep filter and a switch classification for free.  is a little surprising. Inside a Boolean test, empty lists evaluate to false.  shows looping over lists or other collections and also demonstrates that lists can contain mixtures of types.

### 4.2.3. Using list methods
There are so many useful methods on the List type that we cannot provide an example for all of them in the language description. The large number of methods comes from the fact that the Java interface java.util.List is already fairly wide (25 methods in JDK 1.7 and 28 in JDK 1.8).

Furthermore, the GDK adds methods to the List interface, to the Collection interface, and to Object. Therefore, many methods are available on the List type, including all methods of Collection and Object.

Appendix C has the complete overview of all methods added to List by the GDK. The Javadoc of java.util.List has the complete list of its JDK methods.

While working with lists in Groovy, there’s no need to be aware of whether a method stems from the JDK or the GDK, or whether it’s defined in the List or Collection interface. But for the purpose of describing the Groovy List datatype, we fully cover the GDK methods on lists and collections, but not all combinations from overloaded methods and not what’s covered in the previous examples. We provide only partial examples of the JDK methods that we consider important.

Manipulating list content
A first set of methods is presented in the following listing. It deals with changing the content of the list by adding and removing elements; combining lists in various ways; sorting, reversing, and flattening nested lists; and creating new lists from existing ones.

Listing 4.8. Methods to manipulate list content




List elements can be of arbitrary type, including other nested lists. This can be used to implement lists of lists, the Groovy equivalent of multidimensional arrays in Java. For nested lists, the flatten method provides a flat view of all elements.

An intersection of lists contains all elements that appear in both lists. Collections can also be checked for being disjointed—that is, whether their intersection is empty.

Lists can be used like stacks, with usual stack behavior on push and pop . The push operation is relayed to the list’s << (left-shift) operator.

When list elements are comparable, there’s a natural sort. Alternatively, the comparison logic of the sort can be specified as a closure , . The first example sorts lists of lists by comparing their entry at index 0. The second example shows that a single argument can be used inside the closure for comparison. In this case, the comparison is made between the results that the closure returns when fed each of the candidate elements.

Elements can be removed by index  or by value . We can also remove all the elements that appear as values in the second list. These removal methods are the only ones in the listing that are available in the JDK.

The collect method  returns a new list that’s constructed from what a closure returns when successively applied to all elements of the original list. In the example, we use it to retrieve a new list where each entry of the original list is multiplied by two. Other languages call such a method map, but we don’t because it’s so easily confused with the datatype of the same name.[2]

2 The collect method’s name was originally inspired from Smalltalk and is also used in C#, but several popular functional languages use map. Given the increased popularity of functional concepts, future versions of Groovy may supply a map alias for collect.

With findAll , we retrieve a list of all items for which the closure evaluates to true. The example here uses the modulo operator to find all odd numbers.

Two issues related to changing an existing list are removing duplicates and null values. One way to remove duplicate entries is to convert the list to a datatype that’s free of duplicates: a Set. This can be achieved by calling a Set’s constructor with that list as an argument, such as:
```
def x = [1,1,1]
assert [1] == new HashSet(x).toList()
assert [1] == x.unique()
```
If you don’t want to create a new collection but want to keep working on your cleaned list, you can use the unique method, which ensures that the sequence of entries isn’t changed by this operation.

Removing null from a list can be done by keeping all non-nulls—for example, with the findAll methods that you’ve seen previously:
```
def x = [1,null,1]
assert [1,1] == x.findAll{it != null}
assert [1,1] == x.grep{it}
```
You can see there’s an even shorter version with grep, but to understand its mechanics, you need more knowledge about closures (see chapter 5) and the “Groovy truth” (see chapter 6). Just take it for granted until then.

Accessing list content
Lists have methods to query their elements for certain properties, iterate through them, and retrieve accumulated results.

Query methods include a count of given elements in the list, min and max, a find method that finds the first element that satisfies a closure, and methods to determine whether every or any element in the list satisfies a closure.

Iteration can be achieved as usual—forward with each or backward with each-Reverse.

Cumulative methods come in simple and sophisticated versions. The join method is simple: it returns all elements as a string, concatenated with a given string. The inject method is inspired by Smalltalk: it uses a closure to inject new functionality. That functionality operates on an intermediary result and the current element of the iteration. The first parameter of the inject method is the initial value of the intermediary result. In the following listing, we use this method to sum up all elements and then use it a second time to multiply them.

Listing 4.9. List query, iteration, and accumulation




Understanding and using the inject method can be a bit challenging if you’re new to the concept. Note that it’s parallel to the iteration examples, with store playing the role of the intermediary result. The benefit is that you don’t need to introduce that extra variable to the outer scope of your accumulation, and your closure has no side effects on that scope. Other languages often call this kind of method fold or reduce.

There are a host of additional GDK methods we don’t have space to cover in detail, including collate, collectMany, combinations, dropWhile, flatten, groupBy, permutations, take, transpose, and withIndex. Consult the cheat sheet for lists in appendix D for a few more examples and the complete list of GDK methods for lists in the groovy-jdk documentation.[3]

3 Groovy JDK API Documentation describes the methods added to the JDK to make it more groovy: http://docs.groovy-lang.org/docs/latest/html/groovy-jdk/.

The GDK also introduces two convenience methods for producing views backed by an existing list: asImmutable and asSynchronized. These methods use Java’s Collections.unmodifiableList and Collections.synchronizedList to protect the list from unintended content changes and concurrent access. See these methods’ Javadocs for more details on the topic.

### 4.2.4. Lists in action
After all the artificial examples, you deserve to see a real one. Here it is: we’ll implement Tony Hoare’s Quicksort[4] algorithm in listing 4.10. To make things more interesting, we’ll do so in a generic way; we’ll not demand any particular datatype for sorting. We’ll rely on duck typing.[5] For our use, this means that as long as we can use the <, =, and > operators with the list items, we treat them as if they were comparable.

4 For an explanation of Quicksort, sometimes called partition-exchange sort, see http://en.wikipedia.org/wiki/Quicksort.

5 See section 3.2.4 “The case for optional typing” for more details.

The goal of Quicksort is to be sparse with comparisons. The strategy relies on finding a good pivot element in the list that serves to split the list into two sublists: one with all elements smaller than the pivot, and the second with all elements bigger than the pivot. Quicksort is then called recursively on the sublists. The rationale behind this is that you never need to compare elements from one list with elements from the other list. If you always find the perfect pivot, which exactly splits your list in half, the algorithm runs with a complexity of n × log(n). In the worst case, you choose a border element every time, and you end up with a complexity of n2. In the next listing, we choose the middle element of the list, which is a good choice for the frequent case of preordered sublists.

Listing 4.10. Quicksort with lists


In contrast to what we said earlier, we use not two but three lists . Use this implementation when you don’t want to lose items that appear multiple times.

The duck-typing approach is powerful when it comes to sorting different types. We can sort a list of mixed content types , or even sort a string . This is possible because we didn’t demand any specific type to hold the items. As long as that type implements size, getAt(index), and findAll, we’re happy to treat it as a sortable. Actually, we use duck typing twice: for the items and for the structure.

Note
The sort method that comes with Groovy uses Java’s sorting implementation that beats our example in terms of worst-case performance. It guarantees a complexity of n × log(n). But we win on a different front.

Of course, this implementation could be optimized in multiple dimensions. Our goal is to be tidy and flexible, not the fastest on the block.

If we had to explain the Quicksort algorithm without the help of Groovy, we’d sketch it in pseudocode that looks exactly like listing 4.10. In other words, the Groovy code itself is the best description of what it does. Imagine what this can mean to your codebase, when all your code reads like it were a formal documentation of its purpose!

Another extremely common use case where Groovy’s GDK methods for lists add real value is filter/map/reduce style processing of lists of your domain classes. We’ll use a list of URLs but you could imagine a list of customers, invoices, shopping carts, or some other domain objects. Suppose we want to take a list of URLs, select only those having a port with certain characteristics, then transform some information from the URL, then sort the results, and finally combine the results into a single string. This can be done easily with a series of list GDK methods as shown in the following listing.

Listing 4.11. Processing lists of URLs
def urls = [
  new URL('http', 'myshop.com', 80, 'index.html'),
  new URL('https', 'myshop.com', 443, 'buynow.html'),
  new URL('ftp', 'myshop.com', 21, 'downloads')
]

assert urls
    .findAll{ it.port < 99 }
    .collect{ it.file.toUpperCase() }
    .sort()
    .join(', ') == 'DOWNLOADS, INDEX.HTML'
Prior to Java 8, the equivalent Java code to achieve this would be quite cluttered and cumbersome. The Groovy version is simple and very easy to understand in comparison. The Java 8 version is substantially better and the good news is that you can leverage the Java 8 methods with a bit of Groovy syntactic sugar added to boot, as can be seen here:

// Groovy with Java 8
import java.util.stream.Collectors
def commaSep = Collectors.joining(", ")
assert urls.stream()
    .filter{ it.port < 99 }
    .map{ it.file.toUpperCase() }
    .sorted()
    .collect(commaSep) == 'DOWNLOADS, INDEX.HTML'
If you like what you see with these examples, there’s even better news when you see how to perform such operations concurrently using GPars in chapter 18.

You’ve seen that lists are one of Groovy’s strongest workhorses. They’re always at hand; they’re easy to specify inline, and using them is easy due to the operators supported. The plethora of available methods may be intimidating at first, but that’s also the source of lists’ power.

You’re now able to add them to your carriage and let them pull the weight of your code.

The next section about maps will follow the same principles that you’ve seen for lists: extending the Java collection’s capabilities while providing efficient shortcuts.

## 4.3. WORKING WITH MAPS
Suppose you were about to learn the vocabulary of a new language, and you set out to find the most efficient way of doing so. It’d surely be beneficial to focus on the words that appear most often in your texts. So, you’d take a collection of your texts and analyze the word frequencies in that text corpus.[6]

6 Analyzing word frequencies in a text corpus is a common task in computer linguistics and is used for optimizing computer-based learning, search engines, voice recognition, and machine translation programs.

What Groovy mechanisms are available to do this? For the time being, assume that you can work on a large string. You have numerous ways of splitting this string into words. But how do you count and store the word frequencies? You can’t have a distinct variable for each possible word you encounter. Finding a way of storing frequencies in a list is possible but inconvenient—more suitable for a brainteaser than for good code. Maps come to the rescue.

Some pseudocode to solve the problem could look like this:

for each word {
    if (frequency of word is not known)
        frequency[word] = 0
    frequency[word] += 1
}
This looks like the list syntax, but with strings as indexes rather than integers. In fact, Groovy maps appear like lists, allowing any arbitrary object to be used for indexing.

To describe the map datatype, we show how maps can be specified, which operations and methods are available for maps, some surprisingly convenient features of maps, and, of course, a map-based solution for the word-frequency exercise.

### 4.3.1. Specifying maps
The specification of maps is analogous to the list specification that you saw in the previous section. Just like lists, maps make use of the subscript operator to retrieve and assign values. The difference is that maps can use any arbitrary type as an argument to the subscript operator, where lists are bound to integer indexes. Lists are aware of the sequence of their entries, maps are generally not. Specialized maps like java.util.TreeMap may have a sequence to their keys, though.

Simple maps are specified with square brackets around a sequence of items, delimited with commas. The key feature of maps is that the items are key–value pairs that are delimited by colons:

[key1:value1, key2:value2, key3:value3]
In principle, any arbitrary type can be used for keys or values. When using exotic[7] types for keys, you need to obey the rules as outlined in the Javadoc for java.util.Map.

7 Exotic in this sense refers to types of which the instances change their hashCode during their lifetime. There is also a corner case with GStrings if their values write themselves lazily.

The character sequence [:] declares an empty map. Maps are, by default, of type java.util.LinkedHashMap, and can also be declared explicitly by calling the respective constructor. The resulting map can still be used with the subscript operator. In fact, this works with any type of map, as you see in the next listing with java.util.TreeMap.

Listing 4.12. Specifying maps


In the previous listing, we use the putAll(Map) method from java.util.Map to easily fill the example map. One alternative would be to pass myMap as an argument to TreeMap’s constructor.

For the common case of having keys of type String, you can leave out the string markers (single or double quotes) in a map declaration:

assert ['a':1] == [a:1]
Such a convenience declaration is allowed only if the key contains no special characters (it needs to follow the rules for valid identifiers) and isn’t a Groovy keyword.

This notation can also get in the way when, for example, the content of a local variable is used as a key. Suppose you have local variable x with content 'a'. Because [x:1] is equal to ['x':1], how can you make it equal to ['a':1]? The trick is that you can force Groovy to recognize a symbol as an expression by putting it inside parentheses:

def x = 'a'
assert ['x':1] == [x:1]
assert ['a':1] == [(x):1]
It’s rare to require this functionality, but when you need keys that are derived from local symbols (local variables, fields, properties), forgetting the parentheses is a likely source of errors.

### 4.3.2. Using map operators
The simplest operations with maps are storing objects in a map with a key and retrieving them back using that key. Listing 4.13 demonstrates how to do that. One option for retrieving is using the subscript operator. As you’ve probably guessed, this is implemented with Map’s getAt method. A second option is to use the key like a property with a simple dot-key syntax. You’ll learn more about properties in chapter 7. A third option is the get method, which additionally allows you to pass a default value to be returned if the key isn’t yet in the map. If no default is given, null will be used as the default. If on a get(key,default) call the key isn’t found and the default is returned, the (key,default) pair is added to the map.

Listing 4.13. Accessing maps (GDK map methods)


Assignments to maps can be done using the subscript operator or via the dot-key syntax. If the key in the dot-key syntax contains special characters, it can be put into string markers, like so:

myMap = ['a.b':1]
assert myMap.'a.b' == 1
Just writing myMap.a.b wouldn’t work here—that would be the equivalent of calling myMap.getA().getB().

Listing 4.14 shows how information can easily be gleaned from maps, largely using core JDK methods from java.util.Map. Using equals, size, containsKey, and containsValue as in listing 4.14 is straightforward. The keySet method returns a set of keys, a collection that’s flat like a list but has no duplicate entries and no inherent ordering. See the Javadoc of java.util.Set for details. To compare the keySet method against the list of known keys, we need to convert this list to a set. This is done with a small service method toSet.

The value method returns the list of values. Because maps have no idea how their keys are ordered, there’s no foreseeable ordering in the list of values. To make it comparable with the known list of values, we convert both to a set.

Maps can be converted into a collection by calling the entrySet method, which returns a set of entries where each entry can be asked for its key and value property.

Listing 4.14. Query methods on maps


The GDK adds two more informational methods to the JDK map type: any and every . They work analogously to the identically named methods for lists: they return a Boolean value to tell whether any or every entry in the map satisfies a given closure.

Listing 4.14 makes use of the fact that a literally declared map is of type LinkedHashMap and we can therefore rely on the ordering of entries, keys, and values. This feature is so helpful and shields programmers from bugs that arise when relying on such a sequence for arbitrary maps. With the information about the map, we can iterate it over the entries or over keys and values separately. Because the sets that are returned from keySet and entrySet are collections, we can use them with the for-in-collection type loops. The following listing goes through some of the possible combinations.[8]

8 The example uses a default Groovy map that retains order. When using other types of maps, the order in which these iteration methods return values may be undefined.

Listing 4.15. Iterating over maps (GDK)


Map’s each method uses closures in two ways: passing one parameter into the closure means that it’s an entry, and passing two parameters means it’s a key and a value. The latter is more convenient to work with for common cases.

Map content can be changed in various ways, as shown in listing 4.16. Removing elements works with the original JDK methods. New capabilities that the GDK introduces are:

Creating a subMap of all entries with keys from a given collection.
findAll entries in a map that satisfy a given closure.
find one entry that satisfies a given closure, where, unlike lists, there’s no notion of a first entry because there’s no ordering in maps.
collect in a list whatever a closure returns for each entry, optionally adding to a given collection.
Listing 4.16. Changing map content and building new objects from it


The first two examples (clear and remove) in the listing are from the core JDK; the rest are all GDK methods. Only the subMap method  is particularly new here; collect, find, and findAll act as they would with lists, operating on map entries instead of list elements. The subMap method is analogous to subList, but it specifies a collection of keys as a filter for the view onto the original map.

To assert that the collect method works as expected, recall a trick that we discussed about lists: use the every method on the list to make sure that every entry is even. The collect method comes with a second version that takes an additional collection parameter. It adds all closure results directly to this collection, avoiding the need to create temporary lists.

From the list of available methods that you’ve seen for other datatypes, you may miss the dearly beloved isCase for use with grep and switch. Don’t we want to classify with maps? Well, we need to be more specific: Do we want to classify by the keys or by the values? Either way, an appropriate isCase is available when working on Map’s keySet or values.

The GDK introduces two more methods for the map datatype: asImmutable and asSynchronized. These methods use Collections.unmodifiableMap and Collections .synchronizedMap to protect the map from unintended content changes and concurrent access. See these methods’ Javadocs for more details on the topic.

### 4.3.3. Maps in action
In listing 4.17, we revisit the initial example of counting word frequencies in a text corpus. The strategy is to use a map with each distinct word serving as a key. The mapped value of that word is its frequency in the text corpus. We go through all words in the text and increase the frequency value of that respective word in the map. We need to make sure that we can increase the value when a word is hit the first time and there’s no entry yet in the map. Luckily, the get(key,default) method does the job.

We then take all keys, put them in a list, and sort it so that it reflects the order of frequency. Finally, we play with the capabilities of lists, ranges, and strings to print a nice statistic.

The text corpus under analysis is Baloo the Bear’s anthem on his attitude toward life.

Listing 4.17. Counting word frequency with maps




The example nicely combines our knowledge of Groovy’s datatypes . Counting the word frequency is essentially a one-liner. It’s even shorter than the pseudocode that we used to start this section. Having the sort method on the wordList accept a closure turns out to be very beneficial , because it’s able to implement its comparing logic on the wordFrequency map—on an object totally different from the wordList. Just as an exercise, try to do that in Java, count the lines, and judge the expressiveness of either solution. As an advanced exercise, you could try sorting by frequency and then alphabetically both in Groovy and Java. This will make the expressiveness of Groovy even more obvious.

In listing 4.11, we showed you map/filter/reduce style processing with lists. The same style of processing is frequently used with maps too as shown in this example:

def people = [peter: 40, paul: 30, mary: 20]
assert people
    .findAll{ _, age -> age < 35 }
    .collect{ name, _ -> name.toUpperCase() }
    .sort()
    .join(', ') == 'MARY, PAUL'
Lists and maps make a powerful duo. There are whole languages that build on just these two datatypes (such as Perl, with lists and hashes) and implement all other datatypes and even objects upon them. Their power comes from the complete and mindfully engineered Java Collections Framework. Thanks to Groovy, this power is now right at our fingertips.

Until now, we carelessly switched back and forth between Groovy and Java collection datatypes. We’ll throw more light on this interplay in the next section.

## 4.4. NOTES ON GROOVY COLLECTIONS
The Java Collections API is the basis for all the nice support that Groovy gives you through lists and maps. In fact, Groovy not only uses the same abstractions, it even works on the very same classes that make up the Java Collections API.

This is exceptionally convenient for those who come from Java and already have a good understanding of it. If you haven’t, and you’re interested in more background information, have a look at your Javadoc starting at java.util.Collection. The JDK documentation also includes a guide and tutorial about Java collections.

One of the typical peculiarities of the Java collections is that you shouldn’t try to structurally change one while iterating through it. A structural change is one that adds an entry, removes an entry, or changes the sequence of entries when the collection is sequence-aware. This applies even when iterating through a view onto the collection, such as using list[range].

### 4.4.1. Understanding concurrent modification
If you fail to meet this constraint, you’ll see a ConcurrentModificationException. For example, you cannot remove all elements from a list by iterating through it and removing the first element at each step:

def list = [1, 2, 3, 4]
list.each{ list.remove(0) }
// throws ConcurrentModificationException !!
Note
Concurrent in this sense doesn’t necessarily mean that a second thread changed the underlying collection. As shown in the example, even a single thread of control can break the structural stability constraint.

In this case, the correct solution is to use the clear method. The Java Collections API has lots of such specialized methods. When searching for alternatives, consider collect, addAll, removeAll, findAll, and grep.

This leads to a second issue: some methods work on a copy of the collection and return it when finished; other methods work directly on the collection object they were called on (we call this the receiver[9] object).

9 From the Smalltalk notion of describing method calls on an object as sending a message to the receiver.

### 4.4.2. Distinguishing between copy and modify semantics
Generally, there’s no easy way to anticipate whether a method modifies the receiver or returns a copy. Some languages have naming conventions for this. But Groovy couldn’t do so because all Java methods are directly visible in Groovy, and Java’s method names couldn’t be made compliant to such a convention. But Groovy tries to adapt to Java and follow the heuristics that you can spot when looking through the Java Collections API:

Methods that modify the receiver typically don’t return a collection. Examples: add, addAll, remove, removeAll, and retainAll. Counterexample: sort.
Methods that return a collection typically don’t modify the receiver. Examples: grep, findAll, and collect. Counterexample: sort (though we recommend using toSorted in that case). And yes, sort is a counterexample for both, because it returns a collection and modifies the receiver.
Methods that modify the receiver have imperative names. They sound like there could be an exclamation mark behind them. (Indeed, this is Ruby’s naming convention for such methods.) Examples: add, addAll, remove, removeAll, retainAll, and sort. Counterexamples: collect, grep, and findAll, which are imperative but don’t modify the receiver and return a modified copy.
The preceding rules can be mapped to operators, by applying them to the names of their method counterparts: << leftShift is imperative and modifies the receiver (on lists, unfortunately not on strings—doing so would break Java’s invariant of strings being immutable); plus isn’t imperative and returns a copy.

The convention in Groovy is that any method that implements an arithmetic operator (plus, minus, multiply, divide) doesn’t modify the receiver but returns a copy.

These aren’t clear rules but only heuristics to give you some guidance. Whenever you’re in doubt and object identity is important, have a look at the documentation or write a few assertions.

## 4.5. SUMMARY
This has been a long trip through the valley of Groovy’s datatypes. There were lots of paths to explore that led to new interesting places.

We introduced ranges as objects that, as opposed to control structures, have their own time and place of creation, can be passed to methods as parameters, and can be returned from method calls. This makes them very flexible, and once the concept of a range is available, many uses beyond simple control structures suggest themselves. The most natural example you’ve seen is extracting a section of a list using a range as the operand to the list’s subscript operator.

Lists and maps are more familiar to Java programmers than ranges but have suffered from a lack of language support in Java itself. Groovy recognizes just how often these datatypes are used, gives them special treatment in terms of literal declarations, and, of course, provides operators and extra methods to make life even easier. The lists and maps used in Groovy are the same ones encountered in Java and come with the same rules and restrictions, although these become less onerous due to some of the additional methods available on the collections.

Throughout our coverage of Groovy’s datatypes, you’ve seen closures used ubiquitously for making functionality available in a simple and unobtrusive manner. In the next chapter, we’ll demystify the concept, explain the usual and not-so-usual applications, and show how you can spice up your own code with closures.
