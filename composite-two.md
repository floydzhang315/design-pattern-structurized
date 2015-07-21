# 树形结构的处理——组合模式（二）  

## 组合模式概述  

对于树形结构，当容器对象（如文件夹）的某一个方法被调用时，将遍历整个树形结构，寻找也包含这个方法的成员对象（可以是容器对象，也可以是叶子对象）并调用执行，牵一而动百，其中使用了递归调用的机制来对整个结构进行处理。由于容器对象和叶子对象在功能上的区别，在使用这些对象的代码中必须有区别地对待容器对象和叶子对象，而实际上大多数情况下我们希望一致地处理它们，因为对于这些对象的区别对待将会使得程序非常复杂。组合模式为解决此类问题而诞生，它可以让叶子对象和容器对象的使用具有一致性。  

组合模式定义如下：  

组合模式（Composite Pattern）：组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象（即叶子对象）和组合对象（即容器对象）的使用具有一致性，组合模式又可以称为“整体—部分”（Part-Whole）模式，它是一种对象结构型模式。  

在组合模式中引入了抽象构件类 Component，它是所有容器类和叶子类的公共父类，客户端针对 Component 进行编程。组合模式结构如图所示：

![组合模式结构图](images/1347029718_6268.jpg)    

在组合模式结构图中包含如下几个角色：  

- **Component（抽象构件）**：它可以是接口或抽象类，为叶子构件和容器构件对象声明接口，在该角色中可以包含所有子类共有行为的声明和实现。在抽象构件中定义了访问及管理它的子构件的方法，如增加子构件、删除子构件、获取子构件等。  

- **Leaf（叶子构件）**：它在组合结构中表示叶子节点对象，叶子节点没有子节点，它实现了在抽象构件中定义的行为。对于那些访问及管理子构件的方法，可以通过异常等方式进行处理。  

- **Composite（容器构件）**：它在组合结构中表示容器节点对象，容器节点包含子节点，其子节点可以是叶子节点，也可以是容器节点，它提供一个集合用于存储子节点，实现了在抽象构件中定义的行为，包括那些访问及管理子构件的方法，在其业务方法中可以递归调用其子节点的业务方法。  

**组合模式的关键是定义了一个抽象构件类，它既可以代表叶子，又可以代表容器，而客户端针对该抽象构件类进行编程，无须知道它到底表示的是叶子还是容器，可以对其进行统一处理。**同时容器对象与抽象构件类之间还建立一个聚合关联关系，在容器对象中既可以包含叶子，也可以包含容器，以此实现递归组合，形成一个树形结构。  

如果不使用组合模式，客户端代码将过多地依赖于容器对象复杂的内部实现结构，容器对象内部实现结构的变化将引起客户代码的频繁变化，带来了代码维护复杂、可扩展性差等弊端。组合模式的引入将在一定程度上解决这些问题。  

下面通过简单的示例代码来分析组合模式的各个角色的用途和实现。对于组合模式中的抽象构件角色，其典型代码如下所示：  

```
abstract class Component {  
    public abstract void add(Component c); //增加成员  
    public abstract void remove(Component c); //删除成员  
    public abstract Component getChild(int i); //获取成员  
    public abstract void operation();  //业务方法  
}   
```

一般将抽象构件类设计为接口或抽象类，将所有子类共有方法的声明和实现放在抽象构件类中。对于客户端而言，将针对抽象构件编程，而无须关心其具体子类是容器构件还是叶子构件。  

如果继承抽象构件的是叶子构件，则其典型代码如下所示：  
  
```
class Leaf extends Component {
	public void add(Component c) { 
		//异常处理或错误提示 
	}	
		
	public void remove(Component c) { 
		//异常处理或错误提示 
	}
	
	public Component getChild(int i) { 
		//异常处理或错误提示
		return null; 
	}
	
	public void operation() {
		//叶子构件具体业务方法的实现
	} 
}
```

作为抽象构件类的子类，在叶子构件中需要实现在抽象构件类中声明的所有方法，包括业务方法以及管理和访问子构件的方法，但是叶子构件不能再包含子构件，因此在叶子构件中实现子构件管理和访问方法时需要提供异常处理或错误提示。当然，这无疑会给叶子构件的实现带来麻烦。  

如果继承抽象构件的是容器构件，则其典型代码如下所示：  

```
class Composite extends Component {
	private ArrayList<Component> list = new ArrayList<Component>();
	
	public void add(Component c) {
		list.add(c);
	}
	
	public void remove(Component c) {
		list.remove(c);
	}
	
	public Component getChild(int i) {
		return (Component)list.get(i);
	}
	
	public void operation() {
		//容器构件具体业务方法的实现
        //递归调用成员构件的业务方法
		for(Object obj:list) {
			((Component)obj).operation();
		}
	} 	
}
```

在容器构件中实现了在抽象构件中声明的所有方法，既包括业务方法，也包括用于访问和管理成员子构件的方法，如 add()、remove() 和 getChild() 等方法。需要注意的是在实现具体业务方法时，由于容器构件充当的是容器角色，包含成员构件，因此它将调用其成员构件的业务方法。在组合模式结构中，由于容器构件中仍然可以包含容器构件，因此在对容器构件进行处理时需要使用递归算法，即在容器构件的 operation() 方法中递归调用其成员构件的 operation() 方法。

**思考**  
在组合模式结构图中，如果聚合关联关系不是从 Composite 到 Component 的，而是从 Composite 到 Leaf 的，如图所示，会产生怎样的结果？  

![组合模式思考题结构图](images/1347029904_5585.jpg) 