# pyrocms_develop_doc
## Developing Field Types
PyroCMS 開發者文件翻譯
PyroStreams 是設計來專門被繼承的，使得它的功用是確保 PyroStreams 核心之外可以有更多的東西可用。
繼承 PyroStream 主要的用途是用來建立一個新的 field types。
Field types 讓你可以管理資料格式和 input、output 以及每一筆 storage。
他們可以在複雜的幾行程式範圍內儘量讓你的工作完成。

一個簡單的例子；E-Mail Field type, 它允許讓我們指向一個些定的重點。
* 我們應該用哪種 type of column 建立資料庫來保存數據 ( 在這種情況下, Varchar )
* 什麼資料格式下我們需要去輸入資料 ( 一個簡單的 text field )
* 任何額外的資料驗證 ( 在這種情況下，我們需要去確認輸入的是一個有效的 Email Address )

這是一個很簡單的例子
PyroStreams 已經為您插入到各種現有結構來建立複雜或簡單的 field types 和繼承 PyroStreams 的功能。

### Structure
### Field Type Basics

如果你熟悉 PHP ，那麼建立一個 Field type 來使用 streams 是簡單的。
每個 Field type 是一個包含一個 PHP 檔案的資料夾，裡面有一個 clss 包含了一個 Class 裡面有你的 Field type 資訊以及像是 form 輸出東西的 methods。
這裡概述了 Field type file 的基本結構。

* Basics
* Field Parameters
* Languages in Field Types
* Validation
* Working With File Uploads
* CSS/JS Files
* Using View Files
* Field AJAX Functions

#### Basics
*slug 別名*
一開始，要建立一個資料夾及一個你指定的 slug 的檔案。
在這個例子猚，我們將使用 Email field type，它的 slug 被取名為__email__。
建立一個字首為__field__的檔案，接 slug 名稱，接 .php 副檔名。
```
email/field.email.php
```
你可以把檔案放在 __addons/shared_addons/fields_type__ 裡或__addons/[site-slug]/fild_types__，然後 streams 將會識別它並自動使用它。
每個 Field type 都有資料的基本結構而且需要它在那裡正確的運作。
這裡有一個非常基本的 Field type 的例子:

```php
class Field_email 
{
 public $field_type_slug = 'email';

 public $db_col_type = 'varchar';

 /**
  * Output form input
  *
  * @access public
  * @param array
  * @return string
  */
 public function form_output($data)
 {
    $options['name']   = $data['form_slug'];
    $options['id']   = $data['form_slug'];
    $options['value']  = $data['value'];

    return form_input($options);
 }
}
```
如同你所看到的，我們有一個 class name 叫 **Field_yourslug**。在裡面我們有一些基本的類別參數:

|Variable|Description|
|:------:|:-----|
|field_type_slug|The field slug (the same one you are using for the class name and folder/file names).|
|db_col_type|Type of MySQL column that PyroStreams should create to store the data for this field. Any type can be used, but the most common are varchar, text, and int.|

類別變數的一部份，有一個 method 是必要的-__form_output__。
當建立一個 form 表單和允許你去加入邏輯或客製化的輸入在你的欄位的時候這個 method 就會被呼叫。
在這種情況下，我們只要返回一個基本的 input。
你可以在下面的 METHOD 的部份找到更多這個 method 的資料。

當創建立 inputs 時 PyroStreams 使用 [CodeIgniter's Form Helper](http://codeigniter.com/user_guide/helpers/form_helper.html)，而且它在 third party field types 是可以使用的。

#### Tapping Into CodeIgniter
如果你想使用 CodeIgniter Super Object 在你的 Field types，你可以透過 CI Class 參數來存取。
這是自動加到你的 Field type，因此你不需要去使用 get_instance() 語法。

```php
$this->CI->load->library('Typography');
```

#### Optional Class Properties 可選的 Class 屬性
你還可以添加一些其他 Class 的參數來更改 PyroStreams 如何與 field type 的相互影響。

|Variable|Value|Description|
|:------:|:-----:|:-----|
|extra_validation|string|Validation for required, unique, etc. is added automatically with PyroStreams. However, you might need to add some extra validation from time to time. You can do that by adding a class variable named extra_validation.|
|input_is_file|bool|Some field types work with files (such as the image field type). When creating a field type that uses a file, make sure to add a class variable called input_is_file and set it to TRUE.|
|alt_process|bool|Sometimes you don't want to actually have a column created for your field type (such as in the case of having a related rows table). Setting this to true will tell PyroStreams to ignore the column in the stream when it needs to because there is none.|
|version|number|The current version number of the field type.|
|author|array|An array with two keys: name (name of the author) and url URL to the author's website.|

#### Field Parameters
每個 field 可以客製或是預製 field 參數。
例如，很多 fields 就類似 text field 且有 max_length 的 Field 參數的優勢。
然後，你可以讓它們為你自己能夠給你的 Users 增加功能。

客製化 fields (預設的和你所做的) 是放到一個 class 變數到你的 field type class 命名的 custom_parameters。
```php
public $custom_parameters = array('max_length', 'my_custom_one'); 
```

#### Preset Field Parameters 預設參數

|Parameter|Description|
|:------:|:-----|
|max_length|Collects data on the max length of a field.|
|default_value	Collects data on the default value of a field. This is only a simple text input, so if you need a special default input, you should create your own.|

#### Creating Custom Field Parameters
如果你使用自定參數在你的 Field type，你需要把你的 Fields 的名稱放在 language 檔案裡。
可以簡單的使用參數的 slugs 就像 language slugs 一樣。
它們將會被自動啟用。
在你的 Field type 裡:
```php
public $custom_parameters = array('choice_data', 'choice_type');
```
在你的 Language File 裡:
```php
$lang['streams:choice.choice_data'] = 'Choice Data';
$lang['streams:choice.choice_type'] = 'Choice Type';
```

#### Validation 驗證
驗證使用了一些 PyroCMS field types 形式。
* Assignment-added validation 指配新增驗證
  - 這是必須並且獨特，並且由 field 決定分配的模式。
* Standard field type validation 標準 field type 驗證
  - 這在實際的 field type 是標準的驗證規則(見下文)。
* Custom field type validation 客製化 field type 驗證
  - 這是 valiation() 函式裡的客製驗證功能(見下文)。

有了這三種方式去整合驗證，你幾乎可以擁有所以讓你去驗證的所有工具。

##### Standard Field Type Validation
標準 field type 驗證是新增在一個名為 __extra_validation__ 的類別變數
```php
public $extra_validation = 'numeric|integer'; 
```
所有 CodeIgniter 的 Form Validation 規則都可以在這裡使用，並且用 __|__ 字元區分。

##### Custom Field Type Validation 客製化 Field Type 驗證
有時候，標準驗證規則不應付你的輸入的內容，你需要一些客製化的驗證的邏輯。
增加一些客製化的驗證邏輯只要在一個你的 field type 裡公開的驗證函式新增就可以了。
裡面一共有三個參數:

|Parameter|Description|
|:------:|:-----|
|value|The value submitted to the form for your field.|
|mode|Either 'new' or 'edit', depending on if the form is editing an entry or creating a new one.|
|field|The field instance object.|

如果資料驗證失敗，你可以回傳一段錯誤文字。
回傳 Null 值或 true 值來表示你的認證已被通過測試。
以下有一個簡單的例子:

```php
public function validate($value, $mode, $field)
{
    if ($value != 'the value we want')
    {
        return 'The '.$field->field_name.' field needs to be the value we want!';
    }

    return true;
}
```

請記住，你仍然可以存取所有的 __$_POST__ 變數，所以如果你需要抓取 row ID，你可以從 phone data 中使用 __row_edit_id__ 。

```php
$this->CI->input->post('row_edit_id');
```

#### Working With File Uploads 檔案上傳工作
某些 Field type 是檔案相關 ( 例如圖片檔案模式 )。
當建立一個 Field Type 是檔案相關的，請務必增加一個類別變數；命名為 __input_is_file__ 並且設定它為 __true__ 。
```php
public $input_is_file = true;
```
這將會確保必填欄位一切工作正常。因為它需要去檢查 __$_FILE__ 變數，而不是 __$_POST__ 。

#### CSS/JS Files
很多時候你需要在你的 Field type 使用另外的東西。
這可能是一個 CSS 檔案或是一個 View。
PyroStreams 設定允許你可以去拉取這些檔案，無需去猜測你的 Field type 在檔案系統的哪裡。

如果你想要增加 CSS 或 JavaScript 在 PyroCMS 的後端，
你可以把它們輸入一個 CSS 或 JS 資料夾中的 field type 裡，並且透過函式 event() 來新增它們

```php
public function event()
{
    $this->CI->type->add_css('email', 'example.css'));
    $this->CI->type->add_js('email', 'example.js'));
}
```
上面的程式碼將會把 example.css 和 example.js 檔案新增到 Email Field type 的管理區域裡。

#### Using View Files 使用 View 檔案
如果你想從你的 Field Type 載入一個 view 檔案，建立一個 views 資料夾在你的 field type 資料夾，然後把你的 view 檔案放進去。
你可以皹這樣呼叫你的 view 檔案:
```php
$this->CI->type->load_view('field_type_slug', 'view_file', $data, true);
```
第一個參數，它應該是 field type slug，接下來三個參數就跟 CodeIgniter 裡的 __$this->load->view()__ 函式一樣。

#### Field AJAX Functions
如果你需要讓你的 Field Type 可以存取 AJAX 函式，你可以建立一個前綴字為 __ajax___ 的函式在你的 Field Type 裡。
```php
public function ajax_myfunction()
{
    // AJAX functionality here.
}
```
然後你可以透過這個 URL 存取這個函式
```
http://example.com/streams_core/public_ajax/field/[field_type_slug]/myfunction
```
請注意，這些函式是公開存取的，所以如果你需要去自己檢查資料。(例如: 檢查已登入的 user )
原因是，這些 AJAX 函式是被持續公開的，我們無法預測 Field type 在哪裡被使用。
它可能在公用的或是私人的函式。


### Methods
