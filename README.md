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

## Monitoring chart based on publisher.

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/7f51ca91-42a3-43a1-be87-396bfbfed3a6)

We get spikes in 'Consumer Ack's each time we run publisher. This is because when we run publisher, the publisher will send 5 messages to the RabbitMQ server. Since the messages are of "user_created" type, it will be sent to subscriber in which the subscriber will process the events. When it has finished processing the event, it will send Consumer Acks acknowledging that he has received and successfully processed the messages to the RabbitMQ server. In summary, publisher sends 5 messages to the RabbitMQ server when run, which is forwarded to the subscriber, who will process and Ack the message, which would mean that the RabbitMQ server would get a spike of consumer Acks because of such.

## Simulation slow subscriber

**I had run the publisher 5 times. The first screenshot is to show the Total amount of messages in queue that was recorded after I run the publishers(with 21 Total messages) and the second screenshot is to show the shape of the queue graph, after the entire queue was processed**

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/e5098060-5327-4fc6-a7e0-856aa070681f)
![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/6932a062-ee55-49e5-bbba-b95553dfa26b)

**Why did it show 21 total messages were in queue at a time**

When we run the publisher 5 times in quick succession, we would be sending a total of 25 messages at once. Since it takes a second for our application to process each, most of those messages will have to be enqueued into the message queue before they can be sent to the subscriber (otherwise they would not be processed and just be dropped). Thus a lot of them would have to wait for their turn in the queue, while subscriber handles the messages that came before them, thus leading to the spike of messages we saw in the message queue grah. As to why it states 21 specifically, despite the fact that there should be 25 messages is probably because in the time that it took for the RabbitMQ Server UI to update (which it does so quite slowly), the subscriber had handled 3 of the messages and is on their fourth, thus leaving only 21 messages unprocessed and in queue. After peaking at 21, the total amount of messages went to 16 then 11 then 5 then 0 (I am not 100% sure that the number after 11 was 5, I just remember it was either 4, 5 or 6), further proving that the RabbitMQ UI doesn't update in real time, and in the time between updates more than 1 message could be distributed to subscriber and processed. 

## Reflection and Running at least three subscribers

**I had run the publisher 5 times in this run as well. The first screenshot is to show what the subscribers looked like while they were processing the messages, the second screenshot is what they look like after they were done and 3rd screenshot is to show the RabbitMQ message queue graph after completion**

![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/a4e1940e-061f-468b-9d6c-92cc01a7bd52)
![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/4c07dc60-3f53-4b27-bdab-c2fbcb2929bf)
![image](https://github.com/Sirered/adprog-tutorial8-publisher/assets/126568984/67f56934-73bf-4366-839f-0b74238da831)

The reason as to why the message queue graph decreases a lot more steeply when running 3 subscribers as opposed to running only one, is because of the fact that RabbitMQ can divide the messages evenly among the 3 subscribers. This is because all 3 subscribers would be listening to the same server and listening for the same topic("user_created") thus they would all be connected to the same queue. This means that when at least one of them becomes free and gives a consumer Ack, while there are messages in the queue, RabbitMQ will instantly give that subscriber the next message in lin. Since 3 subscribers are working at the same time, this means that work is shared among the subscribers thus theoretically dividing the time needed into 3, which makes handling all 25 messages faster, making the message queue grah a lot steeper.

To improve on the code (other tha just recommenting the code that makes the subscriber sleep for a second on every message) is to make each subscriber instance implement multi-threading. This would have a similar effect to spawning multiple instances of subscriber at a time, but it would not require you to open multiple consoles to run the separate subscriber instances. Plus it would work well even if you were to spawn multiple subscribers, because if we had 3 subscribers, each with a threadpool of 3 threads, that would mean that there could be 9 threads working on processing the message at the same time, which would be a lot faster than just 1 thread per subscriber instance. 

Other than performance, we may consider security. For example username and password of guest isn't the most secure combination out there. Also, if this was a public repository, we would make the URI a secret, so instead of a hard-coded RabbitMQ URI (which does contain the username and password for authentication) since it may allow outsiders to listen on our message queues, when they shouldn't. Furthermore, there is no error-handling within this code, thus we don't have anyway to handle failures other than the application going down/crashing.
