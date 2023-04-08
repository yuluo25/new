---
share: true  
---
# java
@Transient
可以在PO类中忽略一个属性，不在数据库中创建列
## java类加载过程

推荐参考[视频](https://www.bilibili.com/video/BV1g84y1F7df?p=11&vd_source=2604b9f6f798a76f359884d0e22c4226)

![[Pasted image 20220902195137.png]]

## 集合

### 两个list集合合并成一个list集合
[参考](https://blog.csdn.net/weixin_41552767/article/details/107662821)
现在有以下一个场景：需要将集合	
	A：{"id": "12345","name": "zhangsan"}
	B：{"id": "12345", "age": 23}
		合并成一个新的集合
	C：{"id": "12345","name": "zhangsan", "age": 23}
```java
	1、将listA集合转换为map
	 Map<String, Person> map = listA.stream().collect
                (Collectors.toMap(Person ::getId, Person -> Person))
       
    2、合并数据,这里将listA集合的数据合并到listB集合上
	listB.forEach(n -> {
         if(map.containsKey(n.getId())){
         Person person = map.get(n.getId());
         n.setName(person.getName());
         }
     });
```
这里经过第二步处理好的listB就已经是我们想要的listA啦！
```java
	3、如果主键重复了，还可以使用以下方式去重
	
	Map<String, Person> person = listB.stream()
                .collect(Collectors.toMap(Person::getId, Function.identity(), (key1, key2) -> key2));
```


###  list转map 如果重复key的value转为list写法
[参考](https://blog.51cto.com/u_13666149/4715413)
![[Pasted image 20220811143149.png]]


### **Stream流的使用**
---

#### Stream流的中间操作
##### filter 过滤

##### map 映射

##### sorted 排序

##### limit、skip 限制

##### distinct 去重

#### Stream流的最终操作

##### anyMatch,allMatch,noneMatch 匹配

##### reduce 组合

##### count,max,min 计数

##### forEach 迭代

##### collect 汇总

#### **Collectors**
##### Collectors.groupingBy() 详解
1. 分组 计数 排序
```java
//3 apple, 2 banana, others 1
List<String> items =
		Arrays.asList("apple", "apple", "banana",
				"apple", "orange", "banana", "papaya");

Map<String, Long> result =
		items.stream().collect(
				Collectors.groupingBy(
						Function.identity(), Collectors.counting()
				)
		);

System.out.println(result);


//或者 根据PersonId分组
Collectors.groupingBy(PersonPO::getPersonId)
	
```
##### Collectors.mapping 


###  通过Stream 对List，Map操作和互转
1、Map数据转换为自定义对象的List，例如把map的key,value分别对应Person对象两个属性：
```java
List<Person> list = map.entrySet().stream().sorted(Comparator.comparing(e -> e.getKey()))
		.map(e -> new Person(e.getKey(), e.getValue())).collect(Collectors.toList());
```
```java
List<Person> list = map.entrySet().stream().sorted(Comparator.comparing(Map.Entry::getValue))
		.map(e -> new Person(e.getKey(), e.getValue())).collect(Collectors.toList());
```
```java
List<Person> list = map.entrySet().stream().sorted(Map.Entry.comparingByKey())
	.map(e -> new Person(e.getKey(), e.getValue())).collect(Collectors.toList());
```
以上三种方式不同之处在于排序的处理。

2、List对象转换为其他List对象：
```java
 List<Employee> employees = persons.stream()
                .filter(p -> p.getLastName().equals("l1"))
                .map(p -> new Employee(p.getName(), p.getLastName(), 1000))
                .collect(Collectors.toList());
```

3、从List中过滤出一个元素
```java
 User match = users.stream().filter((user) -> user.getId() == 1).findAny().get();
```

4、List转换为Map
```java
public class Hosting {
    private int Id;
    private String name;
    private long websites;
    public Hosting(int id, String name, long websites) {
        Id = id;
        this.name = name;
        this.websites = websites;
    }
    //getters, setters and toString()
}
```

```java
 Map<Integer, String> result1 = list.stream().collect(
                Collectors.toMap(Hosting::getId, Hosting::getName));
```
[参考](https://blog.csdn.net/hgc0907/article/details/80756730)

## 常用api
### java.time

#### 为什么建议使用你LocalDateTime，而不是Date？


## Spring Jpa
### 使用Jpa 选择特定的列
例如：  `SELECT projectId, projectName FROM projects`
实现
#### 方法一
第一步 创建相应的接口
```java
interface ProjectIdAndName{
    String getId();
    String getName();
}
```
第二部 在repository中加入如下方法
```java
List<ProjectIdAndName> findAll();
```

> JPA不寻找特定字段背后的想法是，从表中的一行带来一列或所有列的成本（效率上）是一样的。  
> 上句，对于比较简单的、原始的数据类型来说，这可能不是什么大问题。但是有一个BLOB列会将BLOB加载到内存中从而导致加载速度缓慢

#### 方法二
来自[Spring Doc](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#projections)
```java
 // A sample aggregate and repository
class Person {

  @Id UUID id;
  String firstname, lastname;
  Address address;

  static class Address {
    String zipCode, city, street;
  }
}

interface PersonRepository extends Repository<Person, UUID> {

  Collection<Person> findByLastname(String lastname);
}
```


## 动态代理
###  为什么spring的事务注解@Transaction只能用在public方法上

今天在看spring事务的时候，发现特意有强调@Transaction注解是只能用在public方法上的，但没有说明原因，于是引起了我的好奇心。
在经过我的思考和查阅相关博文确认了原因。

首先，@Transaction一般用在方法上，它只能用在public方法上，那就意味着其不能用在private方法上。于是就可以想一下为什么private方法不可以用用呢？

这时候把思路放到AOP上，spring中很多东西的实现都是依靠AOP，本质上也是依靠代理来实现。事务在spring中的实现其实就是生成bean对象的代理对象。在bean进行创建出实话时， 如果是有事务注解的方法，就会被进行增强，最终形成代理类。

在spring中，有两种动态代理的方式，一种是jdk，它是将原始对象放入代理对象内部，通过调用内含的原始对象来实现原始的业务逻辑，这是一种装饰器模式；而另一种是cglib，它是通过生成原始对象的子类，子类复写父类的方法，从而实现对父类的增强。

jdk中，如果是private的方法，显然是无法访问的，而在cglib中，也是同样，private方法也无法访问。于是原始对象的业务逻辑就丢失了，那么怎么进行增强呢？

于是这个问题实际在于，被aop增强的方法都应该是public的，而不能是private的

## 建议
### 使用流式编程
举个例子，假如你要随机展示 5 至 20 之间不重复的整数并进行排序。实际上，你的关注点首先是创建一个有序集合。围绕这个集合进行后续的操作。但是使用流式编程，你就可以简单陈述你想做什么：

```
// streams/Randoms.java
import java.util.*;
public class Randoms {
    public static void main(String[] args) {
        new Random(47)
            .ints(5, 20)
            .distinct()
            .limit(7)
            .sorted()
            .forEach(System.out::println);
    }
}
```

输出结果：

```
6
10
13
16
17
18
19
```

---

而声明式编程（Declarative programming）是一种：声明要做什么，而非怎么做的编程风格。正如我们在函数式编程中所看到的。**注意**，命令式编程的形式更难以理解。代码示例：

```
// streams/ImperativeRandoms.java
import java.util.*;
public class ImperativeRandoms {
    public static void main(String[] args) {
        Random rand = new Random(47);
        SortedSet<Integer> rints = new TreeSet<>();
        while(rints.size() < 7) {
            int r = rand.nextInt(20);
            if(r < 5) continue;
            rints.add(r);
        }
        System.out.println(rints);
    }
}
```

输出结果：

```
[7, 8, 9, 11, 13, 15, 18]
```

在 `Randoms.java` 中，我们无需定义任何变量，但在这里我们定义了 3 个变量： `rand`，`rints` 和 `r`。由于 `nextInt()` 方法没有下限的原因（其内置的下限永远为 0），这段代码实现起来更复杂。所以我们要生成额外的值来过滤小于 5 的结果。

# UML
![[Pasted image 20220903153142.png]]