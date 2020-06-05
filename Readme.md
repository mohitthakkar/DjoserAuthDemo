# Django 3.0 API with Djoser Authentication

Our API will provide user registration and management services 
for a single page app. We’ve selected Djoser to help us accomplish 
these tasks. 

We create a new empty directory (DjoserAuthDemo), and inside it set up 
our virtual environment. 

Create Directory
> mkdir DjoserAuthDemo

> cd DjoserAuthDemo/

Create virtual environment
> virtualenv -p python3 venv

Activate virtual environment
> . venv/bin/activate

Install packages
> pip install -r requirements.txt

Create App
> django-admin startproject myapp

Update `myapp/myapp/settings.py`, adding the Django REST Framework 
and Djoser to our INSTALLED_APPS section

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'djoser',
]
```

Next, we’ll wire up routes to the Djoser views, giving us a basic 
user registration API. Edit `myapp/myapp/urls.py` to look as follows:

```
from django.urls import re_path, include

urlpatterns = [
    re_path(r'^api/', include('djoser.urls')),
]
```

Migrate tables to create test database
> python manage.py migrate

Create superuser
> python manage.py createsuperuser

Run server locally
> python manage.py runserver

You can point your web browser to http://127.0.0.1:8000/api/ and be 
pleasantly surprised that you’ve already set up endpoints for creating,
editing, and deleting users, as well as resetting lost passwords.

Now, after creating a superuser, Django will store it in its default
`auth_user` table. Use the following command from your terminal to 
query the database for user details. 

> sqlite3 db.sqlite3 -header "SELECT * FROM auth_user;"
 
This table has several fields that we do not require. So let us create
a custom user model, getting rid of the fields we won’t be using. We’re 
going to simplify our API to just have an email address and a password, 
and use them together to allow user logins.

Let us create a new app to handle our custom user model. Execute the 
following command from your top level directory.

> python manage.py startapp myuser

Inside this App, we’ll modify models.py, defining a custom schema for 
our simpler user model. We will define two new classes, MyUserManager 
and MyUser. Inside MyUserManager we define two standard Django functions 
for user creation, create_user and create_superuser. We enforce that a 
unique, non-blank email be provided, raising an error if not.

Add the following code to models.py

```
from django.db import models
from django.contrib.auth.base_user import AbstractBaseUser, BaseUserManager

class MyUserManager(BaseUserManager):
    def create_user(self, email, password=None):
        """
        Creates and saves a User with the given email and password.
        """
        if not email:
            raise ValueError('Users must have an email address')
        user = self.model(
            email=self.normalize_email(email),
        )
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password):
        """
        Creates and saves a superuser with the given email and
        password.
        """
        user = self.create_user(
            email,
            password=password,
        )
        user.is_admin = True
        user.save(using=self._db)
        return user
``` 

For the MyUser class we inherit from Django's AbstractBaseUser class. 
We create columns for the user’s email address, as well as boolean 
flags indicating if the user is_active and/or is_admin. We don’t need 
to create a field for the password, as we inherit this through the 
AbstractBaseUser. Finally, we tell Django that the email field is our 
USERNAME_FIELD in our custom user module.

Add the following code to models.py

```
class MyUser(AbstractBaseUser):
    email = models.EmailField(verbose_name='email address', max_length=255, unique=True)
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)
    objects = MyUserManager()
    USERNAME_FIELD = 'email'
    def __str__(self):
        return self.email
```

In order to use this new model, we’ll need to enable our custom App 
in myapp/myapp/settings.py, adding it to the end of the INSTALLED_APPS 
list:

```
INSTALLED_APPS = [
    …
   ‘myuser’,
]
```

We also need to tell Django that we’re using a custom user model, defining 
AUTH_USER_MODEL in our myapp/myapp/settings.py.

```
AUTH_USER_MODEL = 'myuser.MyUser'
```

We will now delete db.sqlite3, create new Django migrations from our new 
model, and finally use them to create a new database:

> rm db.sqlite3

> python manage.py makemigrations myuser

> python manage.py migrate

We will try creating a user once the authentication mechanism is in place.
Let us now  install the djangorestframework-simplejwt package and add it to 
the requirements.txt file.

> pip install djangorestframework-simplejwt

We will also need to add this to `Installed Apps` section of our `settings.py` 
file.

```
'rest_framework_simplejwt',
```

We will also need to change the authentication settings for the Django Rest
Framework to reflect the use of Simple JWT. Add the following code to the
`settings.py` file:
```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
``` 

Finally, we need to add the following code to `settings.py` file to set the
expiration time of JWT:

```
import os
import datetime

JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(minutes=15),
}
```

---
---
---

Manage dependencies
> pip freeze

To deactivate virtualenv
> deactivate

Migrate tables
> python manage.py migrate

Create superuser
> python manage.py createsuperuser

Run server locally
> python manage.py runserver

You can access home page here
> 127.0.0.1:8000

Access Admin panel here
> 127.0.0.1:8000/admin

To get all available commands:
> python manage.py --help

References:
> https://www.tag1consulting.com/blog/building-api-django-20-part-i
> https://lewiskori.com/blog/user-registration-and-authorization-on-a-django-api-with-djoser-and-json-web-tokens/
