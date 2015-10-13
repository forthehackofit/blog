When yet another JS framework lurks around and everyone starts jumping on the buzz words bandwagon, you might rightfully sense that feeling of "déja vu" and can be excused when caught screaming out of your lungs "Not again !!!".

In the case of ES6 though, Javascript - the language, is the one evolving from its solid predecessor ES5 and you will be wrong to miss the early train on this one.

This post is the first in a series that will take on the goodies that ES6 has brought to the language. Instead of blindly copying and pasting the new syntax, we will ensure to understand how it improves what we grew accostumed to and how to take advaantage of it today in our production code.

Today we will be looking at a long time missing feature of the language: block scoping.

Let's tackle a (seemingly) simple program to get on with on with our learning. The problem is as follow:

Given a list of graduates, we are to write a simple program that write out the name of each graduate to the console every 5 seconds, mimicking someone reading from a list.

We will use an array to hold the list of graduates

<pre>
	var graduates = ["John", "Jane", "Omar", "Aicha", "Yasmine", 	"Mike", "Hope", "Zahra", "Alejandro", "Rosa"];
</pre>

and though we could use the map utility, in this instance we will use a for loop to underline the importance of block scope.

So the body of our for loop will look like 

<pre>
 for(var i = 0, j = graduates.length; i < j; i++) {
  // body of loop here
 }
</pre>

The intent of many that are new to Javascript and mostly those coming from another programing language is that the variables i and j are to exist only for the duration and will only exist within that scope. 
The problem is the for loop or all the other iterators and conditionals (if, else, while) in Javascript written in this form do not provide block scope. Instead the code above is interpreted like this below at runtime

<pre>
 var i; // both i and j are hoisted to the top and are now part of
 var j; // the parent scope in this case the window object
 
 for( i = 0, j = graduates.length; i < j; i++) {
  // body of the for loop
 }
</pre>

To verify that it is indeed what Javascript did tou our previous code at runtime let's log i out after the loop has finished running

<pre>
 for(var i = 0, j = graduates.length; i < j; i++) {
  // body of loop here
 }
 console.log(i);
</pre>

This will log out 10, the final value of i that did not satisfy the loop's condition. Once again if the loop provided scope like a function does in JS, then the log statement should've returned "undefined".

Now armed with that crucial piece of understanding let keep on moving with our solution. In the body of our for loop  we could already start printing out the name of the graduates by adding this:

<pre>
 for(var i = 0, j = graduates.length; i < j; i++) {
  // body of loop here
  console.log(graduates[i]);
 }
</pre>

This is a good first step since the names are printed on the console, but let's be realistic if the intent is to mimick someone reading out a list, there is no way one could shout out all the names this fast. To understand how fast it went let's use two consloe method around our loop

<pre>
 console.time("for_loop_speed);
 for(var i = 0, j = graduates.length; i < j; i++) {
  // body of loop here
  console.log(graduates[i]);
 }
 console.timeEnd("for_loop_speed");
</pre>

The output on the console tells us that it took 2.128ms for the loop to execute, that is 0.0021 seconds. I challenge anyone to read those 10 names that fast.

<pre>
for_loop_speed: 2.128ms
</pre>

Now let's add a timer to the loop so we can pace the output. The code below will seem like a logical thing to do at first:

<pre>
 console.time("for_loop_speed);
 for(var i = 0, j = graduates.length; i < j; i++) {
  // body of loop here
  setTimeout(function() {
   console.log(graduates[i]);
  }, 5000) 
 }
 console.timeEnd("for_loop_speed");
</pre>

There are several things to note from the output which is far from the intended result but which is correct fromthe standpoint of how Javascript works. 
First thing to notice is that it took the loop 0.138ms to complete

<pre>
 for_loop_speed: 0.138ms
</pre>

That's way faster than the previous iteration of the code. If we break down what is happening here it becomes pretty clear why this is happening. 
As long as i is less than j the body of the loop is executed. When the loop encounters the **setTimeout** construct it triggers a call to the **Event loop** that is now made aware something is asking to be executed in at least x number of milliseconds in this case 5000.  

As far as the loop is concerned, the moment it has trigger that event, it's done and will not sit and wait around for the body of the setTimeout to be executed, it's not it's responsibility, so it goes on to the next iteration.  

Now it should make sense that it can go pretty quickly in finishing all the 10 iterations before the first of the actions added to the Event loop gets to fire. Since the loop is already done, i is now equal to 10 so when we ask the program to output a name after 5 seconds we essentially asking it give us the value of graduates[10] which does not exist since the indexes of the array go from 0 to 9. 

To help you get a better grasp of the Event loop and how setTimeout is affected you can take a look at the quote below from the MDN website:

"Calling setTimeout will add a message to the queue after the time passed as second argument. If there is no other message in the queue, the message is processed right away; however, if there are messages, the setTimeout message will have to wait for other messages to be processed. For that reason the second argument indicates a minimum time and not a guaranteed time."

Read the full article here: https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop

## Let's fix our program
Ok now that we know what went wrong with our program let go ahead and fix it. What we need to do is to ensure that each iteration create a "unique" copy of i and that setTimeout at the time of that setup has access to the value of i. 

What our program is missing is an internal scope for i, and since functions are the constructs that creates it, we will give it just that.

<pre>
  for(var i = 0, j = graduates.length; i < j; i++) {
    delayShow(i);
  }
  
  function delayShow(index) {
	setTimeout (function() {  
     console.log(graduates[index]);
   }, index * 5000);
  }
</pre>

We have moved the body of our previous loop into a new function that is called **delayShow**. The reason why this works is that the anonymous function inside the setTimeout has **closure** over the enclosinhg **delayShow** function, so it remembers the value of i at the time that iteration ran. We will cover closures in other post but take a second to really look at this last piece of our program and do not move on until you understand it :)

## The let keyword
If you have read this far and grew impatient in seeing the ES6 part of this post, well the wait is over. Let's dive in. 
The main issue with our earier code was that the loop just did not provide a scope and both i and j declared with the **var** keyword got hoisted to the top.

ES6 introduces the **let** keyword that creates the intended behavior and makes any variable declared with it available only within it's enclosing block. So we could rewrite our loop this way:

<pre>
 for (let i = 0, j = graduates.length; i < j; i++) {
  delayShow(i);
 }

 function delayShow(index) {
	setTimeout (function() {  
     console.log(graduates[index]);
   }, index * 5000);
  }
  </pre>
  
This looks a lot like our non ES6 solution, but the cool part is that we no longer need the extra function just to add an additional scope. So the code above can be rewritten as

<pre>
 for(let i = 0, j = graduates.length; i < j; i++) {
 	setTimeout (function() {  
     console.log(graduates[i]);
   }, i * 5000);
 }
</pre> 

Since a new copy of i is saved with eacg iteration of the loop, every instance of the setTimeout construct remembers the value of i at that time and is able to display the intended and expected result.

One thing to note is that if you tried to log out the values of i or j outside the body of the loop like we did earlier they will throw an **Uncaught Reference Error** since they no longer exists outside of it.

<pre>
	for (let i = 0, j = graduates.length; i < j; i++) {
      // body of loop
    }
    console.log(i); // undefined
    console.log(j); // undefined
</pre>    

What you should remember is that the **let** keyword prevents variable hoisting outside of their enclosing block and create a scope for those variables within that block.

## Using ES6 Today
ES6 is very close to be finalized, but still you cannot just dump it as is in your Javscript and have it work out of the box in your browser. You will need the help of a tool to transpile - meaning converting ES6 to the old syntax that browsers understand. 

If I'm not mistaken I think **Traceur** was the first tool to do that, but like many in the community I have set my eyes on the wonderful **Babel** to take care of the job. Now **Babel** deserves and entire post to cover all the great things it can handle for you and I promise that I will cover in depth pretty soon. 

In the meantime, I highly recommend using **Scratch** a chrome extension that allows you to use ES6 in the Chrome Developer Tools and see the transpiled code that you could easily copy and paste into your code editor. Check the screenshot below

**put scratch image here **

and look at the video to see how I have used it for this exercise.

It was a long post, way longer than I anticipated but hopefully you found it useful and it got you motivated to jump on the ES6 train today. 

Let me know in the comments if you have any questions or suggestions. 


