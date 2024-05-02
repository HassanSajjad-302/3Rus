
## Introduction

3Rus is a new programming language design document.
It is very informally written.
And I am not sure whether this would work or not.


I find Rust not having any undefined behavior is a great feature.
But I have heard that C++ has superior generics.
The original plan was to develop this as a successor to Rust iterating on Rust's inferior generics
and thus making the perfect state-of-the-art programming language.
Hence, its name 3Rus.
But currently, I don't know Rust.
Hence, the syntax is C++-esque.
The goal is to be extremely simpler compared to C++.

What makes the development with C++ complex?
Among a lot of things macros, templates, constexpr, consteval, concepts, and now reflection.
Learning all of the above thoroughly is not an easy task.
But all of the above has its usages.
So how do we simplify C++ or Rust?

I have a new idea.
The idea is double-function syntax.

Let's see whether this simplifies. Again I am unsure whether this works or not.

The idea occurred to me while developing my build-system HMake.
My build-system executes in rounds.
It does 3 rounds of topological sorting and execution in one execution.
We do similar in our programming language.
But we need just 2 rounds.

The cost of double-function syntax is extra braces.
But the benefits are immense.
In the following example, we print the struct members.
We could have serialized instead as easily.

The code executes in 2 rounds.
The code in the first round is converted to normal C++ which is then compiled.

Following is an example


```cpp

struct Book
{
}
{
    int64 publishDate;
    string author;
}

fn print(T t) -> void
{
    if1 (t.hasMember(notPrint))
    {
        this.dontSelect = false;
    }
}
(--) -> void
{
    string str;
    for1( auto[type, name] : t.members())
    {
        if1 (type == string)
        {
            str += name;     
        }
        else
        {
            str += std::toStr(name);
        }
    }
    return str;
}

int main()
{
    Book k;
    std::print(print(k)());
}
```

In 3Rus, there are some extra keywords like ```for1``` and ```if1```.
These indicate ```for``` and ```if``` constructs for round 1.
When using round 1 constructs, round 0 data structures can not be supplied.
Every function can have 2 parts round 1 part and round 0.
The first part is round 1 part while the second part is round 0 part.
In round 1 part, we can only define round 1 variables.
These variables do not exist in round 0.
We can not use round 0 constructs in the round 1 part.
However, we can define round 1 variables in round 0.
And can use round 1 constructs in round 0.
These variables do not exist in round 0.
So, every function has two parts, compile-time and runtime.
compile-time does not exist at runtime.

```fn print(T t) -> void``` declares the parameters and return type of the round 1 part
of the ```print``` function while
```(--) -> void``` declares the parameters and return type of the round 0 part of the
```print``` function.
```(--)``` means that round 1 has the same arguments as round 0.
Also when invoking the function ```print(k)()```, we don't need to pass the variable ```k```
again since the function is accepting the same arguments for both rounds.
Thus second parenthesis is empty.


The above code is first translated into the following and then compiled.

```cpp
struct Book{
    int 64 publishDate;
    string author;
}

fn print(Book t) -> void
{
    string str;
    str += std::toStr(publishDate);
    str += author;
    return str;
}

int main()
{
    Book k;
    std::print(k);
}
```
When the compiler is translating i.e. in round 1, when it reaches this line
```std::print(print(k)());```, it instantiates ```print``` function by calling the
round 1 part of the ```print``` function passing it the ```k``` variable.
Then it reaches the round 0 block of the ```print``` function.
Any round 1 variables declared in the round 1 part are also available in the round 0 part.
If we had declared round 1 variable ```notPrint``` in our type, then calling
this function would have errored out.
```dontSelect``` is a special variable which when set to false causes the function call to not be
selected.
So, this would have been a function not found error.
Because we don't have another overload of ```print``` declared.
```for1( auto[type, name] : t.members())``` iterates on the type ```t``` members.
```str += name;``` is a round 0 statement in a round 1 ```for``` loop.
This is actually ```gen(str += name)```, but we did not specify ```gen```.
```gen``` is a special function.
It can be executed only at round 1.
And round 0 code can be generated using this.
But we did not specify.
We can automatically infer it as we are in the round 0 part.

We don't have any new syntax for templates, macros, constexpr, or reflection.
macros are needed in C++ for code generation.
In 3Rus, we use ```gen``` function for that purpose.
```gen``` embeds code to the body of the current round 0 function.
But, we can also add some other API to modify other round 0 functions or types.
Besides we can have API besides ```members()``` to introspect any aspect of types.

In 3Rus, every instruction belongs to either round 1 or round 0.
This way we achieve a very clean design by having a clear separation between compile-time and runtime.
If only one part of the function is declared, then this is equivalent to ```consteval``` function
i.e. it is considered a round 1 function only and can not be called in round 0.

But sometimes, we want to have the same definition for compile-time and runtime i.e. equivalent to
```constexpr``` function, a function that can be used in both compile-time and runtime contexts.
I am thinking about this.
Currently, we need to have two bodies for both.

We can extend this for variadics, type generics, type specialization, etc.