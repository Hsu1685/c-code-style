# Recommended C style and coding rules

This document describes C code style used by Tilen MAJERLE in his projects and libraries.

## Table of Contents

  - [The single most important rule](#the-single-most-important-rule)
  - [Recommended C style and coding rules](#recommended-c-style-and-coding-rules)
  - [General rules](#general-rules)
  - [Comments](#comments)
  - [Functions](#functions)
  - [Variables](#variables)
  - [Structures, enumerations, typedefs](#structures-enumerations-typedefs)
  - [Compound statements](#compound-statements)
    - [Switch statement](#switch-statement)
  - [Macros and preprocessor directives](#macros-and-preprocessor-directives)
  - [Documentation](#documentation)
  - [Header/source files](#headersource-files)
  - [Artistic Style configuration](#artistic-style-configuration)
  - [Eclipse formatter](#eclipse-formatter)

## The single most important rule

Let's start with the quote from [GNOME developer](https://developer.gnome.org/programming-guidelines/stable/c-coding-style.html.en) site.

> The single most important rule when writing code is this: *check the surrounding code and try to imitate it*.
>
> As a maintainer it is dismaying to receive a patch that is obviously in a different coding style to the surrounding code. This is disrespectful, like someone tromping into a spotlessly-clean house with muddy shoes.
>
> So, whatever this document recommends, if there is already written code and you are patching it, keep its current style consistent even if it is not your favorite style.

## General rules

Here are listed most obvious and important general rules. Please check them carefully before you continue with other chapters.

- 使用 `C99` 標準
- 不要使用 tabs, 而是使用空格
- 每個縮排級別使用 `4` 個空格
- 在關鍵字和左括號之間使用 `1` 個空格
```c
/* OK */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)

/* Wrong */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

- 在函數名和左小括號之間不要使用空格
```c
int32_t a = sum(4, 3);              /* OK */
int32_t a = sum (4, 3);             /* Wrong */
```

- 不要在變數/函數/聚集/類型中使用 `__` 或 `_` 前綴。這是為C語言本身所保留的
    - 對於嚴格的模組私有函數，使用 `prv_` 名稱前綴
- 對有下畫線 `_` 的變數/函數/聚集/類型只使用小寫字母
- 左大括號總是和關鍵字 (`for`, `while`, `do`, `switch`, `if`, ...) 在同一行
> [補充] 右小括號和左大括號中間使用單空格
```c
size_t i;
for (i = 0; i < 5; ++i) {           /* OK */
}
for (i = 0; i < 5; ++i){            /* Wrong */
}
for (i = 0; i < 5; ++i)             /* Wrong */
{
}
```

- 在比較運算子 (comparison operators) 和指派運算子 (assignment operators) 的前和後使用單空格
```c
int32_t a;
a = 3 + 4;              /* OK */
for (a = 0; a < 5; ++a) /* OK */
a=3+4;                  /* Wrong */
a = 3+4;                /* Wrong */
for (a=0;a<5;++a)       /* Wrong */
```

- 在每個逗號後使用單空格
```c
func_name(5, 4);        /* OK */
func_name(4,3);         /* Wrong */
```

- 不要初始化 `static` 和 `global` 變數為 `0` (或是 `NULL`)，讓編譯器來做
```c
static int32_t a;       /* OK */
static int32_t b = 4;   /* OK */
static int32_t a = 0;   /* Wrong */

void
my_func(void) {
    static int32_t* ptr;/* OK */
    static char abc = 0;/* Wrong */
}
```

- 在同一行中宣告所有相同類型的local variables
```c
void
my_func(void) {
    char a;             /* OK */
    char a, b;          /* OK */
    char b;             /* Wrong, variable with char type already exists */
}
```

- 按順序宣告局部變數 (local variables)
    1. 自訂的結構 (structures) 和 枚舉 (enumerations)
    2. 整數類型，範圍更大的的型態優先
    3. 單浮點數/雙浮點數 (Single/Double floating point)
> [補充] (2)類型範圍相同時，無號的優先
```c
int
my_func(void) {
    /* 1 */
    my_struct_t my;     /* First custom structures */
    my_struct_ptr_t* p; /* Pointers too */

    /* 2 */
    uint32_t a;
    int32_t b;
    uint16_t c;
    int16_t g;
    char h;
    /* ... */

    /* 3 */
    double d;
    float f;
}
```

- 總是在區塊的開頭宣告局部變數，在第一個執行語句之前

- 在 `for` 循環之中宣告計數變數
> [補充] 在for之中宣告，除非你後續需要這個計數這個變數
```c
/* OK */
for (size_t i = 0; i < 10; ++i)

/* OK, if you need counter variable later */
size_t i;
for (i = 0; i < 10; ++i) {
    if (...) {
        break;
    }
}
if (i == 10) {

}

/* Wrong */
size_t i;
for (i = 0; i < 10; ++i) ...
```

- 避免在宣告中調用函數來給值，除非是單一變量
```c
void
a(void) {
    /* Avoid function calls when declaring variable */
    int32_t a, b = sum(1, 2);

    /* Use this */
    int32_t a, b;
    b = sum(1, 2);

    /* This is ok */
    uint8_t a = 3, b = 4;
}
```

- 除了 `char`、`float` 或 `double` 以外，總是使用 `stdint.h` 函數庫內的類型來宣告，例如 `uint8_t` 是為 `unsigned 8-bit`等等
- 不要使用 `stdbool.h` 函數庫。使用 `1` 或 `0` 來表示 `true` 或 `false`
```c
/* OK */
uint8_t status;
status = 0;

/* Wrong */
#include <stdbool.h>
bool status = true;
```

- 永遠不與 `true`做比較，例如不使用 `if (check_func() == 1)`，而使用 `if (check_func()) { ... }`
- 總是將指針與 `NULL` 值做比較
```c
void* ptr;

/* ... */

/* OK, compare against NULL */
if (ptr == NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

- 總是使用前遞增/遞減 *pre-increment (and decrement respectively)* 取代後遞增/遞減 *post-increment (and decrement respectively)*
```c
int32_t a = 0;
...

a++;            /* Wrong */
++a;            /* OK */

for (size_t j = 0; j < 10; ++j) {}  /* OK */
```

- 總是使用 `size_t` 作為長度或大小的變數
- 如果函數不應該修改指針所指的記憶體，總是使用 `const` 來修飾 `pointer`
- 如果函數參數/形式參數 (parameter)或變數 (variable)不應該被修改，總是使用 `const` 來修飾
```c

/* When d could be modified, data pointed to by d could not be modified */
void
my_func(const void* d) {

}

/* When d and data pointed to by d both could not be modified */
void
my_func(const void* const d) {

}

/* Not required, it is advised */
void
my_func(const size_t len) {

}

/* When d should not be modified inside function, only data pointed to by d could be modified */
void
my_func(void* const d) {

}
```

- 當函數可以接受任何類型的指針時總是使用 `void *`，不要使用 `uint8_t *`
    - 函數在實現時必須注意正確的類型轉換
```c
/*
 * To send data, function should not modify memory pointed to by `data` variable
 * thus `const` keyword is important
 *
 * To send generic data (or to write them to file)
 * any type may be passed for data,
 * thus use `void *`
 */
/* OK example */
void
send_data(const void* data, size_t len) { /* OK */
    /* Do not cast `void *` or `const void *` */
    const uint8_t* d = data;/* Function handles proper type for internal usage */
}

void
send_data(const void* data, int len) {    /* Wrong, not not use int */
}
```

- 總是使用括號和 `sizeof` 運算子
- 不要使用變數長度陣列 *Variable Length Array* (VLA)。使用動態記憶體分配來來取代標準 C `malloc` 和 `free` 函數或者如果函數庫/專案提供了自定記憶體配置，使用它來實現
    - 參考 [LwMEM](https://github.com/MaJerle/lwmem)，是一個自訂記憶體管理庫
```c
/* OK */
#include <stdlib.h>
void
my_func(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * n); /* OK, Allocate memory */
    arr = malloc(sizeof *arr * n);  /* Wrong, brackets for sizeof operator are missing */
    if (arr == NULL) {
        /* FAIL, no memory */
    }

    free(arr);  /* Free memory after usage */
}

/* Wrong */
void
my_func(size_t size) {
    int32_t arr[size];  /* Wrong, do not use VLA */
}
```

- 總是將變數 (variable)和 0 做比較，除非它被視為 `boolean` 類型
- 不要把 `boolean-treated` 變數和 0 或 1 做比較。用 NOT (`!`) 來取代
```c
size_t length = 5;  /* Counter variable */
uint8_t is_ok = 0;  /* Boolean-treated variable */
if (length)         /* Wrong, length is not treated as boolean */
if (length > 0)     /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */
if (length == 0)    /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */

if (is_ok)          /* OK, variable is treated as boolean */
if (!is_ok)         /* OK, -||- */
if (is_ok == 1)     /* Wrong, never compare boolean variable against 1! */
if (is_ok == 0)     /* Wrong, use ! for negative check */
```

- 總是使用 `/* comment */` 作為註釋，即使是 *single-line* 單行註釋
- 在標頭檔中總是包含帶有 `extern` 關鍵字的`C++` 檢查
- 每個函數必須包含 *doxygen-enabled* 註釋，即使是 `static` 的
- 使用英文名稱/文件的函數、變數、註釋
- 變數使用 *lowercase* 小寫字母
- 如果變數包含多個名稱，請使用底線 *underscore*，例如使用 `force_redraw`。不要使用 `forceRedraw`
- Never cast function returning `void *`, eg. `uint8_t* ptr = (uint8_t *)func_returning_void_ptr();` as `void *` is safely promoted to any other pointer type
    - Use `uint8_t* ptr = func_returning_void_ptr();` instead
- 對於C標準函數庫的 include 檔案，總是使用 `<` 和 `>`，例如 `#include <stdlib.h>`
- 對於自定義函數庫，總是使用 `""` ，例如 `#include "my_library.h"`
- 當要轉換為指針類型時，總是將 asterisk 星號與類型對齊，例如 `uint8_t* t = (uint8_t*)var_width_diff_type`
- 總是尊重專案項目或函數庫中已經使用的代碼風格

## Comments

- 不允許以 `//` 為開頭的註釋，總是使用 `/* comment */`，即使是單行註釋
```c
//This is comment (wrong)
/* This is comment (ok) */
```

- 對於多行註釋，每行使用 `space+asterisk` 空格+星號
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/**
 * Wrong, use double-asterisk only for doxygen documentation
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

- Use `12` indents (`12 * 4` spaces) offset when commenting. If statement is larger than `12` indents, make comment `4-spaces` aligned (examples below) to next available indent
```c
void
my_func(void) {
    char a, b;
                                                /* 下一行的註釋從開頭開始算是12 * 4個空格 */
    a = call_func_returning_char_a(a);          /* This is comment with 12*4 spaces indent from beginning of line */
    b = call_func_returning_char_a_but_func_name_is_very_long(a);   /* This is comment, aligned to 4-spaces indent */
                                                                    /* 上一行的註釋從開頭是17 * 4個空格，因為>12所以到下一個可用的縮排
}
```

## Functions

- Every function which may have access from outside its module, must include function *prototype* (or *declaration*)
- Function name must be lowercase, optionally separated with underscore `_` character
```c
/* OK */
void my_func(void);
void myfunc(void);

/* Wrong */
void MYFunc(void);
void myFunc();
```

- When function returns pointer, align asterisk to return type
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t a, int32_t b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```
- Align all function prototypes (with the same/similar functionality) for better readability
```c
/* OK, function names aligned */
void        set(int32_t a);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t a);
const char * get(void);
```

- Function implementation must include return type and optional other keywords in separate line
```c
/* OK */
int32_t
foo(void) {
    return 0;
}

/* OK */
static const char*
get_string(void) {
    return "Hello world!\r\n";
}

/* Wrong */
int32_t foo(void) {
    return 0;
}
```

## Variables

- Make variable name all lowercase with optional underscore `_` character
```c
/* OK */
int32_t a;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t A;
int32_t myVar;
int32_t MYVar;
```

- Group local variables together by `type`
```c
void
foo(void) {
    int32_t a, b;   /* OK */
    char a;
    char b;         /* Wrong, char type already exists */
}
```

- Do not declare variable after first executable statement
```c
void
foo(void) {
    int32_t a;
    a = bar();
    int32_t b;      /* Wrong, there is already executable statement */
}
```

- You may declare new variables inside next indent level
```c
int32_t a, b;
a = foo();
if (a) {
    int32_t c, d;   /* OK, c and d are in if-statement scope */
    c = foo();
    int32_t e;      /* Wrong, there was already executable statement inside block */
}
```

- Declare pointer variables with asterisk aligned to type
```c
/* OK */
char* a;

/* Wrong */
char *a;
char * a;
```

- When declaring multiple pointer variables, you may declare them with asterisk aligned to variable name
```c
/* OK */
char *p, *n;
```

## Structures, enumerations, typedefs

- Structure or enumeration name must be lowercase with optional underscore `_` character between words
- Structure or enumeration may contain `typedef` keyword
- All structure members must be lowercase
- All enumeration members must be uppercase
- Structure/enumeration must follow doxygen documentation syntax

When structure is declared, it may use one of `3` different options:

1. When structure is declared with *name only*, it *must not* contain `_t` suffix after its name.
```c
struct struct_name {
    char* a;
    char b;
};
```
2. When structure is declared with *typedef only*, it *has to* contain `_t` suffix after its name.
```c
typedef struct {
    char* a;
    char b;
} struct_name_t;
```
3. When structure is declared with *name and typedef*, it *must not* contain `_t` for basic name and it *has to* contain `_t` suffix after its name for typedef part.
```c
typedef struct struct_name {
    char* a;
    char b;
    char c;
} struct_name_t;
```

Examples of bad declarations and their suggested corrections
```c
/* a and b must be separated to 2 lines */
/* Name of structure with typedef must include _t suffix */
typedef struct {
    int32_t a, b;
} a;

/* Corrected version */
typedef struct {
    int32_t a;
    int32_t b;
} a_t;

/* Wrong name, it must not include _t suffix */
struct name_t {
    int32_t a;
    int32_t b;
};

/* Wrong parameters, must be all uppercase */
typedef enum {
    MY_ENUM_TESTA,
    my_enum_testb,
} my_enum_t;
```

- When initializing structure on declaration, use `C99` initialization style
```c
/* OK */
a_t a = {
    .a = 4,
    .b = 5,
};

/* Wrong */
a_t a = {1, 2};
```

- When new typedef is introduced for function handles, use `_fn` suffix
```c
/* Function accepts 2 parameters and returns uint8_t */
/* Name of typedef has `_fn` suffix */
typedef uint8_t (*my_func_typedef_fn)(uint8_t p1, const char* p2);
```

## Compound statements

- Every compound statement must include opening and closing curly bracket, even if it includes only `1` nested statement
- Every compound statement must include single indent; when nesting statements, include `1` indent size for each nest
```c
/* OK */
if (c) {
    do_a();
} else {
    do_b();
}

/* Wrong */
if (c)
    do_a();
else
    do_b();

/* Wrong */
if (c) do_a();
else do_b();
```

- In case of `if` or `if-else-if` statement, `else` must be in the same line as closing bracket of first statement
```c
/* OK */
if (a) {

} else if (b) {

} else {

}

/* Wrong */
if (a) {

}
else {

}

/* Wrong */
if (a) {

}
else
{

}
```

- In case of `do-while` statement, `while` part must be in the same line as closing bracket of `do` part
```c
/* OK */
do {
    int32_t a;
    a = do_a();
    do_b(a);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

- Indentation is required for every opening bracket
```c
if (a) {
    do_a();
} else {
    do_b();
    if (c) {
        do_c();
    }
}
```

- Never do compound statement without curly bracket, even in case of single statement. Examples below show bad practices
```c
if (a) do_b();
else do_c();

if (a) do_a(); else do_b();
```

- Empty `while`, `do-while` or `for` loops must include brackets
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

- If `while` (or `for`, `do-while`, etc) is empty (it can be the case in embedded programming), use empty single-line brackets
```c
/* Wait for bit to be set in embedded hardware unit
uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* Wait bit 13 to be ready */
while (*addr & (1 << 13)) {}        /* OK, empty loop contains no spaces inside curly brackets */
while (*addr & (1 << 13)) { }       /* Wrong */
while (*addr & (1 << 13)) {         /* Wrong */

}
while (*addr & (1 << 13));          /* Wrong, curly brackets are missing. Can lead to compiler warnings or unintentional bugs */
```
- Always prefer using loops in this order: `for`, `do-while`, `while`
- Avoid incrementing variables inside loop block if possible, see examples

```c
/* Not recommended */
int32_t a = 0;
while (a < 10) {
    .
    ..
    ...
    ++a;
}

/* Better */
for (size_t a = 0; a < 10; ++a) {

}

/* Better, if inc may not happen in every cycle */
for (size_t a = 0; a < 10; ) {
    if (...) {
        ++a;
    }
}
```

### Switch statement

- Add *single indent* for every `case` statement
- Use additional *single indent* for `break` statement in each `case` or `default`
```c
/* OK, every case has single indent */
/* OK, every break has additional indent */
switch (check()) {
    case 0:
        do_a();
        break;
    case 1:
        do_b();
        break;
    default:
        break;
}

/* Wrong, case indent missing */
switch (check()) {
case 0:
    do_a();
    break;
case 1:
    do_b();
    break;
default:
    break;
}

/* Wrong */
switch (check()) {
    case 0:
        do_a();
    break;      /* Wrong, break must have indent as it is under case */
    case 1:
    do_b();     /* Wrong, indent under case is missing */
    break;
    default:
        break;
}
```

- Always include `default` statement
```c
/* OK */
switch (var) {
    case 0:
        do_job();
        break;
    default:
        break;
}

/* Wrong, default is missing */
switch (var) {
    case 0:
        do_job();
        break;
}
```

- If local variables are required, use curly brackets and put `break` statement inside.
    - Put opening curly bracket in the same line as `case` statement
```c
switch (a) {
    /* OK */
    case 0: {
        int32_t a, b;
        char c;
        a = 5;
        /* ... */
        break;
    }

    /* Wrong */
    case 1:
    {
        int32_t a;
        break;
    }

    /* Wrong, break shall be inside */
    case 2: {
        int32_t a;
    }
    break;
}
```

## Macros and preprocessor directives

- Always use macros instead of literal constants, specially for numbers
- All macros must be fully uppercase, with optional underscore `_` character, except if they are clearly marked as function which may be in the future replaced with regular function syntax
```c
/* OK */
#define MY_MACRO(x)         ((x) * (x))

/* Wrong */
#define square(x)           ((x) * (x))
```

- Always protect input parameters with parentheses
```c
/* OK */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))

/* Wrong */
#define MIN(x, y)           x < y ? x : y
```

- Always protect final macro evaluation with parenthesis
```c
/* Wrong */
#define MIN(x, y)           (x) < (y) ? (x) : (y)
#define SUM(x, y)           (x) + (y)

/* Imagine result of this equation using wrong SUM implementation */
int32_t x = 5 * SUM(3, 4);  /* Expected result is 5 * 7 = 35 */
int32_t x = 5 * (3) + (4);  /* It is evaluated to this, final result = 19 which is not what we expect */

/* Correct implementation */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))
#define SUM(x, y)           ((x) + (y))
```

- When macro uses multiple statements, protect it using `do-while (0)` statement
```c
typedef struct {
    int32_t px, py;
} point_t;
point_t p;                  /* Define new point */

/* Wrong implementation */

/* Define macro to set point */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)    /* 2 statements. Last one should not implement semicolon */

SET_POINT(&p, 3, 4);        /* Set point to position 3, 4. This evaluates to... */
(&p)->px = (3); (&p)->py = (4); /* ... to this. In this example this is not a problem. */

/* Consider this ugly code, however it is valid by C standard (not recommended) */
if (a)                      /* If a is true */
    if (b)                  /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    else
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */

/* Evaluates to code below. Do you see the problem? */
if (a)
    if (b)
        (&p)->px = (3); (&p)->py = (4);
    else
        (&p)->px = (5); (&p)->py = (6);

/* Or if we rewrite it a little */
if (a)
    if (b)
        (&p)->px = (3);
        (&p)->py = (4);
    else
        (&p)->px = (5);
        (&p)->py = (6);

/*
 * Ask yourself a question: To which `if` statement `else` keyword belongs?
 *
 * Based on first part of code, answer is straight-forward. To inner `if` statement when we check `b` condition
 * Actual answer: Compilation error as `else` belongs nowhere
 */

/* Better and correct implementation of macro */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)    /* 2 statements. No semicolon after while loop */
/* Or even better */
#define SET_POINT(p, x, y)  do {    \   /* Backslash indicates statement continues in new line */
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)                             /* 2 statements. No semicolon after while loop */

/* Now original code evaluates to */
if (a)
    if (b)
        do { (&p)->px = (3); (&p)->py = (4); } while (0);
    else
        do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* Every part of `if` or `else` contains only `1` inner statement (do-while), hence this is valid evaluation */

/* To make code perfect, use brackets for every if-ifelse-else statements */
if (a) {                    /* If a is true */
    if (b) {                /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    } else {
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */
    }
}
```

- Avoid using `#ifdef` or `#ifndef`. Use `defined()` or `!defined()` instead
```c
#ifdef XYZ
/* do something */
#endif /* XYZ */
```

- Always document `if/elif/else/endif` statements
```c
/* OK */
#if defined(XYZ)
/* Do if XYZ defined */
#else /* defined(XYZ) */
/* Do if XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
/* Do if XYZ defined */
#else
/* Do if XYZ not defined */
#endif
```

- Do not indent sub statements inside `#if` statement
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* do when ABC defined */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* Do when XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
    #if defined(ABC)
        /* do when ABC defined */
    #endif /* defined(ABC) */
#else /* defined(XYZ) */
    /* Do when XYZ not defined */
#endif /* !defined(XYZ) */
```

## Documentation

Documented code allows doxygen to parse and general html/pdf/latex output, thus it is very important to do it properly.

- Use doxygen-enabled documentation style for `variables`, `functions` and `structures/enumerations`
- Always use `\` for doxygen, do not use `@`
- Always use `5x4` spaces (`5` tabs) offset from beginning of line for text
```c
/**
 * \brief           Holds pointer to first entry in linked list
 *                  Beginning of this text is 5 tabs (20 spaces) from beginning of line
 */
static
type_t* list;
```

- Every structure/enumeration member must include documentation
- Use `12x4 spaces` offset for beginning of comment
```c
/**
 * \brief           This is point struct
 * \note            This structure is used to calculate all point
 *                      related stuff
 */
typedef struct {
    int32_t x;                                  /*!< Point X coordinate */
    int32_t y;                                  /*!< Point Y coordinate */
    int32_t size;                               /*!< Point size.
                                                    Since comment is very big,
                                                    you may go to next line */
} point_t;

/**
 * \brief           Point color enumeration
 */
typedef enum {
    COLOR_RED,                                  /*!< Red color. This comment has 12x4
                                                    spaces offset from beginning of line */
    COLOR_GREEN,                                /*!< Green color */
    COLOR_BLUE,                                 /*!< Blue color */
} point_color_t;
```

- Documentation for functions must be written in function implementation (source file usually)
- Function must include `brief` and all parameters documentation
- Every parameter must be noted if it is `in` or `out` for *input* and *output* respectively
- Function must include `return` parameter if it returns something. This does not apply for `void` functions
- Function can include other doxygen keywords, such as `note` or `warning`
- Use colon `:` between parameter name and its description
```c
/**
 * \brief           Sum `2` numbers
 * \param[in]       a: First number
 * \param[in]       b: Second number
 * \return          Sum of input values
 */
int32_t
sum(int32_t a, int32_t b) {
    return a + b;
}

/**
 * \brief           Sum `2` numbers and write it to pointer
 * \note            This function does not return value, it stores it to pointer instead
 * \param[in]       a: First number
 * \param[in]       b: Second number
 * \param[out]      result: Output variable used to save result
 */
void
void_sum(int32_t a, int32_t b, int32_t* result) {
    *result = a + b;
}
```

- If function returns member of enumeration, use `ref` keyword to specify which one
```c
/**
 * \brief           My enumeration
 */
typedef enum {
    MY_ERR,                                     /*!< Error value */
    MY_OK                                       /*!< OK value */
} my_enum_t;

/**
 * \brief           Check some value
 * \return          \ref MY_OK on success, member of \ref my_enum_t otherwise
 */
my_enum_t
check_value(void) {
    return MY_OK;
}
```

- Use notation (\`NULL\` => `NULL`) for constants or numbers
```c
/**
 * \brief           Get data from input array
 * \param[in]       in: Input data
 * \return          Pointer to output data on success, `NULL` otherwise
 */
const void *
get_data(const void* in) {
    return in;
}
```

- Documentation for macros must include `hideinitializer` doxygen command
```c
/**
 * \brief           Get minimal value between `x` and `y`
 * \param[in]       x: First value
 * \param[in]       y: Second value
 * \return          Minimal value between `x` and `y`
 * \hideinitializer
 */
#define MIN(x, y)       ((x) < (y) ? (x) : (y))
```

## Header/source files

- Leave single empty line at the end of file
- Every file must include doxygen annotation for `file` and `brief` description followed by empty line (when using doxygen)
```c
/**
 * \file            template.h
 * \brief           Template include file
 */
                    /* Here is empty line */
```

- Every file (*header* or *source*) must include license (opening comment includes single asterisk as this must be ignored by doxygen)
- Use the same license as already used by project/library
```c
/**
 * \file            template.h
 * \brief           Template include file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
```

- Header file must include guard `#ifndef`
- Header file must include `C++` check
- Include external header files outside `C++` check
- Include external header files with STL C files first followed by application custom files
- Header file must include only every other header file in order to compile correctly, but not more (.c should include the rest if required)
- Header file must only expose module public variables/types/functions
- Use `extern` for global module variables in header file, define them in source file later
```
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* This is global variable declaration in header */

#endif

/* file.c ... */
int32_t my_variable;        /* Actually defined in source */
```
- Never include `.c` files in another `.c` file
- `.c` file should first include corresponding `.h` file, later others, unless otherwise explicitly necessary
- Do not include module private declarations in header file

- Header file example (no license for sake of an example)
```c
/* License comes here */
#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* Include headers */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/* File content here */

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */
```

## Artistic style configuration

[AStyle](http://astyle.sourceforge.net/) is a great piece of software that can
help with formatting the code based on input configuration.

This repository contains `astyle-code-format.cfg` file which can be used with `AStyle` software.

```
astyle --options="astyle-code-format.cfg" "input_path/*.c,*.h" "input_path2/*.c,*.h"
```

## Eclipse formatter

Repository contains `eclipse-ext-kr-format.xml` file that can be used with
eclipse-based toolchains to set formatter options.

It is based on K&R formatter with modifications to respect above rules.
You can import it within eclipse settings, `Preferences -> LANGUAGE -> Code Style -> Formatter` tab.
