## Tutorial A - Reflection

**a. What is AMQP?**

AMQP (Advanced Message Queuing Protocol) is an open standard application layer protocol designed for message-oriented middleware. AMQP enables different applications to communicate with each other by sending messages through a message broker. The protocol defines how messages are formatted, transmitted, and acknowledged between producers and consumers. It supports features like message queuing, routing, reliability, and security, making it widely used in distributed and event-driven systems. RabbitMQ is one of the most popular message brokers that implements the AMQP protocol.

**b. What does `guest:guest@localhost:5672` mean?**

- The first `guest` is the **username** used to authenticate to the RabbitMQ message broker.
- The second `guest` is the **password** for that username.
- `localhost` means the message broker is running on the **same machine** as the subscriber.
- `5672` is the **port number** that RabbitMQ listens on for AMQP connections.

So the full URL `amqp://guest:guest@localhost:5672` tells the subscriber to connect to a RabbitMQ broker running locally on port 5672, using the default credentials guest/guest.

### Simulation Slow Subscriber

![RabbitMQ Queue 1](assets/images/rabbitmq_queue1.png)

**Why is the total number of queued messages 11?**

The total number of queued messages reached 11 because the subscriber was made intentionally
slow by uncommenting the `thread::sleep(ten_millis)` line, which adds a 1000ms delay for every
single message it processes. While the subscriber is busy sleeping between each message, the
publisher was run multiple times in quick succession, sending 5 new messages each time. Since
the subscriber can only process 1 message per second but the publisher kept sending batches of
5 messages faster than that, the messages piled up in the queue faster than they could be
consumed. This is the classic producer-consumer imbalance problem, where the producer
(publisher) is significantly faster than the consumer (subscriber), causing messages to accumulate
in the broker queue over time.

### Running at Least Three Subscribers

**Three Subscribers Console**
![Console 1](assets/images/console1.png)
![Console 2](assets/images/console2.png)
![Console 3](assets/images/console3.png)

**RabbitMQ with Three Subscribers**
![RabbitMQ Queue 2](assets/images/rabbitmq_queue2.png)

**Why does the spike reduce quicker with three subscribers?**

With three subscribers running simultaneously, the workload is distributed evenly across all
three consumers by RabbitMQ using a round-robin mechanism. Instead of one subscriber
processing all messages alone at 1 message per second, three subscribers can collectively
process 3 messages per second, tripling the throughput. This is clearly visible in the RabbitMQ
dashboard where the queue peaked at only around 3 messages compared to around 11 before
with a single slow subscriber, and dropped back to 0 much faster. Each subscriber independently
picks up a message from the queue, processes it, and moves on to the next available one, so no
single subscriber is overwhelmed.

**What can be improved?**

Looking at the current code, one thing that can be improved is that the `thread::sleep` and the
`now` variable in the subscriber are declared but `now` is never actually used to measure elapsed
time. If the intent is to simulate or measure processing time, the code should properly use
`now.elapsed()` to track it. On the publisher side, the events are hardcoded with fixed user data,
which is not scalable. A better approach would be to dynamically generate or read user data
from an external source rather than hardcoding five static entries every time the publisher runs.

## Bonus (Simulation Slow Subscriber)

### Simulation Slow Subscriber

![CloudAMQP Queue 1](assets/images/cloudamqp_queue1.png)

Running the slow subscriber experiment on CloudAMQP shows a similar queue buildup
pattern as the local setup. With `thread::sleep` enabled, the subscriber can only process
one message per second. However, compared to the local Docker setup, the queue buildup
is slightly different because each message must travel over the internet to the CloudAMQP
broker before being consumed. The added network latency means the round-trip time per
message is longer than localhost, making the subscriber even slower to catch up with the
publisher. Despite this, the core behavior remains the same, messages accumulate when
the consumer is slower than the producer.

### Running at Least Three Subscribers

**Three Subscribers Console**
![Console 1](assets/images/bonus_console1.png)
![Console 2](assets/images/bonus_console2.png)
![Console 3](assets/images/bonus_console3.png)

**RabbitMQ (CloudAMQP) with Three Subscribers**
![RabbitMQ Queue 2](assets/images/cloudamqp_queue2.png)

With three subscribers connected to the CloudAMQP broker simultaneously, the queue
spike is significantly reduced compared to running a single slow subscriber. The RabbitMQ
dashboard shows Connections: 3, confirming all three subscribers are
active. Even with the added network latency from the cloud broker, the three subscribers
collectively process messages fast enough that the queue never accumulates significantly.
This confirms that horizontal scaling of consumers is an effective strategy regardless of
whether the broker is hosted locally or in the cloud. The main difference from the local
setup is that each message delivery takes slightly longer due to network round-trip time,
but the overall throughput improvement from adding more subscribers still holds true.