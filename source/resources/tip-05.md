# Effective C Tip #5 – Use pre-masking rather than post-masking

In this tip I’d like to offer a simple hint that can potentially make your buffer manipulation code a little more robust at essentially zero cost. I’d actually demonstrated the technique in this posting, but had not really emphasized its value.

Consider, for example, a receive buffer on a communications channel. The data are received a character at a time under interrupt and so the receive ISR needs to know where to place the next character. The question arises as to how best to do this? Now for performance reasons I usually make my buffer size a power of 2 such that I can use a simple mask operation. I then use an offset into the buffer to dictate where the next byte should be written. Code to do this typically looks something like this:

```C
#define RX_BUF_SIZE (32)
#define RX_BUF_MASK  (RX_BUF_SIZE - 1)
static uint8_t rx_buf[RX_BUF_SIZE];         /* receive buffer */
static uint8_t rx_head = 0;                 /* offset into rx_buf[] where next character should be written */

__interrupt void RX_interrupt(void)
{
    uint8_t rx_char;

    rx_char = HW_REG;           /* get the received character */
    rx_buf[rx_head] = rx_char;  /* store the received char */
    ++rx_head;                  /* increment offset */
    rx_head &= RX_BUF_MASK;     /* mask the offset into the buffer */
}
```

In the last couple of lines, I increment the value of RxHead and then mask it, with the intention of ensuring that the next write into rx_buf[] will be in the requisite range. The operative word here is ‘intention’. To see what I mean, consider what would happen if RxHead gets corrupted in some way. Now if the corruption is caused by RFI or some other such phenomenon then you are probably out of luck. However, what if RxHead gets unintentionally manipulated by a bug elsewhere in your code? As written, the manipulation may cause a write to occur beyond the end of the buffer – with all the attendant chaos that would inevitably arise. You can prevent this by simply doing the masking before indexing into the array. That is the code looks like this:

```C
__interrupt void RX_interrupt(void)
{
    uint8_t rx_char;

    rx_char = HW_REG;           /* get the received character */
    rx_head &= RX_BUF_MASK;     /* mask the offset into the buffer */
    rx_buf[rx_head] = rx_char;  /* store the received char */
    ++rx_head;                  /* increment offset */
}
```

What has this bought you? Well by coding it this way you guarantee that you will not index beyond the end of the array regardless of the value of RxHead when the ISR is invoked. Furthermore the guarantee comes at zero performance cost. Of course this hasn’t solved your problem with some other piece of code stomping on RxHead. However it does make finding the problem a lot easier because your problem will now be highly localized (i.e. data are received out of order) versus the system crashes randomly. The former class of problem is considerably easier to locate than is the latter.

So is this effective ‘C’. I think so. It’s a simple technique that adds a little robustness for free. I wouldn’t mind finding a few more like it.
