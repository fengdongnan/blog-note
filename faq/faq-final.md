在 Java 中，被 `final` 修饰的对象属性（实例变量）必须且**只能被赋值一次**，且赋值过程必须**在对象创建完成（构造函数执行完毕）之前**结束。
## 创建时机

具体有 **3 种时机** 对 `final` 对象属性进行赋值：
### 1. 声明字段时直接赋值（显式初始化）

在定义变量的同时直接赋予初始值。

```java
public class Person {
    // 时机 1：声明时直接赋值
    private final String species = "Homo Sapiens";
}
```

### 2. 实例初始化块（构造代码块）中赋值

实例初始化块会在每次创建对象、构造函数执行之前被自动调用。

```Java
public class Person {
    private final String id;

    // 时机 2：在实例代码块中赋值
    {
        id = UUID.randomUUID().toString();
    }
}
```

### 3. 构造函数中赋值（空白 final / Blank Final）

如果声明时没有赋值，也没有在实例代码块中赋值，则该变量被称为**空白 final**。它**必须**在类的每一个构造函数中被赋值。

```Java
public class Person {
    private final String name;
    private final int age;

    // 时机 3：在构造函数中赋值
    public Person(String name) {
        this.name = name;
        this.age = 18; // 必须保证每一个 final 字段都被初始化
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

> **注意**：如果一个构造函数通过 `this(...)` 链式调用了另一个构造函数，则只需要在被调用的构造函数中完成赋值即可，不要重复赋值。

## 规则与限制

- **绝对不能在普通成员方法中赋值**：普通方法是在对象创建完成后才被调用的，且可能被调用多次，违背了 `final` 只能赋值一次且必须在创建时初始化的语义。
- **分支完整性要求**：如果在构造函数中有 `if-else` 或 `try-catch` 分支，编译器会强制要求**每一个分支路径**都能且只能给 `final` 属性赋值一次。

```Java
public class Demo {
    private final int number;

    public Demo(boolean flag) {
        if (flag) {
            number = 1;
        } else {
            number = 2; // 如果漏掉 else 分支，编译器会报错：variable number might not have been initialized
        }
    }
}
```

### 注
1. `static final`（类属性/静态常量）

对于静态 `final` 属性，由于它属于类而非单个对象，其赋值时机在**类加载时**完成，只有 **2 种时机**：
1. 声明时直接赋值：`private static final double PI = 3.14159;`
2. 在静态代码块（`static { ... }`）中赋值。

2.final 修饰的属性必须赋值:
如果既没有在声明时赋值、没有在代码块中赋值，也没有在构造函数中赋值，**代码将直接无法通过编译**

因为`final` 变量与普通变量在 Java 中的**初始化规则不同**：

1. **普通成员变量（非 final）：** 如果你不主动赋值，Java 会隐式地帮它**自动赋予默认值**（例如：数字赋 `0`，`boolean` 赋 `false`，对象引用赋 `null`）。
2. **`final` 成员变量：** Java **不会**为其提供默认值。编译器在编译期间会执行严格的“肯定赋值检查”（Definite Assignment Analysis），一旦发现有任何一条构造路径没给 `final` 变量赋值，编译器会直接报错：
  `error: variable age might not have been initialized`
