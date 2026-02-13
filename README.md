# Crystal-Generator
WebAssembly crystal generator

---
# WebAssembly Hello World in C

This example shows how to compile a simple C program to WebAssembly using **Emscripten** and run it in the browser. [web:2][web:10]

## 1. C source (`hello.c`)

```c
#include <stdio.h>

int main()
{
    printf("Hello, WebAssembly!\\n");
    return 0;
}
```

