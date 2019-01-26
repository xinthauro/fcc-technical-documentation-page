# Effective C Tip #7 – Use strongly typed function parameters

This topic concerns function parameters, and more to the point, how you should choose them in order to make your code considerably more resilient to parameter passing errors.  What do I mean by parameter passing errors? Well consider a function that is intended to draw a rectangle on a display. The lousy way to design this function interface would be something like this:

```C
void draw_rect(int x1, int y1, int x2, int y2, int color, int fill)
{
    ...
}
```

I must have seen a function like this many times. So what’s wrong with this you ask? Well in computer jargon the parameters are too weakly typed. To put it into plain English, it’s way too easy to pass a Y ordinate when you are supposed to pass an X ordinate, or indeed to pass a color when you are supposed to be passing an ordinate or a fill pattern. Although in this case (and indeed in most cases) these types of mistakes are clearly discernible at run time, I’m a firm believer in catching as many problems at compile time as possible. So how do I do this? Well there are various things one can do. The most powerful technique is to use considerably more meaningful data types. In this case, I’d do something like this:

```C
typedef struct {
    int x;
    int y;
} coordinate_t;

typedef enum {
    RED, BLACK, GREEN, PURPLE ... YELLOW
} color_t;

typedef enum {
    SOLID, DOTTED, DASHED .. MORSE
} fill_pattern_t;

void draw_rect(coordinate_t, coordinate_t, color_t color, fill_pattern_t fill)
{
    ...
}
```

Now clearly it’s highly likely that your compiler will complain if you attempt to pass a coordinate to a color and so on – and thus this is a definite improvement. However, nothing I’ve done here will prevent the X & Y ordinates being interchanged. Unfortunately, most of the time you are out of luck on this one – except in the case where you are dealing with certain sizes of display panels with resolutions such as 320 * 64, 320 * 128 and so on. In these cases, the X ordinate must be represented by a uint16_t whereas the Y ordinate may be represented by a uint8_t. In which case my coordinate_t data type becomes:

```C
typedef struct {
    uint16_t x;
    uint8_t y;
} coordinate_t;
```

This will at least cut down on the incidence of parameters being passed incorrectly.

Although you probably will not get much help from the compiler, you can also often get a degree of protection by declaring appropriate parameters as const. A good example of this is the standard C function memcpy(). If like me, you find yourself wondering if it’s memcpy(to, from) or memcpy(from, to), then an examination of the function prototype tells you all you need to know:

```C
void *memcpy(void *s1, const void *s2, size_t n);
```

That is, the first parameter is simply declared as a void *pointer, whereas the second parameter is declared as void *pointer to const. In short the second parameter points to what we are reading from, and hence memcpy is indeed memcpy(to, from). Now I’m sure that many of you are thinking to yourself – so what, the real solution to this is to give meaningful names to the function prototype. For example:

```C
void *memcpy(void *destination, const void *source, size_t n_bytes);
```

Although I agree wholeheartedly with this sentiment, I’ll make two observations:

1. You are assuming that the person reading your code is sufficiently fluent in the language (English in this case) that the names are meaningful to them.

2. Your idea of a meaningful label may not be shared by others. I’ve noticed that this is particularly the case with software, as it seems that all too often the ability to write code and the ability to put a meaningful sentence together are inversely correlated.

The final technique that I employ concerns psychology!  Now one can argue that the failure to pass parameters correctly is due to laziness on behalf of the caller. At the end of the day, this is indeed the case. However, I suspect that in many cases, it’s not because the caller was lazy, but rather it’s because the caller thought they knew what the function parameter ordering is (or should be). A classic example of this of course concerns dates. Being from the UK (or more relevantly – Europe), I grew up thinking of dates as being day / month / year. Here in the USA, they of course use the month / day / year format. Thus when designing a function that needs to be passed the day, month and year, in what order should one declare the parameters? Well in my opinion it’s year, month, day. That is the function should look like this:

```C
void foo(year_t year, month_t month, day_t day)
```

There are several things to note:

1. By putting the year first, one causes both Europeans and Americans to think twice. This is where the psychology comes in!

2. I’ve made the year signed – because it can indeed be negative, whereas the month and day cannot.

3. I’ve made the month a MONTH data type, thus considerably increasing the likelihood that an attempt to pass a day when a month is required will be flagged by the compiler.

4. I’ve made the day yet another data type (that maps well on to its expected range). Furthermore, attempts to pass most year values to this parameter will result in a compilation warning.

Thus I’ve used a combination of psychology and good coding practice to achieve a more robust function interface.
Thus the bottom line when it comes to designing function interfaces:

1. Use strongly typed parameters.

2. Use const where you can.

3. Don’t assume that what is ‘natural’ to you is ‘natural’ to everyone.

4. Do indeed use descriptive parameter names – but don’t assume that everyone will understand them.

5. Apply some pop psychology if necessary.

I hope you find this useful.
