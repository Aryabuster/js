#js function

**hasOwnProperty** 是用来判断一个对象是否有你给出名称的属性或对象。不过需要注意的是，此方法无法检查该对象的原型链中是否具有该属性，该属性必须是对象本身的一个成员。格式:

Js代码 

```
    1. object.hasOwnProperty(proName); 
```

判断proName的名称是不是object对象的一个属性或对象。

- 示例一：

```
var bStr = "Test String".hasOwnProperty("split");   
// 得到false， 因为不能检测原型链中的属性 
"Test String".split(" ");是能成功调用的
```
- 示例二：

```
var bStr1 = String.prototype.hasOwnProperty("split"); //String对象的原型上本来就有这个属性,自然返回true  
```
- 示例三


```
var bObj = ({fnTest:function(){}}).hasOwnProperty("fnTest"); 
// 返回true，因为对象中属性 存在
```

**JavaScript prototype 属性**

prototype 属性使您有能力向对象添加属性和方法。

```
object.prototype.name=value
```

在本例中，我们将展示如何使用 prototype 属性来向对象添加属性：

```
<script type="text/javascript">

function employee(name,job,born)
{
this.name=name;
this.job=job;
this.born=born;
}

var bill=new employee("Bill Gates","Engineer",1985);

employee.prototype.salary=null;
bill.salary=20000;

document.write(bill.salary);

</script>
```
























 