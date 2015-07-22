# 扩展系统功能——装饰模式（三）  

## 完整解决方案  

为了让系统具有更好的灵活性和可扩展性，克服继承复用所带来的问题，Sunny 公司开发人员使用装饰模式来重构图形界面构件库的设计，其中部分类的基本结构如图所示：

![图形界面构件库结构图](images/1333528353_8435.gif) 

在图中，Component 充当抽象构件类，其子类 Window、TextBox、ListBox 充当具体构件类，Component 类的另一个子类 ComponentDecorator 充当抽象装饰类，ComponentDecorator 的子类 ScrollBarDecorator 和 BlackBorderDecorator 充当具体装饰类。完整代码如下所示：

```
//抽象界面构件类：抽象构件类，为了突出与模式相关的核心代码，对原有控件代码进行了大量的简化
abstract class Component
{
       public  abstract void display();
}
 
//窗体类：具体构件类
class Window extends Component
{
       public  void display()
       {
              System.out.println("显示窗体！");
       }
}
 
//文本框类：具体构件类
class TextBox extends Component
{
       public  void display()
       {
              System.out.println("显示文本框！");
       }
}
 
//列表框类：具体构件类
class ListBox extends Component
{
       public  void display()
       {
              System.out.println("显示列表框！");
       }
}
 
//构件装饰类：抽象装饰类
class ComponentDecorator extends Component
{
       private Component component;  //维持对抽象构件类型对象的引用
 
       public ComponentDecorator(Component  component)  //注入抽象构件类型的对象
       {
              this.component = component;
       }
 
       public void display()
       {
              component.display();
       }
}
 
//滚动条装饰类：具体装饰类
class ScrollBarDecorator extends  ComponentDecorator
{
       public ScrollBarDecorator(Component  component)
       {
              super(component);
       }
 
       public void display()
       {
              this.setScrollBar();
              super.display();
       }
 
       public  void setScrollBar()
       {
              System.out.println("为构件增加滚动条！");
       }
}
 
//黑色边框装饰类：具体装饰类
class BlackBorderDecorator extends  ComponentDecorator
{
       public BlackBorderDecorator(Component  component)
       {
              super(component);
       }
 
       public void display()
       {
              this.setBlackBorder();
              super.display();
       }
 
       public  void setBlackBorder()
       {
              System.out.println("为构件增加黑色边框！");
       }
}
```

编写如下客户端测试代码：  

```
class Client
{
       public  static void main(String args[])
       {
              Component component,componentSB;  //使用抽象构件定义
              component = new Window(); //定义具体构件
              componentSB = new  ScrollBarDecorator(component); //定义装饰后的构件
              componentSB.display();
       }
}
```

编译并运行程序，输出结果如下：  

```
为构件增加滚动条！
显示窗体！
```

在客户端代码中，我们先定义了一个 Window 类型的具体构件对象 component，然后将 component 作为构造函数的参数注入到具体装饰类 ScrollBarDecorator 中，得到一个装饰之后对象 componentSB，再调用 componentSB 的 display() 方法后将得到一个有滚动条的窗体。如果我们希望得到一个既有滚动条又有黑色边框的窗体，不需要对原有类库进行任何修改，只需将客户端代码修改为如下所示：  

```
class Client
{
       public  static void main(String args[])
       {
              Component  component,componentSB,componentBB; //全部使用抽象构件定义
              component = new Window();
              componentSB = new  ScrollBarDecorator(component);
              componentBB = new  BlackBorderDecorator(componentSB); //将装饰了一次之后的对象继续注入到另一个装饰类中，进行第二次装饰
              componentBB.display();
       }
}
```

编译并运行程序，输出结果如下：

```
为构件增加黑色边框！
为构件增加滚动条！
显示窗体！
```

我们可以将装饰了一次之后的 componentSB 对象注入另一个装饰类 BlackBorderDecorator 中实现第二次装饰，得到一个经过两次装饰的对象 componentBB，再调用 componentBB 的 display() 方法即可得到一个既有滚动条又有黑色边框的窗体。  

如果需要在原有系统中增加一个新的具体构件类或者新的具体装饰类，无须修改现有类库代码，只需将它们分别作为抽象构件类或者抽象装饰类的子类即可。与图所示的继承结构相比，使用装饰模式之后将大大减少了子类的个数，让系统扩展起来更加方便，而且更容易维护，是取代继承复用的有效方式之一。