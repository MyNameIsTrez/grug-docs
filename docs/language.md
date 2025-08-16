---
weight: 100
---
# Language Reference
This document serves as a reference for the grug programming language.
## Types
Every value in grug is assigned a certain *type*, which represents the kind of
data stored within it. grug has 4 intrinsic types, in addition to types specific
to each game:

* `string` - For all textual data. (such as names or message text)
* `bool` - Either `true` or `false`.
* `i32` - 32-bit integer (i.e. can hold whole numbers between -2147483648 to 2147483647)
* `f32` - 32-bit floating point number (i.e. can hold decimal numbers and
  fractions, but has decreasing precision the farther you get from zero)

There are also `resource` and `entity`, which are types of string. `resource`
holds paths to files containing resources (e.g. images, sounds, text content,
etc.). `entity` holds names of types of entities (e.g. `modname:entityname`).

In addition, each game may define a variety of other types, known as IDs. ID
types can have any name, and are used to represent any other type of data a game
may have (e.g. entities, items, blocks, prototypes, etc.). IDs can also have the
special value `null_id`, useful for initializing a [local
variable](#local-variables) before you have an value ready for it.  Internally,
IDs are just 64-bit integers.

!!! note
    Programmers coming from other languages may notice the lack of any
    collection types (such as lists, hashmaps, etc.). This is an intentional
    design decision; instead, games are expected to supply their own custom
    collections in ways that best fit the game.
## Entities
An *entity* is an object in a game. These can represent almost anything, ranging
from the type of a block, to an enemy in a world, to a world generation
algorithm. Every game defines certain *entity types*, and every file containing
grug code is attached to one of those entities. The entity a given grug file is
attached to is determined in the name. For example, `pistol-gun.grug` is a file
containing a script for a gun entity, because it ends with `-gun`;
`zombie-human.grug` is a file containing a script for a human entity, because it
ends with `-human`.

Multiple instances of a single grug script may exist at the same time. For
example, if we have a `zombie-human.grug` script defining a zombie, then a new
instance of that script (containing its own copies of its [global
variables](#global-variables)) for every zombie that the game spawns.

The special identifier `me` can be used to get the ID of the entity the script
is attached to.
## Functions
A *function* is a segment of code that may optionally take in inputs (called
*arguments*), and may optionally give (or *return*) one output (called its
*return value*). When a function is used, this is called *calling* the function.

There are three kinds of functions in grug: *on* functions, *game* functions,
and *helper* functions.
### On functions
*On functions* are called whenever the game wants to notify the script of
something associated with the script's entity. They may take arguments, but they
may not return any value.

For example, say we have a game that has an `mob` type, representing moving
objects (such as players, enemies, items, etc.). The game may define as many on
functions as it wishes for this entity type, allowing the associated grug file
to listen for different kinds of events.  For example, we could have the
following grug code:
```grug
on_spawn() {
    # The game calls this when the mob spawns
}

on_received_item(giver: mob, item: item) {
    # The game calls this when this mob was given an item by another mob.
}
```
On functions always begin with `on` (hence the name).
### Game functions
*Game functions* are functions defined by the game that can be called from a
grug script. For example, perhaps our game defines a `mob_set_health()` game
function that takes a `mob` and an `i32`, and sets the health of that mob. We
could then use it like so:
```grug
on_spawn() {
    # Set the health of the mob to 20 on spawn.
    mob_set_health(me, 20)
}
```
!!! note
    Functions that don't start with `on_` or `helper_` are automatically assumed
    to be game functions.
### Helper functions
*Helper functions* are functions entirely defined by the grug script. They can
only be accessed inside the same script, and must begin with `helper_`. Example:
```grug
on_spawn() {
    # Call our helper function
    helper_reset_health()
}

# Helper function to reset health of an entity to 20
helper_reset_health() {
    mob_set_health(me, 20)
}
```
Helper functions may also take arguments and return values:
```grug
helper_health_difference(first: mob, second: mob) i32 {
    return mob_get_health(second) - mob_get_health(first)
}
```
!!! note
    The type after the closing parenthesis in the previous example is the return
    type of the function. This must be specified if the function returns a
    value.
## Variables
*Variables* can be thought of as containers for values. At any given time, a
variable may only contain a single value. Each variable has a name and a type.
The value in a variable can only ever be of the type of the variable. So for
example, you couldn't assign a `string` to a variable of type
`i32`:
```grug
# Create a variable with name x and type i32, and set it to 0.
x: i32 = 0
# set x to 5. ok since 5 is of type i32
x = 5
# the following line produces an error since "hello" is of type string, not i32
x = "hello"
```
Variables must also be assigned a value when created:
```grug
# ok, value assigned when created
x: i32 = 0
# error, no initial value assigned
x: i32
```
There are two kinds of variable: *global* variables, and *local* variables.
### Global Variables
Global variables may be accessed in every declared function in a script. Global
variables are declared before any function:
```grug
# ok, declared before functions
x: i32 = 0

on_init() {
    ...
}

# error, must be moved to be before functions
y: i32 = 0
```
!!! note
    Global variables are specific *only to the instance*, not to the script as a
    whole. This means that, for example, every instance of a "player" entity in a game
    would have its own global variables, separate from every other instance.

Global variables may be used in any case where you need state that persists
across multiple on function calls. For example, in a hypothetical combat game,
if you wanted to give a player a weapon after they reached a certain number of
kills, it might look something like this:
```grug
kills: i32 = 0

# Called when a player kills another player
on_kill() {
    kills = kills + 1
    if kills >= 5 {
       player_give_item(me, "sword") 
    }
}

# Called when the player dies
on_die() {
    kills = 0
}
```
### Local variables
Local variables may be declared anywhere in a function and can only be accessed
inside of that function:
```grug
on_init() {
    # declare an integer named x
    x: i32 = 0
    # call a function with x
    x = helper_increment(x)
}

# Adds 1 to a value
helper_increment(value: i32) {
    # ok, since x is from `on_init` it doesn't matter if we use the name here,
    # in a different function.
    x: i32 = value
    return x + 1
}

helper_bad() {
    # error, x is not valid here, since it wasn't declared.
    x = 2
}
```
!!! warning
    Global variables of an ID type may not be assigned to local variables of the
    same ID type. This is a design decision to prevent complicated memory
    management.
## Control Flow
*Control flow* refers to manipulating the order or conditions in which certain
actions are performed. Below is a list of control flow constructs in grug.
### If Statements
`if` statements allow you to execute statements only if a certain condition is
true:
```grug
x: i32 = 0
if x == 5 {
    print("This won't execute.")
}
x = 5
if x == 5 {
    print("This will execute.")
}
# Only "This will execute." will be printed.
```
!!! warning
    In a condition, two equal signs (`==`) should be used, and not a single
    equal sign (`=`). `==` is an operator that takes in two values, and returns
    a `bool` telling whether they're equal, while `=` is used to assign a
    variable. Using `=` in an `if` statement will lead to an error.

In addition, an `if` statement may be followed by an `else` branch, which will
execute if the previous condition was not true:
```grug
x: i32 = 0
if x == 5 {
    print("x equals 5")
} else {
    print("x does not equal 5")
}
# "x does not equal 5" will be printed
```
`if` and `else` branches may be chained:
```grug
if x == 0 {
    print("x == 0")
} else if x == 1 {
    print("x == 1")
} else if x == 2 {
    print("x == 2")
} else {
    print("x is something else")
}
```
### While Statements
`while` statements allow you to repeat something while a certain condition
holds:
```grug
i: i32 = 0
while i < 5 {
    print_i32(i)
    i = i + 1
}
# Prints: 0 1 2 3 4
```
!!! warning
    grug automatically terminates code that take too long to execute (the exact
    amount of time before termination is dependent on the game). When making
    logic that can potentially loop for a long time, be ready to handle any
    oddities that may arise from your code being terminated early.
An infinite loop can be achieved by using `while true`:
```grug
while true {
    print("This never ends!")
}
```
In addition, it is also possible to exit a loop early with `break`, which
instantly jumps past the closing brace of the loop:
```grug
# the below is equivalent to the first example
i: i32 = 0
while true {
    if i >= 5 {
        break
    }
    print_i32(i)
    i = i + 1
}
# Prints: 0 1 2 3 4
```
It is also possible to skip an iteration of a loop with `continue`:
```grug
i: i32 = 0
while i < 5 {
    i = i + 1
    if i == 2 {
        continue
    }
    # i is never 2 here
    print_i32(i)
}
# Prints: 1 3 4 5
```
### Return Statements
`return` returns a value from a function:
```grug
# Returns a value plus one
helper_increment(value: i32) i32 {
    return value + 1
}
```
When `return` is used, the value is immediately returned and no further
execution is performed:
```grug
# Returns a value plus one, capped at max.
helper_increment_capped(value: i32, max: i32) i32 {
    if value >= max {
        # return our value immediately, don't increment
        return value
    }
    # return our value plus one
    return value + 1
}
```
`return` may also be used in functions that don't have a return value:
```grug
# Hypothetical helper to kill only killable mobs in a game
helper_kill_mob(mob: mob) {
    if not mob_is_killable(mob) {
        # can't kill, just return
        return
    }
    # can kill, so do it
    mob_kill(mob)
}
```
## Operators
grug uses an [infix](https://en.wikipedia.org/wiki/Infix_notation) notation for
operators (i.e. the one you're most likely familiar with; where `a+b` means to
add `a` and `b`). Some operators are *unary*, meaning they only act on one value
and go before that value (such as `not` or the negation operator `-`).  Other
operators are *binary* meaning they operate on two values (like `+` or `*`).

Below is a list of operators.
### Arithmetic
Arithmetic operators act on number types (`i32` and `f32`). Each one takes
in numbers, and returns a number of the same type.

| Operator | Function |
| - | - |
| `+` | Addition |
| `-` | Subtraction, when used as a binary operator. Negation when used as a unary operator. |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulo, or remainder |

### Comparison
Comparison operators act on two numbers and compare them, returning a `bool`.

| Operator | Function |
| - | - |
| `>=` | Greater than or equal to |
| `>` | Greater than |
| `<=` | Less than or equal to |
| `<` | Less than |

### Equality
Equality operators act on two values of the same type, and check whether they're
equal, returning a `bool`.

| Operator | Function |
| - | - |
| `==` | Equal |
| `!=` | Not equal |

### Boolean
Boolean operators act on the `bool` type.

| Operator | Function |
| - | - |
| `and` | Logical and. Only true when both values are true. |
| `or` | Logical or. Only true if at least 1 of the values are true. |
| `not` | Unary operator; inverts the input (i.e. `true` goes to `false`, and `false` goes to true) |

`and` and `or` are *short-circuiting* operators. This means that they don't
evaluate their second operand if their first operand already is enough
information to determine the result.

For example, if we have `a and b`, then `b` will not be evaluated if `a` is
false (since, `false` and anything is always `false`). Likewise, if we have `a or
b`, and `a` is `true`, then `b` will not be evaluated (since, `true` or anything
is always `true`).
### Operator Precedence
Operators have a *precedence*, which determines the order of operations (you
might know this as PEMDAS or BODMAS). Below is a list of operators in decreasing
precedence (operators with higher precedence are evaluated first, operators of
equal precedence are evaluated left-to-right):

* `*`, `/`, `%`
* `+`, `-`
* `>=`, `>`, `<=`, `<`
* `==`, `!=`
* `and`
* `or`
