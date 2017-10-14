# Stop Using "print" for Debugging

<div class="meta">
  <span class="date"><small>2017-10-12</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

My favorite quickstart guide to the Python logging module, by [Al Sweigart](https://inventwithpython.com/blog/author/al-sweigart.html):

[Stop Using "print" for Debugging: A 5 Minute Quickstart Guide to Python’s logging Module](https://inventwithpython.com/blog/2012/04/06/stop-using-print-for-debugging-a-5-minute-quickstart-guide-to-pythons-logging-module/)

```
import logging
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')
logging.debug('This is a log message.')
```
