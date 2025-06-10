---
authors:
  - copperlight
categories:
  - Running
date: 2018-04-26
draft: false
pin: false
---

# Garmin 935 Navigation

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

## Introduction

Out in the SF Bay Area, there are tons of trails which are fantastic for running. The
[Trailstompers] site is an excellent guide, covering six distinct regions split into four
difficulty levels. Descriptions of the trails, with some turn navigation advice and GPX
tracks are provided for each of the trails. The tracks can be imported into Garmin watches
and you can use the Navigation feature to help stay on course. There are two issues with
using these tracks directly on the watch:

* When loading a GPX track for navigation, it takes awhile to process the course on the watch
and it ends up having far too many points, throwing out many thousands of them.
* Garmin will make some guesses about what turn-by-turn navigation hints to display, but the
ones that you get end up being either obvious or they do not occur when you need them.

This page is a guide to a track processing process which allows you to add custom turn-by-turn
waypoints to tracks and post-processes them so that they appear correctly on your Garmin watch.

The following sites and tools are used:

* [Trailstompers]
* [GPSies]
* CourseTCX2Fit conversion tool from [@CTCX2Fit]

[Trailstompers]: http://www.trailstompers.com
[GPSies]: https://www.gpsies.com
[@CTCX2Fit]: https://twitter.com/ctcx2fit?lang=en

## Process

* Find a trail you like on [Trailstompers] and download the GPX track.
* Upload the GPX track to [GPSies].
* Edit the track and then modify the track. The course title will display as-is on the Garmin.
* You can add a cue sheet, which will populate turn-by-turn hints on the track.
    * The cue sheet feature is capable of adding waypoints to the course points.
* Add or delete waypoints, as desired. It is best to keep waypoints for forks in the trail.
    * Waypoints cannot be manually added to course points - they must be placed alongside course points.
* Download the track and waypoints as a Garmin Course TCX file.
* Run CourseTCX2FIT and convert the TCX file to a FIT file.
* Connect to your Garmin via USB.
* Access the Garmin storage device and copy the FIT file to the GARMIN/COURSES directory. Eject the Garmin device.
* On your Garmin, load the course.
    * Start > Select: Trail Run > Start
    * Hold Up > Select: Navigation
    * Select: Courses > Select: Named Course > Start
    * Do Course > Start
* The Garmin has 15 characters to display course titles, so choose wisely.
