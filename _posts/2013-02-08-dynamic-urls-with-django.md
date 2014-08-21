---
layout: post
title: Dynamic URLs with Django
date: 2013-02-08 11:51
author: Giacomo Graziosi
comments: true
categories: [python, django]
tags: [cms, corbucci, django, middleware, open source, programming, python]
---
While implementing a configurable permalink structure feature for the Corbucci CMS project I had the need to inject new URLs in the Django routing mechanism. As the URLs are normally designed to fit in the [hardcoded settings.py](https://docs.djangoproject.com/en/dev/topics/http/urls/#example) I had to find another way to edit them at runtime and a middleware was the way to go: as you can read in the [documentation](https://docs.djangoproject.com/en/dev/topics/http/urls/#how-django-processes-a-request) the process_request allows to define an urlconf attribute, when processing the request Django will search for the .urlconf.urlpatterns attribute and this will be used as the "standard URLs" for the routing.

This is how I implemented it:

{% highlight python %}
class GlueMiddleware(object):
    def process_request(self, request):
        urlpatterns = patterns('',
            url(r'^articles/$', ArticleListView.as_view()),
            (r'^', include('urls')),
        )
        u = type('DynUrlConf', (object,), dict(urlpatterns = urlpatterns))
        request.urlconf = u
{% endhighlight %}



Lines 3-6 contain the URLs in the standard format for Django, line 4 is the "dynamic" one, line 5 just include the previous ones (as this will override them). Lines 7 is used to create a class with the urlpatterns attribute at runtime, check the [type function documentation](http://docs.python.org/2/library/functions.html#type) for further details.</pre>
