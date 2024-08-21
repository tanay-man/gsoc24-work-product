# GSoC '24 Work Product
<p align="center">
  <img src="https://github.com/user-attachments/assets/75373aba-02a1-4d25-ba47-008b109da61b" alt="GSoC" width="164"/>
  <img src="https://github.com/user-attachments/assets/4bc20ad5-555a-4198-91cc-6ecd6848e7a2" alt="Python" width="668", height="164"/>
</p>

The following report summarizes the work done during Google Summer of Code 2024 along with the results, scope for improvements and future work. This also serves as the final project report with all the contributions.

## Basic Info
- **Name:** Tanay Manerikar
- **Email:** [manerikartanay@gmail.com](manerikartanay@gmail.com)
- **GitHub Username:** [tanay-man](https://github.com/tanay-man)
- **LinkedIn:** [Tanay Manerikar](https://www.linkedin.com/in/tanay-manerikar-a71911176/)
- **Organization:** Python Software Foundation
- **Sub-Organization:** LPython
- **Project Title:** Implementation of Classes and OOP Features
- **Project Link:** [LPython GitHub Repository](https://github.com/lcompilers/lpython)
- **Project Mentors:** [Ondřej Čertík](https://github.com/certik), [Thirumalai Shaktivel](https://github.com/Thirumalai-Shaktivel)

## Project Synopsis
My project aimed to accomplish the following goals:
- Extend the current structs implementation to include class methods.
- Implement inheritance, including vtables for dynamic dispatch.
- Write tests to ensure the implementation is free of bugs.

## Work Done

**PRs Merged:**

- [StructType Node Update](https://github.com/lcompilers/lpython/pull/2743):  
  This patch updated the ASR node of `StructType` to include the types of all members and methods. This was done to decouple the type of a class/struct from its symbol. In the next step, we plan to remove the symbol altogether from this node, allowing it to be used in a global scope.

- [Default Constructors](https://github.com/lcompilers/lpython/pull/2750):  
  This patch added support for default constructors. One of the major issues I solved was determining the type of the `self` argument. It cannot be of type class as it is not yet defined. We resolved this by creating an empty class and assigning `self` that type.

- [Basic Classes Implemented](https://github.com/lcompilers/lpython/pull/2775):  
  This was a significant PR that implemented `self`, member functions, and the `__init__` function. This allowed running code containing classes without inheritance using LPython.

- [Enabled Method Calling Inside Other Functions](https://github.com/lcompilers/lpython/pull/2782):  
  After the previous patch, there was a bug that prevented LPython from compiling any code with method calls inside other functions. This bug was fixed by this PR.

- [Fixed Copy Semantics of Objects](https://github.com/lcompilers/lpython/pull/2784):  
  LPython doesn’t allow creating references or shallow copies of variables for performance reasons; thus, we defaulted to making deep copies of all objects. This behavior differs from CPython, where shallow copies are made. Implementing reference counting was considered, but due to its high performance cost, it was not deemed a good default choice for LPython. Consequently, it was decided to completely disallow object copies in the same scope.

- [Implemented Multi-Level Attributes](https://github.com/lcompilers/lpython/pull/2794):  
  Previously, there was no support for multi-level attribute function calls, e.g., `self.attr.method()`. This patch enabled compiling code of this pattern.

- [StructConstructor Refactor](https://github.com/lcompilers/lpython/pull/2795):  
  The `StructConstructor` node needs to be replaced by the `__init__` function call for correct initialization of the object's attributes. However, doing this in the AST-to-ASR conversion was semantically lowering the language. Hence, this replacement was refactored in the ASR-to-ASR pass called `ClassConstructor`.

- [Initial Implementation of Inheritance and Polymorphic Function Calls](https://github.com/lcompilers/lpython/pull/2801):  
  This was a significant patch that implemented basic inheritance and polymorphic function calls. It included the implementation of `super()`. It was decided not to support multiple inheritance to avoid complexity. Additionally, it enabled function calls of the type `fn(arg: Base)` to accept arguments of type `Derived`. This was achieved through implicit casting of the argument to the Base class.  

**Issues:**

- [#2680](https://github.com/lcompilers/lpython/issues/2680)
- [#2722](https://github.com/lcompilers/lpython/issues/2722)
- [#2723](https://github.com/lcompilers/lpython/issues/2723)
- [#2776](https://github.com/lcompilers/lpython/issues/2776)
- [#2777](https://github.com/lcompilers/lpython/issues/2777)
- [#2781](https://github.com/lcompilers/lpython/issues/2781)
- [#2797](https://github.com/lcompilers/lpython/issues/2797)
- [#2799](https://github.com/lcompilers/lpython/issues/2799)
- [#2800](https://github.com/lcompilers/lpython/issues/2800)

## Output
Lpython is able to successfully compile and run code using the OOP paradigm.  
Example:
```py
from lpython import i32

class Base():
    def __init__(self:"Base"):
        self.x : i32 = 10

    def get_x(self:"Base")->i32:
        print(self.x)
        return self.x
    
def get_x_static(d: Base)->i32:
    print(d.x)
    return d.x

class Derived(Base):
    def __init__(self: "Derived"):
        super().__init__()
        self.y : i32 = 20 

    def get_y(self:"Derived")->i32:
        print(self.y)
        return self.y       


def main():
    d : Derived = Derived()
    x : i32 = get_x_static(d)
    assert x == 10
    x = d.get_x()
    assert x == 10
    y: i32 = d.get_y()
    assert y == 20

main()
```
Output:
```
(lp) tanay@tanay-man:~/lcompilers/lpython$ lpython integration_tests/class_06.py 
10
10
20
```
