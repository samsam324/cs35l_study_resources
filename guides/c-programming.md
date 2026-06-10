Want hands-on practice? Work through the C for C++ Programmers Tutorial — eleven interactive chapters with a real C compiler running in your browser. This page is the conceptual companion: read it to build the mental model, then go to the tutorial to lock it in through practice.

Welcome to C. If you’ve made it through C++ in CS31 / CS32, you already know more than half of C — because C++ is, historically, a layer built on top of C. The original C++ compiler (Cfront, 1983) literally translated C++ source into C source, then handed it to a C compiler.

So learning C from a C++ background is not about adding new things. It’s about subtracting — peeling away the C++ conveniences (classes, references, exceptions, templates, function overloading) to see what’s underneath. C is small. The 1989 ANSI C specification fits in roughly the same number of pages as a single STL header. That smallness is the whole point.

One way to frame it: in C, you are the CEO and the janitor. You have total control over memory layout, function calls, and the data your program touches — and you also have to clean every byte up yourself. There is no garbage collector, no destructor, no compiler-generated copy assignment, no std::unique_ptr to save you. The freedom and the responsibility are the same thing.

Why Learn C?
Three reasons account for almost every modern C program that ships:

Speed. C compiles directly to machine code with very little “magic” in between. The mapping from a C statement to its CPU instructions is close enough that an experienced reader can predict the assembly output by eye. Linus Torvalds famously argues that this is the reason the Linux kernel is in C: he wants kernel developers to feel the assembly they are writing. Languages that hide too many costs (hidden allocations, hidden virtual calls, hidden bounds checks) make it hard to write code that is fast and predictable.

Direct memory control. Every byte your program touches, you allocated. Every byte you allocated, you can choose when to release. Higher-level languages (Python, JavaScript, Java) decide allocation and freeing on your behalf — convenient, but you cannot squeeze the last 10% of memory out of them. On a 32 KB embedded microcontroller, that 10% is the difference between “ships” and “doesn’t ship.”

Direct hardware access. Device drivers, firmware, and operating-system kernels need to talk to specific memory addresses, specific I/O ports, and specific interrupt vectors. C lets you cast an integer to a pointer and dereference it — which is dangerous and exactly what writing a device driver requires. Rust now offers a safer alternative for new projects, but the existing hardware-interfacing code in the world is overwhelmingly C.

Where C Is Used Every Day
Most of the software you actually run is built on a C foundation, even when you’re typing Python or JavaScript at the surface:

Operating-system kernels. Linux, the Windows NT kernel, macOS’s XNU kernel, BSD, and almost every embedded RTOS — all C. Higher-level OS components (window managers, system frameworks) are often C++, but the core kernel stays in C for speed, predictability, and direct hardware access.
Embedded and IoT devices. Microcontrollers, sensors, wearables, automotive ECUs. Tight memory budgets and hard real-time deadlines push these toward C.
Compilers and assemblers. GCC, Clang’s LLVM backend, and most production assemblers are written in C or C++ — they need to be fast because they will be invoked millions of times across the world’s build farms.
Database management systems. MySQL, PostgreSQL, SQLite, Redis — the core query engines are C. A single SQL query can touch millions of rows, so a 10% slowdown in the inner loop is a real problem.
Library interfaces for everyone else. Python’s NumPy, scientific code reachable from R or MATLAB, TensorFlow’s compute kernels — they expose a C-compatible interface so that any language can call them. C is the lingua franca of inter-language calls.
That last point is worth holding on to: almost every mainstream language can call into C, which means a C library reaches the widest possible audience. We come back to this in When to Choose C Over C++.

What’s Different from C++
C Is Procedural — No Classes, No Objects
In C++, a class bundles data and the functions that operate on it. In C, data and code live in entirely separate places. You write structs to describe data layouts, and free functions to manipulate them. The struct does not know which functions exist; the functions do not belong to the struct.

struct list_element {
    int value;
    struct list_element* next;   // self-referential pointer — linked list
};
That’s the whole “object.” There are no methods, no private, no inheritance, no polymorphism. To “add a method,” you write a free function that takes a pointer to the struct as its first argument:

void list_print(struct list_element* node) {
    while (node != NULL) {
        printf("%d ", node->value);
        node = node->next;
    }
}
This is exactly how C++ implements member functions under the hood — the implicit this pointer is the first argument. C just makes the convention explicit.

Struct field-layout matters in C. The compiler addresses each field by adding the previous fields’ sizes to the struct’s base address. Variable-length data (like a flexible array member) must appear last, because the compiler needs to know exact offsets for every field that comes before it. This is why you’ll see structs in network protocols ordered with fixed-size headers first and the variable-length payload at the end.

No Function Overloading
C++ lets you write two functions named print with different parameter types and dispatches by argument types at compile time (name mangling). C does not.

// C++
void print(int value)   { /* ... */ }
void print(float value) { /* ... */ }

int main() {
    int a = 5;
    float b = 5.0f;
    print(a);   // calls the int version
    print(b);   // calls the float version
}
// C — every function needs a unique name
void printInt(int value)     { /* ... */ }
void printFloat(float value) { /* ... */ }

int main(void) {
    int a = 5;
    float b = 5.0f;
    printInt(a);
    printFloat(b);
    return 0;
}
That’s why the C standard library has families like abs / fabs / labs, or printf with format specifiers (%d, %f, %s) instead of overloads. The cost C avoids is name mangling — the C++ compiler munges every function name with type information so the linker can tell overloads apart, which makes C++ symbols harder to call from other languages.

No Pass-by-Reference — Only Pointers
C++ has two ways to let a function mutate a caller’s variable: references (int&) and pointers (int*). C has only pointers. The caller is responsible for taking the address explicitly with &.

// C++ — pass-by-reference; call site looks like swap(x, y)
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 30, y = 40;
    swap(x, y);
}
// C — caller must pass &x, &y explicitly
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 30, y = 40;
    swap(&x, &y);   // & at the call site is not optional
    return 0;
}
A consequence: in C, every signature tells you whether a function may mutate its argument — if you see a pointer, mutation is possible; if you see a value type, it can’t be. C++ references hide this at the call site, which is more convenient but less explicit. C trades convenience for clarity here.

No try / catch — Error Codes and Output Pointers
C has no built-in exception handling. The convention is to return an error code as the function’s value and use an output pointer for the actual result:

// C++ — throw on error, return the result directly
int safe_divide(int num, int den) {
    if (den == 0) {
        throw std::runtime_error("divide by zero");
    }
    return num / den;
}

int main() {
    try {
        int z = safe_divide(10, 0);
        std::cout << "Result: " << z << "\n";
    } catch (const std::runtime_error& e) {
        std::cerr << "Error: " << e.what() << "\n";
    }
}
// C — return an error code, write the result through a pointer
int safe_divide(int num, int den, int* result) {
    if (den == 0) {
        return -1;          // non-zero means error
    }
    *result = num / den;
    return 0;               // zero means success
}

int main(void) {
    int z;
    if (safe_divide(10, 0, &z) != 0) {
        fprintf(stderr, "Error: division by zero\n");
        return 1;
    }
    printf("Result: %d\n", z);
    return 0;
}
The convention “return zero on success, non-zero on error” matches how shell programs report exit status, and it scales to many error categories by reserving different non-zero values for different failures.

The output-pointer convention is the part that surprises C++ programmers most. When you see a pointer parameter you have to ask which direction it flows — input (the function reads it) or output (the function writes to it). Document this clearly for every function you write; otherwise readers will pass uninitialized memory to your “output” pointer or, worse, pass NULL and crash inside your function. A common documentation idiom is a comment right above the parameter list:

// Returns 0 on success, -1 on division by zero.
// Writes the quotient to *result on success; *result is unchanged on error.
int safe_divide(int num, int den, int* result);
Cognitive load is real here. Because C has no implicit error path, every call site has to remember to check the return value. Forgetting to check is one of the most common bugs in C code. We come back to this in the Memory in C section, where malloc’s NULL return is the canonical example.

Memory in C: malloc, free, and the Two Failure Modes
Dynamic memory in C comes from two standard-library functions:

void* malloc(size_t size);   // request `size` bytes from the heap
void  free(void* ptr);       // return previously-malloc'd memory
malloc returns a void* — a generic pointer with no type — which you cast (in C, implicitly; in C++, explicitly) to the type you want. sizeof is a compile-time operator that gives you the byte size of any type:

// Allocate a flat row-major matrix of ints, rows × cols
int* matrix = malloc(rows * cols * sizeof(int));
if (matrix == NULL) {
    fprintf(stderr, "out of memory\n");
    return 1;
}

// ... use matrix[i * cols + j] ...

free(matrix);
matrix = NULL;   // optional, but defensive — prevents accidental reuse
Two failure modes dominate C memory bugs, and they pull in opposite directions:

C Programming table
Failure mode	What it is	What you observe	Cause
Memory leak	You malloc‘d and never free‘d	Long-running programs grow without bound; the OS eventually kills them	Forgot to free, or freed on the happy path but not on every error path
Segmentation fault	You accessed memory you don’t own	Program crashes immediately with “segfault”	Used a pointer after free, dereferenced NULL, or walked off the end of a buffer
The discipline is: allocate as late as you can, free as early as you can, and never touch the memory after free. Setting the pointer to NULL immediately after free is a cheap defensive habit — a subsequent accidental dereference fails loudly with a segfault instead of silently corrupting whatever was in that memory next.

Why not just let the OS clean up at program exit? That works for short-lived command-line programs, but a long-running server or daemon that leaks even a few bytes per request will exhaust memory after enough requests. Leaks also confuse memory profilers and obscure other bugs. Discipline pays.

C++ programmers using RAII (constructors / destructors, std::unique_ptr, std::vector) don’t have to think about this — the compiler emits free calls at scope exit. C gives you no such help. Every malloc is a contract that you will eventually call free. The tutorial walks through this discipline with an interactive memory inspector — see Power #3 — malloc/free.

Strings Are Just Char Arrays
C has no string type. A “string” is a char array whose last byte is the null terminator '\0':

char  letter = 'a';      // single character — single quotes, ASCII value 97
char* word   = "hello";  // string literal — double quotes, points to 'h','e','l','l','o','\0'
The character '\0' is the byte with ASCII value zero, not the digit '0' (which has ASCII value 48). Every C string ends with '\0'. The standard-library functions strlen, strcpy, strcmp, etc. all walk the array until they hit the null terminator — which means forgetting the terminator turns those functions into out-of-bounds reads that can crash or leak data. Use #include <string.h> to get the string functions.

#include <string.h>

char  name[6] = {'A', 'l', 'i', 'c', 'e', '\0'};   // null-terminated, OK for strlen
char  bad[5]  = {'A', 'l', 'i', 'c', 'e'};         // no terminator! strlen(bad) walks past the array
size_t n = strlen(name);                            // 5 — strlen doesn't count the terminator
const Tells the Compiler “Read Only”
C lets you mark a variable or a pointer’s target as const, which causes the compiler to reject any code that tries to write through that pointer:

char buffer[]    = "Initial string";   // modifiable array on the stack
const char* ro   = buffer;             // ro is a read-only view of buffer
ro[0] = 'X';                           // compile error — ro is const
Use const deliberately. When a function takes const char* s, the signature is a promise: “I will not modify the string you pass me.” Callers can pass string literals safely (writing to a string literal is undefined behavior); maintainers know they don’t need to audit your function for surprise mutations.

You can cast away const — (char*)ro produces a writable pointer to the same memory — but the language documentation correctly tells you not to. Casting away const and writing through the result is undefined behavior if the original object was actually declared const; if it merely had a const view, you’ve defeated a documentation aid that future readers were relying on.

File I/O: fopen, fread, fclose
Reading a binary file in C is three library calls, plus error checking and explicit cleanup:

#include <stdio.h>

int main(void) {
    int buffer[5];

    FILE* file = fopen("input.bin", "rb");   // "rb" = read, binary
    if (file == NULL) {
        perror("Error opening file");        // prints the error and the filename
        return 1;
    }

    // Read up to 5 ints (one count of `sizeof(int)` bytes per int).
    size_t read = fread(buffer, sizeof(int), 5, file);

    for (size_t i = 0; i < read; i++) {
        printf("Element %zu: %d\n", i + 1, buffer[i]);
    }

    fclose(file);
    return 0;
}
The mode string controls permissions: "r" for read, "w" for write (truncates the file), "a" for append, with b added for binary or + added for read-and-write. Pick the narrowest mode that fits your need — the OS uses the mode to enforce sharing rules (many readers, one writer).

The two things to remember:

fopen returns NULL on failure. Check it before every read or write. Forgetting this check is the #1 cause of “my C program crashed and I have no idea why” — the next fread dereferences NULL and segfaults.
Every fopen needs a matching fclose on every path out of the function, including error paths. If you return early without fclose, you’ve leaked a file descriptor. In C++ this is what RAII gives you for free; in C, you write it by hand, often using a goto cleanup; pattern (see goto, Reconsidered below).
Library calls versus system calls. fopen, fread, fclose, malloc, and free are all library calls — they live in libc (the C standard library) and provide a portable API. Inside libc, those calls eventually invoke system calls (open, read, close, mmap, etc.) that talk directly to the kernel. The system-call ABI differs between Linux, macOS, and Windows; libc papers over that so a C program calling fopen works on all three. We pick this up in the next section.

The Compilation Pipeline: Compiler + Linker
When you turn a C source file into an executable, two distinct tools run in sequence:

The compiler / assembler turns each .c file into an .o object file — assembly translated to machine code, but with unresolved references to functions and variables defined elsewhere.
The linker stitches the object files together (plus any libraries) into a single executable, replacing every “I’ll call printf later” placeholder with a real address.
my_program.c         my_other.c
     │                    │
     ▼                    ▼
 (compiler)           (compiler)
     │                    │
     ▼                    ▼
 my_program.o         my_other.o
        │                │
        └──────┬─────────┘
               ▼
         (linker)  ←── libc (printf, malloc, fopen, …)
               │
               ▼
           my_program     (the executable)
Each .c file is compiled independently. The compiler doesn’t know that printf exists — it just sees a declaration in <stdio.h> (a “header file”) and emits an instruction that says “call the function named printf at some address the linker will fill in.” The linker’s job is to resolve every such unresolved symbol against either another .o file in the project or a library on disk.

Static vs. Dynamic Linking
There are two ways the linker can wire your program to a library:

C Programming table
Question	Static linking	Dynamic linking
When	At link time (build)	At program-start time (or first call)
What ships	One self-contained executable	Executable + separate .so / .dll files
Pros	Runs anywhere with no external dependencies	Smaller executables; one library update fixes many programs
Cons	Larger executables; library bug fix requires re-linking every program	Missing library = program won’t start (“DLL hell”); slight runtime overhead
The IKEA analogy is useful: a statically-linked program is fully assembled furniture — you can put it anywhere and use it immediately. A dynamically-linked program is a flat-pack box — smaller to ship, but the recipient has to assemble it against whatever libraries are present on their system, and if a screw is missing the whole thing doesn’t work.

libc as a Portability Layer
Every modern OS ships its own implementation of the C standard library. When you compile a C program for Linux, the linker uses glibc; for macOS, Apple’s libSystem; for Windows under MinGW, MSVCRT; and so on:

    Your C program       (portable C source — same on every platform)
          │
          ▼
        libc             (one implementation per OS — same API)
          │
          ▼
    Operating system     (Linux, macOS, Windows — different syscalls)
          │
          ▼
       Hardware
The fopen you call in your source has the same signature everywhere. The libc on each platform translates that into the OS’s native file-open syscall, which has a different number and a different ABI on each platform. That translation is the reason “write once, recompile-per-target, run on three operating systems” is realistic for C.

When to Choose C Over C++
C++ is a strict superset of most of C, so it’s tempting to ask “why not always use C++?” Three reasons to deliberately drop to C:

Smaller, More Predictable Binaries
C executables are smaller because C doesn’t pull in the C++ runtime support: no virtual function tables, no exception unwinding tables, no implicit constructor/destructor code, no name-mangled symbols. For an embedded firmware image that has to fit in 64 KB of flash, this matters. (Our own in-browser C tutorial uses the Tiny C Compiler — TCC — instead of GCC for exactly this reason; the full GCC binary is too large to ship inside a virtual machine running in your browser tab.)

C also makes execution-time behavior more predictable. A C function call is just a jump to an address. A C++ virtual function call goes through a vtable lookup that the compiler usually can’t devirtualize. A C++ statement inside a try block has an implicit edge to the matching catch handler — meaning every line of code inside the try is potentially a branch point. That’s fine for application code, but it’s a problem for:

Aerospace and medical devices. NASA’s coding standards for flight software restrict C++ to a subset that excludes exceptions and most polymorphism, precisely so that automated verification tools can reason about the program’s control flow. If you can’t reach the device to debug it (because the device is on Mars, or inside a patient), you really want a small, analyzable program.
Hard real-time systems. A C function has a tight, predictable upper bound on its runtime. A C++ function that may throw, may call into a virtual override, or may invoke an allocator with hidden behavior can blow that bound.
Library Interface to Other Languages
This is the killer feature. Almost every mainstream language can call C functions through a foreign function interface:

Python: ctypes (standard library) or cffi
Java: JNI
C#: [DllImport]
Rust: extern "C"
Go: cgo
Ruby, R, Lua, OCaml, Haskell, Swift, …
So if you write a high-performance routine — a numerical solver, a cryptographic primitive, an image filter — and you expose it with a C ABI, everyone can use it. The same routine in C++ would expose name-mangled symbols that change between compilers and standard-library versions, and would force callers to deal with C++ runtime initialization.

The one language that famously cannot call into C is JavaScript running in a browser. This is not a technical limitation — it’s a deliberate security boundary. Browser JavaScript runs inside a sandbox precisely so that a malicious page cannot access your filesystem, your camera, or arbitrary memory. C has unrestricted access to all of those. If browser JavaScript could call into native C code, the entire sandbox guarantee would evaporate. (WebAssembly is the modern workaround: you compile C to a sandboxed bytecode that the browser runs in the same isolated environment as JavaScript.)

goto, Reconsidered
C has a goto statement that jumps to a labeled position in the same function:

#include <stdio.h>

int main(void) {
    int num;
    printf("Enter a number: ");
    scanf("%d", &num);

    if (num > 0) {
        goto positive;
    }
    goto end;

positive:
    printf("It is a positive number.\n");

end:
    printf("Program finished.\n");
    return 0;
}
In 1968, Edsger Dijkstra published a one-page note titled “Go To Statement Considered Harmful”, arguing that unrestricted goto makes it impossible to reason about a program’s state at any point — you cannot tell, from looking at a line of code, what could have led to it executing. The note kicked off the structured-programming movement and effectively killed goto in mainstream code.

The rule for modern C code: prefer if / else / while / for / break / continue / function calls. Don’t use goto to fake a loop or to simulate exception handling across deeply-nested blocks.

The one idiomatic exception: the “cleanup label” pattern in functions that acquire multiple resources, where each resource needs to be released on every error path. The Linux kernel uses this heavily:

int load_config(const char* path) {
    FILE* file   = NULL;
    char* buffer = NULL;
    int   rc     = -1;

    file = fopen(path, "rb");
    if (file == NULL) goto cleanup;

    buffer = malloc(BUFSIZE);
    if (buffer == NULL) goto cleanup;

    if (fread(buffer, 1, BUFSIZE, file) == 0) goto cleanup;

    // ... use file and buffer ...

    rc = 0;   // success

cleanup:
    free(buffer);          // free(NULL) is safe
    if (file) fclose(file);
    return rc;
}
Each early goto cleanup; jumps to a single place that frees whatever was allocated. The alternative is deeply-nested if blocks or duplicating the cleanup code at every error path, both of which are worse. This is the structured use of goto — forward-only, to a single per-function cleanup label — and is generally accepted in modern C style guides.

See Also
Makefiles & GNU Make — how to automate the compile-link pipeline for multi-file C projects, with incremental rebuilds.
Networking — most networking libraries you’ll meet are exposed through a C API for the reasons described above.
Code Smells & Refactoring — refactoring discipline applies to C, but you also have to manually track who owns each pointer.
Practice
C Programming Flashcards
Cards span Remember through Create. Mix of definition recall, code prediction, design-decision reasoning, and small code-writing problems for spaced retrieval practice.

Difficulty:
Intermediate
What is the role of libc, and how does it relate to operating-system system calls?

Show Answer
C Programming Quiz
Test your understanding of C — what's different from C++, how memory and the compilation pipeline actually work, and the design tradeoffs that motivate the language.

Difficulty:
Advanced
You are shipping a CLI tool that depends on libssl. Compare static and dynamic linking — which statement is correct?


A
Dynamic linking produces a single, self-contained binary; static linking always requires the user to install all referenced libraries separately.


B
Static linking links at run time; dynamic linking links at build time. Static linking is therefore always slower at startup.


C
Static linking produces a larger self-contained executable; dynamic linking produces a smaller binary that depends on libssl.so being present at run time.


D
Static and dynamic linking differ only in file extension (.a vs .so); the resulting executable is identical.

---

# Appendix: Lecture 13 Slides (raw extracted text)

The following is the full extracted text of Lecture 13 (L13_C_Make.pdf). This lecture covers BOTH C and Make — the Make portion is also covered in make-and-makefiles.md; the C portion belongs here. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 13 � Make &
C Programming
Assistant Teaching Professor
Computer Science Department
Why Learn C?
  Speed
C compiles directly to machine code & is very close to direct hardware
instructions. This allows experts to optimize performance
  Memory Management
C allows for explicit and direct manipulation of memory.
This requires careful coding. It enables programmers to write highly
optimized code and manage resources precisely.
  Hardware Interaction
C is unmatched when you need to interface directly with hardware, making it
essential for device drivers and firmware.
Most of the Software you use Every Day is
Runs on Applications Written in C
    Operating Systems
  Linux & Windows kernels due to speed & direct access to hardware & memory
    Embedded Systems and IoT devices
  due to small memory footprint and direct hardware access
    Compilers and Assemblers
  due to small memory footprint and direct hardware access
    Database Management Systems
  MySQL, PostgreSQL core code
C++ builds on C
� C is purely procedural (functions and data are kept separate) while
  C++ is object-oriented (encapsulating functions and data into classes)
� No Classes or Objects
   � No inheritance or polymorphism
   � Structs allow you to define data structures
       struct list_element {
                   int value;
                   struct list_element* next; // Linked list
       };
C has No Function Overloading
Each function must have a unique name
In C++:                                            In C:
void print(int value)    Function Overloading      void printInt(int value)
{...}                     (functions are allowed   {...}
                         to have the same name
void print(float value)  but different signature)  void printFloat(float value)
{...}                                              {...}
int main() {                                       int main() {
     int a = 5;                                         int a = 5;
     float b = 5.00;                                    float b = 5.00;
     print(a);                                          printInt(a);
     print(b);                                          printFloat(b);
}                                                  }
         CS 35L Software Construction: Lecture 13 � Make & C Programming         5
         Tobias D�rschmid
C has No Pass-by-References
� Only pointers are available
� Parameters passing always involved either values or pointers
In C++:                                                     In C:                Pure Pointers
void swap(int &a, int &b) {                                 void swap(int *a, int *b) {
                                                               int temp = *a;
   int temp = a;                                               *a = *b;
                                                               *b = temp;
   a = b;
                                                            }
   b = temp;
                                                            int main() {
}                 Pass by Reference                            int x = 30;
                                                               int y = 40;
int main() {      passing arguments to a function where        swap(&x, &y);
   int x = 30;    the function receives a reference to the
   int y = 40;    original variable in memory, rather than
   swap(x, y);
                               a copy of its value
                CS 35L Software Construction: Lecture 13 � Make & C Programming                 6
                Tobias D�rschmid
No built-in try/catch blocks for Error Handling
� Instead: return values are used to indicate error                     Now the result has to be a
                                                                              pointer argument
In C++:                                       In C:
int safe_divide(int num, int den) {           int safe_divide(int num, int den, int* result) {
   if (num == 0) {                               if (num == 0) {
      throw std::runtime_error("div by 0");         return -1; // Indicate an error
   }                                             }
   return numerator / denominator;               *result = num / den;
}                                                return 0; // Indicate success
int main() {                                  }
   int x = 10, y = 0, z;                      int main() {
   try {                                         int x = 10, y = 0, z;
      z = safe_divide(x, y);                     if (safe_divide(x, y, &z) != 0) {
      printf("Result: %d\n", z);                    printf("Error: Division by zero occurred.\n");
   }                                             } else {
   catch (const std::runtime_error& e) {            printf("Result: %d\n", z);
      std::cerr << " Error: Division by zero     }
occurred " << e.what() << std::endl;             return 0;
   }                                          }
}
                    CS 35L Software Construction: Lecture 13 � Make & C Programming                 7
                    Tobias D�rschmid
Again, WWHHYYYY Would Someone Use C
Instead of C++ ???
    Smaller Executable
  C compilers often generate smaller binaries because C doesn't include the
  large runtime support libraries or necessary metadata associated with C++
  features like exceptions or virtual tables & constructors/destructors for objects.
  This is crucial for environments with limited memory (e.g., embedded systems)
    Predictable Execution
  C gives the developer almost complete control over memory allocation and
  function calls. C++ introduces hidden operations (like constructor calls,
  exception unwinding, and virtual function lookups) that can make execution
  time less predictable.
Again, WWHHYYYY Would Someone Use C
Instead of C++ ???
   Library Interface
 Almost all programming languages (including Python, Java, C#) support calling
 C functions & can therefore use C libraries.
 Picking C allows you to write a library with the widest possible compatibility
import ctypes
import os
# 1. Determine the correct library       # 2. Load the shared library
# file name based on the OS              try:
if os.name == 'posix': # Linux or macOS
                                                c_lib = ctypes.CDLL(lib_file)
       lib_file = './my_c_lib.so'        except OSError as e:
elif os.name == 'nt': # Windows
                                                print(f"Error loading library: {e}")
       lib_file = './my_c_lib.dll'              print("Make sure you compiled the C
else:                                    file into the correct shared library.")
                                                exit()
       raise OSError("Unsupported OS.")
Memory Management in C
� void *malloc(size_t size)allocates memory of a given size on the heap
� void free(void *ptr) deallocates the memory
   � If you don't free the memory, you will have a memory leak
   � Accessing memory that is not currently allowed by your program results in a
     segmentation fault ("segfault")
int* matrix1 = (int*)malloc(rows * cols * sizeof(int));
[...]
free(matrix1);
See https://www.geeksforgeeks.org/c/dynamic-memory-allocation-in-c-using-malloc-calloc-free-and-realloc/
Reading a File in C              #include <stdio.h>
                                 int main() {
                                        FILE *file;
                                        int buffer[5];
� fopen opens a file                    // Open the binary file for reading
� fread reads a given amount of         file = fopen("input.bin", "rb");
                                        if (file == NULL) {
  data from a file stream
� fclose closes the file (you                  perror("Error opening file");
                                               return 1;
  should always close the file          }
  after you are done reading)
� All three are library calls           // Read the integers from the file into the buffer
                                        fread(buffer, sizeof(int), 5, file);
                                        for (int i = 0; i < 5; i++) {
                                               printf("Element %d: %d\n", i + 1, buffer[i]);
                                        }
                                        // Close the file
                                        fclose(file);
                                        return 0;
                                 }
See: https://www.geeksforgeeks.org/c/fread-function-in-c/
Library Calls
� fread is a standard C library function that provides buffered input/output
  operations
� While fread interacts with the operating system to read data from files, it does so
  by making calls to lower-level system calls as needed.         Your Program
� malloc and free are also library calls
� When creating an executable, the linker selects                       libc
  the libc implementation for your target operating system       stdio stdlib
                                                                 Operating System
                                                                 Hardware
Compiler & Linker Create an Executable
from Source Code
  Compiler/Assembler
Translates your source code into assembly code (also knows as an "object file")
your_file.c  Compiler        your_file.o
  Linker
Resolves referenced but not defined symbols (function names, global variables)
Copies code sections from object files that define those symbols into the
executable (static linking) or defers until program execution (dynamic linking)
                  library.o
your_file.o       Linker     executable
Important C Language Elements
� Characters & Strings:
   � 'a' is a single character ('\0' not the character for zero but the character
     with ASCII value zero (end of string)
   � "a" is a string (char* or char[]) (don't forget #include <string.h>)
� Use const to indicate that the variable should not be changed
   � const char* does not let you write into the memory of the pointer
� const in C is a great way to avoid accidentally usings a variable in the wrong
  way so it prevents some bugs
� you can cast away the costness but avoid this!)
char buffer[] = "Initial string"; // Modifiable array on the stack
const char *p_const = buffer; // p_const prevents modifications to the buffer
char *p_writable = (char *)p_const; // p_writable re-enables modifications
Goto allows you to jump to a different, Labeled
Code Location
#include <stdio.h>
int main() {
       int num;
       printf("Enter a number: ");
       scanf("%d", &num);
       if (num > 0) {
              goto positive;
       }
       goto end;
positive: // A label named 'positive'
       // This block is reached only by the goto statement above
       printf("It is a positive number.\n");
end: // A label named 'end' for a clean exit point
       printf("Program finished.\n");
       return 0;
}
Edsger Dijkstra: "Go To Statement Considered
                                       Dijkstra's 1968 paper
Harmful"
                                       Scanned image of Edsger Dijkstra's 1968 paper titled 'Go To Statement Considered Harmful,' published in Communications of the ACM. Th
                                       paper argSuees ethaht utsteposf t:h/e/hGOoTmO setapteamgenet ssh.oculwd bie.nablo/~lishsetdofrromm h/tigehear-clehveilnpgrog/rraemamdineg lra/nDguiajgkess.tra68.pdf
#include <stdio.h>
int main() {
       int num;
       printf("Enter a number: ");
       scanf("%d", &num);
       if (num > 0) {
              goto positive;
       }
       goto end;
positive: // A label named 'positive'
       // This block is reached only by the goto statement above
       printf("It is a positive number.\n");
end: // A label named 'end' for a clean exit point
       printf("Program finished.\n");
       return 0;
}
CS 35L Software
Construction
Lecture 13 � Make &
Makefiles
Assistant Teaching Professor
Computer Science Department
Let's Bake a Bruin Cake!
                  Final Cake
Chocolate Layers  Raspberry Filling  Buttercream
flour cocoa eggs sugar raspberies lemon juice butter powdered sugar vanilla
Let's Bake a Bruin Cake!
Was actually SALT! But we   Final Cake
got some new ingredients
  Now it's actually sugar!
    What should we do?
Chocolate Layers            Raspberry Filling  Buttercream
flour cocoa eggs sugar raspberies lemon juice butter powdered sugar vanilla
Let's Bake a Bruin Cake!
                                FiFnainlaCl aCkaeke
ChCohcooclaotlaeteLaLyaeyresrs  RaRsapsbpebreryrrFyilFliinllging  BuBttuettrecrceraemam
flour cocoa eggs sugar raspberies lemon juice butter powdered sugar vanilla
GNU Make is a Universal Build Automation Tool
  Goal: Incremental Build Automation
With so many different source files, compilation & linking steps,
how can I automate the build of a large program while only re-building parts
of the program for which the source files have changed?
  Solution: GNU Make
� Automates building, installing, uninstalling, and testing a package
� Figures out automatically which files it needs to update, based on which
   source files have changed.
� Language-independent (C, C++, Java, LaTeX, ...)]
� Specify steps in Makefile
See https://www.gnu.org/software/make/
                                                                                                            See https://makefiletutorial.com/
Anatomy of a Very Simple Makefile
  Build Target                            Makefile
                                         target: prereq_1 prereq_2 ...
A product / file you want to produce
(e.g., executable, object file, ...)               command_1
or a task you want to perform                      command_2
(e.g., "install", "test").
The name of the build target should be   mysrc.o: mysrc.c
the name of the produced file.
                                                   cc mysrc.c �o mysrc.o
  Prerequisites
                                                       cc is the C compiler
A list of build targets that need to be
made before this build target and files      Command
that are used to make this target
                                          a sequence of shell commands to
                                          build the target or to perform the task
                                                                                                            See https://makefiletutorial.com/
Make Allows you to Build in Multiple Steps
� Before making a build                           Makefile
  target, make first makes                      myprogam: mysrc.o
  all the dependencies              Runs last cc mysrc.o �o myprogam
  (unless they have not
  changed)                                      mysrc.o: mysrc.c
                                  Runs second cc mysrc.c �o mysrc.o
� Dependencies have               mysrc.c:
  changed if and only if the
  modification date of their      Runs first echo "int main()
  file is newer than that of the              {\n\treturn 0; \n}" > mysrc.c"
  build target
                                  Terminal     Run with any build
                                            target you want to make
                                  make myprogram
                                                                                                            See https://makefiletutorial.com/
Make Allows you to Build in Multiple Steps
                               Makefile
                             myprogam: mysrc.o
                  Runs last cc mysrc.o �o myprogam
                                mysrc.o: mysrc.c
                  Runs second cc mysrc.c �o mysrc.o
                  mysrc.c:
                  Runs first echo "int main()
                              {\n\treturn 0; \n}" > mysrc.c"
                  Terminal     Run with any build
                            target you want to make
                  make myprogram
If a Build Target should ALWAYS                         See https://makefiletutorial.com/
be made, Define it as .PHONY
  How it works                            Makefile
                                         .PHONY: test clean
� Commands of build targets listed       test:
   as .PHONY will always be
executed when the build target is                 pytest tests
made.                                    clean:
                                                  rm �r *.o
When to use Phony
� You want to create a file with the name of a build target that a task that
does not make this files (e.g., "test")
� Performance: Skip checking whether the files are new
� Commands commonly made phony: all, clean, test
                             See https://makefiletutorial.com/
Makefiles Support Variables
                   Makefile
                  VARIABLE_NAME = definition
                  SRCS = mysrc1.c mysrc2.c
                  OBJS = $(SRCS:.c=.o)
                  myprog: $(OBJS)
                           cc $(OBJS) -o myprog
                                                  See https://makefiletutorial.com/
Makefiles Support Variables
� Variables are defined with a    Makefile
  string that should substitute  VARIABLE_NAME = definition
  the variable name later
                                 SRCS = mysrc1.c mysrc2.c
� $(VARNAME) uses the
  variable                       OBJS = $(SRCS:.c=.o)
� Make supports string-          myprog: $(OBJS)  Replaces .c with .o
  replace in variables using
  the syntax: $(VAR:search       cc $(OBJS) -o myprog
  _pattern=replacement
  _text)                                          Translates to:
                                 cc mysrc1.o mysrc2.o �o myprog
Pattern Rules are Templates                      See https://makefiletutorial.com/
For Creating Many Rules with the Same Structure
� % is a wildcard that matches                   Can we use variables to
  any non-empty substring
                                Makefile         refactor the Makefile?
� When used in the
  prerequisite pattern, the     VARIABLE_NAME = definition
  wildcard % adds the
                                SRCS = mysrc1.c mysrc2.c
                                OBJS = $(SRCS:.c=.o)
matched string.                 myprog: $(OBJS)
� $@ is the name of the build   cc $(OBJS) -o myprog
target of the rule              %.o: %.c
� $< is the first prerequisite  cc $< -o $@
  of the rule ($^ is all
  prerequisites)                                For mysrc1.o translates to:
                                              cc mysrc1.c �o mysrc1.o
Pattern Rules are Templates                See https://makefiletutorial.com/
For Creating Many Rules with the Same Structure
Variables allow you to refactor   Makefile
your Makefile to reduce          SRCS = mysrc1.c mysrc2.c
duplication.                     TARGET = myprog
If we want to rename the         OBJS = $(SRCS:.c=.o)
program name, we only need to    $(TARGET): $(OBJS)
do this in one place
                                          cc $(OBJS) -o $(TARGET)
                                 %.o: %.c
Can we refactor this Makefile             cc $< -o $@
         even further???         clean:
                                          rm �f $(OBJS) $(TARGET)
Pattern Rules are Templates    See https://makefiletutorial.com/
For Creating Many Rules with the Same Structure
If we want to use a different   Makefile
C compiler (gcc, clang) we     SRCS = mysrc1.c mysrc2.c
                               TARGET = myprog
   can simply change this      OBJS = $(SRCS:.c=.o)
                here           CC = gcc
                               $(TARGET): $(OBJS)
                                        $(CC) $(OBJS) -o $(TARGET)
                               %.o: %.c
                                        $(CC) $< -o $@
                               clean:
                                        rm �f $(OBJS) $(TARGET)
                                                       See https://makefiletutorial.com/
The stereotypical Makefile for C Programs
If we want to compile Makefile
with different compiler      SRCS = mysrc1.c mysrc2.c
flags (arguments to the
                             TARGET = myprog
    compiler) we can
    change this here         OBJS = $(SRCS:.c=.o) Shows all compiler
                             CC = clang
                                                       warnings
                             CCFLAGS = -Wall
Runs static analysis         CHECKFLAGS = --analyze �Xanalyzer
(automatic analysis of your                            -analyzer-checker=core
   source code) to detect    $(TARGET): $(OBJS)
  issue such as resource               $(CC) $(CCFLAGS) -o $(TARGET) $(OBJS)
leaks due to missing free
and fclose, double free,     %.o: %.c
                                       $(CC) $(CCFLAGS) -c $< -o $@
 null pointer dereferences
                             clean:
                             rm �f $(OBJS) $(TARGET)
CS 35L SoftwarecChoensctkru:ction: Lecture 13 � Make & C Programming          31
Please fill out your Exit Tickets on Bruin Learn!
In your own words, please summarize three key insights you learned about C
Programming and Makesfiles today. (Do not copy phrases from the lecture slides)
Why is GNU Make more useful for C/C++ than for Python and JavaScript?
Please describe one use case that applies to C/C++ but not to Python & JS
and one use case that still applies to Python & JS.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slide use images from Flaticon.com (Creators: Freepik)

```
