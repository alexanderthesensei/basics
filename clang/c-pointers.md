# Pointers
A variable is a named location in the memory. Aside from named locations, you can have unnamed locations, or the literal addresses. If you have an operating system, that location is created by it and is a virtual address. If you're operating on bare metal, that's the index of a capacitor group in the actual memory chip.

```c
int main() {
    int value = 32;

    printf("%p = ", &value);
    printf("%d\n", value);
}
```
```bash
user@machine:~$ gcc ptr.c
user@machine:~$ ./a.out
0x7ffd4c0af554 = 32

user@machine:~$ ./a.out
0x7ffc4b276fa4 = 32

user@machine:~$ ./a.out
0x7ffec77ea824 = 32

user@machine:~$ ./a.out
0x7ffff066a1f4 = 32
```

You can get the address of an existing variable using the `&` operator. Each time you run the code that address will be different, so you can't use them instead of names.

```c
int main() {
    int value = 32;
    int* pointer = &value;

    printf("%p = ", pointer);
    printf("%d\n", *pointer);
}
```
```text
0x7ffcaa33bc44 = 32
```

C has a special class of types to address this problem (huh) called pointers. They are variables that store memory addresses instead of values. It's still useful to know what data lives under those addresses, so they inherit its type. The type is followed with an asterisk.

To get a value from an address, you pass an asterisk followed by that address.

```c
int main() {
	int x = 32;
	int* px = &x;
	int** ppx = &px;

	printf("%p -> %p -> %i", ppx, px, x);
}
```
```text
0x7ffccfc4afc0 -> 0x7ffccfc4afbc -> 32
```

## The stack
All the variables you define go on the stack. The stack is a virtual block of memory allocated by the shell before your app is launched. It is very small, 8KiB on my machine:
```bash
user@machine:~$ ulimit -s
8192
```

Let's take a closer look at how it works:
```c
#include <stdio.h>

void second_function() {
    int val;
    printf("2nd: %p\n", &val);
}

int main() {
    int val;
    printf("1st: %p\n", &val);

    second_function();
}
```
```text
1st: 0x7ffe7c77e7e4
2nd: 0x7ffe7c77e7c4
```
We define a variable in a main function, a second function, and a variable inside of it. Then we call the second function and print the addresses of both variables. They differ by 32 bytes. That's the function definition and function call.

If we call the second function twice from main, the variable will have the same address because after the first call completed all the variables inside the function got deleted:
```c
#include <stdio.h>

void second_function() {
    int val;
    printf("2nd: %p\n", &val);
}

int main() {
    int val;
    printf("1st: %p\n", &val);

    second_function();
    second_function();
}
```
```text
1st: 0x7ffe288a0c34
2nd: 0x7ffe288a0c14
2nd: 0x7ffe288a0c14
```

Note that, while the variables were deleted, the values stayed. If you return the address of a local variable and try to read it, the value will still be there:
```c
#include <stdio.h>

int* second_function() {
    int val = 64;
    printf("2nd: %p\n", &val);

    int* pval = &val;

    return pval;
}

int main() {
    int val;
    printf("1st: %p\n", &val);

    int* second_variable = second_function();
    printf("the pointer is %p ", second_variable);
    printf("and the value stored there is %i\n", *second_variable);
}
```
```bash
user@machine:~$ gcc ptr.c -O0
user@machine:~$ ./a.out
1st: 0x7ffdf46d59ec
2nd: 0x7ffdf46d59bc
the pointer is 0x7ffdf46d59bc and the value stored there is 64
```

So if you run another function that will just happen to have a variable at the same address, you will get the old value:
```c
#include <stdio.h>

void second_function() {
    int val = 64;
    printf("2nd: %i\n", val);
}

void third_function() {
    int val;
    printf("still 2nd: %i\n", val);
}

int main() {
    int val;
    printf("1st: %p\n", &val);

    second_function();
    third_function();
}
```
```text
1st: 0x7ffe7dbf4ab4
2nd: 64
still 2nd: 64
```

However, this is not a defined behavior, and the compiler or the operating system may actually clean that memory, here's what happens when I re-enable optimization by removing the `-O0` flag from earlier:
```text
1st: 0x7ffebad4e284
2nd: 64
still 2nd: 0
```

If you try to read a memory location outside of the function you're currently in, the operating system may kill the program. It does that to prevent you from reading or altering memory that belongs to other apps.
```c
#include <stdio.h>

int main() {
    int* pointer = (int*) 0x7ffe7dbf4ab4;

    printf("random address = %i\n", *pointer);
}
```
```text
Segmentation fault (core dumped)
```

Because pointers are just addresses and addresses are just big numbers, you can add a number to a pointer. Doing that will give you the next memory cell.
```c
#include <stdio.h>

int main() {
    int x = 32;
    int y = 64;

    int* pointer_x = &x;
    int* pointer_y = pointer_x + 1; // pointer arithmetic

    printf("x = %i;\n", *pointer_x);
    printf("y = %i;\n", *pointer_y);
}
```
```bash
user@machine:~$ gcc ptr.c -O0 # disable optimization to keep y around
user@machine:~$ ./a.out
x = 32;
y = 64;
```

## The heap
Since the stack is only 8KiB or so, it's not enogh for many tasks. If you, for example, have to process a big spreadsheet, the data will simply not fit. If you do that, your program will segfault.
```c
int main() {
    int x = 65535;
    main(); // artificailly fills all available memory
}
```
```text
Segmentation fault (core dumped)
```

If you need to hold more data in the RAM, you can ask the OS for another continuous chunk by running the `malloc` function, which stands for `memory allocation`. It returns the first byte. Say I need a another 8KiB:
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int* external_memory = malloc(8192);

    *external_memory = 1;
    printf("%i\n", *external_memory);
}
```
```text
1
```

C doesn't have an array type. Making an array in C is just allocating memory, and reading an element of that array is summing the first address with the index of that element.
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int* array = malloc(sizeof(int) * 3);

    *array = 1;
    *(array + 1) = 2;
    *(array + 2) = 3;

    printf("%i, ", *array);
    printf("%i, ", *(array + 1));
    printf("%i\n", *(array + 2));
}
```
```text
1, 2, 3
```
Here, we're allocating enough memory for 3 integers. An integer is 4 bytes, but we use a built-in function to not have to remember that.

The stack is allocated by the shell in the same manner, and cleaned by the shell after your main function exits. If you allocate memory, you must clean it up:
```c
int main() {
	int* array = malloc(sizeof(int) * 3);
	// using array
	free(array);
}
```
If you forget to do that, the program will eat up all of your computer's ram and the latter will freeze.

## Matricies
No matrix type, just a pointer to a pointer.
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int width = 3;
    int height = 2;

    int** matrix = malloc(height * 4);
    *(matrix + 0) = malloc(width * 4);
    *(matrix + 1) = malloc(width * 4);

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            int* row = *(matrix + i);
            *(row + j) = (i+1) * (j+1);
        }
    }

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            int* row = *(matrix + i);
            printf("%i ", *(row + j));
        }
        printf("\n");
    }
}
```

C offers additonal syntax to make this a bit easier to read:
```c
*(pointer + index)
pointer[index]
```

That turns the last example into this:
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int width = 3;
    int height = 2;

    int** matrix = malloc(height * 4);
    for (int i = 0; i < height; i++) {
        matrix[i] = malloc(width * 4);
    }

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            matrix[i][j] = (i+1) * (j+1);
        }
    }

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            printf("%i ", matrix[i][j]);
        }
        printf("\n");
    }
}
```

Then we can simplify it further:
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int width = 3;
    int height = 2;

    int** matrix = malloc(height * 4);

    for (int i = 0; i < height; i++) {
        matrix[i] = malloc(width * 4);
        for (int j = 0; j < width; j++) {
            matrix[i][j] = (i+1) * (j+1);
        }
    }

    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            printf("%i ", matrix[i][j]);
        }
        printf("\n");
    }
}
```
```text
1 2 3 
2 4 6
```

## Strings
There is no string type in c, just a pointer to some chars.
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char* string = malloc(sizeof(char) * 7);
    string[0] = 'H';
    string[1] = 'e';
    string[2] = 'l';
    string[3] = 'l';
    string[4] = 'o';
    string[5] = '!';
    string[6] = '\0';

    int idx = 0;
    while (string[idx] != '\0') {
        printf("%c", string[idx]);
        idx++;
    }
}
```
```text
Hello!
```
To know where the string ends without having to keep its character count around in a separate variable, they always end in `\0`.

Like with arrays, there is additional syntax to keep your code concise:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char* string = malloc(sizeof(char) * 7);
    string = "Hello!";

    int idx = 0;
    while (string[idx] != '\0') {
        printf("%c", string[idx]);
        idx++;
    }
}
```
```text
Hello!
```

And the helper function we just wrote is built into printf:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char* string = malloc(sizeof(char) * 7);
    string = "Hello!";

    printf("%s", string);
}
```
```text
Hello!
```

## Reading CLI arguments
```c
#include <stdio.h>

int main(int argc, char** argv) {
    printf("%i args:\n", argc);

    for(int i = 0; i < argc; i++) {
        printf("- %s\n", argv[i]);
    }
}
```
```bash
user@machine:~$ gcc cli.c -o cli # output file
user@machine:~$ ./cli
1 args:
- ./cli

user@machine:~$ ./cli --help
2 args:
- ./cli
- --help

user@machine:~$ ./cli these are space-separated strings
5 args:
- ./cli
- these
- are
- space-separated
- strings
```
The main function can take either no arguments or two arguments. These two arguments are the integer argument count (usually called `argc`) and the argument values (usually `argv`), which is just an array of strings.

## Homework
- find the detirminant of the matrix `[[1, 2, 1], [2, 3, -2], [3, 0, 4]]`
- write a program that takes a `--name [string]` argument and greets the user
- make a `--help` flag for that program

Written by [alexanderthestudent](https://github.com/alexanderthesensei) on 2024-11-06