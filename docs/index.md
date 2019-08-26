---
layout: default
---

## Introduction
Start here if you are new to using the TOM Toolkit.

[Architecture](/docs/tomarchitecture) - This document describes the
architecture of the TOM Toolkit at a high level. Read this first if you're
interested in how the TOM toolkit works.

[Getting Started](/docs/getting_started) - First steps for getting a TOM
up and running.

[Workflow](/docs/workflow) - The general workflow used with TOMS.

[Frequently Asked Questions](#frequently-asked-questions) Answers to common issues
and customization tasks.

## Extending and Customizing
Start here to learn how to customize the look and feel of your TOM or
add new functionality.

[Custom Settings](/docs/customsettings) - Settings available to the TOM Toolkit
which you may want to configure.

[Customizing TOM Templates](/docs/customize_templates) - Learn how to override
built in TOM templates to change the look and feel of your TOM.

[Adding new Pages to your TOM](/docs/adding_pages) - Learn how to add entirely new
pages to your TOM, displaying static html pages or dynamic database-driven
content.

[Adding Custom Target Fields](/docs/target_fields) - Learn how to add custom
fields to your TOM Targets if the defaults do not suffice.

[Adding Custom Data Processing](/docs/customizing_data_processing) - Learn how
you can process data into your TOM from uploaded data products.

[Building a TOM Alert Broker](/docs/create_broker) - Learn how to build
an Alert Broker module to add new sources of targets to your TOM.

[Changing Request Submission Behavior](/docs/customize_observations) - Learn how
to customize the LCO Observation Module in order to add additional parameters to
observation requests sent to the LCO Network.

[Creating Plots from TOM Data](/docs/plotting_data) - Learn how to create plots
using plot.ly and your TOM data to display anywhere in your TOM.

[The Permissions System](/docs/permissions) - Use the permissions system to limit
access to targets in your TOM.

[Automating Tasks](/docs/automation) - Run commands automatically to keep your TOM
working even when you aren't

## Advanced Topics
[Background Tasks](/docs/backgroundtasks) - Learn how to set up an aysnchronous
task library to handle long running and/or concurrent functions.

[Building a TOM Observation Facility Module](/docs/observation_module) - Learn to
build a module which will allow your TOM to submit observation requests to
observatories.

[Running Custom Code Hooks](/docs/custom_code) - Learn how to run your own scripts
when certain actions happen within your TOM (for example, an observation
completes).

[Scripting your TOM with Jupyter Notebooks](/docs/scripts/) - Use a Jupyer
notebook (or just a pythong console/scripts) to interact directly with your TOM.

## Deployment
Once you've got a TOM up and running on your machine, you'll probably want to
deploy it somewhere so it is permanently accessible by you and your collegues.

[Deploy to Heroku](/docs/deployment_heroku) - [Heroku](https://heroku.com) is a
PaaS that allows you to publicly deploy your web applications without the need
for managing the infrastructure yourself.

[Using Amazon S3 to Store Data for a TOM](/docs/amazons3) - Enable storing data on
the cloud storage service Amazon S3 instead of your local disk.

## Contributing

[Contribution Guide](/docs/contributing) - If you find an issue, you need help with your TOM, you have a useful idea, or you wrote a module you'd like to be included in the TOM Toolkit, start with the contribution guide.

## Support

[Support Guide](/docs/support) - Looking for help? Want to request a feature? Have questions about Github Issues? Take a look at the support guide.

## Frequently Asked Questions

### Can I use Jupyter Notebooks with my TOM?

Yes. First install jupyterlab into your TOM virtualenv:

    pip install jupyterlab

Inside your TOM directory, use the following management command to launch the
notebook server:

    ./manage.py shell_plus --notebook

Under the new notebook menu, choose "Django Shell-Plus". This will create a new
notebook in the correct TOM context.

There is also a [tutorial](/docs/scripts) on interacting with your TOM using
Jupyter notebooks.

### What are tags on the Target form?
You can add tags to targets via the target create/update forms or
programmatically. These are meant to be arbitrary data associated with a target.
You can then search for targets via tags on the target list page, by entering the
"key" and/or "value" fields in the filter list. They will also be displayed on the
target detail pages.

If you'd like to have more control over extra target data, see the documentation
on [Adding Custom Target Fields](/docs/target_fields).

### I try to observe a target with LCO but get an error.

You might not have added your LCO api key to your settings file under the
`FACILITIES` settings. See [Custom Settings](/docs/customsettings#facilities) for
more details.

### How do I create a super user (PI)?
You can create a new superuser using the built in management command:

    ./manage.py createsuperuser

The `manage.py` file can be found in the root of your project.

Alternatively, you can give a user superuser status if you are already logged
in as a superuser by visiting the admin page for users:
[http://127.0.0.1/admin/auth/user/](http://127.0.0.1/admin/auth/user/)


### My science requires more parameters than are provided by the TOM Toolkit.
It is possible to add additional parameters to your targets within the TOM. See
the documentation on [Adding Custom Target Fields](/docs/target_fields).


### Yuck! My TOM is ugly. How do I change how it looks?
You have a few options. If you'd like to rearrange the layout or information on
the page, you can follow the tutorial on
[Customizing your TOM](/docs/customize_templates). If you'd like to modify colors,
typography, etc you'll want to use CSS.
[W3Schools](https://www.w3schools.com/Css/) is a good resource if you are
unfamiliar with Cascading Style Sheets.


### How do I add a new page to my TOM?
We would recommend you read the [Django tutorial](https://docs.djangoproject.com/en/2.2/contents/)
🙂. But if you want the quick and dirty, edit the `urls.py` (located next to
`settings.py`):

```python
from django.urls import path, include
from django.views.generic import TemplateView

urlpatterns = [
    path('', include('tom_common.urls')),
    path('newpage/', TemplateView.as_view(template_name='newpage.html'), name='newpage')
]
```

And make sure `newpage.html` is located within the `templates/` directory in your
project.

This will make the contents of `newpage.html` available under the path
[/newpage/](http://127.0.0.1/newpage/).
