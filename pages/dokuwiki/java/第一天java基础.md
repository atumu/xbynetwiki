title: 第一天java基础 

#  第一天：Java基础 
对象之间的几种基本关系：
(1)泛化(Generalization)
(2)依赖(Dependency)
(3)关联(Association)
(4)聚合(Aggregation)
(5)组合(Composition) 

泛化（继承）
具有层次关系或者可以用树状结构来描述对象关系时，可以考虑使用继承，继承的好处是子类可以容易的使用父类的属性和方法，缺点是子类和父类绑定在一起，不利于后期维护。
在UML中，继承通常是使用空心三角+实线来表示。

依赖
如果A和B是依赖的关系，说明B一般不单独使用，它需要在A中才会发挥作用，通常B是作为A中的方法参数存在的。
在UML中依赖通常使用虚线箭头表示。

关联
如果A和B有关联，那么说明A内部可能会使用到B，但是A和B本身还是独立的关系，通常B会作为A的成员变量存在。
在UML中，关联通常是使用实线箭头来表示，箭头方向是A指向B。

聚合
如果A和B是聚合的，那么说明A和B是“弱拥有”的关系，它们不是独立的关系，但是A和B的生命周期可以使不同的，通常B也是会作为A的成员变量存在。
在UML中，聚合通常是使用空心菱形+实线箭头来表示。

组合
如果A和B是组合的，那么说明A和B是“强拥有”的关系，它们不是独立的关系，并且生命周期也是一样的，通常B作为A的成员变量存在，并且在A的构造函数中进行初始化。
在UML中，组合通常是使用实心菱形+实线箭头表示。