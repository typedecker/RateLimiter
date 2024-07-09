# RateLimiter
Helps in handling rate limits, specifically made to work with discord bots to assist in queueing requests when rate limit clashes are possible.

The lightweight python module provides a `Ratelimiter` class that allows you to ratelimit certain async methods and functions. The ratelimiter class has a decorator and a wrapper method built into it that lets you wrap pre defined or user defined functions with ratelimit checks! Based on the arguments provided it queues up any requests(calls to the async function being ratelimited) that are made after the specified rate limit has been hit. It then slowly clears the pending requests over time within the bounds set by the rate limit.

### Example:
> If you were to call an async function 5 times without any ratelimits the 5 requests would be made right after one another, but if you wrap them around with a rate limit of 1 request per 10 seconds, a request would be made every 10 second, and the calls will finish in 50 seconds. This might result in a slower execution but will successfully keep the program within the ratelimits specified which is important when it comes to discord bots.

## Usage

You first have to initialize an object of the `Ratelimiter` class like so:
```py
import ratelimit_handler as rh

ratelimiter = rh.Ratelimiter(unit_time = 1000, default_rl = 10)
```
* **unit_time:** Unit time is in milliseconds and refers to how much time does the rate limiter wait for, before going through the pending requests. After every `unit_time` seconds the `reset_requests` method of the ratelimiter object is called, which resets the completed requests to 0 and thus the pending requests can be processed until the counter hits the rate limit again.
* **default_rl:** Default RL stands for default rate limit, it is the default value for the number of requests that are allowed to be made per unit_time by any function that is wrapped using this object, if a different rate limit isnt explicitly mentioned.

Now any functions you define can be wrapped in two ways:
1. Using the `ratelimiter.ratelimit(rl = 10)` decorator.
2. Using the `ratelimiter.handle(func, default_rl = None)` method.

### 1. Using the `ratelimiter.ratelimit(rl = 10)` decorator.
The first one is useful when defining a user defined function that needs to be ratelimited everytime it's executed. The decorator registers the function as one of the ones to be ratelimited by the ratelimiter everytime that it's executed:

```py
@ratelimiter.ratelimit(rl = 5) # 5 requests per 1000 milliseconds.
async def my_new_func(*args, **kwargs) :
  ...
  return

for _ in range(50) :
  await my_new_func(*args, **kwargs) # Will be ratelimited everytime
  continue
await ratelimiter.free(my_new_func) # Frees the function from the ratelimit.
```

Note that the argument `rl` passed to the ratelimit decorator stands for rate limit and determines how many requests can be made per unit_time.

Furthermore, `ratelimiter.free(func)` method is used at the end to free the function from the ratelimit imposed on it, and to save up on memory usage due to constant checks for any pending requests.

### 2. Using the `ratelimiter.handle(func, default_rl = None)` method.
The second one is useful when a pre defined function/method imported from a module or elsewhere must be ratelimited everytime it's executed. The handle function can be used to get back a ratelimited version of that function for every execution:

```py
@client.event
async def on_message(message) :
  if message.content.lower() == '!rl_ping' :
    for _ in range(5) :
      ratelimited_send = ratelimiter.handle(message.channel.send)
      await ratelimited_send('This is a ratelimited ping! It will be way cooler than a normal one :D')
      continue
    await ratelimiter.free(message.channel.send)  # Frees the method from the ratelimit.
```

Furthermore, `ratelimiter.free(func)` method is used at the end to free the method from the ratelimit imposed on it, and to save up on memory usage due to constant checks for any pending requests.

--- 

## Upcoming stuff

Do note that this project/module is still under development and will get more features and changes in the future. Some of the planned changes and additions are:
1. Addition of "groups" and changing the internal api to work with group names rather than with function names, that way multiple functions can be grouped up together and can all be ratelimited collectively, which might be a useful feature to have.
2. Adding ways to handle rate limits that are caused by async generators.
3. Ways to handle and assist with scheduled tasks and ratelimit handling for them.

---

*If you are interested in helping in improving this project please reach out to me via discord: `typedecker`,
I'd be happy to hear about your opinions, suggestions, criticism and ideas for this project!*
