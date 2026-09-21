# Crash Course to C++ / Glossaries
## Table of Contents
- [Crash Course to C++ / Glossaries](#crash-course-to-c--glossaries)
  - [Table of Contents](#table-of-contents)
- [Basic Programming Terms](#basic-programming-terms)
  - [variables and data types](#variables-and-data-types)
    - [Frequently used data types](#frequently-used-data-types)
  - [declare, assign, initialize](#declare-assign-initialize)
  - [basic maths](#basic-maths)
  - [compile, build](#compile-build)
  - [error vs bug](#error-vs-bug)
  - [debug](#debug)
- [C++](#c)
  - [header file, source file](#header-file-source-file)
  - [Object Oriented Programming](#object-oriented-programming)
    - [class](#class)
    - [public, private](#public-private)
    - [method](#method)
    - [constructor](#constructor)
    - [destructor](#destructor)
    - [inherit](#inherit)
    - [override](#override)
  - [function](#function)
  - [pointer](#pointer)
- [JUCE](#juce)
  - [What is JUCE? Why are we using JUCE?](#what-is-juce-why-are-we-using-juce)
  - [Projucer](#projucer)
---

# Basic Programming Terms
## variables and data types
`variable` is a container or a box that stores certain data values.

- In C++, it is case-sensitive.

`randomVariable` and `randomvariable` are considered different variables.

- You cannot assign variables with same names.
```c++
int randomVariable=1;
float randomVariable=0.1;
// compile error (WRONG!)
```


`data type` is a type of data - are we storing integer? character? something else? `data type`declares this. 

### Frequently used data types
- `int`: stores integer. no decimals. 
- `float`: stores decimals. `0.1f`. 
    - `double`: stores decimals, but it can store larger number than float.
- `char`: stores one character. `a`, `1`. 
- `bool`: stores `true` or `false`. Useful to make an on-off switch. 

## declare, assign, initialize
- We `declare` a variable.
  ``` c++
  int randomVariable;
  ```
  Then we `assign` a variable.
  ``` c++
    randomVariable = 1;
  ```

- We `initialize` a variable as ~.
  ``` c++
  int randomVariable = 1;
  ```
  `initialize` is `declaring` and `assigning` a variable in one line.

## basic maths
- `==`: equal sign
  - `=` is only used for assigning (or instantiating) a variable.
  - `if (randomVariable == 2)`: is "`randomVariable` is 2" true?
- `++`: plus 1
- `--`: minus 1
- `+=`: plus number then assign
  - ``` c++
    int randomVariable = 1;
    randomVariable += 2;
    // randomVariable == 3
    
## compile, build
- `compile`: Computer can only read 0,1. We need to translate our code into 0 and 1s. If there is something that cannot be translated, it emits "compile error". 
- `build`: Build the code (or group of codes) into an executable file.
  - output: `.exe`, `.app`, `.dmg` files, etc. A runnable program!

## error vs bug
- `error`: something that makes you fail building. Usually *compile error*.
  - c.f. `runtime error`: something that makes your program crash and stop.
- `bug`: you succeeded building. You run the program. The program is not working in the way you intended. 
  - *I made a calculator, but it is telling me `1+2 == 5`!*

## debug
To investigate which code is making a bug, and when it is making a bug.
- debugging with `print` method
    ``` c++
    int variable = 0;
    for (int i = 0; i<4; i++){
        variable++;
    }
    ```
    Q1. How many times this for loop will be iterated?

    Q2. after for loop, what is the value of variable?

    ``` c++
    int variable = 0;
    for (int i = 0; i<4; i++){
        variable++;
        // PRINT variable
        std::cout<<variable;
    }
    std::cout<<variable;
    ```
    output:
    ```
    12344
    ```
    A1. 4

    A2. 4

---

# C++
## header file, source file
- header file: `.h` file. Briefly shows the structure of the code. You write the names of the class, class' methods, variables, etc.
  ```c++
  class Animal {
    private:
        std::string name;
        int age;
    public:
        Animal(std::string animal_name, int animal_age);
        virtual ~Animal(); // `virtual` here: just copy this for now. Explained in "override".

        virtual void walk(); // `virtual`: explained in "override".
        void sleep(int hours);
        float eat(std::string food_name, int amount);
  };
  ```
  We can see `class Animal` has two data members, constructor, destructor, and walk(), sleep(), eat() methods. We don't know the details yet.

- source file: `.cpp` file. The actual code that has "how does this code work?" written.
  It will have a whole source code of "how" walk(), sleep(), and eat() work!

## Object Oriented Programming
One of the programming styles. C++ supports object-oriented programming (OOP). Uses `class`.

### class
Most animals walk, sleep, and eat. Yet they all have different characteristics. They are all born, and they will all die.

Class is like a specification of a group. 

It includes `constructor`, `destructor`, and `method`.

If you create an `Animal` with a `constructor`, that is an `object`. Creating an object from a class is called `instantiate`.

```c++
Animal cat("Nabi", 3); // we instantiated an Animal object named "cat".
```
### public, private
`public` is open to everywhere. Any code can use it.

`private` can only be used by the code inside the class itself (its own methods). Code outside the class cannot touch it, and even child classes like `Dog` cannot.

```c++
Animal cat("Nabi", 3);
cat.walk();  // OK: walk() is public.
cat.name;    // compile error: name is private.
```

### method
Let's think of a function that works in the class for now.

### constructor
The specific method that creates an object. `Animal()`.

### destructor
The specific method that deletes the object. `~Animal()`.

### inherit
All animals walk, sleep, and eat. But a `Dog` is a specific kind of animal that can also bark. A `Bird` is a specific kind of animal that can also fly.

Instead of rewriting the code for walking, sleeping, and eating for every single type of animal, we can just `inherit` *(borrow)* everything from the Animal class.

- Parent Class (Base): `Animal`
- Child Class (Derived): `Dog`, `Bird`...

```
// simplified example: constructors are omitted.
class Dog : public Animal {
    public:
    // Dog automatically has walk(), sleep(), and eat() from Animal.
    void bark();
};
```

### override
What if we just want to change one method just a little bit? Like rabbits, that walk by hopping.

Then we `override` `walk()`: rewrite the internal details.

```
// simplified example: constructors are omitted.
class Rabbit : public Animal {
    public:
       // overriding walk method to hop instead of 'normal' walk
        void walk() override {
            // rabbit's hopping logic
        }
};
```

`override` only works if the parent's method has `virtual` in front of it (look at `virtual void walk();` in `Animal`). Think of it as the parent saying "children are allowed to change this one" in advance.

- You will see `virtual` and `override` a lot in JUCE. JUCE's classes mark some methods as `virtual`, and we `override` them to write our own audio processing code.
- `virtual ~Animal();`: when a class is meant to be inherited, put `virtual` in front of the destructor too. For now, just remember to do it.



## function
format: `return type` `function name`(`parameters`){`details`}
```c++
float eat(std::string food_name, int amount){
    float poop = 0.0;
    // let's say Animal poop half of what they ate.
    for (int i = 0; i<amount; i++){
        poop += 0.5; 
    }
    return poop;
}
```

`float`: return type. we are returning `poop`, which is a float variable.

`eat`: function name. Call `eat()` to return how much poop that animal had for eating.

`std::string food_name, int amount`: parameters to put. To call `eat()`, you should know the food name and the amount of food to eat.

```c++
eat("alfalfa_hay", 5);
// this function will return 2.5f.
```

- `void` in `return type`: if there is no certain thing to return, we use `void`. you can just write `return;` since we are returning nothing!


## pointer
Think of a shortcut or a quick link. It refers to the original variable.
- add `*` to any data type, and that's the pointer to that data type variable.
- `&variable` is the address to the variable. 
``` c++
int* pointer; // I will pointing a integer variable via "pointer" variable.
int var = 1; // I initialized a integer variable "var" as 1.
pointer = &var; // "pointer" variable is storing the address of "var".
```
now, because `pointer` is a quick link to `var`, `*pointer` will change as `var` change.

---

# JUCE
## What is JUCE? Why are we using JUCE?
JUCE is a C++ based framework that makes audio programming easier.

In class, we learned about bit depth, sampling rate, ADC, and DAC. If we had to build all of that from scratch, we would spend most of our time on the plumbing before we could make any sound. With JUCE, we don't have to. We can focus on the part we actually care about: what to do with the sound.

JUCE also comes with many pre-defined functions, macros, and enums. Their names are quite intuitive, so the code is easier to read and write.

## Projucer
Projucer is a program that comes with JUCE. It manages your project files.

- It creates the backbone of your project: the folder structure and the build files (e.g. Makefile, Xcode project, Visual Studio project). You only need to write the source code.
- Once you create a project and set up the file structure, you can close the Projucer window while you code.
- You need to open it again when you add new files or change the project settings.