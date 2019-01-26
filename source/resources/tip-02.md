# Effective C Tips #2 – Defining buffer sizes

This tip is about addressing something that just about every embedded system has – a buffer whose length is a power of two.

In order to make many buffer operations more efficient, it is common practice to make the buffer size a power of two so that simple masking operations may be performed on them, rather than explicit length checks. This is particularly true of communications buffers where data are received under interrupt. As a result, it is common to see code that looks something like this:

```C
#define RX_BUF_SIZE (32)
static uint8_t rx_buf[RX_BUF_SIZE]; /* Receive buffer */

__interrupt void RX_interrupt(void)
{
    static uint8_t rx_head = 0; /* offset into rx_buf[] where next character should be written */
    uint8_t rx_char;

    rx_char = HW_REG;           /* get the received character */

    rx_head &= RX_BUF_SIZE - 1; /* mask the offset into the buffer */
    rx_buf[rx_head] = rx_char;  /* store the received char */
    ++rx_head;                  /* increment offset */
}
```

The first thing I do to make this code more flexible, is to allow the size of the buffer to be overridden on the command line. Thus my declaration for the buffer size now looks like this:

```C
#ifndef RX_BUF_SIZE
    #define RX_BUF_SIZE (32)
#endif
```

This is a useful extension because it allows me to control the resources used by the code without having to edit the code per se. However, this flexibility comes at a cost. What happens if someone was to inadvertently pass a non power of 2 buffer size on the command line? Well as it stands – disaster. However, the fix is quite easy.

```C
#ifndef RX_BUF_SIZE
    #define RX_BUF_SIZE (32)
#endif
#define RX_BUF_MASK (RX_BUF_SIZE - 1)
#if (RX_BUF_SIZE & RX_BUF_MASK)
    #error RX buffer size is not a power of 2
#endif
```

What I’ve done is define another manifest constant, RX_BUF_MASK to be equal to one less than the buffer size. I then test using a bit-wise AND of the two manifest constants. If the result is non zero, then evidently the buffer size is not a power of two and compilation is halted by use of the #error statement. If you aren’t familiar with the #error statement, you’ll find this article I wrote a few years back to be helpful.

Although this is evidently a big improvement, it still isn’t quite good enough. To see, why, consider what happens if RX_BUF_SIZE is zero. Zero is of course a power of two, and so will pass the check. Now most C90 compliant compilers will complain about declaring an array with zero length. However this is legal in C99 compilers in general and GNU compilers in particular. Thus, we also need to protect against this case. Furthermore as Yevheniy was kind enough to point out in the comments, we also have to protect against a buffer size of 1 (as 1 & 0 = 0). So we now get:

```C
#ifndef RX_BUF_SIZE
    #define RX_BUF_SIZE (32)
#endif
#if (RX_BUF_SIZE < 2)
    #error "RX buffer must be a minimum length of 2"
#endif
#define RX_BUF_MASK (RX_BUF_SIZE - 1)
#if (RX_BUF_SIZE & RX_BUF_MASK)
    #error "RX buffer size is not a power of 2"
#endif
```

As a final comment, note that the definition of RX_BUF_MASK has an additional benefit in that it can be used in the mask operation in place of (RX_BUF_SIZE – 1), so that my interrupt handler now becomes:

```C
__interrupt void RX_interrupt(void)
{
    static uint8_t rx_head = 0; /* offset into rx_buf[] where next character should be written */
    uint8_t rx_char;

    rx_char = HW_REG;           /* get the received character */

    rx_head &= RX_BUF_MASK;     /* mask the offset into the buffer */
    rx_buf[rx_head] = rx_char;  /* store the received char */
    ++rx_head;                  /* increment offset */
}
```

So is this effective C? I think so. It’s efficient, it’s flexible and its robustly protected against the sorts of bone headed mistakes that we all make from time to time.
