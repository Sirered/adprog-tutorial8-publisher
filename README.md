# Tutorial 8

## How many data your publlsher program will send to the message broker in one run?

According to the publisher code, on every cargo run of the publisher, 5 messages are sent (since there are 5 calls to the p.publish_event method in the main function). All of these messages has "user_created" as the type of event that they are representing. This matches with the fact that in the subscriber, we have specified for it to listen to and handle messages of that type. Additionally, each of them have a UserCreatedMessage that each contains a unique id and name for each of the users that are 'created'.

## The url of: “amqp://guest:guest@localhost:5672” is the same as in the subscriber program, what does it mean?

In the publisher code, the above URI is the URI set for the publisher new_publisher_queue and it happens to match with the subscriber's listener URI. This means that the publisher, when handling events, will send the messages to the URI amqp://guest:guest@localhost:5672. If the subscriber is renning at the time, the message will be received at the host localhost and the port 5672, and since both the username and password matches with the one set on subscriber, the message will be successfully sent and received to the RabbitMQ server. Since the subscriber is listening to that server, once a message is sent by the publisher and received by the server, it will then be distributed to the subscriber, in which it will call its handle function to process the message received.

## Running RabbitMQ as message broker.

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/c96c06a0-b470-4578-bbbc-daa752bc4814)

## Sending and processing event.

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/584c73f0-f932-45b8-9f2b-da315c73fb6f)
![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/75096889-36b1-4fee-b887-fb2b488aaa1e)

First when we do cargo run on subscriber, it will call the main function of the main.rs. The main function will create a listener that will listen to the RabbitMQ server that we have set up, accepting "user_created" messages and will handle messages it receives using UserCreatedHandler that is defined within the same file. It will then loop infinitely, keeping the program, and thus the listener alive.

Afterwards when we do cargo run on the publisher, it will call the main function in the main.rs in the publisher directory. In this main function, it will create a publisher, connected to the RabbitMQ server, and through that publisher, it sends 5 messages to the RabbitMQ server, all of the "user_created" event type. 

When the RabbitMQ server receives these messages, it will check the event type of the messages it receives. Since the subscriber that is connected to the server is listening for "user_created" events, the messages sent by the publisher (which are all of the "user_created" event type) will be sent and handled by the subscriber. 

When thesubscriber gets the messages, it will call the handle function of the handler set to the listener (which is of type UserCreatedHandler). This handle function will take the message payload (which should contain id and name) and will print out "In Galih’s Computer [2206046696]. Message received:" + the UserCreatedMessage sent by the publisher in the payload.
