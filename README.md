## Tutorial A - Reflection

**a. What is AMQP?**

AMQP (Advanced Message Queuing Protocol) is an open standard application layer protocol designed for message-oriented middleware. AMQP enables different applications to communicate with each other by sending messages through a message broker. The protocol defines how messages are formatted, transmitted, and acknowledged between producers and consumers. It supports features like message queuing, routing, reliability, and security, making it widely used in distributed and event-driven systems. RabbitMQ is one of the most popular message brokers that implements the AMQP protocol.

**b. What does `guest:guest@localhost:5672` mean?**

- The first `guest` is the **username** used to authenticate to the RabbitMQ message broker.
- The second `guest` is the **password** for that username.
- `localhost` means the message broker is running on the **same machine** as the subscriber.
- `5672` is the **port number** that RabbitMQ listens on for AMQP connections.

So the full URL `amqp://guest:guest@localhost:5672` tells the subscriber to connect to a RabbitMQ broker running locally on port 5672, using the default credentials guest/guest.