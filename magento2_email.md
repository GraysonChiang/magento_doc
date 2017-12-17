# Magento2 郵件相關

## 定義郵件模板 
##### <Module>/etc/email_templates.xml

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Email:etc/email_templates.xsd">
    <template id="module_hello_email_template" label="Test Mail" file="hello.html" type="html" module="Infinity_Base" area="frontend"/>
</config>
```
<br>

## 建立郵件模組 (<Module>/view/frontend/email/hello.html)

```html
<!--@subject {{trans "Welcome to %store_name" store_name=$store.getFrontendName()}} @-->
<!--@vars {
"var this.getUrl($store, 'customer/account/')":"Customer Account URL",
"var customer.email":"Customer Email",
"var customer.name":"Customer Name"
} @-->
{{template config_path="design/email/header_template"}}

<p class="greeting">Hello {{trans "%name," name=$name}}</p>

{{template config_path="design/email/footer_template"}}
```

<br>

## 寄送郵件
```php
$templateId = 'module_hello_email_template';
$templateParams = ['name' => 'william'];

$objectManager = \Magento\Framework\App\ObjectManager::getInstance();
/* @var \Magento\Framework\Mail\Template\TransportBuilder $transportBuilder */
$transportBuilder = $objectManager->get('Magento\Framework\Mail\Template\TransportBuilder');
/* @var \Magento\Store\Model\StoreManagerInterface $storeManager */
$storeManager = $objectManager->get( 'Magento\Store\Model\StoreManagerInterface' );
/* @var \Magento\Customer\Helper\Session\CurrentCustomer $customer */
$customer = $objectManager->get( '\Magento\Customer\Helper\Session\CurrentCustomer' );

$transportBuilder->setTemplateIdentifier($templateId)
    ->setTemplateOptions(['area' => 'frontend', 'store' => $storeManager->getStore()->getId()])
    ->setTemplateVars($templateParams)
    ->setFrom('general') // sales,support,custom1,custom2
    ->addTo($customer->getCustomer()->getEmail(), $customer->getCustomer()->getFirstname())
    ->getTransport()
    ->sendMessage();
```
<br>

## 取得郵件模組

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

## 模組變數
```php
vendor/magento/module-sales/Model/Order/Email/Sender/*.php
vendor/magento/module-customer/Model/EmailNotification.php
vendor/magento/module-send-friend/Model/SendFriend.php
```

<br>
### 參考資料：

[Segmentfault.com](https://segmentfault.com/a/1190000005590715)

