# django-mfa2
A Django app that handles MFA, it supports OTOP, U2F, FIDO2 U2F, and Trusted Devices


For FIDO2, both security keys and android-safetynet are supported.

Trusted device is a mode for the user to add a device that doesn't support security keys like iOS and andriod without fingerprints or NFC.

`*Note*: U2F and FIDO2 can only be served under secure context (https)`

Depends on

* pyotp
* python-u2flib-server
* ua-parser
* user-agents
* python-jose
* fido2==0.5


# Installation
1. `pip install django-mfa2`
1. in your settings.py add the application to your installed apps
   ```python
   INSTALLED_APPS=(
   '......',
   'mfa',
   '......')
   ```
1. Add the following settings to your file

   ```python 
   MFA_UNALLOWED_METHODS=()   # Methods that shouldn't be allowed for the user
   MFA_LOGIN_CALLBACK=""      # A function that should be called by username to login the user in session
   MFA_RECHECK=True           # Allow random rechecking of the user
   MFA_RECHECK_MIN=10         # Minimum interval in seconds
   MFA_RECHECK_MAX=30         # Maximum in seconds
   MFA_QUICKLOGIN=True        # Allow quick login for returning users by provide only their 2FA 

   TOKEN_ISSUER_NAME="MDL"      #TOTP Issuer name

   U2F_APPID="https://localhost"    #URL For U2F
   FIDO_SERVER_ID=u"localhost"      # Server rp id for FIDO2
   FIDO_SERVER_NAME=u"MDL"
   FIDO_LOGIN_URL=BASE_URL
   ```
1. Break your login function

   Usually your login function will check for username and password, log the user in if the username and password are correct and create the user session, to support mfa, this has to change
   
      * authenticate the user
      * if username and password are correct , check the user has mfa or not
          * if user has mfa then redirect to mfa page
          * if user doesn't have mfa then call your function to create the user session

   ```python
    def login(request): # this function handles the login form POST
       user = auth.authenticate(username=username, password=password)  
       if user is not None: # if the user object exist
            from mfa.helpers import has_mfa
            res =  has_mfa(username = username,request=request) # has_mfa returns false or HttpResponseRedirect
            if res:
                return res
            return log_user_in(request,username=user.username) 
            #log_user_in is a function that handles creatung user session, it should be in the setting file as MFA_CALLBACK
     ```
1. Add mfa to urls.py
   ```python 
   import mfa
   import mfa.TrustedDevice
   urls_patterns= [
   '...',
   url(r'^mfa/', include(mfa.urls)),
   url(r'devices/add$', mfa.TrustedDevice.add,name="mfa_add_new_trusted_device"), # This short link to add new trusted device
   '....',
    ]
    ```
1. Provide `mfa_auth_base.html` in your templaes with block called 'head' and 'content'
    The template will be included during the user login.
1. To match the look and feel of your project, MFA includes base.html but it need a block named content to added its content in it.
1. Somewhere in your app, add a link to 'mfa_home'
```<l><a href="{% url 'mfa_home' %}">Security</a> </l>```