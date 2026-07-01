# Goldman Sachs OA — OOP MCQ Practice Set

> Note: These are NOT verbatim leaked questions (GS doesn't publicly recirculate exact OA text, and the negative-marking policy means agencies rarely reproduce it exactly). This set is built from the **topics candidates consistently report** across PYQ write-ups (Medium interview reports, PrepInsta, LeetCode Discuss, GfG placement archives) for the GS OA MCQ round. Language skews Java/C++ since that's what GS explicitly tests.

---

## 1. Encapsulation & Abstraction

**Q1.** What best distinguishes encapsulation from abstraction?
A) Encapsulation hides implementation, abstraction hides data
B) Encapsulation binds data and methods together; abstraction hides complexity by exposing only essential features
C) They are the same concept
D) Abstraction is achieved only via private members
**Answer: B**

**Q2.** In Java, which access modifier provides the strongest encapsulation for a class field?
A) public  B) protected  C) private  D) default
**Answer: C**

---

## 2. Polymorphism

**Q3.** Method overloading is an example of:
A) Runtime polymorphism  B) Compile-time polymorphism  C) Inheritance  D) Encapsulation
**Answer: B**

**Q4.** Runtime polymorphism in Java is achieved through:
A) Static methods  B) Method overloading  C) Method overriding + dynamic dispatch  D) Constructors
**Answer: C**

**Q5.** In C++, which is required to achieve runtime polymorphism?
A) Operator overloading  B) Virtual functions + base class pointer/reference  C) Templates  D) Friend functions
**Answer: B**

**Q6.** What is the output behavior when a base class pointer holds a derived class object and calls a non-virtual function that's overridden in the derived class?
A) Derived class version is called (dynamic binding)
B) Base class version is called (static binding)
C) Compile error
D) Undefined behavior always
**Answer: B**

---

## 3. Virtual Functions & Abstract Classes

**Q7.** A pure virtual function in C++ is declared as:
A) `virtual void foo();`
B) `virtual void foo() = 0;`
C) `abstract void foo();`
D) `void foo() = virtual;`
**Answer: B**

**Q8.** A class containing at least one pure virtual function is called:
A) Concrete class  B) Abstract class  C) Final class  D) Static class
**Answer: B**

**Q9.** Can you instantiate an abstract class directly in Java/C++?
A) Yes, always  B) No, never  C) Only with a default constructor  D) Only if it has no methods
**Answer: B**

**Q10.** What is the main difference between an abstract class and an interface in Java (pre-Java 8 concept check)?
A) No difference at all
B) Abstract class can have constructors and instance state; interface (traditionally) cannot
C) Interfaces support multiple inheritance of implementation, abstract classes don't
D) Abstract classes can't have any methods
**Answer: B**

---

## 4. Constructors & Destructors

**Q11.** Which constructor is called first in a derived class object creation (C++)?
A) Derived class constructor  B) Base class constructor  C) Both simultaneously  D) Depends on compiler
**Answer: B**

**Q12.** What is a copy constructor used for?
A) To destroy an object  B) To create a new object as a copy of an existing object  C) To overload operators  D) To initialize static members
**Answer: B**

**Q13.** In C++, when should a destructor be declared `virtual`?
A) Never  B) When the class will be used polymorphically (base class pointer deleting derived object)  C) Only for abstract classes  D) Only for structs
**Answer: B**

**Q14.** What happens if a base class destructor is NOT virtual and you delete a derived object through a base class pointer?
A) Compile error
B) Only base class destructor runs → resource leak / undefined behavior for derived members
C) Only derived destructor runs
D) Nothing, it's always safe
**Answer: B**

---

## 5. Inheritance

**Q15.** The "Diamond Problem" arises in which type of inheritance?
A) Single inheritance  B) Multiple inheritance  C) Hierarchical inheritance  D) Multilevel inheritance
**Answer: B**

**Q16.** How does Java avoid the diamond problem (that C++ multiple inheritance has)?
A) Java doesn't allow multiple inheritance of classes; interfaces resolve default method conflicts explicitly
B) Java doesn't support inheritance at all
C) Java uses virtual base classes automatically
D) Java resolves it silently without programmer intervention
**Answer: A**

**Q17.** In C++, the diamond problem is resolved using:
A) `final` keyword  B) `virtual` inheritance  C) `static` inheritance  D) Templates
**Answer: B**

---

## 6. Binding & Dispatch

**Q18.** Static binding is resolved at:
A) Runtime  B) Compile time  C) Link time  D) Load time
**Answer: B**

**Q19.** Which of these uses dynamic (late) binding in Java by default?
A) Static methods  B) Private methods  C) Instance (non-static, non-final, non-private) methods  D) final methods
**Answer: C**

---

## 7. Overloading vs Overriding

**Q20.** Which is TRUE about method overriding?
A) Must have the same method signature (name + parameters) as the parent
B) Can have a different return type unrelated to the parent's
C) Cannot exist in inheritance
D) Requires the `static` keyword
**Answer: A**

**Q21.** Which is TRUE about method overloading?
A) Requires inheritance
B) Same method name, different parameter list, within the same class (or scope)
C) Resolved at runtime
D) Requires virtual keyword
**Answer: B**

---

## 8. Keywords & Modifiers

**Q22.** The `final` keyword in Java, when applied to a class, means:
A) The class cannot be instantiated
B) The class cannot be subclassed
C) All methods become static
D) The class becomes abstract
**Answer: B**

**Q23.** A `static` member in a class:
A) Belongs to each individual object separately
B) Belongs to the class itself, shared across all instances
C) Can only be accessed by constructors
D) Is destroyed when any one object is destroyed
**Answer: B**

**Q24.** What does the `this` pointer/reference refer to?
A) The base class object
B) The current instance invoking the method
C) A static copy of the class
D) The parent constructor
**Answer: B**

**Q25.** A `friend` function in C++:
A) Is a member function of the class
B) Can access private/protected members of the class despite not being a member
C) Must be virtual
D) Is inherited by derived classes
**Answer: B**

---

## 9. Operator Overloading & Misc.

**Q26.** Which operator CANNOT be overloaded in C++?
A) `+`  B) `[]`  C) `::` (scope resolution)  D) `==`
**Answer: C**

**Q27.** What is "object slicing" in C++?
A) Splitting an object into multiple threads
B) When a derived object is assigned to a base object by value, losing derived-specific data
C) A garbage collection technique
D) A way to overload constructors
**Answer: B**

---

## 10. SOLID Principles (frequently referenced by GS candidates)

**Q28.** The "S" in SOLID stands for:
A) Static Binding  B) Single Responsibility Principle  C) Structured Design  D) Stateless Interface
**Answer: B**

**Q29.** The Liskov Substitution Principle states that:
A) Subclasses should be substitutable for their base classes without breaking correctness
B) Classes should have only one reason to change
C) Interfaces should be small and specific
D) High-level modules should depend on abstractions
**Answer: A**

**Q30.** The Open/Closed Principle means classes should be:
A) Open for modification, closed for extension
B) Open for extension, closed for modification
C) Always public
D) Never inherited from

**Answer: B**

---

## Quick concept checklist to review before the OA
- Encapsulation vs Abstraction vs Data Hiding
- Compile-time (overloading) vs Runtime (overriding) polymorphism
- Virtual functions, pure virtual, vtable basics
- Abstract class vs Interface (Java) / abstract vs concrete class (C++)
- Constructor/destructor order in inheritance, virtual destructors
- Diamond problem + resolution (virtual inheritance in C++, interfaces in Java)
- Static vs dynamic binding
- Access specifiers (public/private/protected/default)
- `final`, `static`, `this`, `friend`, `const` keywords
- Operator overloading rules & restrictions
- Object slicing
- SOLID principles (esp. SRP, OCP, LSP — these show up in GS "design thinking" MCQs)
- Composition vs Inheritance ("has-a" vs "is-a")

*(HackerRank OA typically mixes 3–6 OOP MCQs into a larger set with DSA/OS/DBMS/networking MCQs, per multiple 2024–2026 candidate reports.)*
