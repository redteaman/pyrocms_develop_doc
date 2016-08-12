# pyrocms_develop_doc

## Developing Widgets

Widgets 跟 Plugins 在內容加入上的方式是相當類似的，但是 Widgets 更多了一點靈巧。
它們能達到即使是你最沒有經驗的客戶或者管理者去管理在他們的網站上的智能內容的部份而不需要去學習載入 tags , HTML 或者打電話請你幫忙。

Widget Areas 可以被定義 ( header, sidebar, blog page footers, 及其它)，然後 Whdget Instances 可以它們加入其中。
可用的 Widgets 目前包含了 HTML 區塊、Twitter Feeds, RSS Feeds, Google Maps 和社群書籤。
隨著時間的推移更多的 Widgets 將會被加進來，然後你可以更容易自行操作。

### Where do I put my widgets? 我可以在哪裡放置我的 widgets 檔案？
Widgets 可以直接被儲存在以下三個地方

* /addons/<site-ref>/widgets/<widget-name> 資料夾
* /addons/shared_addons/widgets/<widget-name> 資料夾
* 或是在 Module 資料夾下: /addons/modules/<module-name>/widgets/<widget-name> 資料夾

### What are the main components of a widget? Widget 的主要元件為何
Widget 的主要元件有: 
* Widget 的 class
* views/form.php - 給予管理界面來呈現的樣式
* views/display.php - 讓前端顯示輸出的樣式

### How should a widget be structured? Widget 是如何構成的？
例如，我們想要建立一個名叫「awesome_sauce」的 widget。這個 widget 的結構會是這樣：

```
- awesome_sauce (folder)
  * awesome_sauce.php (widget class)
  - views (folder)
      * form.php (admin view)
      * display.php (frontend view)
```

現在我們來更詳盡的來說明每個元件：

#### The widget class file

```php
<?php if (!defined('BASEPATH')) exit('No direct script access allowed');

class Widget_Awesome_sauce extends Widgets
{
    // The widget title,  this is displayed in the admin interface
    public $title = 'Awesome Sauce';

    //The widget description, this is also displayed in the admin interface.  Keep it brief.
    public $description =  'Display sauce that is awesome.';

    // The author's name
    public $author = 'Some Guy';

    // The authors website for the widget
    public $website = 'http://example.com/';

    //current version of your widget
    public $version = '1.0';

    /**
     * $fields array fore storing widget options in the database.
     * values submited through the widget instance form are serialized and
     * stored in the database.
     */
    public $fields = array(
        array(
            'field'   => 'field_name',
            'label'   => 'field_label',
            'rules'   => 'required'
        )
    );

    /**
     * the $options param is passed by the core Widget class.  If you have
     * stored options in the database,  you must pass the paramater to access
     * them.
     */
    public function run($options)
    {
        if(empty($options['field_name']))
        {
            //return an array of data that will be parsed by views/display.php
            return array('output' => '');
        }

        // Store the feed items
        return array('output' => $options['html']);
    }

    /**
     * form() is used to prepare/pass data to the widget admin form
     * data returned from this method will be available to views/form.php
     */
    public function form()
    {
        $stuff = $this->db->get_stuff();
        return array('stuff' => $stuff);
    }

    /**
     * save() is used to alter submited data before their insertion in database
     */
    public function save($options)
    {
       if(isset($options['foo']) && !isset($options['bar']){
           $options['bar'] = $options['foo'];
       }
       return $options;
    }
}
```
如果你已經做到這裡，並且已經閱讀了上面的程式及註釋，這應該會讓你在開發你的第一個 Widget 上保持在正確的道路。
不過我想要在接下來的動作前指出一件事。

Widget Class 名稱應該要採用以下格式: __Widget_Awesome_sauce__ ，以及必須繼承 __Widget__ 核心 Class。

#### The view files
不用說這裡的 View files 應該只有一部份的 Views。
View files 裡不應該包含了 HTML, body, head 元素。
View Files 必須要被分別命名為 form.php 和 display.php。

### Example usage 使用範例
你需要做的是在開啟一個 theme layout 或 page layout，然後輸入: 
```
{{ widgets:area slug="sidebar" }}
```
