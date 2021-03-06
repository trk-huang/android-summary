# AIDL进程通信

简介

上一章我们已经实现了应用内跨进程操作，但是里面其实是有缺陷的，服务端处理客户端的消息是串行的，消息是一条条处理，所以如果并发量大的话，通过Messager来通信就不大合适了。

> Note: Using AIDL is necessary only if you allow clients from different applications to access your service for IPC and want to handle multithreading in your service. If you do not need to perform concurrent IPC across different applications, you should create your interface by implementing a Binder or, if you want to perform IPC, but do not need to handle multithreading, implement your interface using a Messenger. Regardless, be sure that you understand Bound Services before implementing an AIDL.

主要意思就是你可以用 Messenger 处理简单的跨进程通信,但是高并发量的要用AIDL

我们先来演示一下应用内的AIDLservice：

#### 服务端

1.首先创建一个AIDLSerivce：

```java
/**
 * Created by huangdaju on 17/8/8.
 */

public class AIDLService extends Service {

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }


}
```

2.创建AIDL文件

3.在 Service 中创建一个 Binder,并在 onBind 的时候返回

```java
private Binder mBinder = new IMyAidlInterface.Stub(){
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public void danner() throws RemoteException {
            System.out.println("亲爱的，晚上想吃点啥");
        }
    };
```

#### 客户端

1.创建一个自定义的ServiceConnection

```java
/**
 * Created by huangdaju on 17/8/8.
 */

public class AidlServiceConnection implements ServiceConnection {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {

    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
}
```

2.绑定服务，当成功绑定服务后，把Binder传过来：

```java
Intent intent = new Intent(this, AIDLService.class);
        AidlServiceConnection aidlServiceConnection = new AidlServiceConnection();
        bindService(intent, aidlServiceConnection, BIND_AUTO_CREATE);
```

3.在onServiceConnected\(\)通过asInterface获取服务端传过来的对象，并调用服务端的方法

```java
/**
 * Created by huangdaju on 17/8/8.
 */

public class AidlServiceConnection implements ServiceConnection {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        IMyAidlInterface iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
        try {
            iMyAidlInterface.danner();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
}
```

现在客户端就可以调用danner\(\)方法来进行跨进程通信。现在是客户端可以给服务端发数据。看下图结果:

```java
08-08 18:43:20.299 22426-22426/? I/art: Late-enabling -Xcheck:jni
08-08 18:43:20.362 22426-22426/com.zhiping.alibaba.aidltest:aidlserivce W/System: ClassLoader referenced unknown path: /data/app/com.zhiping.alibaba.aidltest-1/lib/arm
08-08 18:43:20.400 22426-22438/com.zhiping.alibaba.aidltest:aidlserivce I/System.out: 亲爱的，晚上想吃点啥
```

通过AIDL传送负责数据：

* 基本数据类型
* 实现parcelabel接口对象
* ArrayList，并且里面的元素也是要支持AIDL的
* HashMap，并且里面的元素也是要支持AIDL的

```java
/**
 * Created by huangdaju on 17/8/8.
 */

public class Food implements Parcelable {
    private String name;
    private float price;

    // 1.必须实现Parcelable.Creator接口,否则在获取Person数据的时候，会报错，如下：
    // android.os.BadParcelableException:
    // Parcelable protocol requires a Parcelable.Creator object called  CREATOR on class com.um.demo.Person
    // 2.这个接口实现了从Percel容器读取Person数据，并返回Person对象给逻辑层使用
    // 3.实现Parcelable.Creator接口对象名必须为CREATOR，不如同样会报错上面所提到的错；
    // 4.在读取Parcel容器里的数据事，必须按成员变量声明的顺序读取数据，不然会出现获取数据出错
    // 5.反序列化对象


    public Food() {
    }

    protected Food(Parcel in) {
        name = in.readString();
        price = in.readFloat();
    }

    public static final Creator<Food> CREATOR = new Creator<Food>() {
        @Override
        public Food createFromParcel(Parcel in) {
            return new Food(in);
        }

        @Override
        public Food[] newArray(int size) {
            return new Food[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        // 1.必须按成员变量声明的顺序封装数据，不然会出现获取数据出错
        // 2.序列化对象
        dest.writeString(name);
        dest.writeFloat(price);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Food{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

接着我们需要在aidl文件夹下，创建一个名词相同的aidl文件

> 注意：这里我们要通过File-&gt;new-&gt;File的方式创建Food.aidl文件，如果你使用aidl创建文件方式，会提示你 Interface Name must be unique

如下代码：

```java
package com.zhiping.alibaba.aidltest;
parcelable Food;
```

然后回到原来的IMyAidlInterface.aidl文件中，修改方法，如下：

```java
Food danner();
void setFood(in Food food);
```

这里需要注意3个地方：

1. IMyAidlInterface.aidl虽然跟Food.aidl在同一个包下,但是这里还是需要手动import进来

2. 这里声明方法时,需要在参数前面增加一个tag,这个tag有三种,in,out,inout,这里表示的是这个参数可以支持的流向:

   * **in:**这个对象能够从客户端到服务器,但是作为返回值从服务器到客户端的话数据不会传送过去\(不会为null,但是字段都没有赋值\)

   * **out:**这个对象能够作为返回值从服务器到客户端,但是从客户端到服务器数据会为空\(不会为null,但是字段都没有赋值\)

   * **inout:**能从客户端到服务器,也可以作为返回值从服务器到客户端（注意，不要都设为inout，要看需求设置，不然会增加内存开销）

   ![](/assets/QQ20170809-2.png)

   ![](/assets/QQ20170809-1.png)

   ![](/assets/QQ20170809-0.png)

然后修改AIDLService，如下：

```java
/**
 * Created by huangdaju on 17/8/8.
 */

public class AIDLService extends Service {

    private Binder mBinder = new IMyAidlInterface.Stub() {
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {

        }

        @Override
        public Food danner() throws RemoteException {
            System.out.println("亲爱的，晚上想吃点啥");

            Food food = new Food();
            food.setName("花椰菜111");
            food.setPrice(1.6f);
            return food;
        }

        @Override
        public void setFood(Food food) throws RemoteException {

            System.out.println("亲爱的，晚上想吃点啥: " + food.toString());
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }


}
```

 这时就可以跨进程通信了。看如下效果：

```java
08-09 11:04:00.722 7779-7796/com.zhiping.alibaba.aidltest:aidlserivce I/System.out: 亲爱的，晚上想吃点啥: Food{name='花椰菜', price=1.5}
08-09 11:04:00.722 7779-7795/com.zhiping.alibaba.aidltest:aidlserivce I/System.out: 亲爱的，晚上想吃点啥
```

```java
08-09 11:04:00.723 7749-7749/com.zhiping.alibaba.aidltest I/System.out: food1:Food{name='花椰菜111', price=1.6}
```



