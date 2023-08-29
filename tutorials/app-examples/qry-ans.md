# Query-Answer Example App
The `QryAns` app, located in the QryAns directory of [riaps-tutorials](https://github.com/RIAPS/riaps-tutorials), demonstrates how to use Query-Answer messaging. It can be run on a single RIAPS VM node. 

# About Query-Answer Messaging
Query-Answer, (a.k.a. qry-ans), is a bidirectional messaging pattern where a query port sends a message to an answer port, which can then send a message back.

The below RIAPS model snippet shows how each is declared:
```
message QueryTopic;
message AnswerTopic;
	
component Foo {
  qry queryPort : (QueryTopic,AnswerTopic);
  ...
}

component Bar {
  ans answerPort : (QueryTopic,AnswerTopic);
  ...
}
``` 
Topics for each direction of communication, called `QueryTopic` and `AnswerTopic`, are declared with the `message` keyword.  
A component uses the `qry` keyword to declare a query port named `queryPort`. A colon separates the port name from a tuple of the topic names it will send messages to and receive messages from, `QueryTopic` and `AnswerTopic`, in parenthesis.  
Another component uses the `ans` keyword to declare an answer port named `answerPort`. It connects to the same topics in the same order as its associated query port(s).

A query port can be used almost anywhere in a component implementation. For example, the component `Foo` above can send an integer using its `queryPort` by calling:
```python
my_msg = 3
self.queryPort.send_pyobj(my_msg)
```
The message sent can be any [serializable](https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled) Python object. The `queryPort` will send the message to an answer port, in this case `answerPort` declared in the above `Bar` component. When the message is received by `answerPort`, it triggers the poller in `Bar` to look for and run a function named `on_answerPort`, like in the below:
```python
class Bar(Component):
    ...
    def on_answerPort(self):
        qry_msg = self.answerPort.recv_pyobj()
        ...
```
Typically, within the `on_answerPort`, some some action is taken based on the message that was received, and we would like to send some kind of information back to whoever sent the "query". This is most easily done by re-using the `answerPort` in the same invocation of `on_answerPort`, like below:
```python
class Bar(Component):
    ...
    def on_answerPort(self):
        qry_msg = self.answerPort.recv_pyobj()
        ...
        # Perform some calculation or retrieve some data
        ans_msg = "Some data"
        self.answerPort.send_pyobj(ans_msg)
```
The `answerPort` will send the "answer" message back to the query port who last contacted it. In this example, the `queryPort` on component `Foo`. When the `queryPort` receives said message, it alerts the poller in `Foo`, which then looks for a method named `on_queryPort`, like in the below:
```python
class Foo(Component):
    ...
    def on_queryPort(self):
        ans_msg = self.queryPort.recv_pyobj()
        # ans_msg contains the string "Some data"
```