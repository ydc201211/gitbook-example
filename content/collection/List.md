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

