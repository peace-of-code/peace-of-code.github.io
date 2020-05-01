
Over the years I have written code in most of the possible and most bizarre styles you can think of.

I never argues with a team about how the code must look as long as team's code style is self consistent and readable.

For myself I've settled on what I've found most convent over the years. 

My personal motto in code styling was: *"To achieve peace of mind and for the love of code you must achieve peace with the code first..."* 
(I must confess - I've never understood `warrios` in various `codewarrior` tools)

I will use C/C99 as a language to explain the choices made but most of it is well applicable to Java, JavaScript, Rust and almost everything else.

Tabs vs SPACES
---
Enough said: forget TAB existed.

Naming. 
---
NeverMixCamelCase with readable Linux/posix like lower_case_underscore_naming.


comas, spaces and punctuation
---
spaces after comas (like in written English prose) and never inside ()
~~~
function_calls(parameter1, parameter2); // spaces always after "," and nowhere else
~~~
spaces after control statements (special case)
~~~
if (condition) { ... } // spaces after "if", "do", "while", "for", "switch" and never after "(" before ")"
~~~
spaces and "*" pointers two cases:
~~~
// special consideration for types and output parameters:
// "const char*" is treated as pointer type thus "*" follows char without space
void foo(const char* *out) {  
    ... // *out tells that parameter is an address (reference)
}
~~~

Vertical empty spaces
---
Function definitions separated by empty line. Groups of variables may be separated by empty line.
In all other cases do not use vertical separation of fill it with meaningful comment or split code
in small functions that do not need to be read as a War and Peace paragraphs.
~~~
#include <stdio.h>

type_t* global;

int main(int argc, const char* argv[]) {
    return 0;
}
~~~

Indent
---
Indent is **4**. indent 2 and 3 was good for 80x24 terminals. It's 21st century it is 4!
(8 was good for Linus leads to unbearably vertical code)

Header Files
---
~~~
#pragma once // it is more readable then #ifdef guard and well supported in 21st century 
~~~

Constants
---
~~~
enum { // always use anonymous hardened enums for constants
    MODULE_NAME_ONE          =  1, // alignment of constant values and names 
    MODULE_NAME_TWO          =  2, // us useful when define hardware tables
    MODULE_NAME_THIRTY_THREE = 33  // of registers and alike
};

#define ONE 1 // is bad
~~~
`short` enums option of compiler must be checked and turned **OFF**

Local variables
---
Always initialize local variables and keep the scope of variable visibility as small as possible.
~~~
int i;
...        // i has random value before it is initialized
i = foo(); // this is bad keep initialization with declaration

int i = foo(); // always initialize local variables
int j = 0; // compiler will remove unnecessary initialization
struct { int x; int y; } point = {}; // {} is simpler then {0} means the same

int i; // bad
if (something) {
    i = bar();
    the_only_code_that_uses(i);
}

if (something) {
    int i = bar(); // good
    the_only_code_that_uses(i);
}
~~~
special case of leaving large structures uninitialized for performance reason:
~~~
void performance_critical_code() {
    struct large_structurte_s s; // intentionally left uninitialized for performance reasons 
    function_that_guarantees_parameter_will_be_filled_with_good_data_completely(&s);
}
~~~

Variable length array or alloca()
---
Only use when it is important for performance and the size is known to be small bounded and will fit on the stack in all situations.
~~~
int matrixNxN_operation(float* a, float* b, int n) {
    int r = 0;
    assert(0 < n && n <= 4);
    if (0 < n && n <= 4) {
        float c[n][n]; // it is OK we know n is tiny and bounded
        ... // do the operation
    } else {
        r = EINVAL;
    }
    return r;
}
~~~

Post increments/decrements
---
Never/ever use pre-increments/decrements `++i/--i`. Never use increments/decrements in expressions.

Boolean
---
Use `<stdbool.h>`: `bool` `true` `false` it improves readability. Never rely on particular sizeof(bool).
Use bitsets instead.
~~~
bool b = foo();
if (b == true) { /* is wrong and terrible! */ }
if (b) { /* this is boolean expressoin check */ }
b &= bar(); /* is bitwise and wrong */
b = b && bar(); /* instead */
~~~

Integer types
---
Type `int` is used in C to index arrays. Safe assumption on most recent platform that it's int32_t or larger.
Type `size_t` and `isize_t` was one of the biggest posix mistakes and should be avoided as much as possible.
Int all composite types (arrays and structs) and some function parameter sized `<stdint.h>` must be used.

Packed structures
---
Structures interfacing with hardware or network must be packed. Other structures alignment should be left to compiler for optimization reasons.

Variable declaration
---
Only declare single variable for line
~~~
int i, j, k; // is bad
int i = 0; // one variable per line, same for struct fields
int j = 0;
int k = 0;
~~~

Conditions
---
Don't use "yoda-conditions". Never use assignment inside condition or any expression.
~~~
if (1 == x) { ... } // bad - turn on compiler warning assigment inside expression instead and make it error
if (x == 1) { ... } //use this order
if ((i >= 0) && (n > i)) { } // too many parentheses and unreadable order
if (0 <= i && i < n) { } // more natural range check similar to math notation "i in [0..n["
~~~

Curly brackets
---
Always use curly brackets in control statements:
~~~
if (something) foo(); else bar(); // bad
if (something) {
    foo();
} else {
    bar();
}
~~~

Single line control statements
---
Avoid single line control statements except absolutely trivial:
~~~
if (something) { foo(); } else { bar(); } // bad
if (something) { foo(); } // only use when readability is preserved
if (something) { // debugger friendly: simplifies putting breakpoint on foo()
    foo();       // or inserting printf() around foo() call
} 
~~~


switch statements
---
`switch` statements indent and `break`
Only use multiple case labels on a single statement when it is really applicable.
If you need complicated handling of distinct situations that partially overlap
write small inline function for the common case and use it.
~~~
switch (i) {
    case 1: ...; break;   // case always indented 
    case 2: ...; break;   // break is a must
    case 3: foo();        // without break - this is **bad**
    case 4: bar(); break; // this break terminates both "3" and "4" cases
    default: assert(false); break; // default is a must too
}
~~~
better way:
~~~
switch (i) {
    ...
    case 3: foo(); bar(); break;
    case 4: bar(); break;
    ...
}
~~~

Use but do not abuse assert
---
Check conditions that are an absolute must.
~~~
float sqrt(float x) {
    assert(x >= 0);
    ...
}
~~~
better and more robust way would be:
~~~
int sqrt(float* result, float x) { // returns 0 on success and posix error code on failure
    int r = 0;
    assert(x >= 0);
    if (x >= 0) {
        *result = ...;
    } else {
        r = EINVAL;
    }
    return r;
}
~~~
but harder to use. Though in robust code most of the functions must 
return error/success code and it must be checked by caller and acted 
accordingly. In rare situations where the performance is at stake
creative usage if setjmp() and longjmp() or similar exception like API/ABI
may be warranted (study libjpeg source code for a decent architecture 
using this technique).

Early return
---
Avoid early `return` like a plague. Just word of advise. You will be thankful.
While is super simple scenarios like below usage of `return` result/error statements 
may be warranted (the condition is binary, carved in stone and not going to change).
~~~
int sqrt(float* result, float x) { // returns 0 on success and posix error code on failure
    assert(x >= 0);
    if (x >= 0) {
        *result = ...;
        return 0;
    } else {
        return EINVAL;
    }
}
~~~
in more complex scenarios when condition may change in the future and control flow too
it is better not to use early `return` statements because they break control, harder to
spot when code becomes more complex and generally make code much less maintainable/debuggable:
~~~
int sqrt(float* result, float x) { // returns 0 on success and posix error code on failure
    int r = 0;
    assert(x >= 0);
    if (x >= 0) {
        *result = ...;
    } else {
        r = EINVAL;
    }
    return r; // good place to put breakpoint in debugger or insert `printf` before
}
~~~

64-bit portability.
---
Always think about it. And also do not rely on signed pointers being in any specific order.
~~~
    void* p = ...;
    uintptr_t pointer_as_integer = (uintptr_t)p;
~~~

`malloc()`/`free()`
---
Heap has been invented for a reasons. 
Some of them are to keep our life more interesting (and a bit harder).
To simplify leaking something and justifying selling new computer to a customer.
Creating more jobs among software engineers. Selling named engineers plethora of leak hunting tools.
Justifying performance expensive but convenient technologies of garbage collection or automatic reference counting.
And last but not the least, managing unpredictable and unknown at compiler time unbounded amount of data up to the size of available memory.
On multitasking system the last reason above creates it's own cooperation problems.
Never the less, it is important to understand that both `malloc()` and `free()` in addition to poor performance
are also __**synchronization**__ points and do affect the performance of multi-threaded system in a hard and subtle ways:
~~~
void performance_critical_function_that_can_be_called_from_multiple_threads() {
    struct small_s* p = (struct small_s*)malloc(sizeof(struct small_s)); // bad technique
    only_used_locally(p); // why did we have to allocate it on the synchronized heap?
    free(p);
}
~~~


typedef versus struct
---
The issue only exists because of Linus personal opinion, 
absence of anonymous structs in early versions of C language, 
disagreement between C and C++ on anonymous stucts,
and poor job early versions of emacs and vi did on type details visualization.
Despite of all the examples above
~~~
typedef struct { float x; float y; } pointf_t; // is quite clean and useful type definition
~~~
Don't forget struct names and typedef names exist in two distinct namespaces.

callbacks
---
Callback is almost never function alone. It needs to carry data with it.
God save us from a wonderful notions of functors and closures. In C terms it is simple:
~~~


typedef struct something_callback_t;

typedef struct {
    void* that; // because "this" is taken by C++ (and "event" by Microsoft)
    void (*callback)(something_callback_t* cb, ....);
} something_callback_t;

typedef .struct {
    something_callback_t callback;    
} something_t; // callee specific


static void something_callback(something_callback_t* cb, ....) {
    something_t* something = (something_t*)cb->that;
    something_do_something_on_callback(something);
}

static foo(something_callback_t* cb) {
    cb->callback(cb, ...);
}

...
    something_t something;
    something.this = &something;
    something.callback = something_callback;
    foo(&something.cb);
...
~~~
yes, comparing to JavaScript closures on Objective C/Swift blocks it looks verbose and clumsy.
This is why aforementioned languages exist. It would be nice (and wishful thinking) if C21 ANSI 
committee would adopt blocks.

Single File Library
---
Is a great concept for simplified distribution of straight forward pieces of C code.
(If you are not familiar with it - "google" it - it worth every minute of your time)
It is absolute must if you can and you going to distribute your creation.
For the internal codebases do not abuse it - it makes code harder to read and use.


To see this set of rules in action: <a href="https://tinyurl.com/hashtable-c" target="_blank">https://tinyurl.com/hashtable-c<a>
