---
name: c-format
description: Conventions and templates for writing clean, standard C. Use this skill WHENEVER the user asks to write, create, generate, scaffold, or refactor C code - a program, function, header, or module (.c/.h files) - even for small snippets or "just a quick C program." It encodes a consistent house style - section-organized files, snake_case naming, Allman braces, 4-space indentation, aligned declarations, per-function visibility banners, Doxygen comments (full contract at the definition in the `.c`, one-line brief on the header declaration), include guards and `extern "C"` C++ guards, ordered includes, const-correctness, return-code error handling, and copyright/license headers referencing a root LICENSE.md. Apply it to any task whose output is C source code, unless the user explicitly asks for a different style (e.g. K&R, tabs, camelCase) - in which case honor their request and use the rest of these conventions for everything they didn't override.
---

# C Style

This skill produces C source that is consistent, readable, and easy to maintain. **Readability and simplicity are the point** - every rule below exists to make the code easier for a human to scan and reason about, not to satisfy a checklist. When a rule and clarity ever pull in opposite directions, clarity wins. The secondary goal is consistency: any two files written under this style should look like they came from the same hand, so when in doubt, match what's already in the file or project.

## The style at a glance

- **Above all:** optimize for readability and simplicity; clarity beats any individual rule.
- **Naming:** `snake_case` for functions and variables, `UPPER_SNAKE_CASE` for macros/constants/enum values, `snake_case_t` (trailing `_t`) for typedef'd types. **File-scope identifiers (functions, types, macros, enum constants, globals) are prefixed with the file's base name** (or a short form) as a namespace - `example.c` -> `example_push`, `example_t`, `EXAMPLE_OK`. Locals, parameters, and struct members are *not* prefixed.
- **Braces:** Allman - every opening brace goes on its own line, aligned with the statement it opens.
- **Indentation:** 4 spaces, never tabs. One level per nesting depth.
- **Documentation:** Doxygen `/** ... */` at the top of every file and on every non-trivial function (full contract at the definition in the `.c`; a one-line `@brief` on the header prototype), plus short inline `/* ... */` comments marking the important steps inside function bodies.
- **Comment prose:** write comments and docs the way a person writes - plain words, short sentences, no filler. No em dashes (`—`) and no other typographic characters: a spaced hyphen, comma or full stop does the job, and the source stays 7-bit ASCII.
- **Alignment:** column-align the values of consecutive `#define`s, struct members, and consecutive variable declarations/assignments so related code lines up vertically.
- **Section banners:** every section in both `.c` and `.h` - Includes, Macros, Types, Constants, Global variables, prototypes, and Function implementations - opens with a boxed asterisk banner (a plain `/*` block comment, not `/**` - these are visual dividers, not Doxygen; the file header block stays `/**`). Just over 100 columns (the reference files use 103).
- **Function markers:** a full-width single-line `/*...*/` separator precedes *every* documented item - each definition in the `.c`, and each type and each prototype in the `.h` - **including the first item after a section banner**. All separators share the section banners' width (just over 100 cols; 103 in the reference files).
- **Structs & enums:** members/values column-aligned, a trailing comma on the last enum value, and a blank line before the closing brace.
- **Returns:** guard-clause early returns by default; a single return at the end when it removes duplication or pairs with `goto cleanup` - never dogmatic.
- **Headers:** include guard + an `extern "C"` C++ linkage guard around the declarations.
- **Library configuration:** a library checks its user's `lib_config.h` in its own header, right after including it, under a `Configuration checks` banner. Never in the configuration file, which the user owns and no update ever replaces. See [Checking a library's configuration](#checking-a-librarys-configuration).
- **Files:** a header block with `@file`/`@brief` (optionally `@version`) and a copyright/license notice referencing `LICENSE.md` in the project root. For a new project, ask the user for the copyright holder and license before writing.
- **Formatting:** a ready-to-use `.clang-format` ships with this skill - copy it to the project root to auto-apply the mechanical rules.
- **Standard:** target C11 unless the user says otherwise. Use `<stdbool.h>`, `<stdint.h>`, `size_t`, and other standard facilities.
- **MISRA:** on projects that declare conformance, [MISRA C:2012](#misra-c2012) applies on top of everything above, and **wins wherever it disagrees** with a rule here. The two that change the code's shape most: one `return` per function (`Rule 15.5`) and explicitly boolean conditions (`Rule 14.4`, so `if (p != NULL)` not `if (p)`). Every Required rule you break needs a recorded justification (by default a comment block in the suppressions file) and a marker at the site.

Read the rest of this file before writing. For anything non-trivial - a multi-function module, a header + source pair - consult `references/example.h` and `references/example.c`, which are complete, correct worked examples in exactly this style; mirror their structure.

## File organization

Every `.c` and `.h` file is divided into sections that always appear in the same order. Every section gets one of these boxed banners - Includes, Macros, Types, Constants, Global variables, prototypes, implementations, all of them. Omit a section only if it's empty; never reorder them. The asterisk rows run a little past 100 columns; keep that width identical for every section banner in a file:

```c
/*
 * ****************************************************************************************************
 * Section name
 * ****************************************************************************************************
*/
```

**Order for a `.c` file** (each gets its own section banner):
1. File header comment (Doxygen `@file`)
2. Includes
3. Macros / `#define`
4. Types (typedefs, structs, enums, unions)
5. Constants (`static const`, file-scope)
6. Global variables (file-scope)
7. Private function prototypes (the `static`, file-local functions)
8. Public function implementations
9. Private function implementations

Every section listed gets the boxed section banner, **including both implementation sections**. The public function definitions live under `Public function implementations` and the `static` ones under `Private function implementations` - two separate banners, matching the public/private split of the prototypes. Inside each, every definition is preceded by the single-line separator described under [Function banners and separators](#function-banners-and-separators) - the first definition included. So the functions area carries both kinds of marker: a section banner to open the group, and a separator before each definition within it. In headers, the `Types` and `Public function prototypes` sections likewise get their boxed banners, and every item within them (each type, each prototype, the first included) is preceded by the same separator.

**Prototype order and implementation order must always match.** The functions appear in the same sequence in the header's `Public function prototypes`, the `.c`'s `Public function implementations`, and - for `static` helpers - the `.c`'s `Private function prototypes` and `Private function implementations`. A reader who learns the order from the header can then scan the `.c` top-to-bottom and find everything where they expect it. When you add, remove, or reorder a function, update every one of those lists together so they never drift out of sync - check this before considering the work done.

**Order for a `.h` file:**
1. File header comment
2. Include guard open
3. Includes (only what the header itself needs)
4. Configuration checks (only in a library header that includes a user configuration file)
5. C++ linkage guard open (`extern "C"`)
6. Macros
7. Types
8. Public function prototypes
9. C++ linkage guard close
10. Include guard close

### A header-only library

When the whole library is one header of `static inline` functions, the header
is its own `.c` as well, and takes the `.c` file's shape after the header's
items 1 to 7:

1. Private function prototypes, a plain list as in a `.c`, and only for a
   helper that is called before its definition
2. Public function implementations, with the short Doxygen block
3. Private function implementations
4. C++ linkage guard close, include guard close

**No public prototypes.** Every definition is in the same file, before any
code that includes the header can call it, so a prototype is not needed, and
a commented one only says again what the definition above the function
already says. The README's API table is the overview a user reads.

Three things change once every function body lives in a header:

- **Every body is compiled in every file that includes it**, used or not. So
  anything an optional setting of an included vendor header can remove, such as
  a FreeRTOS call behind a `FreeRTOSConfig.h` switch, needs its own `#if`, or
  the library stops compiling in a project that switched it off. Test that with
  a build whose stub headers leave those calls out.
- **C++ code includes it too**, so the code has to be valid C++ as well: cast a
  `void *` explicitly, and use no designated initializers.
- **A function that is never called costs nothing.** GCC and Clang do not warn
  about an unused `static inline` function in a header, and do not emit it.

**One port header per platform**, when a setting picks between whole
implementations (osal does this for each RTOS). Each port holds the complete
code for one platform with no `#if` inside it, and the main header only checks
the setting and includes the one port that matches:

- The main header keeps what every port shares: the includes, the
  configuration checks, the common types and any shared helper. A helper only
  some ports call needs no `#if`, for the reason just above.
- It includes the port **last**, after its own `extern "C"` block has closed,
  since the port uses the types above it. The port opens its own `extern "C"`
  and includes its vendor header outside it.
- Every port starts with a check, so including it on its own, or a port the
  setting does not name, fails with a message instead of a wall of type errors:
  `#if !defined(OSAL_H) || (OSAL_RTOS != OSAL_RTOS_THREADX)` then `#error`.
  `-Wundef` stays quiet: GCC does not evaluate the right side of a true `||`.
- The tests build once per setting, so every port is compiled and tested.

### Include guards

Use `#ifndef`/`#define`/`#endif` guards (more portable than `#pragma once`). Name the macro from the project and file path, uppercased: `PROJECT_MODULE_H`.

### C++ linkage guard

Every header wraps its declarations in an `extern "C"` block so the header can be included from C++ translation units without name mangling. The guard goes *after* the includes (you don't want to wrap the includes) and *around* the macros, types, and prototypes. Putting these together, a header skeleton looks like:

```c
#ifndef GEOMETRY_SHAPES_H
#define GEOMETRY_SHAPES_H

#include <stddef.h>

#ifdef __cplusplus
extern "C"
{
#endif

/* ... macros, types, prototypes ... */

#ifdef __cplusplus
}
#endif

#endif /* GEOMETRY_SHAPES_H */
```

### Include ordering

Group includes in this order:
1. The matching header (in a `.c` file, e.g. `shapes.c` includes `"shapes.h"` first - this proves the header is self-contained).
2. C standard library headers (`<...>`), alphabetized.
3. Other system / third-party headers (`<...>`).
4. Project headers (`"..."`), alphabetized.

A blank line separates the groups:

```c
#include "shapes.h"

#include <math.h>
#include <stdbool.h>
#include <stdio.h>

#include "geometry/point.h"
#include "util/log.h"
```

**Consecutive groups of one include each are written as one block.** The blank
line is a separator, and between two lone includes there is nothing to separate.
Small modules are the common case in an embedded project, and the rule taken
literally gives them a blank line between every single line:

```c
/* Not this. */
#include "seq.h"

#include <stddef.h>

#include "main.h"

/* This. */
#include "seq.h"
#include <stddef.h>
#include "main.h"
```

The order is identical either way, and the order is what carries the meaning: a
reader who wants to know whether `main.h` is a project header reads the quotes,
not the whitespace. The blank lines exist only to help the eye over a long list,
so where there is no long list they are noise. Keep them as soon as any one of
the adjacent groups has two or more includes.

Set `SortIncludes: false` in `.clang-format`. The grouping is a judgement about
what belongs together, and a sorter that does not know the rule above will
flatten or re-split it.

### Quoted includes search their own folder first

`#include "x.h"` looks in the directory of the **including file** before it looks
at any `-I` path. `#include <x.h>` skips that step and searches only the
include paths. This is not a style preference, it is how the preprocessor works,
and it decides which file wins when two have the same name.

It matters most for a library that ships a default configuration header. If
`lib.h` sits beside a `lib_config.h` and does `#include "lib_config.h"`, that
neighbour is found first and **no include path ordering can override it**. So a
build that wants a different configuration, such as the library's own tests,
cannot get one by adding a folder to the include path.

Two ways out, and the first is usually better:

- **Let every build share the one configuration file.** Write the tests against
  the constants in it (`for (i = 0; i < LIB_MAX_ITEMS - 1; i++)`) rather than
  against a literal that only holds for one setting. The tests then exercise
  exactly what users compile.
- **Give the shipped file a different name** from the one the header includes,
  and have the installer or the user rename their copy. The cost is a second
  name for one idea, which readers have to learn.

When the tests have to cover more than one setting, such as each RTOS a library
supports, sharing one file is not enough and renaming is not wanted. Then a test
build can switch the shipped file off without touching it: define its include
guard on the command line, which makes the include expand to nothing, and give
the options with `-D` instead. ee24 builds its suite four times this way:

```cmake
target_compile_definitions(test_ee24_${name} PRIVATE
    EE24_CONFIG_H                   # the guard of ee24_config.h, so it is skipped
    EE24_RTOS=EE24_RTOS_${rtos})
```

Keep one build that uses the shipped file as it is, so the setting users get
first is still the one tested.

### Checking a library's configuration

A library that ships a configuration header checks it in its own public header,
directly after including it, under its own banner:

```c
#include "lib_config.h"

/*
 * ****************************************************************************************************
 * Configuration checks
 * ****************************************************************************************************
*/

/* Required from the first release: missing is an error that names the option. */
#ifndef LIB_MAX_ITEMS
#error "LIB_MAX_ITEMS is not defined. Add it to lib_config.h"
#elif LIB_MAX_ITEMS < 2U
#error "LIB_MAX_ITEMS must be at least 2"
#endif

/* Added in a later release: defaulted, because a configuration file installed
   before it existed will never define it. */
#ifndef LIB_TIMEOUT_MS
#define LIB_TIMEOUT_MS      100U
#elif LIB_TIMEOUT_MS == 0U
#error "LIB_TIMEOUT_MS must be greater than 0"
#endif
```

**Never in the configuration file itself**, not even at the bottom of it. That
file belongs to the user. It is copied once and never replaced, and the user
edits it, trims it, copies it from an old project, or merges several libraries'
settings into one. Anything in it is frozen at install time: a check added in a
later release never reaches anyone who installed earlier, and a check that gets
edited away is simply gone. Then the failure it guarded against happens without
a word. In the sequencer, `SEQ_MAX_TASKS 1U` compiles cleanly and makes every
`seq_task_add()` return `SEQ_ERR_FULL`, usually from an interrupt where nobody
reads the result. The user's file holds the user's choices. What the library
requires of them lives in files the library owns, which every update replaces.

**Not in the `.c` either**, even when only the `.c` uses the value. A default
has to be visible to every file that includes the header the moment an option
sizes a member of a public struct, and one fixed place in every library is
easier to find than a rule about which place applies. This is also how FreeRTOS
does it: `FreeRTOS.h` includes the user's `FreeRTOSConfig.h`, then fills in
defaults and stops the build on anything missing, so it is where an STM32
developer already expects to look.

**Existence first, then range, chained with `#elif`.** An undefined name inside
`#if` quietly counts as `0`, so a lone range check reports a missing option as a
value that is too small, which sends the reader looking for the wrong mistake.
Checking existence first fixes the message, and chaining the range check with
`#elif` means it only ever sees a defined value. Written as two separate blocks
instead, `#error` does not stop the preprocessor, so a missing option produces
both errors and a `-Wundef` warning for one mistake. The chain is also what
MISRA C:2012 `Rule 20.9` requires: every identifier in an `#if` is defined
before it is evaluated.

**Required or defaulted.** An option that exists from the first release and has
no sensible default, a buffer size or a pin, is required. Prefer that where a
default would also work, because a default swallows a typo: `LIB_MAX_ITEM 32U`
leaves the size at its default without a word. An option added in any later
release must be defaulted, or the update breaks every existing build. Add it to
the shipped configuration file too, so new installs see it and can change it.

**Named choices start at 1, never 0.** An option picked from a list of names,
`#define LIB_RTOS LIB_RTOS_THREADX`, is checked against those names. A misspelt
one, `LIB_RTOS_FREERTOS` when no such name exists, also counts as `0` inside
`#if`. If `0` is a valid choice, usually "none", the typo quietly selects it and
the build succeeds with the wrong setting. Numbered from 1, the typo matches
nothing and the "must be one of" check refuses it:

```c
#define LIB_RTOS_NONE       1
#define LIB_RTOS_CMSIS_V2   2
#define LIB_RTOS_THREADX    3

#ifndef LIB_RTOS
#error "LIB_RTOS is not defined. Add it to lib_config.h"
#elif (LIB_RTOS != LIB_RTOS_NONE) && (LIB_RTOS != LIB_RTOS_CMSIS_V2) && \
      (LIB_RTOS != LIB_RTOS_THREADX)
#error "LIB_RTOS must be one of the LIB_RTOS_ values listed in lib_config.h"
#endif
```

The names are defined in the header, before the check. The configuration file
only stores the token, which is not expanded until the check reads it.

The configuration file still says each option's valid range beside its
`#define`, for the person editing it. That comment is the documentation; the
check in the header is what enforces it.

### A driver for a bus peripheral: one shape for every library

Every library that moves data over SPI, I2C, a UART or any other bus takes the
same shape, so an application drives all of them the same way from one state
machine or from seq, and a reader who knows one knows them all. spif and ee24
are the models to copy.

**Two modes, one setting.** `<LIB>_TRANSFER` in the configuration file, with
`<LIB>_TRANSFER_POLLING` as 1 and `<LIB>_TRANSFER_DMA` as 2 (named choices
start at 1, above). Only the code for the chosen mode is compiled in.

**Every call that can take long is a job, in both modes.** The call checks its
arguments, takes the mutex, does the first step and returns at once with
`<LIB>_ERR_NONE` ("started"), or with the error that stopped it. Nothing that
moves data or waits for the chip blocks for longer than one step:

- A write sends one page, then the chip stores it by itself.
- An erase sends its command, then the chip erases by itself, for milliseconds
  or minutes.
- A read by DMA starts the DMA. A read by polling moves one piece of at most
  `<LIB>_POLL_CHUNK` bytes, so the CPU moving every byte never keeps the
  caller longer than one piece takes. A read that fits in one piece is done
  when the call returns.

The chip's own time is the long part, and DMA does not shorten it: 1 KB to a
24C256 is about 26 ms on the bus and 150 ms of page stores, and a 16 MB flash
erase is up to 200 s. So the job model holds without DMA too. Only `init` and
calls of a few bytes, a unique ID, may block.

**Two calls follow a job, with the same names everywhere:**

- `bool <lib>_is_busy(<lib>_t *handle)` moves the job one step and returns at
  once: it reads the next piece by polling, or asks the chip or the HAL once
  whether the piece under way is done and starts the next. It never sleeps.
  `true` while the job runs.
- `<lib>_err_t <lib>_wait(<lib>_t *handle)` loops on `is_busy()` and returns
  the job's result. It sleeps `osal_delay_ms(1)` between questions while the
  chip or the DMA has the job, and none between the pieces of a read by
  polling, which would only slow it down.

A call refused at once stores its result too, so `start; wait` is all simple
code needs. Any other call on the handle during a job returns
`<LIB>_ERR_BUSY` with no traffic. The mutex is held from the call that starts
a job until it ends, so `is_busy()` and `wait()` come from the thread that
started it. The handle carries the job: `job_state`, `job_result`,
`job_data`, `job_len`, `job_done`, `job_address`, `job_step`, `job_start`,
`job_timeout`, and whatever else the device needs.

**What only DMA needs is under `#if (<LIB>_TRANSFER == <LIB>_TRANSFER_DMA)`**:
the helpers that start and follow a DMA transfer, the job states that exist
only with DMA (a page on its way out), and any check that can only be true with
DMA. What only polling needs, the piece reader, sits in the `#else`. A field or
helper both modes use stays unguarded. The polling object must reference no
DMA function of the HAL at all; check it with `nm -u`.

**Times live in the configuration file, never in an argument.** No function
takes a `timeout_ms`. There is one setting per step: the mutex
(`<LIB>_TIMEOUT_MUTEX_MS`, `HAL_MAX_DELAY` as shipped), the bus transfer
(scaled by size when a transfer can be long, as ee24 gives every 256 bytes a
share), and each of the chip's own times, each the datasheet's longest plus
half again. `HAL_MAX_DELAY` waits for ever and is never multiplied, so it
cannot wrap round to a short time.

**A stuck DMA transfer** ends the job with `<LIB>_ERR_TIMEOUT` and leaves the
peripheral usable again: `HAL_SPI_Abort()` for SPI. The HAL cannot abort an
I2C memory transfer, so I2C is set up again with `HAL_I2C_DeInit()` and
`HAL_I2C_Init()`, which runs CubeMX's MspInit and its DMA setup.

**A setting added after a library's first version gets a default** in the
header, as above, so the configuration file a user already has, whose `USER
CODE` part the installer keeps, still builds after an update.

**The tests build once per mode**, plus once with every time `HAL_MAX_DELAY`,
and once more for any setting that changes which code runs, such as a piece
size big enough to show a time scaled by size. The fake HAL ends a DMA
transfer once the test clock passes its end, a `tearDown()` checks that no
chip is left selected, no DMA left running and no mutex left held, and a test
follows a job by hand with `is_busy()` as well as with `wait()`.

## Naming

| Kind | Convention | Example |
|------|-----------|---------|
| Function | `snake_case`, verb-led | `compute_area`, `list_append` |
| Variable | `snake_case`, descriptive | `total_count`, `head_node` |
| File-scope function/type/global | `<file>_` prefix (see below) | `example_push`, `example_t` |
| Macro / compile-time constant | `UPPER_SNAKE_CASE`, `<FILE>_` prefixed | `EXAMPLE_MAX_SIZE` |
| Enum value | `UPPER_SNAKE_CASE`, `<FILE>_` prefixed | `EXAMPLE_OK`, `EXAMPLE_EMPTY` |
| Typedef'd type | `snake_case_t` | `point_t`, `shape_t` |
| Struct/enum/union tag | `snake_case` (when not typedef'd) | `struct list_node` |
| User-overridable callback (weak default) | `_cb` suffix | `os_tickless_pre_sleep_cb` |
| Arch/port-layer function (incl. its callbacks) | `os_arch_` prefix | `os_arch_spinlock_acquire`, `os_arch_core_id_get_cb` |
| Storage a DEFINE macro declares for you | `<name>_<kind>_buf` | `worker_stack_buf`, `sensor_q_queue_buf`, `cmd_msg_buf` |

Avoid single-letter names except conventional loop counters (`i`, `j`, `k`) and obvious math (`x`, `y`). Don't encode types into names (no Hungarian notation). Pick names that make the code read as prose: `if (is_empty(list))`, not `if (chk(l))`.

Macro-declared storage: a `DEFINE` macro that declares an object *and* the storage behind it names that storage **`<name>_<kind>_buf`** - `OS_TASK_DEFINE(worker, 512U)` declares `worker` and `worker_stack_buf`; `OS_QUEUE_DEFINE(sensor_q, ...)` declares `sensor_q_queue_buf`. Lower case because it *is* a variable, and the table above keeps upper case for macros and enum constants - a shouting name would read as one. The `<kind>` is what earns its place: without it two objects generate the same suffix, which is what happened while queues and message buffers both produced `_BUFFER`. Storage that is not an array - a descriptor struct, say - takes the same shape without `_buf`, as `worker_task_storage` does. Say so in the macro's own comment too, since the name never appears in the caller's source.

Pointer placement: a function that returns a pointer attaches the `*` to the return type - `void* os_mem_alloc(size_t size)`, `os_list_node_t* os_list_pop_front(os_list_t *list)` - while parameters and variable declarations keep the `*` with the name (`os_task_t *task`).

Qualifiers and attributes: use `__IO` (CMSIS meaning; guarded fallback define in `os_arch_port_common.h`) instead of a bare `volatile`, and `OS_WEAK` instead of `__attribute__((weak))`. Write qualifiers before the type (`static __IO uint32_t count`), never in the middle of a declarator: when the *pointer object* itself is the shared/volatile thing, introduce a typedef so the qualifier still reads first (`typedef os_timer_t *os_timer_slot_t; static __IO os_timer_slot_t registry[N];`) - note that `os_timer_t __IO *p` would qualify the pointed-to data instead, which is a different meaning.

### Sharing data with an interrupt

`__IO` and a memory barrier make a write *visible* in the right order. Neither
makes a sequence of operations *indivisible*. Any read, modify, publish sequence
on state an interrupt also touches has to be closed off, or an interrupt landing
in the middle of it will use the same stale value and one of the two updates
disappears with nothing reported.

```c
/* Wrong. Two interrupts at different priorities both read the same head, both
   write the same slot, and one of the queued items is lost. */
uint32_t head = queue.head;
queue.fn[head] = item;
__DMB();
queue.head = (head + 1U) % QUEUE_SIZE;
```

Close the window by saving PRIMASK, disabling interrupts, and restoring what was
there. Restoring rather than simply re-enabling is what keeps the function safe
to call from code that had already disabled them:

```c
uint32_t primask = __get_PRIMASK();
__disable_irq();

/* ... claim the slot and publish it ... */

__set_PRIMASK(primask);
```

Keep the section down to the few instructions that must not be split. It costs a
handful of cycles and it is the difference between a queue that is genuinely
interrupt safe and one that only looks it.

This class of bug will not show up in ordinary unit tests, because a sequential
test never interleaves. Test it by making the interleaving deliberate: call a
hook from the vulnerable point that a real build compiles to nothing.

```c
#ifndef LIB_TEST_HOOK
#define LIB_TEST_HOOK() do { } while (0)
#endif
```

### File-name prefix for file-scope identifiers

Give a file its own namespace: **every file-scope identifier starts with the file's base name, or a short form of it.** This applies to the things visible outside their own little scope - functions, typedef'd types, macros, enum constants, and file-scope (global/`static`) variables. So in `widget.c` / `widget.h` you get `widget_create()`, `widget_t`, `WIDGET_MAX_SIZE`, `WIDGET_STATE_IDLE`. The reference files live in `example.c` / `example.h`, so everything there is prefixed `example_` / `EXAMPLE_`. The benefit is a poor man's namespace: any symbol's home file is obvious, and collisions across translation units disappear.

- **Use a short form when the file name is long.** `network_controller.c` -> a `netctl_` or `nc_` prefix is fine; pick one and use it for every identifier in the file.
- **Functions:** `<prefix>_verb` - `example_push`, `example_init`.
- **Types:** `<prefix>_t` (or `<prefix>_<thing>_t`) - `example_t`, `example_status_t`.
- **Macros / enum constants:** the prefix uppercased - `EXAMPLE_GROWTH_FACTOR`, `EXAMPLE_OK`.
- **File-scope variables:** `<prefix>_name` (a leading `g_` for globals is also fine if the project uses it).

**What is *not* prefixed:** anything already scoped - local variables, function parameters, and struct/union members. `stack->count`, `int value`, `size_t i` stay plain; prefixing them would add noise and hurt readability, which defeats the purpose. The prefix is a namespace for file-level symbols, not a decoration for every name.

## Formatting

### Braces (Allman)

The opening brace sits on its own line, at the same indentation as the line that introduces it. This applies to functions, `if`/`else`, loops, `switch`, and bare blocks.

```c
int main(void)
{
    if (argc < 2)
    {
        fprintf(stderr, "usage: prog <file>\n");
        return EXIT_FAILURE;
    }
    else
    {
        process(argv[1]);
    }

    return EXIT_SUCCESS;
}
```

Always brace the body, even one-liners - it prevents a class of bugs when someone later adds a second statement.

### Spacing

- One space after keywords before the paren: `if (`, `while (`, `for (`, `switch (`.
- No space between a function name and its `(`: `compute_area(r)`.
- Spaces around binary operators and after commas: `a + b`, `f(x, y)`.
- No space around unary operators: `*ptr`, `&val`, `i++`, `!flag`.
- Pointer asterisk binds to the variable, not the type: `int *p;`, `char *name;`, `const char *s`.

### Layout

- 4-space indentation, no tabs.
- Aim for <= 80 columns; hard cap 100. Wrap long argument lists or conditions, indented one level.
- One statement and one declaration per line.
- Initialize variables at declaration where practical; declare them at the narrowest useful scope (C99+), not all at the top of the function.
- Separate logical blocks within a function with a single blank line. Never two consecutive blank lines.

### Alignment

Within a contiguous group of related lines, align the columns so the structure reads vertically at a glance. Use spaces (never tabs) to pad. Realign the whole group whenever the longest member changes; if alignment ever makes a group awkward to maintain, a single space is an acceptable fallback.

Align the **values of consecutive `#define`s** (and their trailing comments, if any):

```c
#define MAX_CLIENTS     64
#define BUFFER_SIZE     4096
#define RETRY_LIMIT     3
```

Align **struct/union members** - type column, then name column - and any trailing member comments:

```c
typedef struct
{
    int    *items;   /**< Backing storage, owned by the list. */
    size_t count;    /**< Items currently stored.             */
    size_t capacity; /**< Allocated slots.                    */
} int_list_t;
```

**The asterisk belongs to the name, so it goes in the name column**, not in the
space before it. `*items` starts where `count` and `capacity` start. Getting this
backwards lines the letters up and leaves the asterisk sitting in nobody's
column, which is the version that looks wrong without it being obvious why:

```c
/* Not this: the identifiers align, the asterisk hangs one column back. */
    seq_state_fn_t next_fn;
    void          *user;
    uint32_t       time;

/* This: every declarator starts in the same column, asterisk included. */
    seq_state_fn_t next_fn;
    void           *user;
    uint32_t       time;
```

The same goes for a block of local declarations. Note that clang-format's
`PointerAlignment: Right` does the first of those, so it will undo this. See
[Code formatting](#code-formatting-clang-format) for the full list of what the
tool cannot keep.

Align **consecutive variable declarations and assignments** that form a block, on the `=`:

```c
static int    g_request_count = 0;
static double g_total_latency = 0.0;
static bool   g_initialized   = false;

long   sum    = 0;
size_t count  = 0;
int    status = sum_integers(input, &sum, &count);
```

Alignment applies per group: a blank line or an unrelated statement ends the group, and the next group aligns to its own widest member.

## Comments and documentation

Comments explain **why**, not **what** - the code already says what it does. A comment that restates the code is noise; delete it.

```c
/* BAD: restates the obvious */
i++;  /* increment i */

/* GOOD: explains intent the code can't */
i++;  /* skip the sentinel terminator written by the parser */
```

Use `/* ... */` for block comments. Inline `//` is acceptable for short trailing notes if the project uses it, but prefer `/* */` for consistency in C.

### Writing style

Comments and documentation are prose, so write them the way a person writes: plain words, short sentences, no filler. Say the thing and stop. If a sentence needs three clauses to land, make it two sentences.

Do not use the em dash (`—`). A spaced hyphen (` - `), a comma, a colon or a full stop covers every job it does:

```c
/* BAD: em dash, and non-ASCII in a source file */
/* Checked before the copy — a torn item is worse than a refused send. */

/* GOOD */
/* Checked before the copy: a torn item is worse than a refused send. */
```

Two reasons, and the second is the one that bites. It reads as machine-written, because that is mostly what produces it. And it is not ASCII: everything else in these files is 7-bit, so one stray multi-byte character in a comment is enough to upset an editor with the wrong encoding, a terminal, a patch tool or a compiler that is not expecting UTF-8. Keep comments to characters a hex dump would not surprise anyone with.

The same goes for the other typographic characters a word processor likes to insert: curly quotes, ellipsis characters, non-breaking spaces. Write `"`, `...` and a plain space.

### Inline comments inside functions

Inside a function body, mark the **important steps** with a short `/* ... */` comment on the line above the code it describes. The aim is that someone skimming the function can follow its shape from the comments alone. Comment the parts that carry intent a reader can't infer at a glance - why an order matters, what an edge case means, the reasoning behind a non-obvious calculation - and leave the self-evident lines uncommented so the meaningful comments stand out.

```c
/* Make room before writing past the end of the buffer. */
if (stack->count == stack->capacity)
{
    example_status_t status = example_grow(stack);
    if (status != EXAMPLE_OK)
    {
        return status;
    }
}
```

This is the same "explain why, not what" rule applied step by step: a handful of well-placed lines beat a comment on every statement, which just adds noise and hides the ones that matter.

**The reasoning behind a step goes at the step, not in the block above the function.** The Doxygen block is short: the `@brief`, one line per `@param`, and the `@return`. A warning the caller must not miss fits in the brief itself ("Not from an interrupt."). How it works inside, why pages are written one at a time or why the mutex is created last, belongs on the line that does it. Written in the block instead, the explanation is far from the code it explains, it drifts when the code changes, and the body is left bare, so someone reading the function has nothing to follow. ee24 was written that way first and had to be reworked, twice: paragraphs above each function and almost nothing inside, then paragraphs still left over after the first pass. What the user of the library needs beyond that goes in the README.

```c
/*****************************************************************************************************/
/**
 * @brief Write len bytes starting at address. Not from an interrupt.
 *
 * @param[in,out] handle      Handle from ee24_init().
 * @param[in]     address     First byte to write.
 * @param[in]     data        The bytes to write.
 * @param[in]     len         Bytes to write, up to the end of the chip.
 * @param[in]     timeout_ms  Time for the whole call. HAL_MAX_DELAY waits for ever.
 * @return EE24_ERR_NONE, or the EE24_ERR_ value that says what went wrong.
 */
```

### Doxygen for functions and files

Every file - `.c` and `.h` alike - opens with a header block carrying the file's `@brief` and a copyright/license notice. A `@version` line is optional; include it when the project tracks file or module versions:

```c
/**
 * @file shapes.c
 * @brief Geometric shape creation and area calculation.
 * @version 1.0.0
 *
 * @copyright (c) 2026 <COPYRIGHT HOLDER>
 *            SPDX-License-Identifier: MIT
 *            See LICENSE.md in the project root for the full license text.
 */
```

Keep the `@file`/`@brief` lines short and factual. Use the project's copyright holder, the current year, and an [SPDX identifier](https://spdx.org/licenses/) matching the project's license (`MIT`, `Apache-2.0`, `GPL-3.0-only`, `Proprietary`, ...). See [License and copyright](#license-and-copyright) below for how to choose these and when to write a `LICENSE.md`.

Every public function (and every static function whose purpose isn't obvious) gets a full Doxygen block immediately above its **definition** in the `.c`; the matching header prototype carries only the one-line `@brief`:

```c
/**
 * @brief Compute the area of a shape.
 *
 * @param[in] shape  The shape to measure. Must not be NULL.
 * @return The area in square units, or a negative value for an unknown kind.
 */
double shape_area(const shape_t *shape)
```

Conventions for the block:
- `@brief` is a single line, imperative or noun phrase, ending with a period.
- Leave a blank comment line after the `@brief`. No description paragraph as a
  rule: a warning fits in the brief, the reasoning goes at the step inside the
  function (see [Inline comments inside functions](#inline-comments-inside-functions)),
  and what a user needs beyond that goes in the README.
- One `@param` per parameter, in order. **Every `@param` carries a direction specifier** - `@param[in]` for inputs (including by-value params and `const` pointers), `@param[out]` for pure outputs, `@param[in,out]` for parameters that are both read and modified. Align the param names within a block when there's more than one.
- `@return` describes the value *and its error sentinels*. Omit it only for `void` functions.
- Use `@note`, `@warning`, `@see` sparingly when they add real information.

**Where the documentation lives.** Put the *full* contract - `@brief`, every `@param`, the `@return` - at the function's **definition in the `.c`**, under its banner. The definition is where the code is edited, so keeping the contract next to the implementation keeps the two from drifting apart. The **declaration** carries only a one-line `@brief` so a reader scanning the interface gets oriented without duplicating the whole contract:
- **Public functions** - brief `@brief` on the prototype in the header; full block on the definition in the `.c`.
- **Static (file-local) functions** - **nothing** on the private prototype, no comment and no separator; full block on the definition in the `.c`.

The header is the interface a reader scans without opening the `.c`, so its prototypes earn a one-line brief. A private prototype sits in the same file as its definition, which already carries the brief and the full contract, so a comment on it only says the same thing twice. The `Private function prototypes` section is therefore a plain list, aligned on the function names like any group of declarations, with any `#if` guards kept around the lines they belong to:

```c
/*
 * ****************************************************************************************************
 * Private function prototypes
 * ****************************************************************************************************
*/

static uint32_t      ee24_size_bytes(uint16_t size_kbit);
static ee24_err_t    ee24_check(const ee24_t *handle, uint32_t address, const void *data,
                                size_t len);
static ee24_target_t ee24_target(const ee24_t *handle, uint32_t address);
static void          ee24_sleep(void);
#if EE24_RTOS != EE24_RTOS_NONE
static bool          ee24_kernel_running(void);
#endif
```

The rule reduces to one principle: **the full contract lives at the definition, and a one-line brief only on the header prototype.** Since every definition lives in the `.c`, this means the `.c` carries the longer comments and the `.h` the shorter ones. A **standalone program** (a lone `.c` with no header) has no separate declaration, so its full blocks simply live on the definitions.

Be consistent within a project. If a team prefers the full docs in the `.c` instead, that's a valid choice - just apply it everywhere rather than mixing.

### Function banners and separators

There are two related markers, one for each file kind.

**In `.c` files - a separator before every function definition.** Precede **every** function definition - public and static alike, **including the first one under a section banner** - with a full-width line of asterisks (`/*`, a run of `*`, then `*/`) **the same width as the section banners - just over 100 columns (103 in the reference files)**. It makes function boundaries jump out when scanning a long file. The function's full Doxygen block goes directly beneath the separator, then the definition:

```c
/**********************************************************************************************************/
/**
 * @brief Compute the area of a shape.
 *
 * @param[in] shape  The shape to measure. Must not be NULL.
 * @return The area in square units, or a negative value on error.
 */
double shape_area(const shape_t *shape)
{
    /* ... */
}
```

**In `.h` files - a separator before every prototype and every type.** Under the `Public function prototypes` section banner, precede **every** prototype - **including the first** - with a full-width asterisk separator line, the same width as everything else. The same separator also precedes **each item in the `Types` section** (each typedef, struct, enum, or union). Because the full contract now lives at the definition, the header prototype carries only its one-line `@brief` under the separator:

```c
/*****************************************************************************************************/
/**
 * @brief Initialize an empty stack.
 */
example_status_t example_init(example_t *stack);

/*****************************************************************************************************/
/**
 * @brief Push a value onto the stack.
 */
example_status_t example_push(example_t *stack, int value);
```

Rules:
- **One width throughout the file**, just over 100 columns (103 in the reference files), shared by every marker: the boxed section banner's rows and every single-line separator (before types, prototypes, and definitions). They read as one consistent divider; the boxed section banner stands apart by opening with `/*`, spanning multiple lines, and carrying a label.
- A separator precedes **every** item, the first one in a section included - the section banner announces the group, the separators divide the items within it (so a section banner is followed by a blank line, then the first item's separator). The one exception is a `.c` file's `Private function prototypes`, a plain list with no separators, as described under [Where the documentation lives](#doxygen-for-functions-and-files).
- Both implementation sections (`Public function implementations`, `Private function implementations`) open with the boxed section banner; each definition inside then gets its own separator.
- One blank line sits between the end of one item and the next item's separator.

## Declarations, types, and constants

- Prefer `enum` or `static const` for typed constants; reserve `#define` for compile-time switches, header guards, and conditional compilation.
- Use exact-width types (`int32_t`, `uint8_t`) when size matters, `size_t` for sizes and indices, `bool` (from `<stdbool.h>`) for truth values.
- Apply `const` to any pointer parameter whose pointee the function won't modify, and to local data that never changes. Const-correctness documents intent and catches mistakes.
- Define `main` as `int main(void)` or `int main(int argc, char *argv[])`, and return `EXIT_SUCCESS` / `EXIT_FAILURE` from `<stdlib.h>`.
- Avoid magic numbers; name them as constants or enum values.

### Struct and enum layout

For `struct`, `union`, and `enum` definitions, give the body room to breathe: leave **one blank line after the final member/value, before the closing brace**, and put a **trailing comma on the last enum value**. The blank line visually separates the contents from the `} name;` line, and the trailing comma means adding a member later is a one-line diff that never touches the previous line. Members and their trailing comments stay column-aligned (see [Alignment](#alignment)).

**Aligning members and enum values:**
- In a struct/union, align the member names into a column so they all start at the same place. For a pointer member, the `*` occupies that shared column - so `*name` begins exactly where the non-pointer names begin, and the pointer's identifier sits one place further right. This keeps every declarator (the `*` for pointers, the first letter for non-pointers) flush in one column. Trailing comments align too.
- In an enum, if you give explicit values to the members, give them to **all** of the members and **align the `=`** into a column. (If you don't need explicit values, leave them off entirely - don't number only some.)

```c
typedef enum
{
    EXAMPLE_OK           = 0, /**< Operation succeeded.          */
    EXAMPLE_NO_MEMORY    = 1, /**< Allocation failed.            */
    EXAMPLE_EMPTY        = 2, /**< Pop/peek on an empty stack.   */
    EXAMPLE_BAD_ARGUMENT = 3, /**< A required argument was NULL. */

} example_status_t;

typedef struct
{
    int    *items;   /**< Backing storage, owned by the struct. */
    size_t count;    /**< Items currently stored.               */
    size_t capacity; /**< Allocated slots.                      */

} example_t;
```

## Error handling

Functions report failure through a return value and deliver real results through output pointers, or return a pointer that is `NULL` on failure. Pick one convention per module and stick to it. Two common shapes:

```c
/* Status code out, result via pointer */
status_t parse_config(const char *path, config_t *out_config);

/* Pointer result, NULL on failure */
node_t *list_find(const list_t *list, int key);
```

Always check return values from allocation and I/O. For functions that acquire several resources, the single-exit `goto cleanup` idiom is idiomatic, standard C - it keeps cleanup in one place and avoids leaks on early exit:

```c
status_t load_data(const char *path, data_t *out)
{
    status_t status = STATUS_OK;
    FILE *file = NULL;
    char *buffer = NULL;

    file = fopen(path, "rb");
    if (file == NULL)
    {
        status = STATUS_IO_ERROR;
        goto cleanup;
    }

    buffer = malloc(BUFFER_SIZE);
    if (buffer == NULL)
    {
        status = STATUS_NO_MEMORY;
        goto cleanup;
    }

    /* ... use file and buffer ... */

cleanup:
    free(buffer);
    if (file != NULL)
    {
        fclose(file);
    }
    return status;
}
```

### Where to put the return

**Single exit. One `return` at the end of every function.** This is MISRA C:2012
Rule 15.5, and on a MISRA project it is not negotiable - see
[MISRA C:2012](#misra-c2012) below.

> **This rule changed.** Earlier revisions of this guide made guard-clause early
> returns the default and told you not to be dogmatic about single-exit. Under
> MISRA that guidance is withdrawn: Rule 15.5 wants exactly one exit point, so
> the shapes below replace it. Existing code written the old way is converted,
> not grandfathered.

The mechanical translation is a result variable initialised to the failure case,
guarded assignments, and one `return` at the bottom:

```c
/* BEFORE - guard-clause early returns */
example_status_t example_push(example_t *stack, int value)
{
    if (stack == NULL)
    {
        return EXAMPLE_BAD_ARGUMENT;
    }

    if (stack->count == stack->capacity)
    {
        return EXAMPLE_FULL;
    }

    stack->items[stack->count] = value;
    stack->count++;

    return EXAMPLE_OK;
}

/* AFTER - single exit */
example_status_t example_push(example_t *stack, int value)
{
    example_status_t status = EXAMPLE_BAD_ARGUMENT;

    if (stack != NULL)
    {
        if (stack->count == stack->capacity)
        {
            status = EXAMPLE_FULL;
        }
        else
        {
            stack->items[stack->count] = value;
            stack->count++;
            status = EXAMPLE_OK;
        }
    }

    return status;
}
```

Four rules keep that readable rather than a staircase:

1. **Initialise the result to the failure value.** Then the validation arm needs
   no `else`, and a path you forgot to write fails safe instead of returning
   success by accident.
2. **When nesting would pass three levels, extract a `static` helper.** Deep
   nesting is the real cost of single-exit, and MISRA does not ask you to pay
   it. A helper with its own single exit is both compliant and clearer.
3. **`goto cleanup` is still allowed and still preferred** for functions that
   acquire several resources - MISRA Rule 15.1 permits `goto`, provided it only
   ever jumps *forward* to a label later in the same function, which the cleanup
   idiom does. It gives you one exit and one cleanup path at the same time.
4. **A function that checks, then acts, is one `do { } while (false)` pass.**
   Each check that fails sets the result and `break`s to the single `return`, so
   the work after the last check sits flat instead of inside every check. The
   NimaLTD libraries use this for their public functions.

```c
example_status_t example_send(example_t *dev, const uint8_t *data, size_t len)
{
    example_status_t status = EXAMPLE_NOT_READY;

    assert_param(dev != NULL);
    assert_param(data != NULL);

    /* Runs once. A check that fails breaks out to the one return at the end. */
    do
    {
        if (dev->ready == 0U)
        {
            break;
        }

        if (example_lock(dev) != EXAMPLE_OK)
        {
            status = EXAMPLE_BUSY;
            break;
        }

        /* No break from here on: the lock is held, and is given back below. */
        status = example_transfer(dev, data, len);
        example_unlock(dev);
    }
    while (false);

    return status;
}
```

   - **Pointer arguments are checked by `assert_param` only** (`assert` outside
     the HAL), never tested again in the block. A `NULL` is a bug in the
     caller: a debug build with Enable Full Assert stops at the line, and the
     README says so. The block checks what a correct caller can still meet,
     such as a handle whose init failed, and returns an error for that.
   - `while (false)`, never `while (0)`. Rule 14.4 wants an essentially Boolean
     condition, and Rule 14.3 permits a `do ... while` whose condition is false.
   - Several `break`s in one loop deviate from Rule 15.4 (Advisory). This guide
     records that deviation once, here, for this shape only.
   - **No `break` once something is held**, a mutex taken or a pin driven. From
     that point the work runs to the end of the block, with the release last
     inside it, and a comment says so.
   - A `break` inside an inner `while`, `for` or `switch` leaves only that one,
     so a check that must leave the whole block never sits inside one.
   - **Take the mutex with `osal_mutex_lock()` right there in the block**, and
     map its answer to the library's errors in the two checks that follow it
     (`OSAL_ERR_TIMEOUT` to `_ERR_TIMEOUT`, anything else but `OSAL_ERR_NONE`
     to `_ERR_MUTEX`), each with its own `break`. Never wrap the lock in a
     helper such as `ee_lock()`: he asked for that to be undone in ee 5.0.0.
     The lock is the line that starts the held section, and a reader has to
     see it, its two exits, and the unlock at the bottom in one function.
     ee24's `ee24_read()` is the shape to copy.

`void` functions get the same treatment: wrap the body rather than `return;`
early. A bare `return;` at the end of a `void` function is redundant - leave it
out.

## MISRA C:2012

Projects that declare MISRA conformance follow this section in addition to
everything above. Where a MISRA rule and a style rule in this document disagree,
**the MISRA rule wins** and the style rule is amended to match - as happened to
the guard-clause guidance in [Where to put the return](#where-to-put-the-return).

The reference is **MISRA C:2012, Third Edition, First Revision (2019), with
Amendments 1 and 2**. Rules are cited as `Rule 15.5` and directives as
`Dir 4.9`.

### Categories, and what each one means for you

| Category | Deviation allowed? | What you must do |
|---|---|---|
| **Mandatory** | Never | Fix the code. There is no other option. |
| **Required** | Yes, with a formal deviation record | Fix it, or record a deviation with justification |
| **Advisory** | Yes, no formal record needed | Follow it unless there is a reason not to; note the reason in a comment |

A project claiming conformance must state its category coverage honestly. Say
"MISRA C:2012 compliant, Required and Mandatory rules, with documented
deviations" rather than a bare "MISRA compliant", which claims more than any
real embedded project delivers.

### The deviation record

Every Required rule you break needs a recorded justification and a marker at the
site. Nothing is deviated silently.

Keep the record wherever it will actually be maintained. Two shapes both work:

- **In the suppressions file itself** (`tools/misra/misra-suppressions.txt`),
  as a comment block above each suppression. One file instead of two, and the
  justification is impossible to miss when someone edits the suppression - which
  is exactly the moment it matters. This is the default; the rest of this
  section assumes it.
- **In a separate `MISRA-Deviations.md`**, with the suppressions file citing
  entries by id. Worth it when a certification body wants a standalone
  compliance document, or when the record grows long enough to need headings and
  a table of contents.

Either way the content is the same, and the rule is the same: a suppression with
no matching justification is an undocumented deviation wearing a disguise.

The site marker names the rule and points at the record:

```c
/* MISRA Rule 11.4 deviation (D-001): a memory-mapped peripheral register is
 * only reachable by converting its documented address to a pointer. There is no
 * conforming alternative on this architecture. */
#define OS_ARCH_REG_ICSR  (*(__IO uint32_t *)0xE000ED04UL)
```

A deviation that cannot answer "why is this safe anyway" is a bug, not a
deviation. So every entry answers four questions, in this shape (the ids run
`D-001`, `D-002`, ... in the project's own record; the entry below is the
template, not a real one):

```markdown
### D-nnn - `Rule 11.4`: converting an integer to a pointer

**Applies to:** the `REG_*` definitions in `port/registers.h`.

**What the code does:** converts a documented, architecturally fixed address to
a pointer so a memory-mapped register can be read or written.

**Why there is no conforming alternative:** the addresses are fixed by the
architecture, not by the linker, so no object exists to take the address of.

**What limits the risk:** the addresses are constants from the reference manual,
not computed at runtime; each is `volatile`; each is confined to one macro used
only by the port layer.
```

The record also opens with the **scope**: what ships and is therefore checked,
and what is excluded with a reason per entry. Test harnesses are normally out of
scope - they legitimately use `printf`, which `Rule 21.6` forbids in production
code. List them anyway. "Not checked" is a defensible position; implying it was
checked is not.

Close it by stating what was actually run and what was not. If no commercial
checker has been used and no assessor has reviewed the code, say so, and state
the category coverage honestly - "MISRA C:2012, Mandatory, Required and Advisory
rules, with documented deviations" rather than a bare "MISRA compliant".

### Rules that bite hardest in embedded C

These are the ones that shape how the code looks. The rest you will mostly pass
by writing ordinary careful C.

| Rule | Says | In practice |
|---|---|---|
| `Rule 15.5` | One exit point per function | See [Where to put the return](#where-to-put-the-return). Result variable, one `return` at the bottom |
| `Rule 15.6` | Loop/selection bodies must be braced | Already the house rule: always brace, even one-liners |
| `Rule 15.7` | `if ... else if` chains end in `else` | Add a terminating `else`, even if it only holds a comment saying nothing to do |
| `Rule 16.4` | `switch` must have a `default` | Same idea. `default:` even when unreachable |
| `Rule 16.1`-`16.7` | `switch` well-formedness | Every clause ends in `break`; no fallthrough |
| `Rule 14.4` | `if`/loop conditions must be *boolean* | `if (ptr != NULL)`, never `if (ptr)`. `if (count != 0U)`, never `if (count)` |
| `Rule 10.x` | Essential type model | No mixing signedness or implicit narrowing. Suffix unsigned literals `0U`, cast explicitly |
| `Rule 11.x` | Pointer conversions | The register-cast deviation lives here |
| `Rule 17.7` | Non-void return values must be used | `(void)` cast anything you deliberately ignore |
| `Rule 17.8` | Do not modify a parameter | Copy it into a local first |
| `Rule 8.7` | Objects used in one file are `static` | Anything not in a header gets `static` |
| `Rule 8.9` | Objects used in one function get block scope | Move file-scope statics inside where you can |
| `Rule 2.x` | No dead or unreachable code | Delete it. Do not comment it out |
| `Rule 21.x` | Standard library restrictions | No `stdio.h`, `stdlib.h` allocation, `string.h` misuse in production code |
| `Dir 4.9` | Prefer functions to function-like macros | Use `static inline`. Keep macros for compile-time switches only |
| `Dir 4.12` | Dynamic memory is forbidden | A fixed-size static pool is not dynamic memory; the standard allocator is |

### Boolean conditions (Rule 14.4) in particular

This one touches almost every file, so it is worth stating plainly. The
controlling expression of `if`, `while`, `for` and `?:` must have *essentially
boolean* type. Pointers and integers do not.

```c
/* NON-COMPLIANT */
if (task)          { ... }
if (!count)        { ... }
while (remaining)  { ... }

/* COMPLIANT */
if (task != NULL)      { ... }
if (count == 0U)       { ... }
while (remaining > 0U) { ... }
```

### Scope: what the project declares in or out

State the boundary explicitly in the deviation record. A typical split:

- **In scope** - everything that ships in the product: the library or kernel
  source, its headers, its port layer.
- **Out of scope, and why** - third-party and vendor code you do not own (a
  silicon vendor's HAL, CMSIS headers), and test harnesses, which legitimately
  use `printf` and other facilities `Rule 21.6` forbids in production code.

Out-of-scope code is still listed. "We did not check it" is a valid position;
pretending it was checked is not.

### Checking it

Hand-auditing 143 rules does not scale, so wire up a checker and run it in CI.
The free option is cppcheck's MISRA addon, which ships with cppcheck - nothing
else to install.

| Platform | Install |
|---|---|
| Debian / Ubuntu | `sudo apt install cppcheck` |
| Fedora | `sudo dnf install cppcheck` |
| macOS | `brew install cppcheck` |
| Windows | `winget install Cppcheck.Cppcheck` or `scoop install cppcheck` |

#### Files to create

Four files, in `tools/misra/` off the project root. This is the whole setup and
it is the same in every project - copy it and change the paths.

**`tools/misra/misra.json`** - addon configuration:

```json
{
    "script": "misra.py",
    "args": ["--rule-texts=misra-rule-texts.txt"]
}
```

**`tools/misra/misra-suppressions.txt`** - one entry per deviation, each tagged
with the record id it belongs to:

```text
# D-001  Rule 11.4 - integer to pointer, memory-mapped registers.
misra-c2012-11.4:*/port/registers.h

# Out of scope entirely (see the deviation record).
*:*/test/*
```

**`tools/misra/run-misra.sh`** - the runner. Exits non-zero on any finding, so
it gates CI as is:

```sh
#!/bin/sh
set -eu
here=$(CDPATH='' cd -- "$(dirname -- "$0")" && pwd)
root=$(CDPATH='' cd -- "$here/../.." && pwd)

command -v cppcheck >/dev/null 2>&1 || { echo "cppcheck not found" >&2; exit 2; }

cppcheck \
    --addon="$here/misra.json" \
    --suppressions-list="$here/misra-suppressions.txt" \
    --std=c11 \
    --platform=arm32-wchar_t2 \
    --enable=style,warning \
    --inline-suppr --quiet --error-exitcode=1 \
    -I "$root/include" \
    "$root/src"
```

**A build-system target**, so it runs the same way everywhere. Opt-in, so a
machine without cppcheck still builds:

```cmake
find_program(CPPCHECK_EXECUTABLE cppcheck)
if(CPPCHECK_EXECUTABLE)
    add_custom_target(misra
        COMMAND ${CMAKE_COMMAND} -E env sh "${CMAKE_CURRENT_SOURCE_DIR}/tools/misra/run-misra.sh"
        COMMENT "Running MISRA C:2012 check"
        VERBATIM USES_TERMINAL)
endif()
```

#### Four things to get right

1. **Set `--platform` to the real target.** The essential-type rules (10.x)
   depend on integer widths, so checking a 32-bit embedded target with host
   widths gives confidently wrong answers. `arm32-wchar_t2` for Cortex-M.
2. **The suppressions file must mirror the deviation record, entry for entry.**
   A suppression with no matching record is an undocumented deviation wearing a
   disguise. Review the two together, always.
3. **Supply a config header if the code needs one.** A checker that cannot
   resolve an `#include` silently checks a different program than the compiler
   builds. Stage a default config into a temp dir in the runner if the real one
   is application-supplied.
4. **The rule-text file is not redistributable.** cppcheck prints bare rule
   numbers without it:

   ```text
   [task.c:412]: (style) misra violation (use --rule-texts=<file> to get proper output)
   ```

   The headline texts are MISRA copyright, so no project can ship them. If you
   own a copy of the standard, write `tools/misra/misra-rule-texts.txt` yourself
   in cppcheck's format (`Rule 1.2`, newline, the text, blank line) and
   **gitignore it**. Without it, drop the `args` line from `misra.json`.

#### What this does and does not prove

**Does:** catch regressions on every commit, cheaply, across the rules the addon
implements.

**Does not:** constitute a conformance assessment. The addon does not implement
every rule. Commercial tools (PC-lint Plus, Polyspace, Helix QAC) cover more and
are what a certification body will expect; a certified product needs its own
tool and its own compliance matrix. The free path is still worth having - it is
the difference between "we checked once" and "it stays checked".

## License and copyright

Every source and header file carries a copyright/license notice in its `@file` header block (shown under [Doxygen for functions and files](#doxygen-for-functions-and-files)): the copyright holder, the current year, and an SPDX identifier for the license.

**When scaffolding a new project, ask the user two things before writing files** (don't guess):
1. **Copyright holder** - the name to put in the `(c) <year> <holder>` line.
2. **License** - offer the common choices and a custom option, e.g. `MIT`, `Apache-2.0`, `GPL-3.0-only`, `BSD-3-Clause`, `Proprietary`, or *custom*.

Then:
- For a **standard license**, put its SPDX identifier in each file's header (`SPDX-License-Identifier: Apache-2.0`) and write the license's full text to `LICENSE.md` in the project root.
- For a **custom license**, the header references it with `SPDX-License-Identifier: LicenseRef-Custom`, and the **full custom text goes in `LICENSE.md` at the project root** - that file is the single source of truth. Tell the user the `LICENSE.md` has been created and that they should review/complete its terms.

Either way there is exactly one `LICENSE.md` in the root and the per-file notices point to it rather than repeating the text. Mention the `LICENSE.md` to the user when you create it.

For an existing project, match the license and notice format already in use rather than asking or imposing this one.

## Code formatting (.clang-format)

This skill ships a `.clang-format` that encodes the *mechanical* rules - Allman braces, 4-space indentation, `int *p` pointers, aligned macros/assignments/declarations/trailing-comments, keyword spacing, and no single-line blocks. Copy it to the project root so editors and CI can auto-format to this style. It is written to be idempotent on code already in this style (running it changes nothing).

What it can and can't do:
- **Enforced by clang-format:** brace style, indentation, pointer binding to the variable (`int *p`), column wrapping, vertical alignment of macros/assignments/declarations/trailing comments, spacing, expansion of short blocks.
- **Conventions clang-format cannot enforce - keep applying by hand:** snake_case naming, the file-name prefix on file-scope identifiers, the section banners, the per-function and prototype separators, Doxygen presence and content, the file header/copyright block, include grouping order, the inline `/* */` step comments, and **the pointer-`*`-in-the-name-column alignment for struct members** (clang-format's `PointerAlignment: Right` parks the `*` one column back and aligns the identifier instead, so it will undo this - fix it by hand after formatting). clang-format preserves all of these (it won't delete your banners or comments) but it won't create or check them. For automated naming/prefix checks, use clang-tidy's `readability-identifier-naming`.

Run against code already in this style, the bundled `.clang-format` is a no-op - the reference files pass through it unchanged.

If a project already has its own `.clang-format`, defer to it.

## Workflow

1. If the task touches an existing file or project, read it first and match its established conventions over these defaults where they differ - consistency within a codebase wins.
2. For a new project, ask the user for the copyright holder and license up front (see [License and copyright](#license-and-copyright)); write `LICENSE.md` in the root and drop the bundled `.clang-format` there too.
3. For a new module, write the `.h` first (the public contract), then the `.c`.
4. Lay out the section banners in order before filling them in; it keeps the structure right from the start.
5. Document as you write, not after - the Doxygen block clarifies the function's contract before you implement it.
6. Keep the `.c` implementations in the same order as the header prototypes as you go (see [File organization](#file-organization)).
7. Before finishing, check the function order matches between header and source, every `@param` has a direction, and the file passes the bundled `.clang-format` cleanly.
8. Save real deliverables as actual `.c`/`.h` files the user can use, not just inline code, when the request implies a usable program or module.

When the user overrides a specific choice (a different brace style, tabs, a C standard, camelCase), honor that override and keep applying every convention they didn't change.
