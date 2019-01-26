# Effective C Tip #4 – Prototyping static functions

I have previously talked about the benefits of static functions. Now I’m addressing where to place static functions in a module. This posting is motivated by the fact that I’ve recently spent a considerable amount of time wading through code that locates its static functions at the top of the file. That is the code looks like this:

```C
static void fna(void) {...}

static void fnb(uint16_t a) {...}

...

static uint16_t fnc(void) {...}

void fn_public(void)
{
    uint16_t t;

    fna();
    t = fnc();
    fnb(t);
    ...
}
```

In this approach (which unfortunately seems to be the more common), all of the static functions are defined at the top of the module, and the public functions appear at the bottom. I’ve always strongly disliked this approach because it forces someone that is browsing the code to wade through all the minutiae of the implementation before they get to the big picture public functions. This can be very tedious in a file with a large number of static functions. The problem is compounded by the fact that it’s very difficult to search for a non static function. Yes I’m sure I could put together a regular expression search to do it – but it requires what I consider to be unnecessary work.

A far better approach is as follows. Prototype (declare) all the static functions at the top of the module. Then follow the prototypes with the public functions (thus making them very easy to locate) and then place the static functions out of the way at the end of the file. If I do this, my code example now looks like this:

```C
static void fna(void);
static void fnb(uint16_t a);
static uint16_t fnc(void);

void fn_public(void)
{
    uint16_t t;

    fna();
    t = fnc();
    fnb(t);
    ...
}

static void fna(void) {...}

static void fnb(uint16_t a) {...}

...

static uint16_t fnc(void) {...}
```

If you subscribe to the belief that we only write source code so that someone else can read it then this simple change to your coding style can have immense benefits to the person that has to maintain your code (including a future version of yourself).
