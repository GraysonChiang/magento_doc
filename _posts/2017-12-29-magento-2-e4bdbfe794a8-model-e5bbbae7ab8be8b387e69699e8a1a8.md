---
ID: 96
post_title: Magento 2 使用 Model 建立資料表
author: Grayson
post_excerpt: ""
layout: post
permalink: 'http://magento.im/2017/12/29/magento-2-%e4%bd%bf%e7%94%a8-model-%e5%bb%ba%e7%ab%8b%e8%b3%87%e6%96%99%e8%a1%a8/'
published: true
post_date: 2017-12-29 15:05:19
---
<h2>說明</h2>

Magento 是一個開源的電子商城購物網站，在客製化的時候，難免不了需要自己新增資料表及各式各樣的欄位，在 Magento 內部有實作了很好用的 方法，有助於我們資料表 Schema 的建立及還原。

<h2>執行環境</h2>

<ul>
<li>Ubuntu Linux 16.04 LTS</li>
<li>PhpStorm 2017.3
<br><br></li>
</ul>

<h2>1. Magento Setup  方法</h2>

Magento 實作了一系列的方法，在執行 <code>bin/magento setup:upgrade</code> 的時候會依照順序執行的四個 Class 。

<ul>
<li>InstallSchema</li>
<li>InstallData</li>
<li>UpgradeSchema</li>
<li>UpgradeData</li>
</ul>

所以如果要在 <code>upgrade</code> 的時候執行這些 Class 內程式的話，就必須依照 Magento 的方法實作我們先實作空的檔案當作範例，之後加入 Schema 的資料。

<br>

<h2>2.  建立 InstallSchema Class</h2>

範例如下，看到他繼承 <code>Magento\Framework\Setup\InstallSchemaInterface</code> 這個 Namespace 的 Class。

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
namespace Astralweb\ORM\Setup;

use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class InstallSchema implements InstallSchemaInterface
{
    /**
     * Function install
     * @param SchemaSetupInterface $setup
     * @param ModuleContextInterface $context
     * @return void
     */
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $installer = $setup;
        $installer-&gt;startSetup();
        $installer-&gt;endSetup();
    }
}
</code></pre>

<br>

<h2>3.  建立 InstallData Class</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
namespace Astralweb\ORM\Setup;

use Magento\Framework\Setup\InstallDataInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class InstallData implements InstallDataInterface
{
    /**
     * Function install
     * @param ModuleDataSetupInterface $setup
     * @param ModuleContextInterface $context
     * @return void
     */
    public function install(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
    {
        //install data here
    }
}
</code></pre>

<br>

<h2>4.  建立 UpgradeSchema Class</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
namespace Astralweb\ORM\Setup;

use Magento\Framework\Setup\UpgradeSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

/**
 * @codeCoverageIgnore
 */
class UpgradeSchema implements UpgradeSchemaInterface
{
    /**
     * {@inheritdoc}
     */
    public function upgrade(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup-&gt;startSetup();
        $connection = $setup-&gt;getConnection();
        $setup-&gt;endSetup();
    }
}
</code></pre>

<br>

<h2>5. UpgradeData Class</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
namespace Astralweb\ORM\Setup;

use Magento\Framework\Setup\ModuleDataSetupInterface;
use Magento\Framework\Setup\UpgradeDataInterface;
use Magento\Framework\Setup\ModuleContextInterface;

class UpgradeData implements UpgradeDataInterface
{
    public function upgrade(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup-&gt;startSetup();


        $setup-&gt;endSetup();
    }
}
</code></pre>

<br>

<h2>6.  建立 Schema</h2>

若以 Employee 資料表為例，資料表需求如下

<table>
<thead>
<tr>
  <th>欄位名稱</th>
  <th>型別</th>
  <th>大小</th>
  <th>主鍵</th>
</tr>
</thead>
<tbody>
<tr>
  <td>entity_id</td>
  <td>integer</td>
  <td></td>
  <td>是</td>
</tr>
<tr>
  <td>name</td>
  <td>var_char</td>
  <td>255</td>
  <td>否</td>
</tr>
<tr>
  <td>phone</td>
  <td>var_char</td>
  <td>255</td>
  <td>否</td>
</tr>
<tr>
  <td>department</td>
  <td>var_char</td>
  <td>255</td>
  <td>否</td>
</tr>
<tr>
  <td>created_at</td>
  <td>datetime</td>
  <td></td>
  <td>否</td>
</tr>
<tr>
  <td>updated_at</td>
  <td>datetime</td>
  <td></td>
  <td>否</td>
</tr>
</tbody>
</table>

若將上開欄位轉換為 Magento 的 Schema 格式的程式碼如下：

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
namespace Astralweb\ORM\Setup;

use Magento\Framework\DB\Ddl\Table;
use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class InstallSchema implements InstallSchemaInterface
{
    /**
     * Function install
     * @param SchemaSetupInterface $setup
     * @param ModuleContextInterface $context
     * @return void
     * @throws \Zend_Db_Exception
     */
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $installer = $setup;
        $installer-&gt;startSetup();

        $table = $installer-&gt;getConnection()-&gt;newTable(
            $installer-&gt;getTable('employee_entity')
        )-&gt;addColumn(
            'entity_id',
            Table::TYPE_INTEGER,
            null,
            ['identity' =&gt; true, 'nullable' =&gt; false, 'primary' =&gt; true, 'unsigned' =&gt; true,],
            'Entity ID'
        )-&gt;addColumn(
            'name',
            Table::TYPE_TEXT,
            255,
            ['nullable' =&gt; false,],
            'name'
        )-&gt;addColumn(
            'phone',
            Table::TYPE_TEXT,
            255,
            ['nullable' =&gt; false,],
            'phone'
        )-&gt;addColumn(
            'department',
            Table::TYPE_TEXT,
            255,
            ['nullable' =&gt; false,],
            'department'
        )-&gt;addColumn(
            'created_at',
            Table::TYPE_TIMESTAMP,
            null,
            ['nullable' =&gt; false, 'default' =&gt; Table::TIMESTAMP_INIT,],
            'Creation Time'
        )-&gt;addColumn(
            'update_time',
            Table::TYPE_TIMESTAMP,
            null,
            ['nullable' =&gt; false, 'default' =&gt; Table::TIMESTAMP_INIT_UPDATE,],
            'update time'
        );
        $installer-&gt;getConnection()-&gt;createTable($table);
    }
}
</code></pre>

<br>

<h2>7.  執行 upgrade 指令</h2>

設定完 Schema 之後，請在 <code>command line</code> 裡面輸入以下指令：

<blockquote>
  $ bin/magnet setup:upgrade
</blockquote>

<img src="http://magento.im/wp-content/uploads/2017/12/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7-2017-12-29-23.00.39-1.png" alt="" />

<img src="http://magento.im/wp-content/uploads/2017/12/%E5%9C%96%E7%89%87-1-7.png" alt="" />

執行完後，就可以看到資料表出現在 Magento 的 資料庫內了
<br>

<h3>參考資料：</h3>

<ul>
<li><a href="https://github.com/AstralWebTW/ORM-module" title="Github">Github 程式碼範例</a></li>
</ul>