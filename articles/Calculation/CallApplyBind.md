#### 实现call方法
```
Function.prototype.myCall = function(content){
    const key = Symbol('key');
    // 将实际func变成content的一个属性
    content[key] = this; // 谁调用myCall谁就是this
    
    //const args = [].slice.call(arguments, 1);
    const args = Array.from(arguments).slice(1);
    const res = content[key](...args);
    delete content[key];
    return res;
}

const obj = { name: 'hello' };
function say(){
    console.log('-------------', this.name)
}
say.myCall(obj)
```

#### 实现apply方法
```
Function.prototype.myApply = function(content){
    const key = Symbol('key');
    // 将实际func变成content的一个属性
    content[key] = this; // 谁调用myCall谁就是this
    let res;
    if(arguments[1]){
        res = content[key](...arguments[1])
    } else{
        res = content[key]()
    }

    delete content[key];
    return res;
}

const obj = { name: 'hello' };
function say(){
    console.log('-------------', this.name)
}
say.myApply(obj)
```

#### 实现bind方法
```
// 收集参数，返回一个新方法，调用新方法
Function.prototype.myBind = function(content){
    const fn = this;
    const args = Array.from(arguments).slice(1);
    const newFunc = function(){
        const newArgs = args.concat(Array.from(arguments));
        if(this instanceof newFunc){
            // 返回的newFunc又被实例new了
            fn.apply(this, newArgs)
        } else{
            fn.apply(content, newArgs)
        }
    }
    newFunc.prototype = Object.create(fn.prototype);
    return newFunc
}

const obj = { name: 'hello' };
function say(){
    console.log('-------------', this.name)
}
const meSay = say.myBind(obj);
meSay()
```

##### call、apply都需要搜集参数，将调用方法变成content的一个属性立即执行；bind需要搜集参数，返回一个新方法，新方法被单独调用执行