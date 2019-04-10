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

    与上面的方法的差异是，有一个检查`index`范围的动作，通过Arrays.copy方法，始得在index和index+1位置上的元素相同，最后再将index位置元素替换掉，达到指定位置插入的目的。
