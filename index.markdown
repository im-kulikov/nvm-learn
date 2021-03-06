---
layout: default
---

<div markdown="1" class="toc">
  * toc
  {:toc}
</div>

## Introduction

This tutorial intends to teach you the basics of [Neo Blockchain Virtual Machine](https://github.com/neo-project/neo-vm), usually called NeoVM (or NVM).
NeoVM is a [stack-based machine](https://en.wikipedia.org/wiki/Stack_machine) for processing Smart Contract applications (and also transaction verifications) on [Neo Blockchain](https://neo.org), inspired by many successful stack languages like [Bitcoin Script](https://en.bitcoin.it/wiki/Script), [Microsoft CIL](https://en.wikipedia.org/wiki/Common_Intermediate_Language), [Java Virtual Machine](https://en.wikipedia.org/wiki/Java_virtual_machine) and, finally, the [FORTH language](https://en.wikipedia.org/wiki/Forth_(programming_language)).

FORTH is older of these languages, being proposed in 1970 by [Chuck Moore](https://en.wikipedia.org/wiki/Charles_H._Moore), currently maintained by [Forth Inc](forth.com) and many independent implementations, such as [Gforth](https://www.gnu.org/software/gforth/) and also this nice website/javascript implementation by <a href="https://twitter.com/skilldrick">Nick Morgan</a> called [EasyForth](https://github.com/skilldrick/easyforth).
Stack languages have many [practical applications](https://www.forth.com/resources/forth-apps/), specially due to their simplicity, and easier verification for code correctness (specially on a deterministic environment such as a blockchain).

_(*) If you are new on the Stack Programming world, we strongly recommend reading more on [Stack Data Structure](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)) and following [EasyForth tutorial](https://skilldrick.github.io/easyforth) first._

### NVM Stack Items

NeoVM supports seven different types of stack items.
* Integers: which are in fact Big Integers with positive/negative values limited to 32-bytes (or 256 bits)
* Byte Arrays: general byte arrays
* Booleans: a true/false value
* Arrays: an array can contain a collection of other stack items, including more Arrays (_important: this is a reference type, not value type_)
* Structs: similar to an array, it can contain several stack items inside it
* Maps: a map can contain a byte array mapping from a key to a value, that may be another stack item
* Interop Interfaces: these stack items are only meant to used for interoperating with high-level implementations of NeoVM, such as NeoContract (the Application Engine for Neo Blockchain)

### NVM Opcodes

A NeoVM program is called _script_ (or _NeoVM script_), which is composed by several operations (called _opcodes_). Each opcode has a unique number and a unique name, in order to identify the operation.
**For example:** opcode named `PUSH1`, number 81 (in hex, `81 = 0x51`), generates the number 1; opcode named `ADD`, number 147 (in hex, `147 = 0x93`) adds two numbers.

## Push Some Numbers

All information needs to be on NeoVM stack in order to be processed.
For example, if you want to add two numbers, first you need to put them on the stack.
One difference from other languages is that you need to _first put the operands, and then put the operation_, e.g., operation `1 + 2` is done as `1 2 +` on a stack machine (_put one, put two, then sum the top two elements_).
Where is the result of the operation stored? Again, the result of the operation is also put back on the stack.

Let's try it on practice! Type (don't copy-paste) the following into the
interpreter, typing `Enter` after each line.

    PUSH1
    PUSH2
    PUSH3

{% include editor.html neovm=true %}

What happened? Every time you type a line followed by the `Enter` key, the NVM opcode is executed (state `HALT` means that no errors happened during NeoVM execution).
You should also notice that as you execute each line, the area at the
top fills up with numbers.
That area is our visualization of the stack. It
should look like this:

{% include stack.html stack="1 2 3" %}

Now, into the same interpreter, try the opcode `ADD` followed by the `Enter` key. The top two
elements on the stack, `2` and `3`, have been replaced by `5`.

{% include stack.html stack="1 5" %}

At this point, your editor window should look like this:

<div class="editor-preview editor-text">push1  <span class="output">HALT</span>
push2  <span class="output">HALT</span>
push3  <span class="output">HALT</span>
add  <span class="output">HALT</span>
</div>

Type `ADD` again and press `Enter`, and the top two elements will be replaced by `6`. 
If you try `ADD` one more time NVM would abort execution, because it will try to pop the top two elements off the
stack, even though there's only _one_ element on the stack! This results in a
`FAULT` state on NeoVM:

<div class="editor-preview editor-text">push1  <span class="output">HALT</span>
push2  <span class="output">HALT</span>
push3  <span class="output">HALT</span>
add  <span class="output">HALT</span>
add  <span class="output">HALT</span>
add  <span class="output">FAULT (caused by Stack Underflow)</span>
</div>

You can also write everything in a single line and press `Enter`:

    push10 push3 add

{% include editor.html neovm=true size="small"%}

The main stack now looks like this:

{% include stack.html stack="13" %}

### Polish notation and multiplication

This style, where the operator appears after the operands, is known as
[Reverse-Polish
notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation). 
Let's try to calculate `10 * (5 + 2)`.
You will need also need a multiplication opcode, which is called `MUL` (opcode number `149 = 0x95`).
Try the following:

    push5 push2 add push10 mul

{% include editor.html neovm=true size="small"%}

The execution in NeoVM follows the order of opcodes in the given NVM script.
This way, no parentheses is needed to perform any kind of arithmetic operation.

## Arithmetic operations

You can play with basic arithmetic operations:
* ADD (opcode `0x93`): adds two numbers on stack
* SUB (opcode `0x94`): subtracts two numbers on stack 
* MUL (opcode `0x95`): multiplies two numbers on stack 
* DIV (opcode `0x96`): divides two numbers on stack (integer division)
* MOD (opcode `0x97`): divides two numbers on stack and puts the rest on stack

### Exercises 

**Exercise:** How do you calculate `(10 * (4 - 6) + 15 / 3) mod 7` on NeoVM? What is the expected result?

{% include editor.html neovm=true size="small"%}

**Solution:** result should be `1`.

First step is: `(5 - 6)`. It can be computed using opcodes: `push5 push6 sub`.
Result `-1` should be put on stack, after consuming operands `5` and `6`.

{% include stack.html stack="-1" %}

Second step is: `3 * (5 - 6)`. Since `(5 - 6)` is already on stack, you just need to push number 3 and multiply: `push3 mul`. Stack should contain `-3` now.

{% include stack.html stack="-3" %}

Third step: let's leave `-3` there, because we need to compute `15 / 2` now.
How do we do that? Simple, just `push15 push2 div`. Our stack must have two values now: `-3` and `7` (integer division of 15 and 2).

{% include stack.html stack="-3 7" %}

Fourth step: expression `(3 * (5 - 6) + 15 / 2)` can be computed now. 
We already have the result of `3 * (5 - 6)` and `15 / 2` on stack, so we just `add`.
Number `4` should be on top of the stack now.

{% include stack.html stack="4" %}

Finally, we perform the division by 3 and take the rest: `push3 mod`. Result should be `1`.

{% include stack.html stack="1" %}

Let's review the whole operations (step-by-step) and the result on stack: 
    `push5 push6 sub push3 mul push15 push2 div add push3 mod`

{% include editor.html neovm=true size="small"%}

<div class="editor-preview editor-text">push5 <span class="output">HALT</span>
push6  <span class="output">HALT</span>
sub  <span class="output">HALT</span>
push3  <span class="output">HALT</span>
mul  <span class="output">HALT</span>
push15  <span class="output">HALT</span>
push2  <span class="output">HALT</span>
div  <span class="output">HALT</span>
add  <span class="output">HALT</span>
push3  <span class="output">HALT</span>
mod  <span class="output">HALT</span>
</div>
{% include stack.html stack="1" %}

**Exercise:** How do you calculate `15 / (12 / 4)` using NeoVM opcodes?

{% include editor.html neovm=true size="small"%}

**Solution:** result should be `5`.

Try to solve it and make sure this is well understood before proceeding to next section ;)


## Stack Operations

NeoVM inherits several operations from Forth language, specially those connected to stack operations. Let's take a look at some of them.

### `dup` (opcode `0x76`)

`dup` duplicates the top element of the stack. Example:

    push1 push2 push3 dup

{% include editor.html neovm=true size="small" %}

Resulting stack:

{% include stack.html stack="1 2 3 3" %}

### `drop` (opcode `0x75`)

`drop` removes the top element of the stack. Example:

    push1 push2 push3 drop

Resulting stack:

{% include stack.html stack="1 2" %}

### `swap` (opcode `0x7c`)

`swap` simply swaps the top two elements of the stack. Example:

    push1 push2 push3 push4 swap

Resulting stack:

{% include stack.html stack="1 2 4 3" %}

### `over` (opcode `0x78`)

`over` takes the **second** element from the top of the
stack and duplicates it to the top of the stack. Example:

    push1 push2 push3 over

Resulting stack:

{% include stack.html stack="1 2 3 2" %}

### `rot` (opcode `0x7b`)

`rot` applied a "rotation" the **top three** elements of the stack. Example:

    push1 push2 push3 push4 push5 push6 rot

Resulting stack:

{% include stack.html stack="1 2 3 5 6 4" %}

### Exercises

**Exercise:** try to calculate `15 / (12 / 4)`, starting with `push12` operation.

**Exercise:** practice stack opcodes a little bit.

{% include editor.html neovm=true size="small"%}

In next sections, we will see stack opcodes that receive parameters from stack.

### `pick` (opcode `0x79`)

`pick` reads `n` and copies `n`-th element to top stack. Example:

    push1 push2 push3 push4 push2 pick

Resulting stack:

{% include stack.html stack="1 2 3 4 2" %}

### `roll` (opcode `0x7a`)

`roll` reads `n` and moves `n`-th element to top stack. Example:

    push1 push2 push3 push4 push2 roll

Resulting stack:

{% include stack.html stack="1 3 4 2" %}


### Exercises 

**Exercise:** Consider stack `1 2 3 4 5 6 7 <- top`, how can you put `2` after `6` (and before `7`)? How many stack operations are needed? 

_Result should be `1 3 4 5 6 2 7 <- top`._

{% include editor.html neovm=true size="small"%}

**Solution:** this can be done on two operations.

First, `roll` the `5`-th element to the top stack.

<div class="editor-preview editor-text">push5 <span class="output">HALT</span>
roll  <span class="output">HALT</span>
</div>

{% include stack.html stack="1 3 4 5 6 7 2" %}

Second, `swap` top elements.

{% include stack.html stack="1 3 4 5 6 2 7" %}

**Exercise:** Is there any other way to do it, even if it costs more operations? (_challenge: try to use `tuck` instead of `swap`_)

## Double-Stack model

NeoVM includes a _double-stack_ design, so we need operations also for both the Evaluation Stack (which we've seen already) and the Alternative Stack.

On FORTH language, this stack is called _return stack_, and it is only used for loops and other temporary storages, since it also stores the execution return pointer. This limits the _return stack_ usage, since loops can interfere in stack operations, that's why FORTH also includes a _global storage map_. Although, NeoVM doesn't have a global storage map, it can successfully execute operations only using two stacks and by using arrays as temporary storage (will be explored in later sections).

Next sections will show important double-stack operations.

### `toaltstack` (opcode `0x6b`)

`toaltstack` moves top element (from Evaluation Stack) and moves it to the Alternative Stack. Example:

    push1 push2 push3 toaltstack

Evaluation Stack: 
{% include stack.html stack="1 2" %}

Alternative Stack:
{% include stack.html stack="3" %}

{% include editor.html altstack=true neovm=true size="small"%}

### `fromaltstack` (opcode `0x6c`)

`fromaltstack` moves top element from Alternative Stack and moves it to the Evaluation Stack (the Main Stack). Example:

    push1 push2 toaltstack push3 fromaltstack

Evaluation Stack: 
{% include stack.html stack="1 3 2" %}

Alternative Stack:
{% include stack.html stack="" %}


### `dupfromaltstack` (opcode `0x6a`)

`dupfromaltstack` copies top element from Alternative Stack to the Evaluation Stack. Example:

    push1 push2 toaltstack push3 dupfromaltstack

Evaluation Stack: 
{% include stack.html stack="1 3 2" %}

Alternative Stack:
{% include stack.html stack="2" %}

### Exercises 

**Exercise:** Consider an altstack with value `1` (just run `push1 toaltstack`). What does the instructions `fromaltstack dup toaltstack push5 add` do? Try to re-write them with a small number of opcodes.

{% include editor.html altstack=true neovm=true size="small"%}



## Arrays (unfinished neovm part, do not proceed now)

Forth doesn't exactly support arrays, but it does allow you to allocate a zone of
contiguous memory, a lot like arrays in C. To allocate this memory, use the `allot`
word.

    variable numbers
    3 cells allot
    10 numbers 0 cells + !
    20 numbers 1 cells + !
    30 numbers 2 cells + !
    40 numbers 3 cells + !

{% include editor.html size="small"%}

This example creates a memory location called `numbers`, and reserves three extra
memory cells after this location, giving a total of four memory cells. (`cells`
just multiplies by the cell-width, which is 1 in this implementation.)

`numbers 0 +` gives the address of the first cell in the array. `10 numbers 0 + !`
stores the value `10` in the first cell of the array.

We can easily write words to simplify array access:

    variable numbers
    3 cells allot
    : number  ( offset -- addr )  cells numbers + ;

    10 0 number !
    20 1 number !
    30 2 number !
    40 3 number !

    2 number ?

{% include editor.html size="small"%}

`number` takes an offset into `numbers` and returns the memory address at that
offset. `30 2 number !` stores `30` at offset `2` in `numbers`, and `2 number ?`
prints the value at offset `2` in `numbers`.


## Keyboard Input

Forth has a special word called `key`, which is used for accepting keyboard input.
When the `key` word is executed, execution is paused until a key is pressed. Once
a key is pressed, the key code of that key is pushed onto the stack. Try out the
following:

    key . key . key .

{% include editor.html size="small"%}

When you run this line, you'll notice that at first nothing happens. This is because
the interpreter is waiting for your keyboard input. Try hitting the `A` key, and
you should see the keycode for that key, `65`, appear as output on the current line.
Now hit `B`, then `C`, and you should see the following:

<div class="editor-preview editor-text">key . key . key . <span class="output">65 66 67  ok</span></div>


### Printing keys with `begin until`

Forth has another kind of loop called `begin until`. This works like a `while`
loop in C-based languages. Every time the word `until` is hit, the interpreter
checks to see if the top of the stack is non-zero (true). If it is, it jumps
back to the matching `begin`. If not, execution continues.

Here's an example of using `begin until` to print key codes:

    : print-keycode  begin key dup . 32 = until ;
    print-keycode

{% include editor.html size="small"%}

This will keep printing key codes until you press space. You should see something like this:

<div class="editor-preview editor-text">print-keycode <span class="output">80 82 73 78 84 189 75 69 89 67 79 68 69 32  ok</span></div>

 `key` waits for key input, then `dup` duplicates the keycode from `key`. We
then use `.` to output the top copy of the keycode, and `32 =` to check to see
if the keycode is equal to 32. If it is, we break out of the loop, otherwise we
loop back to `begin`.


## Snake!

Now it's time to put it all together and make a game! Rather than having you type
all the code, I've pre-loaded it into the editor.

Before we look at the code, try playing the game. To start the game, execute the
word `start`. Then use the arrow keys to move the snake. If you lose, you can run
`start` again.

{% include editor.html canvas=true game=true %}

Before we delve too deeply into this code, two disclaimers. First, this is terrible
Forth code. I'm by no means a Forth expert, so there's probably all kinds of things
I'm doing in completely the wrong way. Second, this game uses a few non-standard
techniques in order to interface with JavaScript. I'll go through these now.

## Conditionals, Loops and Function Calls

Conditionals and loops are implemented on NeoVM via the use o jumps.
Basically, a jump changes the current instruction counter and moves it to another 
position in the _NVM script_, affecting which operations will be read after that.
Since this is not related to the stack execution model, it will only be covered in future tutorials.


## Ending Words

Forth is actually much more powerful than what I've taught here (and what I
implemented in my interpreter). A true Forth system allows you to modify how
the compiler works and create new defining words, allowing you to completely
customize your environment and create your own languages within Forth.

A great resource for learning the full power of Forth is the short book
["Starting Forth"](http://www.forth.com/starting-forth/) by Leo Brodie. It's
available for free online and teaches you all the fun stuff I left out. It also
has a good set of exercises for you to test out your knowledge. You'll need to
download a copy of [SwiftForth](http://www.forth.com/swiftforth/dl.html) to run
the code though.
