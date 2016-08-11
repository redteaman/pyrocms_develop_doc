# pyrocms_develop_doc

## Add-on Development

- Developing Plugins
	+ Methods
- Developing Widgets
- Developing Field Types
	+ Structure
	+ Methods
- Developing Modules
	+ Basic Structure
	+ Sample Module

### Developing Plugins 開發 Plug-in

PyroCMS 其中之一的核心概念為 tags。如果你已經熟悉了 PyroCMS，那麼你已經確實地看過它們了。
這裡有一個返回當前 URL 的簡單的 tag 的範例：
```{{ url:current }}```
這裡程式背後的 tags 被稱為 plugin。Plugins 是特別的 PHP 檔案，它的功能被透過 PyroCMS tags 被呼叫，
並且可以像收集 tag 元素的方式來做事情。它們可以簡單的撰寫並結合了複雜的功能，整潔有序的整合進 PyroCMS 佈局。

#### Modular vs Standalone Plugins 模組化 vs 自動安裝 Plugins
僅管它們在結構上是相同的，一個 plugin 可以是一個獨立的檔案，
也可以是一個較大的 Module 裡的 plugin.php 檔案。
Module 內的 Plugin 可以輔助你進行一些事情在你建立的客製化的 Module上，
如同一個獨立的 plugin 像是 Google Maps Plugin 或 在程式上加入列出你的 Tweeter 列表。
一個獨立的 plugin 並不需要一個更大的 Module 結構，它可以自行獨立運作。

#### Example Plugin 範例
這個 Session Plugin 可以在 system/cms/plugins/session.php 找到

```php
<?php defined('BASEPATH') or exit('No direct script access allowed');

/**
 * Session Plugin
 * Read and write session data
 *
 * @package     PyroCMS
 * @author      PyroCMS Dev Team
 * @copyright   Copyright (c) 2008 - 2012, PyroCMS
 */
 
class Plugin_Session extends Plugin
{
    /**
     * Data
     * Loads a piece of session data
     * Usage:
     * {{ session:data name="foo" }}
     *
     * @param   array
     * @return  array
     */
    function data()
    {
        $name = $this->attribute('name');
        $value = $this->attribute('value');

        // If we have a value, let's set it.
        if ($value)
        {
            $this->session->set_userdata($name, $value);
            return;
         }

        // Otherwise, we need to retrieve the value.
        return $this->session->userdata($name);
    }

    /**
     * Flash
     *
     * Loads a piece of flashdata
     * Usage:
     * {{ session:flash name="foo" }}
     * @param   array
     * @return  array
     */
    function flash()
    {
        $name = $this->attribute('name');
        $value = $this->attribute('value');

        // If we have a value, let's set it.
        if ($value)
        {
            $this->session->set_flashdata($name, $value);
            return;
         }

        // Otherwise, we need to retrieve the value.
        return $this->session->flashdata($name);
    }
}

/* End of file theme.php */
```

在上述的程式中，請注意一些重要的事項：
* Class name 為 Plugin_ 開頭 ( 開頭第一個字母為大寫 )
* Plugin 必須繼承 Plugin class
* 每個 tag 功能可以直接對應一個 class 功能
* 每個 function 都有 return，而不是 echo

#### Getting Plugin Tag Attributes 取得 Plugin Tag 屬性
讓 tags 真的強大是它們可以設定屬性，屬性讓你基於輸入資料來自由的編輯 tag 輸出。
例如：

```
{{ session:data name="foo" }}
```

在上面的程式中我們可以存取這樣的名稱參數。

```
$this->attribute('name');
```

如果沒有設定屬性，你可以指定一個預設值

```
$this->attribute('name', 'a default value');
```

如果沒有指定值， $this-> attribute 將會使用這個預設值

#### Tag Pairs tag 配對
Tags 並不總是只有一行 return 簡單的一個字串。
Tags 也可以是成對的，這意味著它們之間有一個 opening 和 closing tag 和其內容。
Plugin 有某些特性用來建立它們能夠簡單的控制的 tag pairs

```
{{ blog:posts limit="5" order-by="title" }}
	<h2>{{ title }}</h2>
		<p>Written by: <a href="/users/profile/{{ author_id }}">{{ author_name }}</a></p>
{{ /blog:posts }}
```

Top tag 使用參數，並且裡面有一個 bottom closing tag。
Tag 內有我們需要依照屬種設定的資料來替換的參數。
在這樣的情況下，我們可以從我們的 plugin function 得到一組關聯的 array 資料，然後參數將會被我們所送出的資料所替代。
同樣的情形，我們可以從這個 plugin 得到 Blog 條目的 array 給 tag ( 在我們這個案例下的每一筆 blog po文 )，
然後在你已經定義好的 opeing 和closing tags 參數之間替換參數。
我們可以從一個例子得知：

```
	return array(
  	array(
    	'title'         => 'First Blog Post',
      	'author_id'     => 1,
	      'author_name'   => 'Phil Sturgeon',
  	),
    array(
		    'title'         => 'Second Blog Post',
		    'author_id'     => 2,
		    'author_name'   => 'Jerel Unruh',
		)
	);
```

送出的參數是有關聯的 arrays 並且可以在 tags 形成迴圈

```
return array(
    array(
        'title'         => 'First Blog Post',
        'author_id'     => 1,
        'author_name'   => 'Phil Sturgeon',
        'categories'    => array(
                    array('category_name' => 'PyroCMS'),
                    array('category_name' => 'Phil Sturgeon')
                )
    ),
    array(
        'title'         => 'Second Blog Post',
        'author_id'     => 2,
        'author_name'   => 'Jerel Unruh',
        'categories'    => array(
                    array('category_name' => 'PyroCMS'),
                    array('category_name' => 'Jerel Unruh')
                )
    )
);
```

你可以使用額外的分類 array 像我們這樣添加：

```
{{ blog:posts limit="5" order-by="title" }}
    <h2>{{ title }}</h2>
    <p>Written by: <a href="/users/profile/{{ author_id }}">{{ author_name }}</a></p>
    {{ categories }}
        {{ category_name }}
    {{ /categories}}
{{ /blog:posts }}
```

#### Tag Pair Raw Content Tag Pair 原始內容
你可以透過你的 plugin file 呼叫 plugin 裡的 tags 來取得及使用完整的內容。

```
$this->content();
```

如果你想分析你的 tags 和 Lex parser 之間，tag pair content 可以在透過 parser instance 解析前進行修改。

```php
$parser = new Lex_Parser();
$parser->scope_glue(':');
return $parser->parse($this->content(), $data = array(), array($this->parser, 'parser_callback'));
```


### Plugin Methods ###

當你建立了一個 plugin，你必須增加一些有用的 methods 到 plugin class 裡。
你可以使用這些常見的東西，像是取得 tags 的 attributes 或 content、解析你的屬性容許呈現在動態內容、載入主題或 Module Views上…等等。

#### Core Methods
這裡所有的都是 Plugins 中所使用的基本 methods。因為你的 plugins 會繼承這個 class，所以要小心不要覆蓋到任何這裡的 functions。

##### content()
這個函式將回傳 tag pair 間的內容。
這只有在使用 tag pair 時以及使用單一 tag 回傳 empty 時被使用。

###### Example
```
{{ navigation:links }}

{{ title }}

{{ /navigation:links }}
```

#### attribute($param, $default = null)
取得 tag 上的一個屬性。
如果屬性不存在，你也可以選擇傳遞一個預設值。

|Name|Default|Required|Description|
|---|:---:|:---:|:---|
|param|  |Yes|The attribute name|
|default|null|No|The default value to return if the attribute is not set|

##### Example
```php
// plugin method
public function current_date() {
  $format = $this->attribute('format', 'M j Y');
  $date = $this->attribute('date', time());
  
  return date($format, $date);
}

// template code
{{ plugin_name:current_date format="D, M j, Y" }}

// output
Tue, Jan 1, 2013
```

#### attributes()
它將會執行 attribute() 函式所有可用的屬性。
回傳所有相關的陣列。

```php
// template code
{{ plugin_name:person id="4" first_name="Jack" last_name="Donaghy" }}

// plugin code
public function person() {
  $person = $this->attributes();
  
  // and more
}

// value of $person
array(
  'id'         => '4',
  'first_name' => 'Jack',
  'last_name'  => 'Donaghy'
);
```

#### set_attribute($param, $value)
它允許你設定指定的值到一個參數上。
你可以在你的 plugin method 裡呼叫它，並且任何使用者傳遞時都會被覆蓋過去。

|Name|Default|Required|Description|
|---|:---:|:---:|:---|
|param|  |Yes|Attribute name|
|value|  |No|Value to set|

#### parse_parameter($value, $data = array())
這個功能允許你分析你的動態參數值，像是 [[segment_2]] 或 [[username]]。
然而，如果你希望和 $data 陣列分析你自己的值。所有的值都會被 LEX 解析器解析，但是是使用[[ ]]來作為分隔符號。
|Name|Default|Required|Description|
|---|:---:|:---:|:---|
|value|  |Yes|The String to parse through|
|data|array()|No|Additional variables to include in parsing|

##### Turning on / off Parameter Parsing
你可以開啟或關閉在任何 tag 透過 parse_params attribute 呼叫參數分析。
如果 parse_params 設定為 yes，我們將會自動分析。
如果設置為別的，就不會。
在預設的狀況下，如果參數不存在，我們當作你想去分析變數裡的參數，所以我們會設定它為 yes

```
// turn off parameter parsing
{{ plugin_name:show_script text="Hello, my name is [[ your name ]]." parse_params="no" }}

// outputs
Hello, my name is [[ your name ]].
```

或者使用參數分析到你的有用的地方

```
// using parameter parsing (default)
{{ plugin_name:show_script text="Hello, my name is [[ username ]]." }}

// outputs
Hello, my name is PhillySturgeon.
```

##### Pre-populated Values
這些變數已經設定好並且可以讓你使用。不過如果你願意，你可以覆蓋掉這些。

```
// uri_segments
segment_1
segment_2
segment_3
segment_4
segment_5
segment_6
segment_7

// user info (if logged in)
user_id
username
```

###### Example
增加你的資料來進行分析

```php
// plugin code
public function ask_question() {
  $question = $this->attribute('text', '[[ color ]], really?');
  $color = $this->attribute('color', 'red');

  $replace = array(
    'color' => $color
  );

  return $this->parse_parameter($question, $replace);
}


// template code
{{ plugin_name:ask_question text="Is [[ color ]] your favorite color?" color="red" }}


// output
Is red your favorite color?
```

#### theme_view($view, $vars = array(), $parse_output = true)
你可以使用這個函式去載入一個 theme view 到你的 plugin 來輸出。
如果你需要動態的載入一個 view file，這是方便的。
這個函式永遠回傳的是 view content。
沒有被自動的寫被頁面中。

|Name|Default|Required|Description|
|---|:---:|:---:|:---|
|vars|array()|No|Pass data to the view for parsing|
|parse_output|true|No|Parse output with LEX Parser?|

##### Example
載入一個 upsell box 並且把它傳遞到動態數據中。

```php
// plugin method
public function upsell() {
  $name = $this->attribute('name', 'default');
  $data = array(
    'title'   => 'Others Also Bought',
    'item_id' => 10
  );

  return $this->theme_view('partials/upsells/'.$name, $data);
}
```

#### module_view($module, $view, $vars = array(), $parse_output = true)
它類似像 theme_view() 函式，但是是在一個 Modules 的資料夾下檢視。
這個函式永遠回傳的是 view content。
沒有被自動的寫被頁面中。

|Name|Default|Required|Description|
|---|:---:|:---:|:---|
|module|  |Yes|The module to look in|
|view|  |Yes|Path to the view file|
|vars|array()|No|Pass data to the view for parsing|
|parse_output|true|No|Parse output with LEX Parser?|

##### Example
載入預設搜尋過濾執行

```php
// plugin method
public function search_form() {
  $type = $this->attributes('type', 'all');
  $data = array(
    'filter' => $type
  );

  return $this->module_view('recipes', 'partials/search_form', $data);
}
```
