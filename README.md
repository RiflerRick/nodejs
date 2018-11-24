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