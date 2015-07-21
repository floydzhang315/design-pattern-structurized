# 深入浅出外观模式（三）  

## 抽象外观类

在标准的外观模式结构图中，如果需要增加、删除或更换与外观类交互的子系统类，必须修改外观类或客户端的源代码，这将违背开闭原则，因此可以通过引入抽象外观类来对系统进行改进，在一定程度上可以解决该问题。在引入抽象外观类之后，客户端可以针对抽象外观类进行编程，对于新的业务需求，不需要修改原有外观类，而对应增加一个新的具体外观类，由新的具体外观类来关联新的子系统对象，同时通过修改配置文件来达到不修改任何源代码并更换外观类的目的。

下面通过一个具体实例来学习如何使用抽象外观类：  

如果在应用实例“文件加密模块”中需要更换一个加密类，不再使用原有的基于求模运算的加密类 CipherMachine，而改为基于移位运算的新加密类 NewCipherMachine，其代码如下：

```
using System;

namespace FacadeSample
{
    class NewCipherMachine
    {
        public string Encrypt(string plainText) 
        {
		    Console.Write("数据加密，将明文转换为密文：");
		    string es = "";
		    int key = 10;//设置密钥，移位数为10
            char[] chars = plainText.ToCharArray();
            foreach(char ch in chars) 
            {
                int temp = Convert.ToInt32(ch);
                //小写字母移位
			    if (ch >= 'a' && ch <= 'z') {
				    temp += key % 26;
			        if (temp > 122) temp -= 26;
                    if (temp < 97) temp += 26;
			    }
                //大写字母移位
			    if (ch >= 'A' && ch <= 'Z') {
                    temp += key % 26;
                    if (temp > 90) temp -= 26;
                    if (temp < 65) temp += 26;
			    }
                es += ((char)temp).ToString();
		    }
            Console.WriteLine(es);
		    return es;
	    }
    }
}
```

如果不增加新的外观类，只能通过修改原有外观类 EncryptFacade 的源代码来实现加密类的更换，将原有的对 CipherMachine 类型对象的引用改为对 NewCipherMachine 类型对象的引用，这违背了开闭原则，因此需要通过增加新的外观类来实现对子系统对象引用的改变。  

如果增加一个新的外观类 NewEncryptFacade 来与 FileReader 类、FileWriter 类以及新增加的 NewCipherMachine 类进行交互，虽然原有系统类库无须做任何修改，但是因为客户端代码中原来针对 EncryptFacade 类进行编程，现在需要改为 NewEncryptFacade 类，因此需要修改客户端源代码。  

如何在不修改客户端代码的前提下使用新的外观类呢？解决方法之一是：引入一个抽象外观类，客户端针对抽象外观类编程，而在运行时再确定具体外观类，引入抽象外观类之后的文件加密模块结构图如图5所示：

![引入抽象外观类之后的文件加密模块结构图](images/1354689057_4412.jpg)  

在图中，客户类 Client 针对抽象外观类 AbstractEncryptFacade 进行编程，AbstractEncryptFacade 代码如下：  

```
namespace FacadeSample
{
    abstract class AbstractEncryptFacade
    {
        public abstract void FileEncrypt(string fileNameSrc, string fileNameDes);
    }
}
```

新增具体加密外观类 NewEncryptFacade 代码如下：  

```
namespace FacadeSample
{
    class NewEncryptFacade : AbstractEncryptFacade
    {
        private FileReader reader;
        private NewCipherMachine cipher;
        private FileWriter writer;

        public NewEncryptFacade()
        {
            reader = new FileReader();
            cipher = new NewCipherMachine();
            writer = new FileWriter();
        }

        public override void FileEncrypt(string fileNameSrc, string fileNameDes)
        {
            string plainStr = reader.Read(fileNameSrc);
            string encryptStr = cipher.Encrypt(plainStr);
            writer.Write(encryptStr, fileNameDes);
        }
    }
}
```

配置文件App.config中存储了具体外观类的类名，代码如下：  

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="facade" value="FacadeSample.NewEncryptFacade"/>
  </appSettings>
</configuration>
```

客户端测试代码修改如下：  

```
using System;
using System.Configuration;
using System.Reflection;

namespace FacadeSample
{
    class Program
    {
        static void Main(string[] args)
        {
            AbstractEncryptFacade ef; //针对抽象外观类编程
            //读取配置文件
            string facadeString = ConfigurationManager.AppSettings["facade"];
            //反射生成对象
            ef = (AbstractEncryptFacade)Assembly.Load("FacadeSample"). CreateInstance (facadeString);
            ef.FileEncrypt("src.txt", "des.txt");
            Console.Read();
        }
    }
}
```

编译并运行程序，输出结果如下：  

```
读取文件，获取明文：Hello world!
数据加密，将明文转换为密文：Rovvy gybvn!
保存密文，写入文件。
```

原有外观类 EncryptFacade 也需作为抽象外观类 AbstractEncryptFacade 类的子类，更换具体外观类时只需修改配置文件，无须修改源代码，符合开闭原则。  

## 外观模式效果与适用场景  

外观模式是一种使用频率非常高的设计模式，它通过引入一个外观角色来简化客户端与子系统之间的交互，为复杂的子系统调用提供一个统一的入口，使子系统与客户端的耦合度降低，且客户端调用非常方便。外观模式并不给系统增加任何新功能，它仅仅是简化调用接口。在几乎所有的软件中都能够找到外观模式的应用，如绝大多数 B/S 系统都有一个首页或者导航页面，大部分 C/S 系统都提供了菜单或者工具栏，在这里，首页和导航页面就是 B/S 系统的外观角色，而菜单和工具栏就是 C/S 系统的外观角色，通过它们用户可以快速访问子系统，降低了系统的复杂程度。所有涉及到与多个业务对象交互的场景都可以考虑使用外观模式进行重构。  

**1 模式优点**  

外观模式的主要优点如下：  

(1) 它对客户端屏蔽了子系统组件，减少了客户端所需处理的对象数目，并使得子系统使用起来更加容易。通过引入外观模式，客户端代码将变得很简单，与之关联的对象也很少。  

(2) 它实现了子系统与客户端之间的松耦合关系，这使得子系统的变化不会影响到调用它的客户端，只需要调整外观类即可。  

(3) 一个子系统的修改对其他子系统没有任何影响，而且子系统内部变化也不会影响到外观对象。  

**2 模式缺点**  

外观模式的主要缺点如下：  

(1) 不能很好地限制客户端直接使用子系统类，如果对客户端访问子系统类做太多的限制则减少了可变性和灵活 性。  

(2) 如果设计不当，增加新的子系统可能需要修改外观类的源代码，违背了开闭原则。  

**3 模式适用场景**  

在以下情况下可以考虑使用外观模式：  

(1) 当要为访问一系列复杂的子系统提供一个简单入口时可以使用外观模式。  

(2) 客户端程序与多个子系统之间存在很大的依赖性。引入外观类可以将子系统与客户端解耦，从而提高子系统的独立性和可移植性。  

(3) 在层次化结构中，可以使用外观模式定义系统中每一层的入口，层与层之间不直接产生联系，而通过外观类建立联系，降低层之间的耦合度。