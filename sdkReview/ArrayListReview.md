# ArrayList源码学习
`ArrayList` 实现于 `List`、`RandomAccess` 接口。可以插入空数据，也支持随机访问。
其中`element`和`size`是比较重要的属性，`element`是一个数组，也是`ArrayList`内部真正存放数据的容器，`size`记录这个list中存放的元素数量

## 构造方法

`ArrayList`有三个构造方法

- 无参构造方法

  ```java
  public ArrayList() {
      this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
  }
  ```

  其中`DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 是`ArrayList`中定义的一个空数组



- 两个有参的构造方法

  ```java
  public ArrayList(int initialCapacity) {
      if (initialCapacity > 0) {
          this.elementData = new Object[initialCapacity];
      } else if (initialCapacity == 0) {
          this.elementData = EMPTY_ELEMENTDATA;
      } else {
          throw new IllegalArgumentException("Illegal Capacity: "+
                                             initialCapacity);
      }
  }
  ```

  - 这个方法传入一个初始容量，作用就是初始化一个`initialCapacity` 大小的数组

  ```java
  public ArrayList(Collection<? extends E> c) {
      elementData = c.toArray();
      if ((size = elementData.length) != 0) {
          // c.toArray might (incorrectly) not return Object[] (see 6260652)
          if (elementData.getClass() != Object[].class)
              elementData = Arrays.copyOf(elementData, size, Object[].class);
      } else {
          // replace with empty array.
          this.elementData = EMPTY_ELEMENTDATA;
      }
  }
  ```

  - 这个方法传入一个`Collection c`,将c转为数组，这个地方有个细节，它判断了一下c的类信息，如果不是`object[]`类型，使用Arrays.copy方法，将它转成`object`,这就是所谓的 类型擦除 的地方

## 常用方法

- add()方法

  - add(E e)

  ```java
  public boolean add(E e) {
      ensureCapacityInternal(size + 1);  // Increments modCount!!
      elementData[size++] = e;
      return true;
  }
  ```

  其中有一个`ensureCapacityInternal`方法

  ```java
  private void ensureCapacityInternal(int minCapacity) {
      if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
          minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
      }
  
      ensureExplicitCapacity(minCapacity);
  }
  ```

  这边主要是比较你传进来的初始容量`minCapacity`和`DEFAULT_CAPACITY`的大小，取其中较大的一个作为初始容量，`ensureExplicitCapacity`是将`elementData`数组进行扩容的地方，具体的实现在grow()方法中

  ```java
  private void grow(int minCapacity) {
      // overflow-conscious code
      int oldCapacity = elementData.length;
      // 新容量为原先的1.5倍
      int newCapacity = oldCapacity + (oldCapacity >> 1);
      if (newCapacity - minCapacity < 0)
          newCapacity = minCapacity;
      // 这边的MAX_ARRAY_SIZE为2^31 - 8 
      if (newCapacity - MAX_ARRAY_SIZE > 0)
          newCapacity = hugeCapacity(minCapacity);
      // minCapacity is usually close to size, so this is a win:
      elementData = Arrays.copyOf(elementData, newCapacity);
  }
  ```

  - add(int index, E element) 将元素插到指定位置

    与上面的方法的差异是，有一个检查`index`范围的动作，通过Arrays.copy方法，使得在index和index+1位置上的元素相同，最后再将index位置元素替换掉，达到指定位置插入的目的。

    ```java
    public void add(int index, E element) {
        rangeCheckForAdd(index);
    
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    ```

- get()方法

  - get(int index) 根据下标获取元素

    实现很简单，先判断下标是否越界，如果没有越界，就直接从`elemenData`数组中取下标为`index`的元素

    ```java
    public E get(int index) {
        rangeCheck(index);
    
        return elementData(index);
    }
    ```

- remove() 方法

  - fastRemove  这是一个私有的方法，顾名思义——快速移除 方法，所谓的快速就是省去了数组越界的判断，也不会返回移除的元素

    ```java
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
    ```

    另外值得一提的一点就是，`ArrayList`中涉及到移除元素的操作(包括 clear )时，都不会去改变`elementData`数组的大小，只会把相应位置元素置为null

##  引起思考的一些点

- `ArrayList`实现了序列化的接口，那么它其中的属性，默认就是可序列化的，但是我们发现`elementData`是用`transient`关键字修饰的，也就是说，序列化的时候会忽略这个属性，通俗的来说，就是如果我们把这个东西存到文件中，然后再读出来，`elementData`会丢失，那事实上这显然是不合理的，既然实现了序列化接口，就要求数据是持久化的。

  ![1554886012803](C:\Users\Klaus\AppData\Roaming\Typora\typora-user-images\1554886012803.png)

  ![1554886048337](C:\Users\Klaus\AppData\Roaming\Typora\typora-user-images\1554886048337.png) 

  那么`ArrayList`是怎么实现`elementData`的可序列化的呢？

  - writeObject(java.io.ObjectOutputStream s)

    这个方法会将`elementData`中的元素保存到流中

    ```java
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();
    
        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);
    
        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
    
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    ```

  - readObject(java.io.ObjectInputStream s)

    这个方法从流中读取元素保存到`elementData`中

    ```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;
    
        // Read in size, and any hidden stuff
        s.defaultReadObject();
    
        // Read in capacity
        s.readInt(); // ignored
    
        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);
    
            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
    ```

  - 这样设计的意义

    仔细的同学会发现，在`writeObject`方法中，我们只将`elementData`中下标为0-size-1的元素放到了流中，而直接序列化`elementData`的话，所需的空间肯定是大于这个(扩容以及各种remove操作)，那么或许这就是一个目的所在，只序列化必要的东西，尽可能地节省空间

- `modCount++;` 操作

  在`add、remove`操作中，都会有这个一条语句，但是`modCount`这个属性我们平时并不会用到，那么它的作用是是什么

  简单来说这是一个为了支持`ArrayList`fail-fast行为的属性，典型应用就是在`writeObject()`方法中，在方法结尾会检验这个`modCount`的值是否变化，如果变化则抛出`ConcurrentModificationException`异常，同样，在foreach循环中进行`add`或者`remove`操作，也会抛出这个异常。

  关注`modCount`具体值其实没有太大的意义，因为很少有人需要知道这个容器被精确操作了多少次，所以它也是用`transient`关键字来修饰的，并不需要持久化存储，因为没意义。
