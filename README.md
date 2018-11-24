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
- [Philip Roberts JSConfEU](https://www.youtube.com/watch?v=8aGhZQkoFbQ&vl=en) 
- [Bryan Hughes NodeJs Interactive North America](https://www.youtube.com/watch?v=qV8FgRt-HJI)
- [Sam Roberts NodeJs Interactive North America](https://www.youtube.com/watch?v=P9csgxBgaZ8)

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

