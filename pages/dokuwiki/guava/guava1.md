title: guava1 

#  Guava中的函数式编程讲解 
**Guava 还引入了函数式编程的概念**，在一定程度上缓解了java在JDK1.8之前没有lambda的缺陷，使使用java书写简洁易读的函数式风格的代码成为可能。
下面就简单的介绍下 Guava 中的一些体现了函数式编程的API。
```

import com.google.common.base.Function;
import com.google.common.base.Predicate;

import static com.google.common.base.Predicates.and;
import static com.google.common.collect.FluentIterable.from;
import static com.google.common.collect.Iterables.filter;
import static com.google.common.collect.Iterables.transform;
import static com.google.common.collect.Lists.newArrayList;


```
  * Function接口：这说明在java编程当中可以引入函数式编程。同时也说明了如何使用Function接口以及最好的使用方式。
  * Functions类：Functions类包含一些实用的方法来操作Fucntion接口的实例。
  * Predicate接口：这个接口是评估一个对象是否满足一定条件，如果满足则返回true。
  * Predicates类：这个类是对于Predicate接口的指南类，它实现了Predicate接口并且非常实用的静态方法。
  * Supplier接口：这个接口可以提供一个对象通过给定的类型。我们也可以看到通过各种各样的方式来创建对象。
  * Suppliers类：这个类是Suppliers接口的默认实现类。

##  Filter 
**filter即从一个集合中根据一个条件筛选元素**
我们先创建一个简单的Person类。
Person.java
```

public class Person {
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

```
如果要产生一个Person类的List，通常的写法可能是这样子。
```

List<Person> people = new ArrayList<Person>();
        people.add(new Person("bowen",27));
        people.add(new Person("bob", 20));
        people.add(new Person("Katy", 18));
        people.add(new Person("Logon", 24));

```
###  newArrayList辅助方法 
而 ` Guava ` 提供了一个` newArrayList `的方法，其自带类型推演，并可以**方便的生成一个List**,并且通过参数传递初始化值。
```

List<Person> people = newArrayList(new Person("bowen", 27),
                new Person("bob", 20),
                new Person("Katy", 18),
                new Person("Logon", 24));

```
当然，这不算函数式编程的范畴，这是 Guava 给我们提供的一个实用的函数。
如果我们选取其中年龄大于20的人，通常的写法可能是这样子。
```

List<Person> oldPeople = new ArrayList<Person>();
        for (Person person : people) {
            if (person.getAge() >= 20) {
                oldPeople.add(person);
            }
        }

```
###  filter方法与Predicate函数接口 
这就是典型的filter模式。**filter即从一个集合中根据一个条件筛选元素**。其中person.getAge() >=20就是这个条件。 Guava 为这种模式提供了一个` filter的方法 `。所以我们可以这样写。
```

List<Person> oldPeople = newArrayList(filter(people, new Predicate<Person>() {
            public boolean apply(Person person) {
                return person.getAge() >= 20;
            }
        }));

```
这里的` Predicate `是 Guava 中的一个接口，我们来看看它的定义。
Predicate.java
```

@GwtCompatible
public interface Predicate<T> {
  boolean apply(@Nullable T input);
  @Override
  boolean equals(@Nullable Object object);
}

```
里面只有一个` apply方法 `，接收一个泛型的实参，返回一个boolean值。**由于java世界中函数并不是一等公民，所以我们无法直接传递一个条件函数，只能通过Predicate这个类包装一下。**
###  And Predicate 
如果要再实现一个方法来查找People列表中所有名字中包含b字母的列表，我们可以用 Guava 简单的实现。
```

List<Person> namedPeople = newArrayList(filter(people, new Predicate<Person>() {
            public boolean apply(Person person) {
                return person.getName().contains("b");
            }
        }));

```
一切是这么的简单。 那么新需求来了，如果现在需要找年龄>=20并且名称包含b的人，该如何实现那？ 可能你会这样写。
```

List<Person> filteredPeople = newArrayList(filter(people, new Predicate<Person>() {
            public boolean apply(Person person) {
                return person.getName().contains("b") && person.getAge() >= 20;
            }
        }));

```
这样写的话就有一定的代码重复，因为之前我们已经写了两个Predicate来分别实现这两个条件判断，能不能重用之前的Predicate那？答案是能。 我们首先将之前生成年龄判断和名称判断的两个Predicate抽成方法。
```

private Predicate<Person> ageBiggerThan(final int age) {
        return new Predicate<Person>() {
            public boolean apply(Person person) {
                return person.getAge() >= age;
            }
        };
    }

private Predicate<Person> nameContains(final String str) {
        return new Predicate<Person>() {
            public boolean apply(Person person) {
                return person.getName().contains(str);
            }
        };
    }

```
而我们的结果其实就是这两个Predicate相与。 Guava 给我们提供了` and方法 `，用于对一组Predicate求与。
```

List<Person> filteredPeople = newArrayList(filter(people, and(ageBiggerThan(20), nameContains("b"))));

```
**由于and接收一组Predicate，返回也是一个Predicate，所以可以直接作为filter的第二个参数。**如果不熟悉函数式编程的人可能感觉有点怪异，但是习惯了就会觉得它的强大与简洁。 
**当然除了and， Guava 还为我们提供了or，用于对一组Predicate求或**。这里就不多讲了，大家可以自己练习下。
##  Map(transform方法与Function函数接口) 
列表操作还有另一个常见的模式，就是**将数组中的所有元素映射为另一种元素的列表，这就是map pattern**。举个例子，求People列表中的所有人名。程序员十有八九都会这样写。
```

List<String> names = new ArrayList<String>();
        for (Person person : people) {
            names.add(person.getName());
        }

```
Guava 已经给我们提供了这种Pattern的结果办法，那就是使用` transform方法 `。
```

List<String> names = newArrayList(transform(people, new Function<Person, String>() {
            public String apply( Person person) {
                return person.getName();
            }
        }));

```
**Function是另外一种用于封装函数的接口对象。**它的定义如下:
Function.java
@GwtCompatible
```

public interface Function<F, T> {
  @Nullable T apply(@Nullable F input);
  @Override
  boolean equals(@Nullable Object object);
}

```
它与Predicate非常相似，但不同的是它接收两个泛型，apply方法接收一种泛型实参，返回值是另一种泛型值。正是这个apply方法定义了数组间元素一对一的map规则。

##  reduce 
**除了filter与map模式外，列表操作还有一种reduce操作。比如求people列表中所有人年龄的和。 Guava 并未提供reduce方法。**具体原因我们并不清楚。但是我们可以自己简单的实现一个reduce pattern。 先定义一个Func的接口。
Func.java
```

public interface Func<F,T> {

         T apply(F currentElement, T origin);

     }

```
apply方法的第一个参数为列表中的当前元素，第二个参数为默认值，返回值类型为默认值类型。 然后我们定义个reduce的静态方法。
Reduce.java
```

public class Reduce {
    private Reduce() {

    }

    public static <F,T> T reduce(final Iterable<F> iterable, final Func<F, T> func, T origin) {

        for (Iterator iterator = iterable.iterator(); iterator.hasNext(); ) {
            origin = func.apply((F)(iterator.next()), origin);
        }

        return origin;
    }
}

```
reduce方法接收三个参数，第一个是需要进行reduce操作的列表，第二个是封装reduce操作的Func，第三个参数是初始值。

我们可以使用这个reduce来实现求people列表中所有人的年龄之和。

```

Integer ages = Reduce.reduce(people, new Func<Person, Integer>() {

            public Integer apply(Person person, Integer origin) {
                return person.getAge() + origin;
            }
        }, 0);

```
我们也可以轻松的写一个方法来得到年龄的最大值。

```

Integer maxAge = Reduce.reduce(people, new Func<Person, Integer>() {

            public Integer apply(Person person, Integer origin) {
                return person.getAge() > origin ? person.getAge() : origin;
            }
        }, 0);

```
##  Fluent pattern(流式API) 
现在新需求来了，需要找出年龄>=20岁的人的所有名称。该如何操作那？我们可以使用filter过滤出年龄>=20的人，然后使用transform得到剩下的所有人的人名。
```

private Function<Person, String> getName() {
        return new Function<Person, String>() {
            public String apply( Person person) {
                return person.getName();
            }
        };
    }
    public void getPeopleNamesByAge() {
        List<String> names = newArrayList(transform(filter(people, ageBiggerThan(20)), getName()));
    }

```
这样括号套括号的着实不好看。能不能改进一下那？ Guava 为我们提供了fluent模式的API,我们可以这样来写。
```

List<String> names = from(people).filter(ageBiggerThan(20)).transform(getName()).toList();

```
Guava 中还有很多好玩的东西，大家时间可以多发掘发掘。这篇文章的源码已经被我放置到 [github](https://github.com/huangbowen521/SpringMessageSpike/tree/master/src/main/java/com/thoughtworks/guava) 中，感兴趣的可以自行查看。


##  API与工具类使用说明 
###  使用Function接口 
函数式编程强调使用函数，以实现其目标与不断变化的状态。这与大多数开发者熟悉的改变状态的编程方式形成对比。Function接口让我们在java代码当中引入函数式编程成为可能。
Function接口当中只有2个方法：
```

public interface Function<F,T> {
　　T apply(F input);
　　boolean equals(Object object);
}

```
apply方法接受一个参数并且返回一个对象。一个好的功能实现应该没有副作用，这意味着当一个对象传入到apply方法当中后应该是保持不变的。下面是一个接受Date对象并且返回Date格式化后字符串的例子：
```

public class DateFormatFunction implements Function<Date,String> {
　　@Override
　　public String apply(Date input) {
　　　　SimpleDateFormat dateFormat = new SimpleDateFormat("dd/mm/yyyy");
　　　　return dateFormat.format(input);
　　}
}

```
###  使用Functions类 
Functions类包含一些实用的方法来操作Fucntion接口的实例。在本节当中，我们将会讲其中的两个方法。
**使用Functions.forMap方法**
forMap方法接受一个Map<K,V>的参数并且返回一个Function<K,V>实例，执行apply方法会在map当中进行查找。例如，考虑下面的类代表美国的一个州。
```

public class State {
　　private String name;
　　private String code;
　　private Set<City> mainCities = new HashSet<City>(); 
　　//省去getter和setter方法
}

```
现在你有一个名为stateMap的Map<String, State>,他的key就是州名的缩写。现在我们可以创建一个通过州代码来查找的函数，你只需要下面几步：
```

Function<String,State> lookup = Functions.forMap(stateMap);
//Would return State object for NewYork
lookup.apply("NY");

```
使用Functions.forMap方法有一个警告。如果传入的key在map当中不存在会抛出IllegalArgumentException异常。然而，有一个重载的forMap方法增加一个默认值参数，如果key没找到会返回默认值。通过使用Function接口来执行state的查找，你可以很容易的改变这个实现。当我们使用Splitter对象来创建一个map或者使用Guava collection包中其他的一些方法来创建map，总之我们可以在我们代码当中借住Guava的力量。

**使用Functions.compose方法**
假设你现在有一个代表city的类，代码如下：
```

public class City {
　　private String name;
　　private String zipCode;
　　private int population;
    //省去getter和setter方法
　　public String toString() {
　　　　return name;
　　}
}

```
考虑下面的情形：你要创建一个Function实例，传入State对象返回当中mainCities逗号分割的字符串。代码如下：
```

public class StateToCityString implements Function<State,String> {
　　@Override
　　public String apply(State input) {
　　　　return Joiner.on(",").join(input.getMainCities());
　　}
}

```
更进一步来说。你希望只有一个Function实例通过传入State的名称缩写来返回这State当中mainCities逗号分割的字符串。Guava提高了一个很好的方法来解决这种情况，Functions.compose方法接受两个Function实例作为参数，并且返回这两个Function组合后的Function。所以我们可以使用上面的两个Function来举一个例子：
```

Function<String,State> lookup = Functions.forMap(stateMap);
Function<State, String> stateFunction = new StateToCityString();
Function<String,String> composed = Functions.compose(stateFunction ,lookup);

```
现在调用composed.apply("NY")方法将会返回："Albany,Buffalo,NewYorkCity"
花一分钟时间来看下方法的调用顺序。composed接受一个“NY”参数并且调用lookup.apply()方法，从lookup.apply()方法中返回的值传入了stateFunction.apply()方法当中并且返回执行结果。可以理解为第二个参数的输入参数就是composed.apply方法的输入参数，第一个参数的输出就是composed.apply 方法的输出。**如果不使用composed 方法，在前面的例子将如下所示：** 
```

String cities = stateFunction.apply(lookup.apply("NY"));

```

###  使用Predicate接口 
Predicate接口与Function接口的功能相似。像Function接口一样，Predicate接口也有2个方法：
```

public interface Predicate<T> {
　　boolean apply(T input)
　　boolean equals(Object object)
}

```
apply方法会返回对输入断定后的结果。在Fucntion接口使用的地方来转换对象，**Predicate接口则用于过滤对象**。

Predicate接口的例子
这是Predicate接口的一个简单的例子，我们使用上面例子当中的City类。我们定义一个Predicate来判断这个城市是否有最小人口。
```

public class PopulationPredicate implements Predicate<City> {
　　@Override
　　public boolean apply(City input) {
　　　　return input.getPopulation() <= 500000;
　　}
}

```
在这个例子当中，我们简单的检查了City对象当中的人口属性，当人口属性小于等于500000的时候会返回true。通常，你可以将Predicate的实现定义成匿名类来过滤集合当中的每一个元素。Predicate接口和Function接口是如此的相似，许多情况下使用Fucntion接口的时候同样可以使用Predicate接口。

###  使用Predicates类 
Predicates类是包含一些实用的方法来操作Predicate接口的实例。Predicates类提供了一些非常有用的方法，从布尔值条件中得到期望的值，也可以使用“and”和“or”方法来连接不同的Predicate实例作为条件，并且如果提供“not”方法一个Predicate实例返回的值是false则“not”方法返回true，反之亦然。同样也有Predicates.compose方法，但是他接受一个Predicate实例和一个Function实例，并且返回Predicate执行后的值，把Function执行后的值当中它的参数。让我们看些例子，我们会更好的理解如何在代码中使用Predicates类。先看个特殊的例子，假设我们有下面两个类的实例。
```

public class TemperateClimatePredicate implements Predicate<City> {
　　@Override
　　public boolean apply(City input) {
　　　　return input.getClimate().equals(Climate.TEMPERATE);
　　}
}
public class LowRainfallPredicate implements Predicate<City> {
　　@Override
　　public boolean apply(City input) {
　　　　return input.getAverageRainfall() < 45.7;
　　}
}

```
值得重申，通常我们会使用匿名类，但为清楚起见，我们将使用具体类。 

**使用Predicates.and方法**
Predicates.and方法接受多个Predicate对象并且返回一个Predicate对象，因此调用返回的Predicate对象的apply方法当所有Predicate对象的apply方法都返回true的时候会返回true。如果其中一个Predicate对象返回false，其他的Predicate对象的执行就会停止。例如，假如我们只允许城市人口小于500,000并且年降雨量小于45.7英寸的。
```

Predicate smallAndDry = Predicates.and(smallPopulationPredicate, lowRainFallPredicate);

```
下面是Predicates.and方法的签名：
```

Predicates.and(Iterable<Predicate<T>> predicates);
Predicates.and(Predicate<T> ...predicates);

```

**使用Predicates.or方法**
Predicates.or方法接受多个Predicate对象并且返回一个Predicate对象,如果当中有一个Predicate对象的apply方法返回true则总方法就返回true。
如果有一个Predicate实例返回true，就不会继续执行。例如，假设我们想包含城市人口小于等于500,000或者有温带气候的城市。
```

Predicate smallTemperate = Predicates.or(smallPopulationPredicate, temperateClimatePredicate);

```
下面是Predicates.or方法的签名：
```

Predicates.or(Iterable<Predicate<T>> predicates);
Predicates.or(Predicate<T> ...predicates);

```

**使用Predicates.not方法**
Predicates.not接受一个Predicate实例并且执行这个Predicate实例的逻辑否的功能。加入我们想获得人口大于500,000的城市。使用这个方法可以代替重写一个Predicate实例：
```

Predicate largeCityPredicate = Predicates.not(smallPopulationPredicate);

```

**使用Predicates.compose方法**
Predicates.compose接受一个Function实例和一个Predicate实例作为参数，并且从Fucntion实例当中返回的对象传入到Predicate对象当中并且进行评估。在下面的例子当中，我们创建一个新的Predicate对象：
```

public class SouthwestOrMidwestRegionPredicate implements Predicate<State> { 
　　@Override
　　public boolean apply(State input) {
　　　　return input.getRegion().equals(Region.MIDWEST) ||
　　　　　　　　input.getRegion().equals(Region.SOUTHWEST);
　　}
}

```
下一步，我们将使用原来的Function实例lookup来创建一个State对象，并且使用上面例子的Predicate实例来评估下这个State是否在MIDWEST或者SOUTHWEST：
```

Predicate<String> predicate = Predicates.compose(southwestOrMidwestRegionPredicate,lookup);

```
###  使用Supplier接口 
Supplier接口只有一个方法如下：
```

public interface Supplier<T> {
　　T get();
}

```
get方法返回泛型T的实例。Supplier接口可以帮助我们实现几个典型的创建模式。**当get方法被调用，我们可以返回相同的实例或者每次调用都返回新的实例。Supplier也可以让你灵活选择是否当get方法调用的时候才创建实例**。并且Supplier是个接口，单元测试也会更简单，相对于其他方法创建的对象，如静态工厂方法。 总之，供应Supplier接口的强大之处在于它抽象的复杂性和对象如何需要创建的细节，让开发人员自由地在他觉得任何方式创建一个对象时最好的方法。让我们看看如何Supplier接口。

一个Supplier接口的例子
下面代码是Supplier接口的例子：
```

public class ComposedPredicateSupplier implements Supplier<Predicate<String>> {
　　@Override
　　public Predicate<String> get() {
　　　　City city = new City("Austin,TX","12345",250000, Climate.SUB_ TROPICAL, 45.3);
　　　　State state = new State("Texas","TX", Sets.newHashSet(city), Region.SOUTHWEST); 
　　　　City city1 = new City("New York,NY","12345",2000000,Climate.TEMPERATE, 48.7); 
　　　　State state1 = new State("New York","NY",Sets.newHashSet(city1), Region.NORTHEAST);
　　　　Map<String,State> stateMap = Maps.newHashMap();
　　　　stateMap.put(state.getCode(),state);
　　　　stateMap.put(state1.getCode(),state1);
　　　　Function<String,State> mf = Functions.forMap(stateMap);
　　　　return Predicates.compose(new RegionPredicate(), mf);
　　}
}

```
在这个例子当中，我们看到使用Functions.forMap关键一个Function实例可以通过State的缩写来查找State，和使用Predicate实例来评估在那些地方是否有这个State。然后将Function实例和Predicate实例作为参数传入到Predicates.compose方法当中并且返回期望的Predicate实例。我们使用了两个静态方法，Maps.newHashMap() 和Sets. newHashSet(),这两个都可以在Guava的包当中找到我们下一章会讲到。现在我们每次调用都会返回新的实例。我们也可以把创建Predicate实例的工作放到ComposedPredicateSupplier 的构造方法中来进行，当每次调用get的时候返回相同的实例。接着往下看，Guava提供了更简单的选择。

一个Suppliers类
正如我们所期望的Guava，有一个Suppliers类的静态方法来操作Supplier实例。在前面的例子，每次调用get方法都会返回一个新的实例。如果我们想改变我们的实现并且每次返回相同的实例，Suppliers给我们一些可选项。
使用Suppliers.memoize方法

Suppliers.memoize方法返回一个包装了委托实现的Supplier实例。当第一调用get方法，会被调用真实的Supplier实例的get方法。memoize方法返回被包装后的Supplier实例。包装后的Supplier实例会缓存调用返回的结果。后面的调用get方法会返回缓存的实例。我们可以这样使用Suppliers.memoize方法：

Supplier<Predicate<String>> wrapped = Suppliers.memoize(composedPredicateSupplier);
只增加一行代码我们就可以返回相同的实例。

使用Suppliers.memoizeWithExpiration方法

Suppliers.memoizeWithExpiration方法与memoize方法工作相同，只不过缓存的对象超过了时间就会返回真实Supplier实例get方法返回的值，在给定的时间当中缓存并且返回

Supplier包装对象。注意这个实例的缓存不是物理缓存，包装后的Supplier对象当中有真实Supplier对象的值。 例如：

Supplier<Predicate<String>> wrapped = Suppliers.memoizeWithExpiration(composedPredicateSupplier,10L,TimeUnit.MINUTES);
这里我们包装了Supplier并且设置了超时时间为10分钟。对于ComposedPredicateSupplier来说没什么不同，但是Supplier返回的对象可能不同，可能从数据库当中恢复，例如memoizeWithExpiration方法会非常有用。

通过依赖注入来使用Supplier接口是强有力的组合。然而，如果你使用Guice（google的依赖注入框架），它包含了Provider<T>接口提供了跟
Supplier<T>接口相同的功能。当然，如果你想利用缓存这个特性你可以使用Supplier接口。


参考：http://www.tuicool.com/articles/6NfeYrb