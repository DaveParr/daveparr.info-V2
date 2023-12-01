+++
title = "Building my first Django project with a Database"
description = "Even more notes from Django for Beginners by William S. Vincent"
date = 2020-06-05
[taxonomies]
tags = ["python", "django", "database"]
+++

I'm working through [Django for Beginners by William S. Vincent](https://djangoforbeginners.com/). Chapter 4 starts using a database to act as a data store for a Message Board app.

This seemed very related to what my buddies do with Laravel, where migrations are run when the data Model in the backend needs to be changed. I was actually a little surprised that at no point did I need to write any SQL though. As someone who has spent their professional life asking databases for info, I was actually a tiny bit disappointed 😜. 

I'm starting to become curious about this `manage.py` file though. All I have in my workspace to represent it is:

```py
#!/usr/bin/env python
"""Django's command-line utility for administrative tasks."""
import os
import sys


def main():
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'blog_project.settings')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()
```

So it's obviously getting some more code under the hood, but I'm starting to think it has super powers. It starts projects and apps, it makes users, it runs migrations and tests as well as the local server. For something that hasn't even really been mentioned other than a line of text to type into the CLI my curiosity is definitely rising. Unfortunately `python manage.py makecoffee' is the only thing that hasn't worked so far. What are your favourite powers of `manage.py`?