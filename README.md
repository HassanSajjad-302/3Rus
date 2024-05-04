## Introduction

3Rus is a new experimental programming language design document.
It is very informally written.

I find Rust not having any undefined behavior is a great feature.
But I have heard that C++ has superior generics.
The original plan was to develop this as a successor to Rust iterating on Rust's inferior generics
and thus making the perfect state-of-the-art programming language.
Hence, its name 3Rus.
But currently, I don't know Rust.
Hence, the syntax is C++-esque.
The goal is to be extremely simpler compared to C++.

What makes the development with C++ complex?
Among a lot of things macros, templates, constexpr, consteval, constinit, concepts,
and now reflection.
Learning all of the above thoroughly is not an easy task.
But all of the above has its usages.
So how do we simplify C++ or Rust?

I have a new idea.
The idea is clear separation between compile-time and runtime.

Let's see whether this simplifies.
At this point, I am unsure whether this works or not but I can not point out the fatal flaw.

The idea occurred to me while developing my build-system HMake.
My build-system executes in rounds.
It does 3 rounds of topological sorting and execution in one execution.
We do similar in our programming language.
But we need just 2 rounds.

In the following example, we print the struct members.
We could have serialized instead as easily.

The code executes in 2 rounds.
The code in the first round is converted to normal C++ which is then compiled.
The first round is called compile-time while the second round is called runtime.

Following is an example

```cpp
fn2 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn3 main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(5);
}
```

This is first translated to:

```cpp
fn3 main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(5);
}
```

And then compiled.
```fn2``` declares equivalent to ```template``` function which is not instantiated if not used.
Let us use it.

```cpp
fn2 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn3 main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max(a,b)());
}
```

This code is first translated to following and then compiled.

```cpp
fn2 max(int32 a, int32 b) -> bool
{
    if (a > b)
    {
        return a;    
    }
    return b; 
}

fn3 main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max(int32, int32)(a, b));
}
```

In this line ```std::print(max(a,b)());```,
we are using a compile-time construct at runtime.
```fn2``` construct can be used at runtime as well as compile-time.
This is called instantiation.
This line instantiates ```max``` function with 2 int32 arguments.
In instantiation, all compile-time instructions are executed and
any runtime code is embedded in ```gen``` compile-time function.
```gen``` is a special compile-time function that embeds code to the current function.
Later on more compile-time functions would be added which could modify
any aspect of the program.
Last parenthesis calls the function.
Empty parenthesis means same arguments while types are automatically deduced.
We could have had directly used the long-form
```std::print(max(int32, int32)(a, b));``` instead.

Now, let us modify the ```max``` definition such that if both arguments are compile-time,
then it runs at compile-time.
Not only we can pass compile-time values to runtime ```fn2``` functions,
but we can also execute these functions at compile-time just like ```constexpr```.

```cpp
fn2 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn3 main() -> int
{
    _int32 a = 3;
    _int32 b = 5;
    std::print(_max(a,b)());
}
```
```_``` is a special operator in 3Rus.
It can be used to declare compile-time constants in runtime block.
Also, it can be used to execute ```fn2``` functions.
Otherwise, ```fn2``` are compile-time functions.
And using them in runtime block instantiates them.
Also, we can not execute a compile-time function with runtime arguments.
But since we have made our variables as compile-time, 
we can execute the ```max``` function at compile-time.

This code is first translated to the following:

```cpp
fn3 main() -> int
{
    std::print(5);
}
```



```cpp

// A strictly runtime struct
struct3 Book
{
int64 publishDate;
string author;
}

// A compile-time function that needs to be instantiated to be used at runtime
fn2 print(auto t) -> void
{
if1 (t.hasMember(notPrint))
{
this.dontSelect = false;
}
}
(--) -> string
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

In 3Rus, there are some extra keywords like ```for1```, ```if1```.
These indicate ```for``` and ```if``` constructs for compile-time.
This is like ```if constexpr``` in C++.
Runtime data structures cannot be used with compile-time constructs.

We have 3 different mechanisms to define functions in 3Rus.
```fn``` creates compile-time function.
A compile-time function can only be used in compile-time contexts and cannot have runtime constructs.
```fn2``` creates a compile time function.
But this function can be supplied with runtime arguments and instantiated at runtime.
```fn3``` creates a strictly runtime function.
Similarly, ```struct``` and ```class``` create compile-time types.
While, ```struct2``` and ```class2``` types are compile-time that can be instantiated at runtime.
While, ```struct3``` and ```class3``` can only be used at runtime.

Compile-time variables do not exist at runtime.
We cannot use runtime constructs at compile-time.

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