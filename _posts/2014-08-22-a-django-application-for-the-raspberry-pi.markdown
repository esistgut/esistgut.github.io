---
layout: post
title:  "A Django / jQuery Mobile application for the Raspberry PI"
date:   2014-08-22 11:41:00
author: Giacomo Graziosi
categories: [embedded, linux, django]
tags: [python, django, jquery mobile, jquery, mobile, raspberry, raspberry pi, rpi, linux, embedded, hardware]
---

<div class="embed-responsive embed-responsive-16by9">
    <iframe src="//www.youtube.com/embed/c-sOYgwTOj0" allowfullscreen></iframe>
</div>

---
This is a late post about a skeleton project I published some time ago: [django-rpi-jqm-sample](https://github.com/esistgut/django-rpi-jqm-sample).
It features a sample application composed of two parts: the first one is a standard Django application, the second one is a [stand-alone daemon](https://github.com/esistgut/django-rpi-jqm-sample/blob/master/rpi_daemon.py). The daemon should be started before the Django application, you can find the details in the above Youtube video.

<!--more-->

The two components communicate with each other using the Django's cache framework. Every time the jQuery Mobile form is submitted the data is pushed to the cache:

{% highlight python %}
def form_valid(self, form):
    cache.set('servo', form.cleaned_data['servo'])
    cache.set('led1', form.cleaned_data['led1'])
    cache.set('led2', form.cleaned_data['led2'])
    return super(Demo1View, self).form_valid(form)
{% endhighlight %}

The same happens the other way around, every time the form is initialized it gets its initial data from the cache:

{% highlight python %}
def get_initial(self):
    servo = cache.get('servo', 0)
    led1 = cache.get('led1')
    led2 = cache.get('led2')
    initial = {'servo': servo, 'led1': led1, 'led2': led2}
    return initial
{% endhighlight %}

The daemon will do continous polling on the cache data and take proper actions when changes are detected:

{% highlight python %}
while True:
    cache_tmp['servo'] = cache.get('servo', "0")
    if cache_cur['servo'] != cache_tmp['servo']:
        servo_control.set_servo(25, int(1200/60*float(cache_tmp['servo']))+900)
        cache_cur['servo'] = cache_tmp['servo']
{% endhighlight %}

Of course the cache framework is not supposed to be used like this, a proper message queueing solution like [RabbitMQ](http://www.rabbitmq.com/) is usually the way to go for this kind of problems, it just seemed a little bit overkill in this particular context so I opted for the cache framework hack :smile:.
