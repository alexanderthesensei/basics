## user@machine:~$ ls

notes:
В реальных программах всегда требуется какие-то входные данные. Даже самые простые команды, которые на первый взгляд не требуют ввода от пользователя, вроде `ls`, должны знать, где они запущены.

---

```c
int main(int argc, char **argv) {
	return 0;
}
```

notes:
[В прошлый раз](c-the-language.md) я упоминал, что функция main может принимать либо два аргумента, либо ни одного. Вот те два аргумента, о которых я говорил. С первым все понятно, это количество аргументов. А вот со вторым все уже интереснее.

---

## Pointers
notes:
Чтобы понять, что он собой представляет, нам сначала нужно разобраться с указателями.
Кроме тех типов, о которых я говорил ранее, существуют типы указателей. Указатель - это *переменная, которая хранит в памяти адрес другой переменной.* Разберем на схеме...

---

![heap.excalidraw](Excalidraw/heap.excalidraw.png)

notes:
Когда вы говорите оболочке запустить какое-либо приложение, она выделяет для него непрерывный блок памяти, называемый стеком. Все переменные и функции, которые вы создаете, попадают сюда. Стек работает очень быстро, но его размер весьма ограничен - всего 8192 КБ на моей машине (согласно `ulimit -s`).

Если вам нужно больше, вы просите у системы. Она выделяет новый непрерывный блок в куче и сообщает вам адрес его первого байта. Этот блок похож на переменную, но вместо имени у него сырой адрес, т.е. его индекс в памяти. При каждом запуске программы адрес будет разным, поэтому хранить его в коде невозможно.

Поэтому существует особый тип переменных, называемых указателями, которые указывают на другую память. Чтобы получить следующий байт, нужно прибавить к адресу 1.

Синтаксически указатель имеет тот же тип, что и данные, на которые он указывает, плюс звездочка перед именем. У переменной Argv, или `значения аргументов`, их две, потому что это указатель на указатель.

---

```c
int main(int argc, char **argv) {
	return 0;
}
```

notes:
Перейдем к тому, зачем нам это нужно. Как вы можете помнить, в C нет типа строки, есть только символ. Также в языке нет типа списка/массива. Зато есть указатели.

Так вот, *массив в C - это указатель на первый элемент непрерывного блока памяти.* Уловили?

Итак, строка - это массив символов, последний из которых - `\0`.

---

```c
void read_char(char *string) {
	printf("%c", string);
}
```

notes:
Попробуем прочитать argv, для этого сделаем функцию для вывода следующего байта в строке. Поскольку массив - это просто указатель на его первый элемент, мы можем прочитать этот элемент напрямую.

---

```bash
user@machine:~$ gcc main.c
main.c: In function ‘main’:
main.c:4:12: warning: format ‘%c’ expects argument of type ‘int’, but argument 2 has type ‘char *’ [-Wformat=]
    4 |   printf("%c", argv[0]);
      |           ~^   ~~~~~~~
      |            |       |
      |            int     char *
      |           %s

user@machine:~$ ./a.out
�
```

notes:
Попробуем прочитать argv, для этого сделаем функцию для вывода следующего байта в строке. Поскольку массив - это просто указатель на первый элемент, мы можем прочитать этот элемент напрямую.

А можем ли? Мы получаем неизвестный символ. Ага,

---

```c
void read_char(char *string) {
	printf("%p", string);
}
```

notes:
Хорошо, есть способ вывести сырое значение, пробуем так...

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
0x7ffd0222aec0
```

notes:
Похоже на адрес.

---

```c
void read_char(char *string) {
	printf("%c", *string); // вытаскиваем 0-й элемент
}

int main(int argc, char **argv) {
	read_char(*argv); // вытаскиваем 0-й элемент
}
```

notes:
Еще раз, указатель - это *переменная, которая хранит адрес*. Попытка прочитать указатель дает нам адрес. Чтобы прочитать значение, хранящееся по этому адресу, мы используем оператор dereference - звездочку, поставленную перед указателем.

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
.
```

notes:
Это дает нам один символ. Читаем дальше:

---

```c
void read_char(char *string, int index) {
	char *letter = string + index;
	printf("%c", *letter);
}

int main(int argc, char **argv) {
	read_char(*argv, 0);
	read_char(*argv, 1);
}
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./
```

notes:
Так как указатель хранит адрес первого элемента, мы можем получить элемент с номером n, прибавив n к этому адресу. Получаем второй.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index;
	printf("%c", *letter);

	read_char(string, index + 1);
}

int main(int argc, char **argv) {
	read_char(*argv, 0);
}
```

notes:
Прочитаем всю строку, сделав так, чтобы функция вызывала саму себя, указывая следующий индекс.

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./a.outHOSTNAME=machine=1HOME=/home/userOLDPWD=/TERM=xtermPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/home/user./a.out
Segmentation fault (core dumped)
```

notes:
Программа считывает все символы, начиная с того, на который мы указали. Проблема только в том, что мы не указали ей, где остановиться. Поэтому читается все подряд, пока система не увидит, что наше приложение пытается читать чужую память, и не убьет его. Это называется `segmentation fault` или сокращенно `segfault`.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index; // вычисляем указатель

	if (index < 3) {
		printf("%c", *letter);
		read_char(string, index + 1);
	}
}

// Функция main такая же, как в прошлом примере
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./a.
```

notes:
Чтобы указать ему, когда остановиться, воспользуемся оператором if. Оператор if напоминает функцию - у него есть условие в круглых скобках и тело в фигурных скобках. Если вы хотите сравнить с чем-то переменную, используйте двойной знак равенства вместо одинарного. Начнем с чтения первых 3 символов.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index;

	if ('\0' != *letter) {
		printf("%c", *letter);
		read_char(string, index + 1);
	}
}
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./a.out
```

notes:
Поскольку каждая строка заканчивается нулевым символом, мы можем использовать его в качестве условия остановки. Теперь прога выводит свое имя.

---

```bash
user@machine:~$ mv a.out program
user@machine:~$ ./program
./program
```

notes:
Первый аргумент - от оболочки, он указывает, где находится программа.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index;

	if ('\0' != *letter) {
		printf("%c", *letter);
		read_char(string, index + 1);
	} else {
		printf("\n");
	}
}

void read_args(int current, int total, char **argv) {
	if (current < total) {
		printf("arg%i: ", current);
		read_char(*(argv + current), 0);
		read_args(current + 1, total, argv);
	}
}

int main(int argc, char **argv) {
	read_args(0, argc, argv);
}
```

notes:
Чтобы прочитать все аргументы, мы можем раскрыть каждый индекс от 0 до argc.

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out first second third fourth
arg0: ./a.out
arg1: first
arg2: second
arg3: third
arg4: fourth
```

notes:
И все работает как ожидалось.

---

```c
void read_args(int current, int total, char **argv) {
	if (current < total) {
		return;
	}

	printf("arg%i: ", current);
	read_char(*(argv + current), 0);
	read_args(current + 1, total, argv);
}
```

notes:
Все работает в соответствии с нашими ожиданиями.

---

```c
#include <stdio.h> // for printf
#include <stdlib.h> // for malloc and free
#include <unistd.h> // for getcwd

int main(int argc, char **argv) {
	int PATH_MAX = 4096;
	char *buffer = malloc(PATH_MAX);
	char *directory = getcwd(buffer, PATH_MAX);
	free(buffer);

	read_char(directory, 0);
}
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
/home/user/c
```

notes:
ls использует текущую директорию, если первый аргумент не передан. Сделаем то же самое.

Функция `getc`urrent`w`orking`d`irectory принимает 2 аргумента: указатель на буфер и его длину.

Мы создаем этот буфер на куче (поскольку он большой) с помощью функции `malloc`, используем его, а затем освобождаем (`free`), возвращая память системе. Если мы будем забывать освобождать выделенную память, наша программа съест несколько гигабайт, и либо будет убита системой, либо убьет систему, вынуждая пользователя перезагрузиться, в зависимости от того, кто из них успеет первым.

---

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char *get_work_dir(int argc, char **argv) {
	if (argc > 1) {
		return *(argv + 1);
	}

	int PATH_MAX = 4096;
	char *buffer = malloc(PATH_MAX);
	char *directory = getcwd(buffer, PATH_MAX);
	free(buffer);

	return directory;
}

int main(int argc, char **argv) {
	char *workdir = get_work_dir(argc, argv);
	printf("%s", workdir);
}
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
/home/user/c
user@machine:~$ ./a.out /etc
/etc
```

notes:
У printf есть встроенный форматтер строк, чтобы каждый раз не писать его вручную, как мы только что делали.

---

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <dirent.h>

char *get_work_dir(int argc, char **argv); // omitted

void print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	printf("%s ", (*entry).d_name);
}

int main(int argc, char **argv) {
	char *path = get_work_dir(argc, argv);

	DIR *stream = opendir(path);
	print_dir_contents(stream);
	print_dir_contents(stream);
	int error = closedir(stream);
	if (error != 0) {
		return 1;
	}
}
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
. ..
```

notes:
Чтобы открыть директорию, нам понадобится функция `opendir`, которая преобразует строку в поток директории. Обратите внимание, что поток имеет нестандартный тип, мы научимся создавать такие в другой раз. В данном случае тип взят из библиотеки `dirent.h`.
Потом мы помещаем ее в нашу функцию, которая вызывает `readdir` на этом потоке и выводит его. Рассмотрим этот момент более детально:

---

```c
void print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	printf("%s ", (*entry).d_name);
}
```

notes:
Readdir - это функция стандартной библиотеки, которая возвращает указатель на следующую запись в каталоге или на null, если мы прочитали все его содержимое. Эта запись имеет тип `struct dirent`.

---

## Structs
```c
struct point {
  int x;
  int y;
};
```

notes:
Структуры (`struct`) - это пользовательские типы данных, которые позволяют объединять несколько переменных в одну.

---

```c
int main() {
	struct point a;
	struct point b;
	a.x = 20;
	b.x = 30;
	printf("%i", a.x); // 20
	printf("%i", b.x); // 30
}
```

notes:
Чтобы прочитать переменную внутри struct, мы используем точечную нотацию.

---

```c
void print_point(struct point *pointer) {
  struct point value = *pointer;
  printf("x: %i, y: %i\n", value.x, value.y);
}

int main() {
  int size = sizeof(struct point);
  struct point *a = malloc(size);
  (*a).x = 20;
  (*a).y = 30;
  print_point(a);
}
```

notes:
Если мы хотим передать struct в другую функцию, она не должна находиться в стеке. Для того, чтобы выделить память, нам нужно знать ее объем. Функция sizeof принимает тип и выдает количество занимаемых им байт.

---

```c
void print_point(struct point *pointer) {
  printf("x: %i, y: %i\n", pointer->x, pointer->y);
}

int main() {
  int size = sizeof(struct point);
  struct point *a = malloc(size);
  a->x = 20;
  a->y = 30;
  print_point(a);
}
```

notes:
разыменование структуры - очень распространенная операция, поэтому существует такое сокращение

---

```c
void print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	printf("%s ", entry->d_name);
}
```

notes:
Вернемся к элементу директории и прочитаем в нем поле под названием d_name.

---

```c
int print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	if (entry == NULL) {
		return -1;
	}
	printf("%s ", entry->d_name);
	print_dir_contents(directory);
	return 0;
}

int main(int argc, char **argv) {
	char *path = get_work_dir(argc, argv);

	DIR *stream = opendir(path);

	print_dir_contents(stream);
	printf("\n");

	int error = closedir(stream);
	if (error != 0) {
		return 1;
	}
}
```

notes:
остается только все это зациклить

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
program .. . main.c a.out point.c 
user@machine:~$ ./a.out /
bin home lib mnt root sys .. dev var . boot opt usr proc lib64 run etc tmp srv sbin media
```

notes:
## Домашка
Добавьте флаг, который заставляет прогу вести себя как `tree`, а не как `ls`.