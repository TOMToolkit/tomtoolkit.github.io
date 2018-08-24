---
title: Creating an Alert Broker with the TOM Toolkit
---

# Creating an Alert Broker with the TOM Toolkit
This guide will walk you through how to create a custom alert broker using the TOM toolkit.

Be sure you've followed the [Getting Started](/docs/getting_started) guide before continuing onto this tutorial.

## What is an Alert Broker?
(Still working on this section - might need more specific info)
A TOM Toolkit Alert Broker is an object which contains the logic for querying a remote broker (e.g [MARS](https://mars.lco.global)), and transforming the returned data into TOM Toolkit Targets.

## Alert Broker - Getting Started

### TOM Alerts module
(probably need to refine this section a bit)
The TOM Alerts module is a Django app which provides the methods and
classes needed to create a custom TOM alert broker. A broker may be created to ingest alerts of an arbitrary form from a remote source. The TOM Alerts module provides tools to transform these alerts
into TOM-specific alerts to be used in the creation of TOM Targets.

### Project Structure
After following the [Getting Started](/docs/getting_started) guide, you will have
a Django project directory of the form:

```
mytom
├── db.sqlite3
├── manage.py
└── mytom
    ├── __init__.py
    ├── __pycache__
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

### Creating a Broker
In this example, we will create a broker named *MyBroker*.

Begin by creating a file `my_broker.py`, and placing it in the `mytom/` directory
of the project. `my_broker.py` will contain the class which defines our custom
TOM Alert Broker.

Our custom broker relies on the TOM Toolkit modules that were installed in the
[Getting Started](/docs/getting_started) guide. Begin by editing `my_broker.py`
to import the necessary modules.

```
from tom_alerts.alerts import GenericQueryForm, GenericAlert, get_service_class
from tom_alerts.models import BrokerQuery
from tom_targets.models import Target
```

In order to add custom forms to our broker, we will also need Django's `forms` module.

```
from django import forms
```

#### Broker forms
To define the query forms for our custom broker, we will begin by creating class
`MyBrokerForm` inside `my_broker.py`, which inherits the `tom_alert` module's `GenericQueryForm`.

This will define the list of forms to be presented within the broker query. For
our example, we'll be querying on target name, RA, DEC, MAG, and Score. The only
required field is the target name.

```
class MyBrokerForm(GenericQueryForm):
    name = forms.CharField(required=True)
    ra = forms.FloatField(required=False)
    dec = forms.FloatField(required=False)
    mag = forms.FloatField(required=False)
    score = forms.FloatField(required=False)
```

See [Working with Forms](https://docs.djangoproject.com/en/2.1/topics/forms/) in
the official Django docs.

#### Broker class
To define our broker, we will create the class `MyBroker`, also inside of `my_broker.py`.
Our broker class will encapsulate the logic for making queries to a remote alert broker, retrieving and sanitizing data, and creating TOM alerts from it.

Begin by defining the class, its name and default form. In our case, the name
will simply be 'MyBroker', and the form will be `MyBrokerForm` - the form that we
just defined.

```
class MyBroker:
    name = 'MyBroker'
    form = MyBrokerForm
```
