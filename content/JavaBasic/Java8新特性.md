# Lambda 表达式

```rust
 (parameters) -> expression。(parameters) ->{ statements; }
```

# 方法引用

**方法引用**是用来直接访问类或者实例的已经存在的方法或者构造方法。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。

当Lambda表达式中只是执行一个方法调用时，不用Lambda表达式，直接通过方法引用的形式可读性更高一些。方法引用是一种更简洁易懂的Lambda表达式。

注意方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。

# Stream API详解

sorted()对流进行排序。distinct()去重。filter：对Stream中元素进行过滤

# Optional

为了解决空指针异常，在Java8之前需要使用if-else这样的语句去防止空指针异常，而在Java8就可以使用Optional来解决。Optional可以理解成一个数据容器，甚至可以封装null，并且如果值存在调用isPresent()方法会返回true

```java
private String getUserName(User user) {
    Optional<User> userOptional = Optional.ofNullable(user);
    return userOptional.map(User::getUserName).orElse(null);
}
```

# Date/time API的改进

在Java8之前的版本中，日期时间API存在很多的问题，比如：

- 线程安全问题：java.util.Date是非线程安全的，所有的日期类都是可变的；
- 设计很差：在java.util和java.sql的包中都有日期类，此外，用于格式化和解析的类在java.text包中也有定义。而每个包将其合并在一起，也是不合理的；
- 时区处理麻烦：日期类不提供国际化，没有时区支持，因此Java中引入了java.util.Calendar和Java.util.TimeZone类；

LocalDate:一个带有年份，月份和天数的日期，使用静态方法now或者of方法进行创建；

LocalTime:表示一天中的某个时间，同样可以使用now和of进行创建；

LocalDateTime：兼有日期和时间；

。