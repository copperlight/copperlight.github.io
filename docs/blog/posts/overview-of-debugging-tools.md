---
authors:
  - copperlight
categories:
  - Debugging
  - Java
date: 2016-10-13
draft: false
pin: false
---

# Overview of Debugging Tools

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

<div style="text-align: center;">
<table>
    <tr> <th>Tool <th>Use Cases
    <tr>
        <td><a href="https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/run.htm#JFRUH164">Java Flight Recorder</a>
        <td>
            <ul>
            <li>first choice for GC allocation pauses and heap pressure
            <li>first choice for thread contention
    <tr>
        <td><a href="http://www.brendangregg.com/blog/2014-06-12/java-flame-graphs.html">Java Flame Graphs</a>
        <td>
            <ul>
            <li>first choice for CPU contention
    <tr>
        <td><a href="https://visualvm.github.io/">VisualVM</a>
        <td>
            <ul>
            <li>second choice for CPU contention
    <tr>
        <td><a href="https://www.yourkit.com/">YourKit Profiler</a>
        <td>
            <ul>
            <li>third choice for CPU contention, other options are more accessible
            <li>not as good as flight recorder for heap pressure
    <tr>
        <td><a href="http://openjdk.java.net/projects/code-tools/jmh/">Java Micro Harness (JMH)</a>
        <td>
            <ul>
            <li>first choice for writing tiny benchmarks to evaluate code samples
</table>
</div>
