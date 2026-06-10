# Make and Makefiles — CS35L Study Guide

This guide combines the comprehensive Make reference (`make.md`) and the Make-specific content from Lecture 13 (L13_C_Make.pdf). It preserves all targets, prerequisites, recipes, automatic variables, .PHONY, pattern rules, variables, implicit rules, incremental builds, the cake analogy, and clang static analysis flags.

---

## 1. Motivation

Imagine you are building a small C program. It just has one file, `main.c`. To compile it, you simply open your terminal and type:

```
gcc main.c -o myapp
```

Easy enough, right?

Now, imagine your project grows. You add `utils.c`, `math.c`, and `network.c`. Your command grows too:

```
gcc main.c utils.c math.c network.c -o myapp
```

Still manageable. But what happens when you join a real-world software team? An operating system kernel or a large application might have thousands of source files. Typing them all out is impossible.

### First Attempt: The Shell Script

To solve this, you might write a simple shell script (`build.sh`) that just compiles everything in the directory: `gcc *.c -o myapp`

This works, but it introduces a massive new problem: **Time**. Compiling a massive codebase from scratch can take minutes or even hours. If you fix a single typo in `math.c`, your shell script will blindly recompile all 9,999 other files that didn't change. That is incredibly inefficient and will destroy your productivity as a developer.

### The "Aha!" Moment: Incremental Builds

What you actually need is a smart tool that asks two questions before doing any work:

1. **What exactly depends on what?** (e.g., "The executable depends on the object files, and the object files depend on the C files and Header files").
2. **Has the source file been modified more recently than the compiled file?**

If `math.c` was saved at 10:05 AM, but `math.o` (its compiled object file) was created at 9:00 AM, the tool knows `math.c` has changed and must be recompiled. If `utils.c` hasn't been touched since yesterday, the tool completely skips recompiling it and just reuses the existing `utils.o`.

This is exactly why **make** was created by **Stuart Feldman at Bell Labs in 1976** (Feldman 1979), and why it remains a staple of software engineering today. Modern development primarily relies on **GNU Make**, a powerful and widely-extended implementation that reads a configuration file called a **Makefile**.

So GNU make is the project's engine that reads recipes from Makefiles to build complex products.

---

## 2. How It Works

Inside a Makefile, you define three main components:

- **Targets**: What you want to build or the task you want to run.
- **Prerequisites**: The files that must exist (or be updated) before the target can be built.
- **Commands**: The exact terminal steps required to execute the target.

When you type `make` in your terminal, the tool analyzes the dependency graph and checks file modification timestamps. It then executes the bare minimum number of commands required to bring your program up to date.

### The Dual Purpose

Makefiles are incredibly powerful—but their design can be confusing at first glance because they serve two distinct purposes:

- **Building Artifacts**: Their primary, traditional use is for compiling languages (like C and C++), where they manage the complex process of turning source code into executable files.
- **Running Tasks**: In modern development, they are frequently used with interpreted languages (like Python) as a convenient shortcut for common project tasks (e.g., `make install`, `make test`, `make lint`, `make deploy`).

### Why We Need Makefiles

Ultimately, Makefiles are heavily relied upon because they:

- **Save massive amounts of time** by enabling incremental builds (only recompiling the specific files that have changed).
- **Automate complex processes** so developers don't have to memorize long or tedious terminal commands.
- **Standardize workflows** across teams by providing predictable, universal commands (like `make test` to run all tests or `make clean` to delete generated files).
- **Document dependencies**, making it perfectly clear how all the individual pieces of a software system fit together.

---

## 3. The Cake Analogy

Think of Makefiles as a **recipe book for baking a complex, multi-layered cake**. Let's make a spectacular three-tier chocolate cake with raspberry filling and buttercream frosting. A Makefile is your ultimate, highly-efficient kitchen manager and master recipe combined.

### Concepts

#### 1. The Targets (What you are making)

In a Makefile, a **target** is the file you want to generate.

- **The Final Target (The Executable)**: This is the fully assembled, frosted, and decorated cake ready for the display window.
- **Intermediate Targets (e.g., Object Files in C)**: These are the individual components that must be made before the final cake can be assembled. In this case, your intermediate targets are the baked chocolate layers, the raspberry filling, and the buttercream frosting. If we know how to bake each individual component and we know how to combine each of them together, we can bake the cake. Makefiles allow you to define the targets and the dependencies in a structured, isolated way that describes each component individually.

#### 2. The Dependencies (What you need to make it)

Every target in a Makefile has **dependencies**—the things required to build it.

- **Raw Source Code (Source Files)**: These are your raw ingredients: flour, sugar, cocoa powder, eggs, butter, and fresh raspberries.
- **Chain of Dependencies**: The Final Cake depends on the chocolate layers, filling, and frosting. The chocolate layers depend on flour, sugar, eggs, and cocoa powder.

### Worked Example of the Cake Recipe

#### Iteration 1: The Basic Rule (The Blueprint)

**The Need**: We need to tell our kitchen manager (`make`) what our final goal is, what it requires, and how to put it together.

**The Syntax**: The most fundamental building block of a Makefile is a **Rule**. A rule has three parts:

- **Target**: What you want to build (followed by a colon `:`).
- **Dependencies**: What must exist before you can build it (separated by spaces).
- **Command**: The actual terminal command to build it. **CRITICAL**: This line must start with a literal **Tab** character, not spaces.

```makefile
# Step 1: The Basic Rule
cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking chocolate_layers, raspberry_filling, and buttercream to make the cake."
	touch cake
```

**Note**: If you run this now (i.e., ask the kitchen manager to bake the cake), `make cake` will complain: "No rule to make target 'chocolate_layers'". It knows it needs them, but it doesn't know how to bake them.

#### Iteration 2: The Dependency Chain

**The Need**: We need to teach `make` how to create the missing intermediate ingredients so it can satisfy the requirements of the final cake.

**The Syntax**: We simply add more rules. The **order of rules in the Makefile does not matter for execution** — `make` reads all the rules, builds a dependency graph from them, and then traverses that graph from the goal target down to the leaves, building each prerequisite before the target that needs it. The **first non-special rule in the file is used as the default goal** if no target is given on the command line.

```makefile
# Step 2: Adding the Chain
cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking layers, filling, and frosting to make the cake."
	touch cake

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Mixing ingredients and baking at 350 degrees."
	touch chocolate_layers

raspberry_filling: raspberries.txt sugar.txt
	echo "Simmering raspberries and sugar."
	touch raspberry_filling

buttercream: butter.txt powdered_sugar.txt
	echo "Whipping butter and sugar."
	touch buttercream
```

Now the kitchen works! But notice we hardcoded "350 degrees". If we get a new convection oven that bakes at 325 degrees, we have to manually find and change that number in every single baking rule.

#### Iteration 3: Variables (Macros)

**The Need**: We want to define our kitchen settings in one place at the top of the file so they are easy to change later.

**The Syntax**: You define a variable with `NAME = value` and you use it by wrapping it in a dollar sign and parentheses: `$(NAME)`.

```makefile
# Step 3: Variables
OVEN_TEMP = 350
MIXER_SPEED = high

cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking layers to make the cake."
	touch cake

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Baking at $(OVEN_TEMP) degrees."
	touch chocolate_layers

buttercream: butter.txt powdered_sugar.txt
	echo "Whipping at $(MIXER_SPEED) speed."
	touch buttercream
```

(The filling rule is omitted here just to keep the example short, but you get the idea).

#### Iteration 4: Automatic Variables (The Shortcuts)

**The Need**: Look at the `chocolate_layers` rule. We list all the ingredients in the dependencies, but in a real C++ program, you also have to list all those exact same files again in the compiler command. Typing things twice causes typos.

**The Syntax**: Makefiles have built-in "Automatic Variables" that act as shortcuts:

- `$@` automatically means "The name of the current target".
- `$^` automatically means "The names of ALL the dependencies".

```makefile
# Step 4: Automatic Variables
OVEN_TEMP = 350

cake: chocolate_layers raspberry_filling buttercream
	echo "Making $@"
	touch $@

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Taking $^ and baking them at $(OVEN_TEMP) to make $@"
	touch $@
```

Now, the command `echo "Taking $^ ..."` will automatically print out: "Taking flour.txt sugar.txt eggs.txt cocoa.txt…". If you add a new ingredient to the dependency list later, the command updates automatically!

#### Iteration 5: Phony Targets (.PHONY)

**The Need**: Sometimes we make a terrible mistake and just want to throw everything in the trash and start completely over. We want a command to wipe the kitchen clean.

**The Syntax**: We create a rule called `clean` that deletes files. However, what if you accidentally create a real text file named "clean" in your folder? `make` will look at the file, see it has no dependencies, and say "The file 'clean' is already up to date. I don't need to do anything."

To fix this, we use `.PHONY`. This tells `make`: "Hey, this isn't a real file. It's just a command name. Always run it when I ask."

```makefile
# Step 5: The Final, Complete Scaffolding
OVEN_TEMP = 350

cake: chocolate_layers raspberry_filling buttercream
	echo "Making $@"
	touch $@

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Taking $^ and baking them at $(OVEN_TEMP) to make $@"
	touch $@

# ... (other recipes) ...

.PHONY: clean
clean:
	echo "Throwing everything in the trash!"
	rm -f cake chocolate_layers raspberry_filling buttercream
```

By typing `make clean` in your terminal, the kitchen is reset. By typing `make cake` (or just `make`, as it defaults to the first rule), your fully automated bakery springs to life.

### The Complete Cake Makefile

```makefile
# ---------------------------------------------------------
# Complete Makefile for a Three-Tier Chocolate Raspberry Cake
# ---------------------------------------------------------

# Variables (Kitchen settings)
OVEN_TEMP = 350
MIXER_SPEED = medium-high

# 1. The Final Target: The Cake
# Depends on the baked layers, filling, and frosting
cake: chocolate_layers raspberry_filling buttercream
	@echo "Assembling the final cake!"
	@echo "-> Stacking layers, spreading filling, and covering with frosting."
	@touch cake
	@echo "Cake is ready for the display window!"

# 2. Intermediate Target: Chocolate Layers
# Depends on raw ingredients (our source files)
chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	@echo "Mixing flour, sugar, eggs, and cocoa..."
	@echo "Baking in the oven at $(OVEN_TEMP) for 30 minutes."
	@touch chocolate_layers
	@echo "Chocolate layers are baked."

# 3. Intermediate Target: Raspberry Filling
raspberry_filling: raspberries.txt sugar.txt lemon_juice.txt
	@echo "Simmering raspberries, sugar, and lemon juice."
	@touch raspberry_filling
	@echo "Raspberry filling is thick and ready."

# 4. Intermediate Target: Buttercream Frosting
buttercream: butter.txt powdered_sugar.txt vanilla.txt
	@echo "Whipping butter and sugar at $(MIXER_SPEED) speed."
	@touch buttercream
	@echo "Buttercream frosting is fluffy."

# 5. Pattern Rule: "Shopping" for Raw Ingredients
# In a real codebase, these would already exist as your code files.
# Here, if an ingredient (.txt file) is missing, Make creates it.
%.txt:
	@echo "Buying ingredient: $@"
	@touch $@

# 6. Phony Target: Clean the kitchen
# Removes all generated files so you can bake from scratch
.PHONY: clean
clean:
	@echo "Cleaning up the kitchen..."
	@rm -f cake chocolate_layers raspberry_filling buttercream *.txt
	@echo "Kitchen is spotless!"
```

### 3. The Rules (The Recipe/Commands)

A rule in a Makefile pairs a target with its prerequisites and a **recipe**: the sequence of shell commands `make` runs to turn those prerequisites into the target. The recipe doesn't have to call a compiler — it's just shell commands, so `make` can drive any tool (linter, packager, doc generator, deployer).

- **Compiling**: The rule to turn flour, sugar, and eggs into a chocolate layer is: "Mix ingredients in bowl A, pour into a 9-inch pan, and bake at 350°F for 30 minutes."
- **Linking**: The rule to turn the individual layers, filling, and frosting into the Final Cake is: "Stack layer, spread filling, stack layer, cover entirely with frosting."

This can be visualized as a **dependency graph**: the final cake depends on chocolate layers, raspberry filling, and buttercream; chocolate layers depend on flour, sugar, and eggs; raspberry filling depends on raspberries and sugar; buttercream depends on butter and powdered sugar.

### The Real Magic: Incremental Baking (Why we use Makefiles)

The true power of a Makefile isn't just knowing how to bake the cake; it's knowing **what doesn't need to be baked again**. Make looks at the "timestamps" of your files to save time.

Imagine you are halfway through assembling your cake. You have your baked chocolate layers sitting on the counter, your buttercream whipped, and your raspberry filling ready. Suddenly, you realize someone mislabeled the sugar. It's actually salt! Oh no! You need to remake everything that included sugar and everything that included these intermediate targets.

- **Without a Makefile**: You would throw away everything. You would re-bake the chocolate layers, re-whip the buttercream, and remake the raspberry filling from scratch. This takes hours (like recompiling a massive codebase from scratch).
- **With a Makefile**: The kitchen manager (`make`) looks at the counter. It sees that the buttercream is already finished and its raw ingredients haven't changed. However, it sees your new packet of sugar (a source file was updated). The manager says: "Only remake the raspberry filling and the chocolate layers, and then reassemble the final cake. Leave the buttercream as is."

If you look closely at the arrows of the dependency graph above and focus on the arrows leaving `[sugar.txt]`, you can immediately see the brilliance of `make`:

- **The Split Path**: The arrow from `sugar.txt` forks into two different directions: one goes to the Chocolate_Layers and the other goes to the Raspberry_Filling.
- **The Safe Zone**: Notice there is absolutely no arrow connecting `sugar.txt` to the Buttercream (which uses powdered sugar instead).
- **The Chain Reaction**: When `make` detects that `sugar.txt` has changed (because you fixed the salty sugar), it travels along those two specific arrows. It forces the Chocolate Layers and Raspberry filling to be remade. Those updates then trigger the double-lined arrows ══▶, forcing the Final Cake to be reassembled.

Because no arrow carried the "sugar update" to the Buttercream, the Buttercream is completely ignored during the rebuild!

### Lecture: The Bruin Cake Scenario

The lecture frames the same scenario with the **UCLA Bruin Cake**:

- Layers: `Final Cake` ← `Chocolate Layers`, `Raspberry Filling`, `Buttercream`
- Leaves: `flour`, `cocoa`, `eggs`, `sugar`, `raspberries`, `lemon juice`, `butter`, `powdered sugar`, `vanilla`
- Scenario: "Was actually SALT! But we got some new ingredients. Now it's actually sugar! What should we do?" — the answer is: only rebuild the items downstream of `sugar` (Chocolate Layers, Raspberry Filling, and the Final Cake). Buttercream is untouched because it depends on `powdered sugar`, not `sugar`.

---

## 4. How `make` Decides What To Rebuild (Reading Conventions for Dependency Graphs)

A reading guide for each diagram (these conventions are the same ones the interactive Makefile tutorial uses):

- **Solid green stripe + check glyph** — the file is up to date.
- **Diagonal-hatched red stripe + filled-circle glyph (pulsing)** — the target is stale; `make` would rebuild it.
- **Dashed border + crosshair glyph** — the target is **phony** (not a file). `make` always runs it.
- **Italic, no border** — the file is a **source**. `make` never rebuilds these; you (or your editor) do.
- **Dashed edge** — an **order-only prerequisite**. The arrow says "must exist before me", not "rebuild me when newer."

### Demo 1 — What `make` checks

A small C project. `app` is the final executable; `main.o`/`util.o` are intermediate object files; `main.c`/`util.c`/`shared.h` are sources. Every target's mtime is greater than its prerequisites' — so all targets are up to date.

When you run `make`, it walks this graph from the top. For each target, it asks one simple question: **is any of my prerequisites newer than me?** If yes, rebuild this target. If no, skip it. Phony targets bypass the comparison entirely (they're always considered "needs running"). That's the entire algorithm.

### Demo 2 — Touching a source file → cascade of staleness

You edit `main.c`. Watch the cascade of staleness, then how `make` rebuilds only what changed. The two-step rhythm is the entire developer feedback loop: **edit → make**, every time.

A common student misconception: "if anything changes, `make` recompiles everything." That's not how it works — **only nodes downstream of the change in the dependency graph are rebuilt**. The graph is the contract that lets `make` skip work safely.

### Demo 3 — Phony targets always run

`clean` is phony (dashed border, crosshair glyph). `make` doesn't compare timestamps for phony targets — it always runs the recipe. Here the recipe is `rm -f *.o app`. After it runs, `app`, `main.o`, `util.o` no longer exist on disk; they show as stale (red hatched stripe) because there's no file to compare against. Sources `main.c`/`util.c` are never touched.

The contrast that makes this concept stick: a non-phony target with no prerequisites would be considered "up to date as long as the file exists." The `.PHONY` declaration is what flips the switch. Common phony targets include `clean`, `install`, `test`, `run`, `dist`, `docs`. They're **verbs (actions)** rather than **nouns (files)**.

### Demo 4 — Order-only prerequisites

An **order-only prerequisite** (after the `|` in the Makefile rule) tells `make`: "this must exist before I run, but don't trigger a rebuild just because it has a newer mtime." The classic use is a build directory whose mtime updates every time a file is written into it — without order-only, every `.o` would be rebuilt every time. Notice: the `app → build` edge is dashed. After we touch `build`, its mtime is the newest in the graph — but `app` stays up to date. With a normal edge, `app` would be marked stale.

Order-only is the answer to one of the most painful "why does my build keep redoing everything?" mysteries. It separates the two distinct ideas that students often conflate: "X must come before Y" vs. "X being newer means Y is out of date." The first is **ordering**, the second is **staleness propagation** — and Makefiles let you choose.

### Demo 5 — Putting it together: edit → build → clean → rebuild

The full developer rhythm. We start fresh, edit a header (which has a wider blast radius than a single `.c`), rebuild, clean, and rebuild from scratch. Each step shows which files are touched and which are skipped.

If you can predict, before clicking, what each step will change in the graph — you have a working mental model of `make`. (Editor headers cascade widely, phony targets always run, missing targets are stale.) That mental model is the single biggest payoff of learning Make: it transfers directly to every other build tool you'll meet later (**Bazel, Gradle, Ninja, esbuild's incremental mode**), because they all reduce to "what's stale, in topological order."

### A Recipe as a Makefile

If your cake recipe were written as a Makefile, it would look exactly like this:

```
Final_Cake: Chocolate_Layers Raspberry_Filling Buttercream
    Stack components and frost the outside.

Chocolate_Layers: Flour Sugar Eggs Cocoa
    Mix ingredients and bake at 350°F for 30 minutes.

Raspberry_Filling: Raspberries Sugar Lemon_Juice
    Simmer on the stove until thick.

Buttercream: Butter Powdered_Sugar Vanilla
    Whip in a stand mixer until fluffy.
```

Whenever you type `make` in your terminal, the system reads this recipe from the top down, checks what is already sitting in your "kitchen", and only does the work absolutely necessary to give you a fresh cake.

---

## 5. Makefile Syntax

### How Do Makefiles Work?

A Makefile is built around a simple logical structure consisting of **Rules**. A rule generally looks like this:

```makefile
target: prerequisites
	command
```

- **Target**: The file you want to generate (like an executable or an object file), or the name of an action to carry out (like `clean`).
- **Prerequisites (Dependencies)**: The files that are required to build the target.
- **Commands (Recipe)**: The shell commands that `make` executes to build the target. (**Note**: Commands MUST be indented with a **Tab** character, not spaces!)

When you run `make`, it looks at the target. If any of the prerequisites have a newer modification timestamp than the target, `make` executes the commands to update the target. The dependency relationships you declare matter immensely; for example, if you remove the object files (`$(OBJS)`) prerequisite from your main executable rule (e.g., `$(TARGET): $(OBJS)`), `make` will no longer trigger a re-link when the object files change, because the dependency relationship has been removed.

### Syntax Basics

To write flexible and scalable Makefiles, you will use a few specific syntactic features:

- **Variables (Macros)**: Variables act as placeholders for command-line options, making the build rules cleaner and easier to modify. For example, you can define a variable for your compiler (`CC = clang`) and your compiler flags (`CFLAGS = -Wall -g`). When you want to use the variable, you wrap it in parentheses and a dollar sign: `$(CC)`.
- **String Substitution**: You can easily transform lists of files. For example, to generate a list of `.o` object files from a list of `.c` source files, you can use the syntax: `OBJS = $(SRCS:.c=.o)`.
- **Automatic Variables**: `make` provides special variables to make rules more concise.
  - `$@` represents the **target name**.
  - `$<` represents the **first prerequisite**.
  - `$^` represents **all prerequisites**.
- **Pattern Rules**: Pattern rules serve as templates for creating many rules with the identical structure. For instance, `%.o : %.c` defines a generic rule for creating a `.o` (object) file from a corresponding `.c` (source) file.

### A Worked Example (Stereotypical Robust C Makefile)

Let's tie all of these concepts together into a stereotypical, robust Makefile for a C program.

```makefile
# Variables
SRCS = mysrc1.c mysrc2.c
TARGET = myprog
OBJS = $(SRCS:.c=.o)
CC = clang
CFLAGS = -Wall

# Main Target Rule
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $(TARGET) $(OBJS)

# Pattern Rule for Object Files
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Clean Target
clean:
	rm -f $(OBJS) $(TARGET)
```

**Breaking it down:**

- **Lines 2-6**: We define our variables. If we later want to use the `gcc` compiler instead, or add an optimization flag like `-O3`, we only need to change the `CC` or `CFLAGS` variables at the top of the file.
- **Lines 9-10**: This rule says: "To build `myprog`, I need `mysrc1.o` and `mysrc2.o`. To build it, run `clang -Wall -o myprog mysrc1.o mysrc2.o`."
- **Lines 13-14**: This pattern rule explains how to turn a `.c` file into a `.o` file. It tells Make: "To compile any object file, use the compiler to compile the first prerequisite (`$<`, which is the `.c` file) and output it to the target name (`$@`, which is the `.o` file)".
- **Lines 17-18**: The `clean` target is a convention used to remove all generated object files and the target executable, leaving only the original source files. You can execute it by running `make clean`.

---

## 6. Lecture 13 — Make & Makefiles

### Why Make Exists (Lecture Framing)

**Goal: Incremental Build Automation** — With so many different source files, compilation & linking steps, how can I automate the build of a large program while only re-building parts of the program for which the source files have changed?

**Solution: GNU Make**
- Automates **building, installing, uninstalling, and testing** a package
- Figures out automatically which files it needs to update, based on which source files have changed.
- **Language-independent** (C, C++, Java, LaTeX, …)
- Specify steps in **Makefile**

(Reference: https://www.gnu.org/software/make/)

### Anatomy of a Very Simple Makefile

```makefile
target: prereq_1 prereq_2 …
	command_1
	command_2

mysrc.o: mysrc.c
	cc mysrc.c -o mysrc.o
```

- **Build Target**: A product / file you want to produce (e.g., executable, object file, …) or a task you want to perform (e.g., "install", "test"). The name of the build target should be the **name of the produced file**.
- **Prerequisites**: A list of build targets that need to be made before this build target and files that are used to make this target.
- **Command**: A sequence of **shell commands** to build the target or to perform the task.
- `cc` is the C compiler.

(Reference: https://makefiletutorial.com/)

### Make Allows you to Build in Multiple Steps

```makefile
myprogam: mysrc.o
	cc mysrc.o -o myprogam        # Runs last

mysrc.o: mysrc.c
	cc mysrc.c -o mysrc.o         # Runs second

mysrc.c:
	echo "int main() {\n\treturn 0; \n}" > mysrc.c"   # Runs first
```

Terminal:
```
make myprogram
```
(Run with any build target you want to make.)

- **Before making a build target, `make` first makes all the dependencies** (unless they have not changed).
- Dependencies have **changed if and only if the modification date of their file is newer than that of the build target**.

The corresponding chain visualized: `myprogam ← mysrc.o ← mysrc.c`.

### If a Build Target Should ALWAYS Be Made, Define it as .PHONY

```makefile
.PHONY: test clean

test:
	pytest tests

clean:
	rm -r *.o
```

**How it works**: Commands of build targets listed as `.PHONY` will always be executed when the build target is made.

**When to use Phony**:
- You want to create a file with the name of a build target that is a task that does not make this file (e.g., "test").
- **Performance**: Skip checking whether the files are new.
- Commands commonly made phony: `all`, `clean`, `test`.

### Makefiles Support Variables

```makefile
VARIABLE_NAME = definition

SRCS = mysrc1.c mysrc2.c
OBJS = $(SRCS:.c=.o)

myprog: $(OBJS)
	cc $(OBJS) -o myprog
```

- Variables are defined with a string that should substitute the variable name later.
- `$(VARNAME)` uses the variable.
- Make supports **string-replace in variables** using the syntax: `$(VAR:search_pattern=replacement_text)`.
- `$(SRCS:.c=.o)` replaces `.c` with `.o`.

Translates to:
```
cc mysrc1.o mysrc2.o -o myprog
```

### Pattern Rules are Templates For Creating Many Rules with the Same Structure

```makefile
SRCS = mysrc1.c mysrc2.c
OBJS = $(SRCS:.c=.o)

myprog: $(OBJS)
	cc $(OBJS) -o myprog

%.o: %.c
	cc $< -o $@
```

- `%` is a **wildcard that matches any non-empty substring**.
- When used in the prerequisite pattern, the wildcard `%` adds the matched string.
- `$@` is the **name of the build target** of the rule.
- `$<` is the **first prerequisite** of the rule (`$^` is **all prerequisites**).

For `mysrc1.o` translates to:
```
cc mysrc1.c -o mysrc1.o
```

### Refactoring with Variables

Variables allow you to refactor your Makefile to reduce duplication. If we want to rename the program name, we only need to do this in one place.

```makefile
SRCS = mysrc1.c mysrc2.c
TARGET = myprog
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	cc $(OBJS) -o $(TARGET)

%.o: %.c
	cc $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

Then introduce `CC` so we can swap compilers (`gcc`, `clang`) in one place:

```makefile
SRCS = mysrc1.c mysrc2.c
TARGET = myprog
OBJS = $(SRCS:.c=.o)
CC = gcc

$(TARGET): $(OBJS)
	$(CC) $(OBJS) -o $(TARGET)

%.o: %.c
	$(CC) $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

### The Stereotypical Makefile for C Programs (Lecture)

This is the final, full-featured Makefile from the lecture, including a static-analysis `check` target via Clang:

```makefile
SRCS = mysrc1.c mysrc2.c
TARGET = myprog
OBJS = $(SRCS:.c=.o)
CC = clang
CCFLAGS = -Wall
CHECKFLAGS = --analyze -Xanalyzer -analyzer-checker=core

$(TARGET): $(OBJS)
	$(CC) $(CCFLAGS) -o $(TARGET) $(OBJS)

%.o: %.c
	$(CC) $(CCFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

check:
	$(CC) $(CCFLAGS) $(CHECKFLAGS) $(SRCS)
```

Annotations (from lecture call-outs):
- **`CC = clang`** — pick the C compiler. If we want to use a different C compiler we can change this here.
- **`CCFLAGS = -Wall`** — compiler flags (arguments to the compiler). `-Wall` **shows all compiler warnings**. If we want to compile with different compiler flags we change this here.
- **`CHECKFLAGS = --analyze -Xanalyzer -analyzer-checker=core`** — **Runs static analysis** (automatic analysis of your source code) to detect issues such as **resource leaks due to missing `free` and `fclose`, double `free`, null pointer dereferences**.
- The **`check`** target runs the static analyzer on all sources without producing object files.

### Lecture Exit Ticket Questions (Make/Makefiles portion)

- "In your own words, please summarize three key insights you learned about C Programming and Makefiles today."
- "Why is GNU Make more useful for C/C++ than for Python and JavaScript? Please describe one use case that applies to C/C++ but not to Python & JS and one use case that still applies to Python & JS."
- "Please leave any questions that you have about today's material and things that are still unclear or confusing to you (if none, simply write N/A)."

---

## 7. Quick-Reference Cheat Sheet

### Rule Skeleton
```makefile
target: prerequisites
	recipe   # Must start with a TAB, not spaces!
```

### Automatic Variables
| Variable | Meaning |
|----------|---------|
| `$@` | The target name of the current rule |
| `$<` | The **first** prerequisite |
| `$^` | **All** prerequisites |
| `$*` | The **stem** matched by the `%` in a pattern rule |
| `$?` | All prerequisites **newer than** the target |

### Common Variables (Convention)
| Variable | Meaning |
|----------|---------|
| `CC` | The C compiler (e.g., `gcc`, `clang`, `cc`) |
| `CFLAGS` | C compiler flags (e.g., `-Wall -g -O2`) |
| `CCFLAGS` | (Lecture's spelling) flags passed to `CC` |
| `CHECKFLAGS` | Lecture-specific: clang static-analysis flags (`--analyze -Xanalyzer -analyzer-checker=core`) |
| `SRCS` | List of source files |
| `OBJS` | List of object files (often `$(SRCS:.c=.o)`) |
| `TARGET` | The name of the final executable |

### String Substitution Reference
```makefile
OBJS = $(SRCS:.c=.o)               # Replace .c with .o for each entry
$(VAR:search_pattern=replacement_text)
```

### Pattern Rule Template
```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

### Phony Targets
```makefile
.PHONY: all clean test install
```
Use for **verbs** (actions), not files. Common: `all`, `clean`, `test`, `install`, `run`, `dist`, `docs`.

### Order-Only Prerequisites
```makefile
target: normal_prereqs | order_only_prereqs
	recipe
```
Order-only prerequisites must exist before the recipe runs, but their newer mtime does NOT trigger a rebuild.

### Make's Decision Algorithm (the entire thing)
For each target visited:
1. Is the target phony? → **Always run** the recipe.
2. Does the target file not exist? → It's stale, rebuild it.
3. Is **any** prerequisite newer than the target? → Rebuild it.
4. Otherwise → Skip; up to date.

Only nodes **downstream** of the change in the dependency graph are rebuilt.

---

## 8. Practice / Self-Test Prompts

(From the `make.md` practice section.)

- **Basic — Automatic variable for target name**: What Automatic Variable represents the file name of the target of the rule? → `$@`
- **Advanced — Generic pattern rule**: Write a generic pattern rule to compile any `.c` file into a corresponding `.o` file, using automatic variables for the target name and the first prerequisite:
  ```makefile
  %.o: %.c
  	$(CC) $(CFLAGS) -c $< -o $@
  ```
- **Intermediate — Evaluate command**: Given the snippet `app: main.o network.o utils.o` followed by the command `$(CC) $(CFLAGS) $^ -o $@`, what exactly does the command evaluate to if `CC=gcc` and `CFLAGS=-Wall`? → `gcc -Wall main.o network.o utils.o -o app`
- **Indentation quiz**: What specific whitespace character MUST be used to indent the command/recipe lines in a Makefile rule?
  - A) Exactly two spaces
  - B) Exactly four spaces
  - C) A newline character followed by a colon
  - **D) A Tab character** (correct)

---

## 9. References

- **(Feldman 1979)**: Stuart I. Feldman (1979) "Make — a Program for Maintaining Computer Programs," *Software: Practice and Experience*, 9(4), pp. 255–265.
- GNU Make: https://www.gnu.org/software/make/
- Makefile Tutorial: https://makefiletutorial.com/
- Edsger Dijkstra (1968), "Go To Statement Considered Harmful," *CACM* — referenced by lecture, but applies to the C portion.


---

# Appendix: Lecture 13 Slides (raw extracted text)

Full L13 extracted text (covers BOTH C and Make).

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


---

# Appendix: Full root `make.md` reference (verbatim)

Motivation
Imagine you are building a small C program. It just has one file, main.c. To compile it, you simply open your terminal and type:

gcc main.c -o myapp

Easy enough, right?

Want to practice? Try the Interactive Makefile Tutorial — 10 hands-on exercises that build from basic rules to automatic variables and pattern rules, with real-time feedback.

Now, imagine your project grows. You add utils.c, math.c, and network.c. Your command grows too:

gcc main.c utils.c math.c network.c -o myapp

Still manageable. But what happens when you join a real-world software team? An operating system kernel or a large application might have thousands of source files. Typing them all out is impossible.

First Attempt: The Shell Script
To solve this, you might write a simple shell script (build.sh) that just compiles everything in the directory: gcc *.c -o myapp

This works, but it introduces a massive new problem: Time. Compiling a massive codebase from scratch can take minutes or even hours. If you fix a single typo in math.c, your shell script will blindly recompile all 9,999 other files that didn’t change. That is incredibly inefficient and will destroy your productivity as a developer.

The “Aha!” Moment: Incremental Builds
What you actually need is a smart tool that asks two questions before doing any work:

What exactly depends on what? (e.g., “The executable depends on the object files, and the object files depend on the C files and Header files”).
Has the source file been modified more recently than the compiled file?
If math.c was saved at 10:05 AM, but math.o (its compiled object file) was created at 9:00 AM, the tool knows math.c has changed and must be recompiled. If utils.c hasn’t been touched since yesterday, the tool completely skips recompiling it and just reuses the existing utils.o.

This is exactly why make was created by Stuart Feldman at Bell Labs in 1976 (Feldman 1979), and why it remains a staple of software engineering today. Modern development primarily relies on GNU Make, a powerful and widely-extended implementation that reads a configuration file called a Makefile.

So GNU make is the project’s engine that reads recipes from Makefiles to build complex products.

How It Works
Inside a Makefile, you define three main components:

Targets: What you want to build or the task you want to run.
Prerequisites: The files that must exist (or be updated) before the target can be built.
Commands: The exact terminal steps required to execute the target.
When you type make in your terminal, the tool analyzes the dependency graph and checks file modification timestamps. It then executes the bare minimum number of commands required to bring your program up to date.

The Dual Purpose
Makefiles are incredibly powerful—but their design can be confusing at first glance because they serve two distinct purposes:

Building Artifacts: Their primary, traditional use is for compiling languages (like C and C++), where they manage the complex process of turning source code into executable files.
Running Tasks: In modern development, they are frequently used with interpreted languages (like Python) as a convenient shortcut for common project tasks (e.g., make install, make test, make lint, make deploy).
Why We Need Makefiles
Ultimately, Makefiles are heavily relied upon because they:

Save massive amounts of time by enabling incremental builds (only recompiling the specific files that have changed).
Automate complex processes so developers don’t have to memorize long or tedious terminal commands.
Standardize workflows across teams by providing predictable, universal commands (like make test to run all tests or make clean to delete generated files).
Document dependencies, making it perfectly clear how all the individual pieces of a software system fit together.
The Cake Analogy
Think of Makefiles as a recipe book for baking a complex, multi-layered cake. Let’s make a spectacular three-tier chocolate cake with raspberry filling and buttercream frosting. A Makefile is your ultimate, highly-efficient kitchen manager and master recipe combined.

Here is how the concepts map together:

Concepts
1. The Targets (What you are making)
In a Makefile, a target is the file you want to generate.

The Final Target (The Executable): This is the fully assembled, frosted, and decorated cake ready for the display window.
Intermediate Targets (e.g., Object Files in C): These are the individual components that must be made before the final cake can be assembled. In this case, your intermediate targets are the baked chocolate layers, the raspberry filling, and the buttercream frosting. If we know how to bake each individual component and we know how to combine each of them together, we can bake the cake. Makefiles allow you to define the targets and the dependencies in a structured, isolated way that describes each component individually.
2. The Dependencies (What you need to make it)
Every target in a Makefile has dependencies—the things required to build it.

Raw Source Code (Source Files): These are your raw ingredients: flour, sugar, cocoa powder, eggs, butter, and fresh raspberries.
Chain of Dependencies: The Final Cake depends on the chocolate layers, filling, and frosting. The chocolate layers depend on flour, sugar, eggs, and cocoa powder.
Worked example of the Cake Recipe
Let’s build the Makefile for our cake recipe.

Iteration 1: The Basic Rule (The Blueprint)
The Need: We need to tell our kitchen manager (make) what our final goal is, what it requires, and how to put it together.

The Syntax: The most fundamental building block of a Makefile is a Rule. A rule has three parts:

Target: What you want to build (followed by a colon :).
Dependencies: What must exist before you can build it (separated by spaces).
Command: The actual terminal command to build it. CRITICAL: This line must start with a literal Tab character, not spaces.
# Step 1: The Basic Rule
cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking chocolate_layers, raspberry_filling, and buttercream to make the cake."
	touch cake
Note: If you run this now (i.e., ask the kitchen manager to bake the cake), make cake will complain: “No rule to make target ‘chocolate_layers’”. It knows it needs them, but it doesn’t know how to bake them.

Iteration 2: The Dependency Chain
The Need: We need to teach make how to create the missing intermediate ingredients so it can satisfy the requirements of the final cake.

The Syntax: We simply add more rules. The order of rules in the Makefile does not matter for execution — make reads all the rules, builds a dependency graph from them, and then traverses that graph from the goal target down to the leaves, building each prerequisite before the target that needs it. The first non-special rule in the file is used as the default goal if no target is given on the command line.

# Step 2: Adding the Chain
cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking layers, filling, and frosting to make the cake."
	touch cake

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Mixing ingredients and baking at 350 degrees."
	touch chocolate_layers

raspberry_filling: raspberries.txt sugar.txt
	echo "Simmering raspberries and sugar."
	touch raspberry_filling

buttercream: butter.txt powdered_sugar.txt
	echo "Whipping butter and sugar."
	touch buttercream
Now the kitchen works! But notice we hardcoded “350 degrees”. If we get a new convection oven that bakes at 325 degrees, we have to manually find and change that number in every single baking rule.

Iteration 3: Variables (Macros)
The Need: We want to define our kitchen settings in one place at the top of the file so they are easy to change later.

The Syntax: You define a variable with NAME = value and you use it by wrapping it in a dollar sign and parentheses: $(NAME).

# Step 3: Variables
OVEN_TEMP = 350
MIXER_SPEED = high

cake: chocolate_layers raspberry_filling buttercream
	echo "Stacking layers to make the cake."
	touch cake

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Baking at $(OVEN_TEMP) degrees."
	touch chocolate_layers

buttercream: butter.txt powdered_sugar.txt
	echo "Whipping at $(MIXER_SPEED) speed."
	touch buttercream
(I’ve omitted the filling rule here just to keep the example short, but you get the idea).

Iteration 4: Automatic Variables (The Shortcuts)
The Need: Look at the chocolate_layers rule. We list all the ingredients in the dependencies, but in a real C++ program, you also have to list all those exact same files again in the compiler command. Typing things twice causes typos.

The Syntax: Makefiles have built-in “Automatic Variables” that act as shortcuts:

$@ automatically means “The name of the current target”.
$^ automatically means “The names of ALL the dependencies”.
# Step 4: Automatic Variables
OVEN_TEMP = 350

cake: chocolate_layers raspberry_filling buttercream
	echo "Making $@" 
	touch $@

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Taking $^ and baking them at $(OVEN_TEMP) to make $@"
	touch $@
Now, the command echo "Taking $^ ..." will automatically print out: “Taking flour.txt sugar.txt eggs.txt cocoa.txt…”. If you add a new ingredient to the dependency list later, the command updates automatically!

Iteration 5: Phony Targets (.PHONY)
The Need: Sometimes we make a terrible mistake and just want to throw everything in the trash and start completely over. We want a command to wipe the kitchen clean.

The Syntax: We create a rule called clean that deletes files. However, what if you accidentally create a real text file named “clean” in your folder? make will look at the file, see it has no dependencies, and say “The file ‘clean’ is already up to date. I don’t need to do anything.”

To fix this, we use .PHONY. This tells make: “Hey, this isn’t a real file. It’s just a command name. Always run it when I ask.”

# Step 5: The Final, Complete Scaffolding
OVEN_TEMP = 350

cake: chocolate_layers raspberry_filling buttercream
	echo "Making $@" 
	touch $@

chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	echo "Taking $^ and baking them at $(OVEN_TEMP) to make $@"
	touch $@

# ... (other recipes) ...

.PHONY: clean
clean:
	echo "Throwing everything in the trash!"
	rm -f cake chocolate_layers raspberry_filling buttercream
By typing make clean in your terminal, the kitchen is reset. By typing make cake (or just make, as it defaults to the first rule), your fully automated bakery springs to life.

Now we get this complete Makefile:

# ---------------------------------------------------------
# Complete Makefile for a Three-Tier Chocolate Raspberry Cake
# ---------------------------------------------------------

# Variables (Kitchen settings)
OVEN_TEMP = 350
MIXER_SPEED = medium-high

# 1. The Final Target: The Cake
# Depends on the baked layers, filling, and frosting
cake: chocolate_layers raspberry_filling buttercream
	@echo "🎂 Assembling the final cake!"
	@echo "-> Stacking layers, spreading filling, and covering with frosting."
	@touch cake
	@echo "✨ Cake is ready for the display window! ✨"

# 2. Intermediate Target: Chocolate Layers
# Depends on raw ingredients (our source files)
chocolate_layers: flour.txt sugar.txt eggs.txt cocoa.txt
	@echo "🥣 Mixing flour, sugar, eggs, and cocoa..."
	@echo "🔥 Baking in the oven at $(OVEN_TEMP) for 30 minutes."
	@touch chocolate_layers
	@echo "✅ Chocolate layers are baked."

# 3. Intermediate Target: Raspberry Filling
raspberry_filling: raspberries.txt sugar.txt lemon_juice.txt
	@echo "🍓 Simmering raspberries, sugar, and lemon juice."
	@touch raspberry_filling
	@echo "✅ Raspberry filling is thick and ready."

# 4. Intermediate Target: Buttercream Frosting
buttercream: butter.txt powdered_sugar.txt vanilla.txt
	@echo "🧁 Whipping butter and sugar at $(MIXER_SPEED) speed."
	@touch buttercream
	@echo "✅ Buttercream frosting is fluffy."

# 5. Pattern Rule: "Shopping" for Raw Ingredients
# In a real codebase, these would already exist as your code files.
# Here, if an ingredient (.txt file) is missing, Make creates it.
%.txt:
	@echo "🛒 Buying ingredient: $@"
	@touch $@

# 6. Phony Target: Clean the kitchen
# Removes all generated files so you can bake from scratch
.PHONY: clean
clean:
	@echo "🧽 Cleaning up the kitchen..."
	@rm -f cake chocolate_layers raspberry_filling buttercream *.txt
	@echo "🧹 Kitchen is spotless!"

3. The Rules (The Recipe/Commands)
A rule in a Makefile pairs a target with its prerequisites and a recipe: the sequence of shell commands make runs to turn those prerequisites into the target. The recipe doesn’t have to call a compiler — it’s just shell commands, so make can drive any tool (linter, packager, doc generator, deployer).

Compiling: The rule to turn flour, sugar, and eggs into a chocolate layer is: “Mix ingredients in bowl A, pour into a 9-inch pan, and bake at 350°F for 30 minutes.”
Linking: The rule to turn the individual layers, filling, and frosting into the Final Cake is: “Stack layer, spread filling, stack layer, cover entirely with frosting.”
This can be visualized as a dependency graph:

Dependency graph: the final cake depends on chocolate layers, raspberry filling, and buttercream; chocolate layers depend on flour, sugar, and eggs; raspberry filling depends on raspberries and sugar; buttercream depends on butter and powdered sugar.

The Real Magic: Incremental Baking (Why we use Makefiles)
The true power of a Makefile isn’t just knowing how to bake the cake; it’s knowing what doesn’t need to be baked again. Make looks at the “timestamps” of your files to save time.

Imagine you are halfway through assembling your cake. You have your baked chocolate layers sitting on the counter, your buttercream whipped, and your raspberry filling ready. Suddenly, you realize someone mislabeled the sugar. It’s actually salt! Oh no! You need to remake everything that included sugar and everything that included these intermediate targets.

Without a Makefile: You would throw away everything. You would re-bake the chocolate layers, re-whip the buttercream, and remake the raspberry filling from scratch. This takes hours (like recompiling a massive codebase from scratch).
With a Makefile: The kitchen manager (make) looks at the counter. It sees that the buttercream is already finished and its raw ingredients haven’t changed. However, it sees your new packet of sugar (a source file was updated). The manager says: “Only remake the raspberry filling and the chocolate layers, and then reassemble the final cake. Leave the buttercream as is.”
If you look closely at the arrows of the dependency graph above and focus on the arrows leaving [sugar.txt], you can immediately see the brilliance of make:

The Split Path: The arrow from sugar.txt forks into two different directions: one goes to the Chocolate_Layers and the other goes to the Raspberry_Filling.
The Safe Zone: Notice there is absolutely no arrow connecting sugar.txt to the Buttercream (which uses powdered sugar instead).
The Chain Reaction: When make detects that sugar.txt has changed (because you fixed the salty sugar), it travels along those two specific arrows. It forces the Chocolate Layers and Raspberry filling to be remade. Those updates then trigger the double-lined arrows ══▶, forcing the Final Cake to be reassembled.
Because no arrow carried the “sugar update” to the Buttercream, the Buttercream is completely ignored during the rebuild!

See it in action: how make decides what to rebuild
The cake metaphor is helpful — but software engineers reason about files, timestamps, and the dependency graph. The five interactive demos below let you watch make make its decisions on a small C project. Each demo uses the same simple graph: app is built from main.o and util.o, which in turn come from main.c and util.c. Some demos add a shared header. Click the command to apply it; click again to undo. Multi-step demos have Back and Auto-play controls; you can also use ← → arrow keys when the demo has focus.

A reading guide for each diagram (these conventions are the same ones the interactive Makefile tutorial uses):

Solid green stripe + ✓ glyph — the file is up to date.
Diagonal-hatched red stripe + ● glyph (pulsing) — the target is stale; make would rebuild it.
Dashed border + ⌖ glyph — the target is phony (not a file). make always runs it.
Italic, no border — the file is a source. make never rebuilds these; you (or your editor) do.
Dashed edge — an order-only prerequisite. The arrow says “must exist before me”, not “rebuild me when newer.”
Demo 1 — What make checks
✓
up to date
●
stale (would rebuild)
⌖
phony target
_
source file
A small C project. app is the final executable; main.o/util.o are intermediate object files; main.c/util.c/shared.h are sources. Every target's mtime is greater than its prerequisites' — so all targets are up to date.

When you run make, it walks this graph from the top. For each target, it asks one simple question: is any of my prerequisites newer than me? If yes, rebuild this target. If no, skip it. Phony targets bypass the comparison entirely (they’re always considered “needs running”). That’s the entire algorithm.

Demo 2 — Touching a source file → cascade of staleness
You edit main.c. Watch the cascade of staleness, then how make rebuilds only what changed. The two-step rhythm is the entire developer feedback loop: edit → make, every time.

Step 0 of 2 (initial state).

←
Back

▶
touch main.c

▷
Auto-play
Graph
Makefile
✓
up to date
●
stale (would rebuild)
⌖
phony target
_
source file
Initial state. 6 nodes, 3 up to date, 3 source.
A common student misconception: “if anything changes, make recompiles everything.” That’s not how it works — only nodes downstream of the change in the dependency graph are rebuilt. The graph is the contract that lets make skip work safely.

Demo 3 — Phony targets always run
clean is phony (dashed border, crosshair glyph). make doesn't compare timestamps for phony targets — it always runs the recipe. Here the recipe is rm -f *.o app. After it runs, app, main.o, util.o no longer exist on disk; they show as stale (red hatched stripe) because there's no file to compare against. Sources main.c/util.c are never touched.


▶
make clean
Graph
Makefile
✓
up to date
●
stale (would rebuild)
⌖
phony target
_
source file
The contrast that makes this concept stick: a non-phony target with no prerequisites would be considered “up to date as long as the file exists.” The .PHONY declaration is what flips the switch. Common phony targets include clean, install, test, run, dist, docs. They’re verbs (actions) rather than nouns (files).

Demo 4 — Order-only prerequisites
An order-only prerequisite (after the | in the Makefile rule) tells make: "this must exist before I run, but don't trigger a rebuild just because it has a newer mtime." The classic use is a build directory whose mtime updates every time a file is written into it — without order-only, every .o would be rebuilt every time. Notice: the app → build edge is dashed. After we touch build, its mtime is the newest in the graph — but app stays up to date. With a normal edge, app would be marked stale.


▶
echo something > build/log
Graph
Makefile
✓
up to date
●
stale (would rebuild)
⌖
phony target
_
source file
Order-only is the answer to one of the most painful “why does my build keep redoing everything?” mysteries. It separates the two distinct ideas that students often conflate: “X must come before Y” vs. “X being newer means Y is out of date.” The first is ordering, the second is staleness propagation — and Makefiles let you choose.

Demo 5 — Putting it together: edit → build → clean → rebuild
The full developer rhythm. We start fresh, edit a header (which has a wider blast radius than a single .c), rebuild, clean, and rebuild from scratch. Each step shows which files are touched and which are skipped.

Step 0 of 4 (initial state).

←
Back

▶
touch shared.h

▷
Auto-play
Graph
Makefile
✓
up to date
●
stale (would rebuild)
⌖
phony target
_
source file
Initial state. 7 nodes, 3 up to date, 1 phony, 3 source.
If you can predict, before clicking, what each step will change in the graph — you have a working mental model of make. (Editor headers cascade widely, phony targets always run, missing targets are stale.) That mental model is the single biggest payoff of learning Make: it transfers directly to every other build tool you’ll meet later (Bazel, Gradle, Ninja, esbuild’s incremental mode), because they all reduce to “what’s stale, in topological order.”

A Recipe as a Makefile
If your cake recipe were written as a Makefile, it would look exactly like this:

Final_Cake: Chocolate_Layers Raspberry_Filling Buttercream Stack components and frost the outside.

Chocolate_Layers: Flour Sugar Eggs Cocoa Mix ingredients and bake at 350°F for 30 minutes.

Raspberry_Filling: Raspberries Sugar Lemon_Juice Simmer on the stove until thick.

Buttercream: Butter Powdered_Sugar Vanilla Whip in a stand mixer until fluffy.

Whenever you type make in your terminal, the system reads this recipe from the top down, checks what is already sitting in your “kitchen”, and only does the work absolutely necessary to give you a fresh cake.

Makefile Syntax
How Do Makefiles Work?
A Makefile is built around a simple logical structure consisting of Rules. A rule generally looks like this:

target: prerequisites
	command
Target: The file you want to generate (like an executable or an object file), or the name of an action to carry out (like clean).
Prerequisites (Dependencies): The files that are required to build the target.
Commands (Recipe): The shell commands that make executes to build the target. (Note: Commands MUST be indented with a Tab character, not spaces!)
When you run make, it looks at the target. If any of the prerequisites have a newer modification timestamp than the target, make executes the commands to update the target. The dependency relationships you declare matter immensely; for example, if you remove the object files ($(OBJS)) prerequisite from your main executable rule (e.g., $(TARGET): $(OBJS)), make will no longer trigger a re-link when the object files change, because the dependency relationship has been removed.

Syntax Basics
To write flexible and scalable Makefiles, you will use a few specific syntactic features:

Variables (Macros): Variables act as placeholders for command-line options, making the build rules cleaner and easier to modify. For example, you can define a variable for your compiler (CC = clang) and your compiler flags (CFLAGS = -Wall -g). When you want to use the variable, you wrap it in parentheses and a dollar sign: $(CC).
String Substitution: You can easily transform lists of files. For example, to generate a list of .o object files from a list of .c source files, you can use the syntax: OBJS = $(SRCS:.c=.o).
Automatic Variables: make provides special variables to make rules more concise.
$@ represents the target name.
$< represents the first prerequisite.
$^ represents all prerequisites.
Pattern Rules: Pattern rules serve as templates for creating many rules with the identical structure. For instance, %.o : %.c defines a generic rule for creating a .o (object) file from a corresponding .c (source) file.
A Worked Example
Let’s tie all of these concepts together into a stereotypical, robust Makefile for a C program.

# Variables
SRCS = mysrc1.c mysrc2.c
TARGET = myprog
OBJS = $(SRCS:.c=.o)
CC = clang
CFLAGS = -Wall

# Main Target Rule
$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $(TARGET) $(OBJS)

# Pattern Rule for Object Files
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

# Clean Target
clean:
	rm -f $(OBJS) $(TARGET)
Breaking it down:

Line 2-6: We define our variables. If we later want to use the gcc compiler instead, or add an optimization flag like -O3, we only need to change the CC or CFLAGS variables at the top of the file.
Line 9-10: This rule says: “To build myprog, I need mysrc1.o and mysrc2.o. To build it, run clang -Wall -o myprog mysrc1.o mysrc2.o.”
Line 13-14: This pattern rule explains how to turn a .c file into a .o file. It tells Make: “To compile any object file, use the compiler to compile the first prerequisite ($<, which is the .c file) and output it to the target name ($@, which is the .o file)”.
Line 17-18: The clean target is a convention used to remove all generated object files and the target executable, leaving only the original source files. You can execute it by running make clean.
Practice
Makefile Flashcards (Syntax Production/Recall)
Test your ability to produce the exact Makefile syntax, rules, and variables based on their functional descriptions.

Difficulty:
Basic
What Automatic Variable represents the file name of the target of the rule?

Show Answer
Makefile Flashcards (Example Generation)
Test your knowledge on solving common build automation problems using Makefile syntax and rules!

Difficulty:
Advanced
Write a generic pattern rule to compile any .c file into a corresponding .o file, using automatic variables for the target name and the first prerequisite.

Show Answer
C Program Makefile Flashcards
Test your ability to read and understand actual Makefile snippets commonly found in real-world C projects.

Difficulty:
Intermediate
Given the snippet app: main.o network.o utils.o followed by the command $(CC) $(CFLAGS) $^ -o $@, what exactly does the command evaluate to if CC=gcc and CFLAGS=-Wall?

Show Answer
Make and Makefiles Quiz
Test your understanding of Makefiles, including syntax rules, execution order, automatic variables, and underlying concepts like incremental compilation.

Difficulty:
Basic
What specific whitespace character MUST be used to indent the command/recipe lines in a Makefile rule?


A
Exactly two spaces


B
Exactly four spaces


C
A newline character followed by a colon


D
A Tab character

References
(Feldman 1979): Stuart I. Feldman (1979) “Make — a Program for Maintaining Computer Programs,” Software: Practice and Experience, 9(4), pp. 255–265.