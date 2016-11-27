---
layout: post
title:  "USING CELERY AND REDIS(MQ) IN YOUR DJANGO PROJECT IN 6 SIMPLE STEPS"
date:   2016-03-22
---

### Step 1. Prepare ingredients

Install redis (using `brew install redis`, `apt-get`), celery and flower (both using `pip install`) and celery\[redis] (`pip install celery[redis]`)

* What is [Redis][redis]?
* What is [Celery][celery]?
* What is [Flower][flower]?

### Step 2. Prep you Django project

Celery has to know that your Django project will be using Celery & workers

#### 1. Make a celery.py at the following path: myproj/myproj/celery.py

{% highlight python %}
from __future__ import absolute_import

import os

from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.settings')

from django.conf import settings  # noqa

app = Celery('proj')

# Using a string here means the worker will not have to
# pickle the object when using Windows.
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)
{% endhighlight %}

#### 2. Add the following to your projects myproj/settings.py

`BROKER_URL = 'redis://localhost:7379/0'`

#### 3. Add the following to your myproj/myproj/__init__.py

{% highlight python %}
from __future__ import absolute_import
from .celery import app as celery_app
{% endhighlight %}

### Step 3. Make a task module in your app and make run routine – a task you want celery workers to do

In your app where you want to make tasks, make a task module. (It can either be a `task.py` or a task folder with `init.py` in it. Celery will go through all your apps defined in `settings.py`’s INSTALLED_APPS and look for the task module.

in `myapp/tasks/my_app_task.py`

{% highlight python %}
from celery import Task

from myapp.models import MyModel

class MyAppTask(Task):

    def run(self, *args):
        # do what you have to do inside the run function
{% endhighlight %}

### Step 4. Go to your app’s model or view(wherever you want to hand over the task) and make Task instance.

In this example, we will hand over a task inside a model method

{% highlight python %}
class MyModel(models.Model):

    ...

    def do_something_to_model(self):
        from myapp.tasks import MyAppTask
        task = MyAppTask()
        task.delay(mymodel.id)

{% endhighlight %}

Take note that the task instance’s delay method has to take in the input exactly the same as the run method defined in the task.

### Step 5. Open 3 terminals and run redis, celery and flower.

1. `$ redis-server`
2. `$ celery --workdir=flexx/ --app=flexx.celery:app worker`
3. `$ celery --workdir=flexx/ --app=flexx.celery:app flower`

When running flower, make sure that your tasks are enlisted in Registered tasks:

### Step 6. If you want to see if it works, Run your Django project and run tasks. Then go to localhost:5555 and monitor your tasks.

In `localhost:5555` you will be greeted by a website(GUI) and in that site you will be able to monitor almost everything celery is doing.


[redis]: http://redis.io/topics/introduction 
[celery]: http://www.celeryproject.org/
[flower]: http://flower.readthedocs.org/en/latest/

