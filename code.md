# Magento 2 常用程式碼


## 取得objectManager

```php
$objectManager = \Magento\Framework\App\ObjectManager::getInstance();
```
<br>

## DataObject 所有 Model 的基礎

```php
$row = new \Magento\Framework\DataObject();
$row->setData($key, $value);
$row->getData($key);
$row->hasData($key);
$row->unsetData();
$row->toXml();
$row->toJson();
$row->debug();
```
<br>


## Session
```php
$session = $objectManager->get('Magento\Framework\Session\Storage');
$session->setXxxx($value);
$session->getXxxx();
$session->hasXxxx();
$session->unsXxxx();
```
<br>

## Cache
```php
// save
$this->cache = $context->getCache();
$this->cache->save(\Zend_Json::encode($data), self::CACHE_PREFIX . $key, [], $lifeTime);

// load
$jsonStr = $this->cache->load(self::CACHE_PREFIX . $cacheKey);
if (strlen($jsonStr)) {
    $this->cachedData = \Zend_Json::decode($jsonStr);
}

// sample
$cache = $objectManager->get(\Magento\Framework\App\CacheInterface::class);
$cached = $cache->load($cacheKey);
if(strlen($cached)) {
    $data = unserialize($cached);
} else {
    $data = [];
    $product = $objectManager->create('Magento\Catalog\Model\ProductRepository')->getById($id);
    $row = new \Magento\Framework\DataObject();
    $row->setData($row->getData());
    $data []= $row;
    $cache->save(serialize($data), $cacheKey);
}

```
<br>

## 判斷是否為首頁

```php
// in action
if($this->_request->getRouteName() == 'cms' && $this->_request->getActionName() == 'index') {
    // is home
}

```
<br>

## Registry 用於內部傳遞臨時變數

```php
/* @var \Magento\Framework\Registry $coreRegistry */
$coreRegistry = $this->_objectManager->get('Magento\Framework\Registry');

// 存储值
$coreRegistry->register('current_category', $category);

// 提取值
$coreRegistry->registry('current_category');
```
<br>

## 取得當前Store
```php
$store = $objectManager->get( 'Magento\Store\Model\StoreManagerInterface' )->getStore();
```
<br>

## 提示訊息 Message

```php
// \Magento\Framework\App\Action\Action::execute()
$this->messageManager->addSuccessMessage(__('You deleted the event.'));
$this->messageManager->addErrorMessage(__('You deleted the event.'));
return $this->_redirect ( $returnUrl );
```
<br>

## log (support_report.log)

```php
\Magento\Framework\App\ObjectManager::getInstance()
    ->get('\Psr\Log\LoggerInterface' )
    ->addCritical('notice message', 
    [
        'order_id' => $order_id,
        'item_id' => $item_id
        // ...
	]);
```
<br>

## 取得當前頁面 URL

```php
$currentUrl = $objectManager->get( 'Magento\Framework\UrlInterface' )->getCurrentUrl();
```
<br>



## 取得指定路由 URL

```php
$url = $objectManager->get( 'Magento\Store\Model\StoreManagerInterface' )->getStore()->getUrl( 'xxx/xxx/xxx' );
```
<br>

## Block 裡的 URL

```php
// 首页地址
$block->getBaseUrl();
// 指定 route 地址
$block->getUrl( '[module]/[controller]/[action]' );
// 指定的静态文件地址
$block->getViewFileUrl( 'Magento_Checkout::cvv.png' );
$block->getViewFileUrl( 'images/loader-2.gif' );
```
<br>


## 取得所有 website 的位置

```php
$storeManager = $objectManager->get('Magento\Store\Model\StoreManagerInterface');
foreach($storeManager->getWebsites() as $website) {
    $store_id = $storeManager->getGroup($website->getDefaultGroupId())->getDefaultStoreId();
    $storeManager->getStore($store_id)->getBaseUrl();
}
```
<br>


## 取得 module 下的文件

```php
/** @var \Magento\Framework\Module\Dir\Reader $reader */
$reader = $objectManager->get('Magento\Framework\Module\Dir\Reader');
$reader->getModuleDir('etc', 'Infinity_Project').'/di.xml';
```
<br>


##取得資源路徑

```php
/* @var \Magento\Framework\View\Asset\Repository $asset */
$asset = $this->_objectManager->get( '\Magento\Framework\View\Asset\Repository' );
$asset->createAsset('Vendor_Module::js/script.js')->getPath();
```
<br>


## 檔案操作

```php
/* @var \Magento\Framework\Filesystem $fileSystem */
$fileSystem = $this->_objectManager->get('Magento\Framework\Filesystem');
$fileSystem->getDirectoryWrite('tmp')->copyFile($from, $to);
$fileSystem->getDirectoryWrite('tmp')->delete($file);
$fileSystem->getDirectoryWrite('tmp')->create($file);
```
<br>


## 檔案上傳
```php
// 上传到media
$request = $this->getRequest ();
if($request->isPost()) {
    try{
        /* @var $uploader \Magento\MediaStorage\Model\File\Uploader */
        $uploader = $this->_objectManager->create(
            'Magento\MediaStorage\Model\File\Uploader',
            ['fileId' => 'personal_photos']
        );
        /* @var $filesystem \Magento\Framework\Filesystem */
        $filesystem = $this->_objectManager->get( 'Magento\Framework\Filesystem' );
        $dir = $filesystem->getDirectoryRead( \Magento\Framework\App\Filesystem\DirectoryList::UPLOAD )->getAbsolutePath();
        $fileName = time().'.'.$uploader->getFileExtension();
        $uploader->save($dir, $fileName);
        $fileUrl = 'pub/media/upload/'.$fileName;
    } catch(Exception $e) {
        // 未上传
    }
}
```

```php
// 上传到tmp
/* @var \Magento\Framework\App\Filesystem\DirectoryList $directory */
$directory = $this->_objectManager->get('Magento\Framework\App\Filesystem\DirectoryList');
/* @var \Magento\Framework\File\Uploader $uploader */
$uploader = $this->_objectManager->create('Magento\Framework\File\Uploader', array('fileId' => 'file1'));
$uploader->setAllowedExtensions(array('csv'));
$uploader->setAllowRenameFiles(true);
$uploader->setFilesDispersion(true);
$result = $uploader->save($directory->getPath($directory::TMP));
$directory->getPath($directory::TMP).$result['file'];
```

<br>


## 產品縮圖
```php
$imageHelper = $objectManager->get( 'Magento\Catalog\Helper\Image' );
$productImage = $imageHelper->init( $product, 'category_page_list' )
  ->constrainOnly( FALSE )
  ->keepAspectRatio( TRUE )
  ->keepFrame( FALSE )
  ->resize( 400 )
  ->getUrl();
```
<br>


## 一般縮圖
```php
$imageFactory = $objectManager->get( 'Magento\Framework\Image\Factory' );
$imageAdapter = $imageFactory->create($path);
$imageAdapter->resize($width, $height);
$imageAdapter->save($savePath);
```
<br>


## 產品属性
```php
/* @var \Magento\Catalog\Model\ProductRepository $product */
$product = $objectManager->create('Magento\Catalog\Model\ProductRepository')->getById($id);
// print price
/* @var \Magento\Framework\Pricing\PriceCurrencyInterface $priceCurrency */
$priceCurrency = $objectManager->get('Magento\Framework\Pricing\PriceCurrencyInterface');
$priceCurrency->format($product->getData('price'));
// print option value
$product->getResource()->getAttribute('color')->getDefaultFrontendLabel();
$product->getAttributeText('color');
// all options
$this->helper('Magento\Catalog\Helper\Output')->productAttribute($product, $product->getData('color'), 'color');
$product->getResource()->getAttribute('color')->getSource()->getAllOptions();
// save attribute
$product->setWeight(1.99)->getResource()->saveAttribute($product, 'weight');
// 库存
$stockItem = $this->stockRegistry->getStockItem($productId, $product->getStore()->getWebsiteId());
$minimumQty = $stockItem->getMinSaleQty();
// Configurable Product 获取父级产品ID
$parentIds = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getParentIdsByChild($productId);
$parentId = array_shift($parentIds);
// Configurable Product 获取子级产品ID
$childIds = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getChildrenIds($parentId);
// 获取Configurable Product的子产品Collection
$collection = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getUsedProducts($product);
// 判断是否Configurable Product
if ($_product->getTypeId() == \Magento\ConfigurableProduct\Model\Product\Type\Configurable::TYPE_CODE);
```
<br>


## 取得購物車中所有商品
```php
/* @var \Magento\Checkout\Model\Session $checkoutSession */
$checkoutSession = $objectManager->get('Magento\Checkout\Model\Session');
foreach($checkoutSession->getQuote()->getItems() as $item) {
    /* @var \Magento\Quote\Model\Quote\Item $item */
    echo $item->getProduct()->getName().'<br/>';
}
```
<br>


## 取得 Product Type 配置
```php
$eavConfig->getAttribute('catalog_product', 'price');
$eavConfig->getEntityType('catalog_product');
```
<br>


## 取得 EAV 属性所有可選選項

```php
/* @var $objectManager \Magento\Framework\App\ObjectManager */
$eavConfig = $objectManager->get( '\Magento\Eav\Model\Config' );
$options = $eavConfig->getAttribute( '[entity_type]', '[attribute_code]' )
    ->getFrontend()->getSelectOptions();
//或者
/* @var $objectManager \Magento\Framework\App\ObjectManager */
$options = $objectManager->create( 'Magento\Eav\Model\Attribute' )
    ->load( '[attribute_code]', 'attribute_code' )->getSource()
    ->getAllOptions( false );
```
<br>


## 取得 config.xml 與 system.xml 裡的參數

```php
$this->_scopeConfig = $this->_objectManager->create('Magento\Framework\App\Config\ScopeConfigInterface');
// 语言代码
$this->_scopeConfig->getValue('general/locale/code', \Magento\Store\Model\ScopeInterface::SCOPE_STORE, $storeId);
```
<br>

## 客戶唯一屬性驗證

```php
if($customer instanceof \Magento\Customer\Model\Customer) {
    /* @var \Magento\Customer\Model\Attribute $attribute */
    foreach($customer->getAttributes() as $attribute) {
        if($attribute->getIsUnique()) {
            if (!$attribute->getEntity()->checkAttributeUniqueValue($attribute, $customer)) {
                $label = $attribute->getFrontend()->getLabel();
                throw new \Magento\Framework\Exception\LocalizedException(
                    __('The value of attribute "%1" must be unique.', $label)
                );
            }
        }
    }
}
```
<br>


## 讀取 design view.xml

```php
<vars module="Vendor_Module">
    <var name="var1">value1</var>
</vars>
```

```php
/* @var \Magento\Framework\Config\View $viewConfig */
$viewConfig = $objectManager->get('Magento\Framework\Config\View');
$viewConfig->getVarValue('Vendor_Module', 'var1');
```
<br>


## 取得郵件模組
##### 雖然叫郵件模組，但也可以用于需要后台编辑模板的程序

```php
// template id, 通常在email_templates.xml定义。如果是在后台加的email template，需要换成template的记录ID，例如90
$identifier = 'contact_email_email_template';
/* @var \Magento\Framework\Mail\TemplateInterface $templateFactory */
$templateFactory = $this->_objectManager->create(
    'Magento\Framework\Mail\TemplateInterface',
    ['data' => ['template_id' => $identifier]]
);
// 模板变量，取决于phtml或后台email template的内容
$dataObject = new \Magento\Framework\DataObject();
$dataObject->setData('name', 'william');
// 决定模板变量取值区域，例如像{{layout}}这样的标签，如果取不到值可以试试把area设为frontend
$templateFactory->setOptions([
    'area' => \Magento\Backend\App\Area\FrontNameResolver::AREA_CODE,
    'store' => \Magento\Store\Model\Store::DEFAULT_STORE_ID
]);
$templateFactory->setVars(['data' => $dataObject]);
return $templateFactory->processTemplate();
```
<br>


## 内容返回
```php
$this->resultFactory = $this->_objectManager->create('Magento\Framework\Controller\Result\RawFactory');
/* @var \Magento\Framework\Controller\Result\Raw $result */
$result = $this->resultFactory->create();
$result->setContents('hello world');
return $result;
```

```php
$this->resultFactory = $this->_objectManager->create('Magento\Framework\Controller\Result\JsonFactory');
/* @var \Magento\Framework\Controller\Result\Json $result */
$result = $this->resultFactory->create();
$result->setData(['message' => 'hellog world']);
return $result;
```
<br>


## HTTP文件
```php
$this->_fileFactory = $this->_objectManager->create('Magento\Framework\App\Response\Http\FileFactory');
$this->_fileFactory->create(
    'invoice' . $date . '.pdf',
    $pdf->render(),
    DirectoryList::VAR_DIR,
    'application/pdf'
);
```
<br>


## 切换貨幣與語言

```php
$currencyCode = 'GBP';
/* @var \Magento\Store\Model\Store $store */
$store = $this->_objectManager->get('Magento\Store\Model\Store');
$store->setCurrentCurrencyCode($currencyCode);

$storeCode = 'uk';
/* @var \Magento\Store\Api\StoreRepositoryInterface $storeRepository */
$storeRepository = $this->_objectManager->get('Magento\Store\Api\StoreRepositoryInterface');
/* @var \Magento\Store\Model\StoreManagerInterface $storeManager */
$storeManager = $this->_objectManager->get('Magento\Store\Model\StoreManagerInterface');
/* @var \Magento\Store\Api\StoreCookieManagerInterface $storeCookieManager */
$storeCookieManager = $this->_objectManager->get('Magento\Store\Api\StoreCookieManagerInterface');
/* @var \Magento\Framework\App\Http\Context $httpContext */
$httpContext = $this->_objectManager->get('Magento\Framework\App\Http\Context');
$defaultStoreView = $storeManager->getDefaultStoreView();
$store = $storeRepository->getActiveStoreByCode($storeCode);
$httpContext->setValue(\Magento\Store\Model\Store::ENTITY, $store->getCode(), $defaultStoreView->getCode());
$storeCookieManager->setStoreCookie($store);

$this->getResponse()->setRedirect($this->_redirect->getRedirectUrl());
```
<br>

## Profiler
```php
\Magento\Framework\Profiler::start(
    'CONFIGURABLE:' . __METHOD__,
    ['group' => 'CONFIGURABLE', 'method' => __METHOD__]
);
\Magento\Framework\Profiler::stop('CONFIGURABLE:' . __METHOD__);
```
<br>  
  
## HTML表單元素

### 日曆元件
```php
$block->getLayout()->createBlock('Magento\Framework\View\Element\Html\Date')
    ->setName('date')
    ->setId('date')
    ->setClass('date')
    ->setDateFormat('M/d/yy')
    ->setImage($block->getViewFileUrl('Magento_Theme::calendar.png'))
    ->setExtraParams('data-validate="{required:true}"')
    ->toHtml();
```
<br>


## Select 元件
```php
$block->getLayout()->createBlock('Magento\Framework\View\Element\Html\Select')
    ->setName('sel1')
    ->setId('sel1')
    ->setClass('select')
    ->addOption('value', 'label')
    ->setValue($default)
    ->setExtraParams('data-validate="{required:true}"')
    ->toHtml();
```
<br>


## 取得當前 Category
```php
 $registry = $objectManager->get('Magento\Framework\Registry');

 $currentCatetory = $registry->registry('current_category');

 echo $catId = $currentCatetory->getId();
```
<br>

## 使用資料Select 

```php
$collection = $this->_objectManager->get( 'Magento\Catalog\Model\ProductFactory' )->create()->getCollection();
echo $collection->load()->getSelectSql( true );
```

<br>

## 使用 Select 取得數據

```php
$conn = $this->_objectManager->get( 'Magento\Framework\App\ResourceConnection' )->getConnection();
$tblMain = $conn->getTableName( 'cms_page' );
$tblStore = $conn->getTableName( 'cms_page_store' );

$select = $conn->select()
        ->distinct()
        ->from( [ 'page' => $tblMain ], [ 'page_id', 't' => 'title' ] )
        ->join( [ 'store' => $tblStore ], 'store.page_id = page.page_id', [ 'sid' => 'store_id' ] )
        ->where( 'is_active = ?', '1' )
        ->order( 'title ASC' )
        ->limit( 5, 1 );

$data = $conn->fetchAll( $select );
```

<br>

## 聚合查询（SUM） 

```php
$conn = $this->_objectManager->get( 'Magento\Framework\App\ResourceConnection' )->getConnection();
$tbl = $conn->getTableName( 'sales_order_item' );

// 所有记录总计
$select = $conn->select()
        ->from( $tbl, [ 'total' => new \Zend_Db_Expr( 'SUM( qty_ordered )' ) ] )
        ->where( 'order_id = ?', '1' );
$result = $conn->fetchOne( $select );

// 各局部统计
$select = $conn->select()
        ->from( $tbl, [ 'order_id', 'total' => new \Zend_Db_Expr( 'SUM( qty_ordered )' ) ] )
        ->group( 'order_id' )
        ->having( 'order_id != ?', '1' );
$result = $conn->fetchAll( $select );
```

<br>

## 更新数据

```php
/** @var $conn \Magento\Framework\App\ResourceConnection */
/** @var $storeId int */
/** @var $ids array */

$tbl = $conn->getTableName( 'sales_order_item' );

// 插入数据
$conn->insert( $tbl, [ 'field_one' => $fieldOne, 'field_two' => $fieldTwo, 'field_three' => $fieldThree ] );

// 插入数据，碰到已存在的记录（primary / unique）则只更新指定的字段
$conn->insertOnDuplicate( $tbl, [ 'field_one' => $fieldOne, 'field_two' => $fieldTwo, 'field_three' => $fieldThree ], [ 'field_three' ] );

// 更新数据
$conn->update( $tbl, [ 'field_one' => $fieldOne, 'field_two' => $fieldTwo ], $conn->quoteInto( 'store_id IN (?)', $storeId ) );

// 删除数据
$conn->delete( $tbl, [ 'id IN (?)' => $ids ] );
```

<br>

## Model

```php
/* @var \Magento\Directory\Model\ResourceModel\Region\Collection $collection */
$collection = $objectManager->get('Magento\Directory\Model\ResourceModel\Region\Collection');
$collection->getSelect()
    ->where('main_table.region_id = ?', 1)
    ->where('main_table.country_id=?', 'US');
foreach($collection as $row) {
    echo $row->getData('default_name');
}
```

<br>
# Entity collection
## 常用方法
```php
$collection->addAttributeToFilter($field, [ 'in' => $arr ]);
$collection->addAttributeToSort($field, 'desc');
$collection->addStoreFilter();
$collection->addUrlRewrite();
$collection->setPageSize( 10 );
$collection->addAttributeToSelect($field);
```
<br>

# 包含 field 的查询
#### Entity collection有查询field的能力，只要查询条件里跟field有关，就会自动把field所在的表join到查询中

```php
$collection->addAttributeToSelect($field, 'left'); // 等同于select xx as field ... left join xxx
$collection->addAttributeToFilter($field, 'field > 0')// 等同于where field > 0
$collection->setOrder($field, 'ASC'); // 等同于order by xxx ASC
```

<br>


# 特價置頂
#### Special Price是個特别的個案，需要在有效期間生效，有效期外不生效

```php
$collection->addAttributeToSelect('special_from_date', 'left');
$collection->addAttributeToSelect('special_to_date', 'left');
$collection->addAttributeToSelect('special_price', 'left');
$collection->getSelect()->order('IF(special_from_date < now() AND special_to_date > now(), special_price, 0) DESC');
```
<br>
### 參考文件：
[Segmentfault.com](https://segmentfault.com/a/1190000005154774#articleHeader31)

[magento-2-create-custom-query](http://www.webmull.com/magento-2-create-custom-query/)
