# 基本概念

|概念  |   描述|
|-----|------|
|Template(模板)|带有额外标记的HTML |
|Directives(指令)|扩展HTML的自定义属性和元素|
|Model(模型)|展示在视图中且与用户交互的数据|
|Scope(域)|存储在控制器中的上下文，可以通过指令和表达式访问|
|Expressions(表达式)|从**scope**中访问变量和函数|
|Compiler(编译器)|解析模板并且实例化指令和表达式|
|Filter(过滤器)|格式化表达式的值|
|View(视图)|用户所看到的东西(DOM)|
|Data Binding(数据绑定)|同步视图与模型之间的数据|
|Controller(控制器)|视图之后的业务逻辑|
|Dependency Injection(依赖注入)|创建并注入对象和函数|
|Injector(注射器)|依赖注入容器|
|Module(模块)|为应用程序的不同部分存储通过注射器设置的控制器，服务，过滤器，指令|
|Service(服务)|可重用的独立于视图的业务逻辑|
##数据绑定示例
` <div ng-app ng-init="qty=1;cost=2">
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
</div> `