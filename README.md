# Tutorial for CUTest C unit testing framework by mity

This step-by-step tutorial shows how to create a unit test suite for modules written in C. It assumes you are using a command-line C compiler (gcc in this case).

* Create a tiny library written in C consisting of a single source and single header file. This is a library file so it doesn't have a `main()`.
* Compile it using the command line (Unix/MacOS)
* Add simple unit tests by including [cutest.h](https://github.com/mity/cutest) by Martin Mitáš, aka [mity](https://github.com/mity) on GitHub
* Run the unit tests. See how both success and failure look.

# MISSING
* Overviews. Explain why each thing is being done.

## Create the library module to test

The library will consist of a simple function to compute the area of a circle. It has some error checking, which is not only good practice but illustrates how to implement tests with incorrect inputs and show expected behavior.

### Create a directory named circle

This is meant to simulate a real project, so give it a directory. In this example, it is off a directory called `c` in the user's working directory.

```bash
# Create a directory named c and a subdirectory named
# circle in the user's working directory
$ mkdir -p ~/c/circle

# Change to it
$ cd ~/c/circle
```

### Create the library source files

* Create a header file named area.h:

```c
#ifndef __AREA__
#define __AREA__
float area(float radius);
#define PI 3.1415927
#endif
```

* Create the library file area.c:

```c
#include "area.h"
/* Obtain the area of a circle given its radius.
 * Return -1 if given an invalid value.
 */
float area(float radius)
{
    if (radius <= 0)
        return -1;    
    return PI * radius * radius;
}
```

Determining the area of a circle is easy, but normally examples contain no error checking. Remember, this is meant to simulate what you'd do with your real code, so this crucial line does the trick:

```c
    if (radius <= 0)
        return -1;    
```

* Make sure area.c compiles correctly:

```bash
$ gcc -std=c99 fsize.c -c
```

#### Notes:

If you're not used to compiling with the command line, some notes.

* `gcc` is the name of the compiler. On MacOS you may need to download [Xcode](https://developer.apple.com/xcode/downloads/). If you haven't done so before you will be required to create a free developer's account for the privilege.
* The command-line flag `-std=c99` forces a compile using the [C99 standard](https://en.wikipedia.org/wiki/C99) version of the language. It is useful here primarily because this code uses `//` comments instead of the older `/*  */` style.
* The command-line flag `-c` means compile but do not make an executable. Compile and correct your source as many times as is necessary until `gcc` yields no output, which means success. There's no `main` in this code so it wouldn't work anyway.

## Create the mininum test harness with a main() and no tests

To test your library, you need a driver file with a `main()` from which you will call the tests. CUTest generates it automatically from a list of test functions you supply it. Let's create the simplest possible test harness using `cutest.h` just to make sure everything compiles properly and the executable runs.

### Obtain cutest.h

First you need `cutest.h`, which can be found on GitHub at [https://github.com/mity/cutest/](https://github.com/mity/cutest/).

* Copy the file [cutest.h](https://raw.githubusercontent.com/mity/cutest/master/include/cutest.h) from [https://raw.githubusercontent.com/mity/cutest/master/include/cutest.h](https://raw.githubusercontent.com/mity/cutest/master/include/cutest.h) into your project directory. 

Yes, Git users, this is an ugly cheat because it bypasses normal use of git and GitHub, but version control isn't part of the tutorial.

### Create and compile the test harness test_area.c

The absolute minimal test configuration is... no tests. The driver executable will be named `test_area` and CUTest generates the `main()` function automatically. This step lets you get the `main()` function running and make sure CUTest compiles without any distractions.yf
* First, create the minimal test harness `test_area.c` as follows:

```c
#include "cutest.h"
#include "area.h"

TEST_LIST = {
    { 0 }
};
```

* Compile the files area.c and test_area.c and create an executable with this command line:

```bash
$ gcc -std=c99 area.c test_area.c -o test_area
```

Compile and correct your source as many times as is necessary until `gcc` yields no output, which means success. There's no `main` in this code so it wouldn't work anyway.

* Run the file:

```bash
$ ./test_area
```

The output looks like this:

```bash

Summary:
  SUCCESS: All unit tests have passed.
```

#### Notes:

* The compile-only flag `-c` has been omitted, of course, because the goal is to create an executable.
* There is a new command-line option. The command-line flag `-o` is followed by the name you wish to give your executable. 


## Add a simple test to test_area.c

The first test couldn't be easier. It simply verifies that the constant PI, defined in the file `area.h`, is correct to 7 digits to the right of the decimal point.

To add a test:

1. Write a test function prototype returning void with void arguments
2. Add a description in quotes, followed by the the name of that test function only (not its entire signature) to the `TEST_LIST` array macro declaration
3. Implement the function
4. Add a TEST_CHECK macro to your test function. It is a macro that evaluates to a nonzero/zero result. (Another macro named TEST_CHECK_ will be discussed later.)
5. Recompile and run your tests

Here's that process in detail.

### 1. Add the prototype `void is_pi_accurate_to_7_digits(void);` above the `TEST_LIST` macro declaration:

Let's revisit the test harness test_area.c. 

To perform its auto-generation magic, CUTest expects all your test functions return `void` and contain a `void` parameter list. Add this prototype above the `TEST_LIST` macro:

```c
void pi_accurate_to_7_digits(void);

TEST_LIST = {
    { 0 }
};
```

### 2. Add a simple description in quotes, followed by a comma, and that function's name to the TEST_LIST macro. Enclose all this in curly braces. It is actually an array declaration so end each entry in the array with a comma too

The description appears when the unit tests are run.

```c
void pi_accurate_to_7_digits(void);

TEST_LIST = {
    { "PI accurate to 7 digits", pi_accurate_to_7_digits },
    { 0 }
};
```

### 3. Add the code for that function below the TEST_LIST macro. 

```c
TEST_LIST = {
    { "PI accurate to 7 digits?", pi_accurate_to_7_digits },
    { 0 }
};

void pi_accurate_to_7_digits(void)
{
    /* Your code goes here */
}

```

### 4. Include a TEST_CHECK macro, which is an expression evaluating to a boolean result

This is any old C function. You can also include arbitrary code. In this case the only thing we need is the `TEST_CHECK` macro, which must evaluate to nonzero in order to pass the unit test:

```c
void pi_accurate_to_7_digits(void)
{
    TEST_CHECK_(PI == 3.1415927);
}
```


* The completed test_area.c file looks like this:

```c
#include "cutest.h"
#include "area.h"

/* 1. Add function prototype returning void with void params. */
void pi_accurate_to_7_digits(void);

/* 2. Add description + function name to TEST_LIST array. */
TEST_LIST = {
    { "PI accurate to 7 digits", pi_accurate_to_7_digits },
    { 0 }
};

/* 3. Implement the function */
void pi_accurate_to_7_digits(void)
{

/* 4. Include TEST_CHECK expression evaluating to boolean result. */
    TEST_CHECK(PI == 3.1415927f);
}
```




    TEST_CHECK_(PI == 3.1415927, "PI should be %.07f. It is \n", PI);
