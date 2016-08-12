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

#### Preset Field Parameters

|Parameter|Description|
|:------:|:-----|
|max_length|Collects data on the max length of a field.|
|default_value	Collects data on the default value of a field. This is only a simple text input, so if you need a special default input, you should create your own.|

#### Creating Custom Field Parameters


### Methods
