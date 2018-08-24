---
layout: post
title: MARS Alert Broker Support
date: 2018-08-23 11:06:00 -0800
author: Austin Riba <ariba@lco.global>
tags:
 - astronomy
 - django
---

Support for querying the [MARS](https://mars.lco.global) (Make Alerts Really Simple) alert broker
is now available in the TOM Toolkit.

MARS hosts all public alerts from the [ZTF](https://www.ztf.caltech.edu/) survey and makes them searchable
via a web and api based interface. Alerts are added as soon as they are released by ZTF.

With the MARS alert module, you can create queries for interesting alerts and automatically
turn them into observation targets, all within your TOM.

This is the first alert module available in the tomtoolkit `tom_alerts` module, while more brokers are
planned for addition in the future. The source code is
[available on github](https://github.com/TOMToolkit/tom_base/blob/master/tom_alerts/brokers/mars.py).

