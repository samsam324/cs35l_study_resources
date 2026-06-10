# C Programming Cheatsheet (candidate items)

## Hello World
```c
#include <stdio.h>
int main(void) {
    printf("Hello\n");
    return 0;
}
```

## Primitive types
- `char` 8-bit; signedness implementation-defined
- `int` typically 32-bit
- `short` ≥16
- `long` ≥32
- `long long` ≥64
- `float double` (no `decimal`)
- `unsigned int` etc. (no negative range)
- `size_t` unsigned, used for sizes/lengths

## Pointers
- `int *p;` declare
- `*p` dereference (get value)
- `&x` address-of (get pointer)
- `p->member` same as `(*p).member`
- `NULL` null pointer
- `int arr[10]` array; `arr` decays to `&arr[0]`
- Pointer arithmetic moves by `sizeof(type)`

## Memory: malloc/free
```c
int *arr = (int*)malloc(n * sizeof(int));
if (arr == NULL) { /* handle */ }
arr[0] = 5;
free(arr);
arr = NULL;
```
- `malloc(size)` allocates UNINITIALIZED memory; returns `NULL` on failure
- `calloc(n, size)` allocates ZEROED memory
- `realloc(ptr, new_size)` resize
- `free(ptr)` deallocate
- **Memory leak**: forget to `free`
- **Double-free**: `free` twice → undefined behavior
- **Dangling pointer**: use after `free` → UB
- **Segfault**: dereferencing NULL or invalid pointer

## Strings
- C strings = `char` arrays terminated by `'\0'`
- `"hello"` is `char[6]` (5 chars + null)
- `strlen(s)` length (excludes `\0`)
- `strcpy(dst, src)` copy (unsafe — no bounds check)
- `strncpy(dst, src, n)` safer
- `strcmp(a, b)` compare; returns 0 if equal
- `strcat(a, b)` concatenate (unsafe)

## Structs
```c
struct Point { int x, y; };
struct Point p = {1, 2};
p.x = 5;

struct Point *pp = &p;
pp->x = 10;     // arrow for pointer access

typedef struct Point Point;   // now usable as just "Point"
```

## Const
```c
const int MAX = 100;
const char *s = "hi";        // pointer to const char (data unchangeable)
char *const p = buf;          // const pointer (where it points unchangeable)
const char *const p = buf;   // both
```

## Function pointers
```c
int (*fp)(int, int) = &add;
int x = fp(2, 3);
```

## Headers / includes
- `#include <stdio.h>` — system header
- `#include "myfile.h"` — local header
- Header guards:
  ```c
  #ifndef MYHEADER_H
  #define MYHEADER_H
  ...
  #endif
  ```

## File I/O (stdio)
```c
FILE *f = fopen("file.txt", "r");   // "r" "w" "a" "rb" etc.
if (f == NULL) { /* error */ }
fread(buf, 1, n, f);
fwrite(buf, 1, n, f);
fgets(buf, size, f);
fprintf(f, "x=%d\n", x);
fclose(f);
```
- `fopen` returns NULL on failure
- Always `fclose`

## I/O formatting
- `%d` int
- `%u` unsigned
- `%ld %lld` long, long long
- `%f` float/double
- `%c` char
- `%s` string (`char *`)
- `%p` pointer
- `%x` hex
- `%%` literal `%`

## Library vs system calls
- **Library calls** = libc functions (`printf`, `fopen`) — buffered, user-space
- **System calls** = direct kernel calls (`write`, `open`) — slower, no buffering

## Compile pipeline
```
source.c ──[preprocessor]── source.i  (expand #include, #define)
         ──[compiler]──── source.s   (assembly)
         ──[assembler]── source.o   (object file / machine code)
         ──[linker]───── a.out      (executable, links libs)
```
- One-shot: `gcc main.c utils.c -o myprog`
- Separate: `gcc -c main.c -o main.o`, then `gcc main.o utils.o -o myprog`

## Static vs dynamic linking
- **Static** — library code copied INTO the binary (`-static`) — bigger but self-contained
- **Dynamic** — library loaded at run time (`.so`/`.dll`) — smaller binary, needs lib at runtime

## C vs C++ (lecture comparison)
- No function overloading
- No references (only pointers)
- No exceptions (return error codes)
- No classes/templates
- Smaller binaries; predictable execution
- You manage memory yourself

## FFI / Python ctypes
```python
import ctypes
import os
if os.name == "posix":
    lib = ctypes.CDLL("./mylib.so")
elif os.name == "nt":
    lib = ctypes.CDLL("./mylib.dll")
```

## goto
- `goto label;`  jumps to `label:`
- Dijkstra "Go To Considered Harmful" (Comm. ACM 1968)
- Generally avoid; one acceptable use: error cleanup in nested resource acquisition

## libc stack (mental model)
```
Your Program  →  libc (stdio, stdlib)  →  OS kernel  →  Hardware
```

## Common bugs
- Uninitialized variables → garbage values
- Off-by-one in array index
- Buffer overflow (writing past array end)
- Forgetting `free` → memory leak
- `free` then use → dangling pointer
- Returning pointer to local variable (it's gone after return)
- Mismatched format specifiers in `printf` → UB

## sizeof
- `sizeof(int)` typically 4
- `sizeof(arr) / sizeof(arr[0])` array length (only inside the scope where `arr` is declared)

## Additional items (potentially missing)

### Preprocessor directives
- `#include <stdio.h>` system header
- `#include "myfile.h"` local header
- `#define PI 3.14159` macro constant
- `#define SQ(x) ((x)*(x))` function-like macro
- `#ifdef NAME / #endif` conditional compile
- `#ifndef GUARD / #define GUARD / ... / #endif` header guard
- `#if defined(__GNUC__)` check macro
- `#pragma once` (modern alt to header guards)

### Common headers
- `<stdio.h>` — printf, fopen, fread, fclose
- `<stdlib.h>` — malloc, free, atoi, exit, rand
- `<string.h>` — strlen, strcpy, strcmp, strcat, memcpy
- `<math.h>` — sqrt, pow, sin, cos (link with `-lm`)
- `<stdbool.h>` — `bool`, `true`, `false`
- `<stdint.h>` — `int32_t`, `uint8_t`, etc.
- `<ctype.h>` — isdigit, isalpha, toupper
- `<errno.h>` — `errno` global, error codes
- `<unistd.h>` — POSIX (fork, exec, sleep)

### Bitwise operators
- `&` AND
- `|` OR
- `^` XOR
- `~` NOT
- `<<` left shift
- `>>` right shift

### Enums
```c
enum Color { RED, GREEN, BLUE };
enum Color c = RED;
```
Values: RED=0, GREEN=1, BLUE=2 by default.

### Unions (memory-sharing)
```c
union Data { int i; float f; char str[20]; };
union Data d;
d.i = 5;       // only one field at a time
```
All fields share the same memory.

### typedef
```c
typedef unsigned int uint;
typedef struct Node Node;
typedef struct { int x, y; } Point;   // anonymous struct + typedef
```

### Storage classes
- `static` (file scope) — limit visibility to the file
- `static` (function-local) — variable persists across calls
- `extern` — declare without defining (defined elsewhere)
- `const` — immutable
- `volatile` — don't optimize reads (memory-mapped I/O, signals)
- `register` — hint to keep in register (rarely used)

### Function declarations vs definitions
- Declaration: `int foo(int x);` (in .h)
- Definition: `int foo(int x) { return x*2; }` (in .c)

### Variadic functions
```c
#include <stdarg.h>
int sum(int n, ...) {
    va_list args;
    va_start(args, n);
    int total = 0;
    for (int i = 0; i < n; i++) total += va_arg(args, int);
    va_end(args);
    return total;
}
```

### errno
```c
#include <errno.h>
FILE *f = fopen("nope", "r");
if (!f) {
    perror("fopen");        // prints "fopen: No such file or directory"
    // strerror(errno) returns the message string
}
```

### main signature variants
```c
int main(void)
int main(int argc, char *argv[])    // command-line args
int main(int argc, char *argv[], char *envp[])   // also env vars
```
- argc = arg count (including program name)
- argv[0] = program name

### Common bugs
- Off-by-one (loop `< n` vs `<= n`)
- `=` vs `==` in conditions
- `if (a = 0)` always false (assignment, not comparison)
- Buffer overflow (writing past array end)
- Forgetting `\0` in C strings
- Integer overflow (signed wraps to negative)
- Implicit type conversions
- Dereferencing NULL
- Returning pointer to local variable
- `sizeof(arr)` on function parameter (array decays to pointer)

### Printf format specifiers reminder
- `%d %i` signed int
- `%u` unsigned
- `%ld %lld` long, long long
- `%f` float/double (default 6 decimals)
- `%.2f` 2 decimals
- `%e` scientific
- `%c` char
- `%s` string
- `%p` pointer
- `%x %X` hex
- `%o` octal
- `%%` literal %

### Scanf
```c
int x;
scanf("%d", &x);    // note the & for non-string types!
char buf[100];
scanf("%99s", buf); // ALWAYS bound %s to prevent overflow
```

### sizeof gotcha
```c
int arr[10];
sizeof(arr) / sizeof(arr[0])    // = 10 (works at declaration scope)

void f(int arr[]) {
    sizeof(arr) / sizeof(arr[0])  // WRONG — arr decayed to pointer!
}
```
