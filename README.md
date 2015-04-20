# javascript异步工作机制

###### 看了几篇关于js异步工作处理机制以及异步编程的文章，有如下总结：

1. javascript语言的执行环境是“单线程”，意思就是如果有多个任务执行，后一个任务必须要等前一个任务执行完毕再执行。

2. 为了解决这个问题，javascript将任务的执行模式分为：同步和异步模式。“同步模式”，程序的执行顺序与任务的执行顺序是一致的。“异步模式”，每一个任务，有一个或多个回调函数（callback），前一个任务结束，不是执行后一个任务，而是执行回调函数，后一个任务无需等到前一个任务完成再执行，所以程序的执行顺序和任务的执行顺序是不一致的、异步的。

3. javascript是单线程的，所以强制所有的异步事件排队等待执行。

4. 从setTimeout和setInterval来看，在执行异步代码的时候有这根本不同。  
        setTimeout ( function ( ) { 
              /* Some long block of code... */ 
              setTimeout (arguments. callee,  10 ); 
        },  10 ); 
    
        setInterval ( function ( ) { 
             /* Some long block of code... */ 
        },  10 );  
    
  setTimeOut回调函数的执行和上一次之间的间隔至少有10ms，可能更多。如果计时器被阻塞而不能执行，它将延迟执行直到下次可能被执行的点才被执行。  
  setInterval的回调函数，将尝试着每10ms执行一次，不论上次是否执行完毕。如果回调函数执行时间足够长，他们将连续执行，并且彼此之间没有时间间隔。

5. 异步编程的4种方法
    * 回调函数  
        若f2等待f1的执行结果，f1执行时间较长，可以将f2写成f1的回调函数  

            function f1(callback) {
                setTimeout(function(){
                    //f1代码
                    callback();
                }, 1000);
            }
          f1(f2);
    回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

    * 事件监听  
       采用事件驱动模式，任务的执行不取决于代码的顺序，而取决于某个事件是否发生。
        
          function f1() {
             setTimeout() {
                 //f1代码
                f1.trigger('done');
             }           
          }
          f1.on('done', f2);

    f1.trigger('done')表示，执行完成后，立即触发done事件，从而开始执行f2。
    这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

    * 发布/订阅

    我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern），又称"观察者模式"（observer pattern）。

            function f1(){
               setTimeout(function () { 
                   // f1的任务代码  
　　　　       jQuery.publish("done");  
　　　　   }, 1000);  
　　      }
            jQuery.subscribe("done", f2);
            jQuery.unsubscribe("done", f2);

    jQuery.publish("done")的意思是，f1执行完成后，向"信号中心"jQuery发布"done"信号，从而引发f2的执行。
此外，f2完成执行后，也可以取消订阅（unsubscribe）。
这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

    * Promises对象

    Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。
简单说，它的思想是，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。

          function f1() {
            var dfd = $.Deferred();
            setTimeout(function(){
                //f1
                dfd.resolve();
            }, 1000);
            return dfd.promise();
          }
          f1().then(f2).then(f3);//指定多个回调函数
          f1().then(f2).fail(f3);//指定发生错误时的回调函数

    这样写的优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚，而且有一整套的配套方法，可以实现许多强大的功能。而且，它还有一个前面三种方法都没有的好处：如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。所以，你不用担心是否错过了某个事件或信号。

