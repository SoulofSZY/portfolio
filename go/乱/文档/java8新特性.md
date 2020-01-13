##java8新特性整理

 - lambda表达式
 - Stream 流式操作
 - Optionals 取代null
 - CompletableFuture 多线程处理
 - 新的日期和时间API
 - 其他

### lambda相关
1. 匿名类的一种取代
2. 单一抽象接口的实现
3. 注意和抽象接口函数签名的匹配
4. 语法糖
#### 接口默认方法
这个特性有些颠覆以前的认知，接口可以有实现了。其实是对函数式接口的迎合，函数式接口：只有单一抽象接口

```java
public interface ForMula {
    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```
#### 方法引用（::）
语法糖，对lambda的进一步简化，进一步增强代码复用，这里的方法可以是构造方法和成员方法 也可以是类方法
```java
    @Test
    public void testMethodReference() {
        Supplier<Person> supplier = Person::new;
        Person person = supplier.get();
        person.setName("szy");
        person.setAge(18);
        System.out.println(person.toString());
    }

    @Test
    public void testMethodReference2() {
        Consumer<Person> consumer = System.out::println;
        Person person = new Person();
        person.setName("szy");
        person.setAge(18);
        consumer.accept(person);
        consumer = consumer.andThen(Person::printName);
        consumer.accept(person);
    }
```
#### java8提供的一些函数式接口 java.util.function.*
java8 内建了一些常用新式的函数式接口 使用时可以参考


### java8中的一些多线程处理
jdk1.0 Thread/Runnable

jdk1.5 Excutors Callable/Future locks countdowmlatch

jdk1.7 ForkJoinPool

jdk1.8 CompletableFuture