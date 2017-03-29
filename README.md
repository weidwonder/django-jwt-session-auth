# django-jwt-session-auth
Simple auth-session implement by integrating jwt with django. Decouple django builtin user. U are free now!

# Version
> 0.1.1


# Attention

Now, this version only support over django-1.10, cause django has updated its middleware structure.
Lower version will get supported in the future.

# Quickstart

Add django-jwt-session-auth middleware to `settings.py`

```python
MIDDLEWARE = [
    ...,
    'django_jwt_session_auth.JwtAuthMiddleware',
]
```

and then customize your own `PAYLOAD_TO_USER` and `USER_TO_PAYLOAD` just like this:

```python
def user_to_payload(user):
    exp = datetime.datetime.now() + datetime.timedelta(seconds=3600 * 7)
    return {
        'user_id': str(user.id),
        'exp': exp
    }


def payload_to_user(payload):
    if not payload:
        return None
    user_id = payload.get('user_id')
    user = YourUserModel.objects.get(user_id)
    return user
```

then add `JWT_AUTH` settings to `settings.py`

```python
# if your functions in auth.py under user app, then settings may like:
JWT_AUTH = {
    'PAYLOAD_TO_USER': 'user.auth.payload_to_user',
    'USER_TO_PAYLOAD': 'user.auth.user_to_payload',
}
```

login user by this

```python
from django_jwt_session_auth import jwt_login
token = jwt_login(your_user, request, expire=your_expire_time_in_sec) # also you can set default expire time in settings
# then return this token to client. After that every time client requests, set HTTP header Authorization="JWT <token>"
```

finally you will get the request jwt user by

```python
class SomeView(View):
    def get(self, request, *args, **kwargs):
        self.user = request.jwt_user  # whatever type of user you want you can customize by PAYLOAD_TO_USER and USER_TO_PAYLOAD
        self.session = request.jwt_session  # dict like object
        data = do_sth()
        return Response(data)
```


# DEFAULT SETTINGS

```python
DEFAULT_JWT_AUTH_SETTING = {
    'PREFIX': 'JWT',
    'ALGORITHM': 'HS256',
    'DECODER': 'django_jwt_session_auth.jwt_decoder',
    'ENCODER': 'django_jwt_session_auth.jwt_encoder',
    'SECRET': '',  # default is django.conf.settings.SECRET_KEY
    'PAYLOAD_TO_USER': None,
    'USER_TO_PAYLOAD': None,
    'USER_KEY': 'pk',
    'TEST_USER_GETTER': None,
    'SESSION': {
        'EXPIRE': 2592000,
        'PREFIX': 'JWT_AUTH_CACHE:'
    }
}
```

//TODO: add instructions of settings.
