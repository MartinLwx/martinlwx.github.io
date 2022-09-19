# How to Use Logging in Python


## Intro

---

Recently I was fine-tuning my deep learning model, and I habitually started to use `print` to print some key information on the terminal. So my workflow is like:

1. I type some hyperparameters to train my model.
2. I manually opened an Excel to record the hyperparameters used and the model evaluation results. And I will go back to step 1. If I am not satisfied with results.

Soon, I became bored with this workflow (~~In fact, I kept this for quite a long time.~~). However, I suddenly forgot to record key information manually. As a result, I had to navigate in the history output of the the terminal slowly. 



At this time, I think I should use `logging`. I don't want to tortue myself anymore🙅‍♂️. That's why I wrote this blog post:hugs:

## Basic

---

### logger level

1. `DEBUG`
When you want to debug your python file
2. `INFO`
Make sure everything goes as expected 
3. `WARNING`
Something unexpected happened. It may cause some problems in the future
4. `ERROR`
Some function may fail
5. `CRITICAL`
The program may stop running

The 5 levels actually mean different severity. By default, tbe `logging` module will only record `WARNING` level or above(`ERROR` + `CRITICAL`)

## Best Practice

---

### Step 1. Import `logging` module and define your customized logger

Of course, you can use the default logger without configurations. See a qucik example :arrow_down:

```python
import logging

logging.warning("WARNING!!!")

# output
# WARNING:root:WARNING!!!
```

#### How to customized your logger

Use `logging.basicConfig(args)`. The `logging` module provides us some args to customize our own logger. For example:

1. `filename`: the logfile filename
- `level`: it decides what type of level(or above)'s messages will be recorded
- `format`: the format of message
- `filemode`: By default, it is equal to `a`, which means append

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("Everything is ok")

# output
# INFO:root:Everything is ok
```

### Step 2. Start logging messages

You can logging whatever messages you like.



The `logging` module provide us some `LogRecord attributes`. For example:

- `%(asctime)s`: Human-readable time 
- `%(levelname)s`: Indicating the logginglevel
- `%(message)s`: logged message

Usually, we will use these `LogRecord attributes` in aforementioned `format` arg. For example:arrow_down:

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s ==> %(message)s")
logging.info("Everything is ok")

# output
# 2022-01-08 00:32:11,579 ==> Everything is ok
```

## Advanced

---

The [previous article](https://martinlwx.github.io/post/how-to-debug-in-python/) talks about how to debug in python. In fact, learning to use `logging` is also a powerful way to help debugging. For example, we can use `logging` to record the report when the program crashed. See a simple example⬇️

```python
import logging

try:
    print(4 // 0)
except Exception as e:
    logging.error("===> Exception detected", exc_info=True)

# output:
# ERROR:root:===> Exception detected
# Traceback (most recent call last):
#   File "hello.py", line 4, in <module>
#     print(4 // 0)
# ZeroDivisionError: integer division or modulo by zero
```

## Refs

---

1. [logging HOWTO](https://docs.python.org/3/howto/logging.html)
2. [logging docs](https://docs.python.org/3/library/logging.html)


