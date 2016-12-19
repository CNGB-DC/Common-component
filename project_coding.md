# project coding
## template part
Because all projects will be required to use the same UI template, it does not need to code the related settings in the template files.

If anyone has some special requirements for the AuthSystem or other components, contact Doo.

### base template
generally, all the template must start with `{% extends 'base.html' %}` to load the public UI frame.

### block
#### project navbar
the block is `{% block project-nav %}{% endblock %}`.

this block will load the `{{ request.main_nav_template }}` template and visit the variable `{{ request.global_search }}`.

generally, do not override this block.

#### alert frame
the block is `{% block alert-frame %}{% endblock %}`.

generally, do not override this block.

#### project content
the block is `{% block project-content %}{% endblock %}`.

#### project foot
the block is `{% block project-foot %}{% endblock %}`.

#### static resource
- the template `base.html` has three cdn js blocks `{% block cdnjs_head %}{% endblock %}`, `{% block cdnjs_middle %}{% endblock %}`, `{% block cdnjs_end %}{% endblock %}` and two local js blocks `{% block js_head %}{% endblock %}`, `{% block js_end %}{% endblock %}`, for preventing js code confilt.
- the template `base.html` has three cdn css blocks `{% block cdncss_head %}{% endblock %}`, `{% block cdncss_middle %}{% endblock %}` and `{% block cdncss_end %}{% endblock %}` and two local css blocks `{% block css_head %}{% endblock %}`, `{% block css_end %}{% endblock %}`, for preventing or prividing css overriding.
- if there is a cdn resource existing, never ever use the local one, otherwise, the project won't be allowed to be online.

### template variable
in the tempalte, visit the `request` variable for common data from middlewares:
```
request.account.id # None or str
request.account.pk # None or str
request.account.username # blank str or str
request.account.is_active # False or True
request.account.type_name # None or local or github or windows or linkedin or google-plus or qq or weibo or *-linked(* is tpa type)
request.account.is_authenticated # False or True
request.project_list # a list of all projects, item is dict, keys are short_name, icon_uri, logo_uri, uri, full_name_en, full_name_zh
request.meta # a dict of current app, keys are short_name, icon_uri, logo_uri, uri, description_zh, description_en, full_name_zh, full_name_en, author, keywords
request.current_app # structure and content are the same to request.meta
request.login_url # str
request.policy_uri # str
request.terms_uri # str
request.tech_email # str
request.read_notify_uri # str
request.new_notify_uri # str
request.cngb # dict, keys are icon_uri, logo_uri, uri, full_name_zh, full_name_en
request.dc # structure is the same to request.cngb
request.main_nav_template # str, main navbar template path, only available when settings.MAIN_NAVBAR_FRAME exists
request.global_search.url # str, global search url, only available when settings.GLOBAL_SEARCH exists
request.global_search.field # str, global search field, only available when settings.GLOBAL_SEARCH exists
request.global_search.options # list, values for global search field. items are dict, keys are name and value. only available when settings.GLOBAL_SEARCH exists
request.global_search.query # str, global search query, only available when settings.GLOBAL_SEARCH exists
```

### javascript variable
visit the `window.lang` for language info, `zh-cn` for Chinese while `other` for all the other languages. 

all the translation texts are stored in `window.translaion_list`.  
```
window.translation_list = {
  alert: {
    auth_status_failed: {
      external_class: 'alert-danger',
      'zh-cn': '查询登陆状态失败，请刷新重试。',
      'default': 'Resquest for authentication status failed, please refresh the page to retry.'
    },
    ...
  },
  text: {
    read_status: {
      'zh-cn': '已读',
      'default': 'read'
    },
    ...
  }
};
```

visit `window.alertFrame.set_alert(code)` to set new alert and `window.alertFrame.alert_exists(code)` to check if the alert is already shown. note that the `code` is key from `window.translation_list.alert`.
```
window.alertFrame.set_alert('auth_status_failed');
```

### ajax format
all the response, if it's json format, the data structure will be the same to below. some ajax load the static resource data, this situation does not need to apply this json format.
```
// has error: code is 2, visit error key for information if necessary
{
  "code": 2,
  "error": "error info"
}
// has warning: code is 1, visit warning key for information if necessary, visit data for data
{
  "code": 1,
  "warning": ["warning info", ...],
  "data": ...
}
// normal: code is 0, visit data for data directly
{
  "code": 0,
  "data": ...
}
```

## backend part
### common settings
Now we require every backend developer to set the django evironment with these settings in `settings.py` file. Shown below.
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
COMPRESS_YUI_BINARY = 'java -jar /dc_assets/program/yuicompressor-2.4.8.jar'
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

# make sure the file main_navbar.html is in /dc_templates/dc_[project code name]/dc_public/
MAIN_NAVBAR_FRAME = 'dc_public/main_navbar.html'

# global search bar settings
GLOBAL_SEARCH = {
  'url': '', # required and cannot be blank
  'field': 'field', # no required, default is 'field'
  'options': [
    {'name': 'Full text', 'value': 'full_text',},
    {'name': 'Symbol', 'value': 'symbol',},
    ...
  ], # options for 'field'
  'query': 'query', # no required, default is 'query'
 }

# import the settings of the packages under the path /dc_utils.
from account.settings import *
from common.settings import *
from mail.settings import *
from session.settings import *
```

### format for json responce
generally, all the responces with json format must follow the structure below.
```
// has error, cannot response normally, code is 2, add error info in error key, no other keys 
{
  "code": 2,
  "error": "error info"
}
// has warning, but can return the result with expect, code is 1, add warning info in warning key, data in data key, no other keys 
{
  "code": 1,
  "warning": ["warning info", ...],
  "data": ...
}
// normal, code is 0, add data in data key, no other keys
{
  "code": 0,
  "data": ...
}
```
