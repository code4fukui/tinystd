# tinystd

WebAssembly (WASM) 環境向けの、非常に小さく自己完結型のC標準ライブラリです。

`tinystd` は、CプロジェクトをコンパクトなWASMモジュールにコンパイルするために必要な、基本的なC標準ライブラリ関数を提供します。完全なC標準ライブラリ（libc）への依存を避け、I/O操作をスタブ化することでバイナリサイズを最小限に抑えます。

## 機能

- **最小限のフットプリント:** WASMのファイルサイズを小さくすることに特化した設計。
- **メモリ管理:** 基本的な `malloc`、`free`、`calloc`、`memcpy`、`memset` の実装。
- **文字列ユーティリティ:** `strlen`、`strspn`、`strcspn`、`strchr`、`strcasecmp`、`memchr` などのコアな文字列関数群。
- **柔軟な数学ライブラリ:**
  - `pow`（`fdlibm` 由来の純粋なC実装）、`sqrt`、`atan2`、`fabs` を含む。
  - オプションとして `USE_JS_MATH` を指定してコンパイルすることで、JavaScriptホスト環境から数学関数（`pow`、`atan2`、`cos`、`sin` など）を直接インポートでき、さらなるサイズの削減とパフォーマンスの向上が期待できます。
- **リンカに優しいスタブ:** `stdio.h` の関数（`fprintf` など）や `assert.h` に対する何もしない（no-op）スタブを提供し、これらを使用するコードでもI/Oやアサーションのロジックを含めることなくコンパイル可能。
- **標準型:** 標準的な整数型のための `stdint.h` を含む。

## 使い方

### 1. サブモジュールとして追加

プロジェクトに `tinystd` を追加します。例えばGitサブモジュールとして追加する場合は以下のようになります。

```sh
touch .gitmodules
git submodule add https://github.com/code4fukui/tinystd
```

### 2. インクルードしてコンパイル

ソースファイルに必要なヘッダをインクルードします。
```c
#include "tinystd/stdlib.h"
#include "tinystd/string.h"
#include "tinystd/math.h"
```

プロジェクトを必要な `tinystd` のソースファイルと共にコンパイルします。

**Emscriptenでの例（純粋なC/WASM）:**
```sh
emcc my_app.c tinystd/stdlib.c tinystd/string.c tinystd/math.c tinystd/pow.c -o my_app.js
```

### 3. （オプション）JavaScriptの数学関数のインポート

ホスト環境の数学関数を使用するには、プリプロセッサフラグ `USE_JS_MATH` を定義します。これにより、Cの実装がJavaScriptからのインポートに置き換わります。

**Emscriptenでの例（`USE_JS_MATH` を使用）:**
```sh
emcc my_app.c tinystd/stdlib.c tinystd/string.c tinystd/math.c -D USE_JS_MATH -o my_app.js
```
*注: このフラグを使用する場合、`pow.c` は不要です。*

## APIリファレンス

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
一般的な数学関数を提供します。`pow`、`atan2`、`cos`、`sin` などの実装は `USE_JS_MATH` フラグに依存します。
```c
double sqrt(double f);
double pow(double f, double n);
double atan2(double y, double x);
float atan2f(float y, float x);
double fabs(double n);
int isnan(float f);
```

### `stdio.h`
コンパイルを通すための一般的なI/O関数のスタブを提供します。すべての出力は破棄されます。
```c
#define fprintf(...) (void)0
#define fputc(...) (void)0
#define stdout (FILE*)0
#define stderr (FILE*)0
```

## 帰属

`pow.c` における `pow()` の実装は、`fdlibm` ライブラリ（`https://www.netlib.org/fdlibm/e_pow.c` にて入手可能）に由来しています。

```
====================================================
Copyright (C) 2004 by Sun Microsystems, Inc. All rights reserved.

Permission to use, copy, modify, and distribute this
software is freely granted, provided that this notice
is preserved.
====================================================
```

## ライセンス

MIT License
