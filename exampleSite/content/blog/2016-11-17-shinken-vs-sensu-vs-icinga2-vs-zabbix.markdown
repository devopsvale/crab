---
tags:
- soft
- monitoring
comments: true
date: 2016-11-17T09:09:55Z
description: 4 monitoring systems comparison from 4k servers point of view.
keywords: shiknen, sensu, icinga, icinga 2, zabbix, monitoring comparison
title: 'Monitoring system comparison: Shinken vs Sensu vs Icinga 2 vs Zabbix'
lastmod: 2016-11-19T12:38:00Z
slug: shinken-vs-sensu-vs-icinga2-vs-zabbix
---

## Disclaimer

That’s a long post with many images and even more text. There is no answer for simple questions like “which is best”, but a collection of information for decisions on these questions, based on your requirements. I’m looking for a system that works on Linux and monitors Linux well, so platform support is not considered. Also, I’m looking for a system that will help me monitor thousands of servers with tens of thousands of services.

In my own opinion, only Zabbix and Icinga 2 are mature enough to be used in enterprise, and the main question one should ask himself is which monitoring philosophy he could relate to, because all of them do the same thing with very different approaches.

<!--more-->

## [Shinken](http://www.shinken-monitoring.org/)

In their own words, Shinken is a monitoring framework, a Python Nagios Core total rewrite, enhancing flexibility and large environment management.

### Scalability

According to the [documentation](https://shinken.readthedocs.io/en/latest/07_advanced/distributed.html), every type of process can be run even on different hosts. That's interesting, because you might want DB in the cheapest place, data receivers in every datacenter, and alerter processes closer to your physical location. Shinken user on the scheme is happy; that's a positive sign:

![Shinken simple distributed architecture](https://shinken.readthedocs.io/en/latest/_images/shinken-architecture.png)

For multi-regional monitoring, there is also an answer, **[Realms](https://shinken.readthedocs.io/en/latest/_images/shinken-architecture-isolated-realms.png)**.

Here, you notice something awesome: Data is collected into regional DB, not into a global one. There are also sub-realms setup for smaller big setups, which requires fewer machines to setup and just one DB:

![Shinken simple multi-regional distributed architecture](https://shinken.readthedocs.io/en/latest/_images/shinken-architecture-global-realm.png)

Another point to consider when you’re talking about scalability is fault tolerance. I’ll quote documentation here:

> Nobody is perfect. A server can crash, an application too. That is why administrators have spares: they can take configurations of failing elements and reassign them. For the moment, the only daemon that does not have a spare is the Arbiter, but this will be added in the future. The Arbiter regularly checks if everyone is available. If a scheduler or another satellite is dead, it sends its conf to a spare node, defined by the administrator. All satellites are informed by this change, so they can get their jobs from the new element and not try to reach the dead one. If a node was lost due to a network interruption and it comes back up, the Arbiter will notice and ask the old system to drop its configuration.

### Configuration systems integration

Automatic hosts and services discovery seems [well-covered](https://shinken.readthedocs.io/en/latest/07_advanced/discovery-with-shinken-advanced.html), and because configuration is kept in files, you could easily generate them yourself with Chef\Puppet, based on information on hosts you have.

### Audit log

Because configuration is stored in files, you could use generic things, like version control system (Git, Mercurial), to track changes and their ownership. From documentation, I found no evidence of Shinken tracking web-interface actions.

### UI

[![Shinken UI](https://github.com/shinken-monitoring/mod-webui/raw/master/doc/animation.gif)](https://www.shinken-enterprise.com/en/product/)

[Shinken WebUI](https://github.com/shinken-monitoring/mod-webui/wiki) is proven itself to work well with thousands of hosts and tens of groups.

### Drawbacks

I found no visible drawbacks, based on the documentation. The only thing that concerns me is its rapid development in the past and [very slow pace of commits](https://github.com/naparuba/shinken/commits/master) in the present: around 40 commits this year; most are pull requests merged, so no new development is going on, and only community-written bugfixes are being included. It’s either too good to move on (which is never the case; even old-timers like vim and emacs get their code updated) or it’s another opensource project that not enough people care about — you should know such things before using such a complex thing as a monitoring system.

[Frédéric Mohier](https://github.com/mohierf) was very kind to give me an insight on reasons of current situation: more than one year ago, some of the main developers of Shinken left the project and made a fork named [Alignak](https://alignak-monitoring.github.io/), which is being activly developed and plan to deliver 1.0 release in December, 2016.

### Links

* [Detection and Handling of State Flapping — Shinken Manual](http://shinken.readthedocs.io/en/latest/07_advanced/flapping.html)

## [Sensu](https://sensuapp.org/)

Sensu is a monitoring framework (platform, as they call themselves) rather than complete monitoring system. Its key features include:

* Puppet \ Chef integration — define what to check and where to send messages in your configuration system
* Reusing existing technical solutions where possible, instead of inventing their own (Redis, RabbitMQ)

Sensu pulls events from queue and executes handlers on them; that’s it. Handlers can send messages, execute something on the server, or do something else you want.

### Scalability

Sensu architecture is flexible, because every component can be replicated and replaced in a few ways. A sample fault-tolerant setup is described in the [following presentation](https://vimeo.com/92770954); here is a generalized view:

![Sensu architectural diagram](https://sensuapp.org/docs/0.25/img/sensu-diagram.gif)

With HAProxy and Redis-sentinel, you can build a setup in which, if one node of a type is alive (Sensu API, Sensu Dashboard, RabbitMQ, Redis), your monitoring will continue to work without manual intervention.

### Configuration systems integration

Built-in (Puppet, Chef, EC2!) but only in [paid](https://sensuapp.org/docs/latest/enterprise/integrations/) version, which sucks for sure, if you have thousands of servers and don’t want to pay for something with free analogues.

### Audit log

[Built-in](https://sensuapp.org/docs/0.23/enterprise/console/audit-logging.html), however, only in Enterprise edition.

### UI

![Uchiwa screenshot](https://raw.githubusercontent.com/sensu/uchiwa/master/docs/uchiwa-ui.png)

Sensu default UI called [Uchiwa](https://uchiwa.io/), seems to have many limitations. It seems too simple for a diversified environment with thousands of servers. The enterprise edition comes with its own dashboard; however, it doesn’t seem to be doing much, except adding a few disabled out-of-the-box features over the opensource part (like audit).

### Drawbacks

* Lack of historical data and very limited ability to check on it;
* Create your own monitoring system; there are no working presets waiting for you;
* Aggregation of events is tricky;
* ~~Message sending is very tricky, which is scary (because this part must be the simplest and most reliable part of monitoring)~~ — not true, I had wrong impression by [documentation](https://sensuapp.org/docs/latest/reference/handlers.html), thanks [x70b1](https://github.com/x70b1) for explanation;
* The “We don’t want to reinvent the wheel” way has its own limitations of which you could be aware of if you have used any such software before (in my case, it was *Prometheus* monitoring system, which left whole sets of features up to the user to implement, like authentication).

### Links

* [Sensu - What I've Learnt](http://roobert.github.io/2015/11/09/Sensu-What/)

* [MOTD integration](https://github.com/solarkennedy/sensu-report)

## [Icinga 2](https://www.icinga.org/)

Icinga is the fork of Nagios, rewritten from scratch in version 2. Opposite to Shinken, it is a good fork with [constant updates](https://github.com/Icinga/icinga2) being made.

### Scalability

General architecture:

![Icinga 2 architecture](https://www.icinga.org/wp-content/uploads/2014/06/icinga2_cluster.png)

Icinga 2 has a well-designed [distributed monitoring scheme](https://www.icinga.org/products/icinga-2/distributed-monitoring/). Only pitfall I found while setting up the test cluster is the amount of settings related to distribution: It could be overwhelming initially.

### Configuration systems integration

Pretty good, here are two presentations: [The Road to Lazy Monitoring with Icinga 2 and Puppet](https://www.youtube.com/watch?v=j2kG8F7rixw) by Tom de Vylder and [Icinga 2 and Puppet: Automated Monitoring](https://www.youtube.com/watch?v=lLsPwI-6UII) by Walter Heck. The key Icinga feature is storing configuration in files, which makes them easy-to-generate on the Puppet side, which I achieved using [PuppetDB](https://docs.puppet.com/puppetdb/latest/) as a data source about all hosts and services.

### Audit log

As I found, audit log is present with [director module](https://www.icinga.org/2016/03/24/icinga-director-released/). There is no build-in audit in IcingaWeb2 at the moment.

### UI

[![Icinga 2 web interface](https://www.icinga.org/wp-content/uploads/2015/10/Screen-Shot-2015-10-02-at-00.11.09.png)](https://www.icinga.org/products/screenshots/icinga-web-2/)

IcingaWeb2 seems like a decent UI with a lot of extension modules for a lot of purposes. From what I’ve seen, it looks most extendable and flexible, yet has all the features you could expect from a monitoring system UI out of the box.

### Drawbacks

The only drawback I’ve found so far is a complexity of initial setup. It’s not that easy to understand the Icinga point of view on monitoring if you’re used to having something different like Zabbix.

## [Zabbix](http://www.zabbix.com/)

Zabbix is a stable and reliable monitoring system with a steady development pace. It has a huge community and most questions you might think of already have an answer somewhere, so you don’t have to worry if something is possible with Zabbix.

### Scalability

Zabbix has a server that communicates with a single DB, and no matter what you do, with every other resource on hand (memory, network, CPU), you will hit DB disk IO limits. With 6000 IOPS on [Amazon](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-io-characteristics.html), we could maintain around 2k new values per second, which is a lot, but still leaves much to be desired. Proxies and [partitioning](https://www.zabbix.org/wiki/Docs/howto/mysql_partition) could improve the situation with performance, but in terms of reliability, you always have a main DB, which is a single point of failure for everything.

### Configuration systems integration

Zabbix is poorly prepared for a diversified environment, managed by a configuration management system. It has some [low-level discovery capabilities](https://www.zabbix.com/documentation/3.0/manual/discovery/low_level_discovery) for hosts and services, but they have their limits and are not tied to a configuration system. The only option for those who seek such integration is to make something themselves using API.

### Audit log

Zabbix logs everything well, except one huge blind spot: Changes [via API are not logged mostly](https://support.zabbix.com/browse/ZBX-2815), which could be or could not be the problem for your case. The other thing I want to mention is that all problems Zabbix has are logged somewhere in its bug tracker, and if they have enough attention, they are getting fixed sooner or later.

### UI

[![Dashboard is a main screen of Zabbix](https://www.zabbix.com/img/screenshots/3.0/screenshots/mon_dashboard.png)](http://www.zabbix.com/screenshots)

Zabbix has UI with all possible features built-in. The only bad thing you could say about it is it’s not extendable at all — you have either to stick with what UI gives you or do something on your own. You have no option to improve UI because of its general complexity.

### Drawbacks

* Very basic analytic on what is going on, not right now, but in general (only tool here is “top 100 fired triggers”, which improved greatly in 3.0);
* Maintenance setting, as opposed to the Nagios-based system, couldn’t be set on trigger level, only on the host, and the whole system is very complex before the 3.2 release redesign;
* Alerts generation out-of-the-box leaves much to be desired. In my case, we had to implement external alerts aggregation system (to be released in opensource someday);
* Investigating Zabbix performance issues without experience turns into a mess, because you have one big server that you should diagnose.
