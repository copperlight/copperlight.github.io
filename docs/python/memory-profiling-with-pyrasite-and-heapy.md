# Memory Profiling with Pyrasite and Heapy

<div class="meta">
  <span class="date"><small>2017-09-17</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

1. Build or install Pyrasite from the develop branch in the GitHub repo.  The PyPi package does not
have the latest code, which allows you to control the IPC timeout.

    1. <https://github.com/lmacken/pyrasite>

1. Install profiling tools.

        sudo apt-get update
        sudo apt-get install pyrasite guppy

1. Enable ptrace, so that you can inject into the process.

        echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

1. Connect to the running process.

        export PYRASITE_IPC_TIMEOUT=10  # default is 5 seconds

        /apps/python/bin/pyrasite-shell 3640
        Pyrasite Shell 2.0
        Connected to '...'
        Python 2.7.12 (default, Nov 19 2016, 06:48:10)
        [GCC 5.4.0 20160609] on linux2
        Type "help", "copyright", "credits" or "license" for more information.
        (DistantInteractiveConsole)

        >>>

1. Enumerate the threads in the current process.

        import sys, traceback

        for thread, frame in sys._current_frames().items():
            print('Thread 0x%x' % thread)
            traceback.print_stack(frame)
            print()

1. Profile process memory with heapy.

        from guppy import hpy
        hp = hpy()
        hp.heap()

1. Debug the pyrasite process, if needed.

        /apps/python/bin/pyrasite --verbose <pid> helloworld.py
