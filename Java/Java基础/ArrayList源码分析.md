# ⭐ArrayList源码分析

---

`ArrayList`的底层是**数组队列**，相当于动态数组。与Java中的数组相比，它的容量能动态增长。

* `ArrayList`继承了`AbstractList`，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等操作。
* `ArrayList`实现了`RandomAccess`接口，`RandomAccess`是一个标志性接口，表明实现这个接口的`List`集合是支持快速随机访问的。在`ArrayList`中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
* `ArrayList`实现了`Cloneable`接口，即覆盖了函数`clone()`，能被克隆。
* `ArrayList`实现了`java.io.Serializable`接口，这意味着`ArrayList`支持序列化，能通过序列化去传输。

和`Vector`不同，**`ArrayList`中的操作不是线程安全的！**所以，建议在单线程中才使用`ArrayList`，而在多线程中可以选择`Vector`或者`CopyOnWriteArrayList`。

✍下面是`ArrayList`核心源码的详细解读：

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class ArrayList<E> extends AbstractList<E> 
    implements List<E>,RandomAccess,Cloneable,java.io.Serializable{
    private static final long serialVersionUID = 8683452581122892189L;
    
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
    
    /**
     * 空数组(用于空实例)
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    
    //用于默认大小空实例的共享空数组实例。
    //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; //非私有以简化嵌套类访问
    
    /**
     * ArrayList所包含的元素个数
     */
    private int size;
}
```

## 1.构造函数

```java
	/**
	 * 带初始容量参数的构造函数(用户可以在创建ArrayList对象时自己指定集合的初始大小)
	 */
	public ArrayList(int initialCapacity){
        if(initialCapacity > 0){
            //如果传入的参数大于0，则创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if(initialCapacity ==0){
            //如果传入的参数等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else{
            //其他情况，抛出异常
            throw new IlleagalArgumentException("IllrgalCapacity:"+initialCapacity);
        }
    }
	
	/**
	 * 默认无参数构造函数
	 * DEFAULTCAPACITY_EMPTY_ELEMENTDATA为0,初始化为10,也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
	 */
	public ArrayList(){
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
	
	/**
	 * 构造一个包含指定集合的元素列表，按照他们由集合的迭代器返回的顺序。
	 */
	public ArrayList(Collection< ? extends E > c){
        //将指定集合转换为数组
        elementData = c.toArray();
        //如果elementData数组的长度不为0
        if((size == elementData.length) != 0){
            //如果elementData不是Object类型数据(c.toArray可能返回的不是Object类型的数组,所以加上下面的语句用于判断)
            if(elementData.getClass()!= Object[].class){
                //将原来不是Object类型的elementData数组的内容,赋值给新的Object类型的elementData数组
                elementData = ArrayList.copyOf(elementData,size,Object[].class);
            } else{
               //其他情况，用空数组代替
                this.elementData = EMPTY_ELEMENTDATA;
            }
        }
    }
```

## 2.扩容机制

### Ⅰ 自动扩容

`ArrayList`自动扩容发生在`add()`方法调用的时候：

```java
	/**
	 * 将指定的元素追加到此列表的末尾
	 */
public boolean add(E e){
    //扩容
    ensureCapacityInternal(size + 1);//Increments modCount!! 变量记录着集合的修改次数 按照元素个数+1,确认数组容量是否够用
    elementData[size++] = e;
    //这里看到ArrayList添加元素的实质就相当于为数组赋值
    return true;
}

	/**
	 * 在此列表中的指定位置插入指定的元素
	 * 先调用rangeCheckForAdd对index进行界限检查;然后调用ensureCapacityInternal方法保证capacity足够大;
	 * 再将从index开始之后的所有成员后移一个位置;将element插入index位置;最后size加1
	 */
	public void add(int index,E element){
        rangrCheckForAdd(index);
        ensureCapacityInternal(size + 1);//Increments modCount！！
        //arraycopy()实现数组之间复制的方法,下面就用到了arraycopy()方法实现数组自己复制自己 
        System.arraycopy(elementData, index, elementData, index + 1, size-index);
       elementData[index] = element;
        size++;
    }
```

`ensureCapacityInternal()`是用来**以最小容量进行扩容**的（将用户传入的参数和默认的容量进行比较，选取最大值）：

```java
	//以最小容量进行扩容
	private void ensureCapacityInternal(int minCapacity){
        ensureExplicitCapacity(calculateCapacity(elementData,minCapacity));
    }
```

`calculateCapacity`用来获取最小容量`minCapacity`：

```java
	//获取最小容量
	private static int calculateCapacity(Object[] elementData,int minCapacity){
        if(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA){
            return Math.max(DEFAULT_CAPACITY,minCapacity);
        }
        return minCapacity;
    }
```

`ensureExplicitCapacity`判断该`ArrayList `是否需要扩容（如果最小扩容量大于现已存在的数组容量，则需要进行扩容）：

```java
	//判断是否需要扩容
	private void ensureExpliciCapacity(int minCapacity){
        modCount++;
        //如果最小扩容量大于现已存在的数组容量，则需要进行扩容
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容,调用此方法代表已经开始扩容了
            grow(minCapacity);
    }
```

`ArrayList`扩容的核心方法`grow`：

```java
	/**
	 * 要分配的最大数组大小
	 */
	private static inal int MAX_ARRAY_SIZE = Integer.Max_VALUE - 8;

	/**
	 * ArrayList扩容的核心方法。
	 */
	private void grow(int minCapacity){
        //oldCapacity为旧容量,newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity右移一位,其效果相当于oldCapacity
        //我们知道位运算的速度远远快于整除运算,整句运算式的结果就是将新容量更新为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //再判断一下新数组的容量够不够,够了就直接使用这个长度创建新数组 若还是小于最小需要容量,那么就把最小需要容量当作数组的新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量
        //若超出了,则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE,如果minCapacity大于MAX_ARRAY_SIZE,则新容量则为Interger.MAX_VALUE,否则,新容量大小则为 MAX_ARRAY_SIZE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //调用Arrays.copyOf方法将elementData数组指向新的内存空间(newCapacity的连续空间),并将elementData的数据复制到新的内存空间
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
		//比较 minCapacity 和 MAX_ARRAY_SIZE
        private static int hugeCapacity(int minCapacity) {
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
        }
```

从 `grow` 方法中我们可以清晰的看出其实 **`ArrayList `扩容的本质就是计算出新的扩容数组的 `size` 后实例化，并将原有数组内容复制到新数组中去。**

### Ⅱ 使用 ensureCapacity 提高添加速度

