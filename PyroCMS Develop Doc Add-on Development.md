# pyrocms_develop_doc

##Add-on Development

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
	
	{{ url:current }}

這裡程式背後的 tags 被稱為 plugin。Plugins 是特別的 PHP 檔案，它的功能被透過 PyroCMS tags 被呼叫，
並且可以像收集 tag 元素的方式來做事情。它們可以簡單的撰寫並結合了複雜的功能，整潔有序的整合進 PyroCMS 佈局。

### Modular vs Standalone Plugins 模組化 vs 自動安裝 Plugins
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

### Getting Plugin Tag Attributes 取得 Plugin Tag 屬性
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

### Tag Pairs tag 配對
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

### Tag Pair Raw Content Tag Pair 原始內容
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
