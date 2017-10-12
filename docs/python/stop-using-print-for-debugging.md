# Stop Using "print" for Debugging

<small>2017-10-12</small>

My favorite quickstart guide to the Python logging module, by [Al Sweigart](https://inventwithpython.com/blog/author/al-sweigart.html):

[Stop Using "print" for Debugging: A 5 Minute Quickstart Guide to Python’s logging Module](https://inventwithpython.com/blog/2012/04/06/stop-using-print-for-debugging-a-5-minute-quickstart-guide-to-pythons-logging-module/)

```
import logging
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')
logging.debug('This is a log message.')
```
