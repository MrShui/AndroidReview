####为什么在Service中创建子线程而不是Activity中
这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

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
   * at_most：去父控件给的最大尺寸，wrap_content

3. onLayout

4. onDraw

5. View状态的改变与恢复(http://www.jianshu.com/p/69ce52d0eddd)

6. 事件处理

   * |   方法/组件   |           dispatachTouchEvent            |          onInterceptTouchEvent           |            onTouchEvent             |
     | :-------: | :--------------------------------------: | :--------------------------------------: | :---------------------------------: |
     | Activity  |          true、false消费、super往下分发          |                    无                     | true消费，false不消费并调用上层组件的onTouchEvent |
     | ViewGroup | true消费，false事件不往下分发冒泡执行上层的onTouchEvent，super执行onInterceptTouchEvent | true拦截事件交给本层onTouchEvent处理，false或super不拦截事件往下继续分发 |                 同上                  |
     |   View    |  true消费，false事件不往下分发，冒泡执行上层onTouchEvent  |                    无                     |                 同上                  |


   * requestDisallowInterceptTouchEvent(true)：让上层View不在调用拦截方法

