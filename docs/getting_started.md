---
title: Getting Started with the TOM Toolkit
---

# Getting Started with the TOM Toolkit

So you've decided to run a Target Observation Manager. This article will help you get started.

The TOM Toolkit is a [Django](https://django-project.org) project. This means you'll be running
an application based on the Django framework when you run a TOM. If you decide to customize
your TOM, you'll be working in Django. You'll likely need some basic understanding of python
and we recommend all users work their way through the
[Django tutorial](https://docs.djangoproject.com/en/2.1/contents/) first before starting with
the TOM Toolkit. It doesn't take long, and you most likely won't need to utilize any advanced
features.

Ready to go? Let's get started.

## Prerequisites

The TOM toolkit requires Python >= 3.7

If you are using Python 3.6 and cannot upgrade to 3.7, install the `dataclasses`
backport:

    pip install dataclasses

## Installing Django

First, you'll need to install Django. We recommend using a
[virtual environment](https://docs.python.org/3/tutorial/venv.html) for your project:

    python3 -m venv tom_env/

Now that we have created the virtual environment, we can activate it:

    source tom_env/bin/activate

You should now see `(tom_env)` prepended to your terminal prompt.

Now, install Django...

    pip install django

...and create a new project, just like in the tutorial:

    django-admin startproject mytom

You should now have a fully functional standard Django installation in `mytom`. Now you can run migrations,
add a super user, and configure the project to your liking. Again, we highly recommend working through
the [Django tutorial](https://docs.djangoproject.com/en/2.1/contents/) if you have not already.

## Installing the TOM Toolkit

The TOM Toolkit is a collection of Django apps that can be used in any Django project. To download it,
use pip:

    pip install https://github.com/TOMToolkit/tom_base/archive/master.zip

Note: the toolkit is not available on PyPI yet, which is why we install from github, but it will be soon.

The above command should pull down `tomtoolkit` and all it's dependencies.

## Adding tomtoolkit to your Django project


First, add the following apps to your project's `settings.py` `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'django.contrib.sites',
    'tom_common',
    'django_comments',
    'bootstrap4',
    'crispy_forms',
    'django_filters',
    'django_gravatar',
    'tom_targets',
    'tom_catalogs',
    'tom_alerts'
    'tom_observations',
]
```

Note that `tom_common` must come before `django_comments`.

The following settings should also be added to your project's `settings.py`:

The TOM Toolkit relies on the Django Sites framework, so we'll add this required setting:

    SITE_ID = 1

In order to be able to access the TOM views, we'll set the projects root url conf to `tom_common`'s:

    ROOT_URLCONF = 'tom_common.urls'

Note that if you'd like to add additional custom views and urls later, you can
[include the urlconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/#including-other-urlconfs) instead.

We also need to set the directory where data products should go. Create a
directory `data` in the project root. Then add the following settings:

    MEDIA_URL = '/data/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'data')

Finally, in order to get good looking forms, let's enable the bootstrap4 crispy pack:

    CRISPY_TEMPLATE_PACK = 'bootstrap4'


## Running the dev server

Now that the toolkit is installed, you're ready to try it out!

First, run the necessary migrations:

    ./manage.py migrate

Now, start the dev server:

    ./manage.py runserver

Your new TOM should now be running on [http://127.0.0.1:8000](http://127.0.0.1:8000)!
