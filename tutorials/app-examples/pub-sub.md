# Publish-Subscribe Messaging
The `PubSub` app, located in the LearnPubSub directory of [riaps-tutorials](https://github.com/RIAPS/riaps-tutorials), demonstrates how to use Publish Subscribe messaging

Publish-Subscribe, (a.k.a. pub-sub), is a one-way messaging pattern from N-to-M ports: publish, or **pub** ports, send messages, and subscribe, or **sub** ports, receive them. Connections between pub and sub ports happen via **topics**. 

Topics are a virtual[^1] message stream: pub ports send messages into the stream, while sub ports receive messages from it. 

The below RIAPS model snippet shows how each is declared:
```
message MsgTopic;
	
component Talker {
  pub pubPort : MsgTopic;
  ...
}

component Listener {
  sub subPort : MsgTopic;
  ...
}
``` 
A topic called `MsgTopic` is declared with the `message` keyword.  
A component uses the `pub` keyword to declare a publish port named `pubPort`. A colon separates the port name from the topic it will publish to, namely `MsgTopic`.  
Another component uses the `sub` keyword to declare a subscribe port named `subPort`. It subscribes to `MsgTopic`, and will receive messages published there[^1]. 

A publish port can be used almost anywhere in a component implementation. For example, the `Talker` component above can send a string using its `pubPort` by calling:
```python
my_msg = "Hello from Talker"
self.pubPort.send_pyobj(my_msg)
```
The message sent can be any [serializable](https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled) Python object. The `pubPort` will send the message to all connected sub ports, in this case `subPort` declared in the above `Listener` component. When the message is received by `subPort`, it triggers the poller in `Listener` to look for and run a function named `on_subPort`, like in the below:
```python
class Listener(Component):
    ...
    def on_subPort(self):
        msg = self.subPort.recv_pyobj()
```



[^1]: Under-the-hood, a topic is not a place in memory, but a string that sub ports use to look up pub ports that they should connect to. When a pub port publishes a message, it sends the message to each connected sub port, one-at-a-time. This means that when a topic has more than one publisher, the received message order is not necessarily the same for at all subscribers