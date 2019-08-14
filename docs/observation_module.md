---
title: Writing an observation module to interface with observatories
---

This guide will walk you through how to create a custom observation facility
module using some mocked up endpoints to simulate a real observatory interface.

You can use this example as the foundation to build an observing facility module
to connect to a real observatory.

Be sure you've followed the [Getting Started](/docs/getting_started) guide before continuing onto this tutorial.

## What is a observing facility module?

A TOM Toolkit observing facility module is a python module which contains the code
necessary to provide an interface to an observing facility in a TOM. Some examples
of existing modules are the [Las Cumbres
Observatory](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facilities/lco.py)
and the
[Gemini](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facilities/gemini.py)
modules. Both allow the submission of observation requests to their respective
observatories through a TOM.

### Prerequisites

You should have a working TOM already. You can start where the [Getting
Started](/docs/getting_started) guide leaves off. You should also be familiar with
the observing facility's API that you would like to work with.

### Defining the minimal implementation

Within any existing module in your TOM you should create a new python module
(file) named `myfacility.py`. For example, if you have a fresh TOM installation
you'll have a directory structure that looks something like this:

    ├── data
    ├── db.sqlite3
    ├── manage.py
    ├── mytom
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── static
    ├── templates
    └── tmp

We'll place our `myfacility.py` file inside the `mytom` directory, next to
`settings.py`.

```python
from tom_observations.facility import GenericObservationFacility, GenericObservationForm


class MyObservationFacilityForm(GenericObservationForm):
    pass


class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    observation_types = [('IMAGING', 'Imaging')]
```

We'll go over what these lines mean soon. First, we'll add a setting to our
project's `settings.py` to tell the TOM Toolkit to use our new class:

```python
TOM_FACILITY_CLASSES = [
    'tom_observations.facilities.lco.LCOFacility',
    'tom_observations.facilities.gemini.GEMFacility',
    'mytom.myfacility.MyObservationFacility'
]
```

Now go ahead and view a target in your TOM, you should see something like this:

![myfacility](/assets/img/observation_module/myfacility.png)

This means our new observation facility module has been successfully loaded.


### GenericObservationFacility and GenericObservationForm

You will have noticed our module consists of two classes that inherit from two
other classes.

`MyObservationFacility` is the class that will contain the "business logic"
for interacting with the remote observatory. This includes methods to submit
observations, check observation status, etc. It inherits from
`GenericObservationFacility`, which contains some functionality that all
observation facility classes will want.

`MyObservationFacilityForm` is the class that will display a GUI form for our
users to create an observation. We can submit observations programatically, but it
is also nice to have a GUI for our users to use.  The `GenericObservationForm`
class, just like the previous super class, contains logic and layout that all
observation facility form classes should contain.

### Implementing observation form selection

You may have noticed that we added a variable for `observation_types` in
`MyObservationFacility`. Even if the facility you're adding only supports one
observation type, you'll still need to add a list of `observation_types` tuples
so that the TOM view knows what options to offer the user. The tuples should be
2-tuples, with the first element being the type enum, and the second element
being the display name of the observation type.

If you only have one type of observation coming through, or you simply plan to
put all of the submission logic in a single form, `MyObservationFacility` can look
like it does above:

```python
from tom_observations.facility import GenericObservationFacility, GenericObservationForm


class MyObservationFacilityForm(GenericObservationForm):
    pass


class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    observation_types = [('OBSERVATION', 'Observation')]
```

However, if you want to separate your logic into multiple forms, either by observation
types, or even just for multiple simple forms with a number of preselected options,
`MyObservationFacility` may look like this:

```python
from tom_observations.facility import GenericObservationFacility, GenericObservationForm


class MyObservationFacilityForm(GenericObservationForm):
    pass


class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    observation_types = [('IMAGING', 'Imaging'), ('SPECTROSCOPY', 'Spectroscopy')]
```

Now, we've defined what types of observations our facility will support, but we need to
choose the right one. The `GenericObservationFacility` also has a method you'll need to
implement that decides which form to render to the user called `get_form`. If you're only
adding one form to your facility, it can look as simple as this:

```python
class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    observation_types = [('OBSERVATION', 'Observation')]

    def get_form(self, observation_type):
        return MyObservationFacilityForm
```

If you're implementing multiple observation forms, however, you'll need a small amount
of business logic to return the correct form:

```python
class MyObservationFacilityImagingForm(GenericObservationForm):
    pass


class MyObservationFacilitySpectroscopyForm(GenericObservationForm):
    pass


class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    observation_types = [('IMAGING', 'Imaging'), ('SPECTROSCOPY', 'Spectroscopy')]

    def get_form(self, observation_type):
        if observation_type == 'IMAGING':
            return MyObservationFacilityImagingForm
        else:
            return MyObservationFacilitySpectroscopyForm
```

### Implementing observation submission

Now that we have the skeleton on an observation module set up, we should make it
do something. Let's assume our observatory only requires us to send 2 parameters
(besides the target data): exposure\_time and exposure\_count. Let's start by
adding them to our form class:

```python
from django import forms
from tom_observations.facility import GenericObservationFacility, GenericObservationForm


class MyObservationFacilityForm(GenericObservationForm):
    exposure_time = forms.IntegerField()
    exposure_count = forms.IntegerField()
```

Notice that we've added the two field definitions on our form. We've also imported
the django form module with `from django import forms`.

Now if we click on the "MyFacility" button on the observation facility list on the
target page, we should see something like this:

![fields](/assets/img/observation_module/fields.png)


Now we'll implement the method that performs an action with our form when we
submit the observation request:

```python
class MyObservationFacility(GenericObservationFacility):
    name = 'MyFacility'
    form = MyObservationFacilityForm

    def submit_observation(self, observation_payload):
        print(observation_payload)
        return [1]

    def validate_observation(self, observation_payload):
        pass

    def get_observation_url(self, observation_id):
        return ''

    def get_observation_status(self, observation_id):
        return ['IN_PROGRESS']

    def get_terminal_observing_states(self):
        return ['IN_PROGRESS', 'COMPLETED']

    def get_observing_sites(self):
        return []

    def data_products(self, observation_id, proudct_id=None):
        return []

```

The important method here is `submit_observation`. This method, when implemented
fully, will send the observation payload to the remote observatory and then return
a list of observation ids. Those Ids will be stored in the database to be used
later: in methods like `get_observation_status(self, observation_id)`. In our
dummy implementation, we simply print out the observation payload and return
single fake id with  `return [1]`.

If you now "submit" an observation using the MyFacility module, you should see
this in the server console:

    {'target_id': 1, 'params': '{"facility": "MyFacility", "target_id": 1, "exposure_time": 23, "exposure_count": 2}'}

That was our print statement!

### Filling in the rest of the functionality
You'll notice we added many more methods other than `submit_observation` to our
Facility class. For now they return dummy data, but when you adapt it to work with
a real observatory you should fill them in with the correct logic so that the
whole module works correctly with the TOM. You can view explanations of each
method [in the source
code](https://github.com/TOMToolkit/tom_base/blob/master/tom_observations/facility.py#L135)


Happy developing!
