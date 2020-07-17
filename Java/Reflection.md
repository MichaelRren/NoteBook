

# Reflection

## 概念
* 无论一个类生成了多少个对象，始终只对应一个`.class`文件
* 首先要获得要使用的对象的 `Class` 对象
* 获取一个类的 `Class`对象的三种方式
    * Class.forName("");
    * 类.class;;
    * 对象.getClass();

## 简单使用
```java
//<?> 表示应用于 Object 以下的所有对象
public static void main(String[] args) throws Exception {
    Class<?> classType = Class.forName("java.lang.String");

    Method[] methods = classType.getDeclaredMethods();

    for (Method method : methods) {
        System.out.println(method);
    }
}
```

### 调用指定的方法，注意需要指定传入的参数，因为有重载的方法，因此不能只依靠方法名;`invoke`方法调用指定的方法。
```java
package com.tju.se;

import java.lang.reflect.Method;

public class ReflectTest {
    public static void main(String[] args) throws Exception {
    Class<?> clz = Class.forName("com.tju.se.ReflectTest");
    Object reflect = clz.newInstance();

    Method method = clz.getMethod("add", new Class[]{int.class, int.class});

    Integer res = (int) method.invoke(reflect, 1, 2);

    System.out.println(res);
}

public int add(int param1, int param2) {
    return param1 + param2;
}

public String echo(String message) {
    return "hello " + message;
}
```
### 调用带参数的构造方法生成对象
* 如果使用不带参数的构造方法，那么只需要使用`clz.newInstance()`方法即可
* 如果需要使用带参数的构造方法，则需要使用`constructor.newInstance(...)`

```java
package com.tju.se;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

public class ReflectTest {
    public Object copy() throws Exception {
        Class<?> classType = ReflectTest.class;

        Constructor constructor = classType.getConstructor(new Class[]{});

        Object object = constructor.newInstance(new Object[]{});

        //与 Object obj = classType.newInstance();
        constructor = classType.getConstructor(new Class[]{int.class});
        object = constructor.newInstance(new Object[]{1});
    
    }
    public static void main(String[] args) throws Exception {
        Class<?> classType = ReflectTest.class;

    }
}

class Customer {
    public Customer() {

    }

    public Customer(int age) {
        this.age = age;
    }

    int age;

    public int getAge() {
        return this.age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```

### 获取成员变量
```java
Field[] fields = classType.getDeclaredFields();
for (Field field : fields) {
    String name = field.getName();
    String firstLetter = name.substring(0, 1);
    String restLetter = name.substring(1, name.length());

    System.out.println(name);
}
```

### 生成数组
* `Array.newInstance()` 生成数组类型的对象
* `Integer.TYPE` 返回的是 int，而 `Integer.class`返回的事 Integer 所对应的 class 对象
```java
int dims = new int[]{5, 10, 15};

//创建一个长宽高分别为 5 10 15 的三维数组
Object array = Array.newInstance(Integer.TYPE, dims);

//获取第一维索引为 3 的数组，因此 他是个二维数组
Object arrayObject = Array.get(array, 3);
//获取一个一维数组
arrayObj = arrayObject.get(arrayObject, 5);
//将一维数组的索引为 10 的元素赋值
Array.setInt(arrayObj, 10, 37)

int[][][] cast = (int[][][]) array;
System.out.println(cast[3][5][10]);
```

### 获取私有方法
* `getMethod` 所得到只能是 `public` 的方法或者接口
* 使用 `getDeclaredMethod` 可以获取到私有的方法
* 同时要搭配 `setAttribute(true)` 使用,压制访问检查
* 对于私有成员变量来说也是类似，但是 field 中有 set 方法，可以为属性赋值，这个属性同样也要设置 `setAccessible = true;`
```java
Private p = new Private();
Class<?> clz = p.getClass();

Method method = clz.getDeclaredMethod("hello", new Class[]{String.class});

method.setAttribute(true);

method.invoke(p, new Object[]{"hello"});
```
```java
package com.tju.se;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class PrivateTest {
    public static void main(String[] args) throws Exception {
        Private p = new Private();

        Class<?> clz = p.getClass();

        Field field = clz.getDeclaredField("name");
        field.setAccessible(true);
        field.set(p, "michael");

        Method method = clz.getMethod("get" + field.getName().substring(0, 1).toUpperCase() + field.getName().substring(1)
                , new Class[]{});
        String name = (String) method.invoke(p, new Object[]{});

        System.out.println(name);
    }
}

class Private {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```