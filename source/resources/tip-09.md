# Effective C Tip #9 – use #warning

Way back in 1999 I wrote an article for Embedded Systems Programming concerning the #error directive. If you aren’t particularly familiar with #error, then I suggest you read the article. While the #error directive has remained one of my most popular tools, I have become an equally big fan of #warning.

Before I delve into the uses of #warning, I must warn you (if you would pardon the pun) that #warning is a non standard directive. However it is supported by IAR, GCC, Microchip’s C18 compiler, Hi-Tech and probably a whole raft of other vendors. In other words, it’s pretty standard for a non-standard feature.

The use of #warning is simple enough:

```C
#warning "This is a warning"
```

This will result in the compiler issuing a warning with the text ‘This is a warning’ printed to stderr. Please note that, just as for #error, there is *no* requirement that the text be in quotes. If you insist on putting quotes around the text, then they will be printed to stderr as well.

With the syntax out of the way, here’s some of the ways that I use #warning.

## Protecting Incomplete Code

Very often when I’m coding, I like to get the big picture in place without worrying about the minutiae of the implementation details.  As a result I end up with functions or loop bodies that are incomplete. In these cases I simply add a #warning to alert me to the issue. For example

```C
void foo(void)
{
   #warning "to be completed"
}
```

Thus what happens now is that whenever I compile the module, I get a warning (i.e. reminder) that there is something important still to be done.

## Commenting Out Code

I *never* comment code out anymore as part of the debugging process, as it is simply too easy to forget that the code has been removed. Instead I use this construct:

```C
void foo(void)
{
# if 0
   /* code to be temporarily removed is here */
#else
   #warning "temporary debug construct, fix me!"
   /* experimental code goes here */
#endif
}
```

In this case whenever I compile the module, I get a warning (i.e. reminder) that I have some experimental code in the image.

## The Key Final Step

While the above are useful constructs, the real power of #warning comes if you configure your compiler to treat warnings as errors for the release build. If you do this and you have inadvertently left incomplete or debug code in the image, then your compilation will fail. In short, this technique will guarantee that you never release code that includes / excludes code that shouldn’t / should be there. That’s effective C.
