# JavaScript设计模式与开发实践

## 原型模式
    原型模式是一种设计模式，也是一种变成泛型，它构成了JavaScriptS这门语言的根本。动态类型语言和鸭子类型。
- 多态
    - 同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。
- 封装 
    - 将信息隐藏。封装数据，封装实现，封装类型，封装变化。
- JavaScript中的原型继承
    - 所有的数据都是对象
    - 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它
    - 对象会记住它的原型
    - 如果对象无法响应某个请求，它会把这个请求委托给**它的构造器的原型**
- 代码：
```JavaScript
//JavaScript中的多态
var takashi = {
    seeMonster: function() {
        console.log("还名字给妖怪");
    }
};

var reiko = {
    seeMonster: function() {
        console.log("决斗，夺妖怪名字")
    }
};

var action = function(natsume) {
    if(natsume.seeMonster instanceof Function) {
        natsume.seeMonster();
    }
};

action(takashi);   //还名字给妖怪
action(reiko);     //决斗，夺妖怪名字

//JavaScript实现原型的继承
var Reiko = function() {};
var Takashi = function() {}; //Takashi继承了Reiko的看见妖怪的能力

Reiko.prototype = {
    seeMonster: true
};

Takashi.prototype = new Reiko();

var _takashi = new Takashi();

console.log(_takashi.seeMonster);    //true

```

## this和call及apply
- this
    - this的指向
        - 作为对象的方法调用，指向该对象
        - 作为普通函数调用，指向全局对象，严格模式下为undefined
        - 构造器调用，指向构造器返回的对象
        - Function.prototype.call或Function.prototype.apply调用
        - 不常用的with和eval
- call和apply
    - call和apply的区别
        - call和apply作用一模一样，区别仅在于传入参数形式的不同
        - apply接受两个参数，第一个参数指定了函数体内this对象的指向，第二个参数为一个带下标的集合，这个集合可以是数组，也可以是类数组，apply方法把这个集合中的元素作为参数传递给被调用的函数
        - call传入的参数数量不固定，第一个参数也是代表函数体内的this指向，从第二个参数开始往后，每个参数被依次传入函数
        - 从内部实现来说，apply比call的效率高
    - call和apply的用途
        - 改变this的指向
        - Function.prototype.bind
        - 借用其他对象的方法
- 代码：
```html
<!-- 这是一个关于this作为普通函数调用时的例子，这时this指向‘window’ -->
<html>
    <body>
        <div id="div1">我是一个div</div>
    </body>
    <script>
        window.id = 'window';

        document.getElementById('div1').onclick = function() {
            alert(this.id);    //输出： "div1"
            var callback = function() {
                alert(this.id);    //输出： "window"
            };
            callback();
        };

    </script>
</html>

```
```javascript
//实现Function.prototype.bind
Function.prototype.bind = function( context ) {
    var self = this;
    return function() {
        return self.apply( context, arguments );
    }
};
```