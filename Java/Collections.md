# Collections
## TreeSet
### 排序的实现
```java
package com.tju.se;

import java.util.Comparator;
import java.util.TreeSet;

public class TreeSetPerson {
    public static void main(String[] args) {
        Person p1 = new Person(10);
        Person p2 = new Person(69);
        Person p3 = new Person(89);
        Person p4 = new Person(100);

        TreeSet<Person> set = new TreeSet<>(new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return -(o1.getSocre() - o2.getSocre());
            }
        });
        set.add(p1);
        set.add(p2);
        set.add(p3);
        set.add(p4);

        System.out.println(set);
    }

}

class Person{
    private int socre;

    Person(int socre) {
        this.socre = socre;
    }

    public int getSocre() {
        return socre;
    }

    @Override
    public String toString() {
        return String.valueOf(socre);
    }
}
```