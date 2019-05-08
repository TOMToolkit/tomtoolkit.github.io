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
