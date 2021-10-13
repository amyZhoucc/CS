# 1. 概述

AsyncTask、IntentService以及HandlerThread的表现形式都有别于传统的线程，但是它们的本质仍然是传统的线程。

AsyncTask来说，它的底层用到了**线程池**，对于IntentService和HandlerThread来说，它们的底层则直接使用了**线程**。

AsyncTask封装了线程池和Handler，它主要是为了方便开发者在子线程中更新UI。

IntentService是一种服务，它不容易被系统杀死从而可以尽量保证任务的执行。

# 2. Android的线程形态

## 2.1 asynctask

是轻量级的异步任务类。它可以在**线程池中执行后台任务**，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。

AsyncTask并不适合进行特别耗时的后台任务

asynctask是一个抽象的泛型类，有3个参数：param、progress、result，如果不需要该参数则可以传递void/unit过去。

1. `onPreExecute()`，**在主线程中执行**，在异步任务执行之前，此方法会被调用，一般可以用于做一些准备工作
2. `doInBackground(Params...params)`，**在线程池中执行**，此方法用于执行异步任务，params参数表示异步任务的输入参数。在此方法中可以通过`publishProgress()`方法来更新任务的进度，publishProgress方法会调用`onProgressUpdate()`方法。另外此方法需要返回计算结果给onPostExecute方法
3. `onProgressUpdate(Progress...values)`，**在主线程中执行**，当后台任务的执行进度发生改变时此方法会被调用。
4. `onPostExecute(Result result)`，**在主线程中执行**，在异步任务执行之后，此方法会被调用，其中result参数是后台任务的返回值，即doInBackground的返回值。

`onCancelled()`方法会被调用，这个时候onPostExecute则不会被调用。

有要求：

1. **AsyncTask的类必须在主线程中加载**，这就意味着第一次访问AsyncTask必须发生在主线程
2. AsyncTask的对象必须在主线程中创建
3. execute方法必须在UI线程调用
4. 不要在程序中直接调用onPreExecute()、onPostExecute、doInBackground和onProgressUpdate方法
5. 一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常。

## 2.2 asynctask的工作原理

在execute中会去调用：executeOnExecutor()方法

在该方法里面：会先执行`onPreExecute()`，然后执行sDefaultExecutor的execute方法，这样线程池才开始执行任务。

sDefaultExecutor实际上是一个**串行的线程池**，一个进程中所有的AsyncTask全部在这个串行的线程池中**排队执行**。

排队执行：

1. 将params封装成FutureTask对象，充当runnable的作用

2. 而该futureTask会被交给SerialExecutor的execute方法去处理

3. SerialExecutor会将该对象插入到任务队列mTask中，等待执行

   1. 如果此时mTask没有在执行的任务，那么scheduletask就会选择下一个执行的任务
   2. 执行完一个换另一个去执行，直到全部执行完毕。

   ——**串行执行的**，

​	AsyncTask中有两个线程池（SerialExecutor和THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler）:

1. SerialExecutor：任务排队
2. THREAD_POOL_EXECUTOR用于真正地执行任务
3. Handler：切换回主线程去执行

