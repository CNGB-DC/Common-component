# django coding
## template settings
Because all projects will be required to use the same UI template, it does not need to code the related settings in the template files.
If anyone has some special requirements for the AuthSystem or other components, contact Doo.
- generally, all the template must start with `{% extends 'base.html' %}` to load the public UI frame.
- if there is a cdn resource existing, never ever use the local one, otherwise, the project won't be allowed to be online.
- the template `base.html` has three cdn js blocks `{% block cdnjs_head %}{% endblock %}`, `{% block cdnjs_middle %}{% endblock %}`, `{% block cdnjs_end %}{% endblock %}` and two local js blocks `{% block js_head %}{% endblock %}`, `{% block js_end %}{% endblock %}`, for preventing js code confilt.
- the template `base.html` has three cdn css blocks `{% block cdncss_head %}{% endblock %}`, `{% block cdncss_middle %}{% endblock %}` and `{% block cdncss_end %}{% endblock %}` and two local css blocks `{% block css_head %}{% endblock %}`, `{% block css_end %}{% endblock %}`, for preventing overriding.
- in the tempalte, visit the `request` variable for some common data. shown below.
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
```
- in the js, visit the `window.lang` for language info, `zh-cn` for Chinese while `other` for all the other language. visit `window.alertFrame` for `set_alert(code)` to set new alert, for `alert_exists(code)` to check if the alert is already shown. all the translation text is stored in `window.translaion_list`. the `code` in methods from `window.alertFrame` is from the keys of `window.translation_list.alert`. 
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
- for ajax, all the response, if it's json format, the data structure will be the same to below. some ajax load the static resource data, this situation does not need to apply this json format.
```
// has error, code is 2, check the error for information if necessary
{
  "code": 2,
  "error": "error info"
}
// has warning, code is 1, check the warning for information if necessary, check data for data
{
  "code": 1,
  "warning": ["warning info", ...],
  "data": ...
}
// normal, code is 0, check data for data directly
{
  "code": 0,
  "data": ...
}
```
## backend settings
Now we require every backend developer to set the django evironment with a common `settings.py` file. Shown below.
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
- generally, all the view function, which return responce loading tempalte file as page, must add the `request` object with key `request` to context.
- generally, all the responces with json format must follow the structure below.
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
