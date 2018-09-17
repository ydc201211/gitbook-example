# List
- ArrayList是基于数组实现的，是一个动态数组，其容量能自动增长
- ArrayList不是线程安全的，Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。

### 1、ArrayList源码分析
- 1、属性
    ArrayList定义了两个私有属性：


        /**
        *  Default initial capacity.
        */
        private static final int DEFAULT_CAPACITY = 10;

        /**
        * Shared empty array instance used for empty instances.
        */
        private static final Object[] EMPTY_ELEMENTDATA = {};

        /**
        *  Shared empty array instance used for default sized
        *  empty instances. We
        * distinguish this from EMPTY_ELEMENTDATA to know how * * much to inflate when
        * first element is added.
        */
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
        /** 
        * The array buffer into which the elements  
        * of the ArrayList are stored. 
        * The capacity of the ArrayList is the  
        *  length of this array buffer. 
        */  
        private transient Object[] elementData;  
    
        /** 
        * The size of the ArrayList (the number of elements it
        * contains). 
        * 
        * @serial 
        */  
        private int size;

- elementData为对象数组，用来存储ArrayList内的元素，size表示元素的数量，DEFAULT_CAPACITY表示默认数组大小为10，（在定义ArrayList不指定大小的时候），EMPTY_ELEMENTDATA表示元素存放的数组，默认为空，DEFAULTCAPACITY_EMPTY_ELEMENTDATA表示扩容的数组，

- 有个关键字需要解释：transient。Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。


        //User类
        public class User implements Serializable {  
            private static final long serialVersionUID = 996890129747019948L;  
            private String name;  
            private transient String psw;  

            public User(String name, String psw) {  
                this.name = name;  
                this.psw = psw;  
            }  

            public String toString() {  
                return "name=" + name + ", psw=" + psw;  
            }  
        }  

        public class TestTransient {  
            public static void main(String[] args) {  
                User user = new User("张三","123456"）;
                System.out.println(userInfo);  
         try {  
             // 序列化，被设置为transient的属性没有被序列化  
             ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream(  
                     "User.out"));  
             o.writeObject(user);  
             o.close();  
         } catch (Exception e) {  
             // TODO: handle exception  
             e.printStackTrace();  
         }
         try {  
             // 重新读取内容  
             ObjectInputStream in = new ObjectInputStream(new FileInputStream(  
                     "UserInfo.out"));  
             User readUser = (User) in.readObject();  
             //读取后psw的内容为null  
             System.out.println(readUser.toString());  
         } catch (Exception e) {  
             // TODO: handle exception  
             e.printStackTrace();  
         }  
        }  
        }

- 2、构造方法

         /**
        * Constructs an empty list with the specified initial * * capacity.
        *
        * @param  initialCapacity  the initial capacity of the list
        * @throws IllegalArgumentException if the specified *initial capacity is negative
        */
        // ArrayList带容量大小的构造函数。
        public ArrayList(int initialCapacity) {
            if (initialCapacity > 0) {
                this.elementData = new Object[initialCapacity];
            } else if (initialCapacity == 0) {
                this.elementData = EMPTY_ELEMENTDATA;
            } else {
                throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
            }
        }

        /**
        * Constructs an empty list with an initial capacity of * ten.
        */
        // ArrayList无参构造函数。默认容量是10。 
        public ArrayList() {
            this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
        }

        // 创建一个包含collection的ArrayList    
        public ArrayList(Collection<? extends E> c) {    
            elementData = c.toArray();    
            size = elementData.length;    
            if (elementData.getClass() != Object[].class)    
                elementData = Arrays.copyOf(elementData, size, Object[].class);    
        }
- 3、核心方法
- get(index)

        /**
        * 返回list中索引为index的元素
        * @param  index 需要返回的元素的索引
        * @return list中索引为index的元素
        * @throws IndexOutOfBoundsException 如果索引超出size
        */
        public E get(int index) {
            //越界检查
            rangeCheck(index);
            //返回索引为index的元素
            return elementData(index);
        }

        /**
        * 越界检查。
        * 检查给出的索引index是否越界。
        * 如果越界，抛出运行时异常。
        * 这个方法并不检查index是否合法。比如是否为负数。
        * 如果给出的索引index>=size，抛出一个越界异常
        */
        private void rangeCheck(int index) {
            if (index >= size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

       /**
        * 返回索引为index的元素
        */
        @SuppressWarnings("unchecked")
        E elementData(int index) {
            return (E) elementData[index];
        }

- 扩容-ensureCapacity等方法

       /**
        * 增加ArrayList容量。
        * 
        * @param   minCapacity   想要的最小容量
        */
        public void ensureCapacity(int minCapacity) {
            // 如果elementData等于DEFAULTCAPACITY_EMPTY_ELEMENTDATA，最小扩容量为DEFAULT_CAPACITY，否则为0
            int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;
            //如果想要的最小容量大于最小扩容量，则使用想要的最小容量。
            if (minCapacity > minExpand) {
                ensureExplicitCapacity(minCapacity);
            }
        }
        


       /**
        * 数组容量检查，不够时则进行扩容，只供类内部使用。
        * 
        * @param minCapacity    想要的最小容量
        */
        private void ensureCapacityInternal(int minCapacity) {
            // 若elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA，则取minCapacity为DEFAULT_CAPACITY和参数minCapacity之间的最大值
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
                minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
            }

            ensureExplicitCapacity(minCapacity);
        }



        /**
        * 数组容量检查，不够时则进行扩容，只供类内部使用
        * 
        * @param minCapacity 想要的最小容量
        */
        private void ensureExplicitCapacity(int minCapacity) {
            modCount++;

            // 确保指定的最小容量 > 数组缓冲区当前的长度  
            if (minCapacity - elementData.length > 0)
                //扩容
                grow(minCapacity);
        }



        /**
        * 分派给arrays的最大容量
        * 为什么要减去8呢？
        * 因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。
        */
        private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;



        /**
        * 扩容，保证ArrayList至少能存储minCapacity个元素
        * 第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
        * 
        * @param minCapacity 想要的最小容量
        */
        private void grow(int minCapacity) {
            // 获取当前数组的容量
            int oldCapacity = elementData.length;
            // 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一半。
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            //如果扩容后的容量还是小于想要的最小容量
            if (newCapacity - minCapacity < 0)
                //将扩容后的容量再次扩容为想要的最小容量
                newCapacity = minCapacity;
            //如果扩容后的容量大于临界值，则进行大容量分配
            if (newCapacity - MAX_ARRAY_SIZE > 0)
                newCapacity = hugeCapacity(minCapacity);
            // minCapacity is usually close to size, so this is a win:
            elementData = Arrays.copyOf(elementData,newCapacity);
        }



        /**
        * 进行大容量分配
        */
        private static int hugeCapacity(int minCapacity) {
            //如果minCapacity<0，抛出异常
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            //如果想要的容量大于MAX_ARRAY_SIZE，则分配Integer.MAX_VALUE，否则分配MAX_ARRAY_SIZE
            return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
        }

1、进行空间检查，决定是否进行扩容，以及确定最少需要的容量

2、如果确定扩容，就执行grow(int minCapacity)，minCapacity为最少需要的容量

3、第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。

4、第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。

5、对扩容后的容量进行判断，如果大于允许的最大容量MAX_ARRAY_SIZE，则将容量再次调整为MAX_ARRAY_SIZE。至此扩容操作结束。

- add( int index, E element)


       /**
        * 在制定位置插入元素。当前位置的元素和index之后的元素向后移一位
        *
        * @param index 即将插入元素的位置
        * @param element 即将插入的元素
        * @throws IndexOutOfBoundsException 如果索引超出size
        */
        public void add(int index, E element) {
            //越界检查
            rangeCheckForAdd(index);
            //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
            ensureCapacityInternal(size + 1);  // Increments modCount!!
            // 对数组进行复制处理，目的就是空出index的位置插入element，并将index后的元素位移一个位置
            System.arraycopy(elementData, index, elementData, index + 1,size - index);
            //将指定的index位置赋值为element
            elementData[index] = element;
            //实际容量+1
            size++;
        }

1、越界检查

2、空间检查，如果有需要进行扩容

3、插入元素

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

- remove( int index) 

        /**
        * 删除list中位置为指定索引index的元素
        * 索引之后的元素向左移一位
        * @param index 被删除元素的索引
        * @return 被删除的元素
        * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
        */
        public E remove(int index) {
            //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
            rangeCheck(index);
            //结构性修改次数+1
            modCount++;
            //记录索引为inde处的元素
            E oldValue = elementData(index);

            // 删除指定元素后，需要左移的元素个数
            int numMoved = size - index - 1;
            //如果有需要左移的元素，就移动（移动后，该删除的元素就已经被覆盖了）
            if (numMoved > 0)
                System.arraycopy(elementData, index+1, elementData, index,
                                numMoved);
            // size减一，然后将索引为size-1处的元素置为null。为了让GC起作用，必须显式的为最后一个位置赋null值
            elementData[--size] = null; // clear to let GC do its work

            //返回被删除的元素
            return oldValue;
        }

       /** 
        * 越界检查。
        * 检查给出的索引index是否越界。
        * 如果越界，抛出运行时异常。
        * 这个方法并不检查index是否合法。比如是否为负数。
        * 如果给出的索引index>=size，抛出一个越界异常
        */
        private void rangeCheck(int index) {
            if (index >= size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

1.检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常

2.将索引大于index的元素左移一位（左移后，该删除的元素就被覆盖了，相当于被删除了）。

3.将索引为size-1处的元素置为null（为了让GC起作用）。

- set( int index, E element)

    /**
    * 替换指定索引的元素
    *
    * @param 被替换元素的索引
    * @param element 即将替换到指定索引的元素
    * @return 返回被替换的元素
    * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
    */
    public E set(int index, E element) {
        //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
        rangeCheck(index);

        //记录被替换的元素
        E oldValue = elementData(index);
        //替换元素
        elementData[index] = element;
        //返回被替换的元素
        return oldValue;
    }