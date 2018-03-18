
# 使用wait/notifyAll来写生产者和消费者
```java

public class ProducerConsumerTest{

  public class Producer extends Thread{
    private Queue<Integer> queue;
    private int maxSize;

    public Producer(Queue<Integer> queue, int maxSize, String name){
      super(name);
      this.queue = queue;
      this.maxSize = maxSize;
    }

    @Override
    public void run(){
      while(true){
        synchornized(queue){
          while(queue.size == maxSize){
            try {
              //队列满
              System.out.println("Queue is full, Producer thread waiting for customer to take something for queue");
              queue.wait();
            }catch (InterrupedException e){
              e.printStackTrace();
            }
          }
          int i = new Random.nextInt();
          System.out.println("Producing value: " + i);
          queue.offer(i);
          queue.notifyAll();
        }
      }
    }
  }


  public class Consumer extends Thread{
    private Queue<Integer> queue;
    private int maxSize;

    public Consumer(Queue<Integer> queue, int maxSize, String name){
      super(name);
      this.queue = queue;
      this.maxSize = maxSize;
    }

    @Override
    pubic void run(){
      while(true){
        synchornized(queue){
          while(queue.isEmpty()){
            try{
              queue.wait();
            }catch(InterrupedException e){
              e.printStackTrace();
            }
          }

          Integer i = queue.remove();
          System.out.println("Consuming value : " + value);
          queue.notifyAll();
        }
      }
    }
  }

  pubic static void main(String[] args){
    Queue<Integer> queue = new LinkedList<Integer>();
    int maxSize = 10;

    Producer producer = new Producer(queue, maxSize, "Producer");
    Consumer consumer = new Consumer(queue, maxSize, "Consumer");

    producer.start();
    consumer.start();
  }
}

```

## 使用Condition
```java
public class Buffer{
  private Lock lock;
  private Condition notFull;
  private Condition notEmpty;
  private Object[] items;
  private int putPtr = 0 , takePtr = 0, count = 0;

  public Buffer(int size){
    this.lock = new ReentrantLock();
    this.notFull = this.lock.newCondition();
    this.notEmpty = this.lock.newCondition();
    this.items = new Object[size];
  }

  public void put(Object o) throws InterrupedException{
    lock.lock();
    try{
      while(count == items.length){
        notFull.await();
      }
      items[putPtr] = o;
      if (++putPtr == items.length) {
        putPtr = 0;
      }
      count ++;
      notEmpty.signal();
    }finally{
      lock.unlock();
    }
  }


  public Object take() throws InterrupedException{
    this.lock.lock();
    try{
      while(count == 0){
        notEmpty.await();
      }
      Object x = items[takePtr];
      if (++takePtr == items.length) {
         takePtr = 0;
      }

      --count;
      notFull.signal();
      return Object;
    }finally{
      this.lock.unlock();
    }
  }
}
```

## 使用BlockingQueue
```java

public class ProducerConsumerTest{
    public class Producer extends Thread{
      private BlockingQueue queue;
      public Producer(BlockingQueue queue){
        this.queue = queue;
      }

      @Override
      public void run(){
        while(true){
          this.queue.put(product());
        }
      }

      public Object product(){
        return new Object();
      }
    }


  public class Consumer extends Thread{
    private BlockingQueue queue;

    public Consumer(BlockingQueue queue){
      this.queue = queue;
    }

    @Override
    public void run(){
      while(true){
        consume(this.queue.take());
      }
    }

    public void consume(Object o){
      //handler
    }
  }
}
```

## 套接字 服务端接收到客户端输入的正方形的边长，计算输出正方形的面积

```java

public class Server{

  public static int PROT = 9999;
  public static int BACKBLOCK = 1;
  public static String address = "localhost";

  public static void main(String[] args){
     ServerSocket server = new ServerSocket(PROT, BACKBLOCK, InetAddress.getByName(address));
     Socket socket = server.accpet();

     DataInputStream dis = null;
     DataOutputStream dos = null;
     try{
       dis = new DataInputStream(new BufferInputStream(socket.getInputStream()));
       dos = new DataOutputStream(new BufferOutputStream(socket.getOutputStream()));

       do{
         double len = dis.readDouble();
         System.out.println("服务器端接收到的边长数据为： " + len);
         double area = len * len;
         dos.writeDouble(area);
         dos.flush();
       }while(dis.readInt() != 0)
     }catch(IOException e){
       e.printStackTrace();
     }finally{

       dis.close();
       dos.close();
       socket.close();
       server.close();
     }
  }
}


public class Client{
  private static final PROT = 9999;
  public static String address = "localhost";
  public static void main(String[] args){
      Socket socket = new Socket(address, PROT);
      boolean flag = false;

      Scanner scanner = new Scanner(System.in);

      DataInputStream dis = null;
      DataOutputStream dos = null;
      try{
        dis = new DataInputStream(new BufferInputStream(socket.getInputStream()));
        dos = new DataOutputStream(new BufferOutputStream(socket.getOutputStream()));
        while(!flag){
          System.out.println("请输入正方形的变成： ");
          double len = scanner.nextDouble();

          dos.writeDouble(len);
          dos.flush();

          double area = dis.readDouble();
          System.out.println("服务器返回的计算面积为:" + area);


         while(true){
           System.out.println("继续计算？(Y/N)");

           String str = scanner.next();
           if ("N".equalIgnoreCase(str)) {
              dos.writeInt(0);
              dos.flush();
              flag = true;
              break;
           }else if("Y".equalIgnoreCase(str)){
              dos.writeInt(1);
              dos.flush();
              break;
           }
         }
      }catch(IOException e){
        e.printStackTrace();
      }finally{
        dos.close();
        dis.close();
        socket.close();
        }
      }

    }
}
```
