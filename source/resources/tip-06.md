# Effective C Tip #6 – Creating a flags variable

Now I’m going to address the topic of creating what I call a flags variable. A flags variable is nothing more than an integral type that I wish to treat as an array of bits, where each bit represents a flag or boolean value. I find these particularly valuable in three situations:

When the CPU has part of its address space that is much faster to access than other regions. Examples are the zero page on 6805 type processors, and the lower 256 bytes of RAM on AVR processors. Depending upon your compiler, you may also want to do this with the bit addressable RAM region of the 8051.
When I’m running short on RAM and thus assigning an entire byte or integer to store a single boolean flag is waste I can’t afford.
When I have a number of related flags where it just makes sense to group them together.
The basic approach is to use bitfields. Now I’m not a huge fan of bitfields – particularly when someone tries to use them to map onto hardware registers. However, for this application they work very well. As usual however, the devil is in the details. To show you what I mean, I’ll first show you a typical implementation of mine, and then explain what I’m doing and why.

```C
typedef union {
    uint8_t     all_flags;      /* allows us to refer to the flags 'en masse' */
    struct {
        uint8_t foo: 1,         /* explanation of foo */
        uint8_t bar: 1,         /* explanation of bar */
        uint8_t spare5: 1,      /* unused */
        uint8_t spare4: 1,      /* unused */
        uint8_t spare3: 1,      /* unused */
        uint8_t spare2: 1,      /* unused */
        uint8_t spare1: 1,      /* unused */
        uint8_t spare0: 1;      /* unused */
    };
} flags_t;

static flags_t flags;           /* allocation for the Flags */

flags.all_flags = 0u;           /* clear all flags */

...

flags.bar = 1u;                 /* set the bar flag */
```

There are several things to note here.

## Use of a union

The first thing to note is that I have used a union of an integral type (uint8_t) and a structure of bitfields. This allows me to access all the flags ‘en masse’. This is particularly useful for clearing all the flags as shown in the example code. Note that our friends at MISRA disallow unions. However, in my opinion, this is a decent example of where they make for better code – except see the caveat below.

## Use of integral type

Standard C requires that only types int and unsigned int may be used for the base type of an integer bitfield. However, many embedded systems compilers remove this restriction and allow you to use any integral type as the base type for a bitfield. This is particularly valuable on 8-bit processors. In this case I have taken advantage of the language extension to use a uint8_t type.

## Use of an anonymous structure

You will note that the bitfield structure is unnamed, and as such is an anonymous structure. Anonymous structures are part of C++ – but not standard C. However, many C compilers support this construct and so I use it as I feel it makes the underlying code a lot easier to read.

## Naming of unused flags

If you look at the way I have named the unused flags, it looks a little odd. That is the first unused flag is spare5, the next spare4 and so on down to spare0. Now I rarely do things on a whim, and indeed this is a good example. So why do I do it this way? Well, there are two reasons:

1. When I first create the structure, I label all the flags, starting from spare7 down to spare0. This inherently ensures that I name precisely the correct number of flags in the structure. To see why this is useful, take the above code and allocate an extra flag in the bitfield structure. Then compile and see if you get a compilation error or warning. Whether you will or not depends upon whether your compiler allows bitfields to cross the storage unit boundary. If it does, then your compiler will allocate two bytes, and the all_flags member of the union will not cover all of the flags. This can come as a nasty surprise (and perhaps explains why MISRA is wary of unions). You can prevent this from happening by naming the flags as shown.

2. When it becomes necessary to allocate a new flag, I simply replace the topmost unused flag (in this example that would be spare5) with its new name, e.g. zap. The remainder of the structure is unchanged. If instead I had named the topmost unused flag ‘spare0’, the next ‘spare1’ and so on, then the code would give a completely misleading picture of how many spare bits are left for future use after I had taken one of the unused flags.

If you look at what I have done here, it’s interesting to note that I have relied upon two extensions to standard C (which violates the MISRA requirement for no use of compiler extensions) and I have also violated a third MISRA tenet via the use of a union. I would not be surprised if I’ve also violated a few other rules as well. Now I don’t do these things lightly, and so I only use this construct when I see real benefit in doing so. I’ll leave it for another day to discuss my overall philosophy regarding adherence to the MISRA guidelines. It is of course up to you the reader to make the determination as to whether this is indeed effective C.
