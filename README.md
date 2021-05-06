# droppedEntriesCount

Author: Nicolás Peña Moreno @npm1

Participate: file an [issue](https://github.com/w3c/performance-timeline/issues/)

## Introduction

Currently, we recommend using PerformanceObserver, which receives performance entries via a callback that is provided in the constructor of the observer.
The observer signals which entries it's interested in via the observe() method.
When called with the buffered flag, this results in buffered entries being dispatched to the observer on the first observer callback.
The entries are buffered in entryType-specific buffers, and they are useful to allow developers to load performance scripts lazily, instead of early during the page load to avoid losing data.

However, such buffers have size limits for memory-saving reasons (see [maxBufferSize](https://w3c.github.io/timing-entrytypes-registry/#dfn-maxbuffersize)).
These size limits may vary by user agent, and, in the case of Resource Timing, may be set by the developer.
Thus we propose adding a parameter to the observer callback: an integer `droppedEntriesCount`.
When the observer is observing an entryType such that the buffer at some point was full, preventing an entry from being added to the buffer, the number of dropped entries increases.
Note that the buffer being full is not a sufficient condition, as there are buffers that easily become full, like for `paint` (as the only entries for these are the first paint and first contentful paint, and hence the buffer size is 2).
The buffer must become full AND an entry must have been dropped due to this for `droppedEntriesCount` to be incremented.
The `droppedEntriesCount` parameter enables analytics providers to measure how often certain kinds of buffers become full and hence provide feedback to browser vendors about the buffer sizes.

## Goals

This small addition to the Performance Timeline aims to continue improvements on performance monitoring on the web.
The ultimate goal is to enable developers and analytics providers to accurately measure multiple aspects of web performance, which will benefit end-users by enabling these measurements to result in performance improvements.
This parameter aims to improve end-user experience by enabling developers to provide feedback on the frequeency on which buffers become full.
If they never become full, user agents may consider reducing the sizes in order to provide more memory savings to edge-case users.
If they frequently become full, user agents may consider increasing the sizes in order to enable developers to reliably measure end-user experience.

## API

The `droppedEntriesCount` addition may already be seen in the [PerformanceObserverCallback](https://w3c.github.io/performance-timeline/#dom-performanceobservercallback). An integer slot has been [added](https://w3c.github.io/performance-timeline/#dfn-dropped-entries-count) to the performance entry buffer map to cache whether there has been a dropped entry. And the parameter value supplied to the callback is computed on the last steps of the [queue an entry algorithm](https://w3c.github.io/performance-timeline/#queue-a-performanceentry). Example usage:

```js
let callbackHasRun = false;
const resourceObserver = new PerformanceObserver( (list, observer, droppedEntriesCount) => {
   // Send data from list.getEntries() to analytics.
   if (!callbackHasRun) {
     if (droppedEntriesCount) {
       // Send a signal that some performance entries have been lost.
     }
     else {
       // Note that performance entries were not lost in this case.
     }
   }
   // Should not report lost entries after the first callback has run.
   callbackHasRun = true;
});
// Retrieve buffered events and subscribe to newer events for Resource Timing.
resourceObserver.observe({type: "resource", buffered: true});

```

## Design discussion

This addition was discussed in a WebPerf WG call, see the [minutes](https://docs.google.com/document/d/e/2PACX-1vQE0yblkBXaUueIEHhmtH36ccxDVmY48ivhPFNOV4m2k9mjbmRXsBWK5b39UM33ay5X8rSUw3IwuBXw/pub).
Here are the highlights:

* Could it be an event, as there is prior art in [onresourcetimingbufferfull](https://www.w3.org/TR/resource-timing-2/#dom-performance-onresourcetimingbufferfull)?
No, as this would require having JS running on the page at the time this happens, which is something we want to avoid.

* Could it be an API providing buffer sizes by entryType? Possibly, but then you'd still need to know what the max size is for the user agent in order to interpret the data.
So this adds more API surface without directly addressing the use-case.

* Could it be a parameter to observe but exposing more information (like, which entryType was the one for which there was a dropped entry?)?
It could but we want to avoid complicating the callback more than needed.
The boolean does not expose the entryType, but in practice most people use one observer per entryType, which means that in practice they will be able to tell.

## References & acknowledgements

Many thanks for valuable feedback and advice from everyone in the WebPerf WG, especially those who participated in the bug discussion: Nic Jansma, Yoav Weiss.
