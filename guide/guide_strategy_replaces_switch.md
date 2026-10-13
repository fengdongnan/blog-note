## 代码问题场景
```java

public double getBookPrice(final User user, final Book book) {

　　  double price = book.getPrice();

　　  switch (user.getLevel()) {
　　    case UserLevel.SILVER:
　　      return price * 0.9;
　　    case UserLevel.GOLD:
　　      return price * 0.8;
　　    case UserLevel.PLATINUM:
　　      return price * 0.75;
　　    default:
　　      return price;
　　  }
　　}

　　public double getEpubPrice(final User user, final Epub epub) {

　　  double price = epub.getPrice();

　　  switch (user.getLevel()) {
　　    case UserLevel.SILVER:
　　      return price * 0.95;
　　    case UserLevel.GOLD:
　　      return price * 0.85;
　　    case UserLevel.PLATINUM:
　　      return price * 0.8;
　　    default:
　　      return price;
　　  }
　　}

```

后续如果新增一个 userLevel , 则需要全局搜索用到user.getLevel() 进行switch判断的地方, 如果漏掉一处则会导致 新增 userLevel 走到 default price , 产生隐藏 bug ; 违反了 开闭原则
## 多态解法
引入一个 UserLevel 的模型，消除 switch
(这里称为多态是从语法实现层面, 其实语言层面的多态就是设计层面策略模式的实现工具)
```Java
　interface UserLevel {

　　  double getBookPrice(Book book);

　　  double getEpubPrice(Epub epub);

　　}

　　class RegularUserLevel implements UserLevel {

　　  public double getBookPrice(final Book book) {
　　    return book.getPrice();
　　  }

　　  public double getEpubPrice(final Epub epub) {
　　    return epub.getPrice();
　　}

　　class GoldUserLevel implements UserLevel {

　　  public double getBookPrice(final Book book) {
　　    return book.getPrice() * 0.8;
　　  }

　　  public double getEpubPrice(final Epub epub) {
　　    return epub.getPrice() * 0.85;
　　  }

　　}

　　class SilverUserLevel implements UserLevel {

　　  public double getBookPrice(final Book book) {
　　    return book.getPrice() * 0.9;
　　  }

　　  public double getEpubPrice(final Epub epub) {
　　    return epub.getPrice() * 0.85;
　　  }
　　}

　　class PlatinumUserLevel implements UserLevel {

　　  public double getBookPrice(final Book book) {
　　    return book.getPrice() * 0.75;
　　  }

　　  public double getEpubPrice(final Epub epub) {
　　    return epub.getPrice() * 0.8;
     }
  }
```

调用:
```java
　public double getBookPrice(final User user, final Book book) {
　　  UserLevel level = user.getUserLevel()
　　  return level.getBookPrice(book);
　　}

　　public double getEpubPrice(final User user, final Epub epub) {
　　  UserLevel level = user.getUserLevel()
　　  return level.getEpubPrice(epub);
　　}
```

有一个不足是后续如果需要新增一个价格维度, 则需要到所有实现类中修改, 比较繁琐


## 优化写法
策略模式抽象, 转为枚举写法
```java
public enum UserLevel {
    REGULAR(1.0, 1.0),
    SILVER(0.9, 0.95),
    GOLD(0.8, 0.85),
    PLATINUM(0.75, 0.8);

    private final double bookDiscount;
    private final double epubDiscount;

    UserLevel(double bookDiscount, double epubDiscount) {
        this.bookDiscount = bookDiscount;
        this.epubDiscount = epubDiscount;
    }

    public double getBookDiscount() { return bookDiscount; }
    public double getEpubDiscount() { return epubDiscount; }
}
```

调用处:
```java
public double getBookPrice(final User user, final Book book) {
    return book.getPrice() * user.getUserLevel().getBookDiscount();
}
```
如果未来新增会员等级，只需在枚举中加一行；如果新增商品折扣维度，也只局限在枚举的属性扩展上。

## 举一反三
这种代码写法可以扩展到更通用的场景: $N \times M$ 二维矩阵组合的场景, 也就是 M 有 x 种模式, N 也有 y 种模式, 且 y 是数值这种比较简单的取值

核心思想是: 以某一个核心维度作为主轴（声明为 Enum），将另一个维度的特化差异作为该枚举的字段、方法或行为内聚进去，从而将一个网状的、散落各处的双重 switch 扁平化为一个高内聚的常数级查找表

### 用户等级 × 商品品类
当前场景就是 角色维度(可转换为基准枚举)下配置有固定的数值, 需要取出进行运算处理

### 国家/地区 × 运输方式

在跨境/物流系统中, 每个国家针对不同运输方式（空运、海运、陆运）的时效阈值、最大申报限额、基础计费公式各不相同

原有写法:

```Java
// 散落在时效预估、运费计算、报关拦截各处
if (country == THAILAND) {
    if (shippingType == AIR) { ... }
    else if (shippingType == SEA) { ... }
} else if (country == ARGENTINA) {
    if (shippingType == AIR) { ... }
}
```

枚举内聚解法（以国家为主轴，绑定运输方式规则）：

```Java
public enum CountryShippingRule {
    THAILAND(
        new ShippingConfig(3, 1000.0),  // 空运: 3天, 限额1000
        new ShippingConfig(15, 5000.0)  // 海运: 15天, 限额5000
    ),
    ARGENTINA(
        new ShippingConfig(7, 500.0),
        new ShippingConfig(30, 2000.0)
    );

    private final ShippingConfig airConfig;
    private final ShippingConfig seaConfig;

    CountryShippingRule(ShippingConfig air, ShippingConfig sea) {
        this.airConfig = air;
        this.seaConfig = sea;
    }

    public ShippingConfig getConfig(ShippingType type) {
        return type == ShippingType.AIR ? airConfig : seaConfig;
    }
}
```
业务层只需调用 CountryShippingRule.of(country).getConfig(shippingType).getMaxValue()

### 适用边界
1.维度数量可控（二维）： 业务呈现明确的 $X \times Y$ 矩阵关系。如果超过三维（比如“国家 × 渠道 × 品类 × 货币”），枚举构造器参数会爆炸，那就必须退化为查数据库配置表。

2.规则是硬性系统契约，而不是运营每天乱改的配置：
- 适合放 Enum：系统协议、状态机扭转、系统级权限、稳定的业务等级规则。
- 不适合放 Enum：运营同学每周都要在后台调一次的营销打折活动（这种依然必须走数据库/配置中心）。

3.选哪一个作为主轴 Enum？
- 把更核心、更稳定或业务主体所在的维度作为外层 Enum（如 UserLevel 或 Country），把附随的、被计算的维度（如 BookDiscount、FeeConfig）作为它的属性。这样代码读起来最符合“主谓宾”的自然语意。