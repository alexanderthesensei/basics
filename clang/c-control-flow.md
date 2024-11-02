Programs nearly never work linearly - they have to decide what to do depending on the inputs. To do that, C has 4 main statements ~~and gotos which you should not know about~~.

## The `if` statement
```c
if (...) {
	...
}
```
The if statement executs its body only if the experssion given to it is not zero.

```c
if (1) {
	printf("This always runs\n");
}
```
```bash
user@machine:~/flow$ gcc control_flow.c
user@machine:~/flow$ ./a.out
This always runs
user@machine:~/flow$
```
A number is an expression

```c
if (2 > 3) {
	printf("This never runs\n");
}
```
```bash
user@machine:~/flow$ gcc control_flow.c
user@machine:~/flow$ ./a.out
user@machine:~/flow$
```
A comparison is an expression

```c
int yes() {
	return 1;
}

if (yes()) {
	printf("This always runs\n");
}
```
```bash
user@machine:~/flow$ gcc control_flow.c
user@machine:~/flow$ ./a.out
This always runs
user@machine:~/flow$ # I will show output at the bottom of the screen
user@machine:~/flow$ # without the build command going forward
```
And a variable or a function's return value is an expression, too.

## Operators
- `>`
- `<`
- `<=`
- `>=`
- `==`
- `!=` (not equal)

Let's go over some common ways to manipulate expressions. These are the comparison operators. They take 2 expressions and return the integer 1 or 0 (for true or false respectively).

```c
int x = 0;

if (x = 2) {
	printf("Issue 1. This runs.\n");
}

printf("Issue 2. Now x = %d.\n", x);
```
```text
Issue 1. This runs.
Issue 2. Now x = 2.
```
Note that the equality check is 2 equal signs and not one. This is because a single equal sign is the assignment operator, we used it earlier to give variables their values. The compiler *will not warn you* if you use the wrong one.

```c
int x = 0;

if (2 = x) {
	printf("Issue 1. This runs.\n");
}

printf("Issue 2. Now x = %d.\n", x);
```
```text
flow.c: In function ‘main’:
flow.c:6:9: error: lvalue required as left operand of assignment
    6 |   if (2 = x) {
      |         ^
```
If you want it to, you can use the yoda notation.

```c
int x = 0;

if (2 == x) {
	printf("Thi doesn't get printed anymore\n");
}

printf("And x is still %d!\n", x);
```
```text
And x is still 0!
```
Here's the corrected code.

- `&&` and
- `||` or
- `!` not

There are also the logical operators. `And` takes 2 integers and returns 1 if both of them are 1. `Or` takes 2 integers and returns 0 if both of them are 0. `Not` takes one expression and returns a zero if it's non-zero and a one if it's zero.

```c
int x = 32;

if (16 < x && x < 64) {
	printf("X ∈ (16; 64)\n");
}
```
```text
X ∈ (16; 64)
```
Here's an example using a combination of the above.

## `else`
```c
int x = 8;

if (16 < x && x < 64) {
	printf("X ∈ (16; 64)\n");
} else {
	printf("X ∉ (16; 64)\n");
}
```
```text
X ∉ (16; 64)
```
Any if statement can have an `else` block that will execute if the expression is zero.

```c
void check_range(int x) {
    if (16 < x && x < 64) {
        printf("X ∈ (16; 64)\n");
    } else {
        if (x <= 16) {
            printf("X ∈ (-∞; 16]\n");
        } else {
            printf("X ∈ [64; +∞)\n");
        }
    }
}
```
```c
int main() {
    check_range(-32);
    check_range(+32);
    check_range(+64);
}
```
```text
X ∈ (-∞; 16]
X ∈ (16; 64)
X ∈ [64; +∞)
```
You can put an if block inside of another if block, but that adds a lot of code and harms readability.

```c
void check_range(int x) {
    if (16 < x && x < 64) {
        printf("X ∈ (16; 64)\n");
    } else if (x <= 16) {
        printf("X ∈ (-∞; 16]\n");
    } else {
        printf("X ∈ [64; +∞)\n");
    }
}
```
```c
int main() {
    check_range(-32);
    check_range(+32);
    check_range(+64);
}
```
```text
X ∈ (-∞; 16]
X ∈ (16; 64)
X ∈ [64; +∞)
```
You can use the `else if` blocks to flatten the same thing.

## Recursion
```c
void sequence(int min, int max) {
    if (0 == min) {
        printf("{"); // first element
    }
    
    printf("%d", min);
    if (max == min) {
        printf("}\n"); // last element
        return;
    } else {
        printf(", "); // continuing
        sequence(min + 1, max); // recursing
    }
}
```
```c
int main() {
    sequence(0, 8);
}
```
```text
{0, 1, 2, 3, 4, 5, 6, 7, 8}
```

With just ifs and functions, you can make loops. This is done by making a function call itself based on some condition. The pattern is called recursion.

## `while`-loops
```c
int index = 0;
int max = 8;
printf("{");

while (index < max) {
	printf("%d, ", index);
	index++; // same as `index = index + 1;`
}

printf("%d}\n", max);
```
```text
{0, 1, 2, 3, 4, 5, 6, 7, 8}
```
C has a syntax for loops, so you don't need to make a function every time you want to repeat something. A `while` loop is like an if statement, except the body executes continuously while the condition is not zero, and not just once. This approach is often simpler.

## `for`-loops
```c
int max = 8;
printf("{");

for (int i = 0; i < max; i++) {
    printf("%d, ", i);
}

printf("%d}\n", max);
```
```text
{0, 1, 2, 3, 4, 5, 6, 7, 8}
```
A for loop is just a shorthand for a while loop with a counter written into it. It has 3 arguments - the coutner variable, the run condition, and what to do after every iteration. The counter variable is often declared in-place and called i (short for index).

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 4; j++) {
        printf("%d ", i * j);
    }
    printf("\n");
}
```
```text
0 0 0 0
0 1 2 3
0 2 4 6
```
Loops can be nested. Here's a multiplication table, for example.

## Homework
- [fizzbuzz](https://en.wikipedia.org/wiki/Fizz_buzz)
- print the [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_sequence)
- make a 10x10 multiplication table

Writer: [alexanderthestudent](https://github.com/alexanderthesensei), 2024-10-05.