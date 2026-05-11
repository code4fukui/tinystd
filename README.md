# tinystd

> 日本語のREADMEはこちらです: [README.ja.md](README.ja.md)

A tiny, self-contained standard library for C in WebAssembly (WASM) environments.

`tinystd` provides essential C standard library functions needed to compile C projects into compact WASM modules. It avoids dependencies on a full C standard library (libc) and stubs out I/O operations to minimize binary size.

## Features

-   **Minimal Footprint:** Designed specifically for small WASM file sizes.
-   **Memory Management:** Basic `malloc`, `free`, `calloc`, `memcpy`, and `memset` implementations.
-   **String Utilities:** A core set of string functions including `strlen`, `strspn`, `strcspn`, `strchr`, `strcasecmp`, and `memchr`.
-   **Flexible Math Library:**
    -   Includes a pure C implementation of `pow` (from `fdlibm`), `sqrt`, `atan2`, and `fabs`.
    -   Optionally, can be compiled with `USE_JS_MATH` to import math functions (`pow`, `atan2`, `cos`, `sin`, etc.) directly from the JavaScript host environment, further reducing size and potentially improving performance.
-   **Linker-Friendly Stubs:** Provides no-op stubs for `stdio.h` functions (`fprintf`, etc.) and `assert.h`, allowing code that uses them to compile without including I/O or assertion logic.
-   **Standard Types:** Includes `stdint.h` for standard integer types.

## Usage

### 1. Add as a Submodule

Add `tinystd` to your project, for example as a Git submodule:

```sh
touch .gitmodules
git submodule add https://github.com/code4fukui/tinystd
```

### 2. Include and Compile

Include the required headers in your source files:
```c
#include "tinystd/stdlib.h"
#include "tinystd/string.h"
#include "tinystd/math.h"
```

Compile your project with the necessary `tinystd` source files.

**Example with Emscripten (Pure C/WASM):**
```sh
emcc my_app.c tinystd/stdlib.c tinystd/string.c tinystd/math.c tinystd/pow.c -o my_app.js
```

### 3. (Optional) Use JavaScript Math Imports

To use the host environment's math functions, define the `USE_JS_MATH` preprocessor flag. This will replace C implementations with imports from JavaScript.

**Example with Emscripten (`USE_JS_MATH`):**
```sh
emcc my_app.c tinystd/stdlib.c tinystd/string.c tinystd/math.c -D USE_JS_MATH -o my_app.js
```
*Note: When using this flag, `pow.c` is not needed.*

## API Reference

### `stdlib.h`
```c
void* memset(void* p, int len, unsigned long n);
void* memcpy(void* p1, const void* p2, unsigned long len);
void* malloc(unsigned long len);
void free(void* p);
void* calloc(unsigned long len, unsigned long size);
```

### `string.h`
```c
unsigned long strlen(const char* s);
unsigned long strspn(const char* s1, const char* s2);
unsigned long strcspn(const char* s1, const char* s2);
char* strchr(const char* s, int n);
int strcasecmp(const char *, const char *);
void* memchr(const void *, int, unsigned long);
```

### `math.h`
Provides common math functions. The implementation of `pow`, `atan2`, `cos`, `sin`, etc., depends on the `USE_JS_MATH` flag.
```c
double sqrt(double f);
double pow(double f, double n);
double atan2(double y, double x);
float atan2f(float y, float x);
double fabs(double n);
int isnan(float f);
```

### `stdio.h`
Provides stubs for common I/O functions to allow compilation. All output is discarded.
```c
#define fprintf(...) (void)0
#define fputc(...) (void)0
#define stdout (FILE*)0
#define stderr (FILE*)0
```

## Attribution

The `pow()` implementation in `pow.c` is derived from the `fdlibm` library, available at `https://www.netlib.org/fdlibm/e_pow.c`.

```
====================================================
Copyright (C) 2004 by Sun Microsystems, Inc. All rights reserved.

Permission to use, copy, modify, and distribute this
software is freely granted, provided that this notice
is preserved.
====================================================
```

## License

MIT License