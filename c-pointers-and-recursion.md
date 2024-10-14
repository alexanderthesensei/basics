## user@machine:~$ ls

notes:
Real programs always take some input. Even a simple command that seimingly takes no input from the user, like `ls`, needs to know where it was launched.

---

```c
int main(int argc, char **argv) {
	return 0;
}
```

notes:
[Last time](c-the-language.md), I mentioned that the main function can take either two arguments or no arguments. Here are the two arguments I was talking about. The first one is obvious, it is the `arg`ument `count`. But what is the second one?

---

## Pointers
notes:
To understand that, we first need to learn about pointers.
Aside from the types I've mentioned earlier, there are pointer types. A pointer is *a variable that stores the memory address of another variable.* Here's a diagram:

---

![heap.excalidraw](Excalidraw/heap.excalidraw.png)

notes:
When you ask the shell to start an app, it allocates a continuous chunk of memory called the stack. All the variables and functions you create go here. The stack is very fast, but also limited in size, just 8192KB on my system (according to `ulimit -s`).

If you need more, you ask the system. It allocates another continuous block on the heap and gives you the address of its first byte. The block is like a variable, but instead of the name it has a raw address, its index inside the memory. Every time you run the program, the address will be different, so storing it in the code is impossible.

That's why there's a special type of variables, called pointers, that point to other memory. To get the next byte, you add 1 to the address.

Syntactically, a pointer has the same type as the data it points to, plus an asterisk before the name. Argv, or `arg`ument `v`alue has 2 because it's a pointer to a pointer.

---

```c
int main(int argc, char **argv) {
	return 0;
}
```

notes:
Now to why we need that. As you remember, C doesn't have a string type, you can only store a character. What it also doesn't have is a continuous list / array type. What it does have is pointers.

So, *an array in c is a pointer to the first element of a continuous block of memory.* Got that?

Now, a string is an array of `char`s, the last of which is `\0`.

---

```c
void read_char(char *string) {
	printf("%c", string);
}
```

notes:
Let's try reading it by making a function to read the next byte in a string. Since an array is just a pointer to its first element, we can read it directly.

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
Or can we? We get an unrecognized symbol. Huh.

---

```c
void read_char(char *string) {
	printf("%p", string);
}
```

notes:
Okay, there's a way to print the raw value, let's try that

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
0x7ffd0222aec0
```

notes:
That looks like an address.

---

```c
void read_char(char *string) {
	printf("%c", *string); // get the first element of *string
}

int main(int argc, char **argv) {
	read_char(*argv); // get the first element of argv
}
```

notes:
Again, a pointer is *a variable that stores an address*. Trying to read the pointer gives us the address. To read the value stored under that address, we use the dereference operator, an asterisk put before the pointer.

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
.
```

notes:
That gives us something. Let's read more:

---

```c
void read_char(char *string, int index) {
	char *letter = string + index; // get a pointer to that element
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
Since the pointer stores the address of the first element, we can get the element number n by adding n to that address. We've got 2.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index; // get a pointer to that element
	printf("%c", *letter);

	read_char(string, index + 1);
}

int main(int argc, char **argv) {
	read_char(*argv, 0);
}
```

notes:
Let's read the entire string by having the function call itself with the next index.

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./a.outHOSTNAME=machine=1HOME=/home/userOLDPWD=/TERM=xtermPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPWD=/home/user./a.out
Segmentation fault (core dumped)
```

notes:
It reads every character starting from the one we pointed it. The only problem is that we never told it when to stop. So it reads everything it can, until the system realizes our app is trying to read someone else's memory and kills it. That is called `segmentation fault` or `segfault` for short.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index; // get a pointer to that element

	if (index < 3) {
		printf("%c", *letter);
		read_char(string, index + 1);
	}
}

// main function is the same as last time
```
```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
./a.
```

notes:
To tell it when to stop we'll use the if statement. An if statement looks much like a function - it has a condition in the brackets and a body in the curly brackets. Let's start by reading the first 3 characters.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index; // get a pointer to that element

	if (*letter != '\0') {
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
Since every string must end in a `\0` character, we can use that as the stop condition. Now it prints its own name.

---

```bash
user@machine:~$ mv a.out program
user@machine:~$ ./program
./program
```

notes:
The first argument is from the shell, it indicates where the program is located.

---

```c
void read_char(char *string, int index) {
	char *letter = string + index;

	if (*letter != '\0') {
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
We can read all the arguments by getting every index from 0 to argc.

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
This works as expected.

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
The return operation immediately stops the current function and gives a value back to the caller. We can use it here to stop early and move everything left.

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
ls uses the current directory if the first argument is not passed. Let's do the same thing.

The `getc`urrent`w`orking`d`irectory function accepts 2 arguments: a string buffer pointer and its length.

We create that buffer on the heap (because it's large) using the `malloc` function, then use it, and then `free` it, giving the memory back to the system. If we repeatedly forget to `free` the memory we allocate, our program will eat up a few GB and either get killed by the system for doing that, or kill the system forcing the user to reboot, whichever happens first.

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
printf has a built-in string formatter so you don't have to write your own each time, like we did

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
To actually open the directory, we need the `opendir` function that converts a string to a directory stream. Note that the strem is of nonstandard type, we'll learn how to create those another time. In this case, the type comes from `dirent.h`.
Then we put it into our function that calls `readdir` on that stream and prints it. Let's take a closer look at it:

---

```c
void print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	printf("%s ", (*entry).d_name);
}
```

notes:
Readdir is a standard library function that returns a pointer to the next directory entry, or to null, if we've read the entire thing. It has a type of `struct dirent`.

---

## Structs
```c
struct point {
  int x;
  int y;
};
```

notes:
Structs are custom data types that allow us to group multiple variables into one.

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
To read a variable inside a struct, we use the dot notation.

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
If we want to pass the struct to a different function, it shouldn't be on the stack. To allocate some memory, we need to know how much. The sizeof function takes a type and outputs a number of bytes it takes.

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
dereferencing a struct is a very common operation, so there is a shorthand

---

```c
void print_dir_contents(DIR *directory) {
	struct dirent *entry = readdir(directory);
	printf("%s ", entry->d_name);
}
```

notes:
back to the directory entry, we are reading the field called d_name.

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
the only thing left to do is to loop it

---

```bash
user@machine:~$ gcc main.c
user@machine:~$ ./a.out
program .. . main.c a.out point.c 
user@machine:~$ ./a.out /
bin home lib mnt root sys .. dev var . boot opt usr proc lib64 run etc tmp srv sbin media
```

notes:
## Homework
Add a flag that makes it behave like `tree` instead of `ls`.