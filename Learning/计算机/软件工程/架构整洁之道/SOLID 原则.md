---
aliases: 架构整洁知道
date created: Monday, April 28th 2025, 1:22:43 pm
date modified: Monday, April 28th 2025, 2:09:02 pm
---

SOLID 原则
依赖反转原则（DIP）主要想告诉我们的是，如果想要设计一个灵 活的系统，在源代码层次的依赖关系中就应该多引用抽象类型，而非 具体实现。 也就是说，在 Java 这类静态类型的编程语言中，==在使用 use、 import、include 这些语句时应该只引用那些包含接口、抽象类或者其 他抽象类型声明的源文件，不应该引用任何具体实现==。
***
## 不要在具体实现类上创建衍生类?
" 不要在具体实现类上创建衍生类 " 是面向对象设计中的一个原则，通常与**依赖倒置原则**和**开闭原则**相关。这一原则强调在设计系统时，应尽量避免直接从具体实现类（即具体类）派生出子类，而是应该依赖于抽象类或接口。以下是这一原则的详细解释：
### 1. 原因
- **减少耦合**：直接依赖具体类会导致系统的耦合度增加，使得代码变得难以维护和扩展。
- **灵活性**：使用抽象类或接口可以使代码更灵活，便于更换具体实现，而不需要修改依赖于这些实现的代码。
- **可测试性**：通过依赖于抽象，可以更容易地进行单元测试，因为可以使用模拟对象替代具体实现。
### 2. 例子
假设我们有一个具体的 `Car` 类和一个 `ElectricCar` 类，我们希望在 `ElectricCar` 基础上扩展功能。
#### 不推荐的做法
```python
class Car:
    def drive(self):
        print("Driving a car")
class ElectricCar(Car):
    def charge(self):
        print("Charging the electric car")
```
在这个例子中，`ElectricCar` 直接继承了 `Car`，这导致了对具体实现的依赖。
#### 推荐的做法
```python
class Vehicle:
    def drive(self):
        pass
class Car(Vehicle):
    def drive(self):
        print("Driving a car")
class ElectricCar(Vehicle):
    def drive(self):
        print("Driving an electric car")
    def charge(self):
        print("Charging the electric car")
```
在推荐的做法中，`Vehicle` 是一个抽象类或接口，`Car` 和 `ElectricCar` 都依赖于它。这种设计使得系统更具扩展性和灵活性。
### 3. 总结
" 不要在具体实现类上创建衍生类 " 是为了促进良好的软件设计实践，确保代码的可维护性、可扩展性和可测试性。通过依赖于抽象而非具体实现，可以更容易地应对变化，减少系统的复杂性。
***
## 不要覆盖（override）包含具体实现的函数?
" 不要覆盖（override）包含具体实现的函数 " 是面向对象设计中的一个原则，主要与**开闭原则**（Open/Closed Principle）和**依赖倒置原则**（Dependency Inversion Principle）相关。这一原则强调在设计类时，尽量避免在子类中重写父类中已经具备具体实现的方法。以下是这一原则的详细解释：
### 1. 原因
- **保持功能一致性**：如果子类重写了父类的方法，可能会导致行为的不一致性，特别是在复杂系统中，可能会引入难以追踪的错误。
- **增强可维护性**：当子类重写父类的具体实现时，父类的行为可能会被破坏，导致维护变得困难。
- **促进代码复用**：如果父类的方法已经实现了某种功能，子类应该复用这些功能，而不是重复实现或修改。
### 2. 例子
假设我们有一个基类 `Animal`，它包含一个具体实现的 `make_sound` 方法，表示动物发出的声音。我们希望创建一个 `Dog` 类来表示狗。
#### 不推荐的做法
```python
class Animal:
    def make_sound(self):
        print("Some generic animal sound")
class Dog(Animal):
    def make_sound(self):  # 这里覆盖了父类的方法
        print("Bark")
```
在这个例子中，`Dog` 类重写了 `Animal` 类中的 `make_sound` 方法，这可能导致其他依赖于 `Animal` 类的代码无法正常工作，尤其当它们期望调用父类的实现时。
#### 推荐的做法
```python
class Animal:
    def make_sound(self):
        print("Some generic animal sound")
class Dog(Animal):
    def make_sound(self):  # 这里可以调用父类的方法
        super().make_sound()  # 调用父类的实现
        print("Bark")
```
在推荐的做法中，`Dog` 类仍然重写了 `make_sound` 方法，但它调用了父类的实现，从而保留了父类的功能，同时添加了特定于狗的行为。
### 3. 总结
" 不要覆盖包含具体实现的函数 " 旨在鼓励开发者在设计类时保持原有功能的一致性和完整性。通过复用父类的实现而不是直接重写，可以提高代码的可维护性和可读性，减少潜在的错误。遵循这一原则，有助于构建更健壮和灵活的面向对象系统。
***
## 工厂模式
在大部分面向对象编程语言中，人们都会选择用抽象工厂模式来解决这个源代码依赖的问题。 
下面，我们通过图 11.1 来描述一下该设计模式的结构。如你所见，`Application` 类是通过 Service ` ` 接口来使用 ConcreteImpl 类的。然而，`Application` 类还是必须要构造 `ConcreteImpl` 类实例。
于是，为了避免在源代码层次上引入对 `ConcreteImpl` 类具体实现的依赖，我们现在让 `Application` 类去调用 `ServiceFactory` 接口的 `makeSvc` 方法。
这个方法就由 `ServiceFactoryImpl` 类来具体提供,它是 `ServiceFactory` 的一个衍生类。该方法的具体实现就是初始化一个 `ConcreteImpl` 类的实例，并且将其以 `Service` 类型返回
![[Pasted image 20250428141207.png]] 图11.1中间的那条曲线代表了软件架构中的抽象层与具体实现层的边界。在这里，所有跨越这条边界源代码级别的依赖关系都应该是单向的，即具体实现层依赖抽象层。 
这条曲线将整个系统划分为两部分组件：抽象接口与其具体实现。抽象接口组件中包含了应用的所有高阶业务规则，而具体实现组件中则包括了所有这些业务规则所需要做的具体操作及其相关的细节信息。 
请注意，这里的控制流跨越架构边界的方向与源代码依赖关系跨越该边界的方向正好相反，源代码依赖方向永远是控制流方向的反转一一这就是DIP被称为依赖反转原则的原因。

在图11.1中，具体实现组件的内部仅有一条依赖关系，这条关系其实是违反DIP的。这种情况很常见，我们在软件系统中并不可能完全消除违反DIP的情况。通常只需要把它们集中于少部分的具体实现组件中，将其与系统的其他部分隔离即可。绝大部分系统中都至少存在一个具体实现组件——我们一般称之为 `main` 组件，因为它们通常是 `main` 函数所在之处。在图11.1中， `main` 函数应该负责创建 `ServiceFactoryImpl` 实例，并将其赋值给类型为 `ServiceFactory` 的全局变量，以便让 `Application` 类通过这个全局变量来进行相关调用。