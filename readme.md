# MadFW

Documentation (in progress) for my pet-project - fast and easy PHP MTV framework.
Most ideas (structure, ORM) from Django, but realised in PHP and enhanced.

<sup>Updated: 2024-02-24</sup>


- [Starting](#starting)
    - [Migrations](#migrations)
- [Structure](#structure)
    - [Setting file](#setting-file)
    - [Config file](#config-file)
    - [Admin file](#admin-file)
    - [Route file](#route-file)
    - [Module structure](#module-structure)
- [Routes](#routes)
- [View](#view)
- [Templates](#templates)
    - [Syntax](#syntax)
    - [Template static methods](#template-static-methods)
    - [Template functions](#template-functions)
- [ORM](#orm)
    - [Classes](#classes)
    - [DB methods](#db-methods)
    - [Model methods](#model-methods)
    - [TreeModel methods](#treemodel-methods)
    - [PageModel methods](#pagemodel-methods)
    - [Query methods](#query-methods)
        - [Conditions syntax](#conditions-syntax)
    - [Examples](#examples)
        - [Models](#models)
        - [Queries](#queries)

- [Forms](#forms)
- [Formsets](#formsets)
- [Form fields](#form-fields)
    - [CharField](#charfield)
    - [TextField](#textfield)
    - [PasswordField](#passwordfield)
    - [Password2Field](#password2field)
    - [PhoneField](#phonefield)
    - [EmailField](#emailfield)
    - [DateField](#datefield)
    - [DateTimeField](#datetimefield)
    - [TimeField](#timefield)
    - [HiddenField](#hiddenfield)
    - [ColorField](#colorfield)
    - [NumberField](#numberfield)
    - [CheckField](#checkfield)
    - [SwitchField](#switchfield)
    - [FileField](#filefield)
    - [ImageField](#imagefield)
    - [ChoicesField](#choicesfield)
    - [SuggestField](#suggestfield)
    - [FunctionField](#functionfield)
    - [MadBlockField](#madblockfield)
    - [MadTextField](#madtextfield)
    - [ManyToManyField](#manytomanyfield)
- [Form exapmles](#form-examples)

- [Functions](#functions) 
    - [Common functions](#common-functions)
    - [Response functions](#response-functions)

- [Admin Models](#admin-models)


# Starting 

Clone base project an setup DB  in config.php.

**'connection' - PDO::__construct $dns string**

Example #1

    const DB_CONFIG = [
        'connection' => 'mysql:host=localhost;dbname=test;charset=utf8',
        'username' => 'root', 
        'password' => 'root',
        'options' => [
            PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"
        ]
    ];

Example #2

    const DB_CONFIG = [
        'connection' => 'sqlite:file.db',
    ];

[Start project DB and update/create first user](#migrations).


# Migrations

Before migrations set DB settings in config.php

    php _apps/_mad/manage.php mm    - make migrations
    php _apps/_mad/manage.php dm    - migrate
    php _apps/_mad/manage.php user LOGIN PASSWORD NAME - Add / update user. NAME if not set equals LOGIN

    php _apps/_mad/manage.php um    - undo last migrate
    php _apps/_mad/manage.php cc    - clear cache
    

# Structure

    _apps (vendor folder, can be renamed)
        _mad (MadFW core)
        _mad_admin (MadAdmin app)
        ...
    _site (project folder, set up in public_html/index.php (_SITE))
        module1
            js
            css
            templates (module templates folder)
                template.html
                ...
            index.php (module views)
            models.php (module models)
            admin.php (admin file for MadAdmin)
            routes.php (module routes)
        module2
            ...
        ...
        settigs.php (common project settings)
        config.php (private project setting)
        routes.php (main routes)
        admin.php (main admin file for MadAdmin)
    _migrations (system folder for store migrations)
    public_html (web site folder)
        js
        css
        index.php
        .htaccess
    
    media (media, files folder, can be moved/renamed, setting up in settings.php)
    _cache (system folder for store template and routes cache)


## Setting file

File with project settings

    /** site settings */
    const SITE_DIR = "/"; 
    const MEDIA = "../media";
    const THUMBS = "thumbs";
    const CACHE = "../_cache";
    const MAX_FILE_SIZE = 30000000;

    /** mad admin settings */
    const ADMIN = "admin";
    const ADMIN_PATH = "../_apps/_mad-admin";

    /** add private settings */
    include ('configs.php');

    /** site modules in which it will be collected static and models migrations */
    const MODULES = [ADMIN_PATH, 'main'];

    const MIDDLEWARE = [ 
        "madStaticCollector"
    ];

    /** language settings */
    const LANGUAGE = "ru";
    setlocale(LC_ALL, 'ru_RU.UTF-8');
    date_default_timezone_set('Europe/Moscow');

    /** Site config vars
    * 
    *  SITE_CONFIG - array of:
    *  "Section name" => [
    *      "var name" => [
    *          "title" => "var title",
    *          "type" => "FormField type",
    *          ... FormField properties like 'default', 'placeholder' ...
    *          ]
    *  ],
    *  ...
    */
    const SITE_CONFIG = [
        "Fieldset" => [
            "key" => ["title" => "Param name", "type" => "type of FormField", "default" => "defalt value"], 
            ...
        ],
        ...
    ];


## Config file

**File with project data for instance. Not in GIT**

    const DEBUG = true;
    const DEBUG_PANEL = true;
    const TEMPLATE_COMPRESS = true;
    const HTTPS = false;

    const DB_CONFIG = [
        'connection' => 'mysql:host=localhost:3306;dbname=mysqldb;charset=utf8',
        // or 'connection' => 'sqlite:path/phpsqlite.db',
        'username' => 'root', 
        'password' => 'root' 
    ];


## Admin file

Main admin file 

    /** Admin list table settings */
    const ADMIN_PER_PAGE = 15;
    const ADMIN_PER_PAGE_ARRAY = [15, 30, 150];

    /** Admin widgets (madblock, madtext) settings */
    const MAD_EDITOR_COLORS = "#ffffff,#da0000,#008000";
    const MAD_BLOCK_PATH = 'main';

    /** Admin panel section config
    * 
    *  ADMIN_SECTIONS - array of:
    *  "page_slug" => [
    *      "title",
    *        
    *       "page_slug" => [
    *          'title' =>   'Title of section',  // required
    *          'module' =>  'module',            // module path in _SITE
    *          'model' =>    Model::class        // MadAdmin model                       
    *      ]
    *  ]
    */
    const ADMIN_SECTIONS = [
        "section" =>
        [
            "Section title",
            "module" => ['title' => "Module title", "module" => "module path", "model" => MadAdminModule::class],
            ...
        ],
        ...
    ];


## Route file

**Example of main route file**
Route file in module has the same structure

    /**  name => path, module - for includer routes from module
    *   name => path, module, view, args
    *    
    *   patterns in path:   
    *   <name:str> - string 
    *   <name:int> - number
    *   * - anything more, only last character
    */
    return [
        'admin' => [ADMIN, ADMIN_PATH],

        'index' => ["/", "main", IndexView::class],
        'media' => ["media/*", "main", mediaGet::class],
        'pages' => ["<page_alias:str>", "main", PageView::class],

        /* custom 404 page */
        'page404' => ['page404', 'main', page404::class]
    ];


## Module structure

Module folders
- **js** - for module JavaScript files, file which name start with # combined in public_html/js/js.min.js
- **css** - for module CSS files, file which name start with # combined in public_html/css/styles.min.css
- **templates** - templates folder, can be named as you wish
    - **template.html** - template files in [template format](#templates) or php native template format
      ...

Module files (recommended names)
- **index.php** - [Views models](#view) (Views from routes call from this file only)
- **models.php** - [models](#models)
- **admin.php** - [admin file for MadAdmin](#admin-models)
- **routes.php** - [routes](#routes)
- **forms.php** - [forms](#forms)

Modules can have submodules. Routes inherited.


# Routes
File: routes.php

**name => [path, module_folder]** - for include routes from module / submodule
**name => [path, module_folder, view, args]** - for call view from module with args (not required)
     
**patterns in path:**
- <name\:str> - string 
- <name\:int> - number
- any more, can be only last character

Example:

    return [
        'route_name' => ['path', 'module_folder', 'ModuleView'],
        'route_name' => ['<var:int>', 'module_folder', 'ModuleView'],
        'route_name' => ['path', 'module_folder'],
        'route_name' => ['<var1:str>/page/<var2:str>/*', 'module_folder', 'ModuleView'],
        'route_name' => ['f_<var1:str>/s_<var2:str>abs/*', 'module_folder', 'ModuleView'],
        'route_name' => ['*', 'module_folder', 'ModuleView'],
    ]; 

# View 
File with classes: index.php

Output rendered html from template with variables from context.
To get variables from route make it static with the same name.
E.g. 
**route:**
'file' => ['<file_id:int>/*', 'module1', 'Module1View'],
**variable in class:**
public static \$file_id - file_id value
public static \$tail    - asterics value

    $template - template path in _SITE
	$headers - additional headers for response

    View::init($request) - common actions, calling before request method
    View::post($request) - actions on POST
    View::get($request) - actions on GET
    View::context() - make context 


**Example code** 

    <?php
    import("main/models");
    
    use Main\Pages;


    class PageView extends View
    {
        public static $page_id;
        public $template = "main/templates/page.html";


        public function init($request)
        {
            $this->page = Pages::find(self::$page_id);
            if (!$this->page) {
                throw new PageNotFound();
            }
        }

        public function get($request) 
        {
        }

        public function context()
        {
            $context = parent::context();
            $context['page'] = $this->page;
            return $context;
        }
    }


# Templates

## Syntax

| Syntax | PHP Template |
| ------ | ----------- |
| {% yield blockname %} | place for block |
| {% block blockname %} | start block |
| {% endblock %} | end block |
| {% extends path %} | path to extend block |
| {% include path %} | include file |
| {{ \$var }} | \<?=\$var ?> |
| {% code %} | \<?php code ?> |
| {{{ \$var }}} | \<?= htmlentities(\$var, ENT_QUOTES, \'UTF-8\') ?> |
| {{! \$var }} | \<?= trim_all_breaks \$var ?> |
| {# comments #} |  |

**all paths are relative to the directory "_SITE"**


## Template static methods

    Template::render(string $template, $data = []) - return rendered html


## Template functions

    csrf()
    url($name, $params = [])
    to_tel($phone)
    paragraph($string)
    html_trim($str)
    thumbnail($source_file, $width, $height, $params = [])
    month($n, $case = false)
    weekday($date)
    rd($date)
    rdw($date)
    rdm($date)
    rdt($date, $mask = "d.m.Y H:i")


**thumbnail**
Make thumbnail file in THUMB folder. 
Return thumbnail filename.
Path in given file saves in thumbnail folder.
e.x. thumbnail('images\image.png', 100, 0) return '\THUMB\images\image_100.png'

**thumbnail \$params**
- crop = true/false
- ext = png/jpg/webp - extention of thumbnail file
- quality =  0-100 - quality of thumbnail file
- left
- top
- upscale = true/false
- bg = #rrggbb - set default thumbnail background
- watermark = ['image' => 'filepath in _SITE/filename.png(PNG24 only)', 'position' => 'top/center/bottom/top-left/top-right/bottom-left/bottom-right']

**month(\$n, $case = false)**
Return russian month name. N - month number. Case = true ('январь'), case = false (января)

**weekday(\$date)**
Return russian weekday name of date

**rd(\$date)**
Return russian date format 'dd.mm.YYYY'

**rdw(\$date)**
Return russian date format 'dd месяц YYYY'
e.x. rdw('2022-02-22') => '22 февраля 2022'

**rdm(\$date)**
Return russian date format 'dd месяц'

**rdt(\$date, \$mask = "d.m.Y H:i")**
Return russian datetime format 'dd.mm.YYYY HH:MM'
\$mask - custom date format


# ORM

## Classes

    DB
    Query
    Model
    TreeModel
    PageModel


## DB methods
    
    DB::run($query, $args = [])
    DB::runFetch($query, $args = [])
    DB::runFetchAll($query, $args = [])

*\$query - PDO query with args, \$args - values for $query*

## Model methods

    ModelClass::find($id)
    ModelClass::save($columns = null) - update or insert (all columns or columns in args)
    ModelClass::insert($args)
    ModelClass::insertOrUpdate($args)
    ModelClass::insertOrIgnore($args)
    ModelClass::update($conditions, $args) ==  ModelClass::filter($conditions)->update($args)
    ModelClass::delete($conditions) == ModelClass:filter($conditions)->delete()


## TreeModel methods

    ModelClass::getTree($elements = null, $parentId = 0, $node = '', $level = 0)
    ModelClass::buildTree($elements = null, $parentId = 0, $node = '', $level = 0, $alias = '')


## PageModel methods

    PageModelObject->prepareSEO()


## Query methods

| Syntax | Return |
| ------ | ------ |
| ModelClass::objects() | Query class |
| ModelClass::objects()->all() | Model objects array |
| ModelClass::objects()->first() | Model object |

**Short form:**
ModelClass::all()

**Available for:**
active, first, all, last, filter, exclude, pair, sum, max, min, limit, values, count, array


| Syntax | Description | return |
| ------ | ----------- | ------ |
| all() | return all records | array of Model objects |
| get() | return first record | Model object |
| first() | return first record | Model object |
| last() | return last record | Model object |
|  |  |  |
| fields(string \$fields) | default = *, set fields to select, e.g. 'id, title' or 'SUM(rating) as s, AVG(rating) as a' | Query |
| filter(array \$conditions, \$or = false) | set select conditions with operator AND or OR. See [conditions syntax](#conditions-syntax) below.  | Query |
| and(array \$conditions, \$or = false) | add conditions with AND (conditions) | Query |
| or(array \$conditions, \$or = false) | add conditions with OR (conditions) | Query |
| exclude(array \$conditions, \$or = false) | as filter but with NOT | Query |
| active() | short form of filter(['active' => true]) | Query |
| rawFilter(\$where, array \$args) | add to select raw expression, e.g. 'AND id \& 1' | Query |
|  |  |  |
| orderBy(string \$order, \$add = false) | set new select order or add to existing order | Query |
| orderBySet(\$column, \$set) | set order by set array (ORDER BY FIND_IN_SET) | Query |
| groupBy(string \$fields) | set GROUP BY | Query | 
| having(array \$conditions, \$or = false) | set HAVING with conditions | Query |
| limit(int \$l1, int \$l2 = null) |  | Query |
|  |  |  |
| delete() | delete selected rows | PDO result |   
| update(array \$args) | update from args selected rows | PDO result |   
|  |  |  |
| values(string \$column) | return array of column values | array |
| pair(array \$columns) | return array of key => value from pair: [key column, value column] | array |
| groups(\$column) | return two level array grouped by \$column (PDO::FETCH_GROUP) | array |
| distinct(\$column) | return distinct array of \$column values | array |
| count() | return rows count  | int |
| sum(string \$column) | SUM(column) | float |
| avg(string \$column) | AVG(column) | float |
| max(string \$column) | MAX(column) | float |
| min(string \$column) | MIN(column) | float |
|  |  |  |
| join(string \$model, string \$field_1, string \$field_2, \$type = "INNER") | join table to select | Query |
| withTable(string \$model) | change table to next filter fields | Query |
|  |  |  |
| select_related(string \$names) | add related fields to select, e.g. 'images, label__images' of 'action__label__image__title' | Query |
| filter_related(string \$name, array \$filters, \$or=false) | add filter to select related fields | Query |
   


### Conditions syntax

| Syntax | SQL |
| ------ | --- |
| 'field' => 'value' | field = value |
| 'field__not' => 'value' | field <> value |
| 'field__isnull' => true/false | field IS NULL / field IS NOT NULL |
| 'field__gt' => 'value' | field > value |
| 'field__gte' => 'value' | field >= value |
| 'field__lt' => 'value' | field < value |
| 'field__lte' => 'value' | field <= value |
| 'field__like' => 'value' | field LIKE value |
| 'field__notlike => 'value' | field NOT LIKE value |
| 'field__in' => array | field IN (array[0], ...) |
| 'field__notin' => array | field NOT IN (array[0], ...) |
| 'field__between' => array | field BETWEEN (array[0], array[1]) |

***Value 'NOW()' is equal NOW() in SQL query***

**Field can be related, e.g. 'product__price__gte' or 'product__action__label__image__title__like'**


## Examples

### Models

    class Pages extends Model {
        protected static $table = 'pages';
        protected static $order = 'title';

        protected static $fields = [
            "title" => ['type' => 'varchar', 'length' => 255, 'null' => true, 'comment' => 'Title'],
            "last_modified" =>   ['type' => 'datetime', 'null' => false, 'default' => 'CURRENT_TIMESTAMP', 'comment' => '']

            'blocks' =>  ['type' => 'children', 'model' => '\\PageBlocks', 'field' => 'page']
        ]
    }

    class PageBlocks extends Model {
        protected static $table = 'pages_blocks';

        protected static $fields = [
            "page" =>       ['type' => 'foreign_key', 'model' => '\\Pages', 'ondelete' => 'SET NULL', 'null' => true, 'comment' => 'Page'],
            "title" => ['type' => 'varchar', 'length' => 255, 'null' => true, 'comment' => 'Title']
        ]
    }

#### Model properties

protected static **\$table** - table name in SQL database
protected static **\$order** - default ORDER for SQL queries


#### Fields params

| Name | Values | Description |
| ------ | --- | --- |
| type | varchar, int, image, file, foreign_key, many_to_many, children, one_to_one, boolean, text, longtext, date, datetime | SQL field type |
| null | true/false | set default NULL on NOT NULL |
| default | value, 'CURRENT_TIMESTAMP' (for datetime default) | for not null field |
| comment | text | field description |
| model | name of model class | for foreign_key, many_to_many, children, one_to_one types |
| filter | simple filters for linked model | for foreign_key, many_to_many, children types |
| field | name of parent field in model | for many_to_many, children types |
| ondelete | CASCADE, SET NULL ... | for foreign_key only, SQL ONDELETE action |
| table | name of many to many table | for many_to_many only |
| length | number | lenght of varchar field |
| upload_dir | folder name or class method name retuning folder name | for image, file fields |
| key | INDEX, UNIQUE | type of index key fo fields if needed |



### Queries

Select page that is modified later then 2022-01-01   
    
    $page = Pages::filter(['last_modified__gte' => '2022-01-01'])->first()

Get all page blocks

    $page->blocks

Get Query for page blocks, and query example

    $page->blocks()
    $page->blocks()->orderBy('title DESC')->all()

Select all pages and fill blocks with retated data

    $pages = Pages::objects()->select_related('blocks')->all()

Select pages where related block title contains 'text'

    $pages = Pages::filter(['blocks__title__like' => '%text%'])->all()



# Forms

Form example in forms.php

    class LoginForm extends Form
    {
        public $model = Accounts::class;

        public $fields = [
            'login' =>    ['type' => 'CharField', 'max_length' => 100, 'min_length' => 5, 'required' => true, 'placeholder' => 'name@mail.ru', 'label' => 'Login'],
            'password' => ['type' => 'PasswordField', 'max_length' => 20, 'min_length' => 1, 'required' => true, 'placeholder' => '••••••••', 'label' => 'Password']
        ];
    }

Form variables 
- **\$fields** - has formfeild descriptions
- **\$model** - set up if form has model
- **\$cleaned_data** - cleaned data after validation
- **\$errors** - form errors

Form methods
- **addField($name, $field, $model = null)** - dynamically add field to form 
- **addError($error)** - dynamically add error to form 
- **is_valid()** - validate form
- **save($model = null)** - save form (with model or custom model)
- **save_m2m()** - save ManyToMany form fields
- **errors()** - output form errors (for template use)


Form view example 
    
    <?php
    import("accounts/forms, accounts/models");

    use \Accounts\Accounts;
    use \Accounts\AccountsTokens;


    class AccountsLogin extends View
    {
        public $template = "accounts/templates/login.html";
        public $form;
        public $account;

        public function get($request)
        {
            $this->form = new LoginForm();
        }

        public function post($request)
        {
            $this->form = new LoginForm($request->data);

            if ($this->form->is_valid()) {
                $this->account = Accounts::login($this->form->cleaned_data['login'], $this->form->cleaned_data['password']);
                
                if ($this->account) {
                    ....
                } else {
                    $this->form->addError("Неправильный логин и/или пароль");
                }
            }
        }

        public function context()
        {
            $context = parent::context();
            $context['form'] = $this->form;
            return $context;
        }
    }


Form template example

    <form method="post" action="{{ url('accounts:login') }}">
        {{ $form->errors() }}
        {{ csrf() }}
        
        {{ $form->login }}
        {{ $form->password }}
        <button type="submit">Войти</button>
    </form>


# Formsets

# Form fields

## CharField

**Example**
All params, 'type' required only

    "name" =>    [
        'type' => 'CharField', 
        'label' => 'Name',     // if set model - set default label from model 'comment'
        'placeholder' => 'Name', 
        'min_length' => 2, 
        'max_length' => 50,    // if set model - set default max_length from model 'length'
        'default' => '',       // if set model - set default from model 'default'
        'required' => true,    // if set model - set default required from model 'null'
        'autocomplete' => false,
        'disabled' => false,
        'readonly' => false,
        'autofocus' => true,
        'style' => 'color:#000',
        'help' => 'short help string'  // for mad admin only, shown after label
    ],


## TextField

## PasswordField

## Password2Field

## PhoneField

## EmailField

## DateField

## DateTimeField

## TimeField

## HiddenField

## ColorField

## NumberField

## CheckField

## SwitchField

## FileField

## ImageField

## ChoicesField
    
The list is taken from:
1. param 'choices'
2. param 'model', may use param 'filter'
3. if form model and form model choices are set - model choices
4. if form model and form related model are set - related model rows, may use param 'filter'


**Example**
All params, 'type' required only

    "name" =>    [
        'type' => 'ChoicesField', 
        'choices' => [],       // list of choices
        'label' => 'Name',     // if set form model - set default label from model 'comment'
        'default' => '',       // if set form model - set default from model 'default'
        'required' => true,    // if set form model - set default required from model 'null'
        'disabled' => false,
        'readonly' => false,
        'autofocus' => true,
        'style' => 'color:#000',
        'dataset' => ['fieldName', ], // if form model is set, fieldName additing to option as data-fieldName = 'option->fieldName'
        'help' => 'short help string',  // for mad admin only, shown after label
       
        'model' => Model::class,  // for data from custom model 
        'filter' => ['active' => true] // for filter data from 'model' or related model from form model
    ],

## SuggestField

## FunctionField

## MadBlockField

## MadTextField

## ManyToManyField


# Form examples


# Functions

## Common functions

| Syntax | Description | return |
| ------ | ----------- | ------ |
| import(string \$files) | files is string contains relative paths to files separated by commas without extention, e.g. 'module1/models, modeule2/index' |  - |
| is_captcha() | check POST data contain captcha data and if it is valid | boolean |
| reverse(\$name, \$params = []) | get url path from routers | string |
| plural(int \$n, array \$forms) | get string in plural form \$from forms by \$n | string |
| switch_to_ru($value) |  | string |
| mark(string \$string, string or array \$query) | highlight string with tag \<mark\> from \$query | string |
| get_limit(int \$page_number, int \$items_count, int \$per_page) | return array for LIMIT  | [\$start, \$per_page] |
| paginate(\$page, \$items_count, \$per_page, \$n = 3) | return pageblocks as array, e.g. [1,0,4,5,6,0,100], where 0 = separator | array |
| values_list(\$rows, \$key, \$flat = true) | return array or string separated by commas of keys from rows | array or string |
| key_array(\$rows, \$column) | make keys for rows from rows column | array |

## Response functions

    response($data, $headers = [])
    json_response($data)
    response_file($file, $filename = false)
    return_file($file, $filename = false)
    redirect($url, $code = 302)
    forbidden()


# Admin Models

Example, all param is not required.

    class MadAdminAccount extends MadAdmin
    {
        public $model = \Accounts\Accounts::class;

        public $list_display = ['login', 'name', 'active', 'last_login']; 
        public $list_display_styles = ['login' => 'flex:1'];
        public $list_display_classes = ['name' => 'cssClassName'];
        public $list_filters = ['active', 'last_login'];
        public $list_order = 'login';
        public $list_editable = ['active'];

        public $search_fields = ['login', 'name'];
        public $search_fields_help = 'Search by login or name';

        public $readonly = ['last_login'];
        public $editonly = false;
        public $collapsed = ['info'];

        public $actions = [
            'action' => ['title' => 'Action title', 'icon' => '&#xf058', 'function' => '\\MadAdminAccount::actionMethod']
        ];

        public $fieldset = [
            'Fieldset title' => ['login', 'name', 'active'],
            'Fieldset title' => ['created', 'last_login', 'statistic'],
            'Fieldset title' => ['orders'],
        ];

        // for rewrite default widget or make function field
        public $widgets = [
            'created' =>   ['type' => 'DateField'],
            'statistic' => ['type' => 'FunctionField', 'function' => 'MadAdminAccount::statistic']
        ];

        public $inlines = [
            'orders' => [
                'model' => \Shop\Orders::class,
                'parent' => 'account',
    
                'readonly' => [],
                'extra' => true,
                'fieldset' => [
                    'title',
                    'created',
                    'total',
                ],

                'widgets' => []
            ];
        ];

        public static statistic($item) {
            ...
            return $string;
        }
    }

Default widgets fo model

    'varchar' =>  ['type' => 'CharField'],
    'text' =>     ['type' => 'TextField'],
    'int' =>      ['type' => 'NumberField'],
    'float' =>    ['type' => 'NumberField'],
    'boolean' =>  ['type' => 'SwitchField'],
    'longtext' => ['type' => 'TextField'],
    'date' =>     ['type' => 'DateField'],
    'datetime' => ['type' => 'DateTimeField'],
    'choices'  => ['type' => 'ChoicesField'],
    'image'  =>   ['type' => 'ImageField'],
    'file'  =>    ['type' => 'FileField'],
    'foreign_key' => ['type' => 'ChoicesField'],
    'many_to_many'  => ['type' => 'ManyToManyField'],
    'children' => ['type' => 'ChildrenField']

Widget FunctionField

    'name' => ['type' => 'FunctionField', 'function' => 'functionName', 'label' => 'Label', 'options' => []]

    Function arguments $item
    Options need only for filter. Also for filter needs function with name 'functionName_filter' (arguments $query, $value)
