# RxUtil2
[![RxUtil2][rxSvg]][rx]  [![api][apiSvg]][api]

一个实用的RxJava2工具类库。在原作者基础上删除了rxbus和rxLifecycle的依赖，更单纯的rxjava线程调度工具类

> 如果你习惯RxJava1，请移步[RxUtil](https://github.com/xuexiangjys/RxUtil)


## 特征

* 订阅池管理。
* 线程调度辅助工具。
* RxBinding 使用工具类。
* RxJava常用方法工具类。

## 1、演示（见代码）


## 2、如何使用
目前支持主流开发工具AndroidStudio的使用，直接配置build.gradle，增加依赖即可.

### 2.1、Android Studio导入方法，添加Gradle依赖

先在项目根目录的 build.gradle 的 repositories 添加:
```
allprojects {
     repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
```

然后在dependencies添加:

```
dependencies {
   ...
   implementation 'io.reactivex.rxjava2:rxjava:2.1.12'
   implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
   //rxbinding的sdk
   implementation 'com.jakewharton.rxbinding2:rxbinding:2.1.1'

   implementation 'com.github.xuexiangjys:RxUtil2:1.1.5'
}
```

### 3.1、RxJavaUtils使用

#### 3.1.1、线程任务

1.RxIOTask：在io线程中操作的任务
```
RxJavaUtils.doInIOThread(new RxIOTask<String>("我是入参123") {
    @Override
    public Void doInIOThread(String s) {
        Log.e(TAG, "[doInIOThread]  " + getLooperStatus() + ", 入参:" + s);
        return null;
    }
});
```

2.RxUITask：在UI线程中操作的任务

```
RxJavaUtils.doInUIThread(new RxUITask<String>("我是入参456") {
    @Override
    public void doInUIThread(String s) {
        Log.e(TAG, "[doInUIThread]  " + getLooperStatus() + ", 入参:" + s);
    }
});
```

3.RxAsyncTask：在IO线程中执行耗时操作 执行完成后在UI线程中订阅的任务。
```
RxJavaUtils.executeAsyncTask(new RxAsyncTask<String, Integer>("我是入参789") {
    @Override
    public Integer doInIOThread(String s) {
        Log.e(TAG, "[doInIOThread]  " + getLooperStatus() + ", 入参:" + s);
        return 12345;
    }

    @Override
    public void doInUIThread(Integer integer) {
        Log.e(TAG, "[doInUIThread]  " + getLooperStatus() + ", 入参:" + integer);
    }
});
```

4.RxIteratorTask:遍历集合或者数组的任务，在IO线程中执行耗时操作 执行完成后在UI线程中订阅的任务。
```
RxJavaUtils.executeRxIteratorTask(new RxIteratorTask<String, Integer>(new String[]{"123", "456", "789"}) {
    @Override
    public Integer doInIOThread(String s) {
        RxLog.e("[doInIOThread]" + getLooperStatus() + ", 入参:" + s);
        return Integer.parseInt(s);
    }

    @Override
    public void doInUIThread(Integer integer) {
        RxLog.e("[doInUIThread]  " + getLooperStatus() + ", 入参:" + integer);
    }
});
```

#### 3.1.2、订阅者Subscriber

1.SimpleSubscriber：简单的订阅者，已对错误进行捕获处理，并对生命周期进行日志记录。可设置IExceptionHandler接口自定义错误处理，设置ILogger接口自定义日志记录。

2.ProgressLoadingSubscriber：带进度条加载的订阅者，实现`IProgressLoader`接口可自定义加载方式。
```
Observable.just("加载完毕！")
        .delay(3, TimeUnit.SECONDS)
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new ProgressLoadingSubscriber<String>(mProgressLoader) {
            @Override
            public void onNext(String s) {
                Toast.makeText(RxJavaActivity.this, s, Toast.LENGTH_SHORT).show();
            }
        });
```

### 3.2、DisposablePool使用

DisposablePool：RxJava的订阅池

1.增加订阅：`add(@NonNull Object tagName, Disposable disposable)` 或者 `add(Disposable disposable, @NonNull Object tagName)`
```
DisposablePool.get().add(RxJavaUtils.polling(5, new Consumer<Long>() {
    @Override
    public void accept(Long o) throws Exception {
        Toast.makeText(RxJavaActivity.this, "正在监听", Toast.LENGTH_SHORT).show();
    }
}), "polling");
```

2.取消订阅：`remove(@NonNull Object tagName)`、`remove(@NonNull Object tagName, Disposable disposable)`、`removeAll()`
```
DisposablePool.get().remove("polling");
```

### 3.3、RxBindingUtils使用

1.setViewClicks:设置点击事件
```
RxBindingUtils.setViewClicks(mBtnClick, 5, TimeUnit.SECONDS, new Action1<Object>() {
    @Override
    public void call(Object o) {
        toast("触发点击");
    }
});
```

2.setItemClicks:设置条目点击事件


### 3.4、RxSchedulerUtils使用

1. 订阅发生在主线程 （  ->  -> main)

```
.compose(RxSchedulerUtils.<T>_main_o())  //Observable使用
.compose(RxSchedulerUtils.<T>_main_f())  //Flowable使用

```

2. 订阅发生在io线程 （  ->  -> io)

```
.compose(RxSchedulerUtils.<T>_io_o())  //Observable使用
.compose(RxSchedulerUtils.<T>_io_f())  //Flowable使用

```

3. 处理在io线程，订阅发生在主线程（ -> io -> main)

```
.compose(RxSchedulerUtils.<T>_io_main_o()) //Observable使用
.compose(RxSchedulerUtils.<T>_io_main_f()) //Flowable使用

```

4. 处理在io线程，订阅也发生在io线程（ -> io -> io)

```
.compose(RxSchedulerUtils.<T>_io_io_o()) //Observable使用
.compose(RxSchedulerUtils.<T>_io_io_f()) //Flowable使用

```

5. 自定义线程池

由于`Schedulers.io()`内部使用的是`CachedWorkerPool`，而他最终创建线程池的方法是`newScheduledThreadPool`，它是一个核心只有1个线程，有效时间为60s，但是线程池的线程容纳数量是`Integer.MAX_VALUE`的线程池。

如果线程在执行的过程中发生了长时间的阻塞，导致线程一直在工作状态的话，线程池将无法回收线程，只能不断地创建线程，最终有可能造成线程创建的数量过多，导致程序OOM。

使用`RxSchedulerUtils.setIOExecutor`可以替换工具类中使用的io线程：

```
RxSchedulerUtils.setIOExecutor(AppExecutors.get().poolIO()); //全局替换

.compose(new SchedulerTransformer<Integer>(AppExecutors.get().poolIO())) //局部替换
```

[rxSvg]: https://img.shields.io/badge/RxUtil2-1.1.5-brightgreen.svg
[rx]: https://github.com/LatoAndroid/RxUtil2
[apiSvg]: https://img.shields.io/badge/API-14+-brightgreen.svg
[api]: https://android-arsenal.com/api?level=14
