[TOC]
下图表明了并发和并行的区别：
![image](http://opy4iwqsf.bkt.clouddn.com/WX20170622-184223@2x.png)
## Future接口
Future接口在java5中被引入，设计初衷是对将来某个时刻会发生的结果进行建模。它建模了一种一步计算。打个比方，你拿了一袋子衣服到你中意的干洗店去洗。干洗店的员工会给你张发票，告诉你什么时候你的衣服会洗好(这就是一个future事件)。衣服干洗的同事，你可以去做其他的事情。Future的另一个有点是它比更底层的Thread更易用，要使用Future，通常你只需要将耗时的操作封装在一个Callable对象中，再将他交给ExecutorService，就万事大吉了。

```
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double>(){
    public Double call(){
        return doSomeLongComputation();
    }
});
doSomethingElse();
Double result = future.get(1,TimeUnit.SECONDS);
```
这种编程方式让你的线程可以在ExecutorService以并发方式调用另一个线程执行耗时操作的同时，去执行一些其他的任务。接着，如果你已经运行到没有异步操作的结果就无法继续任何有意义的工作时，可以调用他的get方法去获取操作的结果。如果操作已经完成，该方法就会立刻返回操作的结果，否则它会阻塞线程，直到操作完成，返回相应的结果。

如果长时间运行的操作永远不反回了会怎样？为了处理这种可能性，虽然Future提供了一个无需任何参数的get方法，还是推荐使用重载的版本，它接受一个超时的参数，通过它，可以定义线程等待Future结果的最长时间。
![image](http://opy4iwqsf.bkt.clouddn.com/WX20170622-191357@2x.png)
### Future接口的局限性
通过第一个例子，知道了Future接口提供了方法来检测异步计算是否已经结束(使用isDone()方法)，等待异步操作结束，以及获取计算的结果。但是这些特性还不足以让你编写简洁的并发代码。比如，我们很难表述Future结果之间的依赖性;从文字描述上很简单，“当长时间计算任务完成时，请将该计算的结果通知到另一个长时间运行的计算任务，这两个计算任务都完成后时，请将该计算的结果通知到另一个长时间运行的计算任务，这两个计算任务都完成后,将计算的结果与另一个查询操作结果合并”。但是，使用Future中提供的方法完成这样的操作又是另一回事。这也是我们需要更具描述能力的特性的原因，比如下面这些：
- 将两个异步计算合并为一个——这两个异步计算之间互相独立，同时第二个又依赖于第一个的结果
- 等待Future集合中的所有任务都完成
- 仅等待Future集合中最快结束的任务完成(有可能因为它们师徒通过不同的方式计算同一个值)，并返回它的结果。
- 通过编程方式完成一个Future任务的执行(即以手工设定异步操作结果的方式)
- 应对Future的完成事件(即当Fureu的完成事件发生时会受到通知，并能使用Future计算的结果进行下一步的操作，不止是简单地紫色等待操作的结果)

### 使用CompletableFuture构建异步应用
为了展示CompletableFuture的强大特性，我们会创建一个名为“最佳加个查询器”的应用，他会查询多个在线商店，依据给定的产品或服务找出最低的价格。这个过程中，你会学到几个重要的技能
- 首先，你会学到如何为你的客户提供异步API(如果你拥有一间在线商店的话，这是非常有帮助的)
- 其次，你会掌握如何让你使用了同步API的代码变为非阻塞代码。你会了解如何使用流水线将两个连续的异步操作合并为一个异步计算操作。这种情况肯定会出现，比如，在线商店返回了你想要购买商品的原始价格，并附带着一个折扣代码——最终，要计算出该商品的实际价格，你不得不访问第二个远程折扣服务，长须改折扣代码对应的折扣比率。
- 你还会学到如何以响应式的方式处理异步操作的完成事件，以及随着各个商店返回他的商品价格，最佳价格查询器如何持续地更新每种商品的最佳推荐，而不是等待所有的商店都返回他们各自的价格(这种方式存在着一定的风险，一旦某件商店的服务中断，用户可能遭受白屏)。

## 实现异步API

为了实现最佳价格查询器应用,让我们从每个商店都应该提供的API定义入手。首先，商店应该声明依据指定产品名称返回价格的方法：

```
public class Shop{
    public double getPrice(String produc){
        
    }
}
```
该方法的内部实现会查询商店的数据库，但也有可能执行一些其他耗时的任务，比如联系其他外部服务。所以会有耗时任务，我们在剩下的内容中，采用delay方法模拟这些长期运行的方法的执行，它会人为地引入1秒钟的延迟。

```
public static void delay(){
    try{
        Thread.sleep(1000L);
    }catch(InterruptedException e){
        thorw new RuntimeExcepiton(e);
    }
}
```
在getPrice方法中引入一个模拟的延迟

```
public double getPirce(String product){
    return calculatePrice(product);
}
private double calculatePrice(String product){
    delay();
    return random.nextDouble() * product.charAt(0) + product.charAt(1);
}
```
很明显，这个API的使用者，调用该方法时，它依旧会被阻塞。为等待同步事件完成而等待1秒钟，这是无法接受的。
### 将同步方法转换为异步方法

```
public Future<Double> getPriceAsync(String product){
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();//创建Completablefuture对象，会包含计算结果
    new Thread(()->{
        double price = calculatePrice(product);//在另一个线程中以异步方式执行计算
        fururePrice.complete(price);//需长时间计算的任务结束并得出结果时，设置future的返回值。
    }).start()
    return futurePrice;//无需等待还没结束的计算，直接返回Future对象。
}
```
### 错误处理
如果没有意外，我们目前开发的代码工作的很正常。但是，如果价格计算过程中产生了错误会怎么样？非常不幸，这种情况下你会得到一个相当糟糕的结果：用于提示错误的异常会被限制在试图计算商品价格的当前线程的范围内，最终会杀死线程。而这会导致等待get方法返回结果的客户端会被永久的阻塞。

```
public Future<Double> getPriceAsync(String product){
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();//创建Completablefuture对象，会包含计算结果
    new Thread(()->{
        try{
            double price = calculatePrice(product);//在另一个线程中以异步方式执行计算
            fururePrice.complete(price);//需长时间计算的任务结束并得出结果时，设置future的返回值。
        }catch(Exception e){
            futurePrice.completeExceptionally(e); 
        }
    }).start()
    return futurePrice;//无需等待还没结束的计算，直接返回Future对象。
}
```
使用工厂方法supplyAsync创建CompletableFuture
目前为止我们已经了解了如何通过编程创建安CompletableFuture对象以及如何获取返回值，虽然看起来这些操作已经比较方便，但还有进一步的提升空间，CompletableFuture类自身提供了大量精巧的工厂方法，使用这些方法能更容易地万恒整个流程。

使用工厂方法supplyAsync创建CompletableFure对象

```
public Future<Double> getPriceAsync(String product){
    return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```
