# Django Tips
Some django tips

<!-- TOC -->

- [Django Tips](#django-tips)
- [Settings Layouts](#settings-layouts)
    - [Python Way](#python-way)
    - [12 Factor Way](#12-factor-way)

<!-- /TOC -->

# Settings Layouts

By default Django will create your application `settings.py`. It will
add this string to `manage.py` and `wsgi.py`.

I've seen three major settings setup's for Django applications for SDLC branches
of Development, Testing, Staging, Production -- or whichever you choose to use.

I'll compare two of them.

## Python Way
Without having to change those files, you can easily create a settings
area with a private environment. This is my favorite way.

**Settings Tree**
```
settings/
    __init__.py
    base.py             <-- original settings.py file
    develop.py
    staging.py
    production.py
    local.py            <-- private settings (.gitignore)
    local.example.py
```

```py
# settings/__init__.py
from base import *
```

```py
# settings/local.py
from develop import *
#or
from staging import *
#or
from production import *

# Secret Settings per Local Machine/Server
# ----------------------------------------
SECRET_KEY = ''
DATABASE = ''
```

**Working in a Team**
It is easy to work with a team using these settings. All a user must do is copy the `local.example.py`
file to `local.py` and customize without fear of committing secret settings.

## 12 Factor Way
There are many various of this setup. It often requires an extension to read parameters
from a file, for example:

- python-dotenv
- django-environ

These follow the 12 factor principal. It attempts to set ENVIRONMENT variables, however
it does not "really" set environment variables, at least not with django-environ.

**Settings Tree**
There are several approaches to this, I'll use one similar to the other.
```
settings/
    __init__.py
    base.py             <-- original settings.py file
    develop.py
    staging.py
    production.py
    .env                <-- private settings (.gitignored)
    .env.example
```

**Where things get Difficult**
The initial setup can take a while depending on how you want to do it. However, that's not the biggest
problem.

The majority of us will use a virtual environment, such as: `virtualenv` or `virtualenvwrapper`.
When it comes time for deployment if you are using `virtualenvwrapper` and `gunicorn` as a starting
point, you will need to create custom virtual environment hooks (`postactivate`, `postdeactivate`) which, is
not a bad thing. Yet, you use these hooks to get the `.env` data, that must get into Django before
the `DJANGO_SETTINGS_MODULE` is loaded, after all we are defining what settings module to use.

This can be frustrating as your `.env` file is better off being a bash script which exports variables.
Then it is sourced from `postactivate` to export them to the environment. You do not want to source your
`.env` within `postactive` from a relative path, this will cause problems from your local server to your
production servers.

What's more, is that unless you create a bash array to set/unset these variables, you have fixed variables
in your .env that you must change in two files if you choose to add/remove one. I have done this with my
`.env` container a bash array of settings that are dynamically looped and set in the environment with an export.

I then updated `postactivate` to trigger `.env` by it's absolute path so that Django would read those settings.
Since you cannot export an array in bash, I had to export a string of the Django setting variables as keynames in
a string like: `DJANGO_SETTINGS_MODULE,SECRET_KEY,DATABASE`. When it came time to `postdeactivate` I had to turn
that string into an array to loop through and unset each variable.
