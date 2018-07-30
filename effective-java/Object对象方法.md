# Object 对象方法
<!-- toc -->

### 覆盖 equals 方法遵守通用约定
* 遵守等价关系：自反性、对称性、传递性、一致性
* 对于任何非 null 的引用值 x，x.equals(null) 必须放回 false
* 覆盖 equals 总要覆盖 hashCode
* 没有特别的需求，不要覆盖 equals 方法。

实现高质量 equals 方法
1. 使用 == 检查对象是否是某个对象的引用
2. 使用 instanceof 检查参数是否是正确的类型
3. 把参数转换为正确的类型
4. 对于类中的 **关键字段**，检查参数对象与该对象中的是否相同

### 覆盖 equals 方法时总是覆盖 hashcode 方法
* 在计算 hashcode 时，不要试图通过排除有意义的成员变量来提供性能。

```java
// Typical hashCode method
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
 }
```

### 总是覆盖 toString 方法
* toSting 方法应该返回对象中所有有意义的成员变量信息

### 明智的覆盖 clone 方法
* public 的 clone 方法不需要抛出异常
* clone 对象比较好的方式是提供一个 copy 的构造函数或者工厂方法

```java
class Person implements Cloneable {
    String name;
    int age;
    String[] phoneNumbers;

    public Person(String name, int age, String[] phoneNumbers) {
        this.name = name;
        this.age = age;
        this.phoneNumbers = phoneNumbers;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", phoneNumbers=" + Arrays.toString(phoneNumbers) +
                '}';
    }

    public Person clone() {
        try {
            Person p =  (Person) super.clone();
            p.phoneNumbers = phoneNumbers.clone();
            return p;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 考虑实现 Comparable 接口
* Java7 之后所有的包装类都提供了 compare 静态方法，因此在compareTo 方法中不推荐直接使用 < 或者 > 运算符

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```
