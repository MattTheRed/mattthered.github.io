---
layout: post
title: Creating a custom user model in Django 1.5 that plays nicely with Django admin
redirect_from:
  - /post/58783289485/creating-a-custom-user-model-in-django-15-that
---
Extending Django’s built in user model used to be a tricky affair in Django 1.4 and below. The classic approach was to store additional information in a separate UserProfile model, but you were still stuck with Django’s built in fields. Don’t need the first_name field? Can’t do that. Want to use email address as your username? Also, not possible. Want to enforce unique=True on a value defined otherwise in the Django User model? You’ll need to resort to some hackery using signals.

With Django 1.5, things have become easier and you can now either extend the existing Django User model, or create your own. The [documentation on how to customize your User Model](http://t.umblr.com/redirect?z=https%3A%2F%2Fdocs.djangoproject.com%2Fen%2Fdev%2Ftopics%2Fauth%2Fcustomizing&t=ZTExOGQ2MDgzMTVkYzY0YjNkNzk4MGMyZTkyOTNhYTZiNTJhYTRmMyxnVEN5MUxxWQ%3D%3D&b=t%3AV3QsY1F6pCug4-HUnFvSyw&p=http%3A%2F%2Fwww.mattthered.com%2Fpost%2F58783289485%2Fcreating-a-custom-user-model-in-django-15-that&m=1) is pretty self explanatory, so I’ll focus on the part that tripped me up: getting your new model working with Django Admin.

Here we have a custom model, extending the AbstractBaseUser class:

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
 
class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    website_url = models.URLField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)
    is_admin = models.BooleanField(default=False)
 
    USERNAME_FIELD = "email"
    REQUIRED_FIELDS = ["website_url"]
 
    objects = CustomUserManager()
 
    def get_full_name(self):
        return self.email
 
    def get_short_name(self):
        return self.email
```
To ensure this plays nicely with Django Admin, you’ll need to make sure you have the following fields (or properties) defined on your model. I’m opting to store these as Booleans, but you could have these defined by custom logic.

* is_active
* is_staff
* is_superuser

Additionally a number of methods must also be defined:

* get_full_name
* get_short_name
* has_perm
* has_module_perms

The last two are being taken care of in this case by the Permissions mixin in this case.

You will also need a custom manager for your model with a create_user & create_superuser method. Here’s an example:

```python
class CustomUserManager(CustomUserManager):
    def create_user(self, email,
                    password=None):
        user = self.model(email=email, password=password)
        user.save(using=self._db)
        return user
 
    def create_superuser(self, email,
                         password):
        user = self.create_user(email,
                                password=password)
        user.is_staff = True
        user.is_admin = True
        user.is_superuser = True
        user.save(using=self._db)
        return user
```
Finally, you will need to add your model to admin.py:

```python
from django.contrib.auth import get_user_model
User = get_user_model()
 
admin.site.register(User)
```
Keep in mind you still could access your User model directly with your import, but its better to use Django’s built in helper method to fetch the User model for you.