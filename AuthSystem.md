# django coding
## template settings
Because all projects will be required to use the same UI template, it does not need to code the related settings in the template files.
If anyone has some special requirements for the AuthSystem or other components, contact Doo.
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

# setup the TEMPLATES.
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

# static resource of all the projects, csrf cookie name, language cookie name must be set to be the same, shown below.
STATIC_URL = '/dc_assets/'
STATIC_ROOT = '/dc_assets/'
CSRF_COOKIE_NAME = 'dc_csrftoken'
LANGUAGE_COOKIE_NAME = 'dc_lang'

# all the projects must use the compressor to minimize the static resource.
COMPRESS_ENABLED = False # change to True if under the online version.
COMPRESS_ROOT = '/dc_assets/dc_cache/'
COMPRESS_OUTPUT_DIR = 'dc_auth' # this is project-special cache directory. e.g. dc_1kite
COMPRESS_OFFLINE = True
COMPRESS_YUI_BINARY = 'java -jar /dc_assets/yui-compressor/yuicompressor-2.4.8.jar'
COMPRESS_CSS_FILTERS = [
  'compressor.filters.css_default.CssAbsoluteFilter',
  'compressor.filters.datauri.CssDataUriFilter',
  'compressor.filters.yui.YUICSSFilter',
]
COMPRESS_YUI_CSS_ARGUMENTS = ''
COMPRESS_YUI_JS_ARGUMENTS = ''
COMPRESS_JS_FILTERS = ['compressor.filters.yui.YUIJSFilter',]
COMPRESS_DATA_URI_MAX_SIZE = 204800

# setup the main sub-path of the uri, e.g. 1kite/
DC_BASE_URL = 'auth/'

# setup the code name of the project, e.g. 1kite
DC_PROJECT_CODE_NAME = 'auth'

DC_REDIRECT_FIELD_NAME = 'next'

# setup the technical support email of the project.
DC_TECH_SUPPORT_EMAIL = 'weixiaofeng@genomics.cn'

# the i18n language file path, e.g. /dc_i18n/dc_1kite/locale
LOCALE_PATHS = ('/dc_i18n/dc_common/locale',)

# the language settings
# the default value of LANGUAGE_CODE is en-us, but this is not in the default LANGUAGES any more, change it to en.
LANGUAGE_CODE = 'en'
# set the available languages for the project, generally, no need to change it.
LANGUAGES = (
    ('en', 'English'),
    ('zh-cn', 'Chinese'),
)

# import the settings of the packages under the path /dc_utils.
from account.settings import *
from common.settings import *
from mail.settings import *
from session.settings import *
```
