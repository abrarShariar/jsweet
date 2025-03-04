JSweet Language Specifications
==============================

Version: 1.0.0

Author: Renaud Pawlak

JSweet JavaDoc API: http://www.jsweet.org/core-api-javadoc/

Content
-------

-   [Basic concepts](#basic-concepts)
    -   [Core types and objects](#core-types-and-objects)
    -   [Classes](#classes)
    -   [Interfaces](#interfaces)
    -   [Enums](#enums)
    -   [Indexed objects](#indexed-objects)
    -   [Globals](#globals)
    -   [Optional parameters](#optional-parameters)
    -   [Ambient declarations](#ambient-declarations)
-   [Auxiliary types](#auxiliary-types)
    -   [Functional types](#functional-types)
    -   [Object types](#object-types)
    -   [String types](#string-types)
    -   [Tuple types](#tuple-types)
    -   [Union types](#union-types)
-   [Semantics](#semantics)
    -   [Main methods](#main-methods)
    -   [Initializers](#initializers)
    -   [Name clashes](#name-clashes)
    -   [Variable scoping in lambda expressions](#variable-scoping-in-lambda-expressions)
    -   [Scope of *this*](#scope-of-this)
-   [Packaging](#packaging)
    -   [Packages and modules](#packages-and-modules)
    -   [Root packages](#root-packages)
    -   [External modules](#external-modules)
-   [Appendix 1: JSweet transpiler options](#appendix-1-jsweet-transpiler-options)

Basic concepts
--------------

This section presents the JSweet language basic concepts. One must keep in mind that JSweet, as a Java-to-JavaScript transpiler, is an extension of Java at compile-time, and executes as JavaScript at runtime. Thus, most Java typing and syntactic constraints will apply at compile time, but some JavaScript semantics may apply at runtime. This document will mention when Java syntactic constructs and semantics are not legal (or differ) in JSweet.

### Core types and objects

JSweet allows the use of primitive Java types, core Java objects (with some restrictions) and of core JavaScript objects, which are defined in the `jsweet.lang` package. Next, we describe the use of such core types and objects.

#### Primitive Java types

JSweet allows the use of Java primitive types (and associated literals).

-   `int`, `byte`, `short`, `double`, `float` are all converted to JavaScript numbers (precision does not matter in JSweet).

-   `char` follows the Java typing rules but is converted to a JavaScript string by the transpiler.

-   `boolean` corresponds to the JavaScript boolean.

-   `java.lang.String` corresponds to the JavaScript `string`. (not per say a primitive type, but is immutable and used as the class of string literals in Java)

Examples of valid statements:

``` java
// warning '==' behaves like the JavaScript one at runtime
int i = 2;
assert i == 2;
double d = i + 4;
assert d == 6;
String s = "string" + '0' + i;
assert s == "string02";
boolean b = false;
assert !b;
```

#### Allowed Java objects

Here follows the list of allowed Java classes in JSweet:

-   `java.lang.Object`

    -   allowed methods: `toString()`

-   `java.lang.String`

    -   allowed methods:

        -   `charAt(int)`

        -   `concat(java.lang.String)`

        -   `indexOf(java.lang.String)`

        -   `lastIndexOf(java.lang.String)`

        -   `lastIndexOf(java.lang.String,int)`

        -   `substring(int)`

        -   `substring(int,int)` (with the JavaScript behavior)

        -   `replace(java.lang.String,java.lang.String)`

        -   `split(java.lang.String)`

        -   `trim()`

        -   `toLowerCase()`

        -   `toUpperCase()`

-   `java.lang.Class`

    -   allowed methods: none

-   `java.lang.Boolean`

    -   allowed methods: none

-   `java.lang.Void`

    -   allowed methods: none

-   `java.lang.Integer`

    -   allowed methods: none

-   `java.lang.Double`

    -   allowed methods: none

-   `java.lang.Number`

    -   allowed methods: none

-   `java.lang.Float`

    -   allowed methods: none

-   `java.lang.Byte`

    -   allowed methods: none

-   `java.lang.Short`

    -   allowed methods: none

-   `java.lang.Iterable`

    -   allowed methods: none (for using the *foreach* loop on indexed objects)

-   `java.lang.Runnable`

    -   allowed methods: none (for declaring lambdas)

-   `java.util.function.*` (for declaring lambdas)

    -   prohibited method names:

        -   `and`

        -   `negate`

        -   `or`

        -   `andThen`

Examples of valid statements:

``` java
// warning '==' behaves like the JavaScript one at runtime
Integer i = 2;
assert i == 2;
Double d = i + 4;
assert d.toString() == "6";
assert d == "6";
BiFunction<String, Integer, String> f = (s, i) -> { return s.substring(i); };
assert "bc" == f.apply("abc", 1);
```

#### Java arrays

Arrays can be used in JSweet and are transpiled to JavaScript arrays. Array initialization, accesses and and iteration are all valid statements.

``` java
int[] arrayOfInts = { 1, 2, 3, 4};
assert arrayOfInts.length == 4;  
assert arrayOfInts[0] == 1;
for(int i : arrayOfInts) {
    arrayOfInts[i] = arrayOfInts[i] - 1;
    assert arrayOfInts[i] == i;
}
```

#### Core JavaScript API

The core JavaScript API is defined in `jsweet.lang` (the full documentation can be found at <http://www.jsweet.org/core-api-javadoc/>). Main JavaScript classes are:

-   `jsweet.lang.Object`: JavaScript Object class. Common ancestor for JavaScript objects functions and properties.

-   `jsweet.lang.Boolean`: JavaScript Boolean class. A wrapper for boolean values.

-   `jsweet.lang.Number`: JavaScript Number class. A wrapper for numerical values.

-   `jsweet.lang.String`: JavaScript String class. A wrapper and constructor for strings.

-   `jsweet.lang.Function`: JavaScript Function class. A constructor for functions.

-   `jsweet.lang.Date`: JavaScript Date class, which enables basic storage and retrieval of dates and times.

-   `jsweet.lang.Array<T>`: JavaScript Array class. It is used in the construction of arrays, which are high-level, list-like objects.

-   `jsweet.lang.Error`: JavaScript Error class. This class implements `java.lang.RuntimeException` and can be thrown and caught with `try` ... `catch` statements.

Programmers should use this API most of the time, which is HTML5 compatible and follows the JavaScript latest supported versions. However, for objects that need to be used with Java literals (numbers, booleans, and strings), the use of the `java.lang` package classes is recommended. For instance, the jQuery API declares `$(java.lang.String)` instead of `$(jsweet.lang.String)`. This allows the programmer to write expressions using literals, such as `$("a")` (for selecting all links in a document).

As a consequence, programmers need to be able to switch to the JavaScript API when coming from a Java object. The `jsweet.util.Globals` class defines convenient static methods to cast back and forth core Java objects to their corresponding JSweet objects. For instance the `string(...)` method will allow the programmer to switch from the Java to the JSweet strings and conversely.

``` java
import jsweet.util.Globals.string;
String str = "This is a test string";
str.toLowerCase(); // valid: toLowerCase it defined both in Java and JavaScript
str.substr(1); // compile error: substr is not a Java method
string(str).substr(1); // valid: string(str) is a jsweet.lang.String.
```

Here is another example that shows the use of the `array` method to access the `push` method available on JavaScript arrays.

``` java
import jsweet.util.Globals.array;
String[] strings = { "a", "b", "c" };
array(strings).push("d");
assert strings[3] == "d";
```

### Classes

Classes in JSweet are very similar to Java classes. For example:

``` java
public class BankAccount {  
    public double balance = 0;  
    public double deposit(double credit) {  
        balance += credit;  
        return this.balance;  
    }  
}  
```

Which is transpiled to the following JavaScript code:

``` java
var BankAccount = (function () {  
    function BankAccount() {  
        this.balance = 0;  
    }  
    BankAccount.prototype.deposit = function(credit) {  
        this.balance += credit;  
        return this.balance;  
    };  
    return BankAccount;  
})();
```

Classes can define constructors, have super classes and be instantiated exactly like in Java. The only restriction compared to Java is that inner classes or anonymous classes are not allowed in JSweet. For instance, the following code will raise an error.

``` java
public class ContainerClass {
    // error: inner classes are not allowed
    public class InnerClass {
    }
}
```

### Interfaces

In JSweet, an interface (a.k.a. object type) can be seen as object signature, that is to say the accessible functions and properties of an object (without specifying any implementation). An interface is defined for typing only and has no runtime representation (no instances), however, they can be used to type objects.

JSweet interfaces can be defined as regular Java interfaces, but also as Java classes annotated with `@jsweet.lang.Interface`, so that is is possible to define properties as fields. Such classes impose many constraints, as shown in the following code.

``` java
@Interface
public class WrongConstructsInInterfaces {
    native public void m1(); // OK
    // error: field initializers are not allowed
    public long l = 4;
    // error: statics are not allowed
    static String s1;
    // error: private are not allowed
    private String s2;
    // error: constructors are not allowed
    public WrongConstructsInInterfaces() {
        l = 4;
    }
    // error: bodies are not allowed
    public void m2() {
        l = 4;
    }
    // error: statics are not allowed
    native static void m3();
    // error: initializers are not allowed
    {
        l = 4;
    }
    // error: static initializers are not allowed
    static {
        s1 = "";
    }
}
```

#### Object typing

In JSweet, typed object can be constructed out of interfaces. If we take the following interface:

``` java
@Interface
public class Point {
    public double x;
    public double y;
}
```

We can create an object typed after the interface. Note that the following code is not actually creating an instance of the `Point` interface, it is creating an object that conforms to the interface.

``` java
Point p1 = new Point() {{ x=1; y=1; }};
```

As a direct consequence, in JSweet it is not allowed to check an instance against an interface type.

``` java
if (p1 instanceof Point) { ... } // compile error 
```

This may seems quite confusing for Java programmers, but you have to remember that, on contrary to Java where interfaces are available as special classes at runtime, in JSweet, interfaces have no reality at runtime. Think of generics, which are of the same kind in Java. As a consequence, the `instanceof` operator is not applicable on interfaces at runtime (like it is not applicable on generics).

#### Optional fields

Interfaces can define *optional fields*, which are used to report errors when the programmer forgets to initialize a mandatory field in an object. Supporting optional fields in JSweet is done through the use of `@jsweet.lang.Optional` annotations. For instance:

``` java
@Interface
public class Point {
    public double x;
    public double y;
    @Optional
    public double z = 0;
}
```

It is the JSweet compiler that will check that the fields are correctly initialized, when constructing an object.

``` java
// no errors (z is optional)
Point p1 = new Point() {{ x=1; y=1; }};
// JSweet reports a compile error since y is not optional
Point p2 = new Point() {{ x=1; z=1; }};
```

#### Special functions in interfaces

In JavaScript, objects can have properties and functions, but can also (not exclusively), be used as constructors and functions themselves. This is not possible in Java, so JSweet defines special functions for handling these cases.

-   `$apply` is used to state that the object can be used as a function.

-   `$new` is used to state that the object can be used as a constructor.

### Enums

JSweet allows the definition of enums similarly to Java, but with some restrictions. The following code declares an enum with tree possible values (`A`, `B`, and `C`).

``` java
enum MyEnum {
    A, B, C
}
```

The following statements are valid statements in JSweet.

``` java
MyEnum e = MyEnum.A;
assert MyEnum.A == e;
assert e.name().equals("A");
assert e.ordinal() == 0;
assert MyEnum.valueOf("A") == e;
assert array(MyEnum.values()).indexOf(MyEnum.valueOf("C")) == 2;
```

On the other hand, unlike Java enums, other members than constants are not allowed, raising JSweet transpilation errors, as shown in the following code.

``` java
public enum WrongConstructsInEnums {

    A, B, C;
    
    // error: fields are not allowed
    public long l = 4;
    // error: constructors are not allowed  
    private WrongConstructsInEnums() {
        l = 4;
    }
    // error: methods are not allowed
    public void m2() {
        l = 4;
    }
    // error: initializers are not allowed
    {
        l = 4;
    }
}
```

### Indexed objects

In JavaScript, object can be seen as maps containing key-value pairs (key is often called *index*, especially when it is a number). So, in JSweet, all objects define the special functions (defined on `jsweet.lang.Object`):

-   `$get(key)` accesses a value with the given key.

-   `$set(key,value)` sets or replace a value for the given key.

-   `$delete(key)` deletes the value for the given key.

The type of keys and values can be overloaded for every object. For example, the `Array<T>` class, will define keys as numbers and values as objects conforming to type `T`.

In the case of objects indexed with number keys, it is allowed to implement the `java.lang.Iterable` interface so that it is possible to use they in *foreach* loops. For instance, the `NodeList` type (from the DOM) defines an indexed function:

``` java
@Interface
class NodeList implements java.lang.Iterable {
    public double length;
    public Node item(double index);
    public Node $get(double index);
}
```

In JSweet, you can access the node list elements with the `$get` function, and your can also iterate with the *foreach* syntax. The following code generates fully valid JavaScript code.

``` java
NodeList nodes = ...
for (int i = 0; i < nodes.length; i++) {
    HTMLElement element = (HTMLElement) nodes.$get(i);
    [...]
}
// same as:
NodeList nodes = ...
for (Node node : nodes) {
    HTMLElement element = (HTMLElement) node;
    [...]
}
```

### Globals

In Java, on contrary to JavaScript, there is no such thing as global variables or functions (there are only static members, but even those must belong to a class). Thus, JSweet introduces reserved `Globals` classes and `globals` packages. These have two purposes:

-   Generate code that has global variables and functions (this is discouraged in Java)

-   Bind to existing JavaScript code that defines global variables and functions (as many JavaScript frameworks do)

In Globals classes, only static fields (global variables) and static methods (global functions) are allowed. Here are the main constraints applying to Globals classes:

-   no non-static members

-   no super class

-   cannot be extended

-   cannot be used as types like regular classes

-   no public constructor (empty private constructor is OK)

-   cannot use $get, $set and $delete within the methods

For instance, the following code snippets will raise transpilation errors.

``` java
class Globals {
    public int a;
    // error: public constructors are not allowed
    public Globals() {
        this.a = 3;
    }
    public static void test() {
        // error: no instance is available
        $delete("key");
    }
}
```

``` java
// error: Globals classes cannot be used as types
Globals myVariable = null;
```

One must remember that `Globals` classes and `global` packages are erased at runtime so that their members will be directly accessible. For instance `mypackage.Globals.m()` in a JSweet program corresponds to the `mypackage.m()` function in the generated code and in the JavaScript VM at runtime. Also, `mypackage.globals.Globals.m()` corresponds to *m()*.

In order to erase packages in the generated code, programmers can also use the `@Root` annotation, which will be explained in Section \[packaging\].

### Optional parameters

In JavaScript, parameters can be optional, in the sense that a parameter value does not need to be provided when calling a function. Except for varargs, which are fully supported in JSweet, the general concept of an optional parameter does not exist in Java. To simulate optional parameters, JSweet supports overloading as long as it is for implementing optional parameters – a very commonly used idiom. Here are some examples of valid and invalid overloading in JSweet:

``` java
String m(String s, double n) { return s + n; }
// valid overloading (JSweet transpiles to optional parameter)
String m(String s) { return m(s, 0); }
// invalid overloading (JSweet error)
String m(String s) { return s; }
```

### Ambient declarations

It can be the case that programmers need to use existing libraries from JSweet. In most cases, one should look up in the available candies (<http://www.jsweet.org/candies-releases/>), which are automatically generated from TypeScript’s DefinitelyTyped. When the candy does not exist, or does not entirely cover what is needed, one can use the `@jsweet.lang.Ambient` annotation, which will make available to the programmers a class definition or an interface. The following example shows the backbone store class made accessible to the JSweet programmer with a simple ambient declaration. This class will be erased during the JavaScript generation.

``` java
@Ambient
class Store {
    public Store(String dbName) {}
}
```

Auxiliary types
---------------

JSweet uses most Java typing features (including functional types) but also extends the Java type system with so-called *auxiliary types*. The idea behind auxiliary types is to create classes or interfaces that can hold the typing information through the use of type parameters (a.k.a *generics*), so that the JSweet transpiler can cover more typing scenarios. These types have been mapped from TypeScript type system, which is much richer than the Java one (mostly because JavaScript is a dynamic language and requests more typing scenarios than Java).

### Functional types

For functional types, JSweet reuses the `java.Runnable` and `java.util.function` functional interfaces of Java 8. These interfaces only support up to 2-parameter functions. Thus, JSweet adds some support for more parameters in `jsweet.util.function`, since it is a common case in JavaScript APIs.

// To be completed

### Object types

// Todo

### String types

For string types, the use of specific types and final instances of these types is the way to simulate string types in Java. For instance, if a `"span"` string type needs to be defined, a Java interface called `span` and a static final field called `span` in a `StringTypes` class will do the job.

// To be completed

### Tuple types

Tuple types represent JavaScript arrays with individually tracked element types. For tuple types, JSweet defines parameterized auxiliary classes `TupleN<T0, ... TN-1>`, which define `$0`, `$1`, ... `$N-1` public fields to simulate typed array accessed (field `$i` is typed with `Ti`).

// To be completed

### Union types

Union types represent values that may have one of several distinct representations. For union types, JSweet takes advantage of the `method overloading` mechanism available in Java. For more general cases, it defines an auxiliary interface `Union<T1, T2>` (and `UnionN<T1, ... TN>`) in the `jsweet.util.union` package. By using this auxiliary type and a `union` utility method, programmers can cast back and forth between union types and union-ed type, so that JSweet can ensure similar properties as TypeScript union types.

The following code shows a typical use of union types in JSweet. It simply declares a variable as a union between a string and a number, which means that the variable can actually be of one of that types (but of no other types). The switch from a union type to a regular type is done through the `jsweet.util.Globals.union` helper method. This helper method is completely untyped, allowing from a Java perspective any union to be transformed to another type. It is actually the JSweet transpiler that checks that the union type is consistently used.

``` java
import static jsweet.util.Globals.union;
import jsweet.util.union.Union;
[...]
Union<String, Number> u = ...;
// u can be used as a String
String s = union(u);
// or a number
Number n = union(u);
// but nothing else
Date d = union(u); // JSweet error
```

The `union` helper can also be used the other way, to switch from a regular type back to a union type, when expected.

``` java
import static jsweet.util.Globals.union;
import jsweet.util.union.Union3;
[...]
public void m(Union3<String, Number, Date>> u) { ... }
[...]
// u can be a String, a Number or a Date
m(union("a string"));
// but nothing else
m(union(new RegExp(".*"))); // compile error
```

Note: the use of Java function overloading is preferred over union types when typing function parameters. For example:

``` java
// with union types (discouraged)
native public void m(Union3<String, Number, Date>> u);
// with overloading (preferred way)
native public void m(String s);
native public void m(Number n);
native public void m(Date d);
```

Semantics
---------

Semantics designate how a given program behaves when executed. Although JSweet relies on the Java syntax, programs are transpiled to JavaScript and do not run in a JRE. As a consequence, the JavaScript semantics will impact the final semantics of a JSweet program compared to a Java program. In this section, we discuss the semantics by focusing on differences or commonalities between Java/JavaSript and JSweet.

### Main methods

Main methods are the program execution entry points and will be invoked globally when a class containing a `main` method is evaluated. For instance:

``` java
public class C {
    private int n;
    public static C instance;
    public static main(String[] args) {
        instance = new C();
        instance.n = 4;
    }
    public int getN() {
        return n;
    }
}
// when the source file containing C has been evaluated:
assert C.instance != null;
assert C.instance.getN() == 4;
```

Main methods do not behave exactly like in Java and depend on the way the program is packaged and deployed.

-   **Regular packaging (no modules)**. With regular packaging, one Java source file corresponds to one generated JavaScript file. In that case, when loading a file in the browser, all the main methods will be invoked, right after the file is evaluated.

-   **Module packaging**. With module packaging (module option), one Java package corresponds to one module. With modules, it is mandatory to have only one main method in the program, which will be the global entry point from which the module dependency graph will be calculated. The main module will use directly or transitively all the other modules.

Because of modules, it is good practice to have only one main method in an application.

### Initializers

Initializers behave like in Java.

For example:

``` java
public class C1 {
    int n;
    {
        n = 4;
    }
}
assert new C1().n == 4;
```

And similarly with static initializers:

``` java
public class C2 {
    static int n;
    static {
        n = 4;
    }
}
assert new C2.n == 4;
```

### Name clashes

On contrary to Java, methods and fields of the same name are not allowed within the same class or within classes having a subclassing link. The reason behind this behavior is that in JavaScript, object variables and functions are stored within the same object map, which basically means that you cannot have the same key for several object members (this also explains that method overloading in the Java sense is not possible in JavaScript).

In order to avoid programming mistakes due to this JavaScript behavior, which is not necessarily known by Java programmers, JSweet adds a semantics check to avoid duplicate names in classes (this also takes into account members defined in parent classes). As an example:

``` java
public class NameClashes {

    // error: field name clashes with existing method name
    public String a;

    // error: method name clashes with existing field name
    public void a() {
        return a;
    }

}
```

In general, it also not possible to have two methods with the same name but with different parameters (so-called overloads). An exception to this behavior is the use of method overloading for defining optional parameters. JSweet allows this idiom under the condition that it corresponds to the following template:

``` java
String m(String s, double n) { return s + n; }
// valid overloading (JSweet transpiles to optional parameter)
String m(String s) { return m(s, 0); }
```

In that case, JSweet will generate JavaScript code with only one method having default values for the optional parameters, so that the behavior of the generated program corresponds to the original one. In this case:

``` java
function m(s, n = 0) { return s + n; }
```

If the programmer tries to use overloading differently, for example by defining two different implementations for the same method name, JSweet will raise a compile error.

``` java
String m(String s, double n) { return s + n; }
// invalid overloading (JSweet error)
String m(String s) { return s; }
```

### Variable scoping in lambda expressions

JavaScript variable scope is known to pose some problems to the programmers, because it is possible to change the reference to a variable from outside of a lambda that would use this variable. As a consequence, a JavaScript programmer cannot rely on a variable declared outside of a lambda scope, because when the lambda is executed, the variable may have been modified somewhere else in the program. For instance, the following program shows a typical case:

``` java
NodeList nodes = document.querySelectorAll(".control");
for (int i = 0; i < nodes.length; i++) {
    HTMLElement element = (HTMLElement) nodes.$get(i); // final
    element.addEventListener("keyup", (evt) -> {
        // this element variable will not change here 
        element.classList.add("hit");
    });
}
```

In JavaScript (note that EcmaScript 6 fixes this issue), such a program would fail its purpose because the `element` variable used in the event listener is modified by the for loop and does not hold the expected value. In JSweet, such problems are dealt with similarly to final Java variables. In our example, the `element` variable is re-scoped in the lambda expression so that the enclosing loop does not change its value and so that the program behaves like in Java (as expected by most programmers).

### Scope of *this*

On contrary to JavaScript and similarly to Java, using a method as a lambda will prevent loosing the reference to `this`. For instance, in the `action` method of the following program, `this` holds the right value, even when `action` was called as a lambda in the `main` method. Although this seem logical to Java programmers, it is not a given that the JavaScript semantics ensures this behavior.

``` java
package example;
import static jsweet.dom.Globals.console;

public class Example {
    private int i = 8;
    public Runnable getAction() {
        return this::action;
    }
    public void action() {
        console.log(this.i); // this.i is 8
    }
    public static void main(String[] args) {
        Example instance = new Example();
        instance.getAction().run();
    }
}
```

Packaging
---------

Packaging is one of the complex point of JavaScript, especially when coming from Java. Complexity with JavaScript packaging boils down to the fact that JavaScript did not define any packaging natively. As a consequence, many *de facto* solutions and quidelines came up along the years, making the understanding of packaging uneasy for regular Java programmers. In spite EcmaScript UMS is supposed to bring a standard for modules in Java, other *de facto* standards remain and the notion of a namespace/package is still not handled by JavaScript, making the notion of a module and a namespace confusing for JavaScript and TypeScript developers themselves.

### Packages and modules

Packages and modules are two similar concepts but for different contexts.

Java packages must be understood as compile-time *namespaces*. They allow a compile-time structuration of the programs through name paths, with implicit or explicit visibility rules. Packages have usually not much impact on how the program is actually bundled and deployed.

Modules must be understood as deployment and runtime *bundles*. The closest concept to a module in the Java world would probably be an OSGi bundle. A module defines imported and exported elements so that they create a strong runtime structure that can be used for deploying software components independently and thus avoiding name clashes. For instance, with modules, two different libraries may define a `util.List` class and be actually running and used on the same VM with no naming issues (as long as the libraries are bundled in different modules).

JSweet uses the Java package concept for namespaces. Modules are a deployment concept and should be created with some deployment scripts involving third party tools. However, for easy deployment, JSweet defines a `module` option that automatically creates a default module organization following the simple rule: one package = one module. With this rule, it is actually possible to run a JSweet program on Node.js without requiring extra work from the programmer. Also, using both the `bundle` and `module` options of JSweet creates a bundle file containing the application modules, and which can be deployed on a browser with no extra packaging.

### Root packages

Root packages are a way to tune the generated code so that JSweet packages are erased in the generated code and thus at runtime. To set a root package, just define a package-info.java file and use the @Root annotation on the package, as follows:

``` java
@Root
package a.b.c;
```

The above declaration means that the `c` package is a root package, i.e. it will be erased in the generated code, as well as all its parent packages. Thus, if `c` contains a package `d`, and a class `C`, these will be top-level objects at runtime. In other words, `a.b.c.d` becomes `d`, and `a.b.c.C` becomes `C`.

Note that root packages do not change the folder hierarchy of the generated files. For instance, the `a.b.c.C` class will still be generated in the `<jsout>/a/b/c/C.js` file (relatively to the `<jsout>` output directory). However, switching on the `noRootDirectories` option will remove the root directories so that the `a.b.c.C` class gets generated to the `<jsout>/C.js` file.

### External modules

When compiling JSweet programs with the `module` options, all external libraries and components must be required as external modules. JSweet can automatically require modules, simply by using the `@Module(name)` annotation. In JSweet, importing or using a class or a member annotated with `@Module(name)` will automatically require the corresponding module at runtime. Please not that it is true only when the code is generated with the `module` option. If the `module` option is off, the `@Module` annotations are ignored.

``` java
package def.jquery;
public final class Globals extends jsweet.lang.Object {
    ...
    @jsweet.lang.Module("jquery")
    native public static def.jquery.JQuery $(java.lang.String selector);
    ...
}
```

The above code shows an excerpt of the JSweet jQuery API. As we can notice, the $ function is annotated with `@Module("jquery")`. As a consequence, any call to this function will trigger the require of the `jquery` module.

Note: the notion of manual require of a module may be available in future releases. However, automatic require is sufficient for most programmers and hides the complexity of having to require modules explicitly. It also brings the advantage of having the same code whether modules are used or not.

Appendix 1: JSweet transpiler options
-------------------------------------

      [-h|--help]

      [-v|--verbose]
            Turn on all levels of logging.

      [--encoding <encoding>]
            Force the Java compiler to use a specific encoding (UTF-8, UTF-16, ...).
            (default: UTF-8)

      [--jdkHome <jdkHome>]
            Set the JDK home directory to be used to find the Java compiler. If not
            set, the transpiler will try to use the JAVA_HOME environment variable.
            Note that the expected JDK version is greater or equals to version 8.

      (-i|--input) <input>
            An input dir containing Java files to be transpiled.

      [--noRootDirectories]
            Skip the root directories (i.e. packages annotated with
            @jsweet.lang.Root) so that the generated file hierarchy starts at the
            root directories rather than including the entire directory structure.

      [--tsout <tsout>]
            Specify where to place generated TypeScript files. (default: .ts)

      [(-o|--jsout) <jsout>]
            Specify where to place generated JavaScript files (ignored if jsFile is
            specified). (default: js)

      [--classpath <classpath>]
            The JSweet transpilation classpath (candy jars). This classpath should
            at least contain the core candy.

      [(-m|--module) <module>]
            The module kind (none, commonjs, amd, system or umd). (default: none)

      [-b|--bundle]
            Bundle up the generated files and used modules to bundle files, which
            can be used in the browser. Bundles contain all the dependencies and are
            thus standalone. There is one bundle generated per entry (a Java 'main'
            method) in the program. By default, bundles are generated in the entry
            directory, but the output directory can be set by using the
            --bundlesDirectory option. NOTE: bundles will be generated only when
            choosing the commonjs module kind.

      [--bundlesDirectory <bundlesDirectory>]
            Generate all the bundles (see option --bundle) within the given
            directory.

      [-d|--debug]
            Set the transpiler to debug mode. In debug mode, source map files are
            generated so that it is possible to debug them in the browser. This
            feature is not available yet when using the --module option.

      [--ignoreAssertions]
            Set the transpiler to ignore 'assert' statements, i.e. no code is
            generated for assertions.
