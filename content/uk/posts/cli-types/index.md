---
layout: post
title: "Built-in .NET types"
description: "Ints, strings, bools, etc"
date: 2012-07-10
nav: fsharp-types
seriesId: "Understanding F# types"
seriesOrder: 10
---

In this post we'll take a quick look at how F# handles the [standard types that are built into .NET](http://msdn.microsoft.com/en-us/library/hfa3fa08%28VS.80%29.aspx).

## Literals

F# uses the same syntax for literals that C# does, with a few exceptions.

I'll divide the built-in types into the following groups:

* miscellaneous types (`bool`, `char`, etc. )
* string types
* integer types (`int`, `uint` and `byte`, etc)
* float types (`float`, `decimal`, etc)
* pointer types (`IntPtr`, etc)

The following tables list the primitive types, with their F# keywords, their suffixes if any, an example, and the corresponding .NET CLR type.

### Miscellaneous types

{{<rawtable>}}
<table class="table table-condensed table-striped">
<colgroup>
<col width="5em">
</colgroup>

<tr>
<th></th>
<th>Object</th>
<th>Unit</th>
<th>Bool</th>
<th>Char<br>(Unicode)</th>
<th>Char<br>(Ascii)</th>
</tr>
<tr>
<td>Keyword</td>
<td>obj</td>
<td>unit</td>
<td>bool</td>
<td>char</td>
<td>byte</td>
</tr>
<tr>
<td>Suffix</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>B</td>
</tr>
<tr>
<td>Example</td>
<td>let o = obj()</td>
<td>let u = ()</td>
<td>true false</td>
<td>'a'</td>
<td>'a'B</td>
</tr>
<tr>
<td>.NET Type</td>
<td>Object</td>
<td>(no equivalent)</td>
<td>Boolean</td>
<td>Char</td>
<td>Byte</td>
</tr>
</table>
{{</rawtable>}}

Object and unit are not really .NET primitive types, but I have included them for the sake of completeness.

### String types

{{<rawtable>}}
<table class="table table-condensed table-striped">
<colgroup>
<col width="5em">
</colgroup>
<tr>
<th></th>
<th>String<br>(Unicode)</th>
<th>Verbatim string<br>(Unicode)</th>
<th>Triple quoted string<br>(Unicode)</th>
<th>String<br>(Ascii)</th>
</tr>
<tr>
<td>Keyword</td>
<td>string</td>
<td>string</td>
<td>string</td>
<td>byte[]</td>
</tr>
<tr>
<td>Suffix</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>Example</td>
<td>"first\nsecond line"</td>
<td>@"C:\name"</td>
<td>"""can "contain"" special chars"""</td>
<td>"aaa"B</td>
</tr>
<tr>
<td>.NET Type</td>
<td>String</td>
<td>String</td>
<td>String</td>
<td>Byte[]</td>
</tr>
</table>
{{</rawtable>}}

The usual special characters can be used inside normal strings, such as `\n`, `\t`, ``\\`, etc. Quotes must be escaped with a backslash:``\'`and`\"`.

In verbatim strings, backslashes are ignored (good for Windows filenames and regex patterns). But quotes need to be doubled.

Triple-quoted strings are new in VS2012. They are useful because special characters do not need to be escaped at all, and so they can handle embedded quotes nicely (great for XML).

### Integer types

{{<rawtable>}}
<table class="table table-condensed table-striped">
<colgroup>
<col width="5em">
</colgroup>
<tr>
<th></th>
<th>8 bit<br>(Signed)</th>
<th>8 bit<br>(Unsigned)</th>
<th>16 bit<br>(Signed)</th>
<th>16 bit<br>(Unsigned)</th>
<th>32 bit<br>(Signed)</th>
<th>32 bit<br>(Unsigned)</th>
<th>64 bit<br>(Signed)</th>
<th>64 bit<br>(Unsigned)</th>
<th>Unlimited<br>precision</th>
</tr>
<tr>
<td>Keyword</td>
<td>sbyte</td>
<td>byte</td>
<td>int16</td>
<td>uint16</td>
<td>int</td>
<td>uint32</td>
<td>int64</td>
<td>uint64</td>
<td>bigint</td>
</tr>
<tr>
<td>Suffix</td>
<td>y</td>
<td>uy</td>
<td>s</td>
<td>us</td>
<td></td>
<td>u</td>
<td>L</td>
<td>UL</td>
<td>I</td>
</tr>
<tr>
<td>Example</td>
<td>99y</td>
<td>99uy</td>
<td>99s</td>
<td>99us</td>
<td>99</td>
<td>99u</td>
<td>99L</td>
<td>99UL</td>
<td>99I</td>
</tr>
<tr>
<td>.NET Type</td>
<td>SByte</td>
<td>Byte</td>
<td>Int16</td>
<td>UInt16</td>
<td>Int32</td>
<td>UInt32</td>
<td>Int64</td>
<td>UInt64</td>
<td>BigInteger</td>
</tr>
</table>
{{</rawtable>}}

`BigInteger` is available in all versions of F#. From .NET 4 it is included as part of the .NET base library.

Integer types can also be written in hex and octal.

* The hex prefix is `0x`. So `0xFF` is hex for 255.
* The octal prefix is `0o`. So `0o377` is octal for 255.


### Floating point types

{{<rawtable>}}
<table class="table table-condensed table-striped">
<colgroup>
<col width="5em">
</colgroup>
<tr>
<th></th>
<th>32 bit<br>floating point</th>
<th>64 bit (default)<br>floating point</th>
<th>High precision<br>floating point</th>
</tr>
<tr>
<td>Keyword</td>
<td>float32, single</td>
<td>float, double</td>
<td>decimal</td>
</tr>
<tr>
<td>Suffix</td>
<td>f</td>
<td></td>
<td>m</td>
</tr>
<tr>
<td>Example</td>
<td>123.456f</td>
<td>123.456</td>
<td>123.456m</td>
</tr>
<tr>
<td>.NET Type</td>
<td>Single</td>
<td>Double</td>
<td>Decimal</td>
</tr>
</table>
{{</rawtable>}}

Note that F# natively uses `float` instead of `double`, but both can be used.

### Pointer types

{{<rawtable>}}
<table class="table table-condensed table-striped">
<colgroup>
<col width="5em">
</colgroup>
<tr>
<th></th>
<th>Pointer/handle<br>(signed)</th>
<th>Pointer/handle<br>(unsigned)</th>
</tr>
<tr>
<td>Keyword</td>
<td>nativeint</td>
<td>unativeint</td>
</tr>
<tr>
<td>Suffix</td>
<td>n</td>
<td>un</td>
</tr>
<tr>
<td>Example</td>
<td>0xFFFFFFFFn</td>
<td>0xFFFFFFFFun</td>
</tr>
<tr>
<td>.NET Type</td>
<td>IntPtr</td>
<td>UIntPtr</td>
</tr>
</table>
{{</rawtable>}}

## Casting between built-in primitive types

*Note: this section only covers casting of primitive types. For casting between classes see the series on [object-oriented programming](/series/object-oriented-programming-in-fsharp/).*


There is no direct "cast" syntax in F#, but there are helper functions to cast between types. These helper functions have the same name as the type (you can see them in the `Microsoft.FSharp.Core` namespace).

So for example, in C# you might write:

```csharp
var x = (int)1.23
var y = (double)1
```

In F# the equivalent would be:

```fsharp
let x = int 1.23
let y = float 1
```

In F# there are only casting functions for numeric types. In particular, there is no cast for bool, and you must use `Convert` or similar.

```fsharp
let x = bool 1  //error
let y = System.Convert.ToBoolean(1)  // ok
```


{{< linktarget "boxing" >}}

## Boxing and unboxing

Just as in C# and other .NET languages, the primitive int and float types are value objects, not classes. Although this is normally transparent, there are certain occasions where it can be an issue.

First, lets look at the transparent case. In the example below, we define a function that takes a parameter of type `Object`, and simply returns it. If we pass in an `int`, it is silently boxed into an object, as can be seen from the test code, which returns an `object` not an `int`.

```fsharp
// create a function with parameter of type Object
let objFunction (o:obj) = o

// test: call with an integer
let result = objFunction 1

// result is
// val result : obj = 1

```

The fact that `result` is an object, not an int, can cause type errors if you are not careful. For example, the result cannot be directly compared with the original value:

```fsharp
let resultIsOne = (result = 1)
// error FS0001: This expression was expected to have type obj
// but here has type int
```

To work with this situation, and other similar ones, you can convert a primitive type to an object directly, by using the `box` keyword:
```fsharp
let o = box 1

// retest the comparison example above, but with boxing
let result = objFunction 1
let resultIsOne = (result = box 1)  // OK
```

To convert an object back to an primitive type, use the `unbox` keyword, but unlike `box`, you must either supply a specific type to unbox to, or be sure that the compiler has enough information to make an accurate type inference.

```fsharp
// box an int
let o = box 1

// type known for target value
let i:int = unbox o  // OK

// explicit type given in unbox
let j = unbox<int> o  // OK

// type inference, so no type annotation needed
let k = 1 + unbox o  // OK
```

So the comparison example above could also be done with `unbox`. No explicit type annotation is needed because it is being compared with an int.

```fsharp
let result = objFunction 1
let resultIsOne = (unbox result = 1)  // OK
```

A common problem occurs if you do not specify enough type information -- you will encounter the infamous "Value restriction" error, as shown below:

```fsharp
let o = box 1

// no type specified
let i = unbox o  // FS0030: Value restriction error
```

The solution is to reorder the code to help the type inference, or when all else fails, add an explicit type annotation. See [the post on type inference for more tips](/posts/type-inference/#troubleshooting-summary).

### Boxing in combination with type detection

Let's say that you want to have a function that matches based on the type of the parameter, using the `:?` operator:

```fsharp
let detectType v =
    match v with
        | :? int -> printfn "this is an int"
        | _ -> printfn "something else"
```

Unfortunately, this code will fail to compile, with the following error:

```fsharp
// error FS0008: This runtime coercion or type test from type 'a to int
// involves an indeterminate type based on information prior to this program point.
// Runtime type tests are not allowed on some types. Further type annotations are needed.
```

The message tells you the problem: "runtime type tests are not allowed on some types".

The answer is to "box" the value which forces it into a reference type, and then you can type check it:

```fsharp
let detectTypeBoxed v =
    match box v with      // used "box v"
        | :? int -> printfn "this is an int"
        | _ -> printfn "something else"

//test
detectTypeBoxed 1
detectTypeBoxed 3.14
```

