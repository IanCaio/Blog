---
layout: blogpost
title:  "C11 - Exploring the standard"
author: "Ian Caio"
date:   2018-03-12 23:38:24 -0300
category: "Others"
comment-section: "https://iancaio.github.io/blog-comments/get/post7comments.json"
comment-post-link: "https://api.staticman.net/v2/entry/IanCaio/blog-comments/master/post7comments"
header-image: "/imgs/posts/headerOthers.png"
---
C is often an overlooked programming language, being taught in some places (very wrongly) as an introductory
one. For that reason, some intermediate and advanced topics of the language give place to a lot of doubt
amongst newcomers. The standard is meant to define the language and guide developers through the path of
programming compliant code that can be widely supported, but they are not always an easy read. This article
is the first from a serie of articles where I'll explore parts of the standard and unveil
good coding practices and mispractices.

One thing that as beginners we don't tend to notice about C, is how abstract it is. Most of the time, we are
trying to correlate the code we write in C with the behavior of the resulting program on the execution environment
we are executing it in. We end up confusing what C _is_ with what it _does_,
and start creating unfounded assumptions about the language.

For this article, I will be discussing the ISO C11 standard, more precisely its latest draft (n1570) before
release. The draft is [available online](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf) for free,
while the official ISO release is only available for purchase. This draft is really close to the final release,
so it's a good resource to begin with.

The C11 standard is divided in 7 sections:

1. [Scope](#section-1---scope)
2. [Normative References](#section-2---normative-references)
3. [Terms, definitions, and symbols](#section-3---terms-definitions-and-symbols)
4. [Conformance](#section-4---conformance)
5. Environment
6. Language
7. Library

In this article, I'll talk about sections 1 to 4, starting to set the ground to the bulk of the standard:
The language description itself. Some sections will either be lightly discussed or not discussed
at all. Others will be discussed only focusing on concepts that I consider a little harder to understand. The reason
for that is that a full breakdown of the standard would definitely not fit a blog article format. I'll try
not to let any critical parts out.

# Section 1 - Scope

This small section will specify what is the scope of the document. It basically states that there are not
constraints on the environments where C programs run. However, the representation of the program itself, including
the semantics, syntax, and representation of input/output data processed by the program will be determined.

Things that won't be specified are for example, the way C source codes are transformed to usable executables,
the way C programs are invoked and how input/output data is transformed before being parsed to C programs.
It basically draws the line where the abstraction meets the real world application.

<figure class="center large">
<img src="{{ site.baseurl }}/imgs/posts/7/Cabstraction.svg" alt="C Abstraction" />
<figcaption>C Abstraction</figcaption>
</figure>

In the image above, C11 would care about the behavior of the C program and how the data reaches and leaves
the C program, but not about how it's converted or what the system data looks like.

# Section 2 - Normative References

In this section there is a list of ISO documents that might be needed to understand some parts of the
C standard document.

# Section 3 - Terms, definitions, and symbols

Some terms are defined before being used exclusively for the C11 ISO document. Most of them are easy
to understand in the way they are described, but I'd like to shed some light on a few of them:

## Argument vs Parameter

Maybe a small source of doubts since we might think of them as synonyms at first. Both of them are associated to
C function calls or preprocessor function-like macros. Being very simplistic, arguments are expressions evaluated when
a function is called that set the parameters values. While parameters themselfs are the objects that acquire the value of
those expressions when the function is entered.

Similarly, for function-like macros, arguments are preprocessor tokens that will set the value of the parameters
when a function-like macro is called and parameters are the identifiers that acquire the value of those preprocessor tokens.

For example:

```c
int add (int a, int b) {
	return a + b;
}

int main(int argc, char **argv){
	int result = add((2 * 3), 3);
	return 0;
}
```

In the code above, `a` and `b` are parameters: Objects that will acquire the values when the
function is called. While the expressions `(2 * 3)` and `3` are arguments to a function call. They
will assign the values `6` and `3` to the parameters `a` and `b` respectively. Those will be
accessible from the moment we enter the function to the moment we leave it.

Now, suppose we have a function-like macro:

```c
#define min(A, B) ((A) < (B) ? (A) : (B))

int x = min(23, 45);
```

In the macro above, `A` and `B` are parameters, while `23` and `45` are arguments. The
token `A` will acquire the value of the preprocessor token `23` and the token `B` the value
of the preprocessor token `45`. Be aware that __preprocessor tokens are not runtime variables__. The following
code __wouldn't work__:

```c
#define min(A, B) ((A) < (B) ? (A) : (B))

int a = 23;
int b = 45;
int x = min(a, b); // The preprocessor isn't aware of the values of 'a' and 'b', so this is invalid.
```

## Memory Location

The concept of memory location is very important as a matter of insurance against _race conditions_. Here, the
programmer will know what objects are guaranteed to be stored in different memory locations, so we can choose
carefully which objects can be accessed simultaneously in multiple threads.

_Scalar types_ and _adjacent bit-fields of non-zero length_ are the two types of objects that will
occupy a singular memory location. I wrote a detailed article about memory locations
[here](https://iancaio.github.io/blog/others/2018/03/11/c11-memory-location.html).

Basically, we can be sure that accessing two different _scalar type_ objects between threads simultaneously is safe.
Accessing different _bit-fields_ on its turn, is only safe if they are separated by either an object of another type
or another bit-field of length 0. Otherwise (if they are adjacent with a non-zero length), there's the possibility of
data races happening because they might be stored in the same memory location.

## Implementation-defined behavior, Locale-specific behavior, Unspecified behavior and Undefined behavior

### Implementation-defined behavior

The standard doesn't specify which action should be taken, giving a set of possibilities, but each implementation
__should document__ what their choice is. Those can be relied on only for a particular implementation, as long as other implementations
are not important for your program. Different implementations might share the same behavior but, since they are not constrained
by the standard, updates to the implementation could break code that relies on it. It would also not be as portable.

### Locale-specific behavior

Action is decided according to local conventions (language, nationality or culture).
It should also be documented by the implementation.
An example of that is the return value of the function __islower__ when used in characters other than the 26 Latin characters.

It's usually acceptable to rely on this type of behavior if the affected part of the program isn't critical, but more tightly
related to the _user interface_. For example, using __islower__ to enforce that an username is written in lower-case characters.
The fact that a character is lower-case is not likely critical to the program functionality (if it's, maybe relying on locale-specific
behavior isn't a good idea!).

### Unspecified behavior

The standard doesn't specify the action to be taken, giving implementations two or more
possibilities. There are no constraints on which to choose. _Implementation-defined behaviors_ are a type of
unspecified behavior, where the implementation documents the choice.

### Undefined behavior

Commonly known as UB, _undefined behavior_ is a result of erroneous code or data. A behavior can be
determined as undefined behavior explicitly, when the standard states that a particular action results in undefined
behavior, or implicitly, when the program construct take a course of action that is not predicted or
specified in the standard. Any particular situation that is not described by the standard is UB.

## Trap Representation

> __3.19.4 trap representation__
>
> an object representation that need not represent a value of the object type

The meaning of _value_ is defined a little earlier on:

> __3.19 value__
>
> precise meaning of the contents of an object when interpreted as having a specific type

So, if a trap representation doesn't need to represent a value of the object type, it means that it doesn't
describe the contents of the object in the scope of the objects type.

The standard later ellaborates more on trap representations:

> __6.2.6 Representations of Types__  
> __6.2.6.1 General__  
>
> 5 Certain object representations need not represent a value of the object type. If the stored value of an object has such a representation and is read by an lvalue expression that does not have character type, the behavior is undefined. If such a representation is produced by a side effect that modifies all or any part of the object by an lvalue expression that does not have character type, the behavior is undefined. Such a representation is called a trap representation.

A trap representation is a representation of an object that does not describe a valid value of that particular objects type. If an _lvalue_ of that
type is used to read the trap representation we have undefined behavior.

Imagine that we had a ficticious object type called `even int`. It would be 8-bits long and used to represent even numbers __only__. Some valid values of
this particular object type would be: `0x00000000` (0), `0x00000010` (2), `0x00000100` (4), and so on. It's possible in memory to represent this object
with an odd number (like `0x00000011` for example), but we would have a trap representation, because this is not a valid value of this particular
object type. There is no `even int` that can be represented by `0x00000011`, so trying to read it using an _lvalue_ of this type would result
in undefined behavior.

This concept will be more explored later in the standard, and is more complex than it looks. But having an idea of what it means will help
when we get to the other sections of the standard that gives more details about trap representations.

# Section 4 - Conformance

This section makes it clearer the wording used to describe requirements and prohibitions of the standard ("shall" and "shall not"),
gives a more detailed description of undefined behavior (violation of "shall" or "shall not" requirements, behavior indicated by
the words "undefined behavior" or omission of any description of the behavior by the standard) and also states that the
preprocessing translation unit shouldn't be compiled in the presence of an `#error` preprocessor directive, unless
it's skipped by some conditional.

Then, there's a differentiation between _strictly conforming programs_ and _conforming programs_:

A _strictly conforming program_ is a program that doesn't use anything that is not specified in the C11 standard. It doesn't cause any
unspecified behavior, undefined behavior or implementation-defined behavior:

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/7/strictlyconforming.svg" alt="Strictly Conforming Program" />
<figcaption>Strictly Conforming Program</figcaption>
</figure>

A _conforming program_ on the other hand, is a program that can be accepted by a _conforming implementation_. It might contain unspecified,
undefined and implementation-defined behavior, and for that reason it might depend on non-portable features. If portability is an important
matter, we should always aim to have strictly conforming programs.

There are two types of _conforming implementations_, hosted and freestanding:

- _Conforming hosted implementation_: Should accept any strictly conforming program.
- _Conforming freestanding implementation_: Should accept any strictly conforming program in which the use of the libraries is confined to `<float.h>`, `<iso646.h>`, `<limits.h>`, `<stdalign.h>`, `<stdarg.h>`, `<stdbool.h>`, `<stddef.h>`, `<stdint.h>` and `<stdnoreturn.h>`.

# Conclusion

The first sections of the standard aren't very dense, but they start building the fundamental concepts that will help the reader
through the next sections, so it's important to really understand them. In the next articles I'll continue to write about the following
sections of the standard.

On section 5, translation environments and execution environments will be described. It will give a clearer idea of how C programs
are compiled and how they are executed.
Section 6 will start talking about the features of the language itself (i.e.: types, casting, operators), and is the most extensive
part of the standard. Finally, section 7 will ellaborate on the C standard libraries.
