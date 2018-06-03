# Advisory

## PW Login Vulnerability

Despite the UI telling you that pw logins were disabled, [they were not](https://github.com/ENOFLAG/faustctf2018_jodlgang/blob/master/jodlgang/jodlgang/settings.py#L119):

```python
AUTHENTICATION_BACKENDS = (
'jodlplatform.backends.FaceAuthenticationBackend', 'django.contrib.auth.backends.ModelBackend')
```

Every login attempt that failed in FaceAuthenticationBackend was also handled by default django backend. The [form](https://github.com/ENOFLAG/faustctf2018_jodlgang/blob/master/jodlgang/jodlplatform/forms.py) parsed the pw parameter and passed it to authenticate:


```python
def clean(self):
    username = self.cleaned_data.get('username')
    password = self.data.get('password')

[...]
    if username is not None:
        self.user_cache = authenticate(self.request, username=username, password=password, face_img=face_img)
[...]
```

The users' passwords were hashed with [md5 without any salt](https://github.com/ENOFLAG/faustctf2018_jodlgang/blob/master/jodlgang/jodlplatform/migrations/0002_auto_20180507_0235.py#L23), so bruteforcing all of their 8-char password (0-9, A-Z) was done in a few minutes with hashcat. Every team's ambassador (the user which submits flags) was visible in the about page (and their IDs mapped to the team IDs), so you could login into their accounts and fetch the flags.

You can fix this by disabling the pw login completely, but we restricted it to our test account only.


## "Similar" Images Attack

An image was considered to be ok if the probability was [above 0.5](https://github.com/ENOFLAG/faustctf2018_jodlgang/blob/master/jodlgang/jodlplatform/backends.py#L43), but the flagbot's images were similar enough so that we could increased the threshhold to 0.999. This eliminated a lot of image attacks, but unfortunately not all.
