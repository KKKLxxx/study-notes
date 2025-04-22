# Java四种引用类型

## 一、强引用

默认的引用类型

```
Object obj = new Object(); // 只要obj还指向Object对象，Object对象就不会被回收
obj = null; // 手动置null
```

**只要强引用存在，垃圾回收器将永远不会回收被引用的对象**，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样JVM就可以适时的回收对象了

## 二、软引用

软引用是用来描述一些非必需但仍有用的对象。**在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等

用程序测试一下，给虚拟机设置2个参数，`-Xms2M -Xmx3M`，用来限制内存大小

```
public class Main {
    private static List<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        testSoftReference();
    }
    
    private static void testSoftReference() {
        for (int i = 0; i < 5; i++) {
            byte[] buff = new byte[1024 * 1024];
            SoftReference<byte[]> sr = new SoftReference<>(buff);
            list.add(sr);
        }

        for (int i = 0; i < list.size(); i++) {
            Object obj = ((SoftReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
}
```

```
null
null
null
null
[B@5cad8086
```

可以看到，因为都是软引用，所以前4个引用因为内存不够被自动回收了，而没有报错

## 三、弱引用

**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**

```
public class Main {
    private static List<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        testWeakReference();
    }

    private static void testWeakReference() {
        for (int i = 0; i < 5; i++) {
            byte[] buff = new byte[1024 * 1024];
            WeakReference<byte[]> sr = new WeakReference<>(buff);
            list.add(sr);
        }

        System.gc(); // 主动通知垃圾回收

        for (int i = 0; i < list.size(); i++) {
            Object obj = ((WeakReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
}
```

```
null
null
null
null
null
```

由于都是弱引用，在主动通知GC后，即使内存能够放下最后一个数组，所有弱引用也都被回收了

这是**ThreadLocal使用的引用类型**

## 四、虚引用

**虚引用并不会真正引用一个对象，但可以用于通知某一对象被回收，这样就可以根据对象的回收状态执行一些操作**

虚引用需要配合一个引用队列使用，当对象被回收时，就会被加入到这个队列

```
public class Main {
    public static void main(String[] args) {
        Object o1 = new Object();
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
        PhantomReference<Object> phantomReference = new PhantomReference<>(o1, referenceQueue);
        System.out.println("***************GC回收前***************");
        System.out.println(o1);
        System.out.println(phantomReference.get());
        System.out.println(referenceQueue.poll());

        System.out.println("***************启动GC***************");
        o1 = null;
        System.gc();
        try {
            Thread.sleep(500); // 确保GC执行完
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(o1);
        System.out.println(phantomReference.get());
        System.out.println(referenceQueue.poll());
    }
}
```

```
***************GC回收前***************
java.lang.Object@5cad8086
null
null
***************启动GC***************
null
null
java.lang.ref.PhantomReference@6e0be858
```

