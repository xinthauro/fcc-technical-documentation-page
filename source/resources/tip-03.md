# Effective C Tip #3 – Exiting an intentional trap

In this tip I’d like to give you a useful hint concerning traps. What exactly do I mean by a trap? Well while C++ has a ‘built in’ exception handler (try searching for ‘catch’ or ‘throw’), C does not (thanks to Uhmmmm for pointing this out). Instead, what I like to do when debugging code is to simply spin in an infinite loop when something unexpected happens. For example consider this code fragment:

```C
switch (foo)
{
    case 0:
        ...
    break;
    case 1:
        ...
    break;
        ...
    default:
        trap();
    break;
}
```

My expectation is that the default case should never be taken. If it is, then I simply call the routine trap(). So what does trap() look like? Well the naive implementation looks something like this:

```C
void trap(void)
{
    for (;;) {
    }
}
```

The idea is that when the system stops responding, stopping the debugger will show that something unexpected happened. However, while this mostly works, it has a number of significant shortcomings. The most important is that leaving code like this in a production release is definitely not a good idea, and so the first modification that needs to be made is to arrange to remove the infinite loop for a release build. This is usually done by defining NDEBUG. The code thus becomes:

```C
void trap(void)
{
#ifndef NDEBUG
    for (;;) {
    }
#endif
}
```

The next problem with this trap function is that it would be ineffective in a system that executes most of its code under interrupt. As a result, it makes sense to disable interrupts when entering the trap. This is of course compiler / platform specific. However it will typically look something like this:

```C
void trap(void)
{
#ifndef NDEBUG
    __disable_interrupts();
    for (;;) {
    }
#endif
}
```

The final major problem with this code is that it’s hard to tell what caused the trap. While you can of course examine the call stack and work backwards, it’s far easier if you instead do something like this:

```C
static volatile bool exit_trap = false;

void trap(void)
{
#ifndef NDEBUG
    __disable_interrupts();
    while (!exit_trap) {
    }
#endif
}
```

What I’ve done is declare a volatile variable called exit_trap and have initialized it to false. Thus when the trap occurs, the code spins in an infinite loop. However by setting Exit_Trap to true, I will cause the loop to be exited and I can then step the debugger and find out where the problem occurred.

Regular readers will perhaps have noticed that this isn’t the first time I’ve used volatile to achieve a useful result.

Incidentally I’m sure that many of you trap errors via the use of the assert macro. I do too – and I plan to write about how I do this at some point.

So does this meet the criteria for an effective C tip? I think so. It’s a very effective aid in debugging embedded systems. It’s highly portable and it’s easy to understand. That’s not a bad combination!
