---
title: Creating an Alert Broker with the TOM Toolkit
---

# Creating an Alert Broker with the TOM Toolkit
This guide will walk you through how to create a custom alert broker using the TOM toolkit.

Be sure you've followed the [Getting Started](/docs/getting_started) guide before continuing onto this tutorial.

## What is an Alert Broker?
(Still working on this section - might need more specific info)
A TOM Toolkit Alert Broker is an object which contains the logic for querying a remote broker (e.g [MARS](https://mars.lco.global)), and transforming the returned data into TOM Toolkit Targets.

# Alert Broker - Getting Started

## TOM Alerts module
(probably need to refine this section a bit)
The TOM Alerts module is a Django app which provides the methods and
classes needed to create a custom TOM alert broker. A broker may be created to ingest alerts of an arbitrary form from a remote source. The TOM Alerts module provides tools to transform these alerts
into TOM-specific alerts to be used in the creation of TOM Targets.

## Project Structure
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

## Creating a Broker
In this example, we will create a broker named __MyBroker__.

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

Finally, import `requests` so that we can fetch some remote broker test data.

```
import requests
```

See [Requests Official API Docs](http://docs.python-requests.org/en/master/)

### Test data

In order to test that our broker *actually*

### Broker forms
To define the query forms for our custom broker, we'll begin by creating class
`MyBrokerForm` inside `my_broker.py`, which inherits the `tom_alert` module's
`GenericQueryForm`.

This will define the list of forms to be presented within the broker query. For
our example, we'll be querying simply on target name.

```
class MyBrokerForm(GenericQueryForm):
    name = forms.CharField(required=True)
```

See [Working with Forms](https://docs.djangoproject.com/en/2.1/topics/forms/) in
the official Django docs.

### Broker class
To define our broker, we'll create the class `MyBroker`, also inside of `my_broker.py`.
Our broker class will encapsulate the logic for making queries to a remote alert broker,
retrieving and sanitizing data, and creating TOM alerts from it.

Begin by defining the class, its name and default form. In our case, the name
will simply be 'MyBroker', and the form will be `MyBrokerForm` - the form that we
just defined!

```
class MyBroker:
    name = 'MyBroker'
    form = MyBrokerForm
```

Each TOM alert broker is required to have a base set of class methods.

#### `fetch_alerts`
`fetch_alerts` is used to query the remote broker, and return a list
of results depending on the parameters passed into the query, so that
these results may be displayed on the query results page. In our case,`fetch_alerts`
will only filter on name, but this can be easily extended to other parameters.

```
@classmethod
    def fetch_alerts(clazz, parameters):
        response = requests.get(broker_url)
        response.raise_for_status()
        test_alerts = response.json()
        return [alert for alert in test_alerts if alert['name'] == parameters['name']]
```

Our implementation will get a response from our test broker source, check that our
request was successful, and return a list of all alerts whose name field matches the
name passed into the query.

#### `fetch_alert`
`fetch_alert` is similar to `fetch_alerts`, but is used in converting a TOM Alert
into a TOM Target. Once a query has been performed, any alert may be made into a TOM
target directly from the query results page.

```
@classmethod
def fetch_alert(clazz, alert_id):
    response = requests.get(broker_url)
    test_alerts = response.json()
    response.raise_for_status()
    for alert in test_alerts:
        if (alert['id'] == int(alert_id)):
            return alert
    return None
```

We query the broker, obtain our JSON data, and return the alert whose id
matches the `alert_id` parameter, so a TOM Alert may be created from it.

#### `to_generic_alert`
In order to standardize alerts and display them in a consistent manner,
the `GenericAlert` class has been defined within the `tom_alerts` library.
This broker method converts a remote alert into a TOM Toolkit `GenericAlert`.

```
@classmethod
def to_generic_alert(clazz, alert):
    return GenericAlert(
        timestamp=alert['timestamp'],
        url=broker_url,
        id=alert['id'],
        name=alert['name'],
        ra=alert['ra'],
        dec=alert['dec'],
        mag=alert['mag'],
        score=alert['score']
    )
```
In our case, the `GenericAlert` attributes match up *almost* directly with our test
data. How convenient! We'll just go ahead and define the `GenericAlert`'s `url`
field as the `broker_url` we retrieved our test data from.

```
...
url=broker_url,
...
```

#### `to_target`
To convert our new alert into a TOM Target, we'll use `to_target`. Similar to
`to_generic_alert`, we'll simply set the relevant TOM Target parameters to
the fields from our alert.

```
@classmethod
def to_target(clazz, alert):
    return Target(
        identifier=alert['id'],
        name=alert['name'],
        type='SIDEREAL',
        designation='MY ALERT',
        ra=alert['ra'],
        dec=alert['dec'],
    )
```
__Probably need to document some of the standard TOM things (Targets, GenericAlerts, etc...)?__
