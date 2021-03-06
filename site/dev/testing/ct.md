Cluster Telemetry
=================

### Contents ###

*   [Overview](#overview)
*   [Framework Usage](#framework_usage)
*   [System Architecture](#system_architecture)
    +   [System Diagram](#system_diagram)
    +   [Detailed explanation of steps](#detailed_explanation)
*   [Code](#code)
*   [Contact Us](#contact_us)

<a name="overview"></a>
Overview
--------

Cluster Telemetry allows you to run [telemetry](https://www.chromium.org/developers/telemetry)'s benchmarks, lua scripts and other tasks using multiple repository patches through Alexa's [top 1 million](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip) web pages.
Developers can use the framework to measure the performance of their patch against the top subset of the internet on both Desktop and Android.
tl;dr documentation is [here](https://www.chromium.org/developers/cluster-telemetry).

SKP files are a binary format for the draw commands Chromium sends to Skia for rasterization. The goal of the project started off with wanting to collect a large repository of 10k SKP files. This repository, after incremental changes in approaches, has since grown to ~910k and now supports running all telemetry benchmarks. The top level feature request of this project was [skia:1268](https://bug.skia.org/1268).

A web application has been created on Google Compute Engine that automates the process of capturing new archives and running telemetry benchmarks at a click of a button; results are emailed to the requester and the web application contains complete history of runs with links to results. You can run telemetry benchmarks at http://ct.skia.org.

The framework also contains the ability to run lua scripts on the SKP repository to scrape web pages. It only takes a few minutes to run a lua scraping script on ~910k SKP files.

Most users will use these three features:

* Chromium Perf. Documentation [here](https://docs.google.com/document/d/1GhqosQcwsy6F-eBAmFn_ITDF7_Iv_rY9FhCKwAnk9qQ/). Webpage [here](https://ct.skia.org/chromium_perf/).
* Chromium Analysis. Documentation [here](https://docs.google.com/document/d/1ziof4lNwDFXyerVbEocdF3_DdUHVnD3FKYB9rShztuE/). Webpage [here](https://ct.skia.org/chromium_analysis/).
* Run Lua Scripts on SKP repositories. Documentation about lua bindings is [here](https://skia.org/user/special/lua). Webpage [here](https://ct.skia.org/lua_script/).

Note: The top 1M web pages includes potentially offensive content. Please use caution when visiting page links from the framework.


<a name="framework_usage"></a>
Framework Usage
---------------

The Chromium Perf page in CT has been used to gather perf data over the top 10k web pages for the following Chromium projects:

* Slimming paint
* Performance data for layer squashing and compositing overlap map
* SkPaint in Graphics Context
* Culling
* New paint dictionary

blink-dev threads discussing how to make Chrome faster using the results gathered from CT:

* [Main thread attribution for top million sites](https://groups.google.com/a/chromium.org/d/msg/blink-dev/-R47hzmkdig/xILVgczlKgQJ)
* [Layout time for top 10k sites](https://groups.google.com/a/chromium.org/d/msg/blink-dev/fkRYGcIQN1g/_uYcAt6G8XsJ)
* [Perf profile for top million sites](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/8qd5SmLF5n0)

Documents detailing data generated by the framework:

* [Loading measurement: alexa top 1,000](https://docs.google.com/a/chromium.org/document/d/1ca_Q7xePmCRqaYnHe7vkpCmKNFNLdDXvzgtUPt9iG8w/edit)
* [Loading measurement: alexa top million](https://docs.google.com/a/google.com/document/d/1hDDUUNE5OUV8eCjtOj7Ow6EZ2DSBCTjQirnA3Rp5pOg/edit)
* [Loading measurement: alexa top million netsim](https://docs.google.com/a/google.com/document/d/1cpLSSYpqi4SprkJcVxbS7af6avKM0qc-imxvkexmCZs/edit)
* [Perf profile - alexa top million sites](https://docs.google.com/a/google.com/document/d/1di__87watociuZj_dm22Cn72UM2xsZBXixbl8TCFQmw/edit)

The framework has also been used to run multiple lua scripts to scrape the SKP repositories for the the following:
chars-vs-glyphs, bitmap transform types, gradient color counter, 3 color gradient checks, etc.
This has been very useful for the Skia team to help determine which parts of the library to optimize and focus on.

All runs are recorded [here](https://ct.skia.org/history/).


<a name="system_architecture"></a>
System Architecture
-------------------

<a name="system_diagram"></a>
### System Diagram

![CT System Diagram](ct-system-diagram.svg)


<a name="detailed_explanation"></a>
### Detailed explanation of steps

1. User submits a Lua script task, a Performance task, an Analysis task, or an Admin task (build chrome, recreate pagesets, recreate webpage archives, capture SKPs) using the GCE web application [here](http://ct.skia.org).

2. Each task is exposed by the web application in JSON. The CT master polls the web application and picks up new tasks. It has the ability to run tasks in parallel.

3. The master triggers swarming tasks using the master scripts [here](https://skia.googlesource.com/buildbot/+/master/ct/go/master_scripts/). The master scripts then check to see when the tasks are done.

4. Swarming bots in the CT pool execute the task using the worker scripts [here](https://skia.googlesource.com/buildbot/+/master/ct/go/worker_scripts/). All generated artifacts (CSV files, logs, SKP files, archives, etc) are then copied to Google Storage.

5. Once swarming tasks complete, master scripts read the generated artifacts from Google Storage and consolidate them (if required).

6. The master scripts then email results of the task to the user who requested it. The scripts also update the status of the task to completed on the web application.


<a name="code"></a>
Code
----

Cluster Telemetry is primarily written in Go with a few python scripts. The framework lives in [master/ct](https://skia.googlesource.com/buildbot/+/master/ct).

<a name="contact_us"></a>
Contact Us
----------

If you have questions, please email <cluster-telemetry@chromium.org> or contact rmistry@ directly.
