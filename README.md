# Snowplow-Mini

[ ![Build Status] [travis-image] ] [travis] [ ![Release] [release-image] ] [releases] [ ![License] [license-image] ] [license]

An easily-deployable, single instance version of Snowplow that serves three use cases:

1. Gives a Snowplow consumer (e.g. an analyst / data team / marketing team) a way to quickly understand what Snowplow "does" i.e. what you put it at one end and take out of the other
2. Gives developers new to Snowplow an easy way to start with Snowplow and understand how the different pieces fit together
3. Gives people running Snowplow a quick way to debug tracker updates (because they can)

## Features

* [x] Data is tracked and processed in real time
* [x] Added Iglu Server to allow for custom schemas to be uploaded
* [x] Data is validated during processing
  - This is done using both our standard Iglu schemas and any custom ones that you have loaded into the Iglu Server
* [x] Data is loaded into Elasticsearch
  - Can be queried directly or through a Kibana dashboard
  - Good and bad events are in distinct indexes

## Topology

Snowplow-Mini runs several distinct applications on the same box which are all linked by named pipes.  In a production deployment each instance could be an Autoscaling Group and each named pipe would be a distinct Kinesis Stream.

* Scala Stream Collector:
  - Starts server listening on port `8080` which events can be sent to.
  - Sends "good" events to the `raw-events-pipe`
  - Sends "bad" events to the `bad-1-pipe`
* Stream Enrich
  - Reads events in from the `raw-events-pipe`
  - Sends "good" events to the `enriched-events-pipe`
  - Sends "bad" events to the `bad-1-pipe`
* Elasticsearch Sink Good
  - Reads events in from the `enriched-events-pipe`
  - Sends the events to the "good" index of the cluster
  - On failure to insert writes error to `bad-1-pipe`
* Elasticsearch Sink Bad
  - Reads events in from the `bad-1-pipe`
  - Sends the events to the "bad" index of the cluster

These events can then be viewed at port `5601` in Kibana.

![](https://raw.githubusercontent.com/snowplow/snowplow-mini/master/resources/topology/snowplow-mini-topology.jpg)

## Roadmap

* [ ] Support loading data into Redshift. To give analysts / data teams a good idea to understand what Snowplow "does".
* [ ] Create UI to indicate what is happening with each of the different subsystems (collector, enrich etc.), so as to provide developers a very indepth way of understanding how the different Snowplow subsystems work with one another

## Documentation

1. [Quick start guide] [get-started-guide]

## Copyright and license

Snowplow Mini is copyright 2016 Snowplow Analytics Ltd.

Licensed under the **[Apache License, Version 2.0] [license]** (the "License");
you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[get-started-guide]: https://github.com/snowplow/snowplow-mini/wiki/Quickstart-guide

[travis]: https://travis-ci.org/snowplow/snowplow-mini
[travis-image]: https://travis-ci.org/snowplow/snowplow-mini.svg?branch=master

[release-image]: http://img.shields.io/badge/release-0.2.1-blue.svg?style=flat
[releases]: https://github.com/snowplow/snowplow-mini/releases

[license-image]: http://img.shields.io/badge/license-Apache--2-blue.svg?style=flat
[license]: http://www.apache.org/licenses/LICENSE-2.0
