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
The idea is a clear separation between compile-time and runtime and unified syntax for everything.

Let's see whether this simplifies.
At this point, I am unsure whether this works or not but I can not point out the fatal flaw.

The idea occurred to me while developing my build-system HMake.
My build-system executes in rounds.
It does 3 rounds of topological sorting and execution in one execution.
We do similar in our programming language.
But we need just 2 rounds.

The code executes in 2 rounds.
The code in the first round is converted to normal C++ which is then compiled.
The first round is called compile-time while the second round is called runtime.

Following is an example

### Example 1

```cpp
fn1 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(5);
}
```

```fn``` creates a runtime function.
While ```fn1``` creates a template-like function that can be used at both
compile-time and runtime and can also be instantiated while ```fn2``` creates a strictly
compile-time function.
This is first translated to:

```cpp
fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(5);
}
```

And then compiled.
```fn1``` declares equivalent to ```template``` function which is not instantiated if not used.
Let us use it.

### Example 2

```cpp
fn1 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max(a,b)());
}
```

This code is first translated into the following and then compiled.

```cpp
fn max(int32 a, int32 b) -> bool
{
    if (a > b)
    {
        return a;    
    }
    return b; 
}

fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max(a, b));
}
```

In this line ```std::print(max(a,b)());```,
we are using a compile-time construct at runtime.
```fn1``` construct can be used at runtime as well as compile-time.
This is called instantiation.
This line instantiates ```max``` function with 2 int32 arguments.
In instantiation, all compile-time instructions are executed and
any runtime code is embedded in ```gen``` compile-time function.
```gen``` is a special compile-time function that embeds code to the current function.
Later on, more compile-time functions would be added which could modify
any aspect of the program.
The last parenthesis calls the function.
Empty parenthesis means the same arguments while types are automatically deduced.
We could have directly used the long-form
```std::print(max(int32, int32)(a, b));``` instead.

Now, let us modify the ```max``` definition such that if both arguments are compile-time,
then it runs at compile-time.
Not only we can pass compile-time values to runtime ```fn1``` functions,
but we can also execute these functions at compile-time just like ```constexpr```.

### Example 3

```cpp
fn1 max(auto a, auto b) -> bool
{
    if (a > b)
    {
        return a;        
    }
    return b;    
}

fn main() -> int
{
    _int32 a = 3;
    _int32 b = 5;
    std::print(_max(a,b)());
}
```

```_``` is a special operator in 3Rus.
It can be used to declare compile-time constants.
Also, it can be used to execute ```fn1``` functions.
Otherwise, ```fn1``` are compile-time functions.
And using them in runtime block instantiates them.
Also, we can not execute a compile-time function with runtime arguments.
But since we have made our variables as compile-time,
we can execute the ```max``` function at compile-time.
We made no changes in ```max``` function but it is prepended with ```_```
operator to invoke it at compile-time.

This code is first translated into the following:

```cpp
fn main() -> int
{
    std::print(5);
}
```

So, ```fn``` creates a runtime function, while ```fn2``` creates a compile-time function.
While ```fn1``` can be both a compile-time and runtime.

The first phase is the translation phase or instantiation phase.
There should be at least one runtime function to initiate the translation process.

Sometimes, we want to have different compile-time arguments vs runtime arguments.
In these cases, we can have two parameter lists with ```fn1``` declarations.

### Example 4

```cpp
fn1 max(bool invert = false)
(auto a, auto b) -> bool
{
    if1 (invert)
    {
        if (a > b)
        {
            return b;
        }
        return a;
    }
    else
    { 
        if (a > b)
        {
             return a;        
        }
        return b;    
    }
   
}

fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max(true)(a,b));
}
```

This is translated to

```cpp
// mangled name needs to be generated. true needs to become part of the function 
// signature. As max(true) is completely different function from max(false).
fn max_true(int32 a, int32 b) -> bool
{
    if (a > b)
    {
        return b;        
    }
    return a;    
}

fn main() -> int
{
    int32 a = 3;
    int32 b = 5;
    std::print(max_true(a, b));
}
```

3Rus has very good support for reflection.
You can modify any aspect of your program.
In the following example, we print the struct members.
We could have serialized instead as easily.

### Example 5

```cpp

// A strictly runtime struct
struct Book
{
    int64 publishDate;
    string author;
}

// A compile-time function that needs to be instantiated to be used at runtime
fn1 print(auto t) -> string
{
    if1 (t.hasMember(notPrint))
    {
        this.dontSelect = false;
    }
    
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

We have 3 different mechanisms to declare struct in 3Rus.
```struct``` and ```class``` create runtime types.
While ```struct1``` and ```class1``` types are compile-time that can be instantiated at runtime.
While ```struct2``` and ```class2``` can only be used at compile-time.

Compile-time variables do not exist at runtime.
We cannot use runtime constructs at compile-time.

When invoking the function ```print(k)()```, we don't need to pass the variable ```k```
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

fn main() -> int
{
    Book k;
    std::print(k);
}
```

When the compiler is translating i.e. in round 1, when it reaches this line
```std::print(print(k)());```, it instantiates ```print``` function .
If we had declared compile-time variable ```notPrint``` in our type, then calling
this function would have errored out.
```dontSelect``` is a special variable which when set to false causes the function call to not be
selected.
So, this would have been a function not found error.
Because we don't have another overload of ```print``` declared.
```for1( auto[type, name] : t.members())``` iterates on the type ```t``` members.
```str += name;``` is a runtime statement in a compile-time ```for``` loop.
This is actually ```gen(str += name)```, but we did not specify ```gen```.
Any runtime code can be generated using this.
But we did not specify.
We can automatically infer it as we are instantiating.

We don't have any new syntax for templates, macros, constexpr, or reflection.
macros are needed in C++ for code generation.
In 3Rus, we use ```gen``` function for that purpose.
```gen``` embeds code to the body of the current round 0 function.
But, we can also add some other API to modify other round 0 functions or types.
Besides we can have API besides ```members()``` to introspect any aspect of types.

```fn2``` is equivalent to consteval function.
While ```fn1``` is templated constexpr function.
While ```fn``` is your normal function.

## Variadics

3Rus has good support for variadics.
A function can receive an unbound number of arguments if it is declared to be accepting
variadic parameter pack.
`auto...` is used for declaring a parameter pack for any type.
While `T...` can be used for declaring a parameter pack of any type `T`.
`fn` function can not have variadic parameters.
Only `fn2` and `fn3` can be variadics.
A parameter pack supports a few compile-time operations such as
`size()` and `operator[](uint32)`.
These operations can be used to perform iteration on the parameter pack
or to index certain elements in a parameter pack of multiple elements.
Like C++, variadics are available in multiple contexts.

### Example 6

```cpp
fn1 printMultiple(auto... args) -> void
{
    for1(auto i : args)
    {
        std::println(i);
    }
}

fn main() -> int
{
    printMultiple(3, 3.5, "Hello", 2, 3.5); 
}
```

This code is first translated to:
```cpp
fn1 printMultiple(int32 a0, float32 a1, const char* a2, int a3, float a4) -> void
{
    std::println(a1);
    std::println(a2);
    std::println(a3);
    std::println(a4);
}

fn main() -> int
{
    printMultiple(3, 3.5, "Hello", 2, 3.5); 
}
```

Suppose, we want to sum multiple integers.
We can write a generic function for that as follows:

### Example 7
```cpp
fn1 sum(uint32... args) -> uint32
{
    uint32 sum = 0;
    for1(uint32 i : args)
    {
        sum += i;
    }
}    

fn main() -> int
{
    std::print(sum(32, 43, 44, 25, 32));
}
```
This generates the following code:

```cpp
fn sum(uint32 a1, uint32 a2, uint32 a3, uint32 a4, uint32 a5)
{
    uint32 sum = 0;
    sum += a1;
    sum += a2;
    sum += a3;
    sum += a4;
    sum += a5;
}

fn main()
{
    std::print(sum(32, 43, 44, 25, 32));
}
```

Because all values are known at compile-time,
we could have executed the very same function at compile-time
and this would have generated this line in the `main` function
`std::print(176)`.

### Type Generics
3Rus supports type generics.
You can use `class1` or `struct1` to define parameterizable types.
The passed arguments during instantiations of such classes or structs can be used in multiple
contexts just like in C++.

```cpp
class1 Stack(auto T) 
{

private:
    std::vector<T> elems; // elements
public:

    fn push(T const& elem) -> void; // push element
    
    fn pop() -> void; // pop element
    
    fn top() const -> T const&; // return top element
    
    fn empty() const -> bool
    { // return whether the stack is empty
        return elems.empty();
    }
};


fn Stack(auto T)::push (T const& elem) -> void
{
    elems.push_back(elem); // append copy of passed elem
}

fn Stack(auto T)::pop () -> void
{
    assert(!elems.empty());
    elems.pop_back(); // remove last element
}

fn Stack(auto T)::top () const -> T const&
{
    assert(!elems.empty());
    return elems.back(); // return copy of last element
}

// The above class1 can be used as
fn main() -> int
{
    Stack<int32> intStack; // stack of ints
    Stack<std::string> stringStack; // stack of strings
    
    // manipulate int stack
    intStack.push(7);
    std::cout << intStack.top() << ’\n’;
    // manipulate string stack
    stringStack.push("hello");
    std::cout << stringStack.top() << ’\n’;
    stringStack.pop();
}
```
The above will be translated to the following code:


```cpp
class1 Stack(int32) 
{

private:
    std::vector<int32> elems; // elements
public:

    fn push(int32 const& elem) -> void; // push element
    
    fn pop() -> void; // pop element
    
    fn top() const -> int32 const&; // return top element
    
    fn empty() const -> bool
    { // return whether the stack is empty
        return elems.empty();
    }
};


fn Stack(int32)::push (int32 const& elem) -> void
{
    elems.push_back(elem); // append copy of passed elem
}

fn Stack(int32)::pop () -> void
{
    assert(!elems.empty());
    elems.pop_back(); // remove last element
}

fn Stack(int32)::top () const -> int32 const&
{
    assert(!elems.empty());
    return elems.back(); // return copy of last element
}

// The above class1 can be used as
fn main() -> int32
{
    Stack<int32> intStack; // stack of ints
    Stack<std::string> stringStack; // stack of strings
    
    // manipulate int stack
    intStack.push(7);
    std::cout << intStack.top() << ’\n’;
    // manipulate string stack
    stringStack.push("hello");
    std::cout << stringStack.top() << ’\n’;
    stringStack.pop();
}
```
`class2` is a strictly compile-time construct.
This can not be instantiated at runtime.
This can only be used in `fn2` functions.
It can only have `fn2` functions and not have `fn` or `fn1`.

```class```, ```class1``` can have any of ```fn```, ```fn1``` or ```fn2```.
classes can have compile-time variable.
Compile-time variables or functions declared inside a class can be modified
using ```_``` operator.
```class1``` and ```class2``` objects can be created at compile-time.
We can modify the compile-time variables of types through both type syntax and
object syntax.
Doing it through type syntax modifies the defaults while doing it through object syntax
modifies the compile-time variable only for that object.

### Example 8

```cpp
struct Cat
{
    _string color = "white";
    fn1 print() -> void
    {
        std::print("The color of all cats is " + color);
    }
};

fn main()
{
    Cat cat;
    _Cat cat2;
    
    // no need for _ for operations on compile-time object.
    cat2.color = "brown"; 
    _Cat cat3; // color = "white" since "brown" was set just for cat2
    
    // Changes globally
    _Cat.color = "black";
    
    // print function is instantiated with "black" color
    cat.print()();
    _Cat.color = "blue";
    
    // Will print black. Once instantiated, the function is not instantiated
    // again
    cat.print()();
}    
```
This code is first translated to:

```cpp
struct Cat
{
    fn print() -> void
    {
        std::print("The color of all cats is " + "black");
    }
};

fn main()
{
    Cat cat;
    cat.print();
}
```

3Rus also supports partial specialization.
The rules for partial specialization are similar to C++.

### Example 9

```cpp
struct1 stack(auto T)
{
};

// Partial Specialization for stack of pointers
// We can also do code-gen based on T using 
// if1 if this definition of the stack is instantiated
struct1 stack(auto T)(T*)
{
    // if1 is supported in class context and global context as well
    if1 (T.type == string)
    {
        // Code instantiated only if stack<string*> is instantiated
    }
};


struct Ball
{
};

// Following is how we can provide a custom definition for stack<Ball>
// Full specialization for stack<Ball>
struct1 stack()(Ball)
{
};
```

So, the user has both options.
Either use compile-time code-gen or partial specialization.
Compile-time code-gen is more effective if we are providing a definition
of the class but partial specialization is more effective for extending foreign types.


Only reflection / compile-time remains to be designed for the 0.1 release.

## Acknowledgements
Folks at #include Discord server have provided valuable feedback.
Special thanks to them.

Special thanks to Discord user madeso for suggesting renaming `fn2` to `fn1`,
`struct2` to `struct1`, `class2` to `class1`, `fn3` to `fn2`, `struct3` to `struct2`
and `class3` to `class2`.

Special thanks to Discord user madeso for providing helpful feedback and pointing
out 2 typos in the design document.