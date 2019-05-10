---
title: Changing How Observations are Submitted
---

The LCO Observation module for the TOM Toolkit ships with a default HTML form that
facilitates submitting basic observations to the LCO network. It may sometimes be
desirable to customize the form to show or hide fields, add new parameters, or
change the submission logic itself, depending on the needs of the project. In this
tutorial we will customize our LCO module to submit multiple observations with
different filters at the same time.

This guide assumes you have followed the [getting
started](/docs/getting_started) guide and have a working TOM up and running.

### Create a new Observation Module

Many methods of customizing the TOM Toolkit involve inheriting/extending existing
functionality. This time will be no different: we'll crate a new observation
module that inherits the existing functionality from
`tom_observations.facilities.LCOFacility`.

First, create a python file somewhere in your project to house your new module.
For example it could live next to your `settings.py`, or if you've started a new
app, it could live there. It doesn't really matter, as
long as it's located somewhere in your project:

    touch mytom/mytom/lcomultifilter.py

Now add some code to this file to create a new observation module:

```python
# lcomultifilter.py
from tom_observations.facilities.lco import LCOFacility


class LCOMultiFilterFacility(LCOFacility):
    name = 'LCOMultiFilter'
```
So what does the above code do?

1. Line 1 imports the LCOFacility that is already shipped with the TOM Toolkit. We
want this class because it contains functionality we will re-use in our own
implementation.
2. Line 4 defines a new class named `LCOMultiFilterFacility` that inherits from
`LCOFacility`.
3. Line 5 sets the name attribute of this class to `LCOMultiFilter`.

What you have done is created a new observation module that is functionally
identical to the existing LCO module, but has a different name: `LCOMultiFilter`.
A good start!

Now we need to tell our TOM where to find our new module so we can use it to
submit observations. Add (or edit) the following lines to your `settings.py`:

```python
# settings.py
TOM_FACILITY_CLASSES = [
    'tom_observations.facilities.lco.LCOFacility',
    'tom_observations.facilities.gemini.GEMFacility',
    'mytom.lcomultifilter.LCOMultiFilterFacility',
]
```
This code lists all of the observation modules that should be available to our
TOM.

With that done, go to any target in your TOM and you should see your new module in
the list:

![observe button](/assets/img/customize_observations/observebutton.png)

You could now use the new module now to make an observation, and it would work the
same as the old LCO module.

Note that if you see an error like: "There was a problem authenticating with LCO"
then you need to [add your LCO api key](/docs/customsettings#facilities) to your
`settings.py` file.

### Adding additional fields

Now that you've created a new observation module that's functionally the same as
the old LCO module, how do we change it? One thing that might be useful is to add some extra
fields to the form: two more choices of filters and exposure times. Back in the
`lcomultifilter.py` file add a new import and create a new class that will become
the new form:

```python
# lcomultifilter.py
from tom_observations.facilities.lco import LCOFacility, LCOObservationForm, filter_choices
from django import forms


class LCOMultiFilterForm(LCOObservationForm):
    filter2 = forms.ChoiceField(choices=filter_choices)
    exposure_time2 = forms.FloatField(min_value=0.1)
    filter3 = forms.ChoiceField(choices=filter_choices)
    exposure_time3 = forms.FloatField(min_value=0.1)


class LCOMultiFilterFacility(LCOFacility):
    name = 'LCOMultiFilter'
    form = LCOMultiFilterForm
```

There is now a new class, `LCOMultiFilterForm` which inherits from
`LCOObservationForm`, the form for the default interface. Additionally there are
definitions for 4 fields: `fiter2`, `exposure_time2`, `filter3`, and
`exposure_time3`.

A `form` attribute has been added on the `LCOMultiFilterFacility`
class, this tells our observation module to use the new `LCOMultiFilterForm`
instead of the default LCO observation form.


### Modifying the form layout

Now that the desired fields have been added to the `LCOMultiFilterForm`, the
form's layout needs to be modified in order to actually display them. In this
example we'll split the form into two rows: one row for the three filter choices
and exposure times, and another row for everything else. Note that the default
form already has fields for `filter` and `exposure_time`, so we'll overwrite the
entire layout so that they appear next to the new fields we added.

The `LCOObservationForm` has a method `layout()` that returns the desired layout
using the [cripsy forms Layout](https://django-crispy-forms.readthedocs.io/en/d-0/layouts.html)
class. Familiarizing yourself with the basic functionality of crispy forms would
be a good idea if you wish to deeply customize your observation module's form.

With our modified layout added, the `lcomultifilter.py` file now looks like this:

```python
# lcomultifilter.py
from tom_observations.facilities.lco import LCOFacility, LCOObservationForm, filter_choices
from django import forms
from crispy_forms.layout import Div


class LCOMultiFilterForm(LCOObservationForm):
    filter2 = forms.ChoiceField(choices=filter_choices)
    exposure_time2 = forms.FloatField(min_value=0.1)
    filter3 = forms.ChoiceField(choices=filter_choices)
    exposure_time3 = forms.FloatField(min_value=0.1)

    def layout(self):
        return Div(
                Div(
                    Div(
                        'group_id', 'proposal', 'ipp_value', 'observation_type', 'start', 'end',
                        css_class='col'
                    ),
                    Div(
                        'instrument_name', 'exposure_count', 'max_airmass',
                        css_class='col'
                    ),
                    css_class='form-row'
                ),
                Div(
                    Div(
                        'filter', 'exposure_time',
                        css_class='col'
                    ),
                    Div(
                        'filter2', 'exposure_time2',
                        css_class='col'
                    ),
                    Div(
                        'filter3', 'exposure_time3',
                        css_class='col'
                    ),
                    css_class='form-row'
                )
        )


class LCOMultiFilterFacility(LCOFacility):
    name = 'LCOMultiFilter'
    form = LCOMultiFilterForm
```

Take a look at the layout and compare it to the [existing lco layout](). A second
row has been added that includes all the filter choices. Note that the original
`filter` and `exposure_time` have been moved from their original location to the
new row.

Now if you select "LCOMultiFilter" from the list of observation facilities on a
target you should see your new form:

![newform.png](/assets/img/customize_observations/newform.png)
