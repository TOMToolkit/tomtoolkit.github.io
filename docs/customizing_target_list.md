---
title: Customizing the Target List
---
# Customizing The Target List

In this tutorial we are going to change our target list page to show both
different target attributes and allow filtering on a different attribute as well.

This tutorial assumes you have already gone through the [Getting
started guide](/docs/getting_started.md).

We'll also be overriding a template in this template. If you've already read the
[Customizing TOM Templates](/docs/customize_templates.md) tutorial this should be
familiar to you.

First, navigate to [http://127.0.0.1/targets](http://127.0.0.1/targets). You
should see something like this:

![Before](/assets/img/customize_views_doc/before.png)

Since we are only interested in sidereal targets, let's start by adding RA and Dec
to the columns displayed, while removing Type.

Just like we did in the [Customizing TOM Templates](/docs/customize_templates.md)
tutorial, we'll copy the original template for the target list into our project
(and make the required configuration changes):

    $ mkdir -p templates/tom_targets
    $ curl -o templates/tom_targets/target_list.html https://raw.githubusercontent.com/TOMToolkit/tom_base/master/tom_targets/templates/tom_targets/target_list.html

Note: make sure you've added your templates directory to the `TEMPLATES` config in
your `settings.py` file:

    'DIRS': [os.path.join(BASE_DIR, 'templates')],

Next, let's edit `templates/tom_targets/target_list.html`. The table is fairly
simple:

{% raw %}
```html
<table class="table">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Type</th>
      <th>Observations</th>
      <th>Saved Data</th>
    </tr>
  </thead>
  <tbody>
    {% for target in filter.qs %}
    <tr>
      <td><a href="{% url 'targets:detail' target.id %}" title="{{ target.identifier }}">{{ target.identifier }}</a></td>
      <td>{{ target.name }}</td>
      <td>{{ target.get_type_display }}</td>
      <td>{{ target.observationrecord_set.count }}</td>
      <td>{{ target.dataproduct_set.count }}</td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```
{% endraw %}

We are simply iterating over the list of targets and displaying them as a table.

Let's modify the html to show RA and Dec and remove the Type column:

{% raw %}
```html
<table class="table">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Observations</th>
      <th>Saved Data</th>
      <th>RA</th>
      <th>Dec</th>
    </tr>
  </thead>
  <tbody>
    {% for target in filter.qs %}
    <tr>
      <td><a href="{% url 'targets:detail' target.id %}" title="{{ target.identifier }}">{{ target.identifier }}</a></td>
      <td>{{ target.name }}</td>
      <td>{{ target.observationrecord_set.count }}</td>
      <td>{{ target.dataproduct_set.count }}</td>
      <td>{{ target.ra }}</td>
      <td>{{ target.dec }}</td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```
{% endraw %}

We've added the two fields to the table header:

```html
<th>RA</th>
<th>Dec</th>
```

And removed the Type header:

```html
<th>Type</th>
```

As well as adding the two fields in the table body:

{% raw %}
```html
<td>{{ target.ra }}</td>
<td>{{ target.dec }}</td>
```
{% endraw %}

And removing the type display:

{% raw %}
```html
<td>{{ target.get_type_display }}</td>
```
{% endraw %}

The result should be the table now displaying RA and Dec:

![After](/assets/img/customize_views_doc/after.png)


This is better, but we still have the Type filter in the filter list to the right.
We'd also like to be able to filter by epoch, so it's tome to customize our view
logic.

First, let's figure out which view is being called when we visit `/targets/`. We
can use the `./manage.py show_urls` command (which is installed by the
[django_extensions](https://github.com/django-extensions/django-extensions) app)
to view a mapping between urls and functions:

    $ ./manage.py show_urls
    ...
    /observations/list/	tom_observations.views.ObservationListView	observations:list
    /observations/manual/	tom_observations.views.ManualObservationCreateView	observations:manual
    /targets/	tom_targets.views.TargetListView	targets:list
    /targets/<pk>/	tom_targets.views.TargetDetail	targets:detail
    /targets/<pk>/delete/	tom_targets.views.TargetDelete	targets:delete
    ...

From that output we can discern that `/targets/` is mapped to the
`tom_targets.views.TargetListView` class.

We can view the source code of this class [on
Github](https://github.com/TOMToolkit/tom_base/blob/master/tom_targets/views.py)

```python
class TargetListView(FilterView):
    template_name = 'tom_targets/target_list.html'
    paginate_by = 25
    model = Target
    filterset_class = TargetFilter
```

This is pretty basic view, but the important part is the `TargetFitler`. This is a
[django-filter](https://django-filter.readthedocs.io/en/master/index.html)
FilterSet class and it's the class we'll want to change to add and remove filters.

By taking a look at the filter class [on
Github](https://github.com/TOMToolkit/tom_base/blob/master/tom_targets/filters.py)
we can see the values we'd like to change: we should remove `type` from the
`fields` attribute of the Meta class and add `epoch`.

Let's create a new file, `filters.py` in our `tom_custom_target` directory and
enter our changed code:

```python
import django_filters

from tom_targets.models import Target


class CustomTargetFilter(django_filters.FilterSet):
    key = django_filters.CharFilter(field_name='targetextra__key', label='Key')
    value = django_filters.CharFilter(field_name='targetextra__value', label='Value')

    class Meta:
        model = Target
        fields = ['epoch', 'identifier', 'name', 'key', 'value']
```

We've replaced `type` with `epoch`. Everything else remains the same.

Now let's create our new View class that can use this filter, `views.py` in our
`tom_custom_target` directory:

```python
from tom_targets.views import TargetListView

from tom_custom_target.filters import CustomTargetFilter


class CustomTargetListView(TargetListView):
    filterset_class = CustomTargetFilter

```

Now we have a new view class that inherits all the functionality of the original,
but uses a different filter class.

The last step is to remap the `/targets/` url to our new view.

If you followed the [Getting
started guide](/docs/getting_started.md) tutorial, you probably set the
`ROOT_URLCONF` setting in `settings.py` to `tom_common.urls`. Now that we want to
customize our url routing, we'll have to change this back to it's default value:

    ROOT_URLCONF = 'tom_custom_target.urls'

We'll then edit our `tom_custom_target/urls.py` file to include the base TOM
urlconf, as well as including our new custom `/targets/` route.

Edit `tom_custom_target/urls.py`:

```python
from django.contrib import admin
from django.urls import path
from django.urls import include

from tom_custom_target.views import CustomTargetListView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('targets/', CustomTargetListView.as_view()),
    path('', include('tom_common.urls')),
]
```

What we've done here.
