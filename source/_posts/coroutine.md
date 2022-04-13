---
tags: Generator, Coroutines, Python
title: Review of How the heck does async/await work in Python 3.5? 
---
## Introduction

This is a review of [How the heck does async/await work in Python 3.5?](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/)
## Generator and Coroutines

My understanding of Generator before reading the article has been something like this:
```python=
def lazy_range(up_to):
    """Generator to return the sequence of integers from 0 to up_to, exclusive."""
    index = 0
    while index < up_to:
        yield index
        index += 1
        
for i in lazy_range(5):
    print(i)
```
> from https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/

It saves memory so that each time it only generates one number and suspend there to come back until for loop call `__next__()` and eventually throw StopIteration which for loop would capture.

Which fits the definition from wikipedia,
>"Coroutines are computer program components that generalize subroutines for nonpreemptive multitasking, by allowing multiple entry points for suspending and resuming execution at certain locations"

or in python documentation,
```
Coroutines are a more generalized form of subroutines. Subroutines are entered at one point and exited at another point. Coroutines can be entered, exited, and resumed at many different points. They can be implemented with the async def statement. See also PEP 492.
```

But since PEP342, we have send() method so we can send values to where the generators paused;

```python=
def jumping_range(up_to):
    """Generator for the sequence of integers from 0 to up_to, exclusive.

    Sending a value into the generator will shift the sequence by that amount.
    """
    index = 0
    while index < up_to:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump


if __name__ == '__main__':
    iterator = jumping_range(5)
    print(next(iterator))  # 0
    print(iterator.send(2))  # 2
    print(next(iterator))  # 3
    print(iterator.send(-1))  # 2
    for x in iterator:
        print(x)  # 3, 4
```
> take from https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/

## yield from example
```python=
def bottom():
    # Returning the yield lets the value that goes up the call stack to come right back
    # down.
    return (yield 42)

def middle():
    return (yield from bottom())

def top():
    return (yield from middle())

# Get the generator.
gen = top()
value = next(gen)
print(value)  # Prints '42'.

# if we call below, bottom() would call return which raised StopIteration since it is a generator
# and yield from would propagate the StopIteration back
# value = next(gen)
# print(value)  # Prints '42'.


try:
    value = gen.send(value * 2) 
    # since top paused at calling yield from middle() again
    # it becomes return 84 which return StopIteration object
    #  with attribute value = 84
except StopIteration as exc:
    value = exc.value
print(value)  # Prints '84'.
```

## Event 

Going back to Wikipedia, an event loop "is a programming construct that waits for and dispatches events or messages in a program"

Python adopts event loop in the form of `asyncio`

## asyncio example
```python=
# python3.4
import asyncio

# Borrowed from http://curio.readthedocs.org/en/latest/tutorial.html.
@asyncio.coroutine
def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        yield from asyncio.sleep(1)
        n -= 1

loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(countdown("A", 2)),
    asyncio.ensure_future(countdown("B", 3))]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

event loop here would monitor the asyncio.Future object e.g. asyncio.sleep(1)

In python 3.5 we can use `@types.coroutine` to label a generator as coroutine
```python=
import types
import asyncio
# Borrowed from http://curio.readthedocs.org/en/latest/tutorial.html.

@types.coroutine
def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        yield from asyncio.sleep(1)
        n -= 1

loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(countdown("A", 2)),
    asyncio.ensure_future(countdown("B", 3))]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

or use async/await, you need to `await` an `awaitable object` (coroutines and other objects that defines an __await__())

```python=
import asyncio
# Borrowed from http://curio.readthedocs.org/en/latest/tutorial.html.


async def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        await asyncio.sleep(1)
        n -= 1

loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(countdown("A", 2)),
    asyncio.ensure_future(countdown("B", 3))]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

The latter is async-based coroutine and the former ones are generator-based courtine and only generator-based coroutines.

>One very key point I want to make about the difference between a generator-based coroutine and an async one is that only generator-based coroutines can actually pause execution and force something to be sent down to the event loop.
>https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/


```python=
# from https://github.com/python/cpython/blob/3.6/Lib/asyncio/tasks.py
@coroutine
def sleep(delay, result=None, *, loop=None):
    """Coroutine that completes after a given time (in seconds)."""
    if delay == 0:
        yield
        return result

    if loop is None:
        loop = events.get_event_loop()
    future = loop.create_future()
    h = future._loop.call_later(delay,
                                futures._set_result_unless_cancelled,
                                future, result)
    try:
        return (yield from future)
    finally:
        h.cancel()
```

The author at the end use 100 lines of python to demontrate how to achieve concurrency using single thread via coroutine. I would like to break the 100 lines to smaller pieces for reading.


First, just like asyncio.sleep; this is a generator-based coroutine; when later being called in `await sleep(delay)`, yield wait_until will pause the generator and propagate back to *async def countdown* (below) and in turns propagate back to *class SleepingLoop* to help contruct the min heap with smallest wait time on heap top.
```python=
@types.coroutine
def sleep(seconds):
    """Pause a coroutine for the specified number of seconds.

    Think of this as being like asyncio.sleep()/curio.sleep().
    """
    now = datetime.datetime.now()
    wait_until = now + datetime.timedelta(seconds=seconds)
    # Make all coroutines on the call stack pause; the need to use `yield`
    # necessitates this be generator-based and not an async-based coroutine.
    actual = yield wait_until
    # Resume the execution stack, sending back how long we actually waited.
    return actual - now
```

The async coroutine task (which can't really pause by itself, but rely on sleep to pause), after the first run triggered by `send(None)` it paused at the await before assigning delta, we enter it again when `task.coro.send(now)` is called after we sleep through the interval of the current smallest wait time in the heap, and now is propgate to sleep and become the local scope actual and finally return and assign to delta in **async def countdown** and we run until it enter sleep again and create another task in the SleepingLoop's heap with new `wait_until`; similarly the next time it is entered with another `task.coro.send(now)` it would repeat the process and assign to waited and finally raise StopIteration (implicit return None at the end of this async coroutine)
```python=
async def countdown(label, length, *, delay=0):
    """Countdown a launch for `length` seconds, waiting `delay` seconds.

    This is what a user would typically write.
    """
    print(label, 'waiting', delay, 'seconds before starting countdown')
    delta = await sleep(delay)
    print(label, 'starting after waiting', delta)
    while length:
        print(label, 'T-minus', length)
        waited = await sleep(1)
        length -= 1
    print(label, 'lift-off!')
```

The SleepingLoop is a class with syncronous function that keep calling `task.coro.send(now)` to resume pause in sleep and in turns countdown.
```python=
class SleepingLoop:

    """An event loop focused on delaying execution of coroutines.

    Think of this as being like asyncio.BaseEventLoop/curio.Kernel.
    """

    def __init__(self, *coros):
        self._new = coros
        self._waiting = []

    def run_until_complete(self):
        # Start all the coroutines.
        for coro in self._new:
            wait_for = coro.send(None)
            heapq.heappush(self._waiting, Task(wait_for, coro))
        # Keep running until there is no more work to do.
        while self._waiting:
            now = datetime.datetime.now()
            # Get the coroutine with the soonest resumption time.
            task = heapq.heappop(self._waiting)
            if now < task.waiting_until:
                # We're ahead of schedule; wait until it's time to resume.
                delta = task.waiting_until - now
                time.sleep(delta.total_seconds())
                now = datetime.datetime.now()
            try:
                # It's time to resume the coroutine.
                wait_until = task.coro.send(now)
                heapq.heappush(self._waiting, Task(wait_until, task.coro))
            except StopIteration:
                # The coroutine is done.
                pass
```

The main function is also syncrounous where we record the start time before we start the run_until_complete withinwhich we enter and exit coroutine **async def countdown** and **def sleep** and time after all tasks are done (heap in the loop being empty) so that we can see the evidence that everything just take around 5 seconds.
```python=
def main():
    """Start the event loop, counting down 3 separate launches.

    This is what a user would typically write.
    """
    loop = SleepingLoop(countdown('A', 5), countdown('B', 3, delay=2),
                        countdown('C', 4, delay=1))
    start = datetime.datetime.now()
    loop.run_until_complete()
    print('Total elapsed time is', datetime.datetime.now() - start)


if __name__ == '__main__':
    main()
```