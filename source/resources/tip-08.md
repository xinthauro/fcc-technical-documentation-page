# Effective C Tip #8 – Structure Comparison

This topic concerns how best to compare two structures (naturally of the same type) for equality. The conventional wisdom is that the only safe way to do this is by explicit member by member comparison, and that one should avoid doing a straight bit (actually byte) comparison using memcmp() because of the problem of padding bytes.

To see why this argument is advanced, one must understand that a compiler is free to place pad bytes between members of a structure so as produce more favorable alignment of the data in memory. Furthermore, the compiler is not obligated to initialize these pad bytes to any particular value. This code fragment illustrates the problem:

```C
typedef struct {
    uint8_t x;
    uint8_t pad1;   /* compiler added padding */
    uint8_t y;
    uint8_t pad2;   /* compiler added padding */
} coord_t;

void foo(void)
{
    coord_t p1 p2;

    p1.x = p2.x = 3;
    p1.y = p2.y = 4;

    /* note pad bytes are not initialized */
    if (memcmp(&p1, &p2, sizeof(p1)) != 0) {
        /* we may get here */
    }
    ...
}
```

Thus, it’s clear that to avoid these kinds of problems, we must do a member by member comparison. However, before you rush off and start writing these member by member comparison functions, you need to be aware of a gigantic weakness with this approach. To see what I mean, consider the comparison function for my coord_t structure. A reasonable implementation might look like this:

```C
bool are_equal(coord_t *p1, coord_t *p2)
{
    return ((p1->x == p2->x)  &&  (p1->y == p2->y));
}

void foo(void)
{
    coord_t p1 p2;
    p1.x = p2.x = 3;
    p1.y = p2.y = 4;
    if (!are_equal(&p1, &p2)) {
        /* we should never get here */
    }
    ...
}
```

Now consider what happens if I add a third member z to the coord_t structure. My structure definition and function foo() become:

```C
typedef struct {
    uint8_t x;
    uint8_t pad1;   /* compiler added padding */
    uint8_t y;
    uint8_t pad2;   /* compiler added padding */
    uint8_t z;
    uint8_t pad3;   /* compiler added padding */
} coord_t;

void foo(void)
{
    coord_t p1 p2;
    p1.x = p2.x = 3;
    p1.y = p2.y = 4;
    p1.z = 6;
    p2.z = 5;
    if (!are_equal(&p1, &p2){
        /* we will not get here */
    }
    ...
}
```

The problem is that I now have to remember to also update the comparison function. Now clearly in a simple case like this, it isn’t a big deal. However, in the real world where you might have a 500 line file, with the comparison function buried miles away from the structure declaration, it is way too easy to forget to update the comparison function. The compiler is of no help. Furthermore it’s my experience that all too often these sorts of problems can exist for a long time before they are caught. Thus the bottom line, is that member by member comparison has its own set of problems.

So what do I suggest? Well, I think the following is a reasonable approach:

1. If there is no way that your structure can change (presumably because of outside constraints such as hardware), then use a member by member comparison.

2. If you are working on a system where structure members are aligned on byte boundaries (which is true to the best of my knowledge for all 8 bit processors, and also most 16 bit processors), then use memcmp(). However, you need to think about doing this very carefully if there is the possibility of the code being ported to a platform where alignment is not on an 8 bit boundary.

3. If you are working on a system that aligns on a non 8 bit boundary, then you must either use member by member comparison, or take steps to ensure that all the bytes of a structure are initialized using memset() before you start assigning values to the actual members. If you do this, then you can probably use memcmp() with a reasonable amount of confidence.

4. If speed is a priority, then clearly memcmp() is the way to go. Just make sure you aren’t going to fall into a pothole as you blaze down the road.

Before I leave this topic, I should mention a few esoteric things for you to consider.

If you use the memcmp() approach you are checking for bit equality rather than value equality. Now most of the time they are the same. Sometimes however, they are not. To illustrate this, consider a structure that contains a parameter that is a boolean. If in one structure the parameter has a value of 1, and in the other structure it has a value of 2, then clearly they differ at the bit level, but are essentially the same at a value level. What should you do in this case? Well clearly it’s implementation dependent. It does however illustrate the perils of structure comparison.

Finally I should mention issues associated with structures that contain pointers. CS guys like to distinguish between deep and shallow structure comparison. I rarely write code where a deep comparison is required, and so for me it’s mostly a non-issue.
