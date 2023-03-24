# MadFW

It is my pet-project - PHP MTV framework.
Most ideas (structure, ORM) from Django, but realised in PHP and enhanced.


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
    const ADMIN_PER_PAGE = 15;

    /** mad admin widgets settings */
    const MAD_EDITOR_COLORS = "#ffffff,#da0000,#008000";
    const MAD_BLOCK_PATH = 'main';

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

    /** site config vars defaults */
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

    /** ***********************************************************************************
    * ADMINPAGES - array of:
    *  "page_slug" => [
    *      "title",
    *        
    *       "page_slug" => [
    *          'title' =>   'Title of section',  // required
    *          'module' =>  'module',            // module path in _SITE
    *          'model' =>    Model::class        // MadAdmin model                       
    *      ]
    *  ]
    *************************************************************************************/

    return [
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
    ]; 

# View 
File with classes: index.php

Output rendered html from template with variables from context.
To get variables from route make it static with the same name.
E.g. 
**route:**
'file' => ['<file_id:int>', 'module1', 'Module1View'],
**variable in class:**
public static $file_id

    $template - template path in _SITE
	$headers - additional headers for response

    View::init($request) - common actions
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

    Template::render(string \$template, \$data = []) - return rendered html


## Template functions

    csrf()
    url($name, $params = [])
    to_tel($phone)
    paragraph($string)
    thumbnail($source_file, $width, $height, $params = [])
    can_edit($name)
    html_trim($str)
    month($n)


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


## Model methods

    ModelClass::find($id)
    ModelClass::save($args)
    ModelClass::insert($args, $orUpdate = false) == ModelClass::objects()->insert($args, $orUpdate = false)
    ModelClass::update($conditions, $args) ==  ModelClass::filter($conditions)->update($args)
    ModelClass::delete($conditions) == filter($conditions)->delete()


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
| groupBy(string \$fields) | set GROUP BY | Query | 
| having(array \$conditions, \$or = false) | set HAVING with conditions | Query |
| limit(int \$l1, int \$l2 = null) |  | Query |
|  |  |  |
| insert(array \$args = [], \$orUpdate = false) | insert (or update) | int LastInsertedId |   
| delete() | delete selected rows | PDO result |   
| update(array \$args) | update from args selected rows | PDO result |   
|  |  |  |
| values(string \$column) | return array of column values | array |
| pair(string \$pair) | return array of key => value from pair: 'key, value' | array |
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

## TextField

## PasswordField

## Password2Field

## PhoneField

## EmailField

## DateField

## DateTimeField

## HiddenField

## ColorField

## NumberField

## CheckField

## SwitchField

## FileField

## ImageField

## ChoicesField

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