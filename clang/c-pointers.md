# Указатели
Переменная - это именованный участок памяти. Кроме именованных участков, вы можете иметь неименованные участки, то есть буквальные адреса. Если вы работаете над операционной системой, этот адрес создается ей и является виртуальным . Если вы работаете напрямую с железом, то это порядковый номер группы конденсаторов в самом чипе памяти.

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

Вы можете получить адрес существующей переменной с помощью оператора `&`. При каждом запуске кода он будет новым, поэтому нельзя использовать адреса вместо имен.

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

Для решения этой проблемы в языке C существует специальный класс типов, называемый указателями. Это переменные, которые хранят не значения, а адреса памяти. При этом полезно знать, какие данные хранятся по этим адресам, поэтому они наследуют тип. За типом следует звездочка.

Чтобы получить значение из адреса, вы пишете звездочку, а затем этот адрес.

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

## Стек
Все переменные, которые прописаны вами, помещаются на стек. Стек - это виртуальный блок памяти, выделяемый оболочкой перед запуском вашей программы. Он очень маленький, всего 8 KiB на моей машине:
```bash
user@machine:~$ ulimit -s
8192
```

Давайте рассмотрим подробнее, как он работает:
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
Мы определяем переменную в главной функции, а также вторую функцию и переменную внутри нее. Затем вызываем вторую функцию и выводим адреса обеих переменных. Они отличаются на 32 байта. Это определение функции и ее вызов.

Если мы дважды вызовем вторую функцию из main, у переменной будет один и тот же адрес, потому что после завершения первого вызова все переменные внутри функции были удалены:
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

Обратите внимание, что, хотя переменные и были удалены, их значения остались. Если вы вернете адрес локальной переменной и попытаетесь ее прочитать, значение все еще будет там:
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

Таким образом, если вы запустите другую функцию, которая по случайности будет иметь переменную с таким же адресом, вы получите старое значение:
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

Однако это не является определенным поведением, и компилятор или операционная система могут на самом деле очистить эту память. Вот что происходит, когда я снова включаю оптимизацию, убрав флаг `-O0` из предыдущей версии:
```text
1st: 0x7ffebad4e284
2nd: 64
still 2nd: 0
```

Если вы попытаетесь прочитать участок памяти, не относящийся к функции, в которой вы сейчас находитесь, операционная система может убить вашу программу. Она делает это, чтобы предотвратить чтение или изменение памяти, принадлежащей другим программам.
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

Поскольку указатели - это просто адреса, а адреса - это большие числа, вы можете добавить число к указателю. В результате вы получите следующую ячейку памяти.
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

## Куча
Поскольку стек составляет всего 8 килобайт или что-то около того, для многих задач его недостаточно. Если, например, вам нужно обработать большую электронную таблицу, то данные просто не поместятся. В этом случае ваша программа за'segfault'ится.
```c
int main() {
    int x = 65535;
    main(); // artificailly fills all available memory
}
```
```text
Segmentation fault (core dumped)
```

Если вам нужно хранить в памяти больше данных, вы можете попросить операционную систему выделить еще один непрерывный блок, запустив функцию `malloc`, которая расшифровывается как `memory allocation`. Она возвращает первый байт. Предположим, мне нужно еще 8 килобайт:
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

В Си нет типа массива. Создание массива в C - это просто выделение памяти, а чтение элемента этого массива - сложение первого адреса с индексом этого элемента.
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
Здесь мы выделяем достаточно памяти для 3 целых чисел. Целое число - это 4 байта, но чтобы не думать об этом, мы используем встроенную функцию.

Стек выделяется оболочкой таким же образом и очищается ей же после выхода из функции main. Если вы выделяете память сами, очищать ее придется вам:
```c
int main() {
	int* array = malloc(sizeof(int) * 3);
	// using array
	free(array);
}
```
Если вы забудете это сделать, программа съест всю оперативную память вашего компьютера, и он зависнет.

## Матрицы
Типа матрицы не существует, есть только указатель на указатель.
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

Си предлагает дополнительный синтаксис, чтобы сделать это немного проще:
```c
*(pointer + index)
pointer[index]
```

Таким образом, последний пример можно переписать так:
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

Затем мы можем упростить его еще больше:
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

## Строки
В c нет строкового типа, только указатели на несколько символов.
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
Чтобы знать, где заканчивается строка, и не хранить количество символов в отдельной переменной, их всегда заканчивают на `\0`.

Как и в случае с массивами, существует дополнительный синтаксис, позволяющий сделать ваш код более лаконичным:
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

А еще вспомогательная функция, которую мы только что написали, встроена в printf:
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

## Чтение аргументов интерфейса командной строки
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
Функция main может принимать либо 0 аргументов, либо 2 аргумента. Эти два аргумента - целочисленный счетчик аргументов (обычно называется `argc`) и значения аргументов (обычно `argv`), представляющие собой массив строк.

## Домашка
- найдите определитель матрицы `[[1, 2, 1], [2, 3, -2], [3, 0, 4]]`.
- напишите программу, которая принимает аргумент `--name [string]` и приветствует пользователя
- добавьте флаг `--help` в эту программу

Автор: [alexanderthestudent](https://github.com/alexanderthesensei), 2024-11-06.

Соавтор: [DeepL Translate](https://deepl.com).