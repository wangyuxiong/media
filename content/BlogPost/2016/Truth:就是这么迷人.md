
### 前言
使用junit框架为基础，写单元测试，集成测试等等。都有可能写下这么一段代码:

```
Set<Foo> foo = ...;
assertTrue(foo.isEmpty()); // or, shudder, foo.size() == 0
```
没有什么复杂的逻辑，无非就是验证foo是否为空得一个断言。这个时候，大家会看到下面这段异常信息，莫名其妙：

```
java.lang.AssertionError
	at org.junit.Assert.fail(Assert.java:92)
	at org.junit.Assert.assertTrue(Assert.java:43)
	…
```
尤其，在每日daily run完成，通过平台只能查看到上面的这段异常日志，很难帮助我们看出问题的所在。最终还是不得不打开IDEA(Eclipse)，重新跑下这个case，找到异常的原因。这种场景是不是很常见？

￼假设，有一种断言异常，很清晰的给出它产生异常的原因，是不是会很方便。如果上面的这段异常信息抛出来得形式如下：

```
org.truth0.FailureStrategy$ThrowableAssertionError: Not true that  is empty
	at org.truth0.FailureStrategy.fail(FailureStrategy.java:33)
	...
```
准确得说出这个异常的原因，是不是感觉会好一些？

### Truth介绍:
废话不多说，我今天要推荐的一个junit扩展，就是要干这个事情。它的目标就是让我们的assertion和failure message具备可读性。举个栗子：

```
ImmutableSet<String> colors = ImmutableSet.of("red", "green", "blue", "yellow");
assertTrue(colors.contains("orange"));  // JUnit
assertThat(colors).contains("orange");  // Truth
```
大家可以执行起来看看，junit给出的肯定非人类可读的。但是Truth就不一样，给出了如此完美的输出：

```
AssertionError: <[red, green, blue, yellow]> should have contained <orange>
```
谁还敢说看不懂错在哪？

### Truth的使用:
truth的使用非常的简单，只需要引入相应的依赖即可。如果不是maven工程，自行下载导入也是可以的。


```
<dependency>
	<groupId>com.google.truth</groupId>
	<artifactId>truth</artifactId>
	<version>0.25</version>
</dependency>
```
有些公司可能会有自己得maven仓库，导致无法下载这个包。没关系，在测试工程的pom文件中加上这个所在的maven仓库也是可以的。

```
<repositories>
		<repository>
			<id>mvn-central</id>
			<name>maven central</name>
			<url>http://repo1.maven.org/maven2/</url>
			<snapshots>
			</snapshots>
			<releases>
			</releases>
		</repository>
</repositories>
<pluginRepositories>
		<pluginRepository>
			<id>mvn_plugin</id>
			<name>maven plugin mirror</name>
			<url>http://repo1.maven.org/maven2/</url>
			<snapshots>
			</snapshots>
			<releases>
			</releases>
		</pluginRepository>
</pluginRepositories>
```

### 基本的用途：
我用到的功能不多，仅仅是在cdp项目的测试中做了一次迁移。将所有的断言修改了。主要包含几个方面：

- 字符串包含 (contains)
- 字符串的比较 (=)
- 数值的比较 （>,=,<）
- Map对象的包含 (containKey)
- null

使用起来也比较简单，需要import如下3个静态类：

```
import static com.google.common.truth.Truth.assertThat;
import static com.google.common.truth.Truth.assert_;
import static com.google.common.truth.TruthJUnit.assume;
```
#### 基本使用:

```
assertThat(someInt).isEqualTo(5);
assertThat(aString).isEqualTo("Blah Foo");
assertThat(aString).contains("lah");
assertThat(foo).isNotNull();
```

####Collections and Maps:


```
assertThat(someCollection).contains("a");                       // contains this item
assertThat(someCollection).containsAllOf("a", "b").inOrder();   // contains items in the given order
assertThat(someCollection).containsExactly("a", "b", "c", "d"); // contains all and only these items
assertThat(someCollection).containsNoneOf("q", "r", "s");       // contains none of these items
assertThat(aMap).containsKey("foo");                            // has a key
assertThat(aMap).containsEntry("foo", "bar");                   // has a key, with given value
assertThat(aMap).doesNotContainEntry("foo", "Bar");             // does not have the given entry
```

这些是经过了基本的封装，满足了大家的日常需求。如果还有其它特别的需求，truth提供了一系列的扩展。我们可以先看这下面这2个表达式的关系：

```
assertThat(someInt).isEqualTo(5)；      //  assert1
assert_().that(someInt).isEqualTo(5);   //  assert2
```

很明显，2者做的是一样的事情。这里assert1最终是会转换成assert2的风格。所以，truth可以由用户自定义来扩展自己得assert。

#### new types:
我们可以创建一些”Subjects”来定义自己的类型，按下面的方式去使用：


```
// customType() returns an adapter (SubjectFactory).
assert_().about(customType()).that(theObject).hasSomeComplexProperty(); // specialized assertion
assert_().about(customType()).that(theObject).isEqualTo(anotherObject); // overridden equality
```

### 后话
truth初接触是从google test blog上看到的。Testing on the Toilet最新的一个专题介绍的就是这个，目前项目已经开源，可以去github官网上看看项目的介绍。很简单但很使用的一个东西。（第一个链接需要翻墙）

- Truth–a fluent assertion framework: [http://googletesting.blogspot.com/2014/12/testing-on-toilet-truth-fluent.html](http://googletesting.blogspot.com/2014/12/testing-on-toilet-truth-fluent.html)
- Truth–Basic Usage: [http://google.github.io/truth/usage/#how-does-truth-work](http://google.github.io/truth/usage/#how-does-truth-work)
