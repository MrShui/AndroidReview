####数据库相关操作

SQLiteOpenHelper类，onCreate，onUpgrade，getReadableDatabase()，getWritableDatabase()内存不足时会报错。

#### Android的数据存储形式

sqlite、sharedPrefrence、File、ContentProvider

#### onNewIntent()调用时机

activity onRestart时，启动模式为singleTask（a->b->a）,启动模式为singleTop（a->b->b）

#### Android退出整个应用

```java
moveTaskToBack(true);
android.os.Process.killProcess(android.os.Process.myPid());
System.exit(1);
```

#### 强、软、弱引用

* 强：即使内存溢出，也不会被回收

* 软：内存不足时会被回收（处理图片，占用内存大的类）

  ```java
  private Handler mHandler;

  private static class MyHandler extends Handler {

      private final WeakReference<MainActivity> wrActivity;

      public MyHandler(MainActivity mainActivity) {
          wrActivity = new WeakReference<MainActivity>(mainActivity);
      }

      @Override
      public void handleMessage(Message msg) {
          super.handleMessage(msg);
          if (wrActivity.get() != null) {
              MainActivity mainActivity = wrActivity.get();
          }
      }
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      mHandler = new MyHandler(this);
  }
  ```

* 弱：垃圾回收扫描到的时候就被回收（handler防止内存泄漏）

* 总结：如果避免内存溢出或是常用的对象则使用软引用，如果对性能有很高要求则使用弱引用

#### 自定义View（http://www.jianshu.com/p/c84693096e41）

1. 自定义属性（attrs.xml，命名空间，代码中获取自定义属性）

2. onMeasure
   * unSpecified：父容器没有对当前view有任何限制，view可以取任意尺寸
   * exactly：尺寸是精确的，match_parent，或者指定了尺寸
   * at_most：取父控件给的最大尺寸，wrap_content

3. onLayout

4. onDraw

5. View状态的改变与恢复(http://www.jianshu.com/p/69ce52d0eddd)

6. 事件处理

   * |   方法/组件   |           dispatachTouchEvent            |          onInterceptTouchEvent           |            onTouchEvent             |
     | :-------: | :--------------------------------------: | :--------------------------------------: | :---------------------------------: |
     | Activity  |          true、false消费、super往下分发          |                    无                     | true消费，false不消费并调用上层组件的onTouchEvent |
     | ViewGroup | true消费，false事件不往下分发冒泡执行上层的onTouchEvent，super执行onInterceptTouchEvent | true拦截事件交给本层onTouchEvent处理，false或super不拦截事件往下继续分发 |                 同上                  |
     |   View    |  true消费，false事件不往下分发，冒泡执行上层onTouchEvent  |                    无                     |                 同上                  |


*    requestDisallowInterceptTouchEvent(true)：让上层View不在调用拦截方法

     #### Android长连接和心跳机制

     nio（非阻塞的io）、mina

     #### Android消息处理机制

     - Looper：App主线程不能执行完代码之后就结束了，所以需要一个死循环，looper就是那个死循环
     - MessageQueue：一个线程在同一时间内只能处理一个消息，不具有并发性，所以需要一个消息队列，来保存消息并安排处理顺序，每一个Looper都会维护一个消息队列，且只有一个
     - Handler：其他线程想要和主线程通讯，Handler负责把消息封装到MessageQueue中，Looper接受Handler封装的MessageQueue，并在handler中回调处理
     - Message：载体

     #### 下拉刷新

     * SwipeRefreshLayou源码分析（https://github.com/hanks-zyh/SwipeRefreshLayout/blob/master/README.md）

       mCircleView（中间转动的圆圈，MaterialProgressDrawable），mTargetView（目标滑动的View），onRefreshListener（刷新动画触发的时候执行）

       继承了NestedScrollingParent，NestedScrollingChild，子View（可滑动的View的onTouchEvent中调用helper方法中的的startNestedScroll，及调用onStartNestedScorll方法）

     #### Retrofit

     动态代理模式（InvocationHandler）

     ***代理模式***

     kotlin by关键字

     ```java
     interface Base {
         fun print()
     }

     class BaseImpl(val x: Int) : Base {
         override fun print() {
             System.out.println(x)
         }
     }

     class Derive(base: Base) : Base by base

     fun main(args: Array<String>) {
         Derive(BaseImpl(5)).print()
     }
     ```

     #### Get和Post区别

     1. get从不对服务器上资源做修改，一般用查询，post一般伴随着对服务器端资源的修改，一般修改和删除
     2. get参数直接拼接在url中，post参数封装在header中
     3. get传送的数据量较小，不能大于2kb，post传送的数据量较大，默认不做限制
     4. get安全性低执行效率高

     #### http和https

     * https是http的安全版
     * https协议需要到ca申请证书
     * https是http下加入了ssl加密传输协议
     * 端口不一样，http：80，https：443

     #### Fragment踩的坑

     * <u>getActivity()空指针</u>：内存重启之后，fragment onDetach()了宿主Activity，但是fragment中还是调用了getActivity() 。***在onAttach里把getActivity用成员变量记录（虽然有内存泄漏的风险，但总比直接crash好）***
     * <u>can not perform this action after onSaveInstanceState</u>异常:onSaveInstanceState()执行，到回到Activity（onResume执行），执行fragment事务就会抛出此异常。***1、使用commitAllowingStateLoss()提交事务（如果恰巧activity被强杀，就会导致事务丢失）;2、onResumeFragment执行事务可以防止事无丢失***
     * <u>fragment重叠：</u>***fragment被加载多次，判断saveInstanceState是否为空，来决定是否需要调用fragment add()方法***
     * <u>一些建议：</u>***1、fragment传递数据，使用setArguments()，然后在onCreate()中使用getArguments()，防止内存重启之后数据丢失；2、使用newInstance(参数)创建fragment，优点是调用者只需关心传递的参数，无需关心key(AndroidStudio自带模板方法)；3、BaseFragment中自己记录fragment的activity，防止getActivity()空指针***
     * <u>懒加载：</u>***onHindChanged()不会回调，需要使用setUserVisibleHint(boolean isVisibleToUser)方法***

     #### RxJava

     * 优点：能把复杂的逻辑串成一条线
     * 实现异步操作的库
     * ***map***返回一个新的数据源；***from（2.0方法为：fromArray）***返回接受集合返回一个可观测的数据源；***flatMap***把一个可观测的数据源转换成另一个可观测的数据源；***filter***过滤操作符，true表示留下，false过滤掉

     #### 内存泄漏和内存溢出

     * ***静态存储区：***存储静态数据，包括static数据和常量，程序运行之后一直存在；***栈区：***方法执行时，局部变量，和对象的引用；***堆区：***new的对象存放在堆区
     * 内存泄漏的原因：***长生命周期的对象引用短生命周期的对象，短生命周期的对象已经不再需要，但是因为长生命周期的对象持有它的引用，导致它无法被回收***
     * 内存泄漏场景：
       1. 监听没有remove
       2. io、以及数据库操作完没有close
       3. 集合类没有删除元素，只增不减
       4. 单例中持有activity context，导致activity context无法被gc回收
       5. 匿名内部类，这个引用传给异步线程持，而异步线程和Activity或是fragment生命周期不同步的时候，会造成内存泄漏，handler（非静态内部类会持有外部类的引用）
       6. handler内存泄漏修复，将内部类改为静态内部类，对外部类的引用改为弱引用；cursor、io等及时关闭；不用的对象即使置空释放内存；监听生命周期结束或用完之后要remove掉。
       7. 可以使用LeakCanary检测内存泄漏

     #### MPAndroidChart

     #### Kotlin

     #### Android6.0权限系统

     #### Android性能优化

     #### 进程和线程

     进程是程序的一次执行，线程是进程中执行的一段代码片段

     #### Socket通讯

     #### RecycleView相关知识

     * onCreateViewHolder(),onBindViewHolder(),集成ViewHolder
     * 添加头、尾布局：维护一个mData，复写getItemType()和getItemCount()
     * 添加分割线：使用ItemDecoration
     * 上拉加载更多

     ```java
     public abstract class EndLessOnScrollListener extends RecyclerView.OnScrollListener{

         //声明一个LinearLayoutManager
         private LinearLayoutManager mLinearLayoutManager;

         //当前页，从0开始    private int currentPage = 0;
         //已经加载出来的Item的数量
         private int totalItemCount;

         //主要用来存储上一个totalItemCount
         private int previousTotal = 0;

         //在屏幕上可见的item数量
         private int visibleItemCount;

         //在屏幕可见的Item中的第一个
         private int firstVisibleItem;

         //是否正在上拉数据
         private boolean loading = true;

         public EndLessOnScrollListener(LinearLayoutManager linearLayoutManager) {
             this.mLinearLayoutManager = linearLayoutManager;
         }

         @Override
         public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
             super.onScrolled(recyclerView, dx, dy); 

             visibleItemCount = recyclerView.getChildCount();
             totalItemCount = mLinearLayoutManager.getItemCount();
             firstVisibleItem = mLinearLayoutManager.findFirstVisibleItemPosition();
             if(loading){
                 if(totalItemCount > previousTotal){
                     //说明数据已经加载结束
                     loading = false;
                     previousTotal = totalItemCount;
                 } 
            }
             //这里需要好好理解
             if (!loading && totalItemCount-visibleItemCount <= firstVisibleItem){
                 currentPage ++;
                 onLoadMore(currentPage);
                 loading = true;
             }
         }

         /**
          * 提供一个抽闲方法，在Activity中监听到这个			     EndLessOnScrollListener
          * 并且实现这个方法
          * */
         public abstract void onLoadMore(int currentPage);}
     ```

     * BaseViewAdapter

     ​

     ​

     ​

     ​

     ​

