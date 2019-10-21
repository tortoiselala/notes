# Spring事务管理

Spring事务管理高层抽象主要包括3个接口：

- `PlatformTransactionManager` 事务管理器
- `TransactionDefinition` 事务定义信息
- `TransactionStatus` 事务具体运行状态

## `PlatformTranstractionManager`

该接口为事务管理器，拥有三个方法，分别是：

- `getTransaction`
- `commit`
- `rollback`

可以看出这个接口仅仅是定义了事务的完成或回退。对于该接口，提供了一个抽象类`AbstractPlatformTransactionManager`的实现，该类不仅实现`PlatformTranstractionManager`的三个方法，还提供了事务生命周期中用到的其他方法。

`AbstractPlatformTransactionManager`提供了以下事务生命周期的掌控：

- 判断是否存在事务
- 应用适当的传播行为
- 在必要时中止或恢复事务
- 在`commit`时检查`rollback-only`标志
- 在`rollback`时应用适当的修改
- 触发注册同步`rollback`

其他具体的事务管理器需要继承该抽象类。

## `TransactionDefinition`

该接口基于EJB CMT属性定义了事务传播行为和事务隔离级别，这些属性将在Spring事务中用到。

### 事务传播行为

在该接口中定义的事务传播行为的属性有以下几个：

- `PROPAGATION_REQUIRED`   支持当前事务，如果当前没有事务，就新建一个事务 ，这是默认的选择。
- `PROPAGATION_SUPPORTS`  支持当前事务，如果当前没有事务，就以非事务方式执行。 
- `PROPAGATION_MANDATORY`  支持当前事务，如果当前没有事务，就抛出异常。  
- `PROPAGATION_REQUIRES_NEW`  新建事务，如果当前存在事务，把当前事务挂起。  
- `PROPAGATION_NOT_SUPPORTED`  以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  
- `PROPAGATION_NEVER`  以非事务方式执行，如果当前存在事务，则抛出异常。 
- `PROPAGATION_NESTED` 

这几个属性的值从0开始编号到6。

#### `PROPAGATION_REQUIRED`   

含义：如果当前正要执行的事务不在另一个事务中，则新建一个事务。

定义如下处理过程：

````java
ServiceA{
    void methond() {
        ServiceB.mothod();
    }
}

ServiceB{
    void method() {
    }
}
````

如果`ServiceB.method`定义事务传播行为为`PROPAGATION_REQUIRED`，那么如果执行`ServiceA.method`时已经处于事务执行当中，那么`ServiceB.method`将不再新建事务。而`ServiceA.method`没有处于事务执行当中，则`ServiceB.method`将新建一个事务执行。

在这种形况下，当事务需要回滚时，如果`ServiceA.method`的事务执行失败，即使`ServiceB.method`已经提交，依然要回滚。

#### `PROPAGATION_SUPPORTS`  

含义： 如果当前在事务中，即以事务的形式运行，如果当前不再一个事务中，那么就以非事务的形式运行 

#### `PROPAGATION_MANDATORY` 

含义：支持当前事务，如果当前没有事务，就抛出异常。  

#### `PROPAGATION_REQUIRES_NEW`  

含义：新建事务，如果当前存在事务，把当前事务挂起。  

#### `PROPAGATION_NOT_SUPPORTED` 

含义：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  

#### `PROPAGATION_NOT_SUPPORTED` 

含义：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  

#### `PROPAGATION_NESTED` 

理解`nested`的关键是`savepoint`。他与`PROPAGATION_REQUIRES_NEW`的区别是，`PROPAGATION_REQUIRES_NEW`另起一个事务，将会与他的父事务相互独立，而`nested`的事务和他的父事务是相依的，他的提交是要等和他的父事务一块提交的。也就是说，如果父事务最后回滚，他也要回滚的。

而`nested`事务的好处是他有一个`savepoint`。

### 事务隔离级别

该接口定义的事务隔离级别有以下几个：

- `ISOLATION_DEFAULT`
- `ISOLATION_READ_UNCOMMITTED`
- `ISOLATION_READ_COMMITTED`
- `ISOLATION_REPEATABLE_READ`
- `ISOLATION_SERIALIZABLE`

隔离级别除了`ISOLATION_DEFAULT`为`-1`外，其余均直接使用`java.sql.Connection`中定义的隔离级别。值得一提的是其余四个隔离级别值分别是`0 1 2 4 8`

## `TransactionStatus`

定义事务状态操作的相关接口

# Spring事务编程方法

Spring支持两种事务管理

- 编程式的事务管理
  - 通过`TransactionTemplate`手动管理
- 使用XML配置声明式事务
  - 通过Spring AOP实现

