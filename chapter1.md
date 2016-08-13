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

&nbsp;&nbsp;&nbsp;&nbsp;第二种标记是双花括号`{{ expression | filter}}`:当编译器遇到这个标记,标记会被替换为对应表达式的值.在模板中的表达式是一个类似JavaScript的代码片段,表达式允许Angular读写变量.注意那些变量不是全局变量,更像是在JavaScript函数作用域中的局部变量.Angular提供一个域(scope)给表达式访问.这些存储在域中的变量的值引用自接下来会提到的*model*中.

&nbsp;&nbsp&nbsp;&nbsp;在上面的例子中包含了一个过滤器.过滤器可以格式化表达式所显示的值.在上面例子中的`currency`过滤器能将一个数字格式化成金额的形式,类似`$1111`.

&nbsp;&nbsp;&nbsp&nbsp在上面的例子中,Angular提供了动态绑定:无论何时输入的值改变,表达式的值也会自动重新计算并且DOM也会随之更新.这就是之后会提到的*双向数据绑定*.

---
##增加UI的逻辑:Controllers
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

在新的JavaScript文件中,我们也可以创建一个模块(module)去注册控制器.在下一节会讨论模块.

接下来的图片展示了控制器,域和视图的工作机制.
![原理图](/assets/concepts-databinding2.png)
