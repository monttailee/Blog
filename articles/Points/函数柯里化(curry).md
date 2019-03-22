### 定义
指封装一个函数，接收原始函数作为参数传入，并返回一个能够接收并处理剩余参数的函数；如果你一头雾水，那么看看下面的常用公式你就明白了：
```
// 公式类型一(当传入参数等于函数参数数量时开始执行):
fn(a,b,c,d) => fn(a)(b)(c)(d);
fn(a,b,c,d) => fn(a, b)(c)(d);
fn(a,b,c,d) => fn(a)(b,c,d);
附通用代码：
function curry(fn,args){
    var length=fn.length;  //fn函数的形参个数
    var args=args || [];   //第一次接收的参数
    return function(){
        var newArgs=args.concat(Array.prototype.slice.call(arguments));//第二次接收的参数
        if(newArgs.length<length){
            return curry.call(this,fn,newArgs);  //参数没有达到形参个数时，递归调用
        }else{
            return fn.apply(this,newArgs);
        }
    }
}


// 公式类型二(当没有参数传入时（且参数数量满足）开始执行):
fn(a,b,c,d) => fn(a)(b)(c)(d)();
fn(a,b,c,d) => fn(a);fn(b);fn(c);fn(d);fn();
附通用代码：
function curry(fn,args){
    var length=fn.length;  //fn函数的形参个数
    var args=args || [];   //第一次接收的参数
    return function(){
        var newArgs=args.concat(Array.prototype.slice.call(arguments));//第二次接收的参数
        if(newArgs.length > 0 || newArgs.length<length){
            return curry.call(this,fn,newArgs);  //参数没有达到形参个数时，递归调用
        }else{
            return fn.apply(this,newArgs);
        }
    }
}
```

### 举个栗子
```
我们想验证手机号，邮箱，怎么做呢？我们用公式一实践一下吧：

①定义入参函数
function check(testString,reg){
    return reg.test(testString);
}

②套用柯里化公式
var _check = curry(check);

var checkPhone = _check(/^1[34578]\d{9}$/);
var checkEmail = _check(/^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/);

checkPhone('183888888');
checkEmail('xxxxx@test.com');

或者：
_check(/^1[34578]\d{9}$/)('183888888')
_check(/^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/)('xxxxx@test.com')
```

#### 柯里化的特点
优点：参数复用/延迟执行/提前返回
缺点：性能损耗
