# pyrocms_develop_doc

## Basic

PyroCMS 裡你應該熟悉了解開發前的一些基本概念。下面的章節將讓你開始：

* 附加組件目錄 ( Add-ons Directory )
* 模塊化路由 ( Modular Routing )
* 環境 ( Environments )
* 分析您的網站
* 基本控制器

### The Add-ons Directory 附加組件目錄

在 /addon/ 目錄是所有的客製化代碼的主要位置，等同於一個 CodeIgniter "Application Package"。此文件夾的主要用途是存放所有第三方或客製化模組，部件 ( widgets ) 和插件 ( plugins )，但它也可以用來分享你的 add-ons 之間的 custom config, libraries, helpers and language files 。

PyroCMS 專業版可同時運作多個網站，因此在 Add-ons 在 PyroCMS 下網站之間進行分配，因此可以儲存在以下位置之中：
* addons/shared_addons - 這是提供給所有網站
* addons/default - 僅適用於默認的站點
* addons/<site-name> - 提供給您的客製化網站

以下是你可以存取的 addon 目錄下的資料夾完整列表：
~~~
config/
helpers/
language/
libraries/
models/
modules/
plugins/
themes/
widgets/
~~~
所有的客製化代碼應該符合這些文件夾，並應以同樣的方式進行開發，如同於你在一個 CodeIgniter application。

一個重要的注意的要點，你如果不知道到那裡尋找，是不能 CodeIgniter Loader class 自動在 /system/ 目錄中 extend  helpers 及 Library 函式庫（system/cms or system/codeigniter 都不行）。要做到這一點的唯一方法是 extend addon 資料夾下的函式庫，並手動引用它。

要擴展核心函式庫，建立類似於下面的 Class：
```php
class Search_Input extends CI_Input
{
    // Do something...
}
```
將它保存在函式庫資料夾，並加載它是必要的：
```php
$this->load->library('search_input');
```
我們不建議在 system/cms 或 system/codeigniter 資料夾擴展或覆蓋函式庫，因為這破壞了輕鬆升級 PyroCMS 的能力，使其難以在以後找到你的程式碼。

以類似的方式有哪些配置文件即可投入config文件夾限制。例如;像 autoload.php，為 database.php 和 routes.php文件的配置文件在這裡將沒有任何效果。建議你只在這裡放 config files，將其下載和你自己的 addon 中使用。

你可以使用 addons/shared_addons/ 資料夾 與舊 addon 資料夾完全相同的方式。唯一的區別是，當你從該網站的管理面板上傳 Module 或一個 theme，它將被放置在 addons/default。然後，如果你升級到專業版在某些時候這 theme、Module 或 widget 將被限制只有一個站點。如果升級到專業版，每增加一個站點 add-on 將為個別創建且放置在自己的資料夾（addon/ site_name）。

### Modular Routing 模塊化路由

路由是一個強大的工具，允許開發人員建立不存在 Module/ [controller]/method/參數 的標準模組模式的自定義URL。Route 配置文件可被置於每個 Module，以幫助每個Moudule Route 的URL，
但 PyroCMS 只知道從第一個 URI segment 下為 Module name，並且使用其 Module 下的 route.php

例如
```
/artists/top-10
```
這個 URL 將告知 PyroCMS 去載入 /addons/modules/artists/config/routes.php
```
/top-10-artists
```
這個測試沒辦法告知哪個 Module 可以使用，因此 route 應該設定在 system/cms/config/routes.php.

編輯 PyroCMS 的 main route file 看起來不像一個好主意，
但如果是昇級時依照 config.php 和 database.php 來備份，這裡是沒有缺點的。
路由做這種方式，因為在每個頁面載入之前，每個單一 route.php 已經被載入了，然而效能將會嚴重被影響。


### Environments 環境
開發人員取決於不論是單一開發應用或者是正式上線環境，通常希望有不同的系統行為，
例如，詳細的 error report 對於開發一個應用程式，它是有用的，但它也可能會帶來安全問題，當系統為“live”時。

環境恆定
默認情況下，PyroCMS自帶環境不斷設置為 develop。
在根目錄 index.php 的頂端，你會看到:
```php
define('PYRO_DEVELOPMENT', 'development');
define('PYRO_STAGING', 'staging');
define('PYRO_PRODUCTION', 'production');

define('ENVIRONMENT', (isset($_SERVER['PYRO_ENV']) ? $_SERVER['PYRO_ENV'] : PYRO_DEVELOPMENT));
```
除了影響一些基本框架的行為（見下一節），你可以使用這個 constant  在自己開發環境及運行程式。

#### 對默認的行為框架
有在PyroCMS一些地方使用環境常數。本節介紹的是默認的框架行為是如何影響。

##### Error Reporting
環境設定為 development 時，它將會把所有的 PHP 錯誤顯示在瀏覽器上。
相反的設定為 production 將會禁止所有錯誤輸出。
在上線狀態下禁用 Error Reporting 是一項很好的安全行為。

##### Configuration Files
或者，你可以有 PyroCMS 載入特定環境下的配置文件。
這可能是用於管理像跨多個環境不同 API Key 的情境。
CodeIgniter Config Class 裡有更多關於環境的細節及描述

##### Setting $_SERVER['PYRO_ENV']
改變環境最簡單的方法就是讓你的服務器知道什麼 envionment PyroCMS 是期望的那樣。
如果你使用用一個漂亮的 UI 及平台來提供服務，像是 Pagoda Box 或 PHP Fog 來進行管理你的主機，但可以做得更複雜些。 
Apache 透過 mod_env 來支援 SetEnv。
這可以在 Apache 設定來完成，也可以在根目錄中打開 .htaccess，並且拿掉該行的 
```
# SetEnv PYRO_ENV production
```
 
 
### Profiling Your Site 分析您的網站
如果你可以在任何時候透過 CodeIgniter profiler 來在一個管理員新增東西加進行記錄。
或是在你的 URL 最後面加入 _debug
```
http://example.com/?_debug
```


### Base Controllers 基本控制器
PyroCMS 附帶了幾個基本控制器擴充，讓您挖掘現有的功能。您可以通過 PyroCMS base controllers 來擴充你的 controllers：
```php
class Blog extends Public_Controller
{
    public function __construct()
    {
```
 
#### MY_Controller
MY_Controller是底層 controller extend，因此，如果您使用的是 Public_Controller 或 Admin_Controller，MY_Controller 的功能已經包括在內。 MY_Controller 執行以下操作：

* 對於SITE_REF不斷檢查，以確保該網站成立
* 設置dbprefix到SITE_REF
* 確保運作是現在
* 定義 CURRENT_LANGUAGE constant
* 加載用戶登錄系統
* 收集有關當前用戶的數據到 $this -> CURRENT_USER
* 處理基於模組和用戶權限
* 在設置上設定輸出及檢查

#### Public_Controller
用於 Front-end controllers，Public_Controller class 將會在你的 controller 之前執行
* 在依照重新導向 Module 進行重新導向
* 調用 public_controller 事件。
* 確認是否禁用該網站，並且在主要時間內設置網站禁用設定
* 載入網站主題，設定主題路徑
* 設定 APPPATH_URI 和 BASE_URI 的 JavaScript 變數
* 設定規範的 URL 連結及 header metadata
* 在 header 的 metadata 設定 RSS blog 連結
* 從變數 Module 中載入變數
* 載入主題選項

#### Admin_Controller
用於所有的 admin controllers，有以下行為：
* 檢查 user 是否具有控制面版的存取
* 如果需要，設定 HTTPS 的 requests
* 設定管理的主題，並載入必要的路徑
* 設定 template enable_parser 選項為 false。可以從你的管理面版來設定它為 true 在使用 PyroCMS tags 
