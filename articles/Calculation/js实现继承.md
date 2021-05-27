##### js实现继承

###### class/extends
```
class Animal {
  constructor (name) {
    this.name = name
  }

  hello () {
    console.log('hello')
  }
}

class Dog extends Animal {
  constructor (name, say) {
    super(name)
    this.say = say
  }
}
```

###### function/new
```
function Animal (name) {
  this.name = name
}

Animal.prototype.hello = () => {
  console.log('hello')
}

function Dog (name, say) {
  // 01 继承属性
  Animal.call(this, name)
  this.say = say
}

// 02 通过连接原型链完成继承
Dog.prototype = Object.create(Animal.prototype)

// 03 再加上 constructor
Dog.prototype.constructor = Dog
// Reflect.defineProperty(Dog.prototype, "constructor", {
//  value: Dog,
//  enumerable: false, // 不可枚举
//  writable: true
// })
```