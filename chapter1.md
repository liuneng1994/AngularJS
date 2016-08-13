# 基本概念

| 概念 | 描述 |
| --- | --- |
| Template\(模板\) | 带有额外标记的HTML |
| Directives\(指令\) | 扩展HTML的自定义属性和元素 |
| Model\(模型\) | 展示在视图中且与用户交互的数据 |
| Scope\(域\) | 存储在控制器中的上下文，可以通过指令和表达式访问 |
| Expressions\(表达式\) | 从**scope**中访问变量和函数 |
| Compiler\(编译器\) | 解析模板并且实例化指令和表达式 |
| Filter\(过滤器\) | 格式化表达式的值 |
| View\(视图\) | 用户所看到的东西\(DOM\) |
| Data Binding\(数据绑定\) | 同步视图与模型之间的数据 |
| Controller\(控制器\) | 视图之后的业务逻辑 |
| Dependency Injection\(依赖注入\) | 创建并注入对象和函数 |
| Injector\(注射器\) | 依赖注入容器 |
| Module\(模块\) | 为应用程序的不同部分存储通过注射器设置的控制器，服务，过滤器，指令 |
| Service\(服务\) | 可重用的独立于视图的业务逻辑 |

## 数据绑定示例

```
<div ng-app ng-init="qty=1;cost=2">
  <b>Invoice:</b>
  <div>
    Quantity: <input type="number" min="0" ng-model="qty">
  </div>
  <div>
    Costs: <input type="number" min="0" ng-model="cost">
  </div>
  <div>
    <b>Total:</b> {{qty * cost | currency}}
  </div>
</div>
```

    上面的例子看上去就像带有一些新标记的普通的HTML.在Angular中,一个这样的文件就叫做模板.当Angular启动你的应用程序时,会通过编译器去解析和处理模板中的新标记.这个被加载,和渲染之后的DOM就称作视图

    第一类新标记称作指令\(directives\).他们为HTML的元素或属性提供特殊的行为.在上面的例子中,我们使用`ng-app`属性,它是一个用于自动初始化应用程序的指令.Angular也定义了一个为`input`元素添加其他行为的指令,`ng-model`指令可以像变量中存储值或从变量更新值.

> **自定义指令访问DOM**:在Angular中,应用程序只应该在指令中访问DOM.这一点很重要,因为访问DOM的部分难于测试.如果你需要直接访问DOM,应该写一个自定义的指令

    第二种标记是双花括号`{{ expression | filter}}`:当编译器遇到这个标记,标记会被替换为对应表达式的值.在模板中的表达式是一个类似JavaScript的代码片段,表达式允许Angular读写变量.注意那些变量不是全局变量,更像是在JavaScript函数作用域中的局部变量.Angular提供一个域\(scope\)给表达式访问.这些存储在域中的变量的值引用自接下来会提到的_model_中.

    在上面的例子中包含了一个过滤器.过滤器可以格式化表达式所显示的值.在上面例子中的`currency`过滤器能将一个数字格式化成金额的形式,类似`$1111`.

    在上面的例子中,Angular提供了动态绑定:无论何时输入的值改变,表达式的值也会自动重新计算并且DOM也会随之更新.这就是之后会提到的_双向数据绑定_.

---

## 增加UI的逻辑:Controllers

**index.html**

```
<div ng-app="invoice1" ng-controller="InvoiceController as invoice">
  <b>Invoice:</b>
  <div>
    Quantity: <input type="number" min="0" ng-model="invoice.qty" required >
  </div>
  <div>
    Costs: <input type="number" min="0" ng-model="invoice.cost" required >
    <select ng-model="invoice.inCurr">
      <option ng-repeat="c in invoice.currencies">{{c}}</option>
    </select>
  </div>
  <div>
    <b>Total:</b>
    <span ng-repeat="c in invoice.currencies">
      {{invoice.total(c) | currency:c}}
    </span>
    <button class="btn" ng-click="invoice.pay()">Pay</button>
  </div>
</div>
```

**Invocie1.js**

```

  this.cost = 2;
  this.inCurr = 'EUR';
  this.currencies = ['USD', 'EUR', 'CNY'];
  this.usdToForeignRates = {
    USD: 1,
    EUR: 0.74,
    CNY: 6.09
  };

  this.total = function total(outCurr) {
    return this.convertCurrency(this.qty * this.cost, this.inCurr, outCurr);
  };
  this.convertCurrency = function convertCurrency(amount, inCurr, outCurr) {
    return amount * this.usdToForeignRates[outCurr] / this.usdToForeignRates[inCurr];
  };
  this.pay = function pay() {
    window.alert('Thanks!');
  };
});

```

尝试运行上面的代码,看发生了哪些变化.

首先,在一个新的Javascript文件中包含了一个控制器.更准确的说,这个文件包含了一个创建控制器实例的构造函数.控制器的目的是为了提供变量和函数给表达式和指令.

除了新文件中包含的控制器,我们还在HTML中增加了一个`ng-controller`指令.这个指令告诉angular这个新的`InvoiceController`控制器对这个元素的指令和它的所有子元素负责.`InvoiceController as invoice`语法告诉Angular实例化这个控制器并且保存在当前作用域的`invoice`变量中.

我们也可以使得在这个页面中的所有表达式不通过控制器实例而是`invoice.`前缀去读写变量.`currencies`被定义在控制器中,并且通过`ng-repeat`指令添加到模板中.因为控制器包含一个`total`函数,所以我们也能绑定`{{invoice.total(...)}}`的值到DOM上.

再次说明这个绑定是动态的,DOM会在函数结果发生改变时自动更新值.这个按钮使用了`ng-click`指令.当按钮被按下时,表达式的值就会被计算出来.

在新的JavaScript文件中,我们也可以创建一个模块\(module\)去注册控制器.在下一节会讨论模块.

接下来的图片展示了控制器,域和视图的工作机制.
![原理图](/assets/concepts-databinding2.png)

---

## 独立于视图的业务逻辑:服务\(Services\)

现在,
接下来的例子会将金钱的转换移动到服务中.
**finance2.js**

```
angular.module('finance2', [])
.factory('currencyConverter', function() {
  var currencies = ['USD', 'EUR', 'CNY'];
  var usdToForeignRates = {
    USD: 1,
    EUR: 0.74,
    CNY: 6.09
  };
  var convert = function(amount, inCurr, outCurr) {
    return amount * usdToForeignRates[outCurr] / usdToForeignRates[inCurr];
  };

  return {
    currencies: currencies,
    convert: convert
  };
});
```

**invoice2.js**

```
angular.module('invoice2', ['finance2'])
 .controller('InvoiceController', ['currencyConverter',
function InvoiceController(currencyConverter) {
 this.qty = 1;
 this.cost = 2;
 this.inCurr = 'EUR';
 this.currencies = currencyConverter.currencies;
 this.total = function total(outCurr) {
 return currencyConverter.convert(this.qty * this.cost, this.inCurr, outCurr); };
 this.pay = function pay() {
 window.alert('Thanks!');
 };
}]);
```

**index.html**

```
<div ng-app="invoice2" ng-controller="InvoiceController as invoice">
  <b>Invoice:</b>
  <div>
    Quantity: <input type="number" min="0" ng-model="invoice.qty" required >
  </div>
  <div>
    Costs: <input type="number" min="0" ng-model="invoice.cost" required >
    <select ng-model="invoice.inCurr">
      <option ng-repeat="c in invoice.currencies">{{c}}</option>
    </select>
  </div>
  <div>
    <b>Total:</b>
    <span ng-repeat="c in invoice.currencies">
      {{invoice.total(c) | currency:c}}
    </span>
    <button class="btn" ng-click="invoice.pay()">Pay</button>
  </div>
</div>
```

![](/assets/concepts-module-service.png)
我们把`convertCurrency`函数和现存的`currencies`定义移动到了新的`finance2.js`中.但是控制器要怎么样才能获得一个分离的函数呢.

这时候依赖注入就要登场了,依赖注入\(DI\)是一种软件设计模式,能够处理对象和函数如何创建,他们怎么获得他们的依赖.在Angular中的所有东西\(指令,过滤器,控制器,服务...\)都是通过依赖注入进行创建和注入的.在Angular中,依赖注入容器被称作注射器\(injector\).

为了使用DI,需要一个地方去注册那些一起工作的对象或函数.在Angular中,模块就是起这个作用.当Angular启动时,它会使用通过`ng-app`定义的模块配置和这个模块所依赖的所有模块的配置.

在上述例子中:模板内包含了`ng-app="invoice2"`.这意味着Angular会使用`invoice2`这个模块作为应用程序的主模块.代码片段`angular.module('invoice2', ['finance2'])`定义了`invoice2`模块依赖`finance2`模块.类似这种方式,Angular也可以让`InvoiceController`使用`currencyConverter`服务.

现在Angular知道了应用程序的所有组成部分,它需要去创建它们.在前一节我们看到了控制器是通过一个工厂函数去创建.对于服务来说也有多种途径去定义它们的工厂.在上面的例子中,我们使用了一个匿名函数作为`currencyConverter`的工厂函数.这个函数应该返回一个`currencyConverter`实例.

回到最初的问题:`InvoiceController`是如何获得`currencyConverter`函数的引用的?在Angular中,只需要简单的在构造函数中定义参数即可.之后,注射器能够按正确的顺序创建所需的对象并注入到依赖它们的工厂中.在我们的例子中,`InvoiceController`有一个`currencyConverter`参数.因此,Angular能够知道控制器与服务之间的依赖,并且使用服务对象作为对象去调用控制器.

最后的一个变化是,我们传递了一个数组给`module.controller`函数而不是一个纯粹的函数,这个数组首先包含了这个控制器需要的服务的名字.最后一项是控制器的贡枣函数.Angular使用数组的语法去定义依赖.是的在压缩代码之后依然可以使用DI,代码压缩可能会重命名控制器构造函数的参数名称成一个更短的,如`a`.

---

##访问服务器
接下来的例子完成从雅虎金融APi获取汇率的功能.
**invoice3.js**
```
angular.module('invoice3', ['finance3'])
.controller('InvoiceController', ['currencyConverter', function InvoiceController(currencyConverter) {
  this.qty = 1;
  this.cost = 2;
  this.inCurr = 'EUR';
  this.currencies = currencyConverter.currencies;

  this.total = function total(outCurr) {
    return currencyConverter.convert(this.qty * this.cost, this.inCurr, outCurr);
  };
  this.pay = function pay() {
    window.alert('Thanks!');
  };
}]);
```
**finance3.js**
```
angular.module('finance3', [])
.factory('currencyConverter', ['$http', function($http) {
  var YAHOO_FINANCE_URL_PATTERN =
        '//query.yahooapis.com/v1/public/yql?q=select * from ' +
        'yahoo.finance.xchange where pair in ("PAIRS")&format=json&' +
        'env=store://datatables.org/alltableswithkeys&callback=JSON_CALLBACK';
  var currencies = ['USD', 'EUR', 'CNY'];
  var usdToForeignRates = {};

  var convert = function(amount, inCurr, outCurr) {
    return amount * usdToForeignRates[outCurr] / usdToForeignRates[inCurr];
  };

  var refresh = function() {
    var url = YAHOO_FINANCE_URL_PATTERN.
               replace('PAIRS', 'USD' + currencies.join('","USD'));
    return $http.jsonp(url).then(function(response) {
      var newUsdToForeignRates = {};
      angular.forEach(response.data.query.results.rate, function(rate) {
        var currency = rate.id.substring(3,6);
        newUsdToForeignRates[currency] = window.parseFloat(rate.Rate);
      });
      usdToForeignRates = newUsdToForeignRates;
    });
  };

  refresh();

  return {
    currencies: currencies,
    convert: convert
  };
}]);
```
**index.html**
```
<div ng-app="invoice3" ng-controller="InvoiceController as invoice">
  <b>Invoice:</b>
  <div>
    Quantity: <input type="number" min="0" ng-model="invoice.qty" required >
  </div>
  <div>
    Costs: <input type="number" min="0" ng-model="invoice.cost" required >
    <select ng-model="invoice.inCurr">
      <option ng-repeat="c in invoice.currencies">{{c}}</option>
    </select>
  </div>
  <div>
    <b>Total:</b>
    <span ng-repeat="c in invoice.currencies">
      {{invoice.total(c) | currency:c}}
    </span>
    <button class="btn" ng-click="invoice.pay()">Pay</button>
  </div>
</div>
```
我们`finance`模块的`currencyConverter`服务现在使用了`$http`.一个由Angular提供的访问后台服务器的内建的服务.`$http`

