# The palindrome interview in Java, Python, Typescript and Rust

In a recent interview - which I call the Palindrome Interview - I was presented with a string and asked for a solution that identified if it was possible to make a palindrome from all characters of the string and, if so, print out a single palindrome. Afterward, I thought about my solution and, also, what that solution would look like in other coding languages. This is a bit of a retrospective of my solution, comparing my implementation in Java, Python, Typescript and Rust.

For links to the source code in GitHub, see the end of the document.

Overall, I navigated __Java__ the best – it’s familiar - and __Rust__ the worst – I felt like Alice going down the rabbit hole. __Python__ and __Typescript__ were the surprises to me. I thought I would navigate Typescript better than Python. It turned out to be other way around.

Returning to hands-on coding after some years away, I would call __Java__ my wheelhouse. The other three I would say are newish to new to me. I’ve used JavaScript over the years as needed, but not got into __Typescript__. In the past, I think I’ve looked at __Python__ code and worked through a hello world tutorial. More recently, I’ve been working through HackerRank Python exercises. __Rust__ is the new kid on the block for me.

My solution to the palindrome interview is,

1. Validate we have an alphabetic string
2. Parse the string into a map of characters with their frequency
3. Validate we have a frequency character map that will produce a palindrome
4. Make a palindrome from the frequency character map

### Overloading methods

In __Java__, (1) and (3) were implemented as overloaded methods,

```
public boolean isNotPalindromeCandidate(String data)
...
public boolean isNotPalindromeCandidate(Map<Character, Long> frequencyOfCharactersMap)
...
```

While __Python__ doesn’t support overloaded methods, there is a way to do it which I think works ok

```
@singledispatch
def is_not_palindrome_candidate(data) -> bool:
...
@is_not_palindrome_candidate.register
def _(data: str) -> bool:
...
@is_not_palindrome_candidate.register
def _(frequency_of_character_map: dict) -> bool:
...
```

Neither __Typescript__ or __Rust__ supported overloaded methods, or if they did, I decided to stay away from how to make that happen. In the end I just created a similarly named method for (1) and (3)

```
// Typescript
function stringIsNotAPalindromeCandidate( data: string | null ): Boolean
function mapIsNotAPalidromeCandidate( map: Map<string, number> | null ): boolean

// Rust
pub fn string_is_not_a_palindrome_candidate( data: Option<&str> ) -> bool
pub fn map_is_not_a_palindrome_candidate( map: &HashMap<String,u32>) -> bool
```

### Nulls and no nulls

Coming from a __Java__ background null is a thing and I’m always aware that null can be lurking. In __Typescript__ null is also a thing. However, in __Python__ and __Rust__ null is not a thing.

In its place __Python__ has the `None` constant that represents the absence of a value or a null value. For me, it’s maybe a distant cousin of a Java `Optional` providing a safe way of representing and handling a null value.

__Rust__ just goes straight to `Option` which can be either `Some` which contains a value or `None` which does not contain a value.

In both __Python__ and __Rust__ I can’t get a null pointer exception because null just doesn’t exist. That’s a nice feature and would have saved me many, many hours in my developer career.

Having and not having null changes what’s in my unit tests.

```
// Java
@Test
void nullStringIsNotAPalindromeCandidate() {
    assertTrue(ps.isNotPalindromeCandidate((String) null));
}
...
@Test
void makePalidromeFromNullStringReturnsEmptyOptional() {
    assertTrue(ps.makePalindromeFrom(null).isEmpty());
}

// Typescript
test('null string is not a palindrome candidate', () => {
    expect( stringIsNotAPalindromeCandidate( null ) ).toBeTruthy();
});
...
test('null string is converted to an empty character frequency map', () => {
    const expectedResult = new Map<string,number>();
    expect ( toCharacterFrequencyMap( null ) ).toEqual(expectedResult); 
});
...
test('null map is not a palindrome candidate', () => {
    expect( mapIsNotAPalidromeCandidate( null ) ).toBeTruthy();
});

# Python
def test_is_not_palindrome_candidate_when_data_is_null():
    assert is_not_palindrome_candidate(None) == True
...
def test_make_palindrome_from_1():
    assert make_palindrome_from(None) == None

// Rust
#[test]
fn none_string_is_not_a_palindrome_candidate() {
    assert_eq!(string_is_not_a_palindrome_candidate(None), true);
}
```

### Simple-complex spectrum

Parsing the string into a character frequency map and later creating a palindrome from the character frequency map are where each language rose or fell.

On the parsing to a map, __Python__ was one line and four words

```
return dict(Counter(data))
```

where data is our string. That impressed and signalled to me that Python is trying to make things easier.

At the other end of that spectrum was __Rust__. I assume it’s clever, but I needed Copilot to provide me with a solution and, while I can see what it’s doing, I don’t know how much Rust exposure I would need to have to come up with the solution myself. The Rust documentation warns the reader that its ownership strategy for memory management is not familiar, but the penny will drop. I think that’s a fair warning. Owning, borrowing, mutating, non-mutating, referencing and pointing were all in the six lines of code that parsed the string into a map.

__Typescript__ was nearly as easy as Python – four lines with just one of them doing the heavy lifting.

```
for( const character of data ) {
        map.set( character, (map.get(character) || 0) + 1 );
}
```

I liked the use of `map.get(character) || 0` to either return the value of the entry for character or initialise it to zero (0).

For __Java__, I think it lies toward the Rust end of the spectrum. It’s elegant in its own `Stream` api way, however, if I wasn’t familiar with the `Stream` apis, this would have felt as complicated as the Rust solution.

```
return data.chars()
    .mapToObj(i -> (char) i)
    .collect(
        Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        )
    );
```

### Functional expressions

We iterate over the map frequency values and confirm that there are only zero or one odd numbers.

On this occasion the __Java__ `Stream` api, plain old __Typescript__ and __Rust__ provided, for me at least, simple and intuitive functional solutions.

```
// Java
return frequencyOfCharactersMap.entrySet().stream()
    .filter( e -> e.getValue() % 2 == 1 )
    .count() > 1;
```

__Python__ provides a solution using list comprehensions.

```
unpaired_count = sum(
    1
    for frequency_of_character in frequency_of_character_map.values()
    if frequency_of_character % 2 == 1
)
```

The list comprehension syntax,

`[expression sequence condition]`

seems functional and simple, it just doesn’t read with the familiar, and I think, intuitive flow of

`sequence-condition-expression`

that both Java and Rust follow.

### Primitive integer types

Final step is to make a palindrome from the characters and their frequency. For this we want an array of empty characters the same length as the total frequency – ie the length of our original string – then iterating over the character frequency map entries, fill the array from the two ends working inwards. If we have a frequency that’s an odd number, we are going to put an instance of that key in the middle of our array.

All four languages were kind of the same here. Above, I called out running into Rust ownership strategy for memory management. On this occasion, I ran into __Rust__ variable types. There just seems to be so many. If I compare integer primitives,

| | integer types |
|-|-|
|Rust| u8, u16, u32, u64, u128, usize, i8, i16, i32, i64, i128, isize|
|Java| int, long|
|Python| int|
|Typescript| number|

I probably made a bad choice for the type of my frequency value in Rust. I chose u32. Summing all the frequency values gave me a u32. However, the index for each element of my palindrome character array was a usize. I found myself constantly casting my u32 to usize.
Fortunately, all languages inferred types for me and did save me a lot of work.  

### Summing a list of values

For summing the frequency values, __Java__ and __Typescript__ were like each other. __Python__ and __Rust__ were like each other and much simpler.

```
// Java
var palindromeLength = frequencyOfCharactersMap.values().stream().mapToInt(Long::intValue).sum();

// Typescript
const palindromeLength: number = Array.from(map.values()).reduce( (s,v) => s + v, 0 );

# Python
palindrome_length = Counter(character_frequency_map).total()

// Rust
let palindrome_length: u32 = map.values().sum();
```

### Creating and initialising a character array

For creating and initialising the palindrome character array, __Java__ was easy as. In __Python__, __Typescript__ and __Rust__ I had to explicitly initialize the array elements.

```
// Java
var palindrome = new char[palindromeLength];

# Python – using list comprehension again
palindrome = ["" for _ in range(palindrome_length)]

// Typescript
const palindrome: string[] = new Array<string>(palindromeLength).fill('');

// Rust – using the vec macro
let mut palindrome: Vec<char> = vec![' '; palindrome_length as usize];
```

### Converting a character array into a string

For returning the palindrome character array as a string.

```
// Java
return String.valueOf( palindrome );

# Python
return "".join(palindrome)

// Typescript
return palindrome.join('');

// Rust
return String::from_iter(palindrome.iter());
```

### Final

In summary, 

__Java__: familiar and felt verbose

__Python__: I can see the appeal and would like to know more

__Typescript__: Sits somewhere between Java and Python

__Rust__: It's going to be a long journey - I’m still seeing white rabbits

___
#### _Source code links_

|||
|-|-|
|Java| https://github.com/jamie-burns0/new-era/tree/java/palindrome-interview|
|Python| https://github.com/jamie-burns0/new-era/tree/python/python/palindrome-interview|
|Typescript| https://github.com/jamie-burns0/new-era/tree/typescript/palindrome-interview|
|Rust| https://github.com/jamie-burns0/new-era/tree/rust/rust/palindrome-interview/palindrome_interview|