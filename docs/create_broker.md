---
title: Creating an Alert Broker with the TOM Toolkit
---

# Creating an Alert Broker with the TOM Toolkit
This guide will walk you through how to create a custom alert broker using the TOM toolkit.

Be sure you've followed the [Getting Started](/docs/getting_started) guide before continuing onto this tutorial.

## What is an Alert Broker?
A TOM Toolkit Alert Broker is an object which contains the logic for querying a remote broker
(e.g [MARS](https://mars.lco.global)), and transforming the returned data into TOM Toolkit Targets.

### TOM Alerts module
The TOM Alerts module is a Django app which provides the methods and
classes needed to create a custom TOM alert broker. A broker may be created to ingest
alerts of an arbitrary form from a remote source. The TOM Alerts module provides
tools to transform these alerts into TOM-specific alerts to be used in the creation of TOM Targets.

# Project Structure
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

# Creating a Broker
In this example, we will create a broker named __MyBroker__.

Begin by creating a file `my_broker.py`, and placing it in the `mytom/` directory
of the project. `my_broker.py` will contain the classes that define our custom
TOM Alert Broker.

Our custom broker relies on the TOM Toolkit modules that were installed in the
[Getting Started](/docs/getting_started) guide. Begin by editing `my_broker.py`
to import the necessary modules.

```python
from tom_alerts.alerts import GenericQueryForm, GenericAlert, get_service_class
from tom_alerts.models import BrokerQuery
from tom_targets.models import Target
```

In order to add custom forms to our broker, we will also need Django's `forms` module.

```python
from django import forms
```
See [Working with Django Forms](https://docs.djangoproject.com/en/2.1/topics/forms/)

Finally, import `requests` so that we can fetch some remote broker test data.

```python
import requests
```
See [Requests Official API Docs](http://docs.python-requests.org/en/master/)

## Test Data

In place of a remote broker, we've uploaded a [sample JSON file to GitHub Gist](https://gist.githubusercontent.com/mgdaily/f5dfb4047aaeb393bf1996f0823e1064/raw/5e6a6142ff77e7eb783892f1d1d01b13489032cc/example_broker_data.json).

For our broker to use this data, we will set `broker_url` to it.
```
broker_url = https://gist.githubusercontent.com/mgdaily/f5dfb4047aaeb393bf1996f0823e1064/raw/5e6a6142ff77e7eb783892f1d1d01b13489032cc/example_broker_data.json
```

## Broker Forms
To define the query forms for our custom broker, we'll begin by creating class
`MyBrokerForm` inside `my_broker.py`, which inherits the `tom_alert` module's
`GenericQueryForm`.

This will define the list of forms to be presented within the broker query. For
our example, we'll be querying simply on target name.

```python
class MyBrokerForm(GenericQueryForm):
    name = forms.CharField(required=True)
```

See [Working with Forms](https://docs.djangoproject.com/en/2.1/topics/forms/) in
the official Django docs.

## Broker Class
To define our broker, we'll create the class `MyBroker`, also inside of `my_broker.py`.
Our broker class will encapsulate the logic for making queries to a remote alert broker,
retrieving and sanitizing data, and creating TOM alerts from it.

Begin by defining the class, its name and default form. In our case, the name
will simply be 'MyBroker', and the form will be `MyBrokerForm` - the form that we
just defined!

```python
class MyBroker:
    name = 'MyBroker'
    form = MyBrokerForm
```

## Required Broker Class Methods
Each TOM alert broker is required to have a base set of class methods. These
methods enable the conversion of remote alert data into TOM-specific
alerts and targets.

#### `fetch_alerts` Class Method
`fetch_alerts` is used to query the remote broker, and return a list
of results depending on the parameters passed into the query, so that
these results may be displayed on the query results page. In our case, `fetch_alerts`
will only filter on name, but this can be easily extended to other query parameters.

```python
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

#### `fetch_alert` Class Method
`fetch_alert` is similar to `fetch_alerts`, but fetches a single alert with a given
`alert_id`. Once a query has been performed, any resulting alert may be made into
a TOM target directly from the query results page.

```python
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

#### `to_generic_alert` Class Method
In order to standardize alerts and display them in a consistent manner,
the `GenericAlert` class has been defined within the `tom_alerts` library.
This broker method converts a remote alert into a TOM Toolkit `GenericAlert`.

```python
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

```python
...
url=broker_url,
...
```

#### `to_target` Class Method
To convert our new alert into a TOM Target, we'll use `to_target`. Just like
`to_generic_alert`, we'll simply set the relevant TOM Target parameters to
the fields from our alert.

```python
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

## Using Our New Alert Broker
Now that we've created our TOM alert broker, let's hook it into our TOM
so that we can ingest alerts and create targets.

The `tom_alerts` module will look in `settings.py` for a list of alert
broker classes, so we'll need to add `MyBroker` to that list.

```python
TOM_ALERT_CLASSES = [
    ...
    'tom_alerts.brokers.mars.MARSBroker',
    'mytom.my_broker.MyBroker',
    ...
]
```
Now, navigate to the top-level directory of your Django project,
where `manage.py` resides and run

```bash
./manage.py makemigrations
./manage.py migrate
./manage.py runserver
```

Navigate to [http://127.0.0.1:8000/alerts/query/list/](http://127.0.0.1:8000/alerts/query/list/)

You should now see 'MyBroker' listed as a broker! Clicking the link will bring you
to the query page, where you can make a query to our sample dataset.

![successful-broker-list]({{"/assets/img/create_broker_doc/success_broker_list.png" | absolute_url}})

### Making a query

Since we're only going to be filtering on the alert's 'name' field, we're only
presented with that option. Name the query whatever you'd like, and we'll check
our remote data source for a target named 'Tatooine'

![example-query]({{"/assets/img/create_broker_doc/example_query.png" | absolute_url}})

Going back to [http://127.0.0.1:8000/alerts/query/list/](http://127.0.0.1:8000/alerts/query/list/),
our new query will appear. Click the 'run' button to run the query.

![populated-query-list]({{"/assets/img/create_broker_doc/populated_query_list.png" | absolute_url}})

The query result will be presented.

![populated-query-list]({{"/assets/img/create_broker_doc/query_result.png" | absolute_url}})

To create a target from any query result, click the 'create target' button. To view the raw
alert data, click the 'view' link.
