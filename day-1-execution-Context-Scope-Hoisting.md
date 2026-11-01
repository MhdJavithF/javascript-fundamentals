Exercise 1 — Predict the output
console.log(a);

var a = 10;

function test() {
  console.log(a);

  var a = 20;

  console.log(a);
}

test();

console.log(a);

Answer these questions:
1. What are the four outputs, in order? undefined, undefined, 20, 10 
2. Why doesn't the first statement inside test() necessarily access the global a? Because the binding of the varaible will check the closest declaration first in block scope then functional and global scope.
3. Which declaration determines the scope of the a inside test()? its a functional scope of var a inside the test()
Don't just give me the output. Explain your reasoning.
Exercise 2 — Identify the error
function example() {
  console.log(count);

  let count = 5;

  console.log(count);
}

example();

Answer:
1. Does either console.log execute successfully? Yes, the 2nd log will execute successfully and print 5
2. What is the exact reason for the error? The 1st log is trying to access the variable before declaration and for let/const the declaration will not hoist to top. the region between the accessing and decaration is called TDZ.
3. Is your explanation different from saying "let is not hoisted"? If so, how? for let/const the declaration will not hoist to top. the region between the accessing and decaration is called TDZ.
Exercise 3 — Lexical scope versus caller scope
const value = "global";

function first() {
  const value = "first";

  second();
}

function second() {
  console.log(value);
}

first();

// Execution context:

function first() {
  const value = "first";

  second();
}

function second() {
  console.log(value);
}

const value = "global";

first();

Predict the output. 'global'

Then explain this question:

If second() is called from inside first(), why doesn't it automatically access first()'s local value? Even the second is called inside the first() th binding of second() func is binded with global object scope.

This is a critical test of lexical scoping.
Exercise 4 — Coding challenge
Implement a function called createCounter using JavaScript.

Requirements:
- The function takes an initial number.
- It returns another function.
- Each time the returned function is called, it increments the counter by one and returns the updated value.
- Each counter instance must maintain its own independent state.
- Do not use a global variable.
Example of the required behavior:
const counterA = createCounter(0);

console.log(counterA()); // 1
console.log(counterA()); // 2
console.log(counterA()); // 3

const counterB = createCounter(10);

console.log(counterB()); // 11
console.log(counterA()); // 4
console.log(counterB()); // 12

Your task is to implement createCounter yourself.

// code

function createCounter(num) {
  let start = num;
  return function(){
    return ++start;
  }
}

Do not worry about making the solution fancy. Focus on correctness and explain how the function maintains its state between calls.

We'll use this challenge to assess your understanding of closures.

Firstly I struggled and got type error like counterA not a function
for this code:
function createCounter(num) {
  let start = num;
  return num++;
}
Later from the error I thought to return as function and then got the solution. NO AI/outsource just thinks from the error.

// explanation

first i created function createCounter(num) { sicen e need to pass argument in the function. then for every call it should retunn a func with +1 incremental value from then start/previous value. whenever the already created function get called it will remember the previous value and reutn +1.

how its remembring is still im bit confused. If my answer is right/wrong give me the easy simple explanation how it remember the previous value.
also we are not passing any number inside counterB(previou) like this then how it remembers the previous value properly.


## Part 6. Senior-level follow-up
Once you've completed the exercises, I want you to answer one practical question.
Scenario: Debugging a frontend application
Imagine you're debugging a React application.

A function reads a variable, but it appears to use an unexpected value. The issue occurs only after a particular sequence of interactions.

You suspect the problem might involve scope or a closure. // this itself hint for me. really a good question let me try my best also im eagger to know the steps and anwser for this one.

How would you investigate the problem?

Structure your answer around these points:
1. What would you inspect first? For eg if it occus in button clicking, I will inspect and see the source tab and which funtion is tryig to access.
2. How would you determine which variable binding the function accesses? May be adding a breakpoint will helps?
3. How would you distinguish a closure-related problem from a React state-update or stale-closure problem? As per my knowledge when we are updating state we should use prev state value instading writing empty (). 
4. What would you change to fix it, and how would you verify the fix? Don't know