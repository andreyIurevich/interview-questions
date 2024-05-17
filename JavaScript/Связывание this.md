При вызове функции создаётся запись называемая контекстом выполнения. Запись содержит информацию о том, откуда была вызвана функция (стек вызовов), как была вызвана, какие параметры были переданы и т.д. Одним из свойств этой записи является ссылка this, которая используется на протяжении выполнения этой фунции.

Связывание this осуществляется не во время написания программы, а во время выполнения. Связывание является контекстным, т.е. базируется на условиях вызова функции.

Стрелочные фунции (`=>`) принимают связывание this от внешней области видимости.

##### Неявная потеря this

Неявная потеря `this` - когда функция с неявным связыванием теряет это связывание, что обычно означает возврат к связыванию по умолчанию.

Пример 1

```javascript
function foo() {
	console.log(this.a);
}

var obj = {
	a: 2,
	foo: foo,
};

var bar = obj.foo;
var a = "global";

bar(); // строка "global"
```

Пример 2

```javascript
function foo() {
	console.log(this.a);
}

function doFoo(fn) {
	fn();
}

var obj = {
	a: 2,
	foo: foo,
};

var a = 'global';

doFoo(obj.foo); // строка 'global'
```

##### Жёсткое связывание

```javascript
function bind(fn, obj) {
	return function() {
		return fn.apply(obj, arguments);
	}
}
```


К месту вызова функции применяются 4 правила:
1. Вызывается с new  - используется сконструированный объект
2. Вызывается с call и apply (или bind) - используется заданный объект
3. Вызывается с контекстным объектом (неявное связывание) - исп. этот контекстный объект
4. По умолчанию: undefined в режиме strict, глобальный объект в противном случае.


##### Задачи

Для места вызова важен только верхний/последний уровень цепоки ссылок на свойства объекта

```javascript
function foo() {
	console.log(this.a);
}

var obj2 = {
	a: 42,
	foo: foo,
};

var obj1 = {
	a: 2,
	obj2: obj2,
};

obj1.obj2.foo(); // 42
```

Стрелочная функция, созданная в foo(), лексически захватывает значение this функиции foo() на момент вызова. Так как foo() была связана по this с obj1, bar - также будет связана по this с obj1. *Лексическое связывание стрелочной функции не может быть переопределено (даже с new)*

```javascript
function foo() {
	return (a) => {
		console.log(this.a);
	};
}

var obj1 = {
	a: 2,
};

var obj2 = {
	a: 3,
};

var bar = foo.call(obj1);
bar.call(obj2); // 2, не 3!
```

```javascript
function foo() {
	const x = 10;
  
  return {
  	x: 20,
    bar: () => {
    	console.log(this.x);
    },
    baz: function() {
    	console.log(this.x);
    },
  };
}

const obj1 = foo();
obj1.bar(); // undefined
obj1.baz(); // 20

const obj2 = foo.call({ x: 30 });

let y = obj2.bar;
let z = obj2.baz;

y(); // 30 
z(); // undefined

obj2.bar(); // 30
obj2.baz(); // 20
```

```javascript
const x = () => {
	console.log(this);
}

x(); // в обоих режимах (strict и обычный будет window)

function foo() {
	console.log(this);
}

foo(); // в режиме strict - undefined, в обычном - window
```

```javascript
1)
'use strict';

console.log(this);

Как в случае включённого режима use strict, так и в отключённом режиме, выводится объект
window в браузере.

2)
'use strict';

function fn() {
	console.log(this); // undefined
}

fn();

3)
function fn() {
	console.log(this); // объект window в браузере
}

fn();

Использование this внутри объекта

4)

const user = {
  name: 'user1',
  print: function() {
  	console.log(this.name);
  }
}

user.print(); // user1

Использование this внутри вложенных объектов

5) 

var obj1 = {
  hello: function() {
    console.log('Hello world');
    return this;
  },
  obj2: {
      breed: 'dog',
      speak: function(){
            console.log('woof!');
            return this;
        }
    }
};
 
console.log(obj1);
console.log(obj1.hello());  // выводит 'Hello world' и возвращает obj1
console.log(obj1.obj2);
console.log(obj1.obj2.speak());  // выводит 'woof!' и возвращает obj2

6)

var obj1 = {
  a: 2,
  print: function() {
  	return function() {
    		console.log(this.a); // undefined
    };
  },
};

console.log(obj1.print()());

7)

var obj1 = {
  a: 2,
  print: function() {
  	return () => { 
    		console.log(this.a); // 2
    };
  },
};

console.log(obj1.print()());

----------------------------------------------
1)

'use strict';

function globalFunc() {
  console.log(this);
}
const globalArrowFunc = () => {
  console.log(this);
}

console.log(this); // window
globalFunc(); // undefined
globalArrowFunc(); // window

Режим use strict выключен

function globalFunc() {
  console.log(this);
}
const globalArrowFunc = () => {
  console.log(this);
}

console.log(this); // window
globalFunc(); // window
globalArrowFunc(); // window

2)

'use strict';

const user = {
  name: 'Bob',
  userThis: this,
  func() {
    console.log(this);
  },
  arrowFunc: () => {
    console.log(this);
  }
};

console.log(user.userThis); // window
user.func(); // user
user.arrowFunc(); // window

Режим use strict выключен

const user = {
  name: 'Bob',
  userThis: this,
  func() {
    console.log(this);
  },
  arrowFunc: () => {
    console.log(this);
  }
};

console.log(user.userThis); // window
user.func(); // user
user.arrowFunc(); // window

3)

'use strict';

const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
  funcArrow() {
    return () => {
      console.log(this);
    }
  },
  arrowFunc: () => {
    return function() {
      console.log(this);
    }
  },
  arrowArrow: () => {
    return () => {
      console.log(this);
    }
  },
};

user.funcFunc()(); // undefined
user.funcArrow()(); // user
user.arrowFunc()(); // undefined
user.arrowArrow()(); // window

Режим use strict выключен

const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
  funcArrow() {
    return () => {
      console.log(this);
    }
  },
  arrowFunc: () => {
    return function() {
      console.log(this);
    }
  },
  arrowArrow: () => {
    return () => {
      console.log(this);
    }
  },
};

user.funcFunc()(); // window
user.funcArrow()(); // user
user.arrowFunc()(); // window
user.arrowArrow()(); // window

4)

'use strict';

var dog = {
  breed: 'Beagles',
  lovesToChase: 'rabbits'
};

function chase() {
  console.log(this.breed); 
}

dog.foo = chase;
dog.foo(); // Beagles

5)

'use strict';

const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
  funcArrow() {
    return () => {
      console.log(this);
    }
  },
  arrowFunc: () => {
    return function() {
      console.log(this);
    }
  },
  arrowArrow: () => {
    return () => {
      console.log(this);
    }
  },
};

const user2 = {
  name: 'Jim',
  funcFunc: user.funcFunc(),
  funcArrow: user.funcArrow(),
  arrowFunc: user.arrowFunc(),
  arrowArrow: user.arrowArrow()
}

user2.funcFunc(); // user2
user2.funcArrow(); // user
user2.arrowFunc(); // user2
user2.arrowArrow(); // window
```
