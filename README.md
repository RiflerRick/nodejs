## Javascript V8 Engine(on chrome)
Javascript is single threaded and it can literally do only one thing at a time. Event driven 
 Essentially the V8 runtime has the following components:
 - Heap: used for allocation of memory
 - Call stack: Just like the stack in any other programming language
 - WebAPIs: Extra features(essentially functions) that chrome gives us
 - Task Queue: Async calls are pushed to the task queue once they are completed.
 - Eventloop: Responsible for pushing to tasks from the task queue to the call stack once the call stack is empty.
 
 ### Typical Example
 ```javascript
console.log("Hi");

setTimeout(function cb(){
    console.log("there");
}, 5); // would execute after 5 seconds atleast. Note that 5 is the minimum amount of time to wait, not the absolute amount of time

console.log("Helloworld!");
``` 
In the above example the first line is a blocking call and it gets loaded onto the call stack first(after `main()` ofcourse). 
The setTimeout function is actually part of chrome WebAPIs, it is an asynchronous function and whenever it is first loaded onto
the call stack, it immediately goes into the context of the web apis. It is here that it spends 5 seconds(as given in the example),
Meanwhile the execution the program continues. The last statement gets loaded onto the stack and gets executed as it is another 
blocking operation. 
Once the execution of the setTimeout function is done, it gets pushed to the task queue. From here the eventloop is responsible for pushing
the tasks on the task queue to call stack once the call stack is empty and only when the call stack is empty. 

Another example can be the following:
```javascript
console.log("Hi");

setTimeout(function cb(){
    console.log("there");
}, 0);

console.log("Helloworld!");
```
The `setTimeout` function with 0 seconds actually might be used to defer the execution of the callback(inside setTimeout) to a later time.

```javascript
console.log("Hi");

$.get('url', function cb(data){ // similar example except we are making an ajax request here using jquery
   console.log(data);    
});

console.log("Helloworld!");
```
 Yet another example can be the following:
 
```javascript
console.log("started");

$.onclick('button', function cb() { // unlike the setTimeout function which goes out of the context of webAPIs once the execution of the function
    // is complete, the onclick function actually stays in the context of the webAPIs forever and listens for the click event
  console.log("Clicked");
})

setTimeout(function onTimeout(){
    console.log("Timeout finished!!!");
}, 5000);

console.log("Done");
```
Callbacks
```javascript
[1,2,3,4].forEach(function(i) { // this is not an async function, the forEach function actually runs synchornously
  console.log(i);
});

//async
function asyncForEach(array, cb) { //takes the array and the callback and is responsible for essentially pushing each of the callback functions
    // to the webapi context. 
  array.forEach(function (){
      setTimeout(cb, 0);
  })
}

asyncForEach([1,2,3,4], function(i) {
    // this functon is indeed the callback function, that will get executed at each iteration. This part will happen inside the webAPI context 
    console.log(i);
})
```

So in this case at each iteration, if the processing that we are doing is slow then the entire execution incase of the async version will happen faster.

*FunFact: The browser would try to repaint the page every 16.6 ms(60fps). But it is constrained by what you are doing in javascript so obviously it cannot render if there is still code getting executed in the stack. There actually exists a different queue called the render queue and that is given more priority than the callback/task queue to push onto the call stack*

For more on eventloops have a look at these links:
- [Philip Roberts JSConfEU: V8](https://www.youtube.com/watch?v=8aGhZQkoFbQ&vl=en) 
- [Bryan Hughes NodeJs Interactive North America: NodeJS not so single threaded](https://www.youtube.com/watch?v=qV8FgRt-HJI)
- [Sam Roberts NodeJs Interactive North America: Eventloop inside out](https://www.youtube.com/watch?v=P9csgxBgaZ8)
- [Bert Belder NodeJS Interactive Europe: Everything you need to know about the eventloop](https://www.youtube.com/watch?v=PNa9OMajw9w)
- [Daniel Khan Blog on Event loop and its metrics](https://www.dynatrace.com/news/blog/all-you-need-to-know-to-really-understand-the-node-js-event-loop-and-its-metrics/#disqus_thread)
- [Guy Podjarny NodeJS Interactive Norht America: Security vulnerabilities](https://www.youtube.com/watch?v=QSMbk2nLTBk)

Blogs:

- [Oluwaseun Omoyajowo Walking inside the event loop](https://medium.freecodecamp.org/walking-inside-nodejs-event-loop-85caeca391a9)

Doc:

- [Nodejs Eventloop official](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

### Inside the Event loop:

![Nodejs event loop](https://raw.githubusercontent.com/RiflerRick/nodejs/master/static/Screenshot%20from%202018-11-25%2000-06-01.png)

There are essentially 4 important phases in the nodejs eventloop. **timers, poll, check and close callbacks**. Each phase of is associated with queue of callbacks and each phase is supposed to do its own operation after which execute all callbacks in its queue.
Phases:
- **Timers**: responsible for executing callbacks scheduled by setTimeout and setInterval.
- **Poll**: This is a critical phase and really the phase where most of the work happens. 
The poll phase has two main functions:
Calculating how long it should block and poll for I/O, then
Processing events in the poll queue.
When the event loop enters the poll phase and there are no timers scheduled, one of two things will happen:

If the poll queue is not empty, the event loop will iterate through its queue of callbacks executing them synchronously until either the queue has been exhausted, or the system-dependent hard limit is reached.

If the poll queue is empty, one of two more things will happen:

If scripts have been scheduled by setImmediate(), the event loop will end the poll phase and continue to the check phase to execute those scheduled scripts.

If scripts have not been scheduled by setImmediate(), the event loop will wait for callbacks to be added to the queue, then execute them immediately.

Once the poll queue is empty the event loop will check for timers whose time thresholds have been reached. If one or more timers are ready, the event loop will wrap back to the timers phase to execute those timers' callbacks.

- **check**: 
This phase allows a person to execute callbacks immediately after the poll phase has completed. If the poll phase becomes idle and scripts have been queued with setImmediate(), the event loop may continue to the check phase rather than waiting.

setImmediate() is actually a special timer that runs in a separate phase of the event loop. It uses a libuv API that schedules callbacks to execute after the poll phase has completed.

Generally, as the code is executed, the event loop will eventually hit the poll phase where it will wait for an incoming connection, request, etc. However, if a callback has been scheduled with setImmediate() and the poll phase becomes idle, it will end and continue to the check phase rather than waiting for poll events.

- **close callbacks**:
If a socket or handle is closed abruptly (e.g. socket.destroy()), the 'close' event will be emitted in this phase. Otherwise it will be emitted via process.nextTick().

Nodejs has an underlying layer of c++ code that runs beneath it. When we run node synchornously node will always run the operation in a single thread which is called the main thread. However when an async function is written in node, the C++ APIs will be used in order to do one of 2 things. When node is started it is always started with a thread pool of 4 threads by default(although this can be changed obviously). Depending on the kind of task(network based, file IO, unix/file sockets ...), node may either try to allocate one of the threads from its thread pool(if the thread is free) to handle the async operation or it may assign that task to C++ async. All this detail is obviously abstracted away from the developer whose purpose in life is essentially to get the application running.   

### Node Event loop inside out
```
int s = socket(); // basically a socket system call
```
The socket system call returns an integer and it is called a file descriptor. File descriptors in general in unix may refer to an actual file descriptor of a file or it may refer to a socket. A file descriptor is basically an integer offset of an array that is kept in the kernel. Every process has an array of file descriptors and that array then has a pointer to an object which is the open control block for whichever resource is attached to the file descriptor and that has a pointer to a virtual function table(*note: whenever a class defines a virtual function, most compilers add a hidden member variable to that class that points to an array of pointers to virtual functions called the virtual method table. A vitual function is basically one which can be overriden in a subclass and essentially manifests in the form of dynamic dispatch or runtime polymorphism*). So in this context it allows for files, sockets and the like to all support **READ** however read does  different things based on what the underlying resource is for instance reading a file is different from reading a socket stream. 

```
// pseudo code
int server = socket();
bind(server, 80);
listen(server); // until listen is called the socket can be used both for accepting connections as well as for making connections

while (int connection == accept(server)) {
    pthread_create(echo, connection);    
}
```
accept is a blocking call. Whenever we call accept it is going to wait for connections and it is literally going to sit there, however once we actually get a connection if we wanted to do something with it then in the time we are doing, we would not be able to do accept another connection so one way to handle it would be to create a thread for every connection. Thats where the pthread_create function is coming in however starting a thread for every single connection is a rather expensive idea beacause a thread is a relatively heavy resource. All we really need to know for communicating over the socket is essentially the file descriptor aka socket descriptor and what we intend to do with that connection. Using a thread for each and every connection is way too resource intensive. 

A better way to do this would be to use epoll. Epoll is a system call that is made to the unix kernel that we can use to ask the kernel to tell us when some relevant happens, for instance we can ask the epoll object to tell the main program when there is a TCP connection made.

Note: Complete understanding of the below pseudo code is not relevant, following the comments is enough
```
// So lets say we have a server socket, the only socket and the only thing we are interested in is that if there is a TCP connection on the 
// relevant port.
// this example has been taken from https://www.youtube.com/watch?v=P9csgxBgaZ8
int server = ...//like before
int enentfd = epoll_create(10);
struct epoll_event_ev = {.events = EPOLLIN, .data.fd = server } // this is essentially he epoll descriptor
epoll_ctl(epollfd, EPOL_CTL_ADD, server, &ev); // here we add the socket descriptor to the epoll loop and tell epoll what we are interested in 
while((int max = epoll_wait(eventfd, events, 10))) 
{
//epoll wait is again going to block on the kernel. The moment we write epoll_wait, we are 
// going to get descheduled and when there is something to show then epoll will tell the main thread

    for (n=0;n<max;n++){
        if(events[n].data.fd == server) {// this is for accepting TCP connections
            //server socket has connection
            int connection = accept(server);
            ev.enents = EPOLLIN; ev.data.df == connection;
            epoll_ctl(eventfd, EPOLL_CTL_ADD, connection, &ev)
        }
        else{
            //connection socket has some data associated with it. This can only happen when a tcp connection has been established
            char buf[4096];
            int size = read(connection, buffer, sizeof buf);//read data out from the connection
        }
    } 
```
So in essence epoll over here is essentially an abstraction for us that is going to ask the kernel to report back to the program when something interesting happens, something interesting that we are going to define. In the above pseudo code we saw that epoll can report to us both when we are getting a connection(accepting the connection), as well as when a data is coming to that socket. 

The above pseudo code is essentially what a nodejs eventloop is. It is semi infinite loop polling and blocking on the O/S until something interesting(that again we define) happens. In the first part of the readme, there was mention of WebAPIs, in case of nodejs these are c++ apis and when an event is registered with the eventloop, c++ would be doing the work in the background and the eventloop would wait for the event from c++.

The event loop actually runs in that same thread where the rest of the code runs. 

The obvious question that one might ask here is that what happens if there is nothing to wait on the epoll loop(event loop). The answer is that node would just exit. So in case during debugging of an application if we find that node is or is not exiting, it is probably because there is nothing to wait on in the event loop. 
In cases where there is a resource which the event loop is waiting on it is possible to `unreference` the resource using `.unref()`.  

What is important for a developer to understand though is that every async call in nodejs does not use the epoll function internally. If it cannot use epoll, it would fallback to the threadpool of nodejs and execute the operation in a thread. 
Everything inside node is not epollable. 
- sockets(net/dgram/http/tls/https/child_process pipes/stdin, stdout, stderr): directly pollable
- time(something like `poll(..., int timeout)` or `epoll_wait(..., int timeout, ...)`): Only one timeout at a time is epollable. nodejs would keep all times sorted from the nearest time to the farthest time. So after one timeout finishes, the next one would kick in.
- File system: not epollable. file system operations will always happen in the thread pool. 
- dns: something like a dns.lookup() call actually calls getaddrinfo() in order resolve the domain name. getaddrinfo() happens in the threadpool and everything else happens using epoll
- signals: directly pollable
- child processes: directly pollable

So to summarize the following uses a threadpool:
- fs
- dns(only for dns lookups)
- crypto
- http.get/request() (if called with a hostname or domain name. then it has to do a dns lookup which is will happen in a threadpool)

### Metric to check performance
**loop time**: should finish within 8-12 milliseconds ideally. If it is taking more time, it is possible that you are doing some CPU intensive operations or are blocking.

### Addressing Security vulnerabilities

[Open source security platform](https://snyk.io/)
[Snyk quick start](https://www.youtube.com/watch?v=l1MT5lr4p9o)