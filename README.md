# openpromise


The goal is of this module is to simplify and extend the use of the native Promise mechanism. It includes variation objects that can be used to control the flow of a program or communicate between different parts of a program asynchronously.   These objects leverage the native promise mechanism while avoiding the convoluted semantic of Promise definition.  All objects are or behave like **Promise**s, can be chained like **Promise**s and allow the use of the ``then`` , ``catch`` and ``finally`` construct

The objects provided are:
|  |  |
|--|--|
| ``OpenPromise`` |  It is the base object in the module; it extends the native ** Promise** by making it possible to resolve it remotely, e.g., outside its body. Its use instead of the native version makes the code much clearer. 
|``Defer``| It is synonymous with``OpenPromise``  and is included for compatibility with an older version of the module. 
|``Delay``| is a **Promise** that resolves after a set time (it replaces the use of the **setTimeout** function)
|``TimeOut``| is a **Promise** that fails after a set time (it replaces the use of the **setTimeout** function)
|``Queue``|  provides an execution queue for functions and  **Promise**s
|``Cycle``| is a Promise-like object that can be retriggered multiple times and remotely. It can be used to communicate or trigger events asynchronously between different parts of a program.
|``Repeater``| is  Promise-like object that retriggers itself after an interval of time, it can be used instead of the **setInterval** function
|``Flipflop``|  Promise-like object acting as an on/off gate that and holding code execution
|``untilResolved``| Function repeatedly trying to obtain a resolved **Promise** from a **Promise** returning function. It can be used to call multiple times an async function  until successful; for example, to request a system resource
 


# 1. OpenPromise/Defer

## Usage

The constructor has no argument:  ```new OpenPromise()``` or ```new Defer()```

The constructor returns an object that is an actual **Promise** object that can be used in every way a  **Promise** can be used. 

The key methods are :

|  |  |
|--|--|
|```resolve(v)```| to pass a value and trigger execution of the **then** code.
|```reject(v)``` |to pass a value and trigger rejection of the **catch** code.
|```fail(v)``` |synonymous with ```reject(v)``` 
|```then(f)``` | code to be executed after resolution (native promise method)
|```catch(f)``` |code to be executed after rejection (native promise method)
|```finally(f)``` |code to be executed after resolution  or rejection (native promise method)


The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been resolved otherwise it is **undefined**
|```rejected``` |is **true** is the OpenPromise has been rejectd otherwise it is **undefined**


resolved


Please note that a rejected **OpenPromise** requires a ```catch(f)``` clause otherwise an unhandled promise rejection error will be generated  (because an **OpenPromise** is a real **Promise**!)

Obviously, any code written using an  **OpenPromise** could be written with a regular **Promise**; however, the normal  **Promise** syntax requires to embedded the resolution/rejection code within the code creating the Promise. Such code can become very convoluted. With **OpenPromise**, the creation, resolution and rejection are separate codes. 

### Example 1

```
var opm= require("openpromise")
var df = new opm.Defer()   
df.then((val)=>{console.debug("received",val)})
df.then(()=>{console.debug("resolved?",df.resolved)})
df.then(()=>{console.debug("rejected?",df.rejected)})
console.debug("resolved?",df.resolved)
console.debug("rejected?",df.rejected)
df.resolve(45)
```
#### Ouput
```
resolved? undefined
rejected? undefined
received 45
resolved? true
rejected? undefined
```

### Example 2
```
var opm= require("openpromise")
var df = new opm.Defer()

df.then((val)=>{console.debug("received",val)})
  .catch((val)=>{console.debug("failed",val)})
  .finally(()=>{console.debug("finally",)})
df.reject(99)
```
#### Output
```
failed 99
finally
```

# 2. Delay

**Delay** is a Promise that resolves itself after a delay.  It can be used instead of the **setTimeout**  function.

A **Delay**  is an **OpenPromise**   and thus also a **Promise**  

|  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|  |
|---|--|
|``new opm.Delay(d, v)`` | Constructor; **d** is the duration in milliseconds, **v** is the resolution value passed to the **then**. If **d** is omitted or if it is **null** or **undefined**, the duration is infinite and the **Delay** behaves like a regular **OpenPromise** (except that the duration can be set later with **reset**)
|``reset(d, val)``|Reinitializes the timer (only possible while it is still running)
 |```resolve(v)```| Forces resolution before the end of duration (inherited from  **OpenPromise**)
|```reject(v)``` |Forces rejection before the end of duration (inherited from  **OpenPromise**)
|```fail(v)``` |Forces rejection before the end of duration (inherited from  **OpenPromise**)
|```then(f)``` | code to be executed after resolution (inherited from  **Promise**)
|```catch(f)``` |code to be executed after rejection (inherited from  **Promise**)
|```finally(f)``` |code to be executed after resolution  or rejection (inherited from  **Promise**)


The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been resolved otherwise it is **undefined**  (inherited from  **OpenPromise**)
|```rejected``` |is **true** is the OpenPromise has been rejectd otherwise it is **undefined**  (inherited from  **OpenPromise**)

### Example 3

```
var opm= require("openpromise")
var dl = opm.Delay(1000,222)

dl.then((val)=>{console.debug("received",val)})
	.catch((val)=>{console.debug("failed",val)})
	.finally(()=>{console.debug("finally",)}))})
```

#### Output
```
received 222
finally
```


# 3. TimeOut

**TimeOut**   is a Promise that fails itself after a delay (unless it has been resolved before).  It can be used instead of the **setTimeout**  function.   

A **TimeOut**    is an **OpenPromise**   and thus also a **Promise**  

|  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|  |
|--|--|
|``new opm.TimeOut(d, v)`` | Constructor. **d** is the duration in milliseconds, **v** is the failure value passed to the **catch**. If **d** is omitted or if it is **null** or **undefined**, the duration is infinite and the **TimeOut** behaves like a regular **OpenPromise** (except that the duration can be set later with **reset**)
|``reset(d, val)``|Reinitializes the timer (only possible while it is still running)
 |```resolve(v)```| Forces resolution before the end of duration (inherited from  **OpenPromise**)
|```reject(v)``` |Forces rejection before the end of duration (inherited from  **OpenPromise**)
|```fail(v)``` |Forces rejection before the end of duration (inherited from  **OpenPromise**)
|```then(f)``` | code to be executed after resolution (inherited from  **Promise**)
|```catch(f)``` |code to be executed after rejection (inherited from  **Promise**)
|```finally(f)``` |code to be executed after resolution  or rejection (inherited from  **Promise**)

The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been resolved otherwise it is **undefined**  (inherited from  **OpenPromise**)
|```rejected``` |is **true** is the OpenPromise has been rejectd otherwise it is **undefined**  (inherited from  **OpenPromise**)

### Example 4

```
var opm= require("openpromise")
var to = new opm.TimeOut(1000,222)

to.then((val)=>{console.debug("received",val)})
	.catch((val)=>{console.debug("failed",val)})
	.finally(()=>{console.debug("finally",)})
```

#### Output
```
failed 222
finally
```

# 4. Queue

**Queue** is a very simple object that can be used to queue the execution of functions or promises. It is just a wrapper for an extensible chain of  ```promise.then(f) ```

Its constructor takes no parameter and it has only one method ```enQueue(f)``` 

```enQueue(f)``` adds **f** to the FIFO execution queue . If **f** returns a **Promise** or **f**  is a **Promise** , the queue will wait for the promise to be resolved or rejected. 

 ```enQueue(f)``` returns a **Promise** that is resolved or rejected when **f**  completes.    If **f**  is a function, the ** Promise**  resolves to the value returned by  **f**  or is rejected with the error code thrown by  **f** .  

 If  **f**  is a **Promise**,  the ** Promise**  returned by  ```enQueue(f)```  is resolved or rejected when  **f**  is resolved or rejected and with the same value.  

The execution of the Queue continues normally, even if one step fails. 

### Example 5 (basic) 
```
var opm= require("openpromise")
var qe = new opm.Queue()
qe.enQueue(()=>{console.debug("Do")})
qe.enQueue((s)=>{console.debug("Re")})
qe.enQueue(()=>{console.debug("Mi")})
```
#### Output
```
Do
Re
Mi
```


### Example 6  (complex)

The following example shows how ```enQueue()``` accepts a function that returns a promise and also returns a promise. 


```
var opm= require("openpromise")

var qe = new opm.Queue()

a = qe.enQueue(new opm.Delay(1000,1))
b = qe.enQueue(()=>{return new opm.TimeOut(1000,2)})
c = qe.enQueue(()=>{return 3})

a.then((x) => {console.log("success a",x)}).catch((x) => {console.log("failure a",x)})
b.then((x) => {console.log("success b",x)}).catch((x) => {console.log("failure b",x)})
c.then((x) => {console.log("success c",x)}).catch((x) => {console.log("failure c",x)})

```

`new opm.Delay(1000,1)`is a **Promise** that resolves to value **1** after one second. 
```()=>{return  new opm.TimeOut(1000,2)}```  is a function that returns a  **Promise** that rejects with value **2** 
after one second. 
`()=>{return  3}`is a plain function that  that return value **3** when executed
#### Output
```
[... one second wait ...]
success a 1   
[... one second wait ...]
failure b 2
success c 3
```

# 5. Cycle

Cycle is a Promise-like object that can be retriggered multiple times and remotely. It can be used to communicate or trigger events asynchronously between different parts of a program.  

## Usage

The constructor has no argument:  ```new opm.Cycle()```

Main methods :
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||
|---|--|
|``new opm.Cycle()`` | Constructor
|```thenAgain(f)``` |Sets **f** to be executed asynchronously every time the **Cycle** is retriggered. **thenAgain** returns a **OpenPromise**/**Defer** that is resolved or rejected when the Cycle is terminatedé
|```repeat(v)```|Re-triggers the cycle, **v** is passed to the excution functions 
|```thenOnce(f)``` |Set **f** to be executed asynchronously once the next time the **Cycle** is retriggered
|```terminate(f)``` |Disables the **Cycle**, resolves the **Cycle** and resolves all **OpenPromise**/**Defer**  created with ```thenAgain(f)```
|```fail(f)``` |Disables the **Cycle** , fails the **Cycle** and fails all **OpenPromise**/**Defer**  created with ```thenAgain(f)```

Other methods :
|  |  |
|---|--|
|```then(f)``` | code to be executed after termination(inherited from  **Promise**)
|```catch(f)``` |code to be executed after failure (inherited from  **Promise**)
|```finally(f)``` |code to be executed after terminationor or failure (inherited from  **Promise**)

The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been terminated otherwise it is **undefined**  (inherited from  **OpenPromise**)
|```rejected``` |is **true** is the OpenPromise has been failed otherwise it is **undefined**  (inherited from  **OpenPromise**)


### Example 7

```
var opm= require("openpromise")
var cy = new opm.Cycle()
cy.thenAgain((val)=>{console.debug("received",val)})
cy.repeat(101)
cy.repeat(102)
cy.repeat(103))
```
#### Ouput
```
received 101
received 102
received 103
```

## Termination

Each individual ```thenAgain(f)``` can be disabled separately, or the **Cycle** can be terminated as a whole. 

###  Termination of an individual  ```thenAgain(f)```
The ```thenAgain(f)``` method returns an **OpenPromise/Defer**  object. This object can be resolved by calling its ```resolve(v)```  method).  Such resolution of the Defer/OpenPromise stops all further triggering by the repeat function; it can also trigger the execution of a final cleanup/finalization function that would have been established with a ``then(f)`` attached to that **Defer/OpenPromise**. Other instances of the ```thenAgain(func)```  will continue to be triggered. 

### Example 8
```
var opm= require("openpromise")
var cy = new opm.Cycle() 
var df = cy.thenAgain((val)=>{console.debug("received",val)}) 		// df is a Defer object.
df.then((val)=>{console.debug("the Defer is resolved with value",val)}) // to be executed after resolution
cy.repeat(1)
setTimeout(
  ()=>{df.resolve(99) 								// the Defer is resolved
  cy.repeat(2)
  },100)
```
The timer is needed to ensure that ```cy.repeat(1)``` is fully executed before ```df.resolve(99)``` is executed. It is not necessarily the case without the timer because ```cy.repeat(1)``` is executed asynchronously (see example 9 for behavior without the timer). 

#### Output
```
received 1
the Defer is resolved with value 99
```


###  Termination of a Cycle

The Cycle object also has a ```terminate(val)``` method. This method disables the Cycle, and all future calls to ```repeat(v)``` will be ignored.  Also, all Defer objects created with the ```thenAgain(func)```  will be resolved with the value passed to terminate.

Please note that a **Cycle** object is really an **OpenPromise** / **Defer**  object that is resolved when it is terminated, so one can use the **then** construct to execute a finalization code just after a **Cycle** is terminated. 

It is recommended to terminate every Cycle that is created to avoid memory leaks; otherwise, all the listening thenAgain will remain in memory. Only the Cycle needs to be terminated. 

### Example 9
```
var opm= require("openpromise")
var cy = new opm.Cycle()
cy.then((val)=>{console.debug("the Cycle is terminated with value",val)})
var df = cy.thenAgain((val)=>{console.debug("received",val)}) // df is a Defer object.
df.then((val)=>{console.debug("the Defer is resolved with value",val)}) // to be excuted after resolution
cy.repeat(1)
cy.terminate(99) // the Cycle is terminated
cy.repeat(2)
```
A timer is not necessary as in example 6 because calls to  ```cy.repeat(val)``` and ```cy.terminate(val)``` are queued and are always executed asynchronously, but in the order, they were made.  

#### Output
```
received 1
the Cycle is terminated with value 99
the Defer is resolved with value 99
```


### Example 10 (complex)

```
var opm= require("openpromise")
var timer
var cy = new opm.Cycle()
var df1 = cy.thenAgain((val)=>{console.debug("1 received",val)})
df1.then((val)=>{console.debug("1 is terminated with value",val)})
var df2 = cy.thenAgain((val)=>{console.debug("2 received",val)})
df2.then((val)=>{console.debug("2 is terminated with value",val)})
var df3 = cy.thenAgain((val)=>{if(val>=3) {df1.resolve(100)}})
var df4 = cy.thenAgain((val)=>{if(val>=5) {
  cy.terminate(999)
  clearInterval(timer)
  }})

let n = 1
timer = setInterval(()=>{cy.repeat(n)
			n = n +1
			}
			,100)
```
#### Ouput
```
1 received 1
2 received 1
1 received 2
2 received 2
1 received 3
2 received 3
1 is terminated with value 100
2 received 4
2 received 5
2 is terminated with value 999
```

## Timing of execution

Calls to methods on a Cycle object can occur asynchronously anywhere in the code where this Cycle object is available.
The only guarantee is that the execution of all ```cy.repeat(val)```, ```cy.terminate(val)``` and```cy.fail(val)``` calls will be queued and executed successively in the order they were made. A call to ```df.resolve(val)``` (where ```df = cy.thenAgain(f)```) is not queued and  ```cy.thenAgain(f)``` can be resolved/disabled before all pending values  from ```cy.repeat(val)``` are treated. 

### Example 11
```
var opm= require("openpromise")
var cy = new opm.Cycle()
var df = cy.thenAgain((val)=>{console.debug("received",val)}) 		// df is a Defer object.
df.then((val)=>{console.debug("the Defer is resolved with value",val)}) // to be excuted after resolution
cy.repeat(1)
df.resolve(99) 								// the Defer is resolved
cy.repeat(2)
```
#### Output
```
the Defer is resolved with value 99
```
Because ```cy.repeat(1)``` is queued and executed asynchronously, ```df ``` is resolved before the value ```1``` is treated, and the value is ignored.

## Failure
### fail(val)

The ```fail(val)``` method of a Cycle object triggers the failure of all Defer objects derived from it with ```thenAgain(f)```. Since these Defer objects are Promises, failure can be caught with ``catch(f)```.
A call to ```fail(val)``` also disables the Cycle, and all future calls to ```repeat(v)``` will be ignored.  

### Example 12
```
var opm= require("openpromise")

var cy1 = new opm.Cycle()
cy1
	.then((v)=>{console.debug("cy1 terminated",v)})
	.catch((v)=>{console.debug("cy1 failed",v)})

cy1.thenAgain((v)=>{console.debug("thenAgain1",v)})
	.then((v)=>{console.debug("then1",v)})
	.catch((v)=>{console.debug("catch1",v)})


cy1.repeat(10)
cy1.terminate(11)


var cy2 = new opm.Cycle()
cy2
	.then((v)=>{console.debug("cy2 terminated",v)})
	.catch((v)=>{console.debug("cy2 failed",v)})

cy2.thenAgain((v)=>{console.debug("thenAgain2",v)})
	.then((v)=>{console.debug("then2",v)})
	.catch((v)=>{console.debug("catch2",v)})

cy2.repeat(20)
cy2.fail(21)
```
#### Output
```
thenAgain1 10
thenAgain2 20
cy1 terminated 11
then1 11
cy2 failed 21
catch2 21
```

# 6. Repeater

**Repeater**   is a Promise-like object that retriggers itself repeatedly after a set time.  It can be used instead of the **setInterval** function. It is derived from **Cycle** and behaves in a similar fashion.


Main methods :
|  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|  |
|---|--|
|``new opm.Repeater(d,f)`` | Constructor; ``d`` is the timer delay, ``f`` is an optional function whose value will be calculated at each cycle and passed as a resolution value
|```thenAgain(f)``` |Sets **f** to be executed asynchronously every time the **Repeater** is retriggered. **thenAgain** returns a **OpenPromise**/**Defer** that is resolved or rejected when the **Repeater**is terminated
|```thenOnce(f)``` |Set **f** to be executed asynchronously once the next time the **Repeater** is retriggered
|```terminate(f)``` |Disables the **Repeater**, resolves the **Repeater** and resolves all **OpenPromise**/**Defer**  created with ```thenAgain(f)```
|```fail(f)``` |Disables the **Repeater** , fails the **Repeater** and fails all **OpenPromise**/**Defer**  created with ```thenAgain(f)```

Other methods :
|  |  |
|---|--|
|```then(f)``` | code to be executed after termination(inherited from  **Promise**)
|```catch(f)``` |code to be executed after failure (inherited from  **Promise**)
|```finally(f)``` |code to be executed after terminationor or failure (inherited from  **Promise**)

The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been terminated otherwise it is **undefined**  (inherited from  **OpenPromise**)
|```rejected``` |is **true** is the OpenPromise has been failed otherwise it is **undefined**  (inherited from  **OpenPromise**)

### Example 13

```
var n = 1

f = ()=>{return n++}
var rp = new opm.Repeater(500,f)
rp.thenAgain((v)=>{console.debug("received",v)})
rp.then(()=>{console.debug("terminated")})
dl = new opm.Delay(5000)
dl.then(()=>{rp.terminate()})
```

#### Output
```
received 1
received 2
received 3
received 4
received 5
received 6
received 7
received 8
received 9
terminated
```


# 7. Flipflop

**Flipflop**   is a Promise-like object that can act as a resettable program gate that can be opened, closed, or flipped. 
|  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|  |
|--|--|
|``new opm.Flipflop()`` | Constructor
 |```on(v)```| Resolve the object (open the gate)
|```off(v)``` |Reset the object (close the gate)
|```flip(v)``` |Switch state between opened and closed
|```fail(v)``` | Reject/fail the object 
|```then(f)``` | code to be executed as soon as the object is on 
|```catch(f)``` |code to be executed in case of failure
|```finally(f)``` |code to be executed as soon as the object is on or in case of failure

The object also exposed the properties.
|  |  |
|--|--|
|```resolved```| is **true** is the OpenPromise has been resolved otherwise it is **undefined**  (inherited from  **OpenPromise**)
|```rejected``` |is **true** is the OpenPromise has been rejectd otherwise it is **undefined**  (inherited from  **OpenPromise**)

### Example 14

```
var opm= require("openpromise")
  
var ff = new opm.Flipflop()
ff.then(()=>{console.debug("Hello 1")})

console.debug(">setting on")
ff.on()
console.debug(">flipping")
ff.flip()
ff.then(()=>{console.debug("Hello 2")})
console.debug(">flipping again")
ff.flip()
console.debug(">and setting off")
ff.off()
ff.then(()=>{console.debug("Hello 3")})
```

#### Output
```
>setting on
>flipping
>flipping again
>and setting off
Hello 1
Hello 2
```
Note that the execution is asynchronous


# 8. untilResolved

**untilResolved** returns an **OpenPromise** representing the result of repeatedly calling an async function until it returns a **Promise** that resolves successfully. A typical scenario would to repeatedly attempt to open a communication sockets or a file until success. 

Practically,  **untilResolved**  returns **OpenPromise**  and then calls the async  function. It then waits for the Promise returned by the async function to be resolved or rejected. If the Promise from the async function is rejected,  **untilResolved**  calls the function again. If the obtained a **Promise** resolves successfully, the Promise  **untilResolved** had return also resolves. 

The call format is the following: **untilResolved(f, n, d)**
|  ||
|--|--|
|``f`` | The asynchronous function to be called, this function should return a **Promise**
 |```n```| an optional integer representing the maximum number of attempts to be made. If omitted, there is no limit to the number of repetitions.
|```d``` |an optional delay in milliseconds between successive calls. If omitted, there is no timeout. 
|```return``` |an **OpenPromise** representing the outcome of the process. The Promise resolves when a call to ``f`` succeeds and fails (is rejected) of ``n`` or ``d`` is exceeded.


### Example 15
```
var opm= require("./openpromise.js")

// the following code creates a test function foo that for 2 seconds returns Promises that fail with 100 milliseconds, after which it returns promises that resolve successfully. 
  
var dl= new opm.Delay(2000)
var n = 1
foo = ()=>{console.debug(n++);
	return dl.resolved ? new opm.Delay(100) : new opm.TimeOut(100)}

 op = opm.untilResolved(foo,100,200)
 op 
	 .then(()=>{console.debug("done")})
	 .catch(()=>{console.debug("give up")})
```
# 9. Acknowledgment

The code of the Defer object is based on the Defer() function proposed by **Carter** in this post:

https://stackoverflow.com/questions/26150232/resolve-javascript-promise-outside-function-scope