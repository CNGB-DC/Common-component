# django coding 
## template settings
Because all projects will be required to use the same UI template, it does not need to code the related settings in the template files.
If anyone has some special requirements for the AuthSystem or other components, contact duwensi@genomics.cn.
## backend settings
Now we request every backend developer to set the django evironment with a common `settings.py` file. Shows below.
```
# include /dc_utils as a package path.
import sys
sys.path.append('/dc_utils')
# setup the MIDDLEWARE_CLASSES and the order of the middle wares is shown below.
MIDDLEWARE_CLASSES = (
  'django.contrib.sessions.middleware.SessionMiddleware', 
  'session.middlewares.DCSessionMiddleware', # connect to the Memcached server and load the session data.
  'common.middlewares.DCCommonMiddleware', # add some useful data to request object, see related section below.
  'account.middlewares.DCAuthenticationMiddleware', # solve the session data and create a virtual account object, see related section below.
  'django.middleware.common.CommonMiddleware',
  'django.middleware.locale.LocaleMiddleware',
  'django.middleware.csrf.CsrfViewMiddleware',
  'django.contrib.auth.middleware.AuthenticationMiddleware',
  'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
  'django.contrib.messages.middleware.MessageMiddleware',
  'django.middleware.clickjacking.XFrameOptionsMiddleware',
  'django.middleware.security.SecurityMiddleware',
)

TEMPLATES = [
  {
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    '''
    the list of DIRS must contain two path: one is /dc_templates/dc_public/, the 
    other one is the project-special template directory, e.g. /dc_templates/dc_1kite/.
    '''
    'DIRS': ['/dc_templates/dc_common/', '/dc_templates/dc_public/',], 
    'APP_DIRS': True, # set this to True if you need the built-in admin site of django, otherwise, whatever.
    'OPTIONS': {
      'context_processors': [
        'django.template.context_processors.debug',
        'django.core.context_processors.i18n', # add this if the website needs the i18n function.
        'django.template.context_processors.request',
        'django.contrib.auth.context_processors.auth',
        'django.contrib.messages.context_processors.messages',
      ],
    },
  },
]
```
