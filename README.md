> [!NOTE]
> In order to read this article you must know:
> - primary data-types
> - structs and arrays

# The beauty of the C programming language
---

C is a statically typed compiled programming language that allows for blazingly fast
performant applications unlike dynamic transpiled languages like python ,*my beloved* lua and other languages

## sizeof()

though seems like a function, sizeof is an operator in c that returns the size in bytes of the parameter it was passed !
it is commonly paired with the `stdint.h` header file !

```c
#include <stdint.h> // the header file for fixed size integers
uint8_t i = 255;

sizeof(uint8_t) == 1; // true !! 8 bits which is 1 byte
sizeof(int8_t) == 1; // also true
sizeof(i) == i // also true

sizeof(int) == 2 // float int and double
                // types byte-sizes depend on the cpu architecture
               // compiler, compiler version, and os
              // which make them unreliable when it comes to size
```
the `stdint.h` includes fixed-size types like : int8_t, int16_t, int32_t, int64_t for signed integers and uint8_t, uint16_t, uint32_t, uint64_t for unsigned integers

## structs

structs are data-types that allow the presence of abstraction into the code structure !
they allow to group primary or non-primary data-types into new user-defined `struct` types !

```c
struct player {
  const char* name;
  int level;
};

struct player player_one = {
  .name = "nizar yatim", // <<- order does not matter
  .level = 0,
};

struct player player_two;
player_two.name = "player two";
player_two.level = 0;

```

#### typedef

```c
typedef type_a alias;
```

typedef allows us to give an alias to type_a type_a still exists but also lives with the nickname : alias \
to avoid the reccurent use of the `struct` keyword typedef is called with the struct type declaration

```c
struct player {
  const char* name;
  int level;
};

typedef struct player player; // aliases struct player to player

player player_a = { ... }; // works just like a normal struct
player player_b = { ... };

```


well now there must be a reason for why we talked about the sizeof operator right ? why would it be introduced at such an early
stage ! well you'd have found out if you just kept reading for god's sake.


```c
struct vector2 {
  int x;
  int y;
};

struct vector2 pos;

```

# Alignment

Let's take these two structs.

```c
struct example_a {
  int8_t a; // 1 byte int
  int32_t b; // 4 bytes int
};

struct example_b {
  int32_t a; // 4 bytes int
  int32_t b;// 4 bytes int
};

```


fundamentally these two structs look nothing alike, right ? The first one is 5 bytes sized ant the second one is 8 bytes sized.\
Well not really, and to prove that we'll use the sizeof operator to print the size in bytes of each one of these structs.

```c
#include <stdio.h>
#include <stdint.h>

struct example_a {
  int8_t a;
  int32_t b;
};

struct example_b {
  int32_t a;
  int32_t b;
};

int main() {
  printf("sizeof(example_a) : %ld\n", sizeof(example_a)); // output : 8
  printf("sizeof(example_b) : %ld\n", sizeof(example_b)); // output : 8
}
```

well what the hell is happening here ? sizeof shows that both of these structs have the exact same size ?
The cause for that is aligment in example_a the first struct is int8_t followed by an int32_t which in total would be 5 bytes
having those 5 bytes in memory just how the struct depicts it would be extremely inefficient and slow so in order to release the charge for the cpu (*optimization*), in order to make it optimized the compiler fills in the gap and adds 3 more bytes between the int8_t and the int32_t so that the int8_t can align in memory with the int32_t.

Memory Layout :

| Byte 1  | Byte 2  | Byte 3  | Byte 4   | Byte 5 -> 8  |
| ------- | ------- | ------- | ------- | -------      | 
| int8_t  | gap     | gap     | gap      | int32_t     |  

the compiler adds 3 padding bytes to fill in the gap between int8_t and int32_t and make them aligned

why is that important ? You might say.
In the realm of networking where one has to send data over the network it is extremely important to send compact data.
since sending less data over the network means faster transfer and that is only one of many reasons of why it is important.

a good practice amongst c developers is to leave all the padding bytes at the very end of your structs.

```c
// bad
struct {
  int8_t number;
  int32_t big_number;
};

// good
struct {
  int32_t big_number;
  int8_t number;
};

// bad
struct {
  int8_t age;
  const char* name;
  int8_t grade;
  int32_t value;
  int32_t second_value;
};

// good
struct {
  const char* name;
  int32_t value;
  int32_t second_value;
  int8_t age;
  int8_t grade;
};

```

# oop style programming

what is oop ? well i'm not gonna sit here and explain it in complicated words because thats f-ing boring instead let's illustrate it in an example. Let's say you want to validate data you get from a user and if that data is invalid you want to throw an error. Usually a beginner would do it like this :
```c

#include <stdio.h>

int main() {

  int i = 0;

  printf("enter a value : ");
  scanf("%d", &i);
  
  if (i <= 0) {
    printf("[error]: invalid i must be an unsigned integer!\n");
    exit(EXIT_FAILURE); // exit the program
  }
  // other code 
  return 0;
}

```

whilst this is a correct way to do it scaling this program and to have multiple errors over a few hundred lines of code would make it way more messy and hard to read.
a solution to this problem is the oop (object oriented programming) design pattern where one would be able to do it like this:

```c
#include <stdio.h>

#define WARNING 0
#define ERROR   1

struct error {
  const char* message;
  int8_t kind;
};

typedef struct error error;

void throw_error(error e) {
  if (error.kind == WARNING)
    printf("[warning]: ");
  else if (error.kind == ERROR)
    printf("[error]: ");
  else {
    printf("error kind must be either warning or error\n");
    exit(EXIT_FAILURE);
  }
  printf("%s\n", error.message);
  if (error.kind == ERROR)
    exit(EXIT_FAILURE); // exit the program
};


int main() {
  int i = 0;
  printf("enter a value : ");
  scanf("%d", &i);
  
  if (i <= 0)
    throw_error((error) {
      .message = "i must be an unsigned integer",
      .kind = ERROR, // so that it exits the program 
    });
  // other code 
  return 0;
}
```
but this example does not truly state the benefits of using the oop paradigm over anything else so let's give another example that shows why the oop design pattern is so important in the design of large scale software.
let's say we want to make a physics engine where different objects can move and collide like walls , entities players etc.

```c
struct object {
  int x, y;
  int height, width;
};
typedef struct object object;
```

considering that the center of the object is the vector2 `{x, y}` its 4 vertices would be : `left: x - width  / 2.0f, right: x + width / 2.0f, top: y - height / 2.0f, bottom: y + height / 2.0f`
to check weither two objects are colliding we'd just have to compare their vertices positions like so:

```c
bool does_collide(object a, object b) {
  return (
    a.right >= b.right && // where right is x + width / 2.0f and left is x - width / 2.0f
    a.left <= b.right  &&
    a.bottom >= b.top  &&
    a.top <= b.bottom  &&
  );
}
```

doing it the typical way we'd have to create 4 variables for each object we create which would make the codebase extremely polluted and hard to read.

# Pointers

A pointer ! what is a pointer ? Seems like some complicated programming word no one knows about ... Is that even useful ?
Hell yeah, a pointer is a variable that holds some memory address and what type that address holds.

![pointer image example](./pointer_img.png)

However there is a small trick to know with pointers:

```c

#define str_literal "this is a string literal"

int main() {

  const char* string_lit = "hello world";
  // any array of characters which size is known before
  // the execution of the program is called a string literal
  // the type of a string literal is a const char* aka a constant pointer
  // to the first character of the string which is h in this example
}

```

in c arrays are pointers that point to the first element of the list, and string literals are just arrays of characters
which means any variable that holds a string points to the first character of the string itself.

### subscripts

to access a member of a list or an array in c we use the index operator like this:

```c

int main() {
  char* string = "hello world";
  char first = string[0];
  char first = 0[string]; // also valid
}

```

What in the actual f- is happening here ? Well you see the index of operator `value[x]` is a subscript which means it gets translated into something else. The underlying translation of the index of operator is the following:

```c
string[value] -> *(string + value)
```
Okay so now we know the why of `0[string]` but how does it actually work ? \
So i did state that `a pointer is a variable that holds some memory address` and memory addresses are just numbers
so can't we multiply add or subtract those numbers ? Yes we can 😎 and that is why c is a godly sent programming language it allows us to do witchcraft with pointer arithmetic.
So for some context we know that arrays and structs have contiguous memory aka their memory is adjacent which means that in the memory itself the first element of an array is followed by its second which is itself followed by its third etc...

| addresses  | arrays in memory |
|--          | -------------    |
| 0x00       | first element    |
| 0x01       | second element   |
| 0x02       | third element    |
| 0x03       | first element    |

So if we have the address of the first element of an array we can get to its n'th element by the following operation:

```c
array <-> &(array[0])      // these two are the same
array[n] <-> *(array + n) // so are these two,since an array points to its first element
```

## Heap and stack

These two principles are **VERY** important so please🙏 bear the burden to stay focused through this entire part !

The heap and the stack are two data structures used in our computers and phones to handle the distribution of memory to different processes and applications.

### The Stack:

A stack is a contiguous memory zone, but keep in mind that contiguous means fast, blazingly fast so the stack is a blazingly fast memory ! But nothing is free, and the stack is no exception to the rule that extreme speed comes with inconveniences which is that the stack is extremely small (*1kb -> 4 kb per stack depending on the platform*)
Whenever a function is called it is assigned a stack :
```c
int add(int a, int b) {
  return a + b;
}
```

However when a function returns or exits its stack dies, all variables created within a function live within its stack which means that everything in a function dies after the function returns.

```c

void my_function() {
  // the stack is created and assigned to the function
  int32_t variable = 5; // push 32 bits on the stack
  int64_t array[5] = {1, 2, 3, 4, 5}; // push 5 * 64 bits onto the stack
  // the stack dies here and all the memory allocated goes back to the operating system
}
```

If we were to recreate a stack in c we would do it like this:

```c
#define STACK_SIZE (2 * 1024) // 2 kilobytes <-> 2 * 1024 bytes
struct Stack {
  int64_t cursor;
  int8_t data[STACK_SIZE];
};
```


### The Heap:

The heap is an other kind of memory, but in order to illustrate the how and why anyone would need another kind of memory we'll try and solve this problem.

Let's say we want to make a sign-in system where people create accounts, whenever an account is created we add it to the list of registered accounts:
```c

struct user {
  uint32_t id;
  // other fields;
};

typedef struct user user;
#define BASE_SIZE 1024

user users[BASE_SIZE];
uint16_t current = 0;

void register_user(user u) {
  if (current < BASE_SIZE)
    users[current++] = u;
}
```

But here lies a problem our system can only handle 1024 created accounts which is very **VERY** small. So how do we handle this problem ? Grow the base size to an even bigger size like 10.000 ? But what if the amount of all-time registered users exceeds the base size ? No , that won't do ! And that is why we have the heap.\
Heap allows us to dynamically allocate memory that won't die even if the function returns unlike the stack.
In order to get memory from the heap we send a request directly to the operating system using the function `malloc()` from the `stdlib.h` header file.

If we want to recreate our current setup using the heap we would do it like this:

```c
#include <stdio.h>
#include <stdlib.h>

struct user {
  uint32_t id;
  // other fields;
};

#define BASE_SIZE 1024
user* create_list(uint32_t base_size) {
  user* list = malloc(sizeof(user) * base_size); // malloc takes as a parameter (an integer) the amount of bytes to allocate
  //we want a list of users
  // its total size in bytes is going to be the size of a single user multiplied by the length of the array
  return list;
}

void register_user(user u) {
  if (current < BASE_SIZE)
    users[current++] = u;
}
```
`malloc(x)` returns a pointer to x amount of bytes on the heap

---
> [!NOTE]
> However there are two inconveniences to the heap *(or conveniences depending on the point of view)*. The first one which
> is that any memory gotten from the heap may or may not be contiguous to any other memory gotten from the heap.
> Which means that:

```c
#include <stdio.h>
#include <stdlib.h>
void my_function() {
  int* my_int = malloc(sizeof(int)); // these two are not contiguous !
  int* my_int_2 = malloc(sizeof(int));
  //
  int* my_list = malloc(sizeof(int) * 10); // first list
  int* my_list_2 = malloc(sizeof(int) * 10); // second list
  my_list[2];
  // same as:
  *(my_list + 2)
  /*
   */
  *(my_list + 10)
  //same as:
  my_list[10];
}
```

> [!WARNING]
> since my_list and my_list_2 are not contiguous we **CANNOT** use my_list to get to my_list_2 that is not possible and will cause undefined behavior !
