---
layout: post
title:  "Syntactic sugarcoating for memory mapped registers"
date:   2016-03-30 19:00:00
categories: embedded C programming
permalink: /articles/register-syntax-sugar
---

Most instruction set architectures (ISAs) nowadays utilize [memory mapped IO][MMIO] (MMIO). Some address ranges are decoded as being directed at IO devices instead of RAM.

For example, on the ARM Cortex-M, the System tick timer's control register is located at `0xE000E010`.
If you wish to calibrate the timer, you can do this easily from C with a tiny bit of pointer fiddling:

```c
*(unsigned long volatile *)0xE000E010 = 5;
```
or the more readable

```c
typedef unsigned long volatile reg;
#define SYSTICK_CTRL (*(reg*)0xE000E010)
SYSTICK_CTRL = 5;
```

In this post, I want to compare different approaches of wrapping access to such a register.




## Approach I: Least effort ##

```c
*(unsigned long volatile *)0xE000E010 = 5;
```

**Pros:** It works and it won't get any faster than that.

The syntax used is standard C syntax, casting integers (`0xE000E010` is an `unsigned int`[^1]) to pointers is implementation-defined behavior, but if you deal with MMIO, the implementation should do the right thing with it.

> The `volatile` is necessary because the compiler is inclined to believe that all variables are its own. Thus running: `*(unsigned long*)0xE000E010 = 0; *(unsigned long*)0xE000E010 = 1;` is the same as `*(unsigned long*)0xE000E010 = 1; ` as far as the compiler is concerned because the `0` is overwritten anyway. 
> 
> With MMIO though, writing has a real world side effect that need not be optimized away. This is the main use for `volatile`[^2].

**Cons:** Magic numbers are bad. You won't get rid of them, but at least put them in a header.

```c
#define SYSTICK_CTRL 0xE000E010
*(unsigned long*)SYSTICK_CTRL = 0
```

isn't much better, so let's skip that one.

Before we move on, let's have a look at the THUMB assembly GCC generated[^3]:

```c
; *(unsigned long volatile *)0xE000E010 = 5;
  b6:	f24e 0310 	movw	r3, #57360	; 0xe010
  ba:	f2ce 0300 	movt	r3, #57344	; 0xe000
  be:	2205      	movs	r2, #5
  c0:	601a      	str	r2, [r3, #0]
```

`r3` will point to our SysTick register. `r2` has the `5` we want to put in. The THUMB ISA doesn't support moving 32 bit immediates at once, so we have a move for the lower 16 bits of the address, then a 16 bit move for the top bits. Finally we store `r3[0]` (i.e. `*r3`) into `r2`.

## Approach II: The usual ##

Probably the most widely used

```c
#define SYSTICK_CTRL (*(unsigned long volatile*)0xE000E010)
SYSTICK_CTRL = 5;
```

After preprocessing, you get the same output as Approach I. But this is more readable and has the magic numbers moved out of the way. You can go the extra mile and have a global header with all addresses, which you then include into the `SysTick.h` header. This is particularly useful, if addresses were to change. Say, you are writing a cheat tool for a game and need to maintain pointers to your health and ammo count. Having all the addresses at one place makes it a bit more straight forward to update the addresses, when the game itself gets updated.

## Approach III: Accessors ##
We won't talk a lot about this one, but for completion's sake:

```c
static inline void setSysTickCtrl(unsigned long x) {
    SYSTICK_CTRL = x;
}

static inline unsigned long getSysTickCtrl(void) {
    return SYSTICK_CTRL;
}
```

The common setter/getter pattern. C99's `inline` is a hint at the compiler, that this function should be inlined (i.e. copy-pasted) into the calling code. `static` specifies internal [linkage], which means the compiler need not expect another function outside the current translation unit (standardese for source file) to call this function.

This approach has [no overhead][inline] but I find it ugly. Having getters and setters allows for mocking out parts for unit testing, but I think mocking should happen at the driver not at the register level.

One could write this as macros at the cost of type safety, but for this, it wouldn't matter.

## Approach IV: Pointer to successive elements ##

Controlling SysTick involves 4 sequential registers in memory. Instead of defining every one, we just define a pointer to the base and use an index

```c
enum {SYSTICK_CTRL, SYSTICK_LOAD, SYSTICK_VAL, SYSTICK_CAL};
unsigned long volatile *SysTick = (void*)0xE000E010;
SysTick[SYSTICK_CTRL] = 5;
SysTick[SYSTICK_VAL] = 0;
```

**Cons:** The `enum` looks ugly, but because enumeration constants share a single name space with other identifiers in C, you should prefix your enums, to avoid identifier clashes. Also, this approach isn't as safe, because you can avoid the enums and use integer indices directly. 

Additionally, `SysTick` is a global object. If we define it in more than one translation unit, the linker will complain. Solution: Make it `static`[^4]. The downside is that we need to silence[^5] warnings about including the header when we don't use `SysTick`.

Finally, the code generated can be[^6] suboptimal:

```c
; SysTick[SYSTICK_CTRL] = 5;
  ba:	f240 0300 	movw	r3, #0
  be:	f2c2 0300 	movt	r3, #8192	; 0x2000
  c2:	681b      	ldr	r3, [r3, #0]
  c4:	2205      	movs	r2, #5
  c6:	601a      	str	r2, [r3, #0]
```

The code grew by a `ldr` (pseudo-)instruction. `ldr` loads a value from memory into a register. Because `SysTick` is a pointer which is prone to change, directly encoding the address as immediate into the instruction stream isn't possible anymore. If only there was a way to tell the compiler, that our SysTick pointer won't ever change... Yes, you guessed right, we need to use a `const`.

Our definition now looks like this:

```c
unsigned long volatile * const SysTick = (void*)0xE000E010;
```

> **Note:** that we want the pointer to be `const` and the integer pointee to be `volatile`. Thus the `volatile` is before the `*` operator and the `const` after it.

After adding `const`, the extra indirection isn't necessary anymore and the generated code is the same as in our first example. The other cons are still there though.

## Approach V: Pointer to struct ##

If you have something with attributes, you would use a struct/record data type. So why not here?

```c
#include <stdint.h>
typedef uint32_t u32;
static struct {
  u32 ctrl;
  u32 load;
  u32 curr;
  u32 cal;
} volatile * const SysTick = (void*)0xE000E010;

SysTick->ctrl = 5;
```

This looks more like it. The code generated is still the same as the first, but the source code is a lot more readable. The abstraction isn't complete though, There is still a pointer you need to dereference. 

While the example here works, others might not. Imagine we have a buffer, where an Ethernet frame is put into. Assuming EtherType 0x0800, we'll have the source IP address at offset 26 of that buffer.

```c
#include <stdint.h>
struct IPv4 {
    uint8_t MAC_header[14];
    uint8_t IPv4_stuff_I_do_not_care_about[12];
    uint32_t source_ip, destination_ip;
    uint8_t even_more_IPv4_stuff_I_do_not_care_about[];
};
```

Let's guess, what is `sizeof(struct IPv4)`? It can't be any less than 34 bytes, but is it exactly so? On my desktop machine and probably on yours, `sizeof(struct IPv4) == 36`.

With `<stddef.h>`'s `offsetof`, you can see the reason for that:

```c
#include <stddef.h>
printf("source_ip should be at $%d, but it's at $%d\n",
    14+12, offsetof(struct IPv4, source_ip));
```
> **Output:** source_ip should be at $26, but it's at $28

These two extra bytes are padding. 28 is divisible by 4 and as such can be directly loaded into a register. If it were instead at 26, it would cross a 4-byte boundary and the compiler would need to emit two memory accesses instead of one[^7]. Thus the struct was optimized to allow a faster access at the cost of 2 bytes of wasted space.
While that's fine if the struct is ours, in this case it's imposed on us, so we need to force the compiler not to use padding.

In Approach V, we didn't run into this problem because every member is suitably aligned already. The compiler couldn't do anything better[^8].

The C11 standard [allows][C11-alignof] us to specify only a minimum alignment for members, but we want a maximum alignment of 1 byte, that is a densely packed struct without holes.

Many compilers, especially those aimed at embedded C development, have this as an extension.
In GCC it looks like this:

```c
#define __packed __attribute((packed))__

struct IPv4 __packed {
    uint8_t MAC_header[14];
    uint8_t IPv4_stuff_I_do_not_care_about[12];
    uint32_t source_ip, destination_ip;
    uint8_t even_more_IPv4_stuff_I_do_not_care_about[];
};
```

Checking again, we see that `source_ip` is now 26 bytes into `struct IPv4`.


Using packed structs makes it easy to shoot yourself into the foot with unaligned accesses[^9]. As we only use the packed struct for the syntax sugar and won't be casting stuff, we should be safe.

## Approach VI: Anonymous pointer to struct ##

Another solution is not to define any variables at all and combine II and V:

```c
struct SysTick_ __packed {
  u32 ctrl;
  u32 load;
  u32 curr;
  u32 cal;
};
#define SysTick (*(struct SysTick_ volatile *)0xE000E010)
```

This approach yields the exact same code, The only difference is that we have to[^10] pollute the struct namespace with a `struct SysTick_`, but that's a non-issue.

A more pressing issue is that, unlike the prior approach, you can't just write `print *SysTick` when it's in scope to read out the registers in GDB. 

## Approach VII: Linker magic ##

The static const pointer to a packed struct approach gave us a nice syntax. Although, the resulting code did use the address directly and not a pointer, the syntax still involved dereferencing.
What about having a `SysTick` variable that can be accessed without dereferencing?

```c
extern volatile struct {
  u32 ctrl;
  u32 load;
  u32 curr;
  u32 cal;
} SysTick;

SysTick.ctrl = 5;
```

This certainly looks nice. You can use `&SysTick` to get back `0xE000E010` and use the `.` operator directly.

Code produced is the same as in the previous approach. It needs a little linker magic though:

```c
SECTIONS
{
    SysTick = 0xE000E010;
}
```

This informs the linker that the SysTick symbol refers to the address of `0xE000E010`. This file, called a ldscript (linker editor script), would be fed to the linker, like this:

```c
$ cc -T addrs.ld
```

Another perk is that this approach is the only one, where you can write `print SysTick` in GDB and have it read out the registers, regardless of the line currently executing. This is because `SysTick` is a global and from GDB's view always in scope. With static pointers, you would have to write `print *'file.c'::SysTick`, if you aren't currently stepping through a file defining `SysTick`.

**Cons:** Obvious disadvantage here is that linker magic differs greatly from linker to linker and specifying the address in C code directly is more portable.

---

**So**, that concludes what I got to say about this. I am going with Approach V for SysTick and the other Cortex-M registers I am wrapping. For my game cheat I am using Approach VII, because I make heavy use of GNU C anyway and having the linker script act as a config file where you can easily change addresses is kinda neat, especially when you are doing this a lot. It also avoids having to cast stuff. This is especially useful when a lot of the addresses are function pointers, writing

```c
extern int player_say(int mode, const char *msg); // extern is superflous, but it conveys the intent better
```

is just better to look at than

```c
static int (* const player_say)(int mode, const char* msg) = (int(*)())0x004073F0;
```

Not to mention the C++[^11] version:

```c++
static int (* const client_print)(
    int surface, int x, int y,
    int font, int red, int green, int blue,
    const char *text, int alignment
) = (int (*)(int,int,int,int,int,int,int,const char*,int))0x004B4DD0;
```

No, thanks.

PS: I haven't written any blog posts before and still trying to figure out what's the right mixture of detail and conciseness. If you got corrections, suggestions or questions, feel free to comment them. :-)

--- 

[^1]: On the 32-Bit ARM Cortex at least.
[^2]: Equally important is the fact that `volatile` accesses are not reordered against each other meaning that the `0` will be written before the `1`. Some ISAs, like our ARM here, buffer memory accesses and do not even enforce an [access order] unless explicitly asked with `volatile` or a memory barrier. Compilers also like to do reordering.
[^3]: `objdump -dS binary_file` but `gcc -S source_file` works too.

[^4]: The alternative of having an extern declaration and defining the variable in only one file would inhibit optimization. See the next disassembly.
[^5]: In GCC by having the definitions between `#pragma GCC diagnostic push` `#pragma GCC diagnostic ignored "-Wunused-variable"` and `#pragma GCC diagnostic pop`.
[^6]: As we already use `static`, the compiler can treat non-const-qualified `SysTick` as `const` anyway as long the current translation unit doesn't modify it. GCC does so, but only if you don't have a `&SysTick` anywhere in the file. If you do, it will generate the superflous indirection shown. C offers the `register` keyword to inhibit taking the address of objects, which could prove useful here, were it allowed at file scope. Jens Gustedt has a nice [article on named constants] and a convincing proposal for allowing `register const` at file scope.
[^7]: We are talking about ARM here. The x86's behavior in that regard is an outlier.
[^8]: This shouln't mean that the compiler might do as it pleases. It is prohibited from reordering members and adding padding before the first. Anywhere else is up to the compiler's discretion, but struct layout for a particular architecture is a quite predictable matter, so skipping `__attribute__((packed))` for `SysTick` without problems is a safe bet.
[^9]:
    While some architectures allow unaligned access and even go great lengths to make the performance hit negligible, many more don't.
    An unaligned access might yield an exception or even worse undefined behavior (like access the next aligned memory address instead).

    ```c
    IPv4 *ip_packet = ...;
    uint32_t ip;
    ip = ip_packet->source_ip; // OK, compiler knows source_ip's alignment
    uint32_t *source_ip = &ip_packet->source_ip;
    ip = *source_ip; // NOT OK, Compiler assumes default alignment, compiles to unaligned access
    ```

    So keep that in mind.

    **Fun fact of the day:** Linux puts IPv4 packets at 2 byte boundaries _not_ divisible by 4, so that source/destintation IP are 32bit-aligned. Similarily, some switch [ASIC]s inject padding bytes into the packet, to allow speedy handling of packets by the switch's CPU.



[^10]: 
    While `#define SysTick ((struct { u32 ctrl, load, curr, cal; } volatile *)0xE000E010)` seems to avoid the namespace pollution, it opens a much worse can of worms. Every use of `SysTick` expands to a different object with a different type, as no two anonymous structs in a translation unit are the same. I can’t think off the bat of an example where this would be problematic for `SysTick`, but for completion’s take, imagine this:

    ```c
    #define SysTick ((struct __packed { u32 ctrl, load, curr, cal; } *)0xE000E010)
    SysTick->ctrl = 0;
    SysTick->ctrl = 1;
    ```
    We dropped the `volatile`. Because the second and third line refer to different objects, they might be reordered. So having `SysTick->ctrl` be `1` or `0` afterwards are equally valid.

[^11]: In C, the address can be cast by `(int (*)())`, because `()` means a variable amount of arguments. In C++, `()` means `(void)`. 



[c11-alignof]: http://en.cppreference.com/w/c/language/_Alignof
[TDD]: https://en.wikipedia.org/wiki/Test-driven_development
[ASIC]: https://en.wikipedia.org/wiki/ASIC
[access order]: http://preshing.com/20120930/weak-vs-strong-memory-models/
[MMIO]: https://en.wikipedia.org/wiki/Memory-mapped_I/O
[linkage]: http://en.cppreference.com/w/c/language/storage_duration
[inline]: https://gcc.gnu.org/onlinedocs/gcc/Inline.html
[article on named constants]: https://gustedt.wordpress.com/2012/08/17/named-constants/
