---
layout: page
title: "Notes"
author: "Dan Wysocki"
---

# Introduction

Languages we'll be using:

- C
- Lisp
- Scala
- Prolog
- Python

Topics:

- syntax
- semantics of functions, scopes, symbols
- composition
    - OOP, FP, declarative
- scripting
- pragmatics
    - range of use
    - design automation
    - performance

# Syntax

One thing every language shares is a set of rules for expressing a program.
They also have some set of semantics. The line `int a = 1` is legal syntax in
many languages (C, Java, etc.)

|--------------+--------------------------------------+-------------------|
| Syntax       | Semantics                            | Implementation    |
|--------------+--------------------------------------+-------------------|
| `int a = 1`  | There exists a storage location `a`, | allocate space,   |
|              | set its value to `1`                 | initialize to `1` |
|--------------|--------------------------------------|-------------------|



|--------------+------+--------------+------|
| Feature      | Java | C            | C++  |
|--------------+------+--------------+------|
| classes      | Yes  | No           | Yes  |
| objects      | Yes  | Not really   | Yes  |
| GC           | Yes  | No           | No   |
| scopes       | Yes  | Yes          | Yes  |
| locals       | Yes  | Yes          | Yes  |
| refrences    | Yes  | No, Pointers | Both |
| bad language | Yes  | No           | Yes  |
|--------------+------+--------------+------|

Syntactic specs

- tokens
    - `+`
    - `}`
    - `123`
    - `fred`
- grammar
    - context-free grammar
        - `S -> IfStat | AssignStat`
        - `IfStat -> if ( Exp ) S`
        - `AssignStat -> ID = Exp`
        - `Exp -> Var | Exp + Exp | Exp * Exp | (Exp)`
        - CFGs can be ambiguous in:
            - precedence
                - `a + b * c`
                    - `a + (b * c)`
                    - `(a + b) * c`
            - associativity
                - `a - b - c`
                    - `a - (b - c)`
                        - right associative
                    - `(a - b) - c`
                        - left associative (most common)
                - `a ** b ** c`
                    - `a ** (b ** c)`
                        - most common
                    - `(a ** b) ** c`
                - `a = b = c = 12`
                    - `((a = b) = c) = 12`
                        - doesn't make sense in most languages
                    - `a = (b = (c = 12))`
        - `Args -> Exp | Exp, Args`
        - `Stats -> Stat; | Stat; Stats`

Bitwise

- C was the first language to provide an interface to bitwise operations like
  people are used to today
    - `&` bitwise AND
    - `|` bitwise OR
    - `^` bitwise XOR
    - `~` bitwise NOT

{% highlight C %}
int a = 2;   /* 00000000000000000000000000000010 */
a |= 0x4;    /* 00000000000000000000000000000110 */
a |= 0xff00; /* 00000000000000001111111100000110 */
if ((a & 2) != 0) /* if 2nd bit is not 0 (ON) */

#define APPLES 0x2
if (a & APPLES) /* same thing */

/* unset bit */
a &= ~0x4    /* 00000000000000001111111100000010 */
{% endhighlight %}

{% highlight C %}
int
isodd(int x)
{
    return x & 0x1;
}

int
multiplyby2n(int x, int n)
{
    return x << n;
}
{% endhighlight %}

- `unsigned`

{% highlight C %}
unsigned int a = -1;
unsigned int b = -2;
(a < b) /* => true */
(a >> b) /* depends on whether a and b are signed/unsigned (confusing) */
{% endhighlight %}

- java learned from C's `>>` mistake and created two operators
    - `>>` signed bit shift right
    - `>>>` unsigned bit shift right
- logical operators
    - `&&`
    - `||`
    - tells you something about K+R's thinking, since they made the bitwise
      operators more convenient to write than the logical ones
    - some ambiguity with precidence
    - `if (a || b & c) /* vs */ if (a || (b & c)) /* vs */ if ((a || b) & c)`
- switch
    - the entire east coast AT&T network went down for a half a day once because
      of an error due to the switch syntax in C

{% highlight C %}
switch(a) {
    case 1: printf(...); /* you probably wanted to break here */
    case 2: return ...;
    default: break;
}
/*
    if a == 1, print, then return.
    if a == 2, return.
    else, break
 */
{% endhighlight %}

- referencing functions before they're defined
    - was too expensive when C was created

{% highlight C %}
int
f(int x)
{
    return g(x);
}

int
g(int x)
{
    return 12;
}

/* Not a fatal error but may as well be                     */
/* Can be fixed by adding `int g(int)` before declaring `f` */

int
main(char** argv, int argc) /* can just be main() */
{
    printf("%d", f(x));

    /* can return nothing, which means return whatever happened to be in
       the memory location that would have been used for an actual return
       value */
}
{% endhighlight %}

- forcing you to have declarations makes the compiler faster
- having all of those declarations at the head of your program sucks
- thus `.h` (header) files were born
    - all declarations get placed in a file with a `.h` extension
    - you include that file in your `.c` program with `#include file.h`

# C

{% highlight C %}
int a = 12;
{% endhighlight %}

valid C statement, but what's the context?

{% highlight C %}
static int b = 13; // global
void f() {
    int a = 12;    // local
    /* could also write `auto int a = 12;` to emphasize that it's local */
    g();
}

main() {
    f();
}
{% endhighlight %}

|-------|
| Stack |
|-------|
| main  |
| f     |
| g     |
|-------|

- Data Types
    - `long` / `long int`
        - has at least as many bits as `int`
        - C99 introduced `long long` which is guaranteed to be 64-bits
    - `int`
        - has at least as many bits as `short`
    - `short`
        - has at least as many bits as `char`
    - `char`
        - at least 8-bits
        - if a machine's byte is at least 8-bits, `char` is the size of a byte
        - C does no guarantees about the character encoding
    - `float`
        - has at least as many bits as `int` (just about always 32-bits)
    - `double`
        - twice the length of a float (probably?)
    - `long double`
        - has at least as many bits as `double`
- structs

{% highlight C %}
struct
point
{
    int x;
    int y;
}

struct
shape
{
    struct point origin;
    int length;
    /* ... */
}

void
draw(struct point *p)
{
    moveto(p->x, p->y);
    /* ... */
}

main()
{
    /* local struct, which goes away when `main` terminates */
    struct point pt;
    pt.x = foo; pt.y = bar;
    draw(&pt); /* unary & is the address-of operator */

    /* struct with allocated memory, which remains after `main` terminates */
    struct point *p = (struct point *) malloc(sizeof(struct point));
    p->x = foo; p->y = bar;
    draw(p);
    /* deallocate the memory used for `p` */
    free(p);
}
{% endhighlight %}

- pointers

{% highlight C %}
main()
{
    int a = 12, b = 17;
    swap(a, b);
}

/* does not work, because everything is local */
void
swap(int x, int y)
{
    int tmp = x;
    x = y;
    y = tmp;
}

/* does work, thanks to pointers */
void
swap(int *x, int *y)
{
    int tmp = *x;
    *x = y;
    *y = tmp;
}

/* now we have to change the call to `swap` */
main()
{
    int a = 12, b = 17;

    swap(&a, &b);
}
{% endhighlight %}

- you cannot write `swap` in Java, because you cannot manipulate pointers
- allowing pointer manipulation opens the door to a lot of bad things

{% highlight C %}
struct point * newOrigin() {
    struct point pt;
    pt.x = foo; pt.y = bar;
    return &pt; /* BOOM, security hole */
}
{% endhighlight %}

- never access the memory location of a local unless you're doing something
  like swap, because those locals disappear when the function terminates,
  and now you're treading in mirky waters

- arrays and pointers

{% highlight C %}
int a[10];

/* lookin good */
a[0] = 1;
/* still good */
a[1] = 4;
/* uh oh... */
a[29] = 6;
{% endhighlight %}

- this is all valid C code
- array bracket notation is actually syntactic sugar for pointer notation
- the following code is equivalent to the previous code

{% highlight C %}
int a[10];

*(&a + 0*sizeof(int))  = 1;
*(&a + 1*sizeof(int))  = 4;
*(&a + 29*sizeof(int)) = 6;
{% endhighlight %}

{% highlight C %}
int
sum(int a[], int n)
{
    int s = 0, i;

    for(i = 0; i < n; i++)
        s += a[i];

    return s;
}
/* equivalent */
int
sum(int *a, int n)
{
    int s = 0, i;

    for(i = 0; i < n; i++)
        s += *(a + i*sizeof(int));

    return s;
}
{% endhighlight %}

- pointers, arrays, and strings
    - no such thing as a string type
    - instead, we have the convention that a string is a `char` array,
      terminated by a null byte, (`0` or `'\0'` for emphasis)
    - `"hello" == { 'h', 'e', 'l', 'l', 'o', '\0' }`
    - `int contains(char x, const char *s)`
    - in Java we might write

{% highlight java %}
boolean contains(char x, String s) {
    for (int i = 0; i < s.length; s++) {
        if (s.charAt(i) == x)
            return true;
    }
    return false;
}
{% endhighlight %}

- in C we don't know the length ahead of time, so we would have to do
  something like

{% highlight C %}
int
contains(char x, const char *s)
{
    int i;
    for(i = 0; s[i] != 0; i++) {
        if (s[i] == x)
            return 1;
    }
    return 0;
}
{% endhighlight %}

- or with pointers we could do

{% highlight C %}
int
contains(char x, const char *s)
{
    while (*s) {
        if (*s++ == x)
            return 1;
    }
    return 0;
}
{% endhighlight %}

- string length

{% highlight C %}
assert strlen("hello") == 5;

int
strlen(char *s)
{
    int i;
    for (i = 0; *s++; i++);
    return i;
}

int
strlen(char *s)
{
    int i;
    while (*s++)
        i++;
    return i;
}
{% endhighlight %}

- standard libraries

{% highlight C %}
#include <std.h>
#include <stdio.h>
#include <stdib.h>
#include <unistd.h>
#include <string.h>
#include "myheader.h"
{% endhighlight %}

- `<string.h>`
    - `char * strsave(char *s)`, returns a new copy of a string

{% highlight C %}
char *
strsave(char *s)
{
    int n = strlen(s);
    char *p = (char *) malloc(sizeof(char) * (n+1));
    char *t = p;
    while (*t++ = *s++);
    return p;
}
{% endhighlight %}

- somewhere down the line, you probably won't need `p` anymore

{% highlight C %}
char *p = strsave(s);
/* ... */
free(p);
{% endhighlight %}

- `strcpy(char *src, char *dst)`, does the same thing as `strsave`, but
  using a provided destination string, instead of one it creates
- variadic functions
    - `varargs.h`
    - `void printf(formatter, ...)`
- unions

{% highlight C %}
union intOrFloat { int i; float f; }

intOrFloat x;
x.f = 1.0;
x.i = 3;
{% endhighlight %}

- bit fields
    - great when they work, but useless in a multithreading context

{% highlight C %}
struct
deviceRegister
{
    int dmaMode : 2;
    int status  : 3;
}
{% endhighlight %}

- typedefs

{% highlight C %}
typedef int pid_t; /* `_t` means "type that is not a pointer", by convention */
typedef unsigned int uint32; /* in <unistd.h> */
typedef struct cell * cell_p; /* `_p` means "pointer type", by convention */
{% endhighlight %}

- macros

{% highlight java %}
static final int STRING_CAP = 256
{% endhighlight %}

{% highlight C %}
#define STRING_CAP 256

/* what you see */
void f() {
    char line[STRING_CAP];
}

/* what the compiler sees */
void f() {
    char line[256];
}
{% endhighlight %}

{% highlight C %}
#define BEGIN {
#define END   }

void f() BEGIN
    char line[STRING_CAP];
END
{% endhighlight %}

{% highlight C %}
int x, y;
int z = x > y ? x : y;
{% endhighlight %}

{% highlight C %}
#define MAX(X, Y) X > Y ? X : Y

int x, y;
int z = MAX(x, y);
/* expands to */
int z = x > y ? x : y;
/* but */
int z = MAX(x++, y++);
/* expands to */
int z = x++ > y++ ? x++ : y++;
/* which increments one variable twice, and the other once */
{% endhighlight %}

{% highlight C %}
#define MAX(X, Y) do { _typeof(X) a=X; b=Y; a > b ? a : b } while(0)
/** defines local variables `a` and `b`, to avoid the whole `x++` issue,
 *  but now we have semicolons, as well as extra variables `a` and `b`,
 *  so we need to put it all in curly braces. Sometimes plain curly braces
 *  don't work, but putting it in a `do {} while(0)` always works.
 **/
{% endhighlight %}

{% highlight C %}
/* one last thing */
#define MAX(X, Y) (X) > (Y) ? (X) : (Y)
/* put parens around everything in the expansion, to avoid clashes like */
2+MAX(1,2);
/* which would expand to */
2+1 > 2 ? 1 : 2;
/* when we really want */
2+(1 > 2 ? 1 : 2);
{% endhighlight %}

{% highlight C %}
#define LINUX

#ifdef (LINUX)

    /* linux only stuff */

#else

    /* non-linux only stuff */

    #error

#endif
{% endhighlight %}

{% highlight bash %}
# you can run the C preprocessor on any language
$ gcc -E prog.java
# there's a standalone tool just for that
$ m4 prog.java
{% endhighlight %}

# Lisp

- two branches of programming languages
    - bottom-up
        - start with the machine, work a language out of that
        - C family of languages
    - top-down
        - start with a model, build a language from that, and finally implement
          it
        - Lisp, Haskell, Prolog
- early languages
    - [they were weird](
        http://bitsavers.trailing-edge.com/pdf/stanford/cs_techReports/STAN-CS-76-562_EarlyDevelPgmgLang_Aug76.pdf)

![](img/knights_of_lambda_calculus.png)

- Lambda Calculus
    - two main operations
        - abstraction `(λx.f(x))`
        - application `(λx.f(x))y`
    - `(λx.x)y => y` identity function
    - need some primitive operations in order to do anything other than
      the identity function, so let's assume we have `+` and `if` defined
    - `(λx.λy.y+x)ab => a+b`
- Lisp

{% highlight cl %}
> (defun id (x) x) ; abstraction form
ID
> (id 2)           ; application form
2
;; both are S-expressions
{% endhighlight %}

{% highlight cl %}
> (defun twice (x) (+ x x))
TWICE
> (twice 2)
4
{% endhighlight %}

{% highlight cl %}
> (defun add (x y) (+ x y))
ADD
> (add 4 2)
6
{% endhighlight %}

{% highlight cl %}
> (defun max (x y)
    (if (> x y)
      x
      y))
MAX
> (max 1 2)
2
> (max 2 1)
2
{% endhighlight %}

- who needs loops?

{% highlight cl %}
> (defun sum-of (n)
    (cond
      ((= n 0) 0)
      (T       (+ n (sum-of (1- n))))))
SUM-OF
> (sum-of 5)
15
{% endhighlight %}

- **lis**t **p**rocessing
    - `'(1 2)` - list of `1` and `2`
    - `'()` - empty list or `()` or `NIL`
    - `'(1)` - list of `1`

{% highlight cl %}
(defun count (l)
  (if (null l)
    0
    (1+ (count (rest l)))))
{% endhighlight %}

## REPL

- read
- eval
- print
- loop

{% highlight cl %}
(defun eval (exp)
  (cond
    ((is-self-evaluating exp)  exp)
    ((eq (car exp) 'quote)     (cdr exp))
    ((is-a-function (car exp)) (eval (subst (body (car exp))
                                            (cdr exp))))
    ((is-car  exp)             (first exp))
    ((is-cdr  exp)             (rest  exp))
    ((is-cond exp)             (for-each exp eval))
    (T                         (ERROR))))
{% endhighlight %}

## Side Effects

{% highlight cl %}
(setq glob 0)

(defun f (x)
  (progn
    (setq glob x) ; mortal sin
    (print x)     ; the only reason you should ever need progn
    (+ x 1)))
{% endhighlight %}

{% highlight cl %}
; can mutate freely
(setq glob 0)
; cannot mutate (implementation defined)
(defconstant BUFSIZE 1024)
; can only set once (implementation defined)
(defparameter retries 2)
{% endhighlight %}

- `setq`
    - in the original implementation of lisp
- `setf`
    - if the var is global:            `setq`
    - if the var is the car of a list: `set-car`
    - if the var is in a let: i dunno, set it anyway?

## Locals

{% highlight cl %}
(defun sq1 (x)
  "Squares x+1"
  (* (+ x 1)
     (+ x 1)))
; looks inelegant, so lets bind (+ x 1) to something
(defun sq1* (x)
  "Squares x+1"
  (let ((y (+ x 1)))
    (* y y)))
; let literally expands to the following (minus the naming and comments)
(defun sq1** (x)
  "Squares x+1"
  (sq** (+ x 1)))
(defun sq** (x)
  "Squares x"
  (* x x))
{% endhighlight %}

## Symbols, Bindings, Scopes

- symbols
    - identifiers
    - strings
    - names
- bindings
    - a mapping from symbol to value or storage location
    - in java we have
        - locals on the stack
        - static bindings
        - can't read dl's handwriting (there's a dynamic in there)
    - in lisp we have
        - either pointers to storage locations
        - or functions of no arguments which return a specific value
        - they're almost the same
    - lifetimes
        - scoped (death outside the scope)
        - static (permanent)
        - unbounded (garbage collected)
        - managed

{% highlight java %}
/* legal java */
class a {
  static int a;
  int a(int b) {
    int a = b;
    return a(a) + a.a;
  }
}
{% endhighlight %}

{% highlight cl %}
(setq a 2) ; doing the devil's work here
(defun f (a)
  (setq a 3))
(f a) => 3
a => 2
{% endhighlight %}

- this doesn't work as it does because of dynamic scoping, but because of
  rebinding

Aliasing

{% highlight java %}
class Point {
  int x, y;
  Point(int x, int y) {
    this.x = x;
    this.y = y;
  }
  /* nothing to see here */
  static Point addPoints(Point a, Point b) {
    return new Point(a.x + b.x,
                     a.y + b.y);
  }
  /* here's where things get weird */
  static void addPoints(Point a, Point b, Point dest) {
    dest.x  = a.x;
    dest.x += b.x;
    dest.y  = a.y;
    dest.y += b.y;
  }
}

class Main {
  public static void main(String[] args) {
    Point p = new Point(1,2);

    Point.addPoints(p, p, p);

    assert p.equals(new Point(2,4)); // addition works here
  }
}
{% endhighlight %}

{% highlight java %}
class Matrix {
  static void multiply(Matrix a, Matrix b, Matrix dest) {
    /* ... */
  }
}

class Main {
  public static void main(String[] args) {
    Matrix m = /* ... */;
    Matrix.multiply(m, m, m); /* will make garbage */
  }
}
{% endhighlight %}

Hygenic macros

{% highlight C %}
#define SWAP(X,Y) typeof(x) t = (x); x = (y); y = (t)

main () {
  int a = 2, b = 3;

  SWAP(a,b); /* fine */

  /* --- */

  int a = 2, t = 4;

  SWAP(a,t); /* ERROR: t already defined */
}
{% endhighlight %}

- we need our macros to use identifiers which are not used elsewhere
    - lisp has `gensym`

{% highlight cl %}
(defmacro head (l)
  `(car ,l))
{% endhighlight %}

- (`) backquote / backtick
- (`,`) unquote

{% highlight cl %}
;; (omit 2 '(1 2 3)) => (1 3)
(defun omit (elem lst)
  (when lst
    (let ((head (car lst)))
      (if (eq elem head)
        (omit elem (cdr lst))
        (cons head (omit (cdr lst)))))))
{% endhighlight %}

- tail recursion

{% highlight cl %}
;; non-tail recursive
(defun fact (x)
  (if (zerop x)
    1
    (* x (fact (1- x)))))
;; tail recursive
(defun fact* (x)
  (fact-iter* x 1))
(defun fact-iter* (x sofar)
  (if (zerop x)
    sofar
    (fact (1- x) (* x sofar))))
;; also tail recursive
(defun fact** (x)
  (fact-iter x 1 1))
(defun fact-iter** (bound count sofar)
  (if (= bound count)
    sofar
    (fact-iter** bound (1+ count) (* count sofar))))
{% endhighlight %}

{% highlight clojure %}
(defn fact [x]
  (loop [x x, sofar 1]
    (if (zero? x)
      sofar
      (recur (dec x) (* x sofar)))))
{% endhighlight %}

## Types

what are they?

ideas:

- description of a location
- tells you what you can do
- permanent attribute?
- preconditions and post conditions

{% highlight cl %}
;; LISP ;;
(* 1 "hi mom") ;; run-time error ;;
{% endhighlight %}

{% highlight java %}
/* JAVA */
1 * "hi mom" /* compile-time error */
{% endhighlight %}

{% highlight python %}
''' PYTHON '''
1 * "hi mom" # evaluates to "hi mom"
{% endhighlight %}



