# UnionPay - 银联支付
simplest union pay - no dependency to any library, simple enough to let you hack.

support:
* [B2C - Web网关支付](src/service/B2C.php)
* [Wap - WAP/手机网页网关支付](src/service/Wap.php)
* [App - App/控件支付](src/service/App.php)
* [B2B - 企业支付](src/service/B2B.php)
* [Direct - 无跳转标准版](src/service/Direct.php)
* [DirectToken - 无跳转Token版](src/service/DirectToken.php)
* [Qrcode - 二维码支付](src/service/Qrcode.php)
* [DirectDebit - 代收](src/service/DirectDebit.php)
* [Charge - 缴费产品](src/service/Charge.php)

### Step 1: config.php - 配置

```php
<?php
return ['test', [
		'version' => '5.1.0',
		'signMethod'=> '01', //RSA
		'encoding' => 'UTF-8',
		'merId' => '700000000000001',
		'returnUrl' => 'http://dev.git.com/union-pay/demo/payreturn.php', //前台网关支付返回
		'notifyUrl' => 'http://dev.git.com/demo/paynotify.php', //后台通知
		'frontFailUrl'   => 'http://dev.git.com/union-pay/demo/payfail.php',
		'refundnotifyUrl' => 'http://dev.git.com.com/demo/refundnotify.php',
		'signCertPath' => dirname(__FILE__).'/cert/acp_test_sign.pfx',
		'signCertPwd' => '000000', //签名证书密码
		'verifyCertPath' => dirname(__FILE__).'/cert/5.0.0/acp_test_verify_sign.cer',  //v5.0.0 required NOTE:该测试环境证书已失效，推荐使用5.1.0
		'verifyRootCertPath' => dirname(__FILE__).'/cert/5.1.0/acp_test_root.cer', //v5.1.0 required
		'verifyMiddleCertPath' => dirname(__FILE__).'/cert/5.1.0/acp_test_middle.cer', //v5.1.0 required
		'encryptCertPath' => dirname(__FILE__).'/cert/acp_test_enc.cer',
		'ifValidateCNName' => false, //正式环境设置为true
	]
];
```


### Step 2: pay.php - 支付

```php
<?php
require_once __DIR__ . "/autoload.php";
use zhangv\unionpay\UnionPay;

list($mode,$config) = include './config.php';
$unionPay = UnionPay::B2C($config,$mode);

$payOrderNo = date('YmdHis');
$sum = 1;
$desc = 'desc';

$html = $unionPay->pay($payOrderNo,$sum,$desc,'');
echo $html;
```

### Step 3: payreturn.php - 支付完成前台返回

```php
<?php
require_once __DIR__ . "/autoload.php";
use zhangv\unionpay\UnionPay;

list($mode,$config) = include './config.php';
$unionPay = UnionPay::B2C($config,$mode);

$postdata = $_REQUEST;
$unionPay->onPayNotify($postdata,function($notifydata){
	echo 'SUCCESS';
	var_dump($notifydata);
});
```

### Step 4: paynotify.php - 支付完成后台通知
```php
<?php
require_once __DIR__ . "/autoload.php";
use zhangv\unionpay\UnionPay;
list($mode,$config) = include './config.php';
$unionPay = UnionPay::B2C($config,$mode);

$notifyData = $_POST;
$respCode = $notifyData['respCode'];
if($respCode == '00'){
	$txnType = $notifyData['txnType'];
	if($txnType == UnionPay::TXNTYPE_CONSUME){
		$unionPay->onPayNotify($notifyData,'demoCallback');
	}elseif($txnType == UnionPay::TXNTYPE_CONSUMEUNDO){
		$unionPay->onPayUndoNotify($notifyData,'demoCallback');
	}elseif($txnType == UnionPay::TXNTYPE_REFUND){
		$unionPay->onRefundNotify($notifyData,'demoCallback');
	}else echo 'fail';
}elseif(in_array($respCode,['03','04','05'])){
	//后续需发起交易状态查询交易确定交易状态
}else{
	echo 'fail';
}


function demoCallback($notifyData){//自定义回调
	var_dump($notifyData);
	print('ok');
}
```