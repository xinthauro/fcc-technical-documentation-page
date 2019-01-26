# Effective C Tips #1 – Using vsprintf()

I’m kicking the series off on the rarely used standard library function, vsprintf(). First, some preamble…

One of the perverse things I tend to do is look through the C standard library and examine functions that on the face of it seem, well, useless. I do this because I think the folks that worked on this stuff were in general very smart and thus had a very good reason for including some of these ‘weird’ functions. One of these is the function ‘vsprintf’. If you go and look up the definition of this function, e.g. here , then you’ll find a rather brain ache inducing description. Now back when I was a lad I’d look at descriptions such as this and simply shrug and walk away. However, about ten years ago I started to make a concerted effort to see if a function such as vsprintf has a real benefit in embedded systems. Here’s what I discovered in this case:

If you are working on a product that contains a VFD or LCD, then you will almost certainly have code that contains a function for writing a string to the display at a specified position. For example:

```C
static void display_write(uint8_t row, uint8_t col, char const *buf)
{
    /* send formatted string to display - hardware dependent */
}
```

Then you will also have a plethora of functions that essentially do the same thing. That is accept some data, allocate a buffer on the stack, use sprintf to write formatted data into the buffer, and then call the function that actually writes the buffer to the display at the required position. Here’s some examples:

```C
void display_temperature(float ambient_temperature)
{
    char buf[10];

    sprintf(buf,"%5.2f", ambient_temperature);
    display_write(6, 8, buf);
}

void display_time(int hours, int minutes, int seconds)
{
    char buf[12];

    sprintf(buf,"%02d:%02d:%02d", hours, minutes, seconds);
    display_write(3, 9, buf);
}
```

There’s nothing really wrong with this approach. However, there is a better way, courtesy of vsprintf().

What one does is to modify display_write() to take a variable length argument list. Then within display_write() use vsprintf() to process the variable length argument list and to generate the requisite string. The basic structure for the function is as follows:

```C
void display_write(uint8_t row, uint8_t column, char const *format, ...)
{
    va_list  args;
    char  buf[MAX_STR_LEN];

    va_start(args, format);
    vsprintf(buf, format, args);    /* buf contains the formatted string */

    /* send formatted string to display - hardware dependent */

    va_end(args);                   /* clean up, do NOT omit */
}
```

My objective here is not to explain how to use variadic arguments or indeed how vsprintf() works, there are dozens of places on the web that will do that. Instead I’m interested in showing you the benefit of this approach. The display_write() function has evidently become more complex; however the functions that call display_Write have become dramatically simplified, as they are now just:

```C
void display_temperature(float ambient_temperature)
{
    display_write(6, 8, "%5.2f", ambient_temperature);
}

void display_time(int hours, int minutes, int seconds)
{
    display_write(3, 9, "%02d:%02d:%02d", hours, minutes, seconds);
}
```

Is this more Effective code? I think so, for the following reasons.

* The higher level functions are now much cleaner and easier to follow.

* All the heavy lifting is localized in one place, which typically dramatically reduces the probability of errors.

Finally, you’ll typically end up with a nice reduction in code size (even though this wasn’t my objective). All in all, not bad for one obscure function.
