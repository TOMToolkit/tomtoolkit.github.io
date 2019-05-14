---
title: TOM Software Architecture
---

The goal of the TOM Toolkit is to make developing TOMs as easy as possible while
providing the flexibility needed to tailor each TOM to it's specific science
case. The motivation for the TOM Toolkit is discussed on the [about](/about/)
page.

The TOM Toolkit (referred to as "the toolkit") provides a framework for
developing a web application that serves the purposes described in the
motivation. This means when users interact with a TOM they will be doing so via
a web browser or through other programs that can "speak" to it over the
Internet. At this time, web based technologies allow a developer to create rich
user interfaces, simplify distribution and choose from a huge variety of
programming languages and frameworks.

[Python](https://python.org) has becomes to go-to language for many in the
science related fields. Fortunately Python also enjoys widespread popularity in
web development. This provides a very unique opportunity for "Pythonistas" to
develop scientific codebases which integrate seamlessly with web based
technologies. One needs not look further than the success of the
[Jupyter](https://jupyter.org) project to see evidence of this.

There has been a lot of development surrounding Python and the web in the last
two decades. One framework in particular, [Django](https://djangoproject.com)
has emerged as one of the more popular choices for web development. Django is
well known for it's maturity, ease of use and modularity.

Instead of reinventing the wheel, it often makes sense to build on the proven
work of others. Thus, it was decided that the toolkit would build on top of the
Django framework. This provides several advantages:

1. The toolkit does not need to re-implement generic functionality that Django
already provides such as template rendering, routing, object relational mapping,
or even higher level functionality like user accounts and database migrations.

2. TOM developers get to take advantage of the massive amounts of existing
knowledge that already exists for Django projects. In fact, much of the extra
functionality that a TOM developer might want to implement need not be dependent
on the the toolkit at all, but can instead be developed by referring to the
[excellent documentation](https://docs.djangoproject.com/en/2.2/) Django
provides.

3. There are [thousands of Django packages](https://djangopackages.org) already
written that can be used in any TOM project. If a TOM developer wants to be able
to generate dynamic plots, or allow their users to login with Google, or even
turn their TOM into a Slack bot, chances are there is already a package
available that might suit their needs.

We **highly recommend** that developers interested in utilizing the TOM Toolkit
familiarize themselves with the basics of Django, especially if they want to
customize the toolkit in any significant fashion. The majority of the [guides
found in the TOM toolkit documentation](/docs/) are simply Django concepts
rewritten in a TOM context.

## Extending and Customizing the TOM Toolkit.

As mentioned before, Django is well known for it's extendibility and modularity.
The toolkit takes advantage of these strengths heavily. In many ways the TOM
Toolkit is a framework within a framework.

After a TOM developer follows the [getting started guide](/docs/geting_started)
they are left with a functioning but generic TOM. It is then up to the developer
to implement the specific features that their science case requires. The toolkit
tries to facilitate this as efficiently as possible and provides
[documentation](/docs/) in areas of customization from [changing the HTML layout
of a page](/docs/customize_templates) to [altering how observations are
submitted](/docs/customize_observations) and even [creating a new alert
broker](/docs/create_broker).

Django, and by extension the toolkit, rely heavily on object oriented
programming, especially inheritance. Most customization in the TOM toolkit comes
from subclassing classes that provide generic functionality and overriding or
extending methods. An experienced Django developer would feel right at home, as
this is how most work is done in Django as well. For example, the
[ObservationRecordDetailView](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/views.py#L143)
in the `tom_observations` module of the toolkit inherits from Django's
[DetailView](https://docs.djangoproject.com/en/2.2/ref/class-based-views/generic-display/#detailview).
This means TOM developers are able to take full advantage of the power of Django
while still benefiting from the basic functionality that the toolkit provides.

This is why we recommend TOM developers familiarize themselves with Django: most
TOM Toolkit features are actually extended Django features.

## Data Storage, Deployment and Tooling

The toolkit is implemented as a web application backed by a relational database,
uses (mostly) server side rendering, and is deployed using wsgi.

The toolkit should support and relational database that Django supports,
including MySql, Postgresql, SQLite, and Oracle. There is nothing stopping a TOM
developer from supplementing their tom with additional databases, even NoSQL
ones. By default SQLite is deployed due to it's ease of use.

For non database storage (data products, fits files, etc) the toolkit can be
configured to use a variety of cloud based storage services via
[django-storages](https://django-storages.readthedocs.io). The documentation
provides a guide for [storing data on Amazon S3](/docs/amazons3). By default
data is stored on disk.

Deployment similarly works with a variety of servers, including uWsgi and
Gunicorn. The documentation provides a guide to [deploying to
Heroku](docs/deployment_heroku) for those who want to get up and running
quickly. Another option is to use Docker: the demo instance of the toolkit is
deployed to a Kubernetes cluster and the
[Dockerfile](https://github.com/TOMToolkit/tom_demo/blob/master/Dockerfile) is
available on Github.

On the frontend, the toolkit utilizes the very popular [Bootstrap4 css
framework](https://getbootstrap.com) for it's layout and general look, making it
easy to pickup for anyone with experience with CSS. Javascript is introduced
sparingly (astronomers love Python!) but is used in various situations to
enhance the user experience and enable functionality such as interactive
plotting and sky maps.

## Django Reusable Apps

As previously mentioned, one of the reasons for Django's popularity is it's
modularity. Django has the concept of [reusable
apps](https://docs.djangoproject.com/en/2.2/intro/reusable-apps/) which are just
python packages that are specifically meant to be used inside a Django project.
The majority of the the toolkit's functionality is implemented in a series of
Django apps. While most of the apps are required, some may be omitted entirely
from a TOM if the functionality is not desired. The following describes each app
that ships with the toolkit and it's purpose.

### TOM Targets

The
[tom_targets](https://github.com/TOMToolkit/tom_base/tree/master/tom_targets)
app is central to the entire TOM Toolkit project. It provides the database
definitions for the storage and retrieval of targets and target lists. It also
provides the views (pages) for viewing, creating, modifying and visualizing
these targets in several ways including the visibility and target distribution
plots.

Nearly every app depends on the `tom_targets` module in some way.


### TOM Observations

The
[tom_observations](https://github.com/TOMToolkit/tom_base/tree/master/tom_observations)
app handles all the logic for submitting and querying observations of targets at
observatories. It defines the database models for observation requests and
provides some views for working with them.
[facility.py](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facility.py)
defines an interface that external facilities (observatories) can implement in
order to integrate with the toolkit:
[gemini.py](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facilities/gemini.py)
and
[lco.py](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facilities/lco.py)
are two examples, and we expect more in the future.

### TOM Dataproducts

Straddling both the `tom_targets` and `tom_observations` packages is
[tom_dataproducts](https://github.com/TOMToolkit/tom_base/tree/master/tom_dataproducts).
This package contains the logic required for storing data related to targets and
observations within the toolkit. Some data products are fetched from on-line
archives (handled by an observatory's observation module) but data can also be
uploaded manually by the toolkit's users.

This module handles details such as where data should be stored (locally on disk
or in the cloud) as well as displaying certain kinds of data. It also provides
code hooks where TOM developers can run their own functions on the data in case
specialized data processing, analytics or pipelining is required.

### TOM Alerts

The [tom_alerts](https://github.com/TOMToolkit/tom_base/tree/master/tom_alerts)
app contains modules related to the functionality of ingesting targets from
various external services. These services, usually called brokers, provide
rapidly changing target lists that are of interest to time domain astronomers.
The
[alerts.py](https://github.com/TOMToolkit/tom_base/blob/master/tom_alerts/alerts.py)
module provides a generic interface that other modules can implement, giving
them the ability to integrate these brokers with the toolkit. There are
currently modules available for [Lasair](https://lasair.roe.ac.uk),
[MARS](https://mars.lco.global) and
[SCOUT](https://cneos.jpl.nasa.gov/scout/intro.html) with more planned for the
future.

### TOM Catalogs

The
[tom_catalogs](https://github.com/TOMToolkit/tom_base/tree/master/tom_catalogs)
app contains functionality related to querying astronomical catalogs. These
"harvester" modules enable the querying and translation of targets found in
databases such as Simbad and JPL Horizons directly into targets within the
toolkit. The
[harvester.py](https://github.com/TOMToolkit/tom_base/blob/master/tom_catalogs/harvester.py)
module provides the basic interface, and there are several modules already
written for Simbad, NED, the MPC, JPL Horizons and the Transient Name Server.

### TOM Setup and TOM Common

The [tom_setup](https://github.com/TOMToolkit/tom_base/tree/master/tom_setup)
package is special in that it's sole purpose is to help TOM developers bootstrap
new TOMs. See the [getting started](/docs/getting_started) guide for an example.
The [tom_common](https://github.com/TOMToolkit/tom_base/tree/master/tom_common)
package contains logic and data that doesn't fit anywhere else.

## Feedback and bug reporting

We hope the TOM Toolkit is helpful to you and your project. If you have any
concerns about implementation details, or questions about your own needs, please
don't hesitate to [reach out](mailto:ariba@lco.global). Issues and pull requests
are also welcome on the project's [GitHub page](https://github.com/TOMToolkit/).








