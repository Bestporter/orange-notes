rabbitmq



docker上运行`docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management`



使用消息确认标记和p refetch_count 你可以建立一个work queue，设置持久化选项让任务存活，即使rabbitmq被重启。

Using message acknowledgments and prefetch_count you can set up a work queue. The durability options let the tasks survive even if RabbitMQ is restarted.





rabbitmq的消息模型，生产者并不会直接把消息直接交给队列，实际上， 通常生产者甚至根本不知道消息是否会被传递到任何队列。 

生产者只能发送消息到一个`exchange`，`exchange`是一个一边接收生产者传送过来的数据，另一边将这些数据传送给队列

> The exchange must know exactly what to do with a message it receives. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the *exchange type*.
>
> exchange必须确切地知道如何处理它收到的消息。是否应将其附加到特定队列？它应该附加到许多队列中吗？或者它应该被丢弃。其规则由*exchange type*定义。 
>
> There are a few exchange types available: direct, topic, headers and fanout. We'll focus on the last one -- the fanout. Let's create an exchange of that type, and call it logs:
>
> 有几种可用的exchange types：直接交换、主题交换、头交换和扇出交换。我们将关注最后一个--the fanout。让我们创建一个这种类型的交换，并将其称为日志：

```python
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body=message)
```

> The empty string denotes the default or *nameless* exchange: messages are routed to the queue with the name specified by routing_key, if it exists.
>
>  空字符串表示默认的或无名的exchange：消息被路由到由routing_key指定的队列（如果存在的话）。 

创建一个exchange type

```python
channel.exchange_declare(exchange='logs',
                         exchange_type='fanout')
```

首先，每当我们连接rabbit的实际，需要刷新一下，清空队列。可以通过`queue_declare(queue='')`实现。在这一点上`result.method.queue`包含随机队列名称。例如，它可能看起来像amq.gen-JzTY20BRgKO-HjmUJj0wLg.

其次，每当消费者连接关闭，队列应该被删除，*There's an exclusive flag for that:*

```python
result = channel.queue_declare(queue='', exclusive=True)
```



### *binding*

> We've already created a fanout exchange and a queue. Now we need to tell the exchange to send messages to our queue. That relationship between exchange and a queue is called a *binding*.



> A binding is a relationship between an exchange and a queue. **This can be simply read as: the queue is interested in messages from this exchange.**

```python
channel.queue_bind(exchange='logs',
                   queue=result.method.queue)
```





> Bindings can take an extra routing_key parameter. T**o avoid the confusion with a basic_publish parameter we're going to call it a binding key**. This is how we could create a binding with a key:

```python
channel.queue_bind(exchange=exchange_name,
                   queue=queue_name,
                   routing_key='black')
```

> We were using a fanout exchange, which doesn't give us too much flexibility - it's only capable of mindless broadcasting.
>
> 我们使用的是fanout exchange，这并没有给我们太多的灵活性-它只能进行无意识的广播。