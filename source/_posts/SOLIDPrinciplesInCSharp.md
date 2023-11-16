---
title: C# 实践设计模式原则 SOLID
tags:
  - 程序设计
  - .NET
categories:
  - 编程
date: 2020-08-20 00:26:12
---

> https://www.cnblogs.com/tiger-wang/p/13525841.html

<!--more-->

通常讲到设计模式，一个最通用的原则是 SOLID：

1. S - Single Responsibility Principle，单一责任原则
2. O - Open Closed Principle，开闭原则
3. L - Liskov Substitution Principle，里氏替换原则
4. I - Interface Segregation Principle，接口隔离原则
5. D - Dependency Inversion Principle，依赖倒置原则

嗯，这就是五大原则。

后来又加入了一个：Law of Demeter，迪米特法则。于是，就变成了六大原则。

原则好理解。怎么用在实践中？

# 一、单一责任原则

单一责任原则，简单来说就是一个类或一个模块，只负责一种或一类职责。

看代码：

```cs
public interface IUser
{
    void AddUser();
    void RemoveUser();
    void UpdateUser();

    void Logger();
    void Message();
}
```

根据原则，我们会发现，对于`IUser`来说，前三个方法：`AddUser`、`RemoveUser`、`UpdateUser`是有意义的，而后两个`Logger`和`Message`作为`IUser`的一部分功能，是没有意义的并不符合单一责任原则的。

所以，我们可以把它分解成不同的接口：

```cs
public interface IUser
{
    void AddUser();
    void RemoveUser();
    void UpdateUser();
}
public interface ILog
{
    void Logger();
}
public interface IMessage
{
    void Message();
}
```

拆分后，我们看到，三个接口各自完成自己的责任，可读性和可维护性都很好。


下面是使用的例子，采用依赖注入来做：

```cs
public class Log : ILog
{
    public void Logger()
    {
        Console.WriteLine("Logged Error");
    }
}
public class Msg : IMessage
{
    public void Message()
    {
        Console.WriteLine("Messaged Sent");
    }
}
class Class_DI
{
    private readonly IUser _user;
    private readonly ILog _log;
    private readonly IMessage _msg;
    public Class_DI(IUser user, ILog log, IMessage msg)
    {
        this._user = user;
        this._log = log;
        this._msg = msg;
    }
    public void User()
    {
        this._user.AddUser();
        this._user.RemoveUser();
        this._user.UpdateUser();
    }
    public void Log()
    {
        this._log.Logger();
    }
    public void Msg()
    {
        this._msg.Message();
    }
}
public static void Main()
{
    Class_DI di = new Class_DI(new User(), new Log(), new Msg());
    di.User();
    di.Log();
    di.Msg();
}
```

这样的代码，看着就漂亮多了。

# 二、开闭原则

开闭原则要求类、模块、函数等实体应该对扩展开放，对修改关闭。

我们先来看一段代码，计算员工的奖金：

```cs
public class Employee
{
    public int Employee_ID;
    public string Name;
    public Employee(int id, string name)
    {
        this.Employee_ID = id;
        this.Name = name;
    }
    public decimal Bonus(decimal salary)
    {
        return salary * .2M;
    }
}
class Program
{
    static void Main(string[] args)
    {
        Employee emp = new Employee(101, "WangPlus");
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp.Employee_ID, emp.Name, emp.Bonus(10000));
    }
}
```

现在假设，计算奖金的公式做了改动。

要实现这个，我们可能需要对代码进行修改：

```cs
public class Employee
{
    public int Employee_ID;
    public string Name;
    public string Employee_Type;

    public Employee(int id, string name, string type)
    {
        this.Employee_ID = id;
        this.Name = name;
        this.Employee_Type = type;
    }
    public decimal Bonus(decimal salary)
    {
        if (Employee_Type == "manager")
            return salary * .2M;
        else
            return
                salary * .1M;
    }
}
```

显然，为了实现改动，我们修改了类和方法。

这违背了开闭原则。


那我们该怎么做？

我们可以用抽象类来实现 - 当然，实际有很多实现方式，选择最习惯或自然的方式就成：

```cs
public abstract class Employee
{
    public int Employee_ID;
    public string Name;
    public Employee(int id, string name)
    {
        this.Employee_ID = id;
        this.Name = name;
    }
    public abstract decimal Bonus(decimal salary);
}
```

然后，我们再实现最初的功能：

```cs
public class GeneralEmployee : Employee
{
    public GeneralEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal Bonus(decimal salary)
    {
        return salary * .2M;
    }
}
class Program
{
    public static void Main()
    {
        Employee emp = new GeneralEmployee(101, "WangPlus");
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp.Employee_ID, emp.Name, emp.Bonus(10000));
    }
}
```

在这儿使用抽象类的好处是：如果未来需要修改奖金规则，则不需要像前边例子一样，修改整个类和方法，因为现在的扩展是开放的。

代码写完整了是这样：

```cs
public abstract class Employee
{
    public int Employee_ID;
    public string Name;
    public Employee(int id, string name)
    {
        this.Employee_ID = id;
        this.Name = name;
    }
    public abstract decimal Bonus(decimal salary);
}

public class GeneralEmployee : Employee
{
    public GeneralEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal Bonus(decimal salary)
    {
        return salary * .1M;
    }
}
public class ManagerEmployee : Employee
{
    public ManagerEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal Bonus(decimal salary)
    {
        return salary * .2M;
    }
}
class Program
{
    public static void Main()
    {
        Employee emp = new GeneralEmployee(101, "WangPlus");
        Employee emp1 = new ManagerEmployee(102, "WangPlus1");
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp.Employee_ID, emp.Name, emp.Bonus(10000));
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp1.Employee_ID, emp1.Name, emp1.Bonus(10000));
    }
}
```

# 三、里氏替换原则

里氏替换原则，讲的是：子类可以扩展父类的功能，但不能改变基类原有的功能。它有四层含义：

1. 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法；
2. 子类中可以增加自己的特有方法；
3. 当子类重载父类的方法时，方法的前置条件（形参）要比父类的输入参数更宽松；
4. 当子类实现父类的抽象方法时，方法的后置条件（返回值）要比父类更严格。

在前边开闭原则中，我们的例子里，实际上也遵循了部分里氏替换原则，我们用`GeneralEmployee`和`ManagerEmployee`替换了父类`Employee`。


还是拿代码来说。

假设需求又改了，这回加了一个临时工，是没有奖金的。

```cs
public class TempEmployee : Employee
{
    public TempEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal Bonus(decimal salary)
    {
        throw new NotImplementedException();
    }
}
class Program
{
    public static void Main()
    {
        Employee emp = new GeneralEmployee(101, "WangPlus");
        Employee emp1 = new ManagerEmployee(101, "WangPlus1");
        Employee emp2 = new TempEmployee(102, "WangPlus2");
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp.Employee_ID, emp.Name, emp.Bonus(10000));
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp1.Employee_ID, emp1.Name, emp1.Bonus(10000));
        Console.WriteLine("Employee ID: {0} Name: {1} Bonus: {2}", emp2.Employee_ID, emp2.Name, emp2.Bonus(10000));
        Console.ReadLine();
    }
}
```

显然，这个方式不符合里氏替原则的第四条，它抛出了一个错误。

所以，我们需要继续修改代码，并增加两个接口：

```cs
interface IBonus
{
    decimal Bonus(decimal salary);
}
interface IEmployee
{
    int Employee_ID { get; set; }
    string Name { get; set; }
    decimal GetSalary();
}
public abstract class Employee : IEmployee, IBonus
{
    public int Employee_ID { get; set; }
    public string Name { get; set; }
    public Employee(int id, string name)
    {
        this.Employee_ID = id;
        this.Name = name;
    }
    public abstract decimal GetSalary();
    public abstract decimal Bonus(decimal salary);
}
public class GeneralEmployee : Employee
{
    public GeneralEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal GetSalary()
    {
        return 10000;
    }
    public override decimal Bonus(decimal salary)
    {
        return salary * .1M;
    }
}
public class ManagerEmployee : Employee
{
    public ManagerEmployee(int id, string name) : base(id, name)
    {
    }
    public override decimal GetSalary()
    {
        return 10000;
    }
    public override decimal Bonus(decimal salary)
    {
        return salary * .1M;
    }
}
public class TempEmployee : IEmployee
{
    public int Employee_ID { get; set; }
    public string Name { get; set; }
    public TempEmployee(int id, string name)
    {
        this.Employee_ID = id;
        this.Name = name;
    }
    public decimal GetSalary()
    {
        return 5000;
    }
}
class Program
{
    public static void Main()
    {
        Employee emp = new GeneralEmployee(101, "WangPlus");
        Employee emp1 = new ManagerEmployee(102, "WangPlus1");
        Console.WriteLine("Employee ID: {0} Name: {1} Salary: {2} Bonus:{3}", emp.Employee_ID, emp.Name, emp.GetSalary(), emp.Bonus(emp.GetSalary()));
        Console.WriteLine("Employee ID: {0} Name: {1} Salary: {2} Bonus:{3}", emp1.Employee_ID, emp1.Name, emp1.GetSalary(), emp1.Bonus(emp1.GetSalary()));

        List<IEmployee> emp_list = new List<IEmployee>();
        emp_list.Add(new GeneralEmployee(101, "WangPlus"));
        emp_list.Add(new ManagerEmployee(102, "WangPlus1"));
        emp_list.Add(new TempEmployee(103, "WangPlus2"));
        foreach (var obj in emp_list)
        {
            Console.WriteLine("Employee ID: {0} Name: {1} Salary: {2} ", obj.EmpId, obj.Name, obj.GetSalary());
        }
    }
}
```

# 四、接口隔离原则

接口隔离原则要求客户不依赖于它不使用的接口和方法；一个类对另一个类的依赖应该建立在最小的接口上。

通常的做法，是把一个臃肿的接口拆分成多个更小的接口，以保证客户只需要知道与它相关的方法。

这个部分不做代码演示了，可以去看看上边单一责任原则里的代码，也遵循了这个原则。

# 五、依赖倒置原则

依赖倒置原则要求高层模块不能依赖于低层模块，而是两者都依赖于抽象。另外，抽象不应该依赖于细节，而细节应该依赖于抽象。

看代码：

```cs
public class Message
{
    public void SendMessage()
    {
        Console.WriteLine("Message Sent");
    }
}
public class Notification
{
    private Message _msg;

    public Notification()
    {
        _msg = new Message();
    }
    public void PromotionalNotification()
    {
        _msg.SendMessage();
    }
}
class Program
{
    public static void Main()
    {
        Notification notify = new Notification();
        notify.PromotionalNotification();
    }
}
```

这个代码中，通知完全依赖`Message`类，而`Message`类只能发送一种通知。如果我们需要引入别的类型，例如邮件和SMS，则需要修改`Message`类。

下面，我们使用依赖倒置原则来完成这段代码：

```cs
public interface IMessage
{
    void SendMessage();
}
public class Email : IMessage
{
    public void SendMessage()
    {
        Console.WriteLine("Send Email");
    }
}
public class SMS : IMessage
{
    public void SendMessage()
    {
        Console.WriteLine("Send Sms");
    }
}
public class Notification
{
    private IMessage _msg;
    public Notification(IMessage msg)
    {
        this._msg = msg;
    }
    public void Notify()
    {
        _msg.SendMessage();
    }
}
class Program
{
    public static void Main()
    {
        Email email = new Email();
        Notification notify = new Notification(email);
        notify.Notify();

        SMS sms = new SMS();
        notify = new Notification(sms);
        notify.Notify();
    }
}
```

通过这种方式，我们把代码之间的耦合降到了最小。

# 六、迪米特法则

迪米特法则也叫最少知道法则。从称呼就可以知道，意思是：一个对象应该对其它对象有最少的了解。

在写代码的时候，尽可能少暴露自己的接口或方法。写类的时候，能不public就不public，所有暴露的属性、接口、方法，都是不得不暴露的，这样能确保其它类对这个类有最小的了解。

这个原则没什么需要多讲的，调用者只需要知道被调用者公开的方法就好了，至于它内部是怎么实现的或是有其他别的方法，调用者并不关心，调用者只关心它需要用的。反而，如果被调用者暴露太多不需要暴露的属性或方法，那么就可能导致调用者滥用其中的方法，或是引起一些其他不必要的麻烦。

&emsp;  

最后说两句：所谓原则，不是规则，不是硬性的规定。在代码中，能灵活应用就好，不需要非拘泥于形式，但是，用好了，会让代码写得很顺手，很漂亮。