---
title: Compiling C++ Code
draft: true
---

## Loops

Here is an example of a `while` statement:

```cpp
int main() {
    int x;
    x = 3;
    while (x < 10) {
        x = x + 1;
    }
}
```

... and here is the exact equivalent in assembler:

```nasm
        pushq   %rbp
        movq    %rsp, %rbp
        movl    $3, -4(%rbp)
        jmp     WHILE
DO:     addl    $1, -4(%rbp)
WHILE:  cmpl    $9, -4(%rbp)
        jle     DO
        movl    $0, %eax
        popq    %rbp
        ret
```

## Functions

Here is an example of a function named `factorial`, which returns the factorial of a given number:

```cpp
int factorial(int x) {
	if (x == 0) {
	    return 1;
	} else {
        return x * factorial(x-1);
    }
}

int main() {
    int x, y;
    x = 3;
    y = factorial(x);
}
```

... and here is the exact equivalent in assembler:

```nasm
FACTORIAL:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    %edi, -4(%rbp)
        cmpl    $0, -4(%rbp)
        jne     ELSE
        movl    $1, %eax
        jmp     ENDIF
ELSE    movl    -4(%rbp), %eax
        subl    $1, %eax
        movl    %eax, %edi
        call    FACTORIAL
        imull   -4(%rbp), %eax
ENDIF   leave
        ret
MAIN:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    $3, -8(%rbp)
        movl    -8(%rbp), %eax
        movl    %eax, %edi
        call    FACTORIAL
        movl    %eax, -4(%rbp)
        movl    $0, %eax
        leave
        ret
```