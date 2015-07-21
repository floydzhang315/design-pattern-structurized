# 不兼容结构的协调——适配器模式（三）  

## 类适配器  

除了对象适配器模式之外，适配器模式还有一种形式，那就是类适配器模式，类适配器模式和对象适配器模式最大的区别在于适配器和适配者之间的关系不同，对象适配器模式中适配器和适配者之间是关联关系，而类适配器模式中适配器和适配者是继承关系，类适配器模式结构如图所示：

![类适配器模式结构图](images/1362099343_7447.jpgg)   

根据类适配器模式结构图，适配器类实现了抽象目标类接口 Target，并继承了适配者类，在适配器类的 request() 方法中调用所继承的适配者类的 specificRequest() 方法，实现了适配。  

典型的类适配器代码如下所示：  

class Adapter extends Adaptee implements Target {  
    public void request() {  
        specificRequest();  
    }  
}  

由于 Java、C# 等语言不支持多重类继承，因此类适配器的使用受到很多限制，例如如果目标抽象类 Target 不是接口，而是一个类，就无法使用类适配器；此外，如果适配者 Adapter 为最终（Final）类，也无法使用类适配器。在 Java 等面向对象编程语言中，大部分情况下我们使用的是对象适配器，类适配器较少使用。  

**思考**  
在类适配器中，一个适配器能否适配多个适配者？如果能，应该如何实现？如果不能，请说明原因？  
  
## 双向适配器  

在对象适配器的使用过程中，如果在适配器中同时包含对目标类和适配者类的引用，适配者可以通过它调用目标类中的方法，目标类也可以通过它调用适配者类中的方法，那么该适配器就是一个双向适配器，其结构示意图如图所示：  

![双向适配器结构示意图](images/1362100282_9857.jpg)   

双向适配器的实现较为复杂，其典型代码如下所示：  
  
```
class Adapter implements Target,Adaptee {  
    //同时维持对抽象目标类和适配者的引用  
    private Target target;  
    private Adaptee adaptee;  
      
    public Adapter(Target target) {  
        this.target = target;  
    }  
      
    public Adapter(Adaptee adaptee) {  
        this.adaptee = adaptee;  
    }  
      
    public void request() {  
        adaptee.specificRequest();  
    }  
      
    public void specificRequest() {  
        target.request();  
    }  
}  
```

在实际开发中，我们很少使用双向适配器。
