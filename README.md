This is an alpha release. Do not use this package. Any part of the API is subject to change.

# frankensync

Python introduced the `async` and `await` syntax in version 3.5, allowing asynchronous code that
is easy to read and maintain.

Except when you are maintaining a library with both a synchronous and asynchronous API. In that
case, code reuse becomes a labyrinth of locked doors and `loop.run_until_complete`. This is 
because introducing coroutines requires that you await on coroutines that await on couroutines that awai[...].
Its turtles all the way down.

Many projects could benefit from adding an asynchronous api to work with python's builtin 
asynchronous framework, asyncio. Existing projects often struggle with bringing in async/await 
syntax as it has a tendency to take over. 

Many projects have taken the strategy of rewriting their code with async/await syntax 
extending as far as needed, and providing the old synchronous api by wrapping the awaitable 
api endpoints in `loop.run_util_complete()`, giving the synchronous api a performance hit.

This module is an experiment to explore another strategy for maintaining sync and async api's 
in the same project, using code generation from the AST.

`frankensync` allows you to write code like the following:


``` python
from frankensync import (
    frankensync,
    AwaitOrNot,
)


def update_conversion_rate_middleware(make_request):
    @frankensync
    async def middleware(method, params)
        rate = params.get('rate', False)
        if rate:
            # AwaitOrNot allows us to code both async and sync execution pathways.
            rate = await AwaitOrNot(
                awaitable = coro_pull_conversion_rate(price), 
                sync_fallback = sync_pull_conversion_rate(price),
            )
            params['rate'] = rate

        # make_request will be a coroutine or a regular callable depending on the
        # calling context. Therefore we don't need to use an AwaitOrNot here.
        return await make_request(method, params)
```

## How does this work?

The above `update_conversion_rate_middleware` undergoes some `ast` transformations that result in
two function definitions equivalent to the following:

``` python
def update_conversion_rate_middleware_coro(make_request):
    async def middleware_ASYNC(method, params)
        rate = params.get('rate', False)
        if rate:
            rate = await coro_pull_conversion_rate(price) 
            params['rate'] = rate

        return await make_request(method, params)

    def middleware_SYNC(method, params)
        rate = params.get('rate', False)
        if rate:
            rate = sync_pull_conversion_rate(price)
            params['rate'] = rate

        return make_request(method, params)
```

`frankensync` returns a function that can determine if it is being called in an `await` statement and return either the generated async function definition or the regular function definition depending on the calling context.

The name `frankensync` was chosen to give developers pause before introducing this abomination into
their projects. This is an experimental library, that is primarly serves as a proof of concept.
