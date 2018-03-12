---
layout: blogpost
title:  "C11 - Memory Location"
author: "Ian Caio"
date:   2018-03-11 20:49:36 -0300
category: "Others"
comment-section: "https://iancaio.github.io/blog-comments/get/post6comments.json"
comment-post-link: "https://api.staticman.net/v2/entry/IanCaio/blog-comments/master/post6comments"
header-image: "/imgs/posts/headerOthers.png"
---
While writing the introductory blog article about the C11 standard, I noticed there was a concept
defined in Section 3 that required some deeper explanation. At some point the complexity of the
discussion grew to a point where it would be better explained having an article of its own. _Memory
Location_ is an important definition in the scope of memory management and multi-threads, specially when
it comes to avoiding Race Conditions.

# Race Conditions

Most Operating Systems will break down any
[user land](https://en.wikipedia.org/wiki/User_space) attempt to modify memory in 3 steps: _Reading_ the value in
memory, _Modifying_ the value in the registers and _Writing_ the new value back to memory.

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/6/memoryoperation.svg" alt="Memory Operation" />
<figcaption>Memory Operation Steps</figcaption>
</figure>

When only one thread is modifying a memory location, the fact those 3 steps happen separately doesn't matter.
However, if we have more than one thread modifying the same memory location we have something called a
[_Race Condition_](https://en.wikipedia.org/wiki/Race_condition),
where one of the steps from a thread might be happening between the steps from another thread.

| Thread 1         | Thread 2         |   | Memory Value |
|------------------|------------------|---|--------------|
|                  |                  |   | 0            |
|       Read       |                  | ← | 0            |
|                  |       Read       | ← | 0            |
| Modify (value++) |                  |   | 0            |
|                  | Modify (value++) |   | 0            |
|       Write      |                  | → | 1            |
|                  |       Write      | → | 1            |

This type of bug is very hard to track, because it behaves differently each time depending mostly on the synchronization
of threads. A simple C code that demonstrates the issue is:

```c
#include <pthread.h>
#include <stdio.h>

// Memory shared between the two threads
int number = 0;

// Increases the value of the memory location by 1
void increase(int *a){
	*a = *a + 1;
}

// Thread entry point: Runs "increase" 10.000 times
void *thread_entry(void *args){
	for(int i = 0; i < 10000; i++){
		increase(&number);
	}

	return NULL;
}

// Main thread: Starts a new thread, runs "increase" 10.000 times, wait for
// the thread to do the same and print the resulting value.
int main(void){
	pthread_t thread1;

	pthread_create(&thread1, NULL, thread_entry, NULL);

	for(int i = 0; i < 10000; i++){
		increase(&number);		
	}

	pthread_join(thread1, NULL);

	printf("Value of number: %u\n", number);

	return 0;
}
```

Without being aware of the race condition, one could expect
that the value of `number` would be 20000, considering it's being increased 10000 times by each thread. In practice,
the value will actually be 20000 or less, depending on the threads synchronization.
Here's the output of the program after being executed 7 times in a row:

```
$: ./race_condition
Value of number: 15290
$: ./race_condition
Value of number: 20000
$: ./race_condition
Value of number: 19674
$: ./race_condition
Value of number: 19320
$: ./race_condition
Value of number: 12550
$: ./race_condition
Value of number: 19407
$: ./race_condition
Value of number: 15036
```

The only way to ensure that this bug doesn't happen, is to either use _atomic operations_ (which in most Operating Systems
is reserved for the Kernel Space) or using _memory locks_. I'll not get in detail about each, because I'd end up writing another
article here (maybe I'll write something on the subject in the future).

Now, for race conditions to happen, one of the requirements is that we are accessing the same memory location in different
threads. Because of that, C11 standard will have a definition of memory location, so we can know which objects are
stored in different memory locations and which objects might be sharing the same location.

# Memory Location

The term memory location refers to the smallest piece of memory that we can manipulate
individually in a system. Usually, it will be 8-bits long (a byte). But the C11 standard was written with
the idea of being an abstraction that gives freedom to the systems that adopt it. It doesn't care if the system
can modify a minimum of 8-bits at once, or 4-bits, or 32-bits. What it cares about is that some objects should occupy a memory
location of their own, so the programmer can be sure that modifying that object won't have a side effect in another
object.

Be aware that two objects can occupy the same memory location __without overlaping__. Their values won't overflow on each
others space, but modifying both objects simultaneously in different
threads will not be safe, because the system will be modifying the same memory block.

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/6/memorylocation.svg" alt="Memory Location" />
<figcaption>Two objects sharing the same memory location</figcaption>
</figure>

The definition of memory location in the standard is:

> __3.14 memory location__
> 
> either an object of scalar type, or a maximal sequence of adjacent bit-fields all having nonzero width

Let's break down what this actually means.

## Scalar Types

The first step to understand the definition of a memory location, is to understand what _scalar types_ are.

Extending the concept from linear algebra, we can imagine that _scalar types_ are types that represent a single unit of something.
This is not very far from the meaning in C.

> __6.2.5 Types__
>
> ... 20. Arithmetic types and pointer types are collectively called scalar types. Array and structure types are collectively called aggregate types.

Scalar types are the union of the _arithmetic types_ and _pointer types_. Pointer types don't require much explanation to this context. A pointer is
a reference to an object of a specified type. All pointers have a _pointer type_.

So, what are _arithmetic types_?

> __6.2.5 Types__
>
> ... 17. The type char, the signed and unsigned integer types, and the enumerated types are collectively called _integer types_.
>
> ... 18. Integer and floating types are collectively called _arithmetic types_.

In the code below we have examples of variables that have _arithmetic types_ and _pointer types_, collectivelly known as _scalar types_.

```c
// SCALAR TYPES:
// Arithmetic Types
char a;
unsigned int b;
signed int c;
long int d;
float e;
double f;
enum month currentMonth;
// Pointer Types
char *p1;
int *p2;
float *p3;
int (*funcPtr)(char *, int);
```

All the objects above are guaranteed to be stored in different memory locations, so it's safe to access one in a thread and __another one__
in another thread. It's __not safe__ to access the same object in two different threads without _memory locks_ though, as explained before.

### The _array type_ case

I only recommend reading this section if you are very curious and feel like questioning every dot in an 'i'.

When I read about scalar types and realized that array types were not part of it (they are instead _aggregate types_), I started questioning
myself whether the standard was giving the programmer any explicit guarantee that different arrays would occupy
different memory locations. I came to a conclusion after some research on the ISO document, and the answer is _no_, it's _not_ guaranteed
that two arrays will occupy different memory locations.

To actually understand what I mean, we have to assume that most likely everything we learned about arrays is __wrong__. Just like in many subjects
we learn things in a practical way and not in an accurate way. People will first learn C to make C programs, and not to understand core concepts of
the C language. For that reason we have a completely misguided definition of arrays in our heads.

> __6.2.5 Types__
>
> ... 20. Any number of derived types can be constructed from the object and function types, as follows:
>  
> \- An array type describes a contiguously allocated nonempty set of objects with a particular member object type, called the element type. The element type shall be complete whenever the array type is specified. Array types are characterized by their element type and by the number of elements in the array. An array type is said to be derived from its element type, and if its element type is T , the array type is sometimes called ‘‘array of T ’’. The construction of an array type from an element type is called ‘‘array type derivation’’.

An array __describes__ a contiguously allocated nonempty set of objects, but it __isn't__ the allocated set itself. Every element of the array is a
singular object of a type specified in the array declaration. The internal representation of the array is basically not constrained, every implementation
can decide how to represent arrays, as long as they describe the contiguously set of objects and respect the rules of the standard regarding arrays
(like implicit type conversion, indexing, etc.).

We sometimes learn about arrays thinking they are a special kind of pointer to the beginning of a stream of objects. That makes a lot of sense when we
think about indexing: If the array points to `0x000000A0`, the fifth element will be the `0x000000A0` address summed with five times the size of the
element type. Perfect! It makes even more sense when we learn that accessing the array without an index returns a pointer to the beginning of the array.

```c
int array[10];
int *p = array; // Pointer now refers to the beginning of the array
```

The above code works the way it does because of the following rule:

> __6.3 Conversions__  
> __6.3.2 Other operands__  
> __6.3.2.1 Lvalues, arrays, and function designators__
>
> ... 3. Except when it is the operand of the sizeof operator, the \_Alignof operator, or the unary & operator, or is a string literal used to initialize an array, an expression that has type ‘‘array of type’’ is converted to an expression with type ‘‘pointer to type’’ that points to the initial element of the array object and is not an lvalue. If the array object has register storage class, the behavior is undefined.

An array type object is implicitly converted to a pointer type in the situations described above, which adds up to our notion of arrays being
special pointers. But they don't have to be..

It helped me a lot to imagine a practical example that was different from what we are used to seeing, but still possible (and inefficient). Imagine
that you are creating an implementation of C. But you decide that every array will be represented by an alphabet letter and you'll retrieve their addresses
from a table. In practice, the user will see the same results when using the array, but the internal representation will be completely different.

| Array Representation | Memory address of first element | Identifier |
|----------------------|---------------------------------|------------|
| a                    | 0x7C3247C0                      | myArray    |
| b                    | 0x7C329DFA                      | otherArray |
| c                    | 0x7C326F15                      | names      |
| d                    | 0x7C325D48                      | numbers    |
| ...                  | ...                             | ...        |

Now, if we try to access the array `myArray[i]`, our implementation will see that we are accessing the array represented by letter `a`. Its address
is `0x7C3247C0`, from where you can reach any element.
If we write `myPointer = myArray` somewhere in the code, the implicit type conversion will return a pointer to the element in `0x7C3247C0`.
However, if you run `&myArray` you'll actually have the address of the array representation, not `0x7C3247C0`, because the implicit conversion
doesn't happen with the unary operator `&`.

The array representation (the letters) can share the same memory location, and it doesn't matter, because we are __never__ modifying the
array type object, but the elements that it describe. On the standard section __6.3.2.1.1__ it's stated that a _modifiable lvalue_ can't
have an array type. We can't change the value of the array, only the value of its elements, hence there's no need to have a separate memory location
for each array representation.

```c
int array[3] = { 1,2,3 };
int array2[3] = { 3,2,1 };

array = array2; // THIS IS INVALID!!!
```

But the array elements have the _element type_ (not the _array type_), which can be a scalar type.
If it is, we are ensured that they are stored in different locations
and we can safely simultaneously access `array[2]` in a thread and `array[5]` in another one for example.

This is just a taste of how the C standards are
abstract to a very extensive level, with the goal of giving freedom for implementations to be comforming even if they decide to have very
different internal structure.

## Bit-fields

Now, lets focus on the second part of the memory location definition: _Memory Location: either an object of scalar type,_
__or a maximal sequence of adjacent bit-fields all having nonzero width__.

When working with bit-fields, in terms of storage, it doesn't make much sense to have a separate memory location for each bit-field. What is
the point of having a bit-field that is 1 bit long if it occupies 8 or more bits? For that reason, if we have a sequence of bit-fields __there is a
big chance__ some of them will share the same memory location. It's then __unsafe__ to access a bit-field in a thread and another adjacent
bit-field in another thread simultaneously.

```c
struct {
  int a:4, b:4;
} myStruct;
// It's not safe to access myStruct.a in a thread and myStruct.b in another one at the same time!!!
```

If the programmer wishes to store two bit-fields in different locations, there must be a bit-field with 0 length (or another
object that is not a bit-field) between them:

```c
struct {
  int a:4, :0, b:4;
  int c:4;
  int d;
  int e:4;
} myStruct;
// It's safe to access myStruct.a and myStruct.b simultaneously in two different threads, because
// there's a 0 length bit-field between them.
// It's also safe to access myStruct.c and myStruct.e simultaneously because they are not adjacent
// (there is another non-bit-field type between them).
```

If we have a system that has a byte (8-bits) as the minimum memory location size, adjacent bit-fields would possibly be stored
as described below (__NOTE:__ there's no specification on the layout of bit-fields in memory, the following layout is just an example of
why bit-fields aren't guaranteed to be in the same memory location and the implications of this):

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/6/bitfield.svg" alt="Bit-field 1" />
<figcaption>Adjacent bit-fields</figcaption>
</figure>

But if we add a zero-length bit-field between adjacent bit-fields, we force them to be stored in different memory locations:

<figure class="center medium">
<img src="{{ site.baseurl }}/imgs/posts/6/bitfield2.svg" alt="Bit-field 2" />
<figcaption>Adjacent bit-fields separated by a zero-length bit-field</figcaption>
</figure>

### Why does it matter whether they share a memory location if we are modifying only the value of an individual bit-field?

The C11 standard defines the term _access_ as:

> __3.1 access__
>
> (execution time action) to read or modify the value of an object  
> ...  
> NOTE 2 "Modify" includes the case where the new value being stored is the same as the previous value.

When we are modifying the value of a single bit-field, we are actually writing to the whole memory location. We change the value
in the bits that are being manipulated and keep all other bits with their previous values. But as stated above, we are also modifying
a value if the new value is the same as the previous one.

Imagine the scenario below:

```c
struct {
  char a:4, b:4;
} myStruct;

// Initial value: 1111 0001 (a = 1111 and b = 0001)
myStruct.a = 0xF;
myStruct.b = 0x1;

// Thread 1:
myStruct.a = 0xA; // 1010
// Thread 2:
myStruct.b = 0x8; // 1000
```

Still considering the system we are using has a minimum memory location size of 8-bits and that the bit-fields layout in the beginning
of the program is `1111 0001`, one could wrongly assume that the new value of `myStruct.a` would be `1010` and the new value of `myStruct.b`
would be `1000`, resulting in `1010 1000`. What truly happens is:

If the thread 1 assigns `1010` to `myStruct.a`, it'll read the value of the memory location, assign a new value to it that updates the value
of `myStruct.a` but preserves the older value of `myStruct.b`, and write the new value to the memory location. Supposing we still
had the initial value of `1111 0001` the operation would look like this:

| Thread 1                     |   | Memory Value |
|------------------------------|---|--------------|
|                              |   | 11110001     |
|       Read (value: 11110001) | ← | 11110001     |
| Modify (new value: 10100001) |   | 11110001     |
|       Write                  | → | 10100001     |

Now, if the thread 2 tried to assign `1000` to `myStruct.b` at the same time, we could have the following situation:

| Thread 1                          | Thread 2                           |   | Memory Value |
|-----------------------------------|------------------------------------|---|--------------|
|                                   |                                    |   | 11110001     |
|       Read (value: 11110001)      |                                    | ← | 11110001     |
|                                   |       Read (value: 11110001)       | ← | 11110001     |
| Modify (new value: 10100001)      |                                    |   | 11110001     |
|                                   | Modify (new value: 11111000)       |   | 11110001     |
|       Write                       |                                    | → | 10100001     |
|                                   |       Write                        | → | 11111000     |

Because the thread 2 read the old value of `myStruct.a` before modifying `myStruct.b`, we end up overwriting the changes
that thread 1 did to `myStruct.a` value in the last step.

As said before, this is not necessarily an accurate portrait of every implementation, since the standard doesn't specify
the memory layout of bit-fields, but it's an example of why adjacent bit-fields without zero-length
are not guaranteed to be in different memory locations.

# Conclusion

This was a very conceptual article, a bit hard to write in an instructive way (so I'm sorry for maybe creating some confusion
in some topics), but it's interesting to explore deeper areas of the C standard. We find out fragile points of our C language
understanding and build a better foundation on understanding what it really is. Not only that, but we find practical cases
where a shallow knowledge of the language would lead to errors and learn how to circumvent them. The bit-field race condition is
a big example: Until we have a deeper understanding of the memory abstraction in C we might make wrong assumptions and end up
hunting one of the worse types of bugs out there without even realizing where we introduced it.
