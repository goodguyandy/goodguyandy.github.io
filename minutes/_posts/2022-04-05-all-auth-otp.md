---
title:  "Force One-Time-Password usage to Django allauth"
---


Some time ago I was tasked for a quick-beta to force the usage of One-Time-Password inside the django-all-auth framework when generated from the awesome "django-cookiecutter". 

I found out an hacky solution, using a middleware:


in _requirements.txt_:

```
django-allauth-2fa
django_otp
```
> remember to put the package version to not have problems in the future!


_in your settings file_:

```
USE_2FA = True
```


_middleware.py_:

> warning: spaghetti code! clean this code before use :) 


```python

from django.shortcuts import redirect
from allauth_2fa.views import TwoFactorSetup, TwoFactorBackupTokens, TwoFactorAuthenticate

from django_otp.plugins.otp_totp.models import TOTPDevice

from config.settings.base import USE_2FA


class CheckOtpMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def process_view(self, request, view_func, *view_args, **view_kwargs):

        if USE_2FA:
            print(view_func.__name__)
            if request.user.is_authenticated:
                totp = TOTPDevice.objects.filter(user=request.user).first()
                if not totp:
                    if view_func.__name__ != "TwoFactorSetup" and view_func.__name__ != "LogoutView" :
                         return redirect("two-factor-setup")
                else:
                    if not totp.confirmed:
                        if view_func.__name__ != "TwoFactorSetup"   and view_func.__name__ != "LogoutView":
                            return redirect("two-factor-setup")



    def __call__(self, request):
        response = self.get



```

Add your middleware and you are ready to go!

