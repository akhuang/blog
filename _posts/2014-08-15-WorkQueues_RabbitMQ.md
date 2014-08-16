---
layout: post
title: "WorkQueues"
date: 2014-08-15 -0946
comments: true
categories: [personal github]
---

#####WorkQueues要点#####


1. 默认情况下队列发送消息给worker后会立即从队列中删除, 但此种情况下, 无法保证消息被worker顺利接收并执行;
   
   解决办法: Message acknowledgment(消息确认)


        //第二个参数为:noAsk, 置为false表示需要worker回应Queue是否已经处理完消息
        channel.BasicConsume("task_queue", false, consumer);

        //执行完任务后调用下面代码告诉Queue消息已处理完毕 
        channel.BasicAck(ea.DeliveryTag, false);
   
2. 在第一点满足的情况下, 如何Queue服务端挂掉也会导致消息丢失

        //标识quque是持久的
        bool durable = true;
        channel.QueueDeclare("task_queue", durable, false, false, null);
    
        var properties = channel.CreateBasicProperties(); 
        properties.SetPersistent(true); //持久化
    
3. Round-robin dispatching(循环调度): 

   默认情况下, Queue/worker1, worker2; 如果Queue发布了5条消息, 第二条消息将耗时很久, 那么worker1/worker2将依次收到消息, worker1: 1/3/5, worker2:2/4, 无论第二条消息执行多久, 第4条消息还是会发给worker2, 不太合理

   Fair dispatch:
   
   //在调用QueueingBasicConsumer之前设置channel的BasciQos, 第二个参数:prefetchcount=1,即worker只能处理一条message, 在处理完前不允许再给它分配消息
   
        channel.BasicQos(0, 1, false);
        var consumer = new QueueingBasicConsumer(channel);
   
