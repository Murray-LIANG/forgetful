# Javascript Basic


## References

https://javascript.info

## Comparison of different types

When comparing values of different types, JavaScript converts the values to numbers.

## Strict equality `===` and `!==`

**Checks the equality without type conversion.** If `a` and `b` are of different types, the `a === b` immediately returns `false` without an attempt to convert them.

```javascript
alert(0 == false);  // true
alert('' == false);  // true
alert(0 === false);  // false
```

## Comparison with `null` and `undefined`
```javascript
alert(null === undefined);  // false
alert(null == undefined);  // true

// null converts to 0, while undefined converts to NaN.
// And NaN is a special numeric value which returns false for all comparisons.
alert(null > undefined);  // false
alert(null <= undefined);  // false
alert(null < undefined);  // false
alert(null >= undefined);  // false

alert( null > 0 );  // false
alert( null >= 0 ); // true, null converts to 0

// NOTE: null and undefined only equal `==` each other and do not equal
// any other value.
alert( null == 0 ); // false
alert( undefined == 0 ); // false
```

- Treat any comparison with `undefined/null` except the strict equality `===` with exceptional care.
- Don't use comparisons `>=` `>` `<` `<=` with a variable which may be `null/undefined`, unless you're really sure of what you're doing. If a variable can have these values, check for them separately.


## Objects

```javascript
let user = new Object();
let user = {};

/////
let user = {
    name: "John",
    age: 30,
    "likes birds": true  // must be quoted
};

user.name = "john";
user["likes birds"] = false;  // must use square brackets


/////
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
    [fruit]: 5,  // the name of the property is taken from the variable fruit
    "fruit": "unknown",  // just a property named "fruit"
};

alert( bag.apple );  // 5 if fruit="apple"
alert( bag[fruit] );  // 5 if fruit="apple"
alert( bag["fruit"] );  // unknown
alert( bag.fruit );  // unknown
```

## Property names limitations

Property names can be any strings or symbols.

Other types are automatically converted to strings.
```javascript
let obj = {
    0: "test"  // same as "0": "test"
};

alert( obj["0"] );
alert( obj[0] );
```

## Property existence

```javascript
let user = {};
// Reading a non-existing property just returns `undefined`.
alert( user.noSuchProperty === undefined );  // true


// Use `in` to check if a property exists.
alert( "noSuchProperty" in user );  // true


// User `for ... in` to visit every property.
let user = {
    name: "John",
    age: 30,
    isAdmin: true
};

for (let key in user) {
    alert( key );
    alert( user[key] );
}
```


## Properties order

Integer properties are sorted, others appear in creation order.

The `Integer property` here means a string can be converted to-and-from an integer without a change.

So, "49" is an integer property name. But "+49" and "1.2" are not.

```javascript
let codes = {
    "49": "Germany",
    "41": "Switzerland",
    "44": "Great Britain",
    // ..,
    "1": "USA"
};

for (let code in codes) {
    alert(code); // 1, 41, 44, 49
}

// Workaround.
let codes = {
    "+49": "Germany",
    "+41": "Switzerland",
    "+44": "Great Britain",
    // ..,
    "+1": "USA"
};

for (let code in codes) {
    // NOTE the `+code`
    alert( +code ); // 49, 41, 44, 1
}

```

## Objects are stored and copied `by reference` for non-primitive.

## Garbage collection

The basic garbage collection algorithm is called **`mark-and-sweep`**.

The following "garbage collection" steps are regularly performed:

1. The garbage collector takes `roots` and "marks" (remembers) them.
2. Then it visits and "marks" all references from them.
3. Then it visits marked objects and marks their references. All visited objects are remembered, so as not to visit the same object twice in the future.
4. …And so on until every reachable (from the `roots`) references are visited.
All objects except marked ones are removed.

Some optimizations are used, like `Generational collection`, `Incremental collection`, `Idle-time collection`.

## Object method
```javascript
let user = {
    name: "John",
    age: 30,

    sayHi() {  // object method definition
        alert(this.name);
    }
    // Or use this to define an object method.
    sayBye: function() {
        alert("Goodbye");
    }
};
```

## Constructor function

Just regular functions, but with two conventions:
1. Named with capital letter first.
2. Should be executed only with `"new"` operator.

```javascript
let user = new User("Jack");

alert(user.name); // Jack
alert(user.isAdmin); // false

function User(name) {
    // this = {};  (implicitly)

    // add properties to this
    this.name = name;
    this.isAdmin = false;

    // return this;  (implicitly)
}
```

## Optional chaining `"?."`

In other words, `value?.prop`:

- works as `value.prop`, if `value` exists,
- otherwise (when `value` is `undefined/null`) it returns `undefined`.

```javascript
let user = {};  // a user without "address" property

alert(user.address.street);  // Error!
alert( user.address?.street );  // undefined (no error)
```

## `"?.()"` and `"?.[]"`
```javascript
let userAdmin = {
    admin() {
        alert("I am admin");
    }
};

let userGuest = {};

userAdmin.admin?.(); // I am admin
userGuest.admin?.(); // nothing (no such method)

/////
let key = "firstName";

let user1 = {
  firstName: "John"
};

let user2 = null;

alert( user1?.[key] ); // John
alert( user2?.[key] ); // undefined
```

## Symbols

A `symbol` represents a unique identifier.

Symbols are guaranteed to be unique.

```javascript
let id1 = Symbol("id");
let id2 = Symbol("id");

alert(id1 == id2); // false
```

Symbols can be used for some **hidden** properties. These properties can be only accessed directly, cannot be accessed when another script loops over our object (e.g. in `for...in`).
```javascript
let id = Symbol("id");
let user = {
    name: "John",
    age: 30,
    [id]: 123
};

for (let key in user) alert(key);  // name, age (no symbols)

// The direct access by the symbol works.
alert( "Direct: " + user[id] );
```

### Global symbols

There is a registry and we can put the application-wide symbols in that registry. These symbols are called `global symbols`.
```javascript
// Read from the global registry.
let id = Symbol.for("id"); // if the symbol did not exist, it is created

// Read it again (maybe from another part of the code).
let idAgain = Symbol.for("id");

// The same symbol.
alert( id === idAgain ); // true


// Symbol.keyFor() is reverse of Symbol.key().
// Get symbol by name.
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// Get name by symbol.
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

## Object to primitive conversion

**All objects are `true` in a boolean context. There are only numeric and string conversions.**

There are three variants of type conversion, so-called "hints".
- `"string"`

    object to string conversion, when we're doing an operation on an object that expects a string.

- `"number"`
    
    object to number conversion, like when we're doing maths.

- `"default"`
  
    Occurs in rare cases when the operator is "not sure" what type to expect.

    For instance, binary plus `+` can work both with strings (concatenates them) and numbers (add them), so both strings and numbers would do. So if a binary plus gets an object as an argument, it uses the "default" hint to convert it.

    Also, if an object is compared using `==` with a string, number or a symbol, it's also unclear which conversion should be done, so the `"default"` hint is used.

    ```javascript
    // binary plus uses the "default" hint
    let total = obj1 + obj2;

    // obj == number uses the "default" hint
    if (user == 1) { ... };
    ```

The greater and less comparison operators, such as `<` `>`, can work with both strings and numbers too. Still, they use the `"number"` hint, not `"default"`. That’s for historical reasons.

In practice though, we don’t need to remember these peculiar details, because all built-in objects except for one case (`Date` object) implement `"default"` conversion the same way as `"number"`. **So if we treat `"default"` and `"number"` the same, like most built-ins do, then there are only two conversions.**


**To do the conversion, JavaScript tries to find and call three object methods:**

1. Call `obj[Symbol.toPrimitive](hint)` – the method with the symbolic key `Symbol.toPrimitive` (system symbol), if such method exists,
2. Otherwise if hint is `"string"`

    try `obj.toString()` and `obj.valueOf()`, whatever exists.

3. Otherwise if hint is `"number"` or `"default"`

    try `obj.valueOf()` and `obj.toString()`, whatever exists.


### Symbol.toPrimitive

There’s a built-in symbol named `Symbol.toPrimitive` that should be used to name the conversion method, like this:
```javascript
obj[Symbol.toPrimitive] = function(hint) {
  // must return a primitive value
  // hint = one of "string", "number", "default"
};


let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

alert(user);  // hint: string -> {name: "John"}
alert(+user);  // hint: number -> 1000
alert(user + 500);  // hint: default -> 1500
```

If there’s no `Symbol.toPrimitive` then JavaScript tries to find them and try in the order:

- `toString` -> `valueOf` for `"string"` hint.
- `valueOf` -> `toString` otherwise.

These methods must return a primitive value. If `toString` or `valueOf` returns an object, then it’s ignored (same as if there were no method). In contrast, `Symbol.toPrimitive` must return a primitive, otherwise there will be an error.

By default, a plain object has following `toString` and `valueOf` methods:

- The `toString` method returns a string `"[object Object]"`.
- The `valueOf` method returns the object itself.

### Further conversions

As we know already, many operators and functions perform type conversions, e.g. multiplication `*` converts operands to numbers.

If we pass an object as an argument, then there are two stages:

1. The object is converted to a primitive (using the rules described above).
2. If the resulting primitive isn't of the right type, it's converted.

For instance:
```javascript
let obj = {
  // toString handles all conversions in the absence of other methods
  toString() {
    return "2";
  }
};

alert(obj * 2); // 4, object converted to primitive "2", then multiplication made it a number
```
1. The multiplication `obj * 2` first converts the object to primitive (that’s a string `"2"`).
2. Then `"2" * 2` becomes `2 * 2` (the string is converted to number).

Binary plus will concatenate strings in the same situation, as it gladly accepts a string:
```javascript
let obj = {
  toString() {
    return "2";
  }
};

alert(obj + 2); // 22 ("2" + 2), conversion to primitive returned a string => concatenation
```

## Methods of primitives

7 primitive types: `string`, `number`, `bigint`, `boolean`, `symbol`, `null` and `undefined`.

It would be great to access to methods and properties of strings, numbers, booleans and symbols.

In order for that to work, a special "object wrapper" that provides the extra functionality is created, and then is destroyed. The "object wrapper" are different for each primitive type and are called: `String`, `Number`, `Boolean` and `Symbol`. Thus, they provide different sets of methods.

```javascript
let str = "hello";
alert(str.toUpperCase());  // HELLO
```
Here's what actually happens in `str.toUpperCase()`:

1. The string `str` is a primitive. So in the moment of accessing its property, a special object is created that knows the value of the string, and has useful methods, like `toUpperCase()`.
2. That method runs and returns a new string (shown by `alert`).
3. The special object is destroyed, leaving the primitive `str` alone.

So primitives can provide methods, but they still remain lightweight.

### Constructors `String`/`Number`/`Boolean` are for internal use only
For example, do not use `new Number(1)`, but you can use `Number("123")` to convert a string to number.

### `null`/`undefined` have no methods

## Numbers

### Regular numbers are `-(2^53 - 1) <= x <= 2^53 - 1`
Stored in 64-bit format IEEE-754, aka. "double precision floating point numbers". 52 bits are used to store the digits, 11 bits store the position of the decimal point (they are zero for integer numbers), and 1 bit is for the sign.

`BigInt` is used to represent integers not in regular number range.

### Supports `_` and `e`.
```javascript
let billion = 1_000_000_000;
let billion = 1e9;
```

### `toString(base)`

`base` can vary from `2` to `36`. By default it's `10`.
```javascript

alert(123456..toString(36));  // Note two dots are used here
alert((123456).toString(36));  // Same as above
```

### Imprecise calculations

```javascript
// For a number too big which would overflow the 64-bit storage,
// infinity is returned.
alert(1e500);  // Infinity

// Loss of precision
alert(0.1 + 0.2 == 0.3);  // false
alert( 0.1 + 0.2 ); // 0.30000000000000004
```

A number is stored in memory in its binary form, a sequence of bits – ones and zeroes. But fractions like `0.1`, `0.2` that look simple in the decimal numeric system are actually unending fractions in their binary form. Why?

`0.1` is one divided by ten 1/10, one-tenth. In decimal numeral system such numbers are easily representable. Compare it to one-third: 1/3. It becomes an endless fraction `0.33333(3)`.

So, division by powers 10 is guaranteed to work well in the decimal system, but division by 3 is not. For the same reason, in the binary numeral system, the division by powers of 2 is guaranteed to work, but 1/10 becomes an endless binary fraction.

There's just no way to store exactly `0.1` or exactly `0.2` using the binary system, just like there is no way to store one-third as a decimal fraction.

`toFixed(n)` could help work around the problem.
```javascript
let sum = 0.1 + 0.2;
alert( sum.toFixed(2) ); // 0.30, actually it is string "0.30"
alert( +sum.toFixed(2) ); // 0.3, use `+` to convert it to number
```

### `isFinite()` and `isNaN()`

Two special numeric values:
- `Infinity` (and `-Infinity`) is a special numeric value that is greater (less) than anything.
- `NaN` represents an error.

`isNaN(value)` converts `value` to a number and then tests it for being `NaN`. You can just use `=== NaN`. Because `NaN` is unique in that it does not equal anything, including itself:
```javascript
alert( NaN === NaN ); // false
```

`isFinite(value)` converts `value` to a number and returns true if it’s a regular number, not `NaN/Infinity/-Infinity`. It is used to validate whether a string value is a regular number:
```javascript
let num = +prompt("Enter a number", '');

// will be true unless you enter Infinity, -Infinity or not a number
alert( isFinite(num) );
```

**An empty or a space-only string is treated as `0` in all numeric functions including `isFinite`.**

### `Object.is` compares values like `===` but isn't exactly same with `===`

- It works with `NaN`: `Object.is(NaN, NaN) === true`, that's a good thing.
- Values `0` and `-0` are different: `Object.is(0, -0) === false`, technically that's true, because internally the number has a sign bit that may be different even if all other bits are zeroes.
- In all other cases, `Object.is(a, b)` is the same as `a === b`.


### `parseInt` and `parseFloat`

`alert(+"100px")` outputs `NaN`. Because `+` or `Number()` conversion is strict.

Then we have `parseInt` and `parseFloat` which "read" a number from a string until they can't. In case of an error, the gathered number is returned. The function `parseInt` returns an integer, whilst `parseFloat` will return a floating-point number:
```javascript
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12, only the integer part is returned
alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading
```

## Strings

### Immutable

### Comparing strings

All strings are encoded using UTF-16. That is: each character has a corresponding numeric code. There are special methods that allow to get the character for the code and back.

```javascript
// different case letters have different codes
alert( "z".codePointAt(0) ); // 122
alert( "Z".codePointAt(0) ); // 90

alert( String.fromCodePoint(90) ); // Z

// 90 is 5a in hexadecimal system
alert( '\u005a' ); // Z
```

```javascript
let str = '';

for (let i = 65; i <= 220; i++) {
  str += String.fromCodePoint(i);
}
alert( str );
// ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
// ¡¢£¤¥¦§¨©ª«¬­®¯°±²³´µ¶·¸¹º»¼½¾¿ÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖ×ØÙÚÛÜ
```
So, `a > Z` and `Ö > O`.

### Correct comparisons

Alphabets are different for different languages. So, the browser needs to know the language to compare.

`str1.localeCompare(str2[, locales[, options]])`

```javascript
const a = 'Österreich';
const b = 'Zealand';

console.log(a < b);  // false, means a > b
console.log(a.localeCompare(b));  // -1, means a < b


const a = 'réservé'; // with accents, lowercase
const b = 'RESERVE'; // no accents, uppercase

console.log(a.localeCompare(b));  // 1
console.log(a.localeCompare(b, 'en', { sensitivity: 'base' }));  // 0
```

## Arrays
```javascript
let arr = new Array();
let arr = ["apple", "pear"];

let fruits = ["Apple", "Orange", "Pear"];
fruits.pop();  // work with the end of the array
fruits.push("Lemon");  // work with the end of the array

fruits.shift();  // work with the beginning of the array
fruits.unshift("Lemon");  // work with the beginning of the array

fruits.push("Orange", "Lemon");  // add multiple elements at once
fruits.unshift("Orange", "Lemon");  // add multiple elements at once
```

### Internals

An array is **a special kind of object**. The square brackets used to access a property `arr[0]` actually come from the object syntax. That’s essentially the same as `obj[key]`, where `arr` is the object, while numbers are used as keys.

They extend objects providing special methods to work with **ordered collections** of data and also the `length` property. But at the core it’s still an object.

Remember, there are only eight basic data types in JavaScript. Array is an object and thus behaves like an object.

For instance, it is copied by reference:
```javascript
let fruits = ["Banana"]

let arr = fruits; // copy by reference (two variables reference the same array)
alert( arr === fruits ); // true

arr.push("Pear"); // modify the array by reference
alert( fruits ); // Banana, Pear - 2 items now
```

…But what makes arrays really special is their internal representation. The engine tries to store its elements **in the contiguous memory area**, one after another, just as depicted on the illustrations in this chapter, and there are other **optimizations** as well, to make arrays work really fast.

**But they all break if we quit working with an array as with an "ordered collection" and start working with it as if it were a regular object.**

For instance, technically we can do this:
```javascript
let fruits = []; // make an array
fruits[99999] = 5; // assign a property with the index far greater than its length
fruits.age = 25; // create a property with an arbitrary name
```
That’s possible, because arrays are objects at their base. We can add any properties to them.

**But the engine will see that we’re working with the array as with a regular object.** Array-specific optimizations are not suited for such cases and will be turned off, their benefits disappear.

The ways to misuse an array:

- Add a non-numeric property like `arr.test = 5`.
- Make holes, like: add `arr[0]` and then `arr[1000]` (and nothing between them).
- Fill the array in the reverse order, like `arr[1000]`, `arr[999]` and so on.

Please think of arrays as special structures to work with the ordered data. They provide special methods for that. **Arrays are carefully tuned inside JavaScript engines to work with contiguous ordered data, please use them this way. And if you need arbitrary keys, chances are high that you actually require a regular object `{}`**.

### A word about "length"
"length" is actually not the count of values in the array, but the greatest numeric index plus one.
```javascript
let fruits = [];
fruits[123] = "Apple";
alert(fruits.length);  // 124
```

If we decrease the `length`, the array is truncated, and the process is irreversible.
```javascript
let arr = [1, 2, 3, 4, 5];

arr.length = 2;
alert(arr);  // [1, 2]

arr.length = 5;
alert(arr[3]);  // undefined: the values do not return
```

### `new Array()`

If `new Array` is called with **a single argument which is a number**, then it creates an array **without items, but with the given length**.

```javascript
let arr = new Array(2);  // will it create an array of [2]? NO!!!
alert( arr[0] );  // undefined! no elements.
alert( arr.length );  // length 2
```

In the code above, `new Array(number)` has all elements undefined.

**To evade such surprises, we usually use square brackets, unless we really know what we’re doing.**

### toString
Arrays have their own implementation of `toString` method that returns a comma-separated list of elements.

For instance:
```javascript
let arr = [1, 2, 3];

alert( arr ); // 1,2,3
alert( String(arr) === '1,2,3' ); // true
```

Also, let’s try this:
```javascript
alert( [] + 1 ); // "1"
alert( [1] + 1 ); // "11"
alert( [1,2] + 1 ); // "1,21"
```
Arrays do not have `Symbol.toPrimitive`, neither a viable `valueOf`, they implement only `toString` conversion, so here `[]` becomes an empty string, `[1]` becomes `"1"` and `[1,2]` becomes `"1,2"`.

When the binary plus `"+"` operator adds something to a string, it converts it to a string as well, so the next step looks like this:
```javascript
alert( "" + 1 ); // "1"
alert( "1" + 1 ); // "11"
alert( "1,2" + 1 ); // "1,21"
```

### Don't compare arrays with `==`
Arrays in JavaScript, unlike some other programming languages, shouldn’t be compared with operator `==`.

This operator has no special treatment for arrays, it works with them as with any objects.

Let’s recall the rules:

- Two objects are equal `==` only if they’re references to the same object.
- If one of the arguments of `==` is an object, and the other one is a primitive, then the object gets converted to primitive, as explained in the chapter `Object to primitive conversion`.
- ...With an exception of `null` and `undefined` that equal `==` each other and nothing else.

The strict comparison `===` is even simpler, as it doesn't convert types.

So, if we compare arrays with `==`, they are never the same, unless we compare two variables that reference exactly the same array.

For example:
```javascript
alert( [] == [] ); // false
alert( [0] == [0] ); // false
```
These arrays are technically different objects. So they aren't equal. The `==` operator doesn't do item-by-item comparison.

Comparison with primitives may give seemingly strange results as well:
```javascript
alert( 0 == [] ); // true

alert('0' == [] ); // false
```
Here, in both cases, we compare a primitive with an array object. So the array `[]` gets converted to primitive for the purpose of comparison and becomes an empty string ''.

Then the comparison process goes on with the primitives, as described in the chapter `Type Conversions`:
```javascript
// after [] was converted to ''
alert( 0 == '' ); // true, as '' becomes converted to number 0

alert('0' == '' ); // false, no type conversion, different strings
```
So, how to compare arrays?

That's simple: don't use the `==` operator. Instead, compare them item-by-item in a loop or using iteration methods.

## Array methods

### `push`/`pop`, `shift`/`unshift`
### `splice`
    
`splice` will shift the rest of elements and make them occupy the freed place.

`arr.splice(start[, deleteCount, elem1, ..., elemN])`: It modifies `arr` starting from the index `start`: removes `deleteCount` elements and then inserts `elem1`, ..., `elemN` at their place. Returns the array of removed elements.
```javascript
let arr = ["I", "study", "JavaScript", "right", "now"];

arr.splice(1, 1); // from index 1 remove 1 element
// arr = ["I", "JavaScript", "right", "now"]

// remove 2 first elements and replace them with another
arr.splice(0, 2, "Let's", "dance");
// arr = ["Let's", "dance", "right", "now"]

// from index 2
// delete 0
// then insert "the" and "javascript"
arr.splice(2, 0, "the", "javascript");
// arr = ["Let's", "dance", "the", "javascript", "right", "now"]

// from index -4 (one step from the end)
// delete 0 elements,
// then insert "and" and "learn"
arr.splice(-4, 0, "and", "learn");
// arr = ["Let's", "dance", "and", "learn", "the", "javascript", "right", "now"]
```

### `slice`

`arr.slice([start], [end])` returns a new array copying to it all items from index `start` to `end` (not including `end`).

```javascript
let arr = ["t", "e", "s", "t"];

alert( arr.slice(1, 3) ); // e,s (copy from 1 to 3)

alert( arr.slice(-2) ); // s,t (copy from -2 till the end)

let arrB = arr.slice(); // make a copy of whole arr.
```

### `concat`

`arr.concat(arg1, arg2...)` returns a new array containing items from `arr`, then `arg1`, `arg2` etc. If an argument `argN` is an array, then all its elements are copied. Otherwise, the argument itself is copied.
```javascript
let arr = [1, 2];

// create an array from: arr and [3,4]
alert( arr.concat([3, 4]) ); // 1,2,3,4

// create an array from: arr and [3,4] and [5,6]
alert( arr.concat([3, 4], [5, 6]) ); // 1,2,3,4,5,6

// create an array from: arr and [3,4], then add values 5 and 6
alert( arr.concat([3, 4], 5, 6) ); // 1,2,3,4,5,6
```

Normally, it only copies elements from arrays. Other objects, even if they look like arrays, are added as a whole:
```javascript
let arr = [1, 2];

let arrayLike = {
  0: "something",
  length: 1
};

alert( arr.concat(arrayLike) ); // 1,2,[object Object]
```
But if an array-like object has a special `Symbol.isConcatSpreadable` property, then it’s treated as an array by `concat`: its elements are added instead:
```javascript
let arr = [1, 2];

let arrayLike = {
  0: "something",
  1: "else",
  [Symbol.isConcatSpreadable]: true,
  length: 2
};

alert( arr.concat(arrayLike) ); // 1,2,something,else
```

### `forEach`
`arr.forEach` allows to run a function for every element of the array.

```javascript
// for each element call alert
["Bilbo", "Gandalf", "Nazgul"].forEach(alert);

// Elaborate about their positions in the target array:

["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {
  alert(`${item} is at index ${index} in ${array}`);
});
```
