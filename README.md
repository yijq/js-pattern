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

_takashi.seeMonster    //true

```