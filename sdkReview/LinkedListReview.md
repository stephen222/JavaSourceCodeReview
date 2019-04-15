# LinkedList源码学习
`LinkedList` 实现于 `List`、`Deque` 接口。它是基于双向链表实现的，同时也有List的特点。
它有三个属性，`first` ,`last` ,`size` , 分别记录这个链表的头元素，尾元素和 其中存放的元素数量

## 构造方法

LinkedList`有两个构造方法

- 无参构造方法

  ```java
   public LinkedList() {
      }
  ```

  可以看到，无参的构造方法没有做任何事



- 有参的构造方法

  ```java
   public LinkedList(Collection<? extends E> c) {
          this();
          addAll(c);
      }
  ```

  - 这个方法传入一个`Collection c`,然后调用addAll()方法

  ```java
  public boolean addAll(int index, Collection<? extends E> c) {
      checkPositionIndex(index);
  
      Object[] a = c.toArray();
      int numNew = a.length;
      if (numNew == 0)
          return false;
  
      Node<E> pred, succ;
      if (index == size) {
          succ = null;
          pred = last;
      } else {
          succ = node(index);
          pred = succ.prev;
      }
  
      for (Object o : a) {
          @SuppressWarnings("unchecked") E e = (E) o;
          Node<E> newNode = new Node<>(pred, e, null);
          if (pred == null)
              first = newNode;
          else
              pred.next = newNode;
          pred = newNode;
      }
  
      if (succ == null) {
          last = pred;
      } else {
          pred.next = succ;
          succ.prev = pred;
      }
  
      size += numNew;
      modCount++;
      return true;
  }
  ```

  - 分析
    - 将 `Collection c`从list的指定位置`index`开始插入，其中定义了两个局部变量`succ`和 `pred` ,当加到list的末端时(包括第一次添加元素)，`succ` 为null，将`last` 赋值给`pred` ,非末端时，需要通过`node(index)`方法找到对应的节点，然后赋值给`succ`，同时将`succ`的`prev`指向`pred`其中有一个小设计是会判断`index`的范围，如果落在前半段,则从前往后查找，否则从后往前查找，这也是LinkedList的一个缺点，无法像ArrayList一样随机查找；
    - 将`Collection c`转成数组之后，遍历数组，新建结点，判断要插入的地方是不是链表的头，是的话，赋值给`first`作为头结点，如果不是头结点，则将它置于`pred`的下一个节点，更新`pred`
    - 最后判断`succ`是否为null，为null的话说明是加在链表尾部的，所以将`pred`赋给`last`，如果`succ`不为null，说明是加在链表的中间，需要将`succ`和`pred`拼接起来，形成一个完整的链表

## 常用方法

- add()方法

  - add(E e)

  ```java
  public boolean add(E e) {
      linkLast(e);
      return true;
  }
  ```

  其中有一个`linkLast(E e)`方法

    ```java
  void linkLast(E e) {
          final Node<E> l = last;
          final Node<E> newNode = new Node<>(l, e, null);
          last = newNode;
          if (l == null)
              first = newNode;
          else
              l.next = newNode;
          size++;
          modCount++;
  }
    ```

  这边新建一个节点`newNode`，置于`last`之后，然后更新`last`指向新建节点`newNode`,此时需要判断加入的元素是否是容器中的第一个元素，如果是第一个元素，那么`first`同样指向`newNode`，此时`first`和`last`的pre和next都是null;如果容器中已经有元素了，那么` final Node<E> newNode = new Node<>(l, e, null);last = newNode;`这两步使得`last`的`pre`指向了`l`,但是`l`的`next`仍然是null，这不符合双向链表的定义，所以需要将`l`的`next`指向`last`

  - addFirst(E e)

    ```java
    public void addFirst(E e) {
        linkFirst(e);
    }
    ```

    其中`linkFirst`方法与`linkLast`逻辑几乎一样，只不过是将新节点加到了`first`之前

- get()方法

  - get(int index) 根据下标获取元素

    ```java
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    ```

    实现很简单，先判断下标是否越界，如果没有越界，通过`node(int index)` 方法，取到第`index`个`Node`，返回它的值

    ```java
    Node<E> node(int index) {
        // assert isElementIndex(index);
    
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    ```

    与`ArrayList`相似的设计思想，判断`index`与首位两端的距离，从更近的一端开始遍历，也就是说，如果要获得或者插入到链表居中位置，会产生比较大的开销

  - poll()方法

    ```java
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    ```

    返回`first`结点的值，并且将它从链表中移除

    ```java
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
    ```

- remove() 方法

  - remove(int index)   移除指定位置的结点，并返回结点的值

    ```java
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    ```

    首先校验`index`是否落在范围内，然后执行`unlink(Node node)`方法

    ```java
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
    	//如果移除的是头结点
        if (prev == null) {
            //将当前结点的后结点作为头结点
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
    	//如果移除的是尾节点
        if (next == null) {
    		//将当前结点的前结点作为尾结点
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
    
        x.item = null;
        size--;
        modCount++;
        return element;
    }
    ```

    思路是这样，将结点x的前结点与后结点连接起来，并返回结点x的值；其中还要处理两种特殊情况：

    ​	-移除的是头结点`first`

    ​	是：将当前结点的下一个结点作为头结点

    ​	否：将前结点`prev`的next指向后结点`next`，将x的`prev`指向null，也就是切断了x和前结点的双链

    ​	-移除的是尾结点`last`

    ​	是：将当前结点前一个结点作为尾结点

    ​	否：将后结点`next`的prev指向前结点`prev`,将x的`last`指向null，切断了x和后结点的双链

    

##  引起思考的一些点

- Queue操作

  `LinkedList`支持队列操作 FIFO，这得益于  offer(E e)加到队尾  poll()取到头结点的值(支持null)    remove()取到头结点的值(如果为null，抛出NoSuchElementException异常)

- Deque操作(双端队列)

  `LinkedList`支持双向队列的操作，offerFirst(E e)、offerLast(E e)、peekLast()等操作

  这样也可以用`LinkedList`来实现栈操作，后进先出，也是十分方便
