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
request.project_list # a list of all projects, contains two pages(list); item in page is dict, keys are short_name, icon_uri, logo_uri, uri, full_name_en, full_name_zh
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
Now we require every backend developer to set the django evironment with these project-special settings in `settings.py` file. Shown below.
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

# all the projects must use the compressor to minimize the static resource.
COMPRESS_ENABLED = False # change to True if under the online version.
COMPRESS_OUTPUT_DIR = 'dc_auth' # this is project-special cache directory. e.g. dc_1kite
# setup the main sub-path of the uri, e.g. 1kite/
DC_BASE_URL = 'auth/'

# setup the code name of the project, e.g. 1kite
DC_PROJECT_CODE_NAME = 'auth'

# the i18n language file path, e.g. /dc_i18n/dc_1kite/locale
LOCALE_PATHS = ('/dc_i18n/dc_common/locale',)

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
# some common settings are all defined in these four settings, 
# look into the files to see the common variales, 
# re-define the variables below these four imports to override the variables if the project has special needs.
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

## normal coding
all the coding:

### javascript
codes must be extracted from html and save as file, coding in html is not allowed. excluding special situation.

all the js file must start with:
```
/*
 * create date: 2016-12-09
 * author: somebody
 * last update: somebody at 2016-12-09
 */
```
if the js file is combined from several onter files, keep these information of each files.

### css
using `@import('/the/url/of/resource')` syntax is not allowed.

all the url in css coding, except using django template tag, must use absolute path, e.g. `background: url(/absolute/path/to/resource);`.

### python
all the functions, classes, etc., must add some requeired information. e.g. :
```
def somefunction():
  // describe what is this doing for
  // create date: 2016-12-09
  // author: somebody
  // last update: somebody at 2016-12-09
```

# AuthSystem usage
## frontend
visit the object `{{ request.account }}` for account information, this object has attributes: `id`, `pk`, `username`, `is_active`, `is_authenticated` and `type_name`.
## backend
1. include the middlewares `session.middlewares.DCSessionMiddleware` and `account.middlewares.DCAuthenticationMiddleware` in the `MIDDLEWARE_CLASSES` of file `settings.py`, as shown above. `session.middlewares.DCSessionMiddleware` must be loaded before `account.middlewares.DCAuthenticationMiddleware`.
2. import anything from `account` and `session` at the bottom of file `settings.py`, `from account.settings import *; from session.settings import *;`, as shown above.
3. import the decorator in your views file, `from account.decorators import login_required`, and add the decorator `@login_required` before your view function.
4. in the view function, visit the object `request.account` for account information, `request.account` has attributes: `id`, `pk`, `username`, `is_active`, `is_authenticated` and `type_name`.

# Project list
## usage
for both frontend and backend, visit list `request.project_list` for all projects' information. this list contains two lists, both structures are the same:
```
[
  {
    'short_name': 'shortname', 
    'icon_uri': '/path/to/standard/icon', 
    'logo_uri': '/path/to/standard/logo', 
    'uri': '/path/to/official/home/', 
    'full_name_en': 'the english full name', 
    'full_name_zh': 'the chinese full name',
  },
  ...
]
```

## manage
visit the site '/auth/admin/dc_project/app/' to manage the project info. please note, this is necessary to add the project information to use the public UI frame, the AuthSystem and the notification system.

# notification system
This system can only use in the LAN.
visit the API '/auth/notify/set_new_notification/' in the LAN using `POST` method to add new notification. post data is:
```
{
  'account': '', // account pk, required.
  'account_type': '', // account type name, required.
  'title_en': '', // title in english
  'title_zh': '', // title in chinese. note, if one of the titles is blank, the blank one will be set the same to the other one. if both are blank, request refused.
  'content_en': '', // content in english
  'content_zh': '', // content in chinese. note, if one of the contents is blank, the blank one will be set the same to the other one. if both are blank, request refused.
}
```
