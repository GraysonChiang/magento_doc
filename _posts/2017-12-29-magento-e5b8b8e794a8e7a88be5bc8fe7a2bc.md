---
ID: 108
post_title: Magento 常用程式碼
author: Grayson
post_excerpt: ""
layout: post
permalink: 'http://magento.im/2017/12/29/magento-%e5%b8%b8%e7%94%a8%e7%a8%8b%e5%bc%8f%e7%a2%bc/'
published: true
post_date: 2017-12-29 15:32:13
---
這邊紀錄了許多常用的程式碼，大部分是從 <code>segmentfault</code> 參考而來，再加上自己的發現的部分程式碼結合而成，單純做個紀錄，希望幫助到大家。

<h2>取得 objectManager</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    //object Maganer 是 Magento 中好用的方法，不過官方不建議直接使用
    $objectManager = \Magento\Framework\App\ObjectManager::getInstance();
</code></pre>

<br>

<h2>DataObject 所有 Model 的基礎</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $row = new \Magento\Framework\DataObject();
    $row-&gt;setData($key, $value);
    $row-&gt;getData($key);
    $row-&gt;hasData($key);
    $row-&gt;unsetData();
    $row-&gt;toXml();
    $row-&gt;toJson();
    $row-&gt;debug();
</code></pre>

<br>

<h2>Session</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $session = $objectManager-&gt;get('Magento\Framework\Session\Storage');
    $session-&gt;setXxxx($value);
    $session-&gt;getXxxx();
    $session-&gt;hasXxxx();
    $session-&gt;unsXxxx();
</code></pre>

<br>

<h2>Cache</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    // save
    $this-&gt;cache = $context-&gt;getCache();
    $this-&gt;cache-&gt;save(\Zend_Json::encode($data), self::CACHE_PREFIX . $key, [], $lifeTime);

    // load
    $jsonStr = $this-&gt;cache-&gt;load(self::CACHE_PREFIX . $cacheKey);
    if (strlen($jsonStr)) {
        $this-&gt;cachedData = \Zend_Json::decode($jsonStr);
    }

    // sample
    $cache = $objectManager-&gt;get(\Magento\Framework\App\CacheInterface::class);
    $cached = $cache-&gt;load($cacheKey);
    if(strlen($cached)) {
        $data = unserialize($cached);
    } else {
        $data = [];
        $product = $objectManager-&gt;create('Magento\Catalog\Model\ProductRepository')-&gt;getById($id);
        $row = new \Magento\Framework\DataObject();
        $row-&gt;setData($row-&gt;getData());
        $data []= $row;
        $cache-&gt;save(serialize($data), $cacheKey);
    }
</code></pre>

<br>

<h2>判斷是否為首頁</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">// in action
if($this-&gt;_request-&gt;getRouteName() == 'cms' &amp;&amp; $this-&gt;_request-&gt;getActionName() == 'index') {
    // is home
}

</code></pre>

<br>

<h2>Registry 用於內部傳遞臨時變數</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php

/* @var \Magento\Framework\Registry $coreRegistry */
$coreRegistry = $this-&gt;_objectManager-&gt;get('Magento\Framework\Registry');

// 存储值
$coreRegistry-&gt;register('current_category', $category);

// 提取值
$coreRegistry-&gt;registry('current_category');
</code></pre>

<br>

<h2>取得當前 Store</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
$store = $objectManager-&gt;get( 'Magento\Store\Model\StoreManagerInterface' )-&gt;getStore();

</code></pre>

<br>

<h2>提示訊息 Message</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    //  \Magento\Framework\App\Action\Action::execute()
    $this-&gt;messageManager-&gt;addSuccessMessage(__('You deleted the event.'));
    $this-&gt;messageManager-&gt;addErrorMessage(__('You deleted the event.'));
    return $this-&gt;_redirect ( $returnUrl );
</code></pre>

<br>

<h2>log ( support_report.log )</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    \Magento\Framework\App\ObjectManager::getInstance()
        -&gt;get('\Psr\Log\LoggerInterface' )
        -&gt;addCritical('notice message', 
        [
            'order_id' =&gt; $order_id,
            'item_id' =&gt; $item_id
            // ...
        ]);
</code></pre>

<br>

<h2>取得當前頁面 URL</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $currentUrl = $objectManager-&gt;get( 'Magento\Framework\UrlInterface' )-&gt;getCurrentUrl();
</code></pre>

<br>

<h2>取得指定路由 URL</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $url = $objectManager-&gt;get( 'Magento\Store\Model\StoreManagerInterface' )-&gt;getStore()-&gt;getUrl( 'xxx/xxx/xxx' );
</code></pre>

<br>

<h2>Block 裡的 URL</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    // 首页地址
    $block-&gt;getBaseUrl();

    // 指定 route 地址
    $block-&gt;getUrl( '[module]/[controller]/[action]' );

    // 指定的静态文件地址
    $block-&gt;getViewFileUrl( 'Magento_Checkout::cvv.png' );
    $block-&gt;getViewFileUrl( 'images/loader-2.gif' );

</code></pre>

<br>

<h2>取得所有 website 的位置</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $storeManager = $objectManager-&gt;get('Magento\Store\Model\StoreManagerInterface');
    foreach($storeManager-&gt;getWebsites() as $website) {
        $store_id = $storeManager-&gt;getGroup($website-&gt;getDefaultGroupId())-&gt;getDefaultStoreId();
        $storeManager-&gt;getStore($store_id)-&gt;getBaseUrl();
    }
</code></pre>

<br>

<h2>取得 module 下的文件</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /** @var \Magento\Framework\Module\Dir\Reader $reader */
    $reader = $objectManager-&gt;get('Magento\Framework\Module\Dir\Reader');
    $reader-&gt;getModuleDir('etc', 'Infinity_Project').'/di.xml';
</code></pre>

<br>

<h2>取得資源路徑</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php"> &lt;?php
    /* @var \Magento\Framework\View\Asset\Repository $asset */
    $asset = $this-&gt;_objectManager-&gt;get( '\Magento\Framework\View\Asset\Repository' );
    $asset-&gt;createAsset('Vendor_Module::js/script.js')-&gt;getPath();
</code></pre>

<br>

<h2>檔案操作</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php"> &lt;?php
    /* @var \Magento\Framework\Filesystem $fileSystem */
    $fileSystem = $this-&gt;_objectManager-&gt;get('Magento\Framework\Filesystem');
    $fileSystem-&gt;getDirectoryWrite('tmp')-&gt;copyFile($from, $to);
    $fileSystem-&gt;getDirectoryWrite('tmp')-&gt;delete($file);
    $fileSystem-&gt;getDirectoryWrite('tmp')-&gt;create($file);
</code></pre>

<br>

<h2>檔案上傳</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php"> &lt;?php
    // 上傳到 pub/media 目錄中
    $request = $this-&gt;getRequest ();
    if($request-&gt;isPost()) {
     try{
         /* @var $uploader \Magento\MediaStorage\Model\File\Uploader */
         $uploader = $this-&gt;_objectManager-&gt;create(
             'Magento\MediaStorage\Model\File\Uploader',
             ['fileId' =&gt; 'personal_photos']
        );
        /* @var $filesystem \Magento\Framework\Filesystem */
        $filesystem = $this-&gt;_objectManager-&gt;get( 'Magento\Framework\Filesystem' );
        $dir = $filesystem-&gt;getDirectoryRead( \Magento\Framework\App\Filesystem\DirectoryList::UPLOAD )-&gt;getAbsolutePath();
        $fileName = time().'.'.$uploader-&gt;getFileExtension();
        $uploader-&gt;save($dir, $fileName);
        $fileUrl = 'pub/media/upload/'.$fileName;
    } catch(Exception $e) {
        // 未上传
    }
}
</code></pre>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
// 上傳到 pub/media/tmp 資料夾
    /* @var \Magento\Framework\App\Filesystem\DirectoryList $directory */
    $directory = $this-&gt;_objectManager-&gt;get('Magento\Framework\App\Filesystem\DirectoryList');

    /* @var \Magento\Framework\File\Uploader $uploader */
    $uploader = $this-&gt;_objectManager-&gt;create('Magento\Framework\File\Uploader', array('fileId' =&gt; 'file1'));
    $uploader-&gt;setAllowedExtensions(array('csv'));
    $uploader-&gt;setAllowRenameFiles(true);
    $uploader-&gt;setFilesDispersion(true);
    $result = $uploader-&gt;save($directory-&gt;getPath($directory::TMP));
    $directory-&gt;getPath($directory::TMP).$result['file'];

</code></pre>

<br>

<h2>產品縮圖</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $imageHelper = $objectManager-&gt;get( 'Magento\Catalog\Helper\Image' );

    $productImage = $imageHelper-&gt;init( $product, 'category_page_list' )
      -&gt;constrainOnly( FALSE )
      -&gt;keepAspectRatio( TRUE )
      -&gt;keepFrame( FALSE )
      -&gt;resize( 400 )
      -&gt;getUrl();
</code></pre>

<br>

<h2>一般縮圖</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $imageFactory = $objectManager-&gt;get( 'Magento\Framework\Image\Factory' );
    $imageAdapter = $imageFactory-&gt;create($path);
    $imageAdapter-&gt;resize($width, $height);
    $imageAdapter-&gt;save($savePath);
</code></pre>

<br>

<h2>產品属性</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /* @var \Magento\Catalog\Model\ProductRepository $product */
    $product = $objectManager-&gt;create('Magento\Catalog\Model\ProductRepository')-&gt;getById($id);

    // print price
    /* @var \Magento\Framework\Pricing\PriceCurrencyInterface $priceCurrency */
    $priceCurrency = $objectManager-&gt;get('Magento\Framework\Pricing\PriceCurrencyInterface');
    $priceCurrency-&gt;format($product-&gt;getData('price'));

    // print option value
    $product-&gt;getResource()-&gt;getAttribute('color')-&gt;getDefaultFrontendLabel();
    $product-&gt;getAttributeText('color');

    // all options
    $this-&gt;helper('Magento\Catalog\Helper\Output')-&gt;productAttribute($product, $product-&gt;getData('color'), 'color');
    $product-&gt;getResource()-&gt;getAttribute('color')-&gt;getSource()-&gt;getAllOptions();

    // save attribute
    $product-&gt;setWeight(1.99)-&gt;getResource()-&gt;saveAttribute($product, 'weight');

    // 库存
    $stockItem = $this-&gt;stockRegistry-&gt;getStockItem($productId, $product-&gt;getStore()-&gt;getWebsiteId());
    $minimumQty = $stockItem-&gt;getMinSaleQty();

    // Configurable Product 获取父级產品 ID
    $parentIds = $this-&gt;objectManager-&gt;get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    -&gt;getParentIdsByChild($productId);
    $parentId = array_shift($parentIds);

    // Configurable Product 获取子级产品ID
    $childIds = $this-&gt;objectManager-&gt;get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    -&gt;getChildrenIds($parentId);

    // 获取Configurable Product的子产品Collection
    $collection = $this-&gt;objectManager-&gt;get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    -&gt;getUsedProducts($product);

    // 判断是否Configurable Product
    if ($_product-&gt;getTypeId() == \Magento\ConfigurableProduct\Model\Product\Type\Configurable::TYPE_CODE);

</code></pre>

<br>

<h2>取得購物車中所有商品</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /* @var \Magento\Checkout\Model\Session $checkoutSession */
    $checkoutSession = $objectManager-&gt;get('Magento\Checkout\Model\Session');
    foreach($checkoutSession-&gt;getQuote()-&gt;getItems() as $item) {
     /* @var \Magento\Quote\Model\Quote\Item $item */
     echo $item-&gt;getProduct()-&gt;getName().'&lt;br/&gt;';
    }
</code></pre>

<br>

<h2>取得 Product Type 配置</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $eavConfig-&gt;getAttribute('catalog_product', 'price');
    $eavConfig-&gt;getEntityType('catalog_product');
</code></pre>

<br>

<h2>取得 EAV 属性所有可選選項</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /* @var $objectManager \Magento\Framework\App\ObjectManager */
    $eavConfig = $objectManager-&gt;get( '\Magento\Eav\Model\Config' );
    $options = $eavConfig-&gt;getAttribute( '[entity_type]', '[attribute_code]' )
    -&gt;getFrontend()-&gt;getSelectOptions();

    //或者
    /* @var $objectManager \Magento\Framework\App\ObjectManager */
    $options = $objectManager-&gt;create( 'Magento\Eav\Model\Attribute' )
    -&gt;load( '[attribute_code]', 'attribute_code' )-&gt;getSource()
    -&gt;getAllOptions( false );
</code></pre>

<br>

<h2>取得 config.xml 與 system.xml 裡的參數</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $this-&gt;_scopeConfig = $this-&gt;_objectManager-&gt;create('Magento\Framework\App\Config\ScopeConfigInterface');

    // 语言代码
    $this-&gt;_scopeConfig-&gt;getValue('general/locale/code', \Magento\Store\Model\ScopeInterface::SCOPE_STORE, $storeId);
</code></pre>

<br>

<h2>客戶唯一屬性驗證</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    if($customer instanceof \Magento\Customer\Model\Customer) {
        /* @var \Magento\Customer\Model\Attribute $attribute */
        foreach($customer-&gt;getAttributes() as $attribute) {
            if($attribute-&gt;getIsUnique()) {
                if (!$attribute-&gt;getEntity()-&gt;checkAttributeUniqueValue($attribute, $customer)) {
                    $label = $attribute-&gt;getFrontend()-&gt;getLabel();
                    throw new \Magento\Framework\Exception\LocalizedException(
                        __('The value of attribute "%1" must be unique.', $label)
                    );
                }
            }
        }
    }
</code></pre>

<br>

<h2>讀取 design view.xml</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-xml">&lt;vars module="Vendor_Module"&gt;
    &lt;var name="var1"&gt;value1&lt;/var&gt;
&lt;/vars&gt;
</code></pre>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /* @var \Magento\Framework\Config\View $viewConfig */
    $viewConfig = $objectManager-&gt;get('Magento\Framework\Config\View');
    $viewConfig-&gt;getVarValue('Vendor_Module', 'var1');
</code></pre>

<br>

<h2>取得郵件模組</h2>

<h5>雖然叫郵件模組，但也可以用於需要後台編輯模板的程式</h5>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    // template id, 通常在 email_templates.xml 定義。如果是在後台加的 email template，需要换成template 的紀錄 ID，例如 90
    $identifier = 'contact_email_email_template';

    /* @var \Magento\Framework\Mail\TemplateInterface $templateFactory */
    $templateFactory = $this-&gt;_objectManager-&gt;create(
        'Magento\Framework\Mail\TemplateInterface',
        ['data' =&gt; ['template_id' =&gt; $identifier]]
    );

    // 模板變數，取决於 phtml 或後台 email template 的内容
    $dataObject = new \Magento\Framework\DataObject();
    $dataObject-&gt;setData('name', 'william');

    // 決定模板變數取值區域，例如像 {{layout}} 這樣的標籤，如果取不到值可以試試把 area 設為 frontend
    $templateFactory-&gt;setOptions([
        'area' =&gt; \Magento\Backend\App\Area\FrontNameResolver::AREA_CODE,
        'store' =&gt; \Magento\Store\Model\Store::DEFAULT_STORE_ID
    ]);

    $templateFactory-&gt;setVars(['data' =&gt; $dataObject]);

    return $templateFactory-&gt;processTemplate();
</code></pre>

<br>

<h2>内容返回</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $this-&gt;resultFactory = $this-&gt;_objectManager-&gt;create('Magento\Framework\Controller\Result\RawFactory');
    /* @var \Magento\Framework\Controller\Result\Raw $result */
    $result = $this-&gt;resultFactory-&gt;create();
    $result-&gt;setContents('hello world');
    return $result;
</code></pre>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php"> &lt;?php
    $this-&gt;resultFactory = $this-&gt;_objectManager-&gt;create('Magento\Framework\Controller\Result\JsonFactory');

    /* @var \Magento\Framework\Controller\Result\Json $result */
    $result = $this-&gt;resultFactory-&gt;create();
    $result-&gt;setData(['message' =&gt; 'hellog world']);
    return $result;
</code></pre>

<br>

<h2>HTTP文件</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php"> &lt;?php
    $this-&gt;_fileFactory = $this-&gt;_objectManager-&gt;create('Magento\Framework\App\Response\Http\FileFactory');
    $this-&gt;_fileFactory-&gt;create(
            'invoice' . $date . '.pdf',
            $pdf-&gt;render(),
            DirectoryList::VAR_DIR,
            'application/pdf'
    );
</code></pre>

<br>

<h2>切换貨幣與語言</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $currencyCode = 'GBP';

    /* @var \Magento\Store\Model\Store $store */
    $store = $this-&gt;_objectManager-&gt;get('Magento\Store\Model\Store');
    $store-&gt;setCurrentCurrencyCode($currencyCode);

    $storeCode = 'uk';

    /* @var \Magento\Store\Api\StoreRepositoryInterface $storeRepository */
    $storeRepository = $this-&gt;_objectManager-&gt;get('Magento\Store\Api\StoreRepositoryInterface');

    /* @var \Magento\Store\Model\StoreManagerInterface $storeManager */
    $storeManager = $this-&gt;_objectManager-&gt;get('Magento\Store\Model\StoreManagerInterface');

    /* @var \Magento\Store\Api\StoreCookieManagerInterface $storeCookieManager */
    $storeCookieManager = $this-&gt;_objectManager-&gt;get('Magento\Store\Api\StoreCookieManagerInterface');

    /* @var \Magento\Framework\App\Http\Context $httpContext */
    $httpContext = $this-&gt;_objectManager-&gt;get('Magento\Framework\App\Http\Context');
    $defaultStoreView = $storeManager-&gt;getDefaultStoreView();
    $store = $storeRepository-&gt;getActiveStoreByCode($storeCode);
    $httpContext-&gt;setValue(\Magento\Store\Model\Store::ENTITY, $store-&gt;getCode(), $defaultStoreView-&gt;getCode());
    $storeCookieManager-&gt;setStoreCookie($store);
    $this-&gt;getResponse()-&gt;setRedirect($this-&gt;_redirect-&gt;getRedirectUrl());
</code></pre>

<br>

<h2>Profiler</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    \Magento\Framework\Profiler::start(
        'CONFIGURABLE:' . __METHOD__,
        ['group' =&gt; 'CONFIGURABLE', 'method' =&gt; __METHOD__]
    );
    \Magento\Framework\Profiler::stop('CONFIGURABLE:' . __METHOD__);
</code></pre>

<br>

<h2>HTML 表單元素</h2>

<h3>日曆元件</h3>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $block-&gt;getLayout()-&gt;createBlock('Magento\Framework\View\Element\Html\Date')
        -&gt;setName('date')
        -&gt;setId('date')
        -&gt;setClass('date')
        -&gt;setDateFormat('M/d/yy')
        -&gt;setImage($block-&gt;getViewFileUrl('Magento_Theme::calendar.png'))
        -&gt;setExtraParams('data-validate="{required:true}"')
        -&gt;toHtml();
</code></pre>

<br>

<h2>Select 元件</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $block-&gt;getLayout()-&gt;createBlock('Magento\Framework\View\Element\Html\Select')
        -&gt;setName('sel1')
        -&gt;setId('sel1')
        -&gt;setClass('select')
        -&gt;addOption('value', 'label')
        -&gt;setValue($default)
        -&gt;setExtraParams('data-validate="{required:true}"')
        -&gt;toHtml();
</code></pre>

<br>

<h2>取得當前 Category</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    // 使用 registr  可以傳遞的變數有很多，這是其中之一
  $registry = $objectManager-&gt;get('Magento\Framework\Registry');
  $currentCatetory = $registry-&gt;registry('current_category');
  echo $catId = $currentCatetory-&gt;getId();

</code></pre>

<br>

<h2>使用資料Select</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $collection = $this-&gt;_objectManager-&gt;get( 'Magento\Catalog\Model\ProductFactory' )-&gt;create()-&gt;getCollection();
    echo $collection-&gt;load()-&gt;getSelectSql( true );
</code></pre>

<br>

<h2>使用 Select 取得數據</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $conn = $this-&gt;_objectManager-&gt;get( 'Magento\Framework\App\ResourceConnection' )-&gt;getConnection();
    $tblMain = $conn-&gt;getTableName( 'cms_page' );
    $tblStore = $conn-&gt;getTableName( 'cms_page_store' );

    $select = $conn-&gt;select()
            -&gt;distinct()
            -&gt;from( [ 'page' =&gt; $tblMain ], [ 'page_id', 't' =&gt; 'title' ] )
            -&gt;join( [ 'store' =&gt; $tblStore ], 'store.page_id = page.page_id', [ 'sid' =&gt; 'store_id' ] )
            -&gt;where( 'is_active = ?', '1' )
            -&gt;order( 'title ASC' )
            -&gt;limit( 5, 1 );

    $data = $conn-&gt;fetchAll( $select );
</code></pre>

<br>

<h2>聚合查询（ SUM ）</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $conn = $this-&gt;_objectManager-&gt;get( 'Magento\Framework\App\ResourceConnection' )-&gt;getConnection();
    $tbl = $conn-&gt;getTableName( 'sales_order_item' );

    // 所有记录总计
    $select = $conn-&gt;select()
            -&gt;from( $tbl, [ 'total' =&gt; new \Zend_Db_Expr( 'SUM( qty_ordered )' ) ] )
            -&gt;where( 'order_id = ?', '1' );
    $result = $conn-&gt;fetchOne( $select );

    // 各局部统计
    $select = $conn-&gt;select()
            -&gt;from( $tbl, [ 'order_id', 'total' =&gt; new \Zend_Db_Expr( 'SUM( qty_ordered )' ) ] )
            -&gt;group( 'order_id' )
            -&gt;having( 'order_id != ?', '1' );
    $result = $conn-&gt;fetchAll( $select );
</code></pre>

<br>

<h2>更新数据</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
/** @var $conn \Magento\Framework\App\ResourceConnection */
/** @var $storeId int */
/** @var $ids array */

$tbl = $conn-&gt;getTableName( 'sales_order_item' );

// 插入数据
$conn-&gt;insert( $tbl, [ 'field_one' =&gt; $fieldOne, 'field_two' =&gt; $fieldTwo, 'field_three' =&gt; $fieldThree ] );

// 插入数据，碰到已存在的记录（primary / unique）则只更新指定的字段
$conn-&gt;insertOnDuplicate( $tbl, [ 'field_one' =&gt; $fieldOne, 'field_two' =&gt; $fieldTwo, 'field_three' =&gt; $fieldThree ], [ 'field_three' ] );

// 更新数据
$conn-&gt;update( $tbl, [ 'field_one' =&gt; $fieldOne, 'field_two' =&gt; $fieldTwo ], $conn-&gt;quoteInto( 'store_id IN (?)', $storeId ) );

// 删除数据
$conn-&gt;delete( $tbl, [ 'id IN (?)' =&gt; $ids ] );
</code></pre>

<br>

<h2>Model</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    /* @var \Magento\Directory\Model\ResourceModel\Region\Collection $collection */
    $collection = $objectManager-&gt;get('Magento\Directory\Model\ResourceModel\Region\Collection');

    $collection-&gt;getSelect()
        -&gt;where('main_table.region_id = ?', 1)
        -&gt;where('main_table.country_id=?', 'US');
            foreach($collection as $row) {
                    echo $row-&gt;getData('default_name');
                }
</code></pre>

<br>

<h1>Entity collection</h1>

<h2>常用方法</h2>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $collection-&gt;addAttributeToFilter($field, [ 'in' =&gt; $arr ]);
    $collection-&gt;addAttributeToSort($field, 'desc');
    $collection-&gt;addStoreFilter();
    $collection-&gt;addUrlRewrite();
    $collection-&gt;setPageSize( 10 );
    $collection-&gt;addAttributeToSelect($field);
</code></pre>

<br>

<h1>包含 Field 的查询</h1>

<h4>Entity Collection 有查詢 field 的能力，只要查詢條件裡跟 field 有關，就會自動把 field 所在的表 join 到查詢中</h4>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php

    $collection-&gt;addAttributeToSelect($field, 'left'); // 等同于select xx as field ... left join xxx
    $collection-&gt;addAttributeToFilter($field, 'field &gt; 0')// 等同于where field &gt; 0
    $collection-&gt;setOrder($field, 'ASC'); // 等同于order by xxx ASC
</code></pre>

<br>

<h1>特價置頂</h1>

<h4>Special Price 是個特别的個案，需要在有效期間生效，有效期外不生效</h4>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-php">&lt;?php
    $collection-&gt;addAttributeToSelect('special_from_date', 'left');
    $collection-&gt;addAttributeToSelect('special_to_date', 'left');
    $collection-&gt;addAttributeToSelect('special_price', 'left');
    $collection-&gt;getSelect()-&gt;order('IF(special_from_date &lt; now() AND special_to_date &gt; now(), special_price, 0) DESC');

</code></pre>

<br><br>

<h2>參考文件</h2>

<ul>
<li><a href="https://segmentfault.com/a/1190000005154774#articleHeader31">Segmentfault.com</a></li>
</ul>