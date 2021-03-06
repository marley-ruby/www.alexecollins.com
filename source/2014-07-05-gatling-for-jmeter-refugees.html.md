---
title: Gatling for JMeter Refugees
date: 2014-07-05 08:04 UTC
tags: gatling, scala, performance, jmeter, testing
---
I've recently been looking at Gatling as an alternative to JMeter (something I've used a lot in the past) for load testing. I want to test at least 10,000 concurrent users, and I knew from using JMeter that I would need to use many more nodes than the six we had in the past.

Here are some notes I've made...

What do we want to achieve when load testing?

* Ensure system performs acceptably for anticipated load.
* ROI.
* Easy to write tests.
* Simple to maintain.
* Exercise concurrency issues normal tests might overlook.
* Early testing, multi environment.
* One click execution:
* Run locally, avoid clustering.
* Run against different targets without rewriting.
* Run on CI.
* Easy A/B testing. 
* Record results long term.

Load testing is:

1. Writing a user simulator.
1. Executing simulation.
1. Gathering metrics:
   1. White box.
   1. Black box.
1. Analysing results.
1. Making changes based on results.

Designing a Simulation
---

* I'm a programmer - so let me program!
* No GUI - command line.
* Gatling DSL much more pleasant than JMeter GUI, even for a (caveat) beginner Scala programmer. No poking around in obscure input boxes.
* Easy to re-ractor, extract or re-use.
* Need something complex? Easy to break out into code.
* Understanding of reactive frameworks and basic functional programming make DSL easier to understand.
* Oriented towards HTTP. JMS? Write your own (or search Github).
* Easy to reuse, or combine scenarios.
* Looks easier to write custom plugins. No GUI requirement.
* Arguably easier to manage and version control. 

Metrics
---

* Pretty basic compared to JMeter. Sensible defaults.
* More advanced reporting a potential weak point.
* Graphite integration.
* Metrics are not just mean latency.
* I want to see both request times, and white box metrics (e.g. query times and the like). App-Dynamics, log files, statsd, etc.     

Execute
---

* More users per node - saturate you network card. But .. evidence has been queried. Remain skeptical.
* Scaling to multiple nodes not out of the box. Almost all similar tools support clustering using controller/worker model.
* Debug using Charles as a proxy.

Reports
---

* HTML reports, good for CI server, sharing. JMeter CI plugins exist.
* Graphite reporting. Might be better for continuous load testing.
* Primitive compared to JMeter.

Conclusion
---
Gatling's DSL is more pleasant to use and certainly more powerful per line of code. This means it's easier to get started, and as you can easily refactor, possibly easier to create very complex scenarios as well as cheaper to maintain. JMeter's tenure means that there's more tool support, including cloud services such as BlazeMeter, as well as a much greater variety of plugins.

What does Gatling need to equal JMeter?

* Cloud tooling.
* Clustering.
* Plugins.
