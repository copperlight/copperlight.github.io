---
authors:
  - copperlight
categories:
  - Shell
date: 2017-08-15
draft: false
pin: false
---

# Schedule Ad-Hoc Jobs for Later on an Instance with atd

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

<https://www.computerhope.com/unix/uat.htm>

The purpose of this technique is to allow you to disconnect from the instance, while leaving the job
running (i.e. not dependent on a shell fork).

```
(root) # at now + 1 minute
warning: commands will be executed using /bin/sh
at> /apps/something/bin/upload_to_s3.sh
at> ^d
job 2 at Wed Aug  9 20:37:00 2017

(root) # atq
2   Wed Aug  9 20:37:00 2017 a root
```

If you need to remove and reschedule a job, you can do the following:

```
(root) # atrm 2
```

The following are examples of casual times that can be used with at:

expression | time
-----------|-----
noon | 12:00 PM October 18 2014
midnight | 12:00 AM October 19 2014
teatime | 4:00 PM October 18 2014
tomorrow | 10:00 AM October 19 2014
noon tomorrow | 12:00 PM October 19 2014
next week | 10:00 AM October 25 2014
next monday | 10:00 AM October 24 2014
fri | 10:00 AM October 21 2014
NOV | 10:00 AM November 18 2014
9:00 AM  | 9:00 AM October 19 2014
2:30 PM | 2:30 PM October 18 2014
1430 | 2:30 PM October 18 2014
2:30 PM tomorrow | 2:30 PM October 19 2014
2:30 PM next month | 2:30 PM November 18 2014
2:30 PM Fri | 2:30 PM October 21 2014
2:30 PM 10/21 | 2:30 PM October 21 2014
2:30 PM Oct 21 | 2:30 PM October 21 2014
2:30 PM 10/21/2014 | 2:30 PM October 21 2014
2:30 PM 21.10.14 | 2:30 PM October 21 2014
now + 30 minutes | 10:30 AM October 18 2014
now + 1 hour | 11:00 AM October 18 2014
now + 2 days | 10:00 AM October 20 2014
4 PM + 2 days | 4:00 PM October 20 2014
now + 3 weeks | 10:00 AM November 8 2014
now + 4 months | 10:00 AM February 18 2015
now + 5 years | 10:00 AM October 18 2019
