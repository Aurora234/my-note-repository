## 原理

### java运行原理

Java 的运行机制是其“一次编写，到处运行”（Write Once, Run Anywhere）理念的核心。它依赖于 **Java 虚拟机（JVM）** 和 **字节码（Bytecode）** 的设计，将源代码与底层硬件/操作系统解耦。以下是 Java 运行机制的详细解释：

**整体流程概览**

```bash
.java 源代码 
    ↓ （编译）
.class 字节码文件 
    ↓ （由 JVM 加载并解释/编译执行）
在任意支持 JVM 的平台上运行
```

 **1.** ***编写源代码（.java 文件）***

- 开发者用 Java 语言编写程序，保存为 `.java` 文件。
- 示例：`HelloWorld.java`

 **2.** ***编译成字节码（.class 文件）***

- 使用 **`javac` 编译器** 将 `.java` 文件编译为 **平台无关的字节码（bytecode）**，生成 `.class` 文件。

- 字节码不是机器码，而是一种中间格式，专为 JVM 设计。

- 命令示例：

  bash

  

  ```
  javac HelloWorld.java  # 生成 HelloWorld.class
  ```

> ✅ 特点：
>
> - 字节码与操作系统、CPU 架构无关。
> - 同一份 `.class` 文件可在 Windows、Linux、macOS 上运行（只要安装了对应平台的 JVM）。

------

 **3.** ***JVM 加载并执行字节码***

当运行 `java HelloWorld` 时，JVM 执行以下过程：

 **(1)** ***类加载（Class Loading）***

- **类加载器（ClassLoader）** 负责将 `.class` 文件加载到内存中。
- 分为：
  - Bootstrap ClassLoader（加载核心库，如 `java.lang.*`）
  - Extension ClassLoader（加载扩展库）
  - Application ClassLoader（加载用户类路径上的类）

**(2)** ***字节码验证（Verification）***

- JVM 验证字节码是否合法、安全（防止恶意代码破坏 JVM）。
- 检查类型安全、操作数栈溢出等。

**(3)** ***执行引擎（Execution Engine）***

这是真正“运行”代码的部分，主要有两种方式：

 **a.** ***解释执行（Interpretation）***

- JVM 内置的 **解释器（Interpreter）** 逐条读取字节码并翻译成机器指令执行。
- 优点：启动快；缺点：执行慢。

 **b.** ***即时编译（JIT Compilation, Just-In-Time）***

- 对频繁执行的“热点代码”（hot spots），JVM 会通过 **JIT 编译器**（如 HotSpot 中的 C1/C2 编译器）将其**编译成本地机器码**，直接由 CPU 执行。
- 优点：长期运行性能接近 C/C++；缺点：首次执行有编译开销。

> 🔥 现代 JVM（如 HotSpot）结合了解释 + JIT，实现“启动快 + 运行快”。

**(4)** ***垃圾回收（Garbage Collection, GC）***

- JVM 自动管理内存，通过 GC 回收不再使用的对象，避免内存泄漏。
- 开发者无需手动 `free` 内存。

### java内存分配

根据《Java 虚拟机规范》，JVM 在执行 Java 程序时会将内存划分为若干个运行时数据区：

```
+----------------------------------+
|           Method Area             | ← 存储类信息、常量、等（线程共享）（加载class文件）
+----------------------------------+
|           Heap (堆)               | ← 静态变量和几乎所有对象实例和数组都在这里分配（线程共享）（所有new的对象）
+----------------------------------+
| Thread 1: Stack (虚拟机栈)        | ← 存储局部变量、方法调用帧（线程私有）
| Thread 2: Stack                   |
| ...                               |
+----------------------------------+
|         PC Register              | ← 当前线程执行的字节码行号（线程私有）
+----------------------------------+
|   Native Method Stack            | ← 本地方法（如 JNI 调用）使用的栈（线程私有）
+----------------------------------+
```

> ✅ **重点**：
>
> - **堆（Heap）** 是 Java 内存分配的主战场，**几乎所有对象都分配在堆上**。
> - **栈（Stack）** 存放基本类型变量和对象引用（reference），但**不存放对象本身**。





## 基础语法

### 逻辑语法

**switch**

> default和case的位置任意，但注意case穿透特性

> 表达式只能是：字符/整数byte short int/ 枚举 / 字符串

新语法jdk14以上

- 箭头标签
- case后面可以写多个值，用“,"隔开
- switch可以运行结果
- yield关键字，结合上一条特性使用。

> 如果case只有一行可以省略大括号和yield

 **1.箭头语法 `->`（单行表达式）**

适用于简单逻辑，自动返回右侧表达式的值。

java



```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

> 🔔 注意：
>
> - 多个常量用逗号分隔：`case A, B, C -> ...`
> - **必须覆盖所有可能值**，否则编译报错（除非有 `default`）

------

**2.代码块语法`{ ... yield ... }`（多行逻辑）**

当需要执行多条语句、条件判断或副作用（如日志）时使用：

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> {
        System.out.println("Weekend or start");
        yield 6; // 必须用 yield 返回值
    }
    case TUESDAY -> {
        boolean isHoliday = checkHoliday();
        yield isHoliday ? 0 : 7;
    }
    default -> throw new IllegalStateException("Invalid day: " + day);
};
```

> ⚠️ 规则：
>
> - 使用 `{}` 就必须用 `yield` 显式返回（或抛异常）
> - 不能混用 `break`（旧语法）和 `yield`（新语法）

------

**3.作为表达式 vs 作为语句**

- **表达式**：有返回值，可赋值给变量或用于计算：

  ```java
  String result = switch (status) { ... };
  ```

- **语句**：无返回值，仅执行操作（但仍推荐用新语法）：

  ```java
  switch (command) {
      case "start" -> System.out.println("Starting...");
      case "stop"  -> System.out.println("Stopping...");
      default      -> System.err.println("Unknown command");
  }
  ```



### 存储容器

#### 数组

静态初始化：

```java
int arr[] = new int [] {1, 2, 3};
int arr [] = {1, 2, 3};
int[] arr = {1,2,3};
```

获取数组长度：

```java
length = arr.length;
IDEA快捷语法：快捷遍历数组
arr.fori
```

动态初始化：

```java
int arr [] = new int [3]
```

#### 随机数

```java
Radom r = new Radom();
int n = r.nextInt(10); 0 ~ 10不含
int n = r.nextInt(2, 10); 2 ~ 10不含
```

#### 枚举

是一个特殊的javabean类

```java
public enum OrderStatus {
    // 订单状态枚举值
    PENDING("待支付", 1),
    PAID("已支付", 2),
    SHIPPED("已发货", 3),
    DELIVERED("已送达", 4),
    CANCELLED("已取消", 5),
    REFUNDED("已退款", 6);

    // 状态描述
    private final String description;
    // 状态编码
    private final int code;

    // 构造方法
    private OrderStatus(String description, int code) {
        this.description = description;
        this.code = code;
    }

    // 获取状态描述
    public String getDescription() {
        return description;
    }

    // 获取状态编码
    public int getCode() {
        return code;
    }

    // 根据编码获取订单状态
    public static OrderStatus fromCode(int code) {
        for (OrderStatus status : OrderStatus.values()) {
            if (status.getCode() == code) {
                return status;
            }
        }
        throw new IllegalArgumentException("未知的订单状态编码: " + code);
    }

    // 重写 toString 方法，方便打印
    @Override
    public String toString() {
        return "OrderStatus{" +
                "description='" + description + '\'' +
                ", code=" + code +
                '}';
    }
}

```

```java
//测试
public class Main {
    public static void main(String[] args) {
        // 获取订单状态
        OrderStatus status = OrderStatus.PAID;
        System.out.println("当前状态: " + status.getDescription());
        System.out.println("状态编码: " + status.getCode());

        // 根据编码查找状态
        OrderStatus foundStatus = OrderStatus.fromCode(3);
        System.out.println("编码为3的状态: " + foundStatus);

        // 遍历所有状态
        for (OrderStatus s : OrderStatus.values()) {
            System.out.println(s);
        }
    }
}
```

1. 每一个枚举项，都是该枚举类的对象，每一个对象都是通过构造方法创建出来的
2. 枚举项在底层就是常量，默认用publc static final
3. 枚举类的第一行必须是枚举项，枚举项之间用逗号隔开，一分号作为结尾
4. 枚举类的构造方法必须是private修饰，不让外界创建本类的对象
5. 编译器会给枚举类新增两个默认存在的方法：values(), valueOf()



### 方法

#### 方法重载

- 在同一个类中，定义了多个同名的方法，这些方法具有相似的功能
- 每个方法具有不同的参数类型和参数个数，这些同名的方法，就构成了方法重载
- 简单理解：同一个类，**方法名相同**，**参数列表不同（类型/个数/顺序）**的方法，无需看返回值类型





## 面向对象

### 面向对象原理

Java中基本数据类型不可以进行引用传递（无法实现一个方法进行交换变量的值），但所有对象都是引用传递



#### Java中对象的内存分配

1. Student stu = new Student(); 创建对象的七步：
   1. 加载class字节码文件
   2. 申明等号左边的局部变量
   3. 在堆里面开辟一个空间（对象）
   4. 给对象中的属性进行默认初始化
   5. 给对象中的属性进行显示初始化
   6. 给对象中的属性利用构造方法进行初始化
   7. 把对象的内存地址赋值给等号左边的变量
2. 方法出栈之后，方法里面的变量全部消失
3. 如果没有任何地方使用堆里面的对象，那么对象也会从堆里面消失。
4. 方法区里面的字节码信息一把不会消失，除非关闭虚拟机（关闭IDE）

### 关键字

**this关键字**：表示所在方法调用者（对象）的地址

##### **static关键字**

> 修饰成员变量：静态变量，被该类所有对象共享（**赋值一次，全实例共享，但可修改**）
>
> 调用方式：
>
> - 类名调用
> - 对象名调用
>
> 静态变量不属于对象，属于类
>
> 随着类的加载而加载，优先于对象而存在

>修饰成员方法：静态方法，多用于测试类和工具类中，Javabean类很少会用
>
>调用方式：
>
>- 类名调用
>- 对象名调用

- 静态方法只能访问静态变量和其他静态方法
- 非静态方法可以访问静态变量或者静态方法，也可以访问非静态的成员变量
- 静态方法中没有this关键字
- 静态只能调用静态，非静态都可以调用，静态无this

##### **final关键字**

> 表示最终，不可变。可以修饰变量、类、方法。赋值一次，不可修改

> 修饰基本数据类型：变量里面记录的数据无法改变
>
> 修饰引用数据类型：变量记录的内存不可改变，但对象的属性值可以改变



### 三大特征

- 封装：将对象的**内部状态（属性）和行为（方法）** 打包在一起，并**对外隐藏实现细节**，只通过受控的公共接口（如 public 方法）与外界交互。
- 继承：子类（Subclass）可以**继承**父类（Superclass）的**非私有属性和方法**，并在其基础上扩展或修改功能，实现“**is-a**”关系。
- 多态：同一个引用变量，可以指向不同子类的对象，并在运行时调用对应子类的方法。即“一种接口，多种形态”。

#### 继承与重写

使用 `extends` 关键字：

```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Barking!"); }
}

// 使用
Dog dog = new Dog();
dog.eat();  // 继承自 Animal
dog.bark(); // 自己的方法
```

- Java **只支持单继承**（一个类只能有一个直接父类）
- 可通过 **接口（interface）** 实现多重继承行为
- 构造器不能被继承，但可通过 `super()` 调用父类构造器
- 编译器会默认给javabean集成Object类

**super关键字**：可以调用父类的属性和方法， 如：`super.name`

**this关键字**：优先访问本类，在访问父类

**重写**：private私有方法，static静态方法，final最终方法不能被重写；子类重写的方法申明一般跟父类一致



**子类可以继承哪些内容？**：

- 构造方法：不能被继承，可以用super调用。
- 成员变量：可以被继承，private私有的也可以，但私有的无法直接调用。
- 成员方法：
  - 虚方法可以被继承（普通的方法，非static、final、private的方法）。
  - final修饰的最终方法不能被继承，可以被调用。编译时确定位置，运行时直接调用
  - static修饰的静态方法不能被继承，可以被调用。编译时修改为类名调用，运行时直接调用
  - private修饰的私有方法不能被继承，不能被调用
  - 方法重写：本质是子类替换虚方法表里面的地址值



**权限修饰符**

|           | 同一个类 | 本包中的其他类 | 不同包下的子类 | 不同包下的无关类 |
| :-------- | -------- | -------------- | -------------- | ---------------- |
| private   | 1        |                |                |                  |
| 缺省/默认 | 1        | 1              |                |                  |
| protect   | 1        | 1              | 1              |                  |
| public    | 1        | 1              | 1              | 1                |



#### 多态

实现条件：

1. 继承或实现接口
2. 方法重写（Override）
3. 父类引用指向子类对象

```java
class Animal {
    void sound() { System.out.println("Animal makes a sound"); }
}

class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow!"); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("Woof!"); }
}

// 多态使用
Animal myPet1 = new Cat(); // 父类引用指向 Cat 对象
Animal myPet2 = new Dog(); // 父类引用指向 Dog 对象

myPet1.sound(); // 输出：Meow!
myPet2.sound(); // 输出：Woof!
```

好处：

- **解耦调用者与具体实现**（程序依赖抽象，而非具体类）
- 提高可扩展性（新增子类无需修改调用代码）
- 支持设计模式（如策略模式、工厂模式）

> 多态的本质：**编译时看类型（Animal），运行时看对象（Cat/Dog）**

##### 注解

```java
@Override
public void show(){
    
}
```

- 注解 **不是代码的一部分**，而是“关于代码的代码”。
- 它们以 `@` 开头，例如：`@Override`、`@Deprecated`、`@SuppressWarnings`。
- 注解本身**不做任何事**，需要有**处理器（processor）** 来解析并执行相应逻辑（如编译器、IDE、Spring 框架等）。

##### 多态调用成员的特点

- 变量调用：编译看左边，运行也看左边
  - 编译看左边：在把java文件编译成class文件的时候，看父类中有没有这个变量，如果有编译成功，否则失败
  - 运行也看左边：在代码真正运行的的时候，使用父类中的变量
- 方法调用：编译看左边，运行看右边
  - 编译看左边：看父类中有没有这个方法，如果没有代码报错
  - 运行看右边：在代码真正运行的时候，运行的是子类里面的方法，如果子类没有重写父类里面的方法，使用的还是父类

### 抽象

**抽象方法**：

- **没有方法体**（即没有 `{}` 实现）的方法。
- 必须用 `abstract` 关键字修饰。
- **只能声明在抽象类或接口中**（在接口中可省略 `abstract`）。

```java
public abstract void draw(); // 没有 {}
```

- 抽象方法**只有声明，没有实现**。
- 子类**必须重写（override）** 所有抽象方法，否则子类也必须声明为 `abstract`。

**抽象类：**

- 用 `abstract` 关键字修饰的类。
- **不能被实例化**（不能用 `new` 创建对象）。
- 可以包含：
  - 抽象方法（0 个或多个）
  - 具体方法（有实现的普通方法）
  - 字段（成员变量）
  - 构造器（用于子类调用）

```java
public abstract class Shape {
    protected String color;

    // 具体方法（有实现）
    public void setColor(String color) {
        this.color = color;
    }

    // 抽象方法（子类必须实现）
    public abstract double getArea();
    public abstract void draw();
}
```

#### 关键性总结

1. **有抽象方法的类必须是抽象类**

   ```java
   // 错误！编译失败
   class MyClass {
       abstract void foo(); // ❌ 普通类不能有抽象方法
   }
   ```

2. **抽象类可以没有抽象方法**（但很少见）
   防止外界创建本类的对象

   ```java
   abstract class Utility { // 仅为了防止实例化
       public static void helper() { ... }
   }
   ```

3. **子类必须实现所有抽象方法，除非它也是抽象类**

   ```java
   abstract class PartialShape extends Shape {
       // 可以不实现 getArea()，因为自己也是 abstract
   }
   ```

4. **抽象类可以有构造器**（用于初始化共享字段）

   ```java
   abstract class Animal {
       protected String name;
       public Animal(String name) { this.name = name; }
   }
   class Dog extends Animal {
       public Dog(String name) { super(name); }
   }
   ```

### 接口

接口就是一个规则，是独立于继承体系以外的规则。

**接口（Interface）** 是一种**抽象类型**，用于定义**行为的契约（Contract）**。它规定了实现该接口的类**必须提供哪些方法**，但不关心这些方法的具体实现。接口是 Java 实现**多态**和**解耦设计**的核心机制之一。

**特点**

| 特性       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 完全抽象   | 接口中的方法默认是 `public abstract`（Java 8 前）            |
| 不能实例化 | 不能用 `new Interface()` 创建对象                            |
| 多实现     | 一个类可以 `implements` 多个接口（弥补 Java 单继承限制）     |
| 成员变量   | 所有字段默认是 `public static final`（即常量）               |
| 默认方法   | Java 8+ 支持 `default` 方法（带实现）                        |
| 静态方法   | Java 8+ 支持 `static` 方法（带实现）                         |
| 私有方法   | Java 9+ 支持 `private` 方法（用于辅助 `default`/`static` 方法） |

#### **接口的基本语法**

 **1.** **定义接口**

```java
public interface Drawable {
    // 抽象方法（可省略 public abstract）
    void draw();

    // 常量（可省略 public static final）
    String TYPE = "Vector Graphic";
}
```

 **2.** **实现接口**

```java
public class Circle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }
}

// 一个类可实现多个接口
public class Rectangle implements Drawable, Resizable {
    public void draw() { ... }
    public void resize() { ... }
}
```

> ✅ 必须实现接口中所有**抽象方法**，否则类必须声明为 `abstract`。
>
> 接口可以多继承，多个接口有同样的方法，实现类只需要重写一次

- 类和类的关系：只能单继承
- 类和接口：可以多实现接口，可以继承一个类的同时实现多个接口
- 接口和接口：接口可以多继承接口

#### **Java 8+ 接口的增强**

1. **默认方法（Default Methods)**

- 使用 `default` 关键字提供**默认实现**
- 允许在不破坏已有实现类的情况下**向后兼容地扩展接口**
- 如果实现多个接口中有相同名字的default方法，则必须进行重写

```java
public interface Drawable {
    void draw();

    default void setColor(String color) {
        System.out.println("Setting color to: " + color);
    }
}

// 实现类可选择是否重写 setColor()
class Circle implements Drawable {
    public void draw() { ... }
    // setColor() 自动继承
}
```

 **2. 静态方法（Static Methods）**

- 提供工具方法，通过接口名直接调用

```java
public interface MathUtils {
    static int max(int a, int b) {
        return a > b ? a : b;
    }
}

// 调用
int m = MathUtils.max(10, 20);
```

 **3. 私有方法（Java 9+）**

- 用于封装 `default` 或 `static` 方法中的公共逻辑

```java
public interface MyInterface {
    default void method1() { sharedLogic(); }
    default void method2() { sharedLogic(); }

    // 普通的私有方法
    private void sharedLogic() {
        System.out.println("Reusable code");
    }
}
```

```java
public interface MyInterface {
    public static void method1() { sharedLogic(); }
    public static void method2() { sharedLogic(); }
	// 静态的私有方法
    private static void sharedLogic() {
        System.out.println("Reusable code");
    }
}
```

### 内部类

**内部类（Inner Class）** 是指定义在**另一个类内部的类**。它提供了一种更优雅、更封装的方式来组织代码，尤其适用于表示“**紧密关联**”或“**辅助性**”的逻辑结构。

**1. 成员内部类（Non-static Inner Class）**

- 最常见的内部类。
- **必须依附于外部类的实例**才能创建。
- 可访问外部类的**所有成员**（包括 private）。

```java
public class Outer {
    private int x = 10;

    // 成员内部类
    class Inner {
        void print() {
            System.out.println("x = " + x); // 直接访问外部类私有字段
        }
    }

    public void createInner() {
        Inner in = new Inner(); // 在外部类中创建
        in.print();
    }
}

// 创建成员内部类的方式
// 外部创建方式1（需先有 Outer 实例）
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner(); // 注意语法：outer.new Inner()
// 外部创建方式2
Outer.Inner inner = new Outer.new Inner();
```

> ⚠️ 编译后生成：`Outer.class` 和 `Outer$Inner.class`
>
> 若内部类用private修饰，则可以在外部类提供一个方法返回新建的内部类对象

**访问变量**

若有多个重名变量时：

```java
public class Outer {

    int a = 30;

    class Inner {
        int a = 20;
        public void show() {
            int a = 10;
            System.out.println(a); //访问方法内的局部变量
            System.out.println(this.a);// 访问内部类的成员变量
            System.out.println(Outer.this.a);// 访问外部类的成员变量
        }
    }
```



------

 **2. 静态内部交类（Static Nested Class）**

- 用 `static` 修饰，**不持有外部类引用**。
- 更像一个**顶级类**，只是命名空间嵌套在外部类中。
- **不能访问外部类的非 static 成员**。否则需要创建外部类的对象

```java
public class Outer {
    private static String name = "Outer";
    private int value = 100; // 非 static

    static class StaticNested {
        void display() {
            System.out.println(name);      // ✅ 可访问 static 成员
            // System.out.println(value);  // ❌ 编译错误！
        }
    }
}

// 创建方式（无需外部类实例）
Outer.StaticNested nested = new Outer.StaticNested();
// 调用静态方法
Outer.Inner.display();
```

> ✅ **优点**：
>
> - 内存开销小（无隐式外部类引用）
> - 常用于 **Builder 模式**（如 `AlertDialog.Builder`）

注：只要是静态的东西，都可以类名点直接获取

 **3. 局部内部类（Local Inner Class）**

- 定义在**方法、构造器或作用域块**内。
- **只能在该作用域内使用**。类似方法内的局部变量
- 可访问：
  - 外部类的所有成员
  - 方法中的 **final 或 effectively final（事实 final）局部变量**

```java
public void process() {
    final int factor = 10; // 或去掉 final，只要不修改（effectively final）

    class LocalProcessor {
        void run() {
            System.out.println("Result: " + (x * factor)); // 访问外部类 x 和局部变量 factor
        }
    }

    LocalProcessor lp = new LocalProcessor();
    lp.run();
}
```

> ⚠️ 局部内部类**不能有访问修饰符**（如 `public`、`private`），也不能是 `static`。

------

 **4. 匿名内部类（Anonymous Inner Class）**重要

- **没有名字**的内部类，通常用于**继承类或实现接口**。
- 语法：`new 父类(参数) { ... }` 或 `new 接口() { ... }`
- 常用于简化**单方法接口**的实现（Java 8 前广泛用于事件处理）。
- 匿名内部类等价于：没有名字的java类 + 实现接口/继承类 + 重写方法 + 创建对象

```java
// 匿名内部类实现 Runnable 接口，并作为Thread构造器的参数
Thread t = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Running in anonymous class");
    }
});

// 等价于 Lambda（Java 8+）
Thread t2 = new Thread(() -> System.out.println("Lambda"));
```

> 🔍 编译后生成：`Outer$1.class`, `Outer$2.class`...





---



## 常用API

API技术文档：[Overview (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/)

**何时不需要导包**：

- 使用本包内的类
- 使用`java.lang`包

### Random

```java
import java.util.Random;

public class Test {

    static void main() {
        Random r = new Random();
        System.out.println(r.nextInt(100));
        System.out.println(r.nextDouble(1.5));

        for (int i=0; i<10; i++){
            System.out.println(r.nextDouble(2.5, 3.1));
        }
```

### String

- 所有的字符串都是java.lang包下的String类
- 字符串的内容是不可变的，被创建完之后，内容不可变

```java
    static void main() {
        String s = "abc";
        String s1 = new String("abc");//创建一个新的字符串对象
        String s2 = new String();

        char[] chars = {'a', 'b', 'c'};
        String s3 = new String(chars);

        byte[] bytes = {97, 98, 99};
        String s4 = new String(bytes);
        System.out.println(s4);

```

#### 内存管理

堆中的串池

<img src="C:\Users\aurora\Desktop\研究生\研究生任务\笔记\picture_bed\image-20260304222832496.png" alt="image-20260304222832496" style="zoom:67%;" />

<img src="C:\Users\aurora\Desktop\研究生\研究生任务\笔记\picture_bed\image-20260304223238995.png" alt="image-20260304223238995" style="zoom:80%;" />

#### 比较

- 基本数据类型比较的是数据值
- 引用数据类型（如字符串）比较的是地址
- 总结：“==”比较的是变量存储的内容，引用数据类型存储的是地址

```java
System.out.println("abc".equals("abc"));  // true
System.out.println("abc" == "abc");  //true 地址一样
System.out.println("abc".equals("abcd"));  //false
System.out.println("abc".equalsIgnoreCase("ABC"));  //true
```

#### 字符串索引

与数组类似

```java
String str = "hello world";
char c = str.charAt(7); // 获取指定索引位置的字符
int len = str.length(); // 获取字符串的长度
// str.length().fori 遍历字符串
for (int i = 0; i < len; i++) {
    System.out.println(str.charAt(i));
}
```

#### 拼接

```java
// 创建一个长度为10的随机字符串,并将其转换为字符串
Random random = new Random();
int[] arr = new int[10];
for (int i = 0; i < arr.length; i++) {
    arr[i] = random.nextInt((int)'a', (int)'z' + 1);
}
String str = "";
for (int i = 0; i < arr.length; i++) {
    str += (char)arr[i];
}
System.out.println(str);
```

#### 数据脱敏（截取和替换）

- String substring(int beginIndex, int endIndex)
  下标，包头不包尾

```java
String str = "significant";
String subStr1 = str.substring(5);// 截取从5位开始到末尾
String subStr2 = str.substring(0, 5); // 截取从0位开始到5位（不包含5位）
String newStr = str.substring(0,2) + "***" + str.substring(str.length()-1);
System.out.println(newStr);
```

```java
String str = "你玩的好菜啊，自刎归天吧你！";
// 第一个参数是被替换的字符串，第二个参数是替换的字符串
String res = str.replace("自刎归天", "***");
System.out.println( res); // 你玩的好菜啊，***吧你！
```

```java
// 定义一个敏感词库
String[] words = {"傻", "傻X", "傻B", "傻A", "傻C"};
String str = "你是傻X吧！";
for (int i = 0; i < words.length; i++) {
    if (str.contains(words[i])) {
        str = str.replace(words[i], "***");
    }
}
System.out.println(str);//你是***X吧！
```

#### 其他常用方法

```java
String str = "significant";
// 判断字符串中是否包含指定的字符串
boolean b1 = str.contains("sign"); // true
// 判断字符串的开头和结尾
boolean b2 = str.startsWith("sig"); // true
// 索引从1开始，是否以ign开头
boolean b3 = str.startsWith("ign", 1); // true
boolean b4 = str.endsWith("g"); // false

// 查找字符或者字符串第一次和最后一次出现的位置，如果不存在则返回-1
int index1 = str.indexOf("n"); // 3
int index2 = str.lastIndexOf("n"); // 9

// 判断字符串是否为 空
boolean b5 = str.isEmpty(); //  false
System.out.println(b5);

// 将字符串转换为字符数组
char[] chars = str.toCharArray();
for (char c : chars) {
    System.out.print(c);
}
//大小写转换
String str1 = str.toUpperCase();
String str2 = str.toLowerCase();
// 去除头尾的空格
String str3 = str.trim();

System.out.println( "%s,\n %s ".formatted(str1, str2) );
```

#### StringBulider

- 普通的字符串拼接方式很慢，因为字符串一经创建不可改变，在拼接时产生了很多冗余的字符串
- 而StringBulider是一个容器，拼接时把需要拼接的字符串放到容器中，没有冗余内容产生。

```java
// 展示StringBulider的拼接速度
long start = System.currentTimeMillis();

StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100000; i++){
    sb.append(i);
}

long end = System.currentTimeMillis();
System.out.println(end - start);
```

```java
        StringBuilder sb = new StringBuilder();
        // 拼接字符串
        sb.append("hello").append("world").append("java");
        // 反转字符串
        sb.reverse();
        int length = sb.length();
        // 容器转换为字符串
        String result = sb.toString();
```

### ArrayList集合

集合是一个长度可变的容器

<img src="C:\Users\aurora\Desktop\研究生\研究生任务\笔记\picture_bed\image-20260306200700912.png" alt="image-20260306200700912" style="zoom:67%;" />

**创建ArrayList对象**

```java
// 创建不指定类型的ArrayList集合,类似Python的list容器，可以添加任意数据类型的数据
ArrayList list = new ArrayList();
// 指定泛型，只能添加指定数据类型的数据
ArrayList<Integer> listInt = new ArrayList<>();
```

**常用方法：**

- 添加 add方法

  > 无法直接添加基本数据类型，若需要则需要转成对应的包装类

- 删除 remove
- 修改 set
- 查询 get
- 长度 size()

```java
// 指定泛型，只能添加指定数据类型的数据
ArrayList<String> listString = new ArrayList<>();

//添加元素
// add方法任意情况下都可以添加成功，永远不会失败
// 为何这样设计？：为了与其他类型的集合统一，方便使用，都继承自Collection接口，抽象方法就是这样设计的
listString.add("hello");
listString.add(1 , "world");
listString.add("java");
listString.add("good");

// 删除元素
// 根据内容删除
boolean res = listString.remove("hello");
// 根据索引删除，索引越界会报异常
String res1 = listString.remove(0);

// 修改元素
// 根据索引修改,并返回被修改的元素
String res2 = listString.set(0, "hello world");

// 查询元素
// 根据索引查询
String res3 = listString.get(0);

// 查询集合的长度
int len = listString.size();
```

