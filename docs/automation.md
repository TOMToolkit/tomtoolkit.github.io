---
title: Automating tasks for your TOM
---

Your TOM may have a need to run a task on a regular schedule without human intervention. With the help of a built-in Django feature and cron, this can be accomplished. Perhaps you want to check for and download data from your scheduled observations every hour, or see if any brokers have published new candidates that meet the criteria of a previous search--all that would be required is a bit of code to call those built-in functions, and a crontab update.

## Create a management command

Django provides the ability to register actions using [management commands](https://docs.djangoproject.com/en/2.2/howto/custom-management-commands/). These actions can then be called from the command line.

Let's walk through a command to download observation data every hour. The first thing to be done is to create a `management/commands` directory within your application. The structure should look like this:

```
mytom/
├── manage.py
└── app/
    ├── __init__.py
    ├── models.py
    ├── tests.py
    ├── views.py
    └── management/
        └── commands/
            └── updatedata.py
```

A management command simply needs a class called `Command` that inherits from `BaseCommand`, and a `handle` class method that contains the logic for the command.

```python
from django.core.management.base import BaseCommand
from tom_observations import facility
from tom_observations.models import ObservationRecord


class Command(BaseCommand):

  help = 'Downloads data for all completed observations'

  def handle(self, *args, **options):
```

Now, we need to add the logic to query the facilities for data. First, we instantiate a class object for each facility:

```python
    facility_classes = {}
    for facility_name in facility.get_service_classes():
      facility_classes[facility_name] = facility.get_service_class(facility_name)()
```

Then, we iterate over each incomplete `ObservationRecord`, update the status in the database, and save the data products locally for that ObservationRecord.

```python
    observation_records = ObservationRecord.objects.all()
    for record in observation_record:
      if record.status not in facility_classes[record.facility].get_terminal_observing_states():
        facility_classes[record.facility].update_observation_status(record.observation_id)
        facility_classes[record.facility].save_data_products(record)
```

So our final management command should look like this:

```python
from django.core.management.base import BaseCommand
from tom_observations import facility
from tom_observations.models import ObservationRecord


class Command(BaseCommand):

  help = 'Downloads data for all completed observations'

  def handle(self, *args, **options):
    facility_classes = {}
    for facility_name in facility.get_service_classes():
      facility_classes[facility_name] = facility.get_service_class(facility_name)()
    observation_records = ObservationRecord.objects.all()
    for record in observation_records:
      if record.status not in facility_classes[record.facility].get_terminal_observing_states():
        facility_classes[record.facility].update_observation_status(record.observation_id)
        facility_classes[record.facility].save_data_products(record)

    return 'Success!'
```

### Adding parameters

Management commands also provide the ability to accept parameters. Doing this is as simple as implementing `add_arguments` as a class method on your `Command` class. Let's say we want to ensure that our command can be run for a single target:

```python
  def add_arguments(self, parser):
    parser.add_argument('--target_id', help='Download data for a single target')
```

That code will process any additional parameters, and we simply need to handle them in our, well, `handle` class method.

```python
  def handle(self, *args, **options):
    if options['target_id']:
      try:
        target = Target.objects.get(pk=options['target_id'])
      except ObjectDoesNotExist:
        raise Exception('Invalid target id provided')
        
    facility_classes = {}
    for facility_name in facility.get_service_classes():
    ...
```

Finally, we filter our initial set of observation records, so this line:

```python
    observation_records = ObservationRecord.objects.all()
```

will become this:

```python
    observation_records = ObservationRecord.objects.filter(target=target)
```

And our final finished command looks as follows:

```
from django.core.management.base import BaseCommand
from tom_observations import facility
from tom_observations.models import ObservationRecord


class Command(BaseCommand):

  help = 'Downloads data for all completed observations'
  
  def add_arguments(self, parser):
    parser.add_argument('--target_id', help='Download data for a single target')

  def handle(self, *args, **options):
    if options['target_id']:
      try:
        target = Target.objects.get(pk=options['target_id'])
      except ObjectDoesNotExist:
        raise Exception('Invalid target id provided')
    facility_classes = {}
    for facility_name in facility.get_service_classes():
      facility_classes[facility_name] = facility.get_service_class(facility_name)()
    observation_records = ObservationRecord.objects.filter(target=target)
    for record in observation_records:
      if record.status not in facility_classes[record.facility].get_terminal_observing_states():
        facility_classes[record.facility].update_observation_status(record.observation_id)
        facility_classes[record.facility].save_data_products(record)

    return 'Success!'
```

## Automating a management command

### Using cron

On Unix-based systems, [cron](https://linux.die.net/man/8/cron) can be used to automate running of a Django management command. The syntax is very simple, as commands look like this:

`30 2 * 6 3 /path/to/command /path/to/parameters`

In the above case, the first five values, which can either be numbers or asterisks, represent elements of time. From left to right, they are minutes, hours, day of the month, month of the year, and day of the week. Our example would run a command every Wednesday (fourth day of the week, starting from 0) in June (sixth month of the year, starting from 1) at 2:30 AM.

Scheduling can be made more complex as well--values can be comma-separated or presented as a range. Refer to the abundance of cron documentation for more information. An excellent beginner's guide can be found [here](https://www.ostechnix.com/a-beginners-guide-to-cron-jobs/).

Now, how is cron called? Well, cron jobs are run by the system, and it reads the commands that need to be called from a cron table, or crontab. To edit this file, simple call `crontab -e`.

### Using cron with a management command

To make this more specific to our example, let's say we want to update the observation data every hour. The command we would normally run in our project directory would be the following:

`python manage.py updatedata`

However, cron is a system-level operation, so the command needs to be directory-agnostic, and we need to ensure we're using the right Python version. If you have a virtualenv, the command should be the absolute path to the Python interpreter in the virtualenv. If your TOM is in a Docker container, it should be the version of Python running in the container. Otherwise, just ensure that it's at least version 3.6 or higher.

So, the line in our crontab should be as follows:

`0 * * * * /path/to/virtualenv/bin/python /path/to/project/manage.py updatedata`

This will run every day on the hour. And that's it! Just exit the crontab and it will automatically restart cron, then your command will run on the next hour.
