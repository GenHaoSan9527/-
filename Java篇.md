## <font style="color:rgba(0, 0, 0, 0.88);">接口和抽象类有什么区别？</font>
### 回答重点
接口和抽象类在设计动机上有所不同。  
接口的设计是自上而下的。我们知晓某一行为，于是基于这些行为约束定义了接口，一些类需要有这些行为，因此实现对应的接口。  
抽象类的设计是自下而上的。我们写了很多类，发现它们之间有共性，有很多代码可以复用，因此将公共逻辑封装成一个抽象类，减少代码冗余。  
所谓的自上而下指的是先约定接口，再实现。  
而自下而上的是先有一些类，才抽象了共同父类（可能和学校教的不太一样，但是实战中很多时候都是因为重构才有的抽象）。

### 其他区别：
1）方法实现  
接口中的方法默认是 public 和 abstract（但在 Java8 之后可以设置 default 方法或者静态方法）。  
抽象类可以包含 abstract 方法（没有实现）和具体方法（有实现）。它允许子类继承并重用抽象类中的方法实现。  
2）构造函数和成员变量  
接口不能包含构造函数，接口中的成员变量默认为 public static final，即常量。  
抽象类可以包含构造函数，成员变量可以有不同的访问修饰符（如 private、protected、public），并且可以不是常量。  
3）多继承  
抽象类只能单继承，接口可以有多个实现。

### 扩展知识
#### 接口的演变
+ Java 8：引入了 default 和 static 方法，使得接口不仅仅是方法的声明，还可以提供具体的实现。default 方法允许在接口中添加新的方法实现，而不影响已经实现该接口的类。
+ Java 9：引入了私有方法，允许在接口中定义私有方法，用于 default 方法的内部逻辑复用。 
+ Java 14：引入了 sealed 接口（仅在某些子类中使用），进一步增强了接口的功能

## <font style="color:rgba(0, 0, 0, 0.88);">JDK 动态代理和 CGLIB 动态代理有什么区别？</font>
### 回答重点
JDK 动态代理是基于接口的，所以要求代理类一定是有定义接口的。  
CGLIB 基于 ASM 字节码生成工具，它是通过继承的方式生成目标类的子类来实现代理类，所以要注意 final 方法。  
它们之间的性能随着 JDK 版本的不同而不同，以下内容取自：haiq的博客

+ <font style="color:rgba(0, 0, 0, 0.88);">jdk6 下，在运行次数较少的情况下，jdk 动态代理与 cglib 差距不明显，甚至更快一些；而当调用次数增加之后，cglib 表现稍微更快一些</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">jdk7 下，情况发生了逆转！在运行次数较少（1,000,000）的情况下，jdk 动态代理比 cglib 快了差不多30%；而当调用次数增加之后(50,000,000)，动态代理比 cglib 快了接近1倍</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">jdk8 表现和 jdk7 基本一致</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">扩展 JDK 动态代理</font>
<font style="color:rgba(0, 0, 0, 0.88);">JDK 动态代理是基于接口的代理，因此要求代理类一定是有定义的接口，使用 java.lang.reflect.Proxy 类和 java.lang.reflect.InvocationHandler 接口实现。  
</font><font style="color:rgba(0, 0, 0, 0.88);">以下为一个简单 JDK 动态代理示例：</font>

<font style="color:rgba(0, 0, 0, 0.88);"></font>

```java
// 接口
public interface Service {
    void perform();
}

// 需要被代理的实现类
public class ServiceImpl implements Service {
    @Override
    public void perform() {
        System.out.println("mianshiya.com");
    }
}


```

jdk动态代理处理类

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class ServiceInvocationHandler implements InvocationHandler {
    private final Object target;

    public ServiceInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method invoke");
        Object result = method.invoke(target, args);
        System.out.println("After method invoke");
        return result;
    }
}

```

创建并使用动态代理对象



```java
import java.lang.reflect.Proxy;

public class DynamicProxyDemo {
    public static void main(String[] args) {
        Service target = new ServiceImpl();
        Service proxy = (Service) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new ServiceInvocationHandler(target)
        );

        proxy.perform();
    }
}

```

### 扩展 CGLIB
CGLIB 基于 ASM 字节码生成工具，它是通过继承的方式来实现代理类，所以不需要接口，可以代理普通类，但需要注意 final 方法（不可继承）。  
同样来看个示例：



```java
public class Service {
    public void perform() {
        System.out.println("mianshiya.com");
    }
}

```

CGLIB动态代理处理类

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class ServiceMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method invoke");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method invoke");
        return result;
    }
}

```

创建并使用动态代理对象

```java
import net.sf.cglib.proxy.Enhancer;

public class CglibDynamicProxyDemo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Service.class);
        enhancer.setCallback(new ServiceMethodInterceptor());

        Service proxy = (Service) enhancer.create();
        proxy.perform();
    }
}

```

## <font style="color:rgba(0, 0, 0, 0.88);">你使用过 Java 的反射机制吗？如何应用反射？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Java 的反射机制是指在运行时获取类的结构信息（如方法、字段、构造函数）并操作对象的一种机制。反射机制提供了在运行时动态创建对象、调用方法、访问字段等功能，而无需在编译时知道这些类的具体信息。  
</font><font style="color:rgba(0, 0, 0, 0.88);">反射机制的优点：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">可以动态地获取类的信息，不需要在编译时就知道类的信息。</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">可以动态地创建对象，不需要在编译时就知道对象的类型。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">可以动态地调用对象的属性和方法，在运行时动态地改变对象的行为。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
<font style="color:rgba(0, 0, 0, 0.88);">一般在业务编码中不会用到反射，在框架上用的较多，因为很多场景需要很灵活，不确定目标对象的类型，届时只能通过反射动态获取对象信息。  
</font><font style="color:rgba(0, 0, 0, 0.88);">例如 Spring 使用反射机制来读取和解析配置文件，从而实现依赖注入和面向切面编程等功能。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">反射的性能考虑：</font>
<font style="color:rgba(0, 0, 0, 0.88);">反射操作相比直接代码调用具有较高的性能开销，因为它涉及到动态解析和方法调用。  
</font><font style="color:rgba(0, 0, 0, 0.88);">所以在性能敏感的场景中，尽量避免频繁使用反射。可以通过缓存反射结果，例如把第一次得到的 Method 缓存起来，后续就不需要再调用 Class.getDeclaredMethod ，也就不需要再次动态加载了，这样就可以避免反射性能问题。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">反射基本概念：</font>
<font style="color:rgba(0, 0, 0, 0.88);">Class：反射机制的核心，通过 Class 类的实例可以获取类的各种信息。  
</font><font style="color:rgba(0, 0, 0, 0.88);">反射的主要功能：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">创建对象：通过 Class.newInstance() 或 Constructor.newInstance() 创建对象实例。</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">访问字段：使用 Field 类访问和修改对象的字段。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">调用方法：使用 Method 类调用对象的方法。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">获取类信息：获取类的名称、父类、接口等信息。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">反射的使用：</font>
<font style="color:rgba(0, 0, 0, 0.88);">1）获取 Class 对象:</font>

<font style="color:rgba(0, 0, 0, 0.88);"></font>

```java
Class<?> clazz = Class.forName("com.mianshiya.MyClass");
// 或者
Class<?> clazz = MyClass.class;
// 或者
Class<?> clazz = obj.getClass();

```

<font style="color:rgba(0, 0, 0, 0.88);">2）创建对象:</font>

```java
Object obj = clazz.newInstance(); // 已过时
Constructor<?> constructor = clazz.getConstructor();
Object obj = constructor.newInstance();

```

<font style="color:rgba(0, 0, 0, 0.88);">3）访问字段:</font>

```java
Field field = clazz.getField("myField");
field.setAccessible(true); // 允许访问 private 字段
Object value = field.get(obj);
field.set(obj, newValue);

```

<font style="color:rgba(0, 0, 0, 0.88);">4）调用方法:</font>

```java
Method method = clazz.getMethod("myMethod", String.class);
Object result = method.invoke(obj, "param");

```

### 反射的最佳实践：
+ 限制访问：尽量避免过度依赖反射，尤其是在性能关键的代码中。
+ 使用缓存：缓存反射获取的类、方法、字段等信息，减少反射操作的频率。 
+ 遵循设计原则：在设计系统时，尽量使用更稳定和易于维护的设计方案，只有在确实需要时才使用反射。

## .Java的执行流程是什么？
### 回答重点
Java 程序的执行流程经历了从编译到字节码的生成，再到类加载和 JIT 编译的过程，最终在 JVM 中执行。并且在程序运行过程中，JVM 负责内存管理、垃圾回收和线程调度等工作。  
主要流程如下：

1. 源代码：编写 .java 文件。
2. 编译：使用 javac 编译器生成 .class 字节码文件。 
3. 类加载：JVM 的类加载器加载 .class 文件到内存中。 
4. 解释执行：JVM 将字节码转为机器码执行。 
5. JIT 编译：JVM 根据需要将热点代码编译为机器码。 
6. 运行：执行 main 方法中的逻辑。 
7. 垃圾回收：JVM 管理内存，并回收不再使用的对象。 
8. 程序结束：main 方法结束，JVM 清理资源，退出程序。

## <font style="color:rgba(0, 0, 0, 0.88);">说说 Java 中 HashMap 的原理？</font>
### 回答重点
HashMap 是基于哈希表的数据结构，用于存储键值对（key - value ）。其核心是将键的哈希值映射到数组索引位置，通过数组 + 链表（在 Java 8 及之后是数组 + 链表 + 红黑树）来处理哈希冲突。

HashMap 使用键的 hashCode() 方法计算哈希值，并通过 indexFor 方法（JDK 1.7 及之后版本移除了这个方法，直接使用 (n - 1) & hash ）确定元素在数组中的存储位置。哈希值是经过一定扰动处理的，防止哈希值分布不均匀，从而减少冲突。

HashMap 的默认初始容量为 16，负载因子为 0.75。也就是说，当存储的元素数量超过 16×0.75 = 12 个时，HashMap 会发扩容操作，容量×2并重新分配元素位置。这种扩容是比较耗时的操作，频繁扩容会影响性能。

### 扩展知识
#### HashMap 的红黑树优化：
从 Java 8 开始，为了优化当多个元素映射到同一个哈希桶（即发生哈希冲突）时的查找性能，当链表长度超过 8 时，链表会转变为红黑树。红黑树是一种自平衡二叉搜索树，能够将最坏情况下的查找复杂度从 O(n) 降低到 O(log n)。  
如果树中元素的数量低于 6，红黑树会转换回链表，以减少不必要的树操作开销。

#### hashCode()和equals()的重要性：
HashMap 的键必须实现 hashCode() 和 equals() 方法。hashCode() 用于计算哈希值，以决定键的存储位置，而 equals() 用于比较两个键是否相同。在 put 操作时，如果两个键的 hashCode() 相同，但 equals() 返回 false ，则这两个键会被视为不同的键，存储在同一个桶的不同位置。  
误用 hashCode() 和 equals() 会导致 HashMap 中的元素无法正常查找或插入。

### 默认容量与负载因子的选择：
默认容量是 16，负载因子为 0.75，这个组合是在性能和空间之间找到的平衡。较高的负载因子（如 1.0）会减少空间浪费，但增加了哈希冲突的概率；较低的负载因子则会增加空间开销，但减少哈希冲突。  
如果已知 HashMap 的容量需求，建议提前设定合适的初始容量，以减少扩容带来的性能损耗。





## <font style="color:rgba(0, 0, 0, 0.88);">Java 中有哪些集合类？请简单介绍</font>
+ <font style="color:rgb(28, 31, 35);">LinkedList：基于双向链表，插入、删除快，查询速度慢。 </font>
+ <font style="color:rgb(28, 31, 35);">Vector：线程安全的动态数组，类似于 ArrayList，但开销较大。</font>

### <font style="color:rgb(28, 31, 35);">Set 接口：</font>
+ <font style="color:rgb(28, 31, 35);">HashSet：基于哈希表，元素无序，不允许重复。 </font>
+ <font style="color:rgb(28, 31, 35);">LinkedHashSet：基于链表和哈希表，维护插入顺序，不允许重复。 </font>
+ <font style="color:rgb(28, 31, 35);">TreeSet：基于红黑树，元素有序，不允许重复。</font>

<font style="color:rgb(28, 31, 35);">所以网上有些说 Set 是无序集合非常不准确，因为需要看具体的实现类。</font>

### <font style="color:rgb(28, 31, 35);">Queue 接口：</font>
+ <font style="color:rgb(28, 31, 35);">PriorityQueue：基于优先级堆，元素按照自然顺序或指定比较器排序。 </font>
+ <font style="color:rgb(28, 31, 35);">LinkedList：可以作为队列使用，支持 FIFO（先进先出）操作。</font>

### <font style="color:rgb(28, 31, 35);">Map 接口：</font>
<font style="color:rgb(28, 31, 35);">存储的是键值对，也就是给对象（value）设置了一个 key，这样通过 key 可以找到那个 value。</font>

+ HashMap：基于哈希表，键值对无序，不允许键重复。
+ LinkedHashMap：基于链表和哈希表，维护插入顺序，不允许键重复。
+ TreeMap：基于红黑树，键值对有序，不允许键重复。
+ Hashtable：线程安全的哈希表，不允许键或值为 null。
+ ConcurrentHashMap：线程安全的哈希表，适合高并发环境，不允许键或值为 null。

这题一般会在面试刚开始的时候被问，主要用于暖场热身，不需要说的那么详细，大致讲下，面试官会根据你回答的点继续挖的。

PS：可尝试结合类图，对常见的实现类关系重点记忆，面试时可对自己擅长的实现类详细介绍或引导，也可做一定的取舍。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745740306416-7973d763-f696-4dff-b60b-946084ab2f8c.png)

## <font style="color:rgba(0, 0, 0, 0.88);">Java 中 HashMap 的扩容机制是怎样的？</font>
### 回答重点
HashMap 中的扩容是基于负载因子（load factor）来决定的。默认情况下，HashMap 的负载因子为 0.75，这意味着当 HashMap 的已存储元素数量超过当前容量的 75% 时，就会触发扩容操作。

例如，初始容量为 16，负载因子为 0.75，则扩容阈值为 16 × 0.75 = 12 。当存入第 13 个元素时，HashMap 就会触发扩容。

当触发扩容时，HashMap 的容量会扩大为当前容量的两倍。例如，容量从 16 增加到 32，从 32 增加到 64 等。

扩容时，HashMap 需要重新计算所有元素的哈希值，并将它们重新分配到新的哈希桶中，这个过程称为rehashing。每个元素的存储位置会根据新容量的大小重新计算哈希值，并移动到新的数组中。

### 扩展知识
#### rehashing 细节
按照我们的思维，每一个元素应该是重新 hash 一个一个搬迁过去。  
在 1.7 的时候就是这样实现的，然而 1.8 在这里做了优化，关键点就在于数组的长度是 2 的次方，且扩容为 2 倍。  
因为数组的长度是 2 的 n 次方，所以假设以前的数组长度（16）二进制表示是 010000，那么新数组的长度（32）二进制表示是 100000，这个应该很好理解吧？  
它们之间的差别就在于高位多了一个 1，而我们通过 key 的 hash 值定位其在数组位置所采用的方法是（数组长度 - 1）& hash 。我们还是拿 16 和 32 长度来举例：  
16 - 1 = 15，二进制为 001111  
32 - 1 = 31，二进制为 011111  
所以重点就在 key 的 hash 值的从右往左数第五位是否是 1，如果是 1 说明需要搬迁到新位置，且新位置的下标就是原下标 +16（原数组大小），如果是 0 说明吃不到新数组长度的高位，那就还是在原位置，不需要迁移。  
所以，我们刚好拿老数组的长度（010000）来判断高位是否是 1，这里只有两种情况，要么是 1 要么是 0 。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745741957858-d47d0230-2a30-43e7-bc53-08d60548ff83.png)从上面的源码可以看到，链表的数据是一次性计算完，然后一堆搬运的，因为扩容时候，节点的下标变化只会是原位置，或者原位置+老数组长度，不会有第三种选择。  
上面的位操作，包括为什么是原下标+老数组长度等，如果你不理解的话，可以举几个数带进去算一算，就能理解了。

### 总结一下：
+ 当扩容发生时，HashMap 并不是简单地将元素复制到新数组中。每个元素的哈希值会根据新的数组容量重新计算，因此元素的存储位置会发生变化
+ Java 8 之后的扩容不需要每个节点重新 hash 算下标，因为元素的新位置只与高位有关，通过和老数组长度的 & 计算是否为 0 就能判断新下标的位置，因此链表中的元素可能只需要部分移动。这一优化减少了扩容时的计算开销。

### 扩容的考虑
每次扩容都需要遍历当前所有的元素，重新计算哈希值并移动它们到新的位置，因此扩容是一个比较耗时的操作。如果频繁扩容，整体性能会明显下降。

### 优化策略：
如果能预估到 HashMap 中大概会存储多少数据，可以在创建 HashMap 时就指定一个较大的初始容量，以减少扩容次数。例如，对于存储 100 万个元素的 HashMap，可以直接设置一个 1024×1024 的初始容量，避免多次扩容。



## <font style="color:rgba(0, 0, 0, 0.88);">你了解 Java 线程池的原理吗？</font>
### 回答重点
线程池是一种池化技术，用于预先创建并管理一组线程，避免频繁创建和销毁线程的开销，提高性能和响应速度。  
它几个关键的配置包括：核心线程数、最大线程数、空闲存活时间、工作队列、拒绝策略。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745826699361-587995f2-62bc-42af-9168-5684aa29189f.png)

主要工作原理如下：

1. 默认情况下线程不会预创建，任务提交之后才会创建线程（不过设置 prestartAllCoreThreads 可以预创建核心线程）。
2. 当核心线程满了之后不会新建线程，而是把任务堆积到工作队列中。 
3. 如果工作队列放不下了，然后才会新增线程，直至达到最大线程数。 
4. 如果工作队列满了，然后也已经达到最大线程数了，这时候来任务会执行拒绝策略。 
5. 如果线程空闲时间超过空闲存活时间，并且当前线程数大于核心线程数的则会销毁线程，直到线程数等于核心线程数（设置 allowCoreThreadTimeOut 为 true 可以回收核心线程，默认为 false）。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745826773321-97b5b661-8c5f-447f-874b-0f35de41b8f2.png)

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745826790868-e7f7c59d-3f21-43e5-bc6d-7f71931487fb.png)

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745826805334-95cd1615-88ac-4742-aee8-2b0b7eeb644d.png)

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745826820243-b397a49e-294e-4aac-a652-ccec3a5145e0.png)

### 为什么线程池要先使用阻塞队列，而不是直接增加线程？
因为每创建一个线程都会占用一定的系统资源（如栈空间、线程调度开销等），直接增加线程会迅速消耗系统资源，导致性能下降。  
使用阻塞队列可以将任务暂存，避免线程数量无限增长，确保资源利用率更高。  
如果阻塞队列都满了，说明此时系统负载很大，再去增加线程到最大线程数去消化任务即可。  
举个例子：老板现在手下有10个人在干活（核心线程数），突然活变多了，每个人干不过来了，此时老板不会立马招人，它会让这些活积累一下（放到阻塞队列中），看看过段时间能不能消化掉。如果老板发现这个活积累的实在太多了（队列满了），他才会继续招人（达到最大线程数）。这就是所谓的人员（线程）有成本。

## <font style="color:rgba(0, 0, 0, 0.88);">你使用过哪些 Java 并发工具类？</font>
### 回答重点
比如：ConcurrentHashMap、AtomicInteger、Semaphore、CyclicBarrier、CountDownLatch、BlockingQueue 等等。  
这个问题只要把你知道的一些并发类名字说出来就行了，然后等面试官选择其中一个去询问即可（一般需要结合简历中项目的业务场景，所以需要根据自己的业务提前准备）。  
具体的并发类分析看扩展知识。

### 扩展知识
#### 1. ConcurrentHashMap
+ **作用**：是一个线程安全且高效的哈希表，支持并发访问。
+ **用法**：多个线程可以同时进行读写操作，而不会导致线程安全问题。

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key1", 1);
Integer value = map.get("key1");
map.computeIfAbsent("key2", k -> 2);
```

#### 2. AtomicInteger
+ **作用**：提供一种线程安全的方式对 int 类型进行原子操作，如增减、比较。
+ **用法**：适用于需要频繁对数值进行无锁操作的场景。

```java
AtomicInteger atomicInt = new AtomicInteger(0);
atomicInt.incrementAndGet(); // 递增
atomicInt.decrementAndGet(); // 递减
atomicInt.compareAndSet(1, 2); // 比较并设置
```

### 3. Semaphore
+ **作用**：控制访问资源的线程数，可以用来实现限流或访问控制。
+ **用法**：在资源有限的情况下，控制同时访问的线程数量。

```java
Semaphore semaphore = new Semaphore(3);
try {
    semaphore.acquire(); // 获取许可
    // 执行任务
} finally {
    semaphore.release(); // 释放许可
}
```

### 4. CyclicBarrier
+ **作用**：让一组线程到达一个共同的同步点，然后一起继续执行。常用于分阶段任务执行。
+ **用法**：适用于需要所有线程在某个点都完成后再继续的场景。



```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("所有线程都到达了屏障点");
});
Runnable task = () -> {
    try {
        // 执行任务
        barrier.await(); // 等待其他线程
    } catch (Exception e) {
        e.printStackTrace();
    }
};
new Thread(task).start();
new Thread(task).start();
new Thread(task).start();

```

### 5. CountDownLatch
+ **作用**：一个线程（或多个）等待其他线程完成操作。
+ **用法**：适用于主线程需要等待多个子线程完成任务的场景。

```java
CountDownLatch latch = new CountDownLatch(3);
Runnable task = () -> {
    try {
        // 执行任务
    } finally {
        latch.countDown(); // 任务完成，计数器减一
    }
};
new Thread(task).start();
new Thread(task).start();
new Thread(task).start();
latch.await(); // 等待所有任务完成
System.out.println("所有任务都完成了");
```

### 6. BlockingQueue
+ **作用**：是一个线程安全的队列，支持阻塞操作，适用于生产者 - 消费者模式。
+ **用法**：生产者线程将元素放入队列，消费者线程从队列中取元素，队列为空时消费者线程阻塞。

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();
Runnable producer = () -> {
    try {
        queue.put("item"); // 放入元素
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
};
Runnable consumer = () -> {
    try {
        String item = queue.take(); // 取出元素
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
};
new Thread(producer).start();
new Thread(consumer).start();
```

## <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">什么是 Java 的 CAS（Compare-And-Swap）操作？</font>
### 回答重点
CAS 是一种硬件级别的原子操作，它比较内存中的某个值是否为预期值，如果是，则更新为新值，否则不做修改。  
**工作原理**：

+ **比较（Compare）**：CAS 会检查内存中的某个值是否与预期值相等。 
+ **交换（Swap）**：如果相等，则将内存中的值更新为新值。 
+ **失败重试**：如果不相等，说明有其他线程已经修改了该值，CAS 操作失败，一般会利用重试，直到成功。

### 扩展知识
#### 举例说明
我们经常有累加需求，比较一个值是否等于 1，如果等于 1 我们将它替换成 2，如果等于 2 替换成 3。  
这种比较在多线程的情况下就不安全，比如此时同时有两个线程执行到比较值是否等于 1，然后两个线程发现都等于 1。然后两个线程都将它变成了 2，这样明明加了两次，值却等于 2。

| 时间轴 | 线程 A | 线程 B |
| --- | --- | --- |
|  | a == 1? | a == 1? |
|  | a = 2 | a = 2 |


最终 a 执行了两次加法，但是值等于 2。

这种情况其实加锁可以解决，但是加锁是比较消耗资源的。  
因此硬件层面就给予支持，将这个比较和交换的动作封装成一个指令，这样就保证了原子性，不会判断值确实等于 1，但是替换的时候值已经不等于 1 了。  
这指令就是 CAS。  
CAS 需要三个操作数，分别是旧的预期值(图中的 1)，变量内存地址(图中 a 的内存地址)，新值(图中的 2)。  
指令是根据变量地址拿到值，比较是否和预期值相等，如果是的话则替换成新值，如果不是则不替换。

#### CAS 的优缺点
**优点**：

+ **无锁并发**：CAS 操作不使用锁，因此不会导致线程阻塞，提高了系统的并发性和性能。 
+ **原子性**：CAS 操作是原子的，保证了线程安全。

**缺点**：

+ **ABA 问题**：CAS 操作中，如果一个变量值从 A 变成 B，又变回 A，CAS 无法检测到这种变化，可能导致错误。 
+ **自旋开销**：CAS 操作通常通过自旋实现，可能导致 CPU 资源浪费，尤其在高并发情况下。 
+ **单变量限制**：CAS 操作仅适用于单个变量的更新，不适用于涉及多个变量的复杂操作。

#### ABA 问题
ABA 问题是指当变量值从 A 变为 B 再变回 A 时，CAS 操作无法检测到这种变化。解决 ABA 问题的一种常见方法是引入版本号或时间戳，每次更新变量时同时更新版本号，从而检测到变量的变化。  
Java 中的 `AtomicStampedReference` 就提供了版本号解决方案，它内部提供了一个 `Pair` 封装了引用和版本号，利用 `volatile` 保证了可见性。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745827535091-45e35b95-3325-4b6b-af7f-4b1ed59c68fe.png)

在内部CAS中，添加了版本符号的对比：

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745827555402-60336cd5-c1a3-4780-9b13-0efc5a567196.png)

这样就避免了ABA的问题。简单实用示例如下



```java
    private AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(0, 0);

    public void updateValue(int expected, int newValue) {
        int[] stampHolder = new int[1];
        Integer currentValue = atomicStampedReference.get(stampHolder);
        int currentStamp = stampHolder[0];

        boolean updated = atomicStampedReference.compareAndSet(expected, newValue, currentStamp, currentStamp + 1);
        if (updated) {
            System.out.println("Value updated to " + newValue);
        } else {
            System.out.println("Update failed");
        }
    }

```

<font style="color:rgb(28, 31, 35);">Java 还提供了一个 `AtomicMarkableReference` 类，原理和 `AtomicStampedReference` 类似，差别就是它内部只要一个 bool 值，只能表示数据是否被修改过。 </font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745827671817-f853ccf2-286c-4072-aadf-145120edd1fd.png)而 `AtomicStampedReference` 中的 `stamp` 是 `int`，可以表现数据被修改了几次。其它原理都是一致的。

### CAS 在 Java 中的实现
在 Java 中，CAS 操作由 `sun.misc.Unsafe` 类提供，但该类是内部类，不推荐直接使用。具体是通过 JNI（Java Native Interface）调用这些底层的硬件指令来实现 CAS 操作，从而确保操作的原子性。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745827734385-1a2f3e0f-263a-4aef-bc88-7d4b1b54d44d.png)

<font style="color:rgb(28, 31, 35);">在 Java 中，可以使用并发包中 Atomic 类（如 AtomicInteger、AtomicLong 等），这些类封装了 CAS 操作，提供了线程安全的原子操作: </font>

```java
 boolean updated = atomicInteger.compareAndSet(expected, newValue);
    if (updated) {
        System.out.println("Value updated to " + newValue);
    } else {
        System.out.println("Update failed");
    }

```

CAS 操作的底层实现依赖于硬件的原子指令，如 x86 架构上的 cmpxchg 指令。  
在 JDK9 之前，会根据当前处理器是否是多处理器在 cmpxchg 前加上 lock 前缀，给对应写指令的内存区域加锁，使得其它处理器无法读写这块内存区域，保证指令执行的原子性。如果是单处理器则不会添加 lock 前缀指令。  
但是 JDK9 移除了这个判断，直接添加 lock 前缀指令（基本上市面上都是多处理器了）。

### CAS 总线风暴
lock 前缀指令会把写缓冲区中的所有数据立即刷新到主内存中。  
在对称多处理器架构下，每个 cpu 都会通过嗅探总线来检查自己的缓存是否过期。如果某个 cpu 刷新自己的数据到主存，就会通过总线通知其它 cpu 过期对应的缓存，这就实现了内存屏障，保证了缓存一致性。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1745827816708-f6719d49-8a0f-4645-8b26-3fb70a8eee41.png)

<font style="color:rgb(28, 31, 35);">而通过总线来回通信称为 cache 一致性流量。因为都需要通过总线通信，如果这个流量过大，总线就会成为瓶颈，导致本地缓存更新延迟。 如果 CAS 修改同一个变量并发很高，就会导致总线风暴。这也是 CAS 高并发下的一个性能瓶颈。 </font>

<font style="color:rgb(28, 31, 35);"></font>

## <font style="color:rgba(0, 0, 0, 0.88);">说说 AQS 吧？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">如果面试官问你为什么需要 AQS，不要长篇大论，容易把自己和面试官绕进去。  
</font><font style="color:rgba(0, 0, 0, 0.88);">就这样简要的回答：</font>

<font style="color:rgba(0, 0, 0, 0.88);">简单来说 AQS 就是起到了一个抽象、封装的作用，将一些排队、入队、加锁、中断等方法提供出来，便于其他相关 JUC 锁的使用，具体加锁时机、入队时机等都需要实现类自己控制。</font>

<font style="color:rgba(0, 0, 0, 0.88);">它主要通过维护一个共享状态（state）和一个先进先出（FIFO）的等待队列，来管理线程对共享资源的访问。</font>

<font style="color:rgba(0, 0, 0, 0.88);">state 用 volatile 修饰，表示当前资源的状态。例如，在独占锁中，state 为 0 表示未被占用，为 1 表示已被占用。</font>

<font style="color:rgba(0, 0, 0, 0.88);">当线程尝试获取资源失败时，会被加入到 AQS 的等待队列中。这个队列是一个变体的 CLH 队列，采用双向链表结构，节点包含线程的引用、等待状态以及前驱和后继节点的指针。</font>

<font style="color:rgba(0, 0, 0, 0.88);">AQS 常见的实现类有 ReentrantLock、CountDownLatch、Semaphore 等等。</font>

<font style="color:rgba(0, 0, 0, 0.88);">然后面试官会引申问你具体 ReentrantLock 的实现原理是怎样的呢？</font>

## <font style="color:rgba(0, 0, 0, 0.88);">Synchronized 和 ReentrantLock 有什么区别？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Synchronized 是 Java 内置的关键字，实现基本的同步机制，不支持超时，非公平，不可中断，不支持多条件。  
</font><font style="color:rgba(0, 0, 0, 0.88);">ReentrantLock 是 JUC 类库提供的，由 JDK 1.5 引入，支持设置超时时，可避免死锁，比较灵活，且支持公平锁，可中断，支持多条件判断。  
</font><font style="color:rgba(0, 0, 0, 0.88);">ReentrantLock 需要手动解锁，而 Synchronized 不需要，它们都是可重入锁。  
</font><font style="color:rgba(0, 0, 0, 0.88);">一般情况下用 Synchronized 足矣，比较简单，而 ReentrantLock 比较灵活，支持的功能比较多，所以复杂的情况用 ReentrantLock 。  
</font><font style="color:rgba(0, 0, 0, 0.88);">性能问题：很多年前，Synchronized 性能不如 ReentrantLock，现在基本上性能是一致的。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">可重入锁</font>
<font style="color:rgba(0, 0, 0, 0.88);">重入锁指的是同一个线程在持有某个锁的时候，可以再次获取该锁而不会发生死锁。例如以下代码：outer 还需要调用 inner，它们都用到了同一把锁，如果不可重入那么就会导致死锁。</font>

```java
public class ReentrantLockExample {
    private final ReentrantLock lock = new ReentrantLock();

    public void outer() {
        lock.lock();
        try {
            inner();
        } finally {
            lock.unlock();
        }
    }

    public void inner() {
        lock.lock();
        try {
            // critical section
        } finally {
            lock.unlock();
        }
    }
}
```

<font style="color:rgba(0, 0, 0, 0.88);">在递归调用或循环调用上锁时，可重入这个特性就十分重要了。</font>

#### <font style="color:rgba(0, 0, 0, 0.88);">可重入锁实现方式</font>
<font style="color:rgba(0, 0, 0, 0.88);">一般可重入锁是通过计数的方式实现，例如维护一个计数器，当前线程抢到锁则+1，如果当前线程再次抢到锁则继续+1。如果当前线程释放锁之后，则计数器-1，当减到 0 则释放当前锁。</font>

#### <font style="color:rgba(0, 0, 0, 0.88);">扩展 Synchronized 性能优化</font>
<font style="color:rgba(0, 0, 0, 0.88);">Synchronized 在 JDK 1.6 之后进行了很多性能优化，主要包括以下几种：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">偏向锁：如果一个锁被同一个线程多次获得，JVM 会将该锁设置为偏向锁，以减少获取锁的代价。</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">轻量级锁：如果没有线程竞争，JVM 会将锁设置为轻量级锁，使用 CAS 操作代替互斥同步。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">锁粗化：JVM 会将一些短时间内连续的锁操作合并为一个锁操作，以减少锁操作的开销。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">锁消除：JVM 在 JIT 编译时会检测到一些没有竞争的锁，并将这些锁去掉，以减少同步的开销。</font>

## <font style="color:rgba(0, 0, 0, 0.88);">Java 中 volatile 关键字的作用是什么？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">volatile 它的主要作用是保证变量的可见性和禁止指令重排优化。  
</font><font style="color:rgba(0, 0, 0, 0.88);">1）可见性（Visibility）：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">volatile 关键字确保变量的可见性。当一个线程修改了 volatile 变量的值，新值会立即被刷新到主内存中，其他线程在读取该变量时可以立即获得最新的值。这样可以避免线程间由于缓存一致性问题导致的“看见”旧值的现象。  
</font><font style="color:rgba(0, 0, 0, 0.88);">2）禁止指令重排序（Ordering）：</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">volatile 还通过内存屏障来禁止特定情况下的指令重排序，从而保证程序的执行顺序符合预期。对 volatile 变量的写操作会在其前面插入一个 StoreStore 屏障，而对 volatile 变量的读操作则会在其后插入一个 LoadLoad 屏障。这确保了在多线程环境下，某些代码块执行顺序的可预测性。</font>

