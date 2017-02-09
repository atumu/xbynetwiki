title: java枚举与泛型 

#  java枚举与泛型、迭代结合 
It works exactly the same way as if the Class is passed:
```

public static <E extends Enum<?>> void iterateOverEnumsByInstance(E e)
{
    iterateOverEnumsByClass(e.getClass());
}

public static <E extends Enum<?>> void iterateOverEnumsByClass(Class<E> c)
{
    for (E o: c.getEnumConstants()) {
        System.out.println(o + " " + o.ordinal());
    }
}

```
Usage:

enum ABC { A, B, C }
...
iterateOverEnumsByClass(ABC.class);
iterateOverEnumsByInstance(ABC.A);

参考http://stackoverflow.com/questions/2203861/how-do-i-write-a-generic-for-loop-for-a-java-enum