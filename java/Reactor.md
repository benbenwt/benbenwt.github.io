>同步阻塞：请求IO，等待IO就绪并一直（阻塞），再去read（阻塞），从内核内存拷贝到用户内存。
>同步非阻塞：请求IO，轮询IO是否就绪（非阻塞），然后再read（阻塞），从内核内存拷贝到用户内存。
>IO多路复用：请求IO，注册文件描述符FDS，阻塞等待事件通知，然后read（阻塞），从内核内存拷贝到用户内存。
>异步IO：请求IO，等待通知完成，此完成是指已经把数据从内核内存复制到了用户内存（非阻塞），不需要再阻塞的复制数据到用户内存了。
### 阻塞、非阻塞
>阻塞：请求IO的程序，在DMA读取数据到系统内存、系统内存复制到用户内存这个过程中一直阻塞，直到其复制到用户内存了，然后再执行后续的代码，直接使用用户内存数据bytes。
>非阻塞：请求IO的程序，在上述的DMA->系统内存、系统内存->用户内存的过程中不等待，而是通过轮询的或事件信号等方式检查哪个IO完成了，即已经将数据复制到用户内存了，那么就调用该IO的后续步骤。
非阻塞为什么提高了cpu利用率、性能：因为在DMA->系统内存->用户内存的过程中，cpu实际只在系统内存->用户内存过程工作，在DMA->系统内存实际是空闲的，但是程序却在这段时间一直等待，通过非阻塞可以利用这段空闲时间，去提交其他IO请求，或执行后续操作。
### 同步、异步
>线程同步是指多个线程对于资源的使用需要符合规则
>异步是指：直到数据被拷贝到用户内存，才通知程序进行处理，程序无需进行阻塞的数据读写过程。

### NIO、Netty
### Reactor事件编程模型

### 观察者模式
https://www.runoob.com/design-pattern/observer-pattern.html
>适用于一对多的关系，一个Subject对应多个Obserber，只用修改Subject的值，绑定的Observer也动态的变更。
>Subject：维持一份信息，记录有哪些Observer关注了当前Subject。
>Observer：维护一份信息，当前Observer关注了哪一个Subject。
```
public class Subject {
   
   private List<Observer> observers 
      = new ArrayList<Observer>();//存储关注此subject的obserber
   private int state;
 
   public int getState() {
      return state;
   }
 
   public void setState(int state) {
      this.state = state;
      notifyAllObservers();
   }
 
   public void attach(Observer observer){
      observers.add(observer);      
   }
 
   public void notifyAllObservers(){
      for (Observer observer : observers) {
         observer.update();
      }
   }  
}

public abstract class Observer {
   protected Subject subject;//存储observer关注的subject
   public abstract void update();
}
```

>IO多路复用
>redis IO多路复用