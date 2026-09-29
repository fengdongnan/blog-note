# 创建及销毁对象
## 静态工厂方法替代构造器
### 优势
1.更清晰的构造方法名称

采用传统构造器时, 构造器的名字必须和类名一致，只能靠参数列表来区分不同的构造器, 例如以下user类:

```java
public class User {
    private String id;
    private String phone;

    // 构造器 1：通过 ID
    public User(String id) {
        this.id = id;
    }

    // 报错！编译不通过！因为参数类型都是 String，JVM 无法区分这两个构造器
    // public User(String phone) {
    //     this.phone = phone;
    // }
}
```
或者为了绕过这个限制, 颠倒参数列表顺序: 一个 (String, int)，另一个 (int, String), 这样对 api 使用者很不友好

采用静态工厂方法, 可以解决 类型冲突 和 语义不明 问题:
```java
public class User {
    private String id;
    private String phone;

    private User() {} // 把构造器私有化，外部只能通过静态工厂方法进行创建
    
    public static User createById(String id) {//方法语义清晰
        User user = new User();
        user.id = id;
        return user;
    }
    
    public static User createByPhone(String phone) {
        User user = new User();
        user.phone = phone;
        return user;
    }
}
```
2.不必每次调用都创建新对象

类似享元模式, 创建对象时可以复用缓存的对象:
- 避免对象的大量创建, 造成 gc 压力
- 实现单例模式, 保证对于不可变值类，不存在两个相等的实例. （即：当且仅当 a == b 时 a.equals(b)，枚举 Enum 提供此实现）通过 a==b 地址比较也可以获得比 equals() 更高的比较效率

3.返回接口或者子类, 实现代码解耦
```java
List<String> list = List.of("A", "B", "C"); // Java 9+ 静态工厂方法
```
就是通过 静态工厂方法 实现的一个泛化工具, 直接调用获得返回值, 不关心其内部实现(否则需要显示 new ArrayList<>()).

> jdk历史演进:
> 
> Java 8 之前：接口里不能写静态方法。所以 JDK 设计了 Collection 接口，然后搞了个 Collections 辅助类（里面全都是静态工厂方法，比如 Collections.emptyList() 和其他排序, 创建线程集合等工具方法）。 
>
> Java 8 及以后：接口允许写 static 方法, 所以现在的 JDK 直接把静态工厂方法写在接口里（比如 List.of()、Set.of()）, 替代了Collections的一部分 集合创建方法。

4.根据入参类型动态调整返回不同的出参子类

根据传入参数的不同，返回对象的具体类（Class）可以发生变化。只要是声明返回类型的子类即可。甚至在后续版本升级时，返回的具体类也可以随时调整

参考EnumSet:EnumSet 没有 public 构造器，全靠静态工厂方法创建。在 OpenJDK 中，它的底层有两种不同的子类实现：

- 如果枚举类型元素数量 $\le$ 64 个：静态工厂会返回 RegularEnumSet 实例（内部用一个简单的 long 变量做按位操作，极其省内存、速度极快）。 
- 如果枚举类型元素数量 $\ge$ 65 个：静态工厂会返回 JumboEnumSet 实例（内部用一个 long[] 数组支持更多元素）。

如果静态工厂方法内部做迁移改动, 对调用方也是不感知的: 如果有性能更高的 SuperEnumSet，只需要改一下 EnumSet.noneOf() 静态工厂里的 if-else 条件即可

5.方法编写时，返回对象的类甚至不需要存在

JDBC SPI 框架  没看懂

### 缺点 

1.无法被继承:

强制采用静态工厂方法后构造器方法访问权限限定为 private, 子类无法调用父类的构造方法

2.API不易见性

### 命名规范
- from：类型转换方法，只接受单个参数，返回该类型的对应实例（如 Date.from(instant)）。 
- of：聚合方法，接受多个参数，把它们组装成该类型的实例（如 EnumSet.of(JACK, QUEEN, KING)）。 
- valueOf：比 from 和 of 更繁琐的替代形式（如 BigInteger.valueOf(...)）。 
- instance 或 getInstance：返回根据参数描述的实例（不保证每次都是新对象，可能返回缓存的同一个对象）。 
- create 或 newInstance：类似于 getInstance，但严格保证每次调用都返回一个全新的对象。 
- getType：用于工厂方法写在另一个不同的类里的情况，Type 表示返回对象的类型（如 Files.getFileStore(path)）。 
- newType：同上（在另一个类里），且严格保证每次返回全新对象（如 Files.newBufferedReader(path)）。 
- type：getType 和 newType 的极简简写版（如 Collections.list(...)）。

## Builder 建造者模式

当参数数量多，且存在大量可选参数

历史方案1: 重叠构造器模式

- 不可观性: 参数含义不直观 
- 参数不易控性: 容易混淆错传参数

历史方案2: JavaBeans 模式

不能保证多属性赋值的原子性

