---
title: 'Java面向对象'
description: 'Java面向对象编程学习笔记'
pubDate: 'Sep 3 2026'
tags: ['Java', '面向对象']
---

## Java面向对象

### 一、怎么理解面向对象？简单说一下封装、继承、多态

面向对象是一种编程范式，它将现实世界中的事物抽象为对象。对象通常具有两类特征：一类是属性，也就是对象保存的数据；另一类是行为，也就是对象可以执行的方法。面向对象编程以对象为中心，通过对象之间的交互来完成程序功能。

例如，可以把一只狗抽象成 `Dog` 对象：它的名字、年龄属于属性，进食、奔跑、叫属于行为。这样可以将数据和操作数据的方法组织在一起，使代码更容易维护、复用和扩展。

Java 面向对象的三大特性包括：**封装、继承和多态**。

#### 1. 封装

封装是指将对象的属性（数据）和行为（方法）结合在一起，并隐藏对象的内部实现细节，只通过对象提供的接口与外界交互。

封装的主要作用是保护数据、降低代码之间的耦合，并让对象的使用方式更加简单。例如，类可以将字段设置为 `private`，再通过 `public` 方法控制外部对字段的访问和修改：

```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

外部代码不需要了解 `name` 的具体存储方式，只需要调用 `getName()` 和 `setName()` 即可。这种方式也方便在方法中加入校验逻辑。

#### 2. 继承

继承是一种让子类自动拥有父类属性和方法的机制。它可以减少重复代码，并建立类与类之间的层次关系。Java 中使用 `extends` 表示类继承：

```java
class Animal {
    void eat() {
        System.out.println("动物进食");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("狗在叫");
    }
}

Dog dog = new Dog();
dog.eat();  // 继承父类的方法
dog.bark(); // 调用子类自己的方法
```

继承体现了类与类之间的 `is-a` 关系，例如“狗是一种动物”。子类可以复用父类已有的代码，也可以增加自己的属性和方法，或者重写父类方法。

#### 3. 多态

多态是指不同类的对象对同一个消息作出不同响应。也就是说，同一个接口或父类引用，指向不同的实现对象时，调用同一个方法可能表现出不同的行为。

多态可以让程序面向父类或接口编程，而不需要依赖具体的子类实现，从而提高代码的灵活性、扩展性和复用性。

### 二、多态体现在哪些方面？

多态在 Java 中主要体现在以下几个方面：

#### 1. 方法重载：编译时多态

方法重载是指同一个类中可以定义多个同名方法，但它们的参数列表不同。参数列表的区别可以是参数类型、参数数量或参数顺序。编译器会根据传入参数的不同，在编译时确定要调用的方法。

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}

Calculator calculator = new Calculator();
calculator.add(1, 2);     // 调用 int 版本
calculator.add(1.0, 2.0); // 调用 double 版本
```

#### 2. 方法重写：运行时多态

方法重写是指子类提供父类中同名方法的具体实现。运行时，JVM 会根据对象的实际类型决定调用哪个版本的方法，这是实现多态的主要方式。

```java
class Animal {
    void sound() {
        System.out.println("动物发出声音");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("汪汪");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("喵喵");
    }
}

Animal animal = new Dog();
animal.sound(); // 输出“汪汪”

animal = new Cat();
animal.sound(); // 输出“喵喵”
```

虽然变量 `animal` 的编译时类型是 `Animal`，但它实际指向的对象可以是 `Dog` 或 `Cat`。调用 `sound()` 时，JVM 会根据对象的实际类型选择对应的重写方法。

#### 3. 接口与实现

多个不同的类可以实现同一个接口，并分别提供接口方法的具体实现。程序可以使用接口类型的引用调用这些方法，而不需要关心具体的实现类。

```java
interface Animal {
    void makeSound();
}

class Dog implements Animal {
    @Override
    public void makeSound() {
        System.out.println("汪汪");
    }
}

class Cat implements Animal {
    @Override
    public void makeSound() {
        System.out.println("喵喵");
    }
}

Animal animal = new Dog();
animal.makeSound(); // 调用 Dog 的实现

animal = new Cat();
animal.makeSound(); // 调用 Cat 的实现
```

#### 4. 向上转型和向下转型

在 Java 中，可以使用父类类型的引用指向子类对象，这叫向上转型。向上转型通常是安全的，也是实现多态的基础：

```java
class Animal {
}

class Dog extends Animal {
    void bark() {
        System.out.println("汪汪");
    }
}

Dog dog = new Dog();
Animal animal = dog; // 子类对象自动向上转型为父类类型
```

向上转型后，变量只能直接调用父类中定义的方法，但对象在运行时仍然是子类对象。

向下转型是将父类引用转换为子类类型，需要显式进行，并且存在类型不兼容的风险。如果父类引用实际指向的不是目标子类对象，运行时会抛出 `ClassCastException`：

```java
Animal animal = new Animal();
Dog dog = (Dog) animal; // 运行时抛出 ClassCastException
```

进行向下转型前，应该使用 `instanceof` 检查对象的实际类型：

```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal; // 确认是 Dog 对象后再进行转型
    dog.bark();
}
```

### 三、多态解决了什么问题？

多态允许父类或接口统一处理不同的子类对象。在实际代码中，调用方只依赖稳定的父类或接口，新增子类时通常不需要修改原有调用逻辑。

例如，定义一个统一的 `makeSound()` 接口后，`Dog`、`Cat` 等不同对象都可以通过同一个接口被调用，各自执行自己的实现。这样可以减少大量的 `if-else` 判断，提高代码的扩展性和可维护性。

多态也是许多设计模式和设计原则的基础，例如策略模式、基于接口而非实现编程、依赖倒置原则和里氏替换原则等。

### 四、面向对象的设计原则你知道有哪些吗？

面向对象编程中的六大原则：

- **单一职责原则（SRP）**：一个类应该只有一个引起它变化的原因，即一个类应该只负责一项职责。例如：考虑一个员工类，它应该只负责管理员工信息，而不应该负责其他无关工作。
- **开放封闭原则（OCP）**：软件实体应该对扩展开放，对修改封闭。例如：通过制定接口来实现这一原则，比如定义一个图形类，然后让不同类型的图形继承这个类，而不需要修改图形类本身。
- **里氏替换原则（LSP）**：子类对象应该能够替换掉所有父类对象。例如：一个正方形是一个矩形，但如果修改一个矩形的高度和宽度时，正方形的行为应该如何改变就是一个违反里氏替换原则的例子。
- **接口隔离原则（ISP）**：客户端不应该依赖那些它不需要的接口，即接口应该小而专。例如：通过接口抽象层来实现底层和高层模块之间的解耦，比如使用依赖注入。
- **依赖倒置原则（DIP）**：高层模块不应该依赖低层模块，二者都应该依赖于抽象；抽象不应该依赖于细节，细节应该依赖于抽象。例如：如果一个公司类包含部门类，应该考虑使用组合/聚合关系，而不是将公司类继承自部门类。
- **最少知识原则（Law of Demeter）**：一个对象应当对其他对象有最少的了解，只与其直接的朋友交互。

### 五、抽象类和普通类区别？

- **实例化**：普通类可以直接实例化对象，而抽象类不能被实例化，只能被继承。
- **方法实现**：普通类中的方法可以有具体的实现，而抽象类中的方法可以有实现，也可以没有实现。
- **继承**：普通类和抽象类在继承规则上完全一样——都只能被单继承（`extends`），都可以实现多个接口（`implements`），这一点两者没有区别。
- **实现限制**：普通类可以被其他类继承和使用，而抽象类一般用于作为基类，被其他类继承和扩展使用。

### 六、Java 抽象类和接口的区别是什么？

**两者的特点：**

- 抽象类用于描述类的共同特性和行为，可以有成员变量、构造方法和具体方法。适用于有明显继承关系的场景。
- 接口用于定义行为规范，可以多实现，只能有常量和抽象方法（Java 8 以后可以有默认方法和静态方法）。适用于定义类的能力或功能。

**两者的区别：**

- **实现方式**：实现接口的关键字为 `implements`，继承抽象类的关键字为 `extends`。一个类可以实现多个接口，但一个类只能继承一个抽象类。因此，使用接口可以间接地实现多重继承。
- **方法方式**：接口只有定义，不能有方法的实现；Java 1.8 中可以定义 `default` 方法体，而抽象类可以有定义与实现，方法可在抽象类中实现。
- **访问修饰符**：接口成员变量默认为 `public static final`，必须赋初值，不能被修改；接口中的抽象方法默认为 `public abstract`。从 Java 8 起接口可以定义 `default` 和 `static` 方法（带方法体），从 Java 9 起还可以定义 `private` 方法用于辅助 `default` 方法的实现。抽象类中成员变量默认为 `default` 访问权限，可在子类中被重新定义，也可被重新赋值；抽象方法被 `abstract` 修饰，不能被 `private`、`static`、`synchronized` 和 `native` 等修饰，必须以分号结尾，不带花括号。
抽象类用于继承，因此不能使用`final`修饰符，`final`会禁止类被继承或者方法被重写。
- **变量**：抽象类可以包含实例变量和静态变量，而接口只能包含常量（即静态常量）。

### 七、接口里面可以定义哪些方法？

- **抽象方法**

  抽象方法是接口的核心部分，所有实现接口的类都必须实现这些方法。抽象方法默认为 `public` 和 `abstract`，这些修饰符可以省略。

  ```java
  public interface Animal {
      void makeSound();
  }
  ```

- **默认方法**

  默认方法是在 Java 8 中引入的，允许接口提供具体实现。实现类可以选择重写默认方法。

  ```java
  public interface Animal {
      void makeSound();

      default void sleep() {
          System.out.println("Sleeping...");
      }
  }
  ```

- **静态方法**

  静态方法也是在 Java 8 中引入的，它们属于接口本身，可以通过接口名直接调用，而不需要实现类的对象。

  ```java
  public interface Animal {
      void makeSound();

      static void staticMethod() {
          System.out.println("Static method in interface");
      }
  }
  ```

- **私有方法**

  私有方法是在 Java 9 中引入的，用于在接口中为默认方法或其他私有方法提供辅助功能。这些方法不能被实现类访问，只能在接口内部使用。

  ```java
  public interface Animal {
      void makeSound();

      default void sleep() {
          System.out.println("Sleeping...");
          logSleep();
      }

      private void logSleep() {
          System.out.println("Logging sleep");
      }
  }
  ```

### 八、解释 Java 中的静态变量和静态方法

在 Java 中，静态变量和静态方法是与类本身关联的，而不是与类的实例（对象）关联。它们在内存中只存在一份，可以被类的所有实例共享。

#### 静态变量

静态变量（也称为类变量）是在类中使用 `static` 关键字声明的变量。它们属于类而不是任何具体的对象。主要特点：

- **共享性**：所有该类的实例共享同一个静态变量。如果一个实例修改了静态变量的值，其他实例也会看到这个更改。
- **初始化**：静态变量在类被加载时初始化，只会对其进行一次内存分配。
- **访问方式**：静态变量可以直接通过类名访问，也可以通过实例访问，但推荐使用类名。

示例：

```java
public class MyClass {
    static int staticVar = 0; // 静态变量

    public MyClass() {
        staticVar++; // 每创建一个对象，静态变量自增
    }

    public static void printStaticVar() {
        System.out.println("Static Var: " + staticVar);
    }
}

// 使用示例
MyClass obj1 = new MyClass();
MyClass obj2 = new MyClass();
MyClass.printStaticVar(); // 输出 Static Var: 2
```

#### 静态方法

静态方法是在类中使用 `static` 关键字声明的方法。类似于静态变量，静态方法也属于类，而不是任何具体的对象。主要特点：

- **无实例依赖**：静态方法可以在没有创建类实例的情况下调用。对于静态方法来说，不能直接访问非静态的成员变量或方法，因为静态方法没有上下文的实例。
- **访问静态成员**：静态方法可以直接调用其他静态变量和静态方法，但不能直接访问非静态成员。
- **多态性**：静态方法不支持重写（Override），但可以被隐藏（Hide）。

```java
public class MyClass {
    static int count = 0;

    // 静态方法
    public static void incrementCount() {
        count++;
    }

    public static void displayCount() {
        System.out.println("Count: " + count);
    }
}

// 使用示例
MyClass.incrementCount(); // 调用静态方法
MyClass.displayCount();   // 输出 Count: 1
```

#### 使用场景

- **静态变量**：常用于需要在所有对象间共享的数据，如计数器、常量等。
- **静态方法**：常用于助手方法（utility methods）、获取类级别的信息或者是没有依赖于实例的数据处理。
