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

![RabbitMQ_Queue](assets/images/rabbitmq_queue.png)

**Why is the total number of queued messages 90?**

The total number of queued messages reached 90 because the subscriber was made intentionally
slow by uncommenting the `thread::sleep(ten_millis)` line, which adds a 1000ms delay for every
single message it processes. While the subscriber is busy sleeping between each message, the
publisher was run multiple times in quick succession, sending 5 new messages each time. Since
the subscriber can only process 1 message per second but the publisher kept sending batches of
5 messages faster than that, the messages piled up in the queue faster than they could be
consumed. This is the classic producer-consumer imbalance problem, where the producer
(publisher) is significantly faster than the consumer (subscriber), causing messages to accumulate
in the broker queue over time.