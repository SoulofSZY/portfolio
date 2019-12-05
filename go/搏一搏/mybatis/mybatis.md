## mybatis3
1. 基本代码 扩展点
2. 整合spring 扩展点
3. mapper生成
4. tk-mybatis/pagehelper
5. mybatis-plus


### 基本概念
1. DataSource  pool
2. TransactionFactory
3. SqlSessionFactoryBuild
4. SqlSessionFactory
5. SqlSession
6. Mapper 
7. xml/annotation
8. 本地缓存和二级缓存

**作用域&生命周期**

SqlSessionFactoryBuild：
可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的XML解析资源可以被释放给更重要的事情

SqlSessionFactory：
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式

SqlSession：
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中

映射器实例：
映射器是一些由你创建的、绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。因此从技术层面讲，任何映射器实例的最大作用域是和请求它们的 SqlSession 相同的。尽管如此，映射器实例的最佳作用域是方法作用域。 也就是说，映射器实例应该在调用它们的方法中被请求，用过之后即可丢弃。 并不需要显式地关闭映射器实例，尽管在整个请求作用域保持映射器实例也不会有什么问题，但是你很快会发现，像 SqlSession 一样，在这个作用域上管理太多的资源的话会难于控制。 为了避免这种复杂性，最好把映射器放在方法作用域内

**xml配置**

properties 属性
> 通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性

settings 设置
>MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为

typeAliases 类型别名
>类型别名是为 Java 类型设置一个短的名字。 它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余

typeHandlers 类型处理器
>扩展点1  重写类型处理器或创建自定义类型处理器来处理不支持的或非标准的类型【TypeHandler/BaseTypeHandler】
>xml 优先于 注解

处理枚举类型
>扩展点2 针对枚举类型的处理  比较单一

objectFactory 对象工厂
>扩展点3 结果对象新实例创建 

plugins 插件
>扩展点4 主要的扩展点 已映射语句执行过程中的某一点进行拦截调用

environments 环境
> 数据源 事务

databaseIdProvider 数据库厂商标识
> 辅助功能 简短数据库标识

mappers 映射器
> 实际业务开发  sql

    
