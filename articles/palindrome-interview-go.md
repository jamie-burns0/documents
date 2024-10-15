# The palindrome interview in Go

Following on from The palindrome interview in __Java__, __Python__, __Typescript__ and __Rust__ (https://github.com/jamie-burns0/documents/blob/main/articles/palindrome-interview.md), I started to explore __Go__ as an alternative to __Rust__.

### Why Go

So why did I start to look at __Go__? There were a few thoughts going on.

First, __Rust__ is held up as the go-to language for writing system code. I've never been down the system code path and I don't see it happening in the future. So not a great investment for me and it would over-complicate my language toolbox.

__Go__ on the other hand is marketed as easy to learn, great for teams and for building simple, secure and scalable systems. This sounds like my world.

Second, in StackOverflow's 2024 developer survey (https://survey.stackoverflow.co/2024/technology), __Go__ is on par with __Rust__ by popularity, and 7th by admired/desired. So, there's a market for it.

And, third, on the topic of market, I was reminded of a book Patrick Snowball introduced to Suncorp Group during his time as CEO, *Blue Ocean Strategy: How to Create Uncontested Market Space and Make the Competition Irrelevant* (https://www.blueoceanstrategy.com/books/blue-ocean-strategy-book/). The bit I remember was the idea of red and blue oceans. Red oceans are bloodied with competition - fish eating fish. There are a lot of __Javascript__/__Typescript__, __Python__ and __Java__ developers competing in the market. Blue oceans are not so bloodied by competition. Maybe __Go__ is a language that will present opportunities for me.

After getting my development environment setup for __Go__, I found a great series of *Go Class* videos on YouTube by Matt Holiday (https://www.youtube.com/watch?v=1MXIGYrMk80). After watching a few, I decided to jump in and see if I could implement my palindrome interview solution in __Go__.

### A quick recap on the palindrome interview solution

In a recent interview - which I call the Palindrome Interview - I was presented with a string and asked for a solution that identified if it was possible to make a palindrome from all characters of the string and, if so, print out a single palindrome. Afterward, I thought about my solution and, also, what that solution would look like in other coding languages.

My solution to the palindrome interview is,

1. Validate we have an alphabetic string
2. Parse the string into a map of characters with their frequency
3. Validate we have a frequency character map that will produce a palindrome
4. Make a palindrome from the frequency character map

The learnings and observations below are compared with __Java__, which is my wheelhouse.

### It all starts with main

A __Go__ application starts with a `main()` method, 

```
package main
import(
   // packages to import
)
func main() {
   // the code
}
```

Having `package`, `import` and `main`, and organising code into packages and importing these was immediately familiar. I felt eased into __Go__.

### Syntactic sugar

Let's look at the method for step 1

```
func IsStringAPalindromeCandidate(s string) bool {
   ...
}
```

A couple of syntactic things are backward for me. 

The first was __method visibility__. This is a __publicly accessible__ method, so it has to start  with an __uppercase__ letter - `IsStringAPalindromeCandidate` instead of `isStringAPalindromeCandidate`. 

Obviously the convention in __Java__ is that classes start with uppercase and methods start with lowercase. So, I had more than a few compiler errors with method visibility. As I got used to it, it was a simple way to introduce two levels of visibility without the need for keywords - uppercase is public, lowercase is not public. __Java__ offers four levels of visibility and uses three keywords - `public`, `protected`, `private` - to enable this.

The second was declaring the type of a variable and method return type. The syntax for a variable is variable name followed by variable type. For a method, the return type is at the end.

Very opposite to __Java__. In __Java__, I might have,

```
public boolean isStringAPalindromeCandidate(String s)
```

After exploring what seems to be __Go__'s backward way of writing it, I found I'm so used to __Java__ that I automatically unjumble the __Java__ method signature in my head to arrive at a sentence like,

"The public method `isStringAPalindromeCandidate` takes a variable `s` of type `String` and returns a value of type `boolean`"

__Go__ unjumbles this at the syntax level. I think this is great for someone who is starting out on their coding journey. We can see __Java__ thinking about this with JEP-477, 

>...Evolve the Java programming language so that beginners can write their first programs without needing to understand language features designed for large programs...

Another interesting syntactic quirk is the use of commas in lists. For some of my unit tests, I created and initialised `map[string]int`. For the last entry in the map, the compiler pulled me up on __missing__ a trailing comma. This is what it wanted to see,

```
func TestAValidMapWithMultipleEntriesReturnsTrue( t *testing.T ) {
    m := map[string]int {
        "a": 2,
	"b": 4,
    }
    ...
```

Anywhere else, that trailing comma after the `4` would have been a syntax error. But with __Go__, its a syntax error if there's __no__ trailing comma. In a weird way, its good. I don't know how many times I've added a new last entry to a list like this - in JSON files, in enums, in `List.of(...)` - and forgotten to add a comma to what is now the second-last line. A nice thing with the __Go__ syntax, is that second-last line would have had a trailing comma and I wouldn't have had an error.

One final thought about the syntactic sugar is the lack of noise. I don't need as many parentheses and I don't need semi-colons. The code looks a lot cleaner.

### Overloading methods

__Go__ doesn't seem to support method overloading, so as with __Rust__ and __Typescript__ in their palindrome implementations, I had to create similarly named methods for step 1 and step 3.

```
func IsStringAPalindromeCandidate(s string) bool
func IsFreqCharacterMapAPalindromeCandidate( freqCharacterMap map[string]int) bool
```

I do like method overloading in __Java__ and I use it a lot. I  don't know why __Go__ doesn't seem to support it. Same number of methods, but now there's redundant information in each method signature which seems to out of step with the language style.

### Nil is not null

As a __Java__ developer, I'm constantly aware of `null`, and frequently test for it. __Go__ doesn't have `null`, but it does have `nil` which I think of as `null` - even though its possibly not.

What's interesting about `nil` is how __Go__ makes it useful where it can. For example, in step 3, I want to know if we can make a palindrome from the frequency character map. In __Java__ I would first test for a null or an empty map, and return false if we had either. In __Go__, I only have to test for an __empty__ map.

```
func IsFreqCharacterMapAPalindromeCandidate( freqCharacterMap map[string]int) bool {
    if len( freqCharacterMap ) == 0 {
        return false
    }
    ...
```

__Go__ makes a `nil` map useful here by returning `zero` for its length. Likewise, I could have read the `nil` map, `freqCharacterMap["a"]`, which would have returned `zero` - in __Java__ I would have got a `NullPointerException`. However, I can't write to the `nil` map because there's no backing map - its `nil`. 

### Built-in support for unit testing

I'm cautious on my thoughts about this. Having built-in support makes unit testing a first-class citizen - which has to be a good thing. 

However, I'm used to the feature richness of JUnit/Jupiter with Mockito and AssertJ - setup and teardown, every sort of assertion you can imagine, mocking. (Maybe the complexity of the Java world leads to a need for equally complex frameworks and libraries).

Writing the unit tests in __Go__ was like stepping out of a luxury car and jumping on a push bike. I think the tests are good. They are certainly simple and there's no barrier to entry. For me, they just don't look or feel like real tests. 

Perhaps I'm just being a bit posh and need my feathers ruffled. For now, I'm going to assume there's a lot I don't know and, maybe, I need to change.

### Iterating over the map

In step 4, we need to iterate over the frequency character map twice. First, to find how long our palindrome will be - I could have passed in the length based on the original string of characters, but I elected not to do that. Second, to build the actual palindrome from the characters in the map. In my case, on the second iteration, __Go__ returned the map entries in a different order to the first iteration. This caused a problem in my unit tests for the method. I was assuming the entries would be in a known order - and in my solution in the other four languages this was the case. To overcome this, I had to introduce a sorted list of the map keys.

There's plenty more __Go__ to learn. Maybe there's a way to create and populate a sorted map.

### Strings

Something I feel I didn't handle well is the string of characters we want to make our palindrome from. In three of the methods I'm dealing with our string of characters. In one it gets converted to an array of bytes. In another, I think into a rune and then back into a string. In the final one, I build an array of strings - single character strings in this case - and then join each array of single character strings into a string. It all looks messy. There has to be a better way - which I will have to explore some other time.

```
func IsStringAPalindromeCandidate(s string) bool {
    matched, _ := regexp.Match(`^[a-z]*$`, []byte(s))
    ...
}
```
```
func FrequencyOfCharacterMap( data string ) map[string]int {
    ...
    for _, c := range data {
        freqMap[string(c)]++
    }
    ...
```
```
func MakePalindromeFromFreqCharacterMap( freqCharacterMap map[string]int ) string {
    ...
    palindrome := make([]string, palindromeLength)
    ...
    return strings.Join(palindrome, "")
```

### Summary

I might have found a new language that I'm at home with.

I found it easy to code my palindrome solution in __Go__. The language was concise but not cryptic. The nuances were manageable and made things simpler. For me, __Go__ lives up to its easy to learn and simple claims. I'll be spending more time discovering __Go__ and finding a place for it in my language toolbox.

Back to the __Go Class__ videos to learn some more __Go__.

___
#### Source code
https://github.com/jamie-burns0/new-era/tree/go/palindrome-interview-go
