

#### "products types" vs "sum types"

#### Introduction

Programing languages from the "ML family" (SML, OCaml, F#, ... Haskell, ...) provide a feature called Algebraic Data Types (ADT). It's all about two main arithmetic operations: addition and multiplication. But not in the context of numbers, but in the context of data types. It's interesting that if you are an object-oriented programmer, you have been dealing with classes and interfaces. So you have been dealing with "product types" (multiplication). You maybe have been even working with "sum types" (addition) modeling your data types in a particular way. But it wasn't as easy as writing a class because your language didn't support syntax for it.

#### Products types

Let's take a look at very simple code below:

```typescript
interface OptionalNumber {
    hasValue: boolean;
    value?: number;
}
function tryParseNumber(text: string): OptionalNumber {
    const n = Number(text);
    return Number.isNaN(n) ? { hasValue: false } : { hasValue: false, value: n };
}
```

We just want to convert a text into a number. The text can be `123abc` so it's not a correct number value. `OptionalNumber` wraps the simple value of type `number` adding a `hasValue` flag. It stores information about whether the value really exists. `OptionalNumber` is an **interface** type and as I said at the beginning of this chapter, it's a "product type". So where is the operation of multiplication?

Let's say that **data type** is a set of correct values. `boolean` type contains two values, `true` and `false`. To simplify our example, let's say that the `number` type is an integer from the range of `0..255`. It is a set of 256 possible values. `OptionalNumber` is also a data type, it is like a composition of types `boolean` and `number`. It contains 512 (2 x 256) correct values. Why? Just look at those values, from `(true,0), (true,1) ...` until `(true,255)` and then from `(false,0), (false,1) ...` until `(false,255)`. It is a cartesian product of all possible values from two sets of values.

"Product types" are also known as "each of types" or "AND types". Every time you define a class `Person` with properties `name, age, address`, you can think of it as a combination of 3 values, one of `string` type, one of `number` type and one of `Address` type. 

Let's take a look once again at the signature of `tryParseNumber` function `string -> OptionalNumber`. This function tries to be honest with the caller. It's not returning `number` type pretending that everything is always ok, the same time throwing some exception for improper string representations of numbers. A mathematician would call this function "a total function" because it can be executed for any input argument value, not just the proper ones.

The problem is that the definition of the `OptionalNumber` type allows us to represent an invalid state. Property `value` is optional, the values `{hasValue: false, value: 123}` or `{ hasValue: true}` also belong to `OptionalNumber` set of values, but they represent invalid state. The programmers need to write code checking against the invalid state and this can be the source of many bugs. The fun fact is that the .net [Nullable](https://docs.microsoft.com/en-us/dotnet/api/system.nullable-1) type and Java [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) type are defined exactly this way. "Sum types" can be very useful in [Making Impossible States Impossible](https://www.youtube.com/watch?v=IcgmSRJHu_8).

##### Tuples

Many programming languages support feature called tuples. It's very useful when we want to combine together two, three or more values. Those values can potentially be of different types. Function `findMinAndMax(values: number[])` can return `{min:number; max:number}` but also `[number, number]`. It is just a pair of two numbers. In TypeScript where is no special syntax support for tuples. The pair of numbers is just an array of numbers with precisely two elements inside. We can think about the tuples as a special case of a class or interface whose properties are named `item1`,`item2`, .... In programming languages from ML family, the tuple type mentioned here would be written as `int * int`. After all, it is just yet another kind of "product type".

#### Sum types

##### TypeScript union types

TypeScript language supports a special feature called [union types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types). We can define very strangely looking type:

```typescript
type BooleanOrNumber = boolean | number;
```

Now let's say we have a variable `a: BooleanOrNumber`. What possible values can be assigned to this variable? It can be `true` or `false` but it can be also `0` or `1` ... until `256`. `BooleanOrNumber` type is a set of 258 (2 + 256) values. You have just learned the "sum types". They are also called "one of types", "OR types" or "choice types".

You are probably curious how we can work with such a `a` variable if it can be any of two types, `boolean` or `number`. Typical code would look like this:

```typescript
if (typeof a === "boolean") {
    // here 'a' is of type 'boolean'
} else {
    // here 'a' is of type 'number'
}
```

We need to be able to check what the type of variable is in a particular place of code. Thanks to the TypeScript language feature called "control flow analysis", the type of variable `a` can be inferred based on the usage context. `if` statement can change the type of variable.

##### TypeScript literal types

The other interesting feature of TypeScript language is [literal types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#string-literal-types)

```typescript
var b: "hidden" = "hidden";
```

We can think of variable `b` as it would be of type `string`. We can pass it into any place where `string` type is required or we can call on it any method from `string` type. `hidden` type is like a `string` type restricted to only one valid value,  namely `hidden`.

##### TypeScript discriminated unions

Union types can be combined with literal types in a few different ways. The example below shows one way of implementing **enum** type in TypeScript:

```typescript
type Visibility = "hidden" | "visible";
```

[Discriminated unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions) feature combines union types and literal types in a slightly more complicated way. We can think of TypeScript discriminated unions types as a canonical representation of "sum types" feature known from functional programming languages. Using  discriminated unions feature we can define a better version of `OptionalNumber` type like this: 

```typescript
type NumberOption =
    | { type: "some", value: number }
    | { type: "none" };

function tryParseNumber_(text: string): NumberOption {
    const n = Number(text);
    return Number.isNaN(n) ? { type: "none" } : { type: "some", value: n };
}
```

`NumberOption` type can be one of two possible cases. Each of them is an object type containing the same property of string literal type. In this particular example, the property is called `type`. It determines the specific case (union) and it will be used during the process of control flow analysis we will see in a moment.

The next function called `tryParsePositiveNumber` converts string into number only if the string value represents a correct positive value (`some` case) and the lack of value  (`none` case) is returned otherwise.

```typescript
function tryParsePositiveNumber(text: string): NumberOption {
    const nOption = tryParseNumber_(text);
    switch (nOption.type) {
        case "some": {
            // here 'nOption' is of type '{ type: "some", value: number }'
            return nOption.value < 0 ? { type: "none" } : { type: "some", value: nOption.value };
        }
        case "none": { 
            // here 'nOption' is of type '{ type: "none" }'
            return { type: "none" };
        }
    }    
}
```


There are two interesting things in the TypeScript code above. The first is the way how TypeScript compiler infers types. Inside each of `switch/case` statements the type of `nOption` variable is different. The value of `nOption.type` expression determines the type of `nOption` variable. In our previous example (`OptionalNumber` type),  it was possible to define values like `{hasValue: false, value: 123}` or `{ hasValue: true}`. Both of them were representing invalid states. Here  `NumberOption` type is defined as "sum type". Values like `{type: "none", value: 123}` or `{ type: "some"}` causes errors during compilation because they not match object schema.

The second thing is that the compiler forces us to handle both cases (`some` and `none`) inside the `switch/case` statement. If we comment the `none` case, we will get an error saying that this function can potentially return `undefined` value but the function signature informs only about `NumberOption`. It is because in JavaScript functions without `return` expression returns `undefined` by default. In general, this feature is called [exhaustiveness checking](https://www.typescriptlang.org/docs/handbook/advanced-types.html#exhaustiveness-checking). It's crucial in functional languages where there is very often a closed set of possible cases ("sum types" in this case) and the programmer should handle all cases to avoid exceptions at runtime. We will come back to this topic in a dedicated section speaking about **pattern matching**.

##### OOP and sum types

At this point, some of you may say "enough! we can also express 'sum types' in OOP languages". Sample implementation could look like this:

```typescript
class NumberOption { } 
class SomeNumberOption extends NumberOption {     
    value: number
    constructor(value: number) {
        super();
        this.value = value;        
    }
}
class NoneNumberOption extends NumberOption { }
```

Yes, this is true. Each case is like a class inheriting from base class representing "sum type". Each of often called "case class" can have optionally additional data. But the key point here is the amount of code we need to write to have "sum types" supporting all features functional programmer got used to. I am talking here about features like pattern matching, constructors, immutability, structural comparison and equality. None of them is supported by the OOP implemenation presented above. We will be talking about those features in the following sections in great detail. But for now, believe me, and just take a look at how extremely succinct F# implementation of  `NumberOption` type is. This definition supports all mentioned features.

```fsharp
type NumberOption =
 | None
 | Some of int
```

Comparing to other mainstream programming languages TypeScript is very unique in this regard. It allows to represent "sum types" in a very elegant way at the type system level. Type inference and control flow analysis work better than in most other static programming languages. After compilation to JavaScript "sum types" disappear and we have plain JavaScript objects. They can be easily serialized to and deserialized from JSON format, then stored on disk or sent through the wire 

##### Generic and recursive types (Option<T>, Result<T,E>, List<T>, Tree<T>)

Type `NumberOption` can only wrap numbers. If we want to wrap values of any type, we can define `Option<T>` as a generic type.

```typescript
type Option<T> =
    | { type: "some", value: T }
    | { type: "none" };
```

In all previous examples type `NumberOption` can be changed into `Option<number>` and the code will compile correctly.

`Result<T,E>`  is another very popular type in languages from ML family.

```typescript
type Result<T, E> =
    | { type: "ok", value: T }
    | { type: "error", err: E };
```

It is like `Option<T>` type but it can store some additional information when there is no value (`none` case). Function `tryParsePositiveNumber` returned `none` case when the text was not a correct number or a positive number. We didn't know what the true reason was. Know we can change the implementation to use  `Result<T,E>` type.

```typescript
function tryParsePositiveNumber_(text: string): Result<number, string> {
    const nOption = tryParseNumber_(text);
    switch (nOption.type) {
        case "some": {
            return nOption.value < 0 ? { type: "error", err: "number is negative" } :
                { type: "ok", value: nOption.value };
        }
        case "none": {
            return { type: "error", err: "text is not a number" };
        }
    }
}
```

Just by reading the signature of the function we know a lot about the function. Ok, it is not perfect because the error is stored as a string so we have to parse and extract detailed information. But still, maybe it is better than throwing an exception at runtime. We can always use our own custom type as an error, for instance `enum ParseError { NotANumber, NegativeNumber}` or `type  ParseError_ = { type: "notANumber" } | { type: "negativeNumber" }`. 

`List<T>` type a very simple implementation of a linked list. 

```typescript
type List<T> =
    | { type: "cons"; head: T; tail: List<T> }
    | { type: "empty" };
```

It is an example of a recursive type because inside the definition of the type the same type is used. Case `const`  is a shortcut for the word "constructor". The collection of item like `["a","b"]` can be represented as `List<string>` type the following way `{ type: "cons", head: "a", tail: { type: "cons", head: "b", tail: { type: "empty" } } }`. I know, the definition looks a little bit verbose but no worries we will simplify it when we come to the constructor functions.

`Tree<T>` type a very simple representation of a binary tree.

```typescript
type Tree<T> =
    | { type: "node"; value: T; left: Tree<T>, right: Tree<T> }
    | { type: "emptynode" }
```

Those four examples are always presented when it comes to explaining "sum types" in any book or course about ML family language. TypeScript language supports generic types and allows us to define recursive sum types. I think we have everything we need to create the definition of "sum types" in a very convenient and succinct way.

##### Option<T> and nullability

`Option<T>` type allows expressing potential lack of value. It is heavily used in functional programming languages like OCaml, F#, Elm or Haskell. Sometimes it is named as `Maybe<T>` type with two cases, `just` and `nothing`. It is interesting that the new mainstream languages like Swift, Kotlin, Rust or even lately C# 8.0 are treating `null` value in a special way.

Let's take look at the typical example showing the problem with `null` value. We model an entity called `Person` containing tree properties `firstName`, `lastName` and `middleName`. All of them are of `string` type.

```typescript
interface Person {
    firstName: string;
    lastName: string;
    middleName: string;
}
```
Now let's say we want to write a function called `formatPersonInfo(person: Person): string` returning a summary text information for a passed argument of type `Person`. Suddenly we get `NullReferenceException`, it's because the `middleName` property is null. It is very unlikely that the property `firstName` or `lastName` will be null. Our UI and business logic validate the data and even SQL database schema does not allow NULL values. But it is very possible that `middleName` will be `null` because a lot of people just not have a middle name. Just by looking at the definition of `Person` type the programmer can not see the difference between `middleName` property and two other properties. All of then are `string` type and by default, the string type can always be `null`. But we could assume that `null` (the lack of value) value can never be assigned to any variable until it is declared explicitly as `Option<T>`. So in our example, `middleName` could be declared as `Option<string>`. The property needs to be treated differently than the other two properties because the property type is different. Every time someone is using properties `firstName` and `lastName`, he can be sure the values are not null. But the property `middleName` has to be handled in a special way in case of the absence of value. Thanks to beeing explicit in choosing types for our data model, we lower the chance of getting `NullReferenceException` at runtime.

In world of function programming languages, type `Option<T>` is very often called **a monad type** . That means it provides two special methods called `map` and `bind`. Thanks to them work with nullable values is very convenient because we can write something like this `person.middleName.map( n => n.toUpperCase())` or even this `person1.middleName.bind( n1 => person2.middleName.map(n2 => n1 + " " + n2) )`. Lambda functions are called only in case of value presence (`some` case) so the lack of values is propagated automatically returning new `Option<T>` at the end. The details informations about this topic can be found in a dedicated chapter about Monads.

##### Constructors and immutability

Defining sum types as discriminated unions in TypeScript is very succinct and elegant. We use JavaScript object literal as `{type: "none"}` to create an instance of sum type. In ML family languages for each case of sum type we get automatically the constructor function to create an instance of sum type. For instance, previously presented F# type `type NumberOption = None | Some of int` provides a function called `Some`. It takes `int` argument and returns `NumberOption` so it can be executed like this  `Some 10`. It would be nice to have this feature in TypeScript too.

But just before we start. I would like to point out one very important thing. You don't have to use the constructor functions at all. In JavaScript creating object instances using object literal is very convenient, it is like a code pattern seen everywhere. Instances of sum types are just instances of JavaScript objects. TypeScript gives us type checking and an intellisense inside IDE at compile time. If we define some sum type and then create the instance of it in one or two places, maybe the additional functions are completely unnecessary. An example of such a case is when we model the actions using Redux pattern. Some actions are used only once or twice inside the entire codebase. In my opinion, modeling them as discriminated unions without constructor functions is a very good option. We don't use any classes, enums, functions, ...  or other external tools and libraries. Just pure TypeScript. More information about this topic can be found [here](https://github.com/ngrx/platform/issues/723).

But there are situations where the constructor functions can be very useful. For instance, when we use a general purpose sum types like `Option<T>` or `Result<T,E>` types. Very often those types are used all over the place. So it's very handy to call just `some(123)` instead of `{type: "some", value: 123}` or `none` instead of `{type: "none"}`. And even more, because `some` is a just regular function, we can write something like this `[1,2,3].map(some)`.

What I am trying to say here is think what works for you. I really like the way how the sum types can be expressed using only discriminated unions in TypeScript. The definition of custom sum type in TypeScript looks almost identical to the definition in any ML programming language. In those languages, we get the constructor functions for free. In TypeScript, we need to make some extra steps to have it. So take a look at a few ways how those steps could look like.

First off, let's see how constructor functions would look like for a non-generic sum type like `type NumberOption = { type: "some", value: number } | { type: "none" };`

```typescript
const some = (value: number): NumberOption => ({ type: "some", value });
const none = (): NumberOption => ({ type: "none" });
```

Defining manually those function can be a very cumbersome and error-prone task, especially for sum types with many cases.  **powerfp** library provides a code generator tool than can automatically generate those functions based on the type definition placed in TypeScript source file. The generator tool uses a TypeScript compiler to parse source code and create AST (Abstract Syntax Tree) of sum type. Then it analyses AST, extracts the structure of the sum type and generates the source code of constructor functions. Running the generator for `NumberOption` type would generate the following code:

```typescript
type Option_some = UnionChoice<NumberOption, "some">;
const some = (value: number) => ({ type: "some", value }) as Option_some;
type Option_none = UnionChoice<NumberOption, "none">;
const none = { type: "none" } as Option_none;
```

There are two main differences between this code and the previously written code by hand:
- the `none` case is not a function but it is a const value. It makes sense when you think about it for a minute. Assuming that object instance created for `none` case is immutable (the object can not be change), returning the new instance each time the constructor function is called would be an unnecessary waste of memory. `none` case does not have any additional properties so the eventual constructor function would always return an object with the same structure
- the functions or const values are not of type `NumberOption` but of specially generated types like `Option_some` and `Option_none`. Type `Option_some` is an equivalent of `{ type: "some", value: number }`, type `Option_none` is an equivalent of `{ type: "none" }` type. The idea here is to restrict returned types to specific case types. Usage of those types is not mandatory but can be very useful. The magic `UnionChoice` type used above is a part of **powerseq** library. It accepts two type arguments. The first one is the name of the sum type, the second one is the name of the union case that will be extracted from the specified sum type. You don't have to fully understand the definition of this type. If you are interested, the definition is [here](https://github.com/marcinnajder/powerfp/blob/master/src/types.ts) and it looks something like this (`Extract` type comes from the TypeScript standard library):

```typescript
type TypedObj<T = string> = { type: T };
type UnionChoice<T extends TypedObj<string>, TT extends T["type"]> = Extract<T, { type: TT }>;
```

This solution is not perfect. We need to use some external tools like the code generator. The second solution starts with the opposite end. We define constructor functions ourself than infer the type of sum type from those functions.

```typescript
const some = (value: number) => ({ type: "some", value } as const);
const none = { type: "none" } as const;
```

The code is almost the same as the code automatically generated by the tool. The only difference is the `as const` keyword. TypeScript language provides an amazing type inference capabilities so we can write fewer type annotations. If we define a variable as `const none = { type: "none" }`, the type of this variable will be inferred as `{type: string}`. TypeScript library contains the definition of a generic type `Readonly<T>` so any type `T` can be converted to a "read-only" version of itself. The type `Readonly<{type: string}>` is equivalent to the type `{readonly type: string}`. All properties are marked with **readonly** keyword so they cannot be changed after initialization. The type `Readonly<{type: "none"}>` is equivalent to the type `{readonly type: "none"}`. Note that the type of `type` property is restricted to literal type `"none"` instead of type `string`. The construct `as const` is just a shorthand for wrapping inferred type into `Readonly<T>` type. The real returned type of `some` function is inferred as `{readonly type: "some", readonly value: number}` and the type of variable `none` is inferred as`{readonly type: "none"}`. Thanks to this `type` property type restriction process, we can define `NumberOption_` type based on the construction functions.

```typescript
type NumberOption_ = SumType<typeof none | typeof some>;
```

Once again, `SumType` type comes from **powerfp** library and it is not crucial to understand sum types topic. But if you are curious, it looks like this:

```typescript
type SumType<T> = T extends (...args: any) => any ? ReturnType<T> : T;
```

We read this type the following way: if type `T` is a function, return result type of this function. Otherwise, return type `T`. `ReturnType<T>` is built-in TypeScript type.

So we have constructor functions and sum types at the same time without the necessity of using any external tools. We just use  TypeScript type system features. That is pretty impressive. But there is one problem. This solution does not work for generic types. Let's take a look a constructor functions for `Option<T>` type:

```typescript
const none = { type: "none" } as const;
const some = <T>(value: T) => ({ type: "some", value } as const);
```

And let's try to define sum type like this:

```typescript
type Option<T> = SumType<typeof none | typeof some<T>>;
```

But unfortunately, this is not a correct TypeScript code. We can not write code like `typeof some<T>`. Only code `typeof some` is correct. But even if it would compile successfully, it does not work as expected. `typeof some` is equivalent to `{readonly type: "some", readonly value: unknown}`.  The type of `value` property is `unknown`,  it is because we didn't use type `T` at all.  Fortunately, we can use some workaround to make it work. 

```typescript
class O<T>{
    some() {
        return some<T>(null as any);
    }
}
type Option<T> = SumType<typeof none | O<T>["some"]>;
```

Thanks to generic helper class `O<T>` we can infer correctly the type of `some` function.  The function `some` needs to be closed over `T` type used in the definition of `Option<T>` type. The same trick works for `Resule<T,E>` type.

```typescript
const ok = <T>(value: T) => ({ type: "ok", value }) as const;
const error = <E>(err: E) => ({ type: "error", err }) as const;
class R<T, E>{
    ok() {
        return ok<T>(null as any);
    }
    error() {
        return error<E>(this as any);
    }
}
type Result<T, E> = SumType<R<T, E>["ok"] | R<T, E>["error"]>;
```

Unfortunately, this tick does not work for recursive types like `List<T>` or `Tree<T>`. If we try to do this, we will get a compiler error. It is because those types are ... recursive. Let's take a look at definition of `List<T>` type: 

```typescript
type List<T> = { type: "cons"; head: T; tail: List<T> } | { type: "empty" };
```

`const<T>(head: T, tail: List<T>)` constructor function would take an argument `tail` of type `List<T>`. The definition of type `List<T>` would look like this:

```typescript
type List<T> = SumType<TClass<T>["const"] | typeof empty>
```

`TClass<T>` helper class would have method `const` and this method would use constructor function `const` defined at the beginning.  So we have a full cycle  `const<T>` -> `List<T>` -> `TClass<T>["const"]` -> `const<T>`.

Let's summarize what we have seen so far. For each sum type, it is very convenient to have a set of constructor functions. One function for each sum type case. The code of these functions can be generated automatically or can be written by hand. We can infer the definition of sum type from the constructor functions but it is only possible for non-recursive types. For generic types, we need to define some extra helper class. For me personally, it is not a big problem because this class is not exported from the JavaScript module file where it has been defined. It is used only for the definition of sum type and probably it will be deleted by "tree shaking" tools during the process of building a production version of application anyway.

In the end, I would like to mention one issue. At some point, we have used TypeScript syntax like `as const` so each case of sum type is wrapped into `Readonly<T>` type. This way sum type is defined as immutable so we can not change the value of any property. We got the immutable version sum type because it has been built from the constructor functions. We can always use `Readonly<T>`  type we use the first approach defining sum type manually as a discriminated union type. 

```typescript
type Option<T> = Readonly<{ type: "some", value: T } | { type: "none" }>
```

Immutability is one of the key features of functional programming and it is described in great detail in one of the chapters. In most functional programming languages immutability is used by default. In TypeScript, objects are mutable by default. The code above defining an immutable version of `Option<T>`type is not as elegant as the mutable version. Especially when it comes to sum types with many cases. It all depends on your preferences. Immutability is a really nice feature but it requires some extra code in TypeScript. But there is one danger case with mutable sum types. When the sum type has no additional properties besides `type` property (this is a more typical case than you think) there will be no constructor function but only a constant value. `none` case of `Option<T>` type is an example of such a situation. One singleton version of the object is used everywhere in your application so the property `type` of this object should have never be changed. To summarize, immutability at the TypeScript type-level helps.

##### Pattern matching 


Pattern matching is a language feature coming from functional programming languages. It allows us to match an instance of object to many possible patterns (kinds of objects). If any of the patterns match, an appropriate value is returned or piece of code is executed. It is like  `if/then/else` or `switch/case` statement extended with many additional features. Most functional programming languages support different kinds of patterns for different kinds of data types (such as records, tuples, collections, sum types, ...). There is a whole separate chapter about pattern matching in this book. Right here we will be talking about pattern matching only in the context of sum types. TypeScript is a JavaScript. It does not support pattern matching natively but it is powerful enough to emulate it in various ways. 

Sum type represents one value from a closed set of possible values. The typical code working with sum type executes some action for each of the possible cases. The point here is to always remember to handle all cases. If someone adds a new case to the sum type in the future, we expect to get an error forcing us to handle it. In the previous part of this chapter, we saw the implementation of `tryParsePositiveNumber` function.

```typescript
function tryParsePositiveNumber_(text: string): Result<number, string> {
    const nOption = tryParseNumber_(text);
    switch (nOption.type) {
        case "some": {
            return nOption.value < 0 ? error("number is negative") : ok(nOption.value);
        }
        case "none": {
            return error("text is not a number");
        }
    }
}
```
Now let's implements the same function using a helper function called `matchUnion`

```typescript
function tryParsePositiveNumber__(text: string): Result<number, string> {
    const nOption = tryParseNumber_(text);
    return matchUnion(nOption, {
        some: ({ value }) => value < 0 ? error("number is negative") : ok(value),
        none: () => error("text is not a number")
    });
}
```

The function` matchUnion` emulates pattern matching. It takes a value of any sum type and the JS object representing all possible case branches. The name of the property corresponds to the value of `type` property of the passed sum type. The value of the property can be a function or just a value. The function takes an argument matching the case being currently handled. In the example above, first function takes `{type:"some", value:T}` type and the second takes `{type:"none"}` type. Because the second function is not using its argument, the function can be replaced by just the value so the code would look like this `none: error("text is not a number"),`. The type of  `matchUnion` call corresponds to the type returned by the function handler. In this case, it is `Return<number, string>`. If you forget to handle one of the possible cases, the TypeScript compiler returns an error.

There is also one very important distinction between code using `switch/case` and the code using `matchUnion` call. Unfortunately in JS `switch/case` (and also `if/then/else`) is a statement. It doesn't return any value so it can't be used as a variable assignment or the value returned from a function. In many functional languages, those statements are actually expressions. And because function call in JS is an expression, it can be used in all those places. Our previous example could be shorten using just expressions the following way:

```typescript
const tryParsePositiveNumber___ = (text: string): Result<number, string> =>
    matchUnion(tryParseNumber_(text), {
        some: ({ value }) => value < 0 ? error("number is negative") : ok(value),
        none: error("text is not a number")
    });
```

There are some situations where we don't want to handle each case differently. Sometimes we want to handle one or two cases in a specific way and then we provide a handler for all other cases. It is like `default:` clause in `switch/case` statements. In the context of pattern matching it is very often called **wildcards**.

Let's say we have built-in functionality allowing us to read the content of text files. In case of an error, `readFileContent` function returns detailed information about the reason behind the problem.

```typescript
declare type FileError =
    | { type: "fileNotFound", filePath: string }
    | { type: "pathTooLong" }
    | { type: "unauthorizedAccess" };

declare function readFileContent(filePath: string): Result<string, FileError>;
```

Now let's write a simple code reading the content of some configuration file and then printing some information into the console.

```typescript
const configFileOrError = readFileContent("config.json");
if (isUnion(configFileOrError, "ok")) {
    // here 'configFileOrError' is of type '{ type : "ok", value : string }'
    console.log("Config file content:", configFileOrError.value);
} else {
    // here 'configFileOrError' is of type '{ type : "error", err : string }'
    const errorMessage = matchUnion(configFileOrError.err, {
        fileNotFound: ({ filePath }) => `file '${filePath}' not found`,
        _: e => `some other error of type '${e.type}'`
    })
    console.error(errorMessage);
}
```


We introduced a new function called `isUnion`. This function returns `boolean` value. The value `true` is returned if the first argument matches the specified union, value `false` is returned otherwise. This function is implemented as **type guard** in TypeScript so the compiler knows which type the variable `configFileOrError` inside each of branches.

We have also used `matchUnion` function passing JS object with a **wildcard** `_` property. Handler function for `_` is used when the value of property `type` of the first argument has no matching property in JS object.  `matchUnion` function is defined in such way that the existence of `_` property JS object changes how this object is validated via TypeScript compiler. It does not require to provide handlers for all possible cases of sum types. If the value of this property `_` is a function, this function takes an argument of sum type passed into `matchUnion` function. In this example the compiler knows for sure that `e` argument has property `type`, it is a common property across all types describing the unions.

At the end, let's look at the implementations of `matchUnion` and `isUnion` functions. Both of them are provided by the **powerfp** library. The implementation can change over time, here we just look at the current simplified implementation. 

```typescript
type TypedObj<T = string> = { type: T };
type UnionChoice<T extends TypedObj<string>, TT extends T["type"]> = Extract<T, { type: TT }>;
type ValueOrFunc<T, R> = R | ((value: T) => R);
type ExhaustiveMatchTypedObj<T extends TypedObj<string>, R> = {
    [P in T["type"]]: ValueOrFunc<Extract<T, { type: P }>, R>;
}
type MatchTypedObj<T extends TypedObj<string>, R> =
    | ExhaustiveMatchTypedObj<T, R>
    | (Partial<ExhaustiveMatchTypedObj<T, R>> & { _: R | ((union: T) => R) });

function matchUnion<T extends TypedObj, R>(unionType: T, obj: MatchTypedObj<T, R>): R {
    if (unionType.type in obj) {
        return callValueOrFunc((obj as any)[unionType.type], unionType);
    } else if ("_" in obj) {
        return callValueOrFunc(obj["_"], unionType);
    } else {
        throw new Error(`Matching union type failed: '${Object.keys(obj).join(",")}' type not found, only '${unionType.type}' types were specified`);
    }
}

function callValueOrFunc<T, R>(value: ValueOrFunc<T, R>, unionType: T): R {
    return typeof value === "function" ? (value as f<T, R>)(unionType) : value;
}

function isUnion<T extends TypedObj, Type extends T["type"]>(x: T, unionType: Type): x is UnionChoice<T, Type> {
    return x.type === unionType;
}
```

You don't have to understand it fully, especially if you are not familiar with more advanced TypeScript features like [mapped types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types).  `MatchTypedObj<T,R>` type describes JS object passed as a second argument to `matchUnion` function. It maps some union type `T` into JS object whose properties correspond to all possible cases. Type `R` says what type will be returned from the `matchUnion` function. When we look just at the function body, it is trivial JS code. It is petty amazing how expressive TypeScript type system is and how well type inference works. And remember, after the compilation process only simple JS code will remain.

#### Summary

Once again, I want to point out how powerful "sum types" feature is. Most programmers were using only "product types" with classes, interfaces or tuples. It is like using only half of our brain. I definitely encourage you to read an amazing book [Domain modeling made functional](https://pragprog.com/book/swdddf/domain-modeling-made-functional) or at least watch the  [this](https://fsharpforfunandprofit.com/video/#domain-modeling-made-functional) video. I will change the way you model your data forever.

