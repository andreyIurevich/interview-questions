При вызове функции создаётся запись называемая контекстом выполнения. Запись содержит информацию о том, откуда была вызвана функция (стек вызовов), как была вызвана, какие параметры были переданы и т.д. Одним из свойств этой записи является ссылка this, которая используется на протяжении выполнения этой фунции.

Связывание this осуществляется не во время написания программы, а во время выполнения. Связывание является контекстным, т.е. базируется на условиях вызова функции.

Стрелочные фунции (`=>`) принимают связывание this от внешней области видимости.

##### Неявная потеря this

Неявная потеря контекста (`this`) — это ситуация, когда метод объекта вызывается таким образом, что он превращается в обычную функцию. В итоге `this` перестает указывать на объект и начинает указывать на глобальный объект (`window`) или `undefined`.

Вот основные сценарии, где это происходит:
1. Присваивание метода переменной
2. Передача метода в качестве колбэка

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
Жесткое связывание решает проблему "потери" функцией её предполагаемого связывания this

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

---

## Группа 1: Стрелочные vs Обычные функции

Эти задачи проверяют главное отличие: у обычных функций `this` определяется в момент вызова, а у стрелочных — берется из внешнего лексического окружения (в момент создания).

## Задача 1.1: Метод объекта как стрелка

```javascript
const user = {
  name: 'Alex',
  greet: () => {
    console.log(this.name);
  },
  greetOld() {
    console.log(this.name);
  }
};

user.greet();    // Что выведет? (undefined / '' в браузере)
user.greetOld(); // Что выведет? ('Alex')
```

- Почему: Стрелочная функция `greet` была создана в глобальной области видимости (сам объект `{...}` не создает новую область видимости). Поэтому её `this` ссылается на глобальный объект `window` / `global`.

## Задача 1.2: Стрелка внутри обычной функции (Замыкание контекста)

```javascript
const filter = {
  numbers:,
  pattern: 2,
  getFiltered() {
    return this.numbers.filter(function(num) {
      return num === this.pattern; // Обычная функция
    });
  },
  getFilteredArrow() {
    return this.numbers.filter((num) => {
      return num === this.pattern; // Стрелочная функция
    });
  }
};

console.log(filter.getFiltered());      // [] (ошибка тихая, т.к. this.pattern === undefined)
console.log(filter.getFilteredArrow()); // [2]
```

- Почему: Обычная функция внутри метода `filter` вызывается самим движком JS без контекста (`this` становится `window/undefined`). Стрелочная функция успешно берет `this` от родительского метода `getFilteredArrow` (который равен объекту `filter`).

---

## Группа 2: Потеря контекста (Context Loss)

Самая частая ошибка в реальном коде, когда метод объекта присваивают в переменную или передают как колбэк.

## Задача 2.1: Копирование метода в переменную

```javascript
const person = {
  name: 'John',
  sayName() {
    console.log(this.name);
  }
};

const johnSay = person.sayName;
johnSay(); // Что выведет? (undefined / '' в браузере)
```

- Почему: Вызов `johnSay()` идет «без точки» — это обычный вызов функции. Контекст теряется, `this` равен глобальному объекту (или `undefined` в strict mode).

## Задача 2.2: Метод внутри setTimeout

```javascript
const button = {
  text: 'Click me',
  click() {
    console.log(this.text);
  }
};

setTimeout(button.click, 1000); // Что выведет через секунду? (undefined)
```

- Почему: В `setTimeout` передается ссылка на функцию. Спустя секунду браузер вызывает её изнутри себя примерно так: `callback()`. Контекст `button` утерян.

---

## Группа 3: Явное связывание (`call`, `apply`, `bind`)

Задачи на умение принудительно переопределять контекст и знание особенностей метода `bind`.

## Задача 3.1: Повторный bind

```javascript
function show() {
  console.log(this.value);
}

const obj1 = { value: 1 };
const obj2 = { value: 2 };

const bound = show.bind(obj1).bind(obj2);
bound(); // Что выведет? (1)
```

- Почему: Метод `.bind()` создает новую обертку вокруг функции и жестко привязывает контекст один раз и навсегда. Все последующие вызовы `.bind()` или `.call()` на этой обертке уже не могут изменить её внутренний `this`.

## Задача 3.2: Передача context-метода в call

```javascript
const car = {
  brand: 'Tesla',
  getBrand() {
    return this.brand;
  }
};

const bike = { brand: 'Ducati' };

console.log(car.getBrand.call(bike)); // Что выведет? ('Ducati')
```

- Почему: Мы берем саму функцию `getBrand` и принудительно вызываем её в контексте объекта `bike` с помощью `.call()`.

---

## Группа 4: Продвинутые цепочки и Strict Mode

Задачи на «внимательность», где комбинируются цепочки вызовов или проверяется режим `'use strict'`.

## Задача 4.1: Строгий режим

```javascript
'use strict';
function test() {
  console.log(this);
}
test(); // Что выведет? (undefined)
```

- Почему: Без `'use strict'` обычный вызов функции подставляет вместо `this` объект `window/global`. В строгом режиме автоподстановка отключена, поэтому `this` остается `undefined`.

## Задача 4.2: Цепочка вызовов (Метод возвращает функцию)

```javascript
const group = {
  title: 'JS Theory',
  showTitle() {
    return function() {
      console.log(this.title);
    };
  }
};

group.showTitle()(); // Что выведет? (undefined)
```

- Почему: Первый вызов `group.showTitle()` возвращает _новую обычную функцию_. А вот второй вызов `()` происходит уже без контекста (без точки). Чтобы это работало, внутреннюю функцию нужно делать стрелочной.

---

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

```js
const userService = {
	currentFilter: 'active',
	
	users: [
		{ name: 'Alex', status: 'active' },
		{ name: 'Nick', status: 'deleted' },
	],

	getFilteredUsers: function () {
		return this.users.filter(function (user) {
			return user.status === this.currentFilter;
		})
	}
}

console.log(userService.getFilteredUsers()) // []
// в режиме use strict this = undefined, будет ошибка TypeError
```

```js
var a = {
	name: 'a',
	foo: function () {
		console.log(this.name);
	}
};

a.foo(); // a

var bar = a.foo;

bar(); // undefined

var b = {
	name: 'b'
};

b.foo = a.foo;

b.foo(); // b

var c = {
	name: 'c'
};

bar.call(c, 1, 2); // c
a.foo.apply(b, [1, 2]); // b
a.foo.bind(b).call(c); // b
a.foo.bind(b).bind(c)(); // b
```

```js
var a = {
  firstName: "Bill",
  lastName: "Ivanov",
  sayName: function () {
    console.log(this.firstName);
  },
  sayLastName: () => {
    console.log(this.lastName);
  }
};

a.sayName(); // "Bill"

var b = a.sayName;

b(); // undefined

a.sayName.bind({ firstName: "Boris" })(); // "Boris"

a.sayName(); // "Bill"

a.sayLastName(); // undefined

a.sayName.bind({ firstName: "Boris" }).bind({ firstName: "Tom" })(); // "Boris"
a.sayLastName.bind({ lastName: "Petrov" })(); // undefined
```

```js
const user = {
  name: 'Алексей',
  greet() {
    console.log(`Привет, я ${this.name}`);
  },
  arrowGreet: () => {
    console.log(`Привет, я ${this.name}`);
  },
  delayedGreet() {
    setTimeout(function() {
      console.log(`Привет, я ${this.name}`);
    }, 1000);
  },
  fixedDelayedGreet() {
    setTimeout(() => {
      console.log(`Привет, я ${this.name}`);
    }, 1000);
  }
};

// Вызовы:
user.greet(); // 'Алексей'
user.arrowGreet(); // undefined
user.delayedGreet(); // undefined
user.fixedDelayedGreet(); // 'Алексей'

```

**Разбор вызовов:**

1. `user.greet()` ➡️ `'Привет, я Алексей'`  
    Обычный метод объекта. Когда функция вызывается «через точку» (`user.greet()`), контекстом `this` автоматически становится объект перед точкой.
2. `user.arrowGreet()` ➡️ `'Привет, я undefined'`  
    Стрелочные функции не имеют своего `this`. Они берут его из внешнего (лексического) окружения в момент создания. Внешним окружением для объекта `user` является глобальная область видимости (в браузере — `window`). Объект `user` не создает свою область видимости (scope) для `this` — её создают только функции.
3. `user.delayedGreet()` ➡️ `'Привет, я undefined'` (через 1 сек)  
    Ты абсолютно прав: `this` внутри обычной функции ссылается на `window`.
    
    - Почему: Обычную функцию внутри `setTimeout` вызывает не твой код, её вызывает сам браузерный движок внутри асинхронного таймера. А когда обычная функция вызывается без явного контекста (не через точку), её `this` по умолчанию сбрасывается на глобальный объект (`window` в нестрогом режиме или `undefined` в `use strict`).
    
4. `user.fixedDelayedGreet()` ➡️ `'Привет, я Алексей'` (через 1 сек)  
    И снова ты абсолютно прав: стрелочная функция заимствует `this` из родительской функции. Родительская функция `fixedDelayedGreet` была вызвана как метод объекта `user`, поэтому её `this` равнялся объекту `user`. Стрелка, родившись внутри этого метода, намертво зафиксировала этот контекст в своем замыкании.

```js
const BoundUser = User.bind({name: "Bob"}, "Alice");

const user = new BoundUser();

console.log(user.name);
```

```js
function User(name) {
  this.name = name;
}

const BoundUser = User.bind(null, "Alice");

const user = new BoundUser(); // аналогично new User("Alice")

console.log(user.name);
```

`bind` фиксирует аргумент `"Alice"`, но фиксированный `this` при вызове через `new` игнорируется. Конструктор получает новый созданный объект как `this`, поэтому поле `name` записывается в него.