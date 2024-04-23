# Tutorial 8

## How many data your publlsher program will send to the message broker in one run?

According to the publisher code, on every cargo run of the publisher, 5 messages are sent (since there are 5 calls to the p.publish_event method in the main function). All of these messages has "user_created" as the type of event that they are representing. This matches with the fact that in the subscriber, we have specified for it to listen to and handle messages of that type. Additionally, each of them have a UserCreatedMessage that each contains a unique id and name for each of the users that are 'created'.

## The url of: “amqp://guest:guest@localhost:5672” is the same as in the subscriber program, what does it mean?

In the publisher code, the above URI is the URI set for the publisher new_publisher_queue and it happens to match with the subscriber's listener URI. This means that the publisher, when handling events, will send the messages to the URI amqp://guest:guest@localhost:5672. If the subscriber is renning at the time, the message will be received at the host localhost and the port 5672, and since both the username and password matches with the one set on subscriber, the message will be successfully sent and received to the RabbitMQ server. Since the subscriber is listening to that server, once a message is sent by the publisher and received by the server, it will then be distributed to the subscriber, in which it will call its handle function to process the message received.

## Running RabbitMQ as message broker.

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/fa4c8240-afd1-4809-ade0-da613cd2f038)
