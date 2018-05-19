# JavaScript设计模式与开发实践

----

## 原型模式
    原型模式是一种设计模式，也是一种变成泛型，它构成了JavaScript这门语言的根本。动态类型语言和鸭子类型。
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

## 闭包和高阶函数
- 闭包
    - 变量的作用域
    - 变量的生命周期
        - 循环为DOM节点绑定事件
    - 更多作用：
        - 封装变量
        ```javascript
             /*阶乘函数*/
            var mult = (function() {
                var cache = {};/*结果缓存*/
                var calculate = function() {
                    var a = 1;
                    for ( var i = 0, l = arguments.length; i < l; i++ ) {
                        a = a * arguments[i];
                    }

                    return a;
                };

                return function() {
                    var args = Array.prototype.join.call( arguments, "," );
                    if( args in cache ) {
                        return cacge[ args ];
                    }
                    return cache[ args ] = calculate.apply( null, arguments );
                };
            })();
        ```
        - 延续局部变量的寿命
        ```javascript
            /*img进行数据上报,使用闭包防止Image对象被销毁,以解决请求丢失的问题*/
            var report = (function() {
                var imgs = [];
                return function( src ) {
                    var img = new Image();
                    imgs.push( img );
                    img.src = src;
                }
            })();
        ```
    - 闭包和面向对象设计
        ```javascript
            var extent = function() {
                var value = 0;
                return {
                    call: function() {
                        value++ ;
                        console.log(value)
                    }
                }
            };

            var _extent = extent();

            _extent.call()  /* 1 */
            _extent.call()  /* 2 */
            _extent.call()  /* 3 */
        ```
    - 用闭包实现命令模式
    - 闭包与内存管理
        - 闭包和内存泄漏有关系的地方，使用闭包的同时比较容易形成循环引用，如果闭包的作用域中保存着一些DOM节点，这时就有可能造成内存泄漏。这本身并非闭包的问题，也并非JavaScript的问题。
- 高阶函数
    - 函数作为参数传递
        - 回调函数
            - ajax回调函数
        - Array.prototype.sort
    - 函数作为返回值输出
        - 判断数据的类型
        ```javascript
        var Type = {};

        for (var i = 0, type; type = [ "String", "Array", "Number" ][ i++ ];) {
            (function(type){
                Type['is' + type] = function( obj ) {
                    return Object.prototype.toString.call( obj ) === '[object '+ type +']';
                }; 
            } )(type)
        }



        ```
        - getSingle
        ```javascript
        var getSingle = function( fn ) {
            var ret;
            return function() {
                return ret || ( ret = fn.apply( this, arguments ) );/*this指向函数调用的作用域*/
            }
        };

        ```
    - 高阶函数实现AOP(面向切面编程)
    ```javascript
    Function.prototype.before = function( beforefn ) {
        var _self = this;
        return function() {
            beforefn.apply( this, arguments );
            return _self.apply( this, arguments );
        };
    };

    Function.prototype.after = function( afterfn ) {
        var _self = this;
        return function() {
            var ret = _self.apply( this, arguments );
            afterfn.apply( this, arguments );
            return ret;
        }
    };

    var func = function() {
        console.log( 2 );
    };

    func = func.before(function(){
        consoel.log( 1 );
    }).after(function(){
        console.log( 3 );
    });

    func(); /*1 , 2 , 3*/
    ```
    - 高阶函数的其他应用
        - currying
        ```javascript
        var currying = function( fn ) {
            var args = [];
            return function() {
                if(arguments.length === 0) {
                    return fn.apply( this, arguments );
                } else {
                    [].push.apply( args, arguments );
                    return arguments.callee;
                }
            }
        };

        var cost = (function() {
            var money = 0;

            return function() {
                for ( var i = 0, l = arguments.length; i < l; i ++ ) {
                    money += arguments[ i ];
                }
                return money;
            }
        })();
        ```
        - uncurrying
        ```javascript
        Function.prototype.uncurrying = function() {
            var self = this;
            return function() {
                var obj = Array.prototype.shift.call( arguments );
                return self.apply( obj, arguments );
            }
        };

        Function.prototype.uncurrying = function() {
            var self = this;
            return function() {
                return Function.prototype.call.apply( self, arguments );
            }
        }
        ```
        - 函数节流
        ```javascript
        var throttle = function( fn, interval ) {

            var _self = fn,
                timer,
                firstTime = true;
            
            return function() {
                var args = arguments,
                    _me = this;

                if( firstTime ){
                    _self.apply( _me, args );
                    return firstTime = false;
                }

                if( timer ) {
                    return false;
                }

                timer = setTimeout(function() {
                    clearTimeout(timer);
                    timer = null;
                    _self.apply( _me, args );
                }, interval || 500);
            };
        };
        ```
        - 分时函数
        ```javascript
        var timeChunk = function( ary, fn, count ) {
            var obj,
                t;
            
            var start = function() {
                for ( var i = 0; i < Math.min( count || 1, ary.length ); i++ ) {
                    var obj = ary.shift();
                    fn(obj);
                }
            }

            return function() {
                t = setInterval(function() {
                    if( ary.length === 0 ) {
                        return clearInterval(t);
                    }
                    start();
                }, 200);
            };
        };
        ```
        - 惰性加载函数
        ```javascript
        var addEvent = function( elem, type, handler ) {
            if( window.addEventListener ) {
                addEvent = function( elem, type, handler ) {
                    elem.addEventListener( type, handler, false )
                };
            } else {
                addEvent = function( elem, type, handler ) {
                    elem.attachEvent( 'on' + type, handler );
                }
            }

            addEvent( elem, type, handler );
        };
        ```

## 单例模式
- 定义： 保证一个类仅有一个实例，并提供一个访问它的全局访问点。
- 单例模式实现一：面向对象
```javascript
var Singleton = function( name ) {
    this.name = name;
    this.instance = null;
};

Singleton.prototype.getName = function() {
    alert(this.name);
};

Singleton.getInstance = function( name ) {
    if( !this.instance ) {
        this.instance = new Singleton( name );
    }

    return this.instance;
};


var instance1 = Singleton.getInstance("naruto");
var instance2 = Singleton.getInstance("itach");

console.log( instance1 === instance2 ) //true
instance2.getName(); //naruto

```

- 单例模式实现二：闭包
```javascript
var Singleton = function( name ) {
    this.name = name;
};

Singleton.prototype.getName = function() {
    alert(this.name);
};

Singleton.getInstance = (function(){
    var instance = null;

    return funtion( name ) {
        if( !instance ) {
            instance = new Singleton( name );
        }

        return instance;
    }
})();

```

- 单例模式实现三：透明的单例模式
```javascript
var createDiv = (function() {
    var instance;

    var createDiv = function( html ) {
        if( instance ) {
            return instance;
        }

        this.html = html;
        this.init();

        return instance = this;
    };

    createDiv.prototype.init = function() {
        var div = document.createElement('div');

        div.innerHTML = this.html;
        document.body.appendChild(div);
    }
})();

var div1 = new createDiv("<div>1<div>");
var div2 = new createDiv("<div>2<div>");

alert(div1 === div2); //true
```

- 单例模式实现四：用代理实现单例模式
```javascript
var createDiv = function( html ) {
    this.html = html;
    this.init();
};

CreateDiv.prototype.init = function() {
    var div = document.createElement('div');

    div.innerHTML = this.html;
    document.body.appendChild(div);
};

var ProxySingletonCreateDiv = (function() {

    var instance;

    return function( html ) {
        if( !instance ) {
            instance = new createDiv(html);
        }

        return instance;
    }

})();

```

- 通用的单例模式
```javascript
var getSingle = function( fn ) {
    var ret;
    return funtion() {
        return ret || ( result = fn.apply(this, arguments) );
    }
};
```




