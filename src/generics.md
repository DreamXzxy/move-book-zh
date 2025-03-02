# 泛型 (generics)

Generics can be used to define functions and structs over different input data types. This language feature is sometimes referred to as *parametric polymorphism*. In Move, we will often use the term generics interchangeably with type parameters and type arguments.

泛型可用于定义具有不同输入数据类型的函数和结构体。这种语言特性被称为参数多态。在 Move语言中，我们经常将交替使用术语泛型与类型参数和类型参数。

Generics are commonly used in library code, such as in vector, to declare code that works over any possible instantiation (that satisfies the specified constraints). In other frameworks, generic code can sometimes be used to interact with global storage many different ways that all still share the same implementation.

泛型通常用于库代码中，例如 `vector`，以声明适用于任何可实例化(满足指定约束)的代码。在其他框架中，泛型代码有时可用多种不同的方式与全局存储交互，这些方式有着相同的实现。


## 声明类型参数 (Declaring Type Parameters)

Both functions and structs can take a list of type parameters in their signatures, enclosed by a pair of angle brackets `<...>` .

函数和结构体都可以在其签名中采用类型参数列表，由一对尖括号括起来 `<...>` 。

### 泛型函数 (Generic Functions)


Type parameters for functions are placed after the function name and before the (value) parameter list. The following code defines a generic identity function that takes a value of any type and returns that value unchanged.

函数的类型参数放在函数名称之后和(值)参数列表之前。以下代码定义了一个名为id的泛型函数，该函数接受任何类型的值并返回原值。

```move
fun id<T>(x: T): T {
    // this type annotation is unnecessary but valid
    (x: T)
}
```

Once defined, the type parameter `T` can be used in parameter types, return types, and inside the function body.

一旦定义，类型参数 `T` 就可以在参数类型、返回类型和函数体内使用。

### 泛型结构体 (Generic Structs)

Type parameters for structs are placed after the struct name, and can be used to name the types of the fields.

结构体的类型参数放在结构名称之后，并可用于命名字段的类型。

```move
struct Foo<T> has copy, drop { x: T }

struct Bar<T1, T2> has copy, drop {
    x: T1,
    y: vector<T2>,
}
```

[Note that type parameters do not have to be used](#unused-type-parameters)

请注意，[未使用的类型参数](#unused-type-parameters)

## 类型参数 (Type Arguments)

### 调用泛型函数 (Calling Generic Functions)

When calling a generic function, one can specify the type arguments for the function's type parameters in a list enclosed by a pair of angle brackets.

调用泛型函数时，可以在由一对尖括号括起来的列表中指定函数类型参数。

```move
fun foo() {
    let x = id<bool>(true);
}
```

If you do not specify the type arguments, Move's [type inference](#type-inference) will supply them for you.

如果您不指定类型参数，Move语言的[类型推断](#type-inference)功能将为您匹配正确的类型

### 使用泛型结构 (Using Generic Structs)

Similarly, one can attach a list of type arguments for the struct's type parameters when constructing or destructing values of generic types.

类似地，在构造或销毁泛型类型的值时，可以为结构体的类型参数附加一个参数列表。

```move
fun foo() {
    let foo = Foo<bool> { x: true };
    let Foo<bool> { x } = foo;
}
```

If you do not specify the type arguments, Move's [type inference](#type-inference) will supply them for you.

如果您不指定类型参数，Move 语言的[类型推断](#type-inference)功能将为您自动补充(supply)。

### 类型参数不匹配 (Type Argument Mismatch)

If you specify the type arguments and they conflict with the actual values supplied, an error will be given

如果您指定类型参数与实际提供的值不匹配，则会报错

```move
fun foo() {
    let x = id<u64>(true); // error! true is not a u64
}
```

同样地

```move
fun foo() {
    let foo = Foo<bool> { x: 0 }; // error! 0 is not a bool
    let Foo<address> { x } = foo; // error! bool is incompatible with address
}
```

## Type Inference (类型推断)

In most cases, the Move compiler will be able to infer the type arguments so you don't have to write them down explicitly. Here's what the examples above would look like if we omit the type arguments.

在大多数情况下，Move 编译器能够推断类型参数，因此您不必显式地写下它们。这是上面例子中省略类型参数写法的示例。

```move
fun foo() {
    let x = id(true);
    //        ^ <bool> is inferred

    let foo = Foo { x: true };
    //           ^ <bool> is inferred

    let Foo { x } = foo;
    //     ^ <bool> is inferred
}
```

Note: when the compiler is unable to infer the types, you'll need annotate them manually. A common scenario is to call a function with type parameters appearing only at return positions.

注意：当编译器无法推断类型时，您需要手动标注它们(类型参数)。一个常见的场景是调用一个类型参数只出现在返回位置的函数。

```move
address 0x2 {
    module m {
        using std::vector;

        fun foo() {
            // let v = vector::new();
            //                    ^ The compiler cannot figure out the element type.

            let v = vector::new<u64>();
            //                 ^~~~~ Must annotate manually.
        }
    }
}
```

However, the compiler will be able to infer the type if that return value is used later in that function

但是，如果稍后在该函数中使用该返回值，编译器将能够推断其类型。

```move
address 0x2 {
    module m {
        using std::vector;

        fun foo() {
            let v = vector::new();
            //                 ^ <u64> is inferred
            vector::push_back(&mut v, 42);
        }
    }
}
```

## Unused Type Parameters (未使用的类型参数)

For a struct definition, an unused type parameter is one that
does not appear in any field defined in the struct, but is checked statically at compile time.
Move allows unused type parameters so the following struct definition is valid:

对于结构体定义，未使用的类型参数是没有出现在结构体中定义的任何字段中，但在编译时会静态检查的类型参数。Move语言允许未使用的类型参数，因此以下结构定义是有效的：

```move
struct Foo<T> {
    foo: u64
}
```

This can be convenient when modeling certain concepts. Here is an example:

这在对某些概念进行建模时会很方便。这是一个例子：

```move
address 0x2 {
    module m {
        // Currency Specifiers
        struct Currency1 {}
        struct Currency2 {}

        // A generic coin type that can be instantiated using a currency
        // specifier type.
        //   e.g. Coin<Currency1>, Coin<Currency2> etc.
        struct Coin<Currency> has store {
            value: u64
        }

        // Write code generically about all currencies
        public fun mint_generic<Currency>(value: u64): Coin<Currency> {
            Coin { value }
        }

        // Write code concretely about one currency
        public fun mint_concrete(value: u64): Coin<Currency1> {
            Coin { value }
        }
    }
}
```

In this example, `struct Coin<Currency>` is generic on the `Currency` type parameter,
which specifies the currency of the coin and allows code to be written either generically on any currency or
concretely on a specific currency.
This genericity applies even when the `Currency` type parameter does not appear in any of the fields defined in `Coin`.

在此示例中， `struct Coin<Currency>` 是类型参数为 `Currency` 的泛型结构体，它指定 `Coin` 的类型参数是 `Currency`，这样就允许代码选择是使用任意类型 `Currency` 或者是指定的具体类型 `Currency` 。即使 `Currency` 类型参数没有出现在定义的任何字段中，这种泛型性也适用结构体 `Coin`。

### Phantom Type Parameters 

In the example above, although `struct Coin` asks for the `store` ability, neither `Coin<Currency1>` nor `Coin<Currency2>` will have the `store` ability.
This is because of the rules for [Conditional Abilities and Generic Types](./abilities.md#conditional-abilities-and-generic-types) and the fact that `Currency1` and `Currency2` don't have the `store` ability, despite the fact that they are not even used in the body of `struct Coin`. 
This might cause some unpleasant consequences.
For example, we are unable to put `Coin<Currency1>` into a wallet in the global storage.

在上面的例子中，虽然 `struct Coin` 要求有 `store` 能力，但 `Coin<Currency1>` 和 `Coin<Currency2>` 都没有 `store` 能力。这是因为 [条件能力与泛型类型](./chatper_19_abilities.md#conditional-abilities-and-generic-types)的规则, 而实际上 `Currency1`和 `Currency2` 本身都没有 `store` 能力，尽管它们甚至没有在`struct Coin` 的主体中使用. 这可能会导致一些不好的后果。例如，我们无法将 `Coin<Currency1>` 放入全局存储的一个钱包中。

One possible solution would be to add spurious ability annotations to `Currency1` and `Currency2` (i.e., `struct Currency1 has store {}`).
But, this might lead to bugs or security vulnerabilities because it weakens the types with unnecessary ability declarations.
For example, we would never expect a resource in the global storage to have a field in type `Currency1`, but this would be possible with the spurious `store` ability.
Moreover, the spurious annotations would be infectious, requiring many functions generic on the unused type parameter to also include the necessary constraints.

一种可能的解决方案是向 `Currency1` 和 `Currency2` 添加虚假能力标注(例如：`struct Currency1 has store {}`) 。但是，这可能会导致 bugs 或安全漏洞，因为它削弱了类型安全，声明了不必要的能力。例如，我们永远不会期望全局存储中的资源具有 `Currency1` 类型的字段，但这对于虚假 `store` 能力是可能发生的。
此外，虚假标注具有传染性，需要在许多未使用类型参数的泛型函数上也引入必要的约束。

Phantom type parameters solve this problem. Unused type parameters can be marked as *phantom* type parameters,
which do not participate in the ability derivation for structs.
In this way, arguments to phantom type parameters are not considered when deriving the abilities for generic types, thus avoiding the need for spurious ability annotations.
For this relaxed rule to be sound, Move's type system guarantees that a parameter declared as phantom is either not used at all in the struct definition, or it is only used as an argument to type parameters also declared as phantom.

Phantom 类型参数解决了这个问题。未使用的类型参数可以标记为 *phantom* 类型参数，不参与结构体的能力推导。这样，在派生泛型类型的能力时，不考虑 phantom type 的参数，从而避免了对虚假能力标注的需要。为了使这个宽松的规则合理，Move 的类型系统保证声明为 `phantom` 的参数要么不在结构定义中使用，要么仅用作声明为 `phantom` 的类型参数的参数。

#### 声明 (Declaration)


In a struct definition a type parameter can be declared as phantom by adding the `phantom` keyword before its declaration.
If a type parameter is declared as phantom we say it is a phantom type parameter.
When defining a struct, Move's type checker ensures that every phantom type parameter is either not used inside the struct definition 
or it is only used as an argument to a phantom type parameter.

`phantom` 在结构定义中，可以通过在声明之前添加关键字来将类型参数声明为幻影。如果一个类型参数被声明为 phantom，我们就说它是 phantom 类型参数。
定义结构体时，Move语言的类型检查器确保每个 phantom 类型参数要么不在结构定义中使用，要么仅用作 phantom 类型参数的参数。

More formally, if a type is used as an argument to a phantom type parameter we say the type appears in _phantom position_.
With this definition in place, the rule for the correct use of phantom parameters can be specified as follows: **A phantom type parameter can only appear in phantom position**.

更正式地说，如果将类型用作 phantom 类型参数的输入参数，我们说该类型出现在 _phantom 位置_。有了这个定义，正确使用  phantom 参数的规则可以指定如下： ** phantom 类型参数只能出现在 phantom 位置**。

The following two examples show valid uses of phantom parameters.
In the first one, the parameter `T1` is not used at all inside the struct definition.
In the second one, the parameter `T1` is only used as an argument to a phantom type parameter.

以下两个示例显示了 phantom 参数的 合法用法。在第一个中，`T1` 在结构体定义中根本不使用参数。在第二种情况下，参数 `T1` 仅用作 phantom 类型参数的参数。

```move
struct S1<phantom T1, T2> { f: u64 }
                  ^^
                  Ok: T1 does not appear inside the struct definition


struct S2<phantom T1, T2> { f: S1<T1, T2> }
                                  ^^
                                  Ok: T1 appears in phantom position
```      

The following code shows examples of violations of the rule:

以下代码展示违反规则的示例：

```move
struct S1<phantom T> { f: T }
                          ^
                          Error: Not a phantom position

struct S2<T> { f: T }

struct S3<phantom T> { f: S2<T> }
                             ^
                             Error: Not a phantom position
```                               

#### 实例化 (Instantiation)

When instantiating a struct, the arguments to phantom parameters are excluded when deriving the struct abilities.
For example, consider the following code:

实例化结构时，派生结构功能时会排除幻影参数的输入参数。例如，考虑以下代码：

```move
struct S<T1, phantom T2> has copy { f: T1 }
struct NoCopy {}
struct HasCopy has copy {}
```
Consider now the type `S<HasCopy, NoCopy>`. Since `S` is defined with `copy` and all non-phantom arguments have copy then `S<HasCopy, NoCopy>` also has copy.

现在考虑类型 `S<HasCopy, NoCopy>`。由于 `S` 用 `copy` 定义，且所有非 phantom 参数  具有 `copy` 能力，所以 `S<HasCopy, NoCopy>` 也具有 `copy` 能力。

#### 具有能力约束的 Phantom 类型参数 (Phantom Type Parameters with Ability Constraints)

Ability constraints and phantom type parameters are orthogonal features in the sense that phantom parameters can be declared with ability constraints.
When instantiating a phantom type parameter with an ability constraint, the type argument has to satisfy that constraint, even though the parameter is phantom.
For example, the following definition is perfectly valid:

能力约束和 phantom 类型参数是正交特征，因此 phantom 参数声明时可以用能力进行约束。当使用能力约束实例化一个 phantom 类型参数时，类型参数必须满足该约束，即使参数是。
例如，以下定义是完全有效的：

```move
struct S<phantom T: copy> {}
```

The usual restrictions apply and `T` can only be instantiated with arguments having `copy`.

应用(phantom)通常的限制并且 `T` 只能用具有 `copy` 的参数实例化。

## 约束 (Constraints)

In the examples above, we have demonstrated how one can use type parameters to define "unknown" types that can be plugged in by callers at a later time. This however means the type system has little information about the type and has to perform checks in a very conservative way. In some sense, the type system must assume the worst case scenario for an unconstrained generic. Simply put, by default generic type parameters have no [abilities](./abilities.md).

This is where constraints come into play: they offer a way to specify what properties these unknown types have so the type system can allow operations that would otherwise be unsafe.

在上面的例子中，我们已经演示了如何使用类型参数来定义“未知”类型，这些类型可以在稍后被调用者插入。然而，这意味着类型系统几乎没有关于类型的信息，并且必须以非常保守的方式执行检查。从某种意义上说，类型系统必须假设不受约束的泛型时的最坏场景。简单地说，默认情况下泛型类型参数没有[能力](./abilities.md)。

这就是约束发挥作用的地方：它们提供了一种方法来指定这些未知类型具有哪些属性，因此类型系统可以允许相应的操作，否则会不安全。

### 声明约束 (Declaring Constraints)

Constraints can be imposed on type parameters using the following syntax.

可以使用以下语法对类型参数施加约束。

```move
// T is the name of the type parameter
T: <ability> (+ <ability>)*
```

The `<ability>` can be any of the four [abilities](./abilities.md), and a type parameter can be constrained with multiple [abilities](./abilities.md) at once. So all of the following would be valid type parameter declarations

`<ability>` 可以是四种[能力](./abilities.md)中的任何一种，一个类型参数可以同时被多个能力约束。因此，以下所有内容都是有效的类型参数声明

```move
T: copy
T: copy + drop
T: copy + drop + store + key
```

### 验证约束 (Verifying Constraints)

Constraints are checked at call sites so the following code won't compile.

在调用的地方会检查约束，因此以下代码无法编译。

```move
struct Foo<T: key> { x: T }

struct Bar { x: Foo<u8> }
//                  ^ error! u8 does not have 'key'

struct Baz<T> { x: Foo<T> }
//                     ^ error! T does not have 'key'
```

```move
struct R {}

fun unsafe_consume<T>(x: T) {
    // error! x does not have 'drop'
}

fun consume<T: drop>(x: T) {
    // valid!
    // x will be dropped automatically
}

fun foo() {
    let r = R {};
    consume<R>(r);
    //      ^ error! R does not have 'drop'
}
```

```move
struct R {}

fun unsafe_double<T>(x: T) {
    (copy x, x)
    // error! x does not have 'copy'
}

fun double<T: copy>(x: T) {
    (copy x, x) // valid!
}

fun foo(): (R, R) {
    let r = R {};
    double<R>(r)
    //     ^ error! R does not have copy
}
```

For more information, see the abilities section on [conditional abilities and generic types](./abilities.html#conditional-abilities-and-generic-types)

有关更多信息，请参阅有关[条件能力与泛型类型](./abilities.md#conditional-abilities-and-generic-types)

## 递归的限制 (Limitations on Recursions)

### 递归结构 (Recursive Structs)

Generic structs can not contain fields of the same type, either directly or indirectly, even with different type arguments. All of the following struct definitions are invalid:

泛型结构不能直接或间接包含相同类型的字段，即使使用不同的类型参数也是如此。以下所有结构定义均无效：

```move
struct Foo<T> {
    x: Foo<u64> // error! 'Foo' containing 'Foo'
}

struct Bar<T> {
    x: Bar<T> // error! 'Bar' containing 'Bar'
}

// error! 'A' and 'B' forming a cycle, which is not allowed either.
struct A<T> {
    x: B<T, u64>
}

struct B<T1, T2> {
    x: A<T1>
    y: A<T2>
}
```

### 高级主题：类型-级别递归 (Advanced Topic: Type-level Recursions)

Move allows generic functions to be called recursively. However, when used in combination with generic structs, this could create an infinite number of types in certain cases, and allowing this means adding unnecessary complexity to the compiler, vm and other language components. Therefore, such recursions are forbidden.

Move语言允许递归调用泛型函数。但是，当与泛型结构体结合使用时，在某些情况下可能会创建无限数量的类型，这意味着会给编译器、vm 和其他语言组件增加不必要的复杂性。因此，这种递归是被禁止的。

被允许的用法:
```move
address 0x2 {
    module m {
        struct A<T> {}

        // Finitely many types -- allowed.
        // foo<T> -> foo<T> -> foo<T> -> ... is valid
        fun foo<T>() {
            foo<T>();
        }

        // Finitely many types -- allowed.
        // foo<T> -> foo<A<u64>> -> foo<A<u64>> -> ... is valid
        fun foo<T>() {
            foo<A<u64>>();
        }
    }
}
```

不被允许的用法:

```move
address 0x2 {
    module m {
        struct A<T> {}

        // Infinitely many types -- NOT allowed.
        // error!
        // foo<T> -> foo<A<T>> -> foo<A<A<T>>> -> ...
        fun foo<T>() {
            foo<Foo<T>>();
        }
    }
}

address 0x2 {
    module n {
        struct A<T> {}

        // Infinitely many types -- NOT allowed.
        // error!
        // foo<T1, T2> -> bar<T2, T1> -> foo<T2, A<T1>>
        //   -> bar<A<T1>, T2> -> foo<A<T1>, A<T2>>
        //   -> bar<A<T2>, A<T1>> -> foo<A<T2>, A<A<T1>>>
        //   -> ...
        fun foo<T1, T2>() {
            bar<T2, T1>();
        }

        fun bar<T1, T2> {
            foo<T1, A<T2>>();
        }
    }
}
```

Note, the check for type level recursions is based on a conservative analysis on the call sites and does NOT take control flow or runtime values into account.

请注意，类型级别递归的检查是基于对调用现场的保守分析，并且不考虑控制流或运行时值。

```move
address 0x2 {
    module m {
        struct A<T> {}

        fun foo<T>(n: u64) {
            if (n > 0) {
                foo<A<T>>(n - 1);
            };
        }
    }
}
```


The function in the example above will technically terminate for any given input and therefore only creating finitely many types, but it is still considered invalid by Move's type system.

上例中的函数将在技术上给定有限的输入，因此只会创建有限多的类型，但仍然会被 Move 语言的类型系统认为是无效的。
