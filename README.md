>在项目开发中，由于前后端通常要一起开工，作为前端的我们许多工作需要在后台返回的基础上进行，因此，大部分时间，编写完成的代码看不到实在的效果，这就导致在后台完成工作之前前端无法进行很好的调试。因此，项目的进度在一定程度上来说使前后台的串行工作，而不是并行。然而mock.js可以帮我们很好地解决这个问题。[Mockjs官网](http://mockjs.com/)

#### mock使用教程
#####数据模板定义
'name|rule': value name 为属性名, rule 为规则, value 为值，属性名和生成规则之间用|分隔，生成规则的格式有7种：
- 'name|min-max': value
- 'name|count': value
- 'name|min-max.dmin-dmax': value //.dmin-dmax 小数点后保留的位数范围
- 'name|min-max.dcount': value //小数点后保留dcount位
- 'name|count.dmin-dmax': value
- 'name|+step': value //从value递增/减

属性值可以包含占位符(如@name)，属性值指定了最终值的初始值和数据类型.

属性值是字符串 String
- 'name|min-max': 'value' 通过重复 'value' 生成一个字符串，重复次数大于等于 min，小于等于 max。
- 'name|count': 'value' 通过重复 'value' 生成一个字符串，重复次数等于 count。
属性值是数字 Number
- 'name|+1': 100 属性值自动加 1，初始值为 100
- 'name|1-100': 100 生成一个大于等于 1、小于等于 100 的整数，属性值 100 只用来确定类型。
- 'name|1-100.1-10': 100 生成一个浮点数，整数部分大于等于 1、小于等于 100，小数部分保留 1 到 10 位。

属性值是布尔型 Boolean
- 'name|1': value 随机生成一个布尔值，值为 true 的概率是 1/2，值为 false 的概率是 1/2。
- 'name|min-max': value 随机生成一个布尔值，值为 value 的概率是 min / (min + max)，值为 !value 的概率是 max / (min + max)。

属性值是对象 Object
- 'name|min-max': {} 从属性值 {} 中随机选取 min 到 max 个属性。
- 'name|count': {} 从属性值 {} 中随机选取 count 个属性。

属性值是数组 Array
- 'name|1': [{}, {} ...] 从属性值 [{}, {} ...] 中随机选取 1 个元素，作为最终值。
- 'name|min-max': [{}, {} ...] 通过重复数组中的元素值 [{}, {} ...] 生成一个新数组，重复次数大于等于 min，小于等于 max。
- 'name|count': [{}, {} ...] 通过重复数组中的元素值 [{}, {} ...] 

生成一个新数组，重复次数为 count。
属性值是数组 Function
- 'name': function(){} 执行函数 function(){}，取其返回值作为最终的属性值，上下文为 
- 'name' 所在的对象。

数据占位符定义
占位符 只是在属性值字符串中占个位置，并不出现在最终的属性值中。占位符 的格式为：
- @占位符
- @占位符(参数 [, 参数])
- 用 @ 来标识其后的字符串是 占位符。
- 占位符 引用的是 Mock.Random 中的方法。
- 通过 Mock.Random.extend() 来扩展自定义占位符。
- 占位符 也可以引用 数据模板 中的属性。
- 占位符 会优先引用 数据模板 中的属性

#####常用方法
1.Mock.mock( rurl?, rtype?, template|function(options) )根据数据模板生成模拟数据。
- rurl：可选。表示需要拦截的 URL，可以是 URL 字符串或 URL 正则。例如 //domain/list.json/、'/domian/list.json'。
- rtype：可选。表示需要拦截的 Ajax 请求类型。例如 GET、POST、PUT、DELETE 等。
- template：可选。表示数据模板，可以是对象或字符串。例如 { 'data|1-10':[{}] }、'@EMAIL'。
- function(options)：可选。表示用于生成响应数据的函数。
- options：指向本次请求的 Ajax 选项集。

2.Mock.mockjax(library) 覆盖（拦截） Ajax 请求，目前内置支持 jQuery、Zepto、KISSY。
3.Mock.Random Mock.Random 是一个工具类，用于生成各种随机数据。Mock.Random 的方法在数据模板中称为“占位符”，引用格式为 @占位符(参数 [, 参数]) 。
4.Mock.tpl(input, options, helpers, partials) 基于 Handlebars、Mustache 的 HTML 模板生成模拟数据。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>mockjs-demo</title>
</head>

<body>
    <h1>Hello World!</h1>
    <script src="require.js"></script>
    <script type="text/javascript">
    require.config({
        paths: {
            "jquery": "node_modules/jquery/dist/jquery.min",
            "mockjs": "node_modules/mockjs/dist/mock-min",
        }
    });
    // 使用 Mock
    require(['mockjs', 'jquery'], function(Mock, $) {　　　　
        Mock.mock('http://123.com', {
            // 'name': "MarcoHan",
            name: {
                first: '@FIRST', 
                middle: '@FIRST',
                last: '@LAST',
                full: '@first @middle @last'
            },
            'age|1-100': 100,
            'color': '@COLOR',  //@占位符实际引用的是Mock.Random里面的方法，具体查看API
            // 'color': Mock.Random.color(),  
            // 属性 list 的值是一个数组，其中含有 1 到 10 个元素
            'list|1-10': [{
                // 属性 id 是一个自增数，起始值为 1，每次增 1
                'id|+1': 1
            }]
        });

        // // 输出结果
        // console.log(JSON.stringify(data, null, 4))

        $.ajax({
            url: 'http://123.com'
        }).done(function(data, status, xhr) {
            console.dir(data);
        })
    });
    </script>
</body>

</html>
```

- 参考文献
1、[利用mock生成假数据](https://github.com/Marco2333/react-demo/tree/master/demo/demo05%20mock)