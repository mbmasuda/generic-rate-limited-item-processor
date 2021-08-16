# generic-rate-limited-item-processor

## What this package contains

This package contains the code that will allow you to create a queue or stack
of items to process. The default behavior is a queue, but the internal implementation
uses a deque so you can easily subclass this code and override the methods that
push and pop items to change the behavior from a queue to a stack.

Basically, there is a class called `GenericRateLimitedItemProcessor` that is a
thread and when the thread is started, the items in the deque of things to process
are popped off one at a time and whatever they are supposed to do gets started
when the `GenericRateLimitedItemProcessor` pops the next item and calls the item's
`start()` method.

Each item in the `GenericRateLimitedItemProcessor`'s deque must implement a `start()`
method, but this is checked generically via a metaclass implementation of
`__instancecheck__` so that any callable `start` attribute anywhere in the MRO will
satisfy this requirement. Thus, items in the `GenericRateLimitedItemProcessor`'s deque
can be of mixed types and include threads and simple non-threads as well.

Because subclasses of `threading.Thread` will obviously already have a `start()` method
since that is how the thread's `run()` implementation gets started, that is the easy
way of making items to be processed. However, if you don't need anything complicated
and the overhead of making a bunch of threads is unneccessary, you can implement the
simnplest class possible, like this:

```python
class SimplestPossibleItemThatCanBeProcessed:
    def start(self):
        pass
```

and replace the `pass` with whatever you need to happen.


## Installation

```bash
$ python3 -m pip install grip-mbmasuda
```


## Example usage (not a comprehensive tutorial)

```python
from src.grip.grip import GenericRateLimitedItemProcessor


class NumberItem:
    def start(self):
        return 42


class SayHiItem:
    def start(self):
        return 'hi!'

# make some items; you can mix and match and have different types of items processed by
# the same processor
items = [NumberItem() for x in range(5)] + [SayHiItem() for x in range(5)]

# create two item processors that will process the same data
# the only requirement about the items is that they all have a callable start() method
# this item processor will process as fast as possible
item_processor_that_processes_as_fast_as_possible = GenericRateLimitedItemProcessor(iterable=items)

# this one's processing rate is limited to 1000 items in 1.25 seconds
item_processor_with_rate_limited_to_1000_items_in_1.25_seconds = GenericRateLimitedItemProcessor(iterable=items,
                                                                                                 num_items=1000,
                                                                                                 num_seconds=1.25)


# start processing!
item_processor_that_processes_as_fast_as_possible.start()
item_processor_with_rate_limited_to_1000_items_in_1.25_seconds.start()

# wait for all items's start() to be called
# if you want your items to be processed asynchronosuly, you should have your items
# be a subclass of threading.Thread, or the processing they do should be extremely
# minimal and not block, otherwise calling start() on the subsequent items in the item
# processor could be delayed and your item processor will not appear to be processing
# at the rate you think it should be processing at
item_processor_that_processes_as_fast_as_possible.join()
item_processor_with_rate_limited_to_1000_items_in_1.25_seconds.join()

# the items where calling start() raised an exception are collected in this deque
problem_items = item_processor_that_processes_as_fast_as_possible.unsuccessfully_started

# the items where calling start() was successful and did not raise are collected in this deque
good_items = item_processor_that_processes_as_fast_as_possible.successfully_started

# just because an item ends up in the successfully_started deque does not mean your item's
# processing code exited without errors; it only means that the item's start() was called
# without raising an excaption and you are responsible for going through the items in the
# successfully_started deque to check status. For example, if the item is a thread whose
# run() has code that makes an HTTP request, then you are responsible for checking the
# request's response to ensure it did not come back with an HTTP 4xx or 5xx (or whatever
# you need it to do)
```


## Testing


```text
$ py.test -vvvv
====================================================== test session starts ======================================================
platform darwin -- Python 3.9.6, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- <redacted>
cachedir: <redacted>
rootdir: <redacted>
collected 26 items

test/test_everything.py::TestPorcelain::test_make_new_instances PASSED                                                    [  3%]
test/test_everything.py::TestPorcelain::test_isinstance_works_correctly PASSED                                            [  7%]
test/test_everything.py::TestPorcelain::test_simplest_possible_item_that_can_be_processed PASSED                          [ 11%]
test/test_everything.py::TestPorcelain::test_mixed_and_matched_items PASSED                                               [ 15%]
test/test_everything.py::TestPorcelain::test_valid_items_can_be_processed PASSED                                          [ 19%]
test/test_everything.py::TestPorcelain::test_rate_limiting_works[100-1] PASSED                                            [ 23%]
test/test_everything.py::TestPorcelain::test_rate_limiting_works[3-1.98324729923487] PASSED                               [ 26%]
test/test_everything.py::TestPorcelain::test_rate_limiting_works[2344-0.3] PASSED                                         [ 30%]
test/test_everything.py::TestPlumbing::test_make_new_instance[iterable_data0-None] PASSED                                 [ 34%]
test/test_everything.py::TestPlumbing::test_make_new_instance[iterable_data1-4] PASSED                                    [ 38%]
test/test_everything.py::TestPlumbing::test_make_new_instance[iterable_data2-None] PASSED                                 [ 42%]
test/test_everything.py::TestPlumbing::test_make_new_instance[iterable_data3-7] PASSED                                    [ 46%]
test/test_everything.py::TestPlumbing::test_make_new_instance[iterable_data4-3] PASSED                                    [ 50%]
test/test_everything.py::TestPlumbing::test_make_new_instance[abcdefg-None] PASSED                                        [ 53%]
test/test_everything.py::TestPlumbing::test_get_next_item[iterable_data0-None] PASSED                                     [ 57%]
test/test_everything.py::TestPlumbing::test_get_next_item[iterable_data1-4] PASSED                                        [ 61%]
test/test_everything.py::TestPlumbing::test_get_next_item[iterable_data2-None] PASSED                                     [ 65%]
test/test_everything.py::TestPlumbing::test_get_next_item[iterable_data3-7] PASSED                                        [ 69%]
test/test_everything.py::TestPlumbing::test_get_next_item[iterable_data4-3] PASSED                                        [ 73%]
test/test_everything.py::TestPlumbing::test_get_next_item[abcdefg-None] PASSED                                            [ 76%]
test/test_everything.py::TestPlumbing::test_append_item[iterable_data0-None] PASSED                                       [ 80%]
test/test_everything.py::TestPlumbing::test_append_item[iterable_data1-4] PASSED                                          [ 84%]
test/test_everything.py::TestPlumbing::test_append_item[iterable_data2-None] PASSED                                       [ 88%]
test/test_everything.py::TestPlumbing::test_append_item[iterable_data3-7] PASSED                                          [ 92%]
test/test_everything.py::TestPlumbing::test_append_item[iterable_data4-3] PASSED                                          [ 96%]
test/test_everything.py::TestPlumbing::test_append_item[abcdefg-None] PASSED                                              [100%]

====================================================== 26 passed in 3.95s =======================================================
```



## How to get in touch

Please email me at <mbmasuda.github@gmail.com>
