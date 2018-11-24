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
 