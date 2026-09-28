`Promise.all` позволяет запустить множество промисов параллельно и дождаться, пока все они выполнятся. Если любой из промисов завершится с ошибкой, то промис, возвращённый `Promise.all`, немедленно завершается с этой ошибкой.

```js
let urls = [
'https://api.github.com/users/iliakan',
'https://api.github.com/users/remy',
'https://api.github.com/users/jeresig'
];

// Преобразуем каждый URL в промис, возвращённый fetch
let requests = urls.map(url => fetch(url));

// Promise.all будет ожидать выполнения всех промисов
Promise.all(requests)
	.then(responses => responses.forEach(
		response => alert(`${response.url}: ${response.status}`
)));
```

`Promise.allSettled` всегда ждёт завершения всех промисов. В массиве результатов будет

- `{status:"fulfilled", value:результат}` для успешных завершений,
- `{status:"rejected", reason:ошибка}` для ошибок.

`Promise.race` - очень похож на `Promise.all`, но ждёт только первый _выполненный_ промис, из которого берёт результат (или ошибку).

`Promise.any` - похож на `Promise.race`, но ждёт только первый _успешно выполненный_ промис, из которого берёт результат.

`Promise.resolve(value)` – возвращает успешно выполнившийся промис с результатом `value`.

`Promise.reject(error)` – возвращает промис с ошибкой `error`

[Подробнее](https://learn.javascript.ru/promise-api#promise-resolve-reject) 

Пример реализации `Promise.all`

```js
function myPromiseAll(taskList) {
  //to store results 
  const results = [];
  
  //to track how many promises have completed
  let promisesCompleted = 0;

  // return new promise
  return new Promise((resolve, reject) => {

    taskList.forEach((promise, index) => {
     //if promise passes
      promise.then((val) => {
        //store its outcome and increment the count 
        results[index] = val;
        promisesCompleted += 1;
        
        //if all the promises are completed, 
        //resolve and return the result
        if (promisesCompleted === taskList.length) {
          resolve(results)
        }
      })
         //if any promise fails, reject.
        .catch(error => {
          reject(error)
        })
    })
  });
}
```

Пример реализации `Promise.allSettled`

```javascript
function promiseAllSettled(promises) {
  return new Promise(resolve => {
    const results = [];
    let pending = promises.length;

    if (pending === 0) {
      return resolve([]);
    }

    promises.forEach((p, i) => {
      Promise.resolve(p)
        .then(value => {
          results[i] = { status: "fulfilled", value };
        })
        .catch(reason => {
          results[i] = { status: "rejected", reason };
        })
        .finally(() => {
          pending--;
          if (pending === 0) {
            resolve(results);
          }
        });
    });
  });
};
const p1 = Promise.resolve(42);
const p2 = Promise.reject("Ошибка");
const p3 = new Promise(res => setTimeout(() => res("Готово"), 200));

promiseAllSettled([p1, p2, p3]).then(console.log);
```

Пример реализации `Promise.race`

```javascript
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    for (const p of promises) {
      Promise.resolve(p)
        .then(resolve)
        .catch(reject);
    }
  });
};

const p1 = new Promise(res => setTimeout(() => res("Первый!"), 1000));
const p2 = new Promise(res => setTimeout(() => res("Второй!"), 500));
const p3 = new Promise((_, rej) => setTimeout(() => rej("Ошибка!"), 200));

promiseRace([p1, p2, p3])
  .then(result => console.log("Результат:", result))
  .catch(error => console.log("Ошибка:", error));
// через 200 мс p3 завершится с ошибкой
```

Пример реализации `Promise.any`

```javascript
function promiseAny(promises) {
  return new Promise((resolve, reject) => {
    let errors = [];
    let pending = promises.length;

    if (pending === 0) {
      // По спецификации — сразу ошибка AggregateError
      return reject(new AggregateError([], "All promises were rejected"));
    }

    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i])
        .then(value => {
          resolve(value);          // Первый успешно завершившийся — победитель
        })
        .catch(err => {
          errors[i] = err;
          pending--;

          if (pending === 0) {
            // Если все завершились с ошибкой
            reject(new AggregateError(errors, "All promises were rejected"));
          }
        });
    }
  });
};
const p1 = new Promise((_, rej) => setTimeout(() => rej("Ошибка 1"), 300));
const p2 = new Promise((_, rej) => setTimeout(() => rej("Ошибка 2"), 100));
const p3 = new Promise(res => setTimeout(() => res("Успех!"), 200));

promiseAny([p1, p2, p3])
  .then(result => console.log("Результат:", result))
  .catch(err => console.log("AggregateError:", err.errors));
// через 200 мс p3 успешно завершится  
```

Разные задачи про промисы

```javascript
function resolveAfter2Seconds(x) {
	return new Promise((resolve) => {
		setTimeout(() => {
			resolve(x);
		}, 2000);
	});
}

async function add1(x) {
	const a = await resolveAfter2Seconds(20); // 2
	const b = await resolveAfter2Seconds(30); // 2

	return x + a + b;
}

async function add2(x) {
	const promise_a = resolveAfter2Seconds(20);
	const promise_b = resolveAfter2Seconds(30);

	return x + (await promise_a) + (await promise_b); // 2
}

add1(10).then(console.log) // вернёт результат через 4 секунды
add2(20).then(console.log) // вернёт результат через 2 секунды
```

Задачи на "проваливание" промисов.

Если then() передаётся что-то отличное от функции (например, промис), это интерпретируется как then(null) и в следующий по цепочке промис «проваливается» результат предыдущего.

```javascript
Promise
  .reject('a')
  .catch(p => p + 'b')
  .catch(p => p + 'c')
  .then(p => p + 'd')
  .finally(p => p + 'e')
  .then(p => { console.log(p); })
  
// Вывод: abd
```

```javascript
Promise
  .resolve('a')
  .catch(p => p + 'f')
  .then(p => p + 'c')
  .then(p => p + 's')
  .then(p => { console.log(p); })
  
// Вывод: acs
```

```javascript
Promise
  .reject('a')
  .then(p => p + '1', p => p + '2') // a2
  .catch(p => p + 'b')
  .catch(p => p + 'c')
  .then(p => p + 'd1') // a2d1
  .then('d2')
  .then(p => p + 'd3') // a2d1d3
  .finally(p => p + 'e')
  .then(p => console.log(p));
  
// Вывод: a2d1d3
```

```js
Promise.resolve(1)
	.then(x => x + 1)
	.then(x => { throw x })
	.then(x => console.log(x))
	.catch(err => console.log(err)) // 2
	.then(x => Promise.resolve(1))
	.catch(err => console.log(err))
	.then(x => console.log(x)) // 1
// Вывод: 2 1
```

```javascript
const fn = async (n) => {
	await new Promise(res => setTimeout(res, 100));
	return n*n;
};

asyncLimit(fn, 50)(5); // rejected: Превышен лимит времени исполнения
asyncLimit(fn, 150)(5); // resolved: 25

const fn2 = async (a, b) => {
	await new Promise(res => setTimeout(res, 120));
	return a + b;
};

asyncLimit(fn2, 100)(1, 2); // rejected: Превышен лимит времени исполнения
asyncLimit(fn2, 150)(1, 2); // resolved: 3

const asyncLimit = (fn, delay) => {
	return async (...args) => {
		return Promise.race([
			fn(...args),
			new Promise((_, reject) => {
				setTimeout(() => reject("Timeout error"), delay)
			})
		])
	}
};
```

Реализуйте функцию `retryPromise(fn, retries)`, которая повторяет выполнение промиса при ошибке.

**Требования:**

- Принимает функцию, возвращающую промис, и количество попыток
- При успехе возвращает результат
- При ошибке повторяет попытку (до retries раз)
- Если все попытки неудачны - выбрасывает последнюю ошибку
- Первая попытка не считается за retry

```js
async function retryPromise(fn, retries) {
  // Количество полных попыток = 1 (первая) + retries (повторные)
  const totalAttempts = 1 + retries;

  for (let i = 0; i < totalAttempts; i++) {
    try {
      // Цикл ЗАМРЕТ на этой строчке, пока промис не выполнится!
      return await fn(); 
    } catch (err) {
      // Если это была последняя попытка — выбрасываем ошибку наружу
      if (i === totalAttempts - 1) {
        throw err;
      }
      // Если попытки еще есть, цикл просто перейдет на следующий шаг (итерацию)
      console.log(`Попытка ${i + 1} провалилась, пробуем снова...`);
    }
  }
}
```

Если ты используешь обычный классический цикл (`for`, `while` или `for...of`) внутри `async`-функции, то оператор `await` будет честно блокировать цикл и заставлять его дожидаться завершения асинхронной функции перед тем, как перейти к следующей итерации.

## Почему здесь `await` работает внутри цикла, а в `forEach` — нет?

Очень часто разработчики путают обычные циклы с методом массивов `.forEach()`. Вот в чем разница:

1. В обычном цикле `for` / `while`: `await` находится непосредственно внутри тела самой `async`-функции. Движок JS буквально ставит на паузу выполнение всей функции `retryPromise`, включая счетчик цикла. Следующий шаг `i++` не выполнится, пока `await` не отпустит поток.
2. В методе `.forEach(async () => { await fn() })`: Метод `forEach` — это обычная синхронная функция. Она просто запускает переданный ей колбэк много раз подряд. Она не умеет ждать промисы. В этом случае все итерации действительно запустятся одновременно и параллельно, не дожидаясь друг друга.

Именно поэтому для последовательных асинхронных действий (как в нашей задаче с retry) классический цикл `for` или `while` подходит просто идеально и защищает от переполнения стека вызовов, которое теоретически может случиться при слишком глубокой рекурсии.

---

Реализуйте функцию `promiseLimit(tasks, limit)`, которая выполняет не более N промисов одновременно.

**Требования:**

- Принимает массив функций и лимит параллельных выполнений
- Выполняет максимум limit задач одновременно
- Когда задача завершается, запускается следующая
- Возвращает массив всех результатов в исходном порядке
- Если задача падает - продолжает выполнение остальных

```js
const promise1 = new Promise((resolve) => {  
  setTimeout(() => {  
    resolve('1');  
  }, 1000);  
});  
  
const promise2 = new Promise((resolve, reject) => {  
  setTimeout(() => {  
    reject('error 2');  
  }, 500);  
});  
  
const promise3 = new Promise((resolve, reject) => {  
  setTimeout(() => {  
    resolve('3');  
  }, 3000);  
});  
  
const tasks = [  
  () => promise1,  
  () => promise2,  
  () => promise3,  
];  
  
async function promiseLimit(tasks, limit) {  
  const results = new Array(tasks.length);  
  let currentIndex = 0;  
  
  async function worker() {  
    while (currentIndex < tasks.length) {  
      const myIndex = currentIndex;  
      currentIndex++;  
      const task = tasks[myIndex];  
      try {  
        results[myIndex] = await task();  
      } catch (err) {  
        results[myIndex] = err;  
      }  
    }  
  }  
  
  const currentLimit = Math.min(limit, tasks.length);  
  const workers = [];  
  
  for (let i = 0; i < currentLimit; i++) {  
    workers.push(worker());  
  }  
  
  await Promise.all(workers);  
  
  return results;  
}  
  
const results = await promiseLimit(tasks, 2);  
console.log('-> results ', results);
```

Один воркер внутри себя работает последовательно. Но мы запускаем несколько воркеров одновременно. Они работают параллельно, деля между собой одну общую очередь задач.

Давай визуализируем это на примере. У нас есть 6 задач и лимит = 2.

Мы запускаем ровно 2 воркера одновременно: Воркер А и Воркер Б.

```text
Очередь задач: [ Задача 1,  Задача 2,  Задача 3,  Задача 4,  Задача 5,  Задача 6 ]
                 ▲           ▲
                 │           │
             Воркер А    Воркер Б
```

## Как это происходит во времени:

1. Старт:
    
    - Воркер А берет Задачу 1 и начинает её выполнять (`await`).
    - Воркер Б в эту же миллисекунду берет Задачу 2 и начинает её выполнять (`await`).
    - _Сейчас параллельно выполняются 2 задачи. Лимит соблюден._
    
2. Воркер Б финишировал первым: Допустим, Задача 2 была короткой и завершилась быстрее.
    
    - Воркер Б сохраняет результат Задачи 2.
    - Его внутренний цикл `while` тут же переходит на следующий виток. Воркер Б смотрит на общую очередь и берет Задачу 3.
    - _В этот момент Воркер А всё еще ждет Задачу 1, а Воркер Б уже делает Задачу 3._
    
3. Воркер А финишировал: Наконец завершилась Задача 1.
    
    - Воркер А сохраняет результат Задачи 1.
    - Его цикл `while` делает виток, он смотрит на очередь и берет следующую свободную — Задачу 4.
    

## В чём отличие от обычной последовательности?

Если бы мы делали просто последовательно, задачи шли бы строго друг за другом: 1 ➔ 2 ➔ 3 ➔ 4.  
В нашей схеме они идут пачками по две: (1 и 2 параллельно) ➔ как только освободилось место ➔ в игру вступает 3, затем 4 и так далее, пока весь массив не закончится.

Любая асинхронная функция (`async function`) ВСЕГДА возвращает промис. У неё просто нет другого выбора.

Даже если ты не пишешь слово `return` внутри `async`-функции, движок JavaScript автоматически оборачивает её результат в промис под капотом.

Вот как это работает в деталях:

## 1. Что возвращает `runWorker()`?

Когда ты вызываешь `runWorker()`, движок JavaScript мгновенно возвращает объект `Promise` в состоянии `pending` (ожидание).

- Пока внутри воркера крутится цикл `while` и выполняются `await task()`, этот промис остаётся «подвешенным» (ожидающим).
- Как только цикл `while` завершается (задачи закончились) и функция доходит до своей закрывающей фигурной скобки `}`, движок JS автоматически переводит этот промис в состояние `fulfilled` (успешно выполнен) со значением `undefined` (так как явного `return` нет).

## 2. Как это видит `Promise.all`?

В массив `workers` мы складываем как раз эти автоматически созданные промисы:

```javascript
workers.push(runWorker()); // В массив падает [Promise <pending>, Promise <pending>]
```

`Promise.all` принимает этот массив промисов и начинает за ними следить. Для него это самые обычные промисы. Как только последний воркер завершает свой цикл и его неявный промис переходит в состояние `fulfilled`, `Promise.all` понимает: «Всё, работа окончена!» — и пропускает код дальше к строчке `return results;`.

## Визуальное сравнение:

То, что ты пишешь:

```javascript
async function runWorker() {
  while (currentIndex < tasks.length) {
    await task();
  }
}
```

То, как это видит и выполняет движок JavaScript под капотом:

```javascript
function runWorker() {
  // Автоматическая обертка в промис
  return new Promise((resolve, reject) => {
    // ... тут крутится логика цикла ...
    
    // Как только цикл закончился, неявно вызывается resolve()
    resolve(undefined); 
  });
}
```

---
Реализуйте `debounceAsync(fn, delay)` - debounce для асинхронных функций.

**Что такое debounce?**  
Это паттерн, который откладывает выполнение функции до тех пор, пока не пройдёт delay мс после последнего вызова. Используется для оптимизации поиска при вводе текста.

**Требования:**

- Откладывает выполнение функции на delay мс
- При повторном вызове отменяет предыдущий таймер и начинает заново
- Возвращает Promise с результатом последнего вызова
- Все pending вызовы резолвятся с результатом последнего

```js
const promise1 = new Promise((resolve) => {
  setTimeout(() => {
    resolve('1');
  }, 1000);
});

function debounceAsync(fn, delay) {
  let timeoutId = null;
  let sharedPromise = null;
  let sharedResolve = null;
  let sharedReject = null;

  return function (...args) {
    // 1. Если это новый цикл дебаунса, создаем ОДИН общий промис для всех вызовов
    if (!sharedPromise) {
      sharedPromise = new Promise((resolve, reject) => {
        sharedResolve = resolve;
        sharedReject = reject;
      });
    }

    // 2. Сбрасываем предыдущий таймер, если он был
    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // 3. Запускаем новый таймер. 
    // Заметьте: args берутся из ПОСЛЕДНЕГО вызова благодаря замыканию!
    timeoutId = setTimeout(async () => {
      try {
        const result = await fn(...args);
        sharedResolve(result); // Резолвим ОДИН общий промис для всех
      } catch (error) {
        sharedReject(error);   // Или отклоняем его
      } finally {
        // Очищаем состояние для следующего (будущего) цикла дебаунса
        timeoutId = null;
        sharedPromise = null;
        sharedResolve = null;
        sharedReject = null;
      }
    }, delay);

    // 4. Все вызовы в течение delay получают ссылку на один и тот же промис
    return sharedPromise;
  };
}

const search = debounceAsync(async () => {
  return await promise1;
}, 500);

const searchRes = await search();

console.log('-> searchRes ', searchRes);
```

---

Реализуйте `asyncMap(array, asyncFn)` - аналог Array.map для async функций.

**Задача:**  
Обычный `Array.map()` не умеет ждать async функции. Нужно реализовать версию, которая:

- Применяет async функцию к каждому элементу массива
- Дожидается выполнения всех промисов
- Возвращает массив результатов в том же порядке
- Выполняет все задачи параллельно (не последовательно!)

```js
async function asyncMap(array, asyncFn) {
	return Promise.all(array.map(asyncFn));
}
```

`Promise.all` ждет, пока _все_ промисы перейдут в состояние `fulfilled` (выполнено). После этого он возвращает один новый промис, который разрешается в массив чистых результатов.

Подобная задача AsyncFilter:

```js
async function asyncFilter(array, asyncCallback) {
  const masks = await Promise.all(array.map(asyncCallback));
  return array.filter((_, index) => masks[index]);
}

const result = await asyncFilter([1, 2, 3, 4], async (x) => x % 2 === 0);
```

---
Реализуйте метод `promiseFinally(promise, onFinally)`, который вызывается всегда.

**Что такое finally?**  
Это колбэк, который выполняется ВСЕГДА - независимо от того, промис завершился успешно или с ошибкой. Используется для очистки ресурсов.

**Требования:**

- Принимает промис и функцию onFinally
- Выполняет onFinally после завершения промиса (в любом случае)
- Не изменяет результат промиса (resolve остаётся resolve, reject остаётся reject)
- Возвращает промис с оригинальным результатом

```js
function promiseFinally(promise, onFinally) {
	return new Promise((resolve, reject) => {
		promise
			.then((res) => {
				resolve(res);
			})
			.catch((error) => {
				reject(error);
			})
			.finally(onFinally);
	});
}
```

---

Реализуйте `cachePromise(fn)` - кэширует результаты async функции (мемоизация).

**Что такое кэширование/мемоизация?**  
Сохранение результатов выполнения функции, чтобы при повторном вызове с теми же аргументами вернуть сохранённый результат вместо повторного выполнения.

**Требования:**

- Принимает async функцию и возвращает обёрнутую версию
- При первом вызове с аргументами - выполняет функцию и сохраняет результат
- При повторном вызове с теми же аргументами - возвращает сохранённый результат
- Ключ кэша = JSON.stringify(arguments)

```js
function cachePromise(fn) {
	const cache = new Map();

	return async function(...args) {
		const key = JSON.stringify(args);

		if (cache.has(key)) {
			return cache.get(key);
		}
	
		const result = await fn(...args);

		cache.set(key, result);

		return result;
	};
}
```

Т.к. при данном подходе кэшируется результат промиса, то при одновременном вызове функций с одинаковыми параметрами может быть проблема "гонки условий". Нужно сохранять сам промис, тогда функция будет возвращать этот промис при одинаковых параметрах.

```js
function cachePromise(fn) {
  const argsMap = new Map();

  return function (...args) { // async здесь больше не нужен, так как мы возвращаем промис напрямую
    const argsStr = JSON.stringify(args); // Используем args вместо arguments

    if (argsMap.has(argsStr)) {
      return argsMap.get(argsStr);
    }

    // Сохраняем промис сразу, не дожидаясь await
    const promise = fn(...args);
    argsMap.set(argsStr, promise);
    
    // Удаляем промис из кэша, если запрос упал с ошибкой,  
	// чтобы при следующем вызове функция попробовала выполниться снова  
	// promise.catch(() => {  
	//	argsMap.delete(argsStr);  
	// });
    
    return promise;
  };
}

```

Реализовать функцию asyncReduce, которая принимает в качестве аргументов - массив, асинхронную функцию и начальное значение.

```js
// Асинхронная функция, имитирующая запрос к БД или калькулятору на сервере
const asyncSum = (acc, el) => new Promise(resolve => {
  setTimeout(() => resolve(acc + el), 100);
});

async function asyncReduce(arr, fn, initVal) {
  // Проверяем, передано ли начальное значение (учитываем количество аргументов)
  const hasInitVal = arguments.length >= 3;
  
  // Если массив пустой и начального значения нет — выбрасываем стандартную ошибку
  if (arr.length === 0 && !hasInitVal) {
    throw new TypeError('Reduce of empty array with no initial value');
  }

  // Определяем стартовый индекс и начальное значение аккумулятора
  let startIndex = hasInitVal ? 0 : 1;
  let acc = hasInitVal ? initVal : arr[0];

  // Итерируемся по массиву, начиная с нужного индекса
  for (let i = startIndex; i < arr.length; i++) {
    // Передаем в fn: аккумулятор, текущий элемент, индекс и сам массив.
    // Обязательно делаем await, так как функция fn асинхронная!
    acc = await fn(acc, arr[i], i, arr);
  }

  return acc;
}

async function main() {
  const numbers =;
  
  // Вызов с начальным значением 10 (10 + 1 + 2 + 3 + 4)
  const result = await asyncReduce(numbers, asyncSum, 10);
  console.log(result); // Выведет 20 через 400 мс
}

main();
```

реализация через нативный reduce с цепочкой .then:

```js
function asyncReduceWithNative(arr, fn, initVal) {
  const hasInitVal = arguments.length >= 3;
  
  if (arr.length === 0 && !hasInitVal) {
    throw new TypeError('Reduce of empty array with no initial value');
  }

  // 1. Определяем стартовую точку для аккумулятора
  // Если initVal передан, оборачиваем его в Promise.resolve(), чтобы у нас ВСЕГДА был промис.
  // Если не передан, берем первый элемент (тоже оборачивая в промис).
  const startValue = hasInitVal ? Promise.resolve(initVal) : Promise.resolve(arr[0]);
  const startIndex = hasInitVal ? 0 : 1;

  // 2. Отрезаем часть массива, если нужно начать со 2-го элемента
  const arrayToReduce = startIndex === 1 ? arr.slice(1) : arr;

  // 3. Запускаем нативный reduce
  return arrayToReduce.reduce((promiseAcc, el, index) => {
    // Вычисляем реальный индекс для передачи в колбэк пользователя
    const realIndex = startIndex + index; 

    // Возвращаем НОВЫЙ промис, который ждет разрешения предыдущего
    return promiseAcc.then(async (resolvedAcc) => {
      // Когда прошлый шаг завершен, вызываем функцию fn для текущего элемента
      return await fn(resolvedAcc, el, realIndex, arr);
    });
  }, startValue);
}
```

Нативный цикл `arrayToReduce.reduce` **пробегается по всем элементам массива сразу (синхронно и мгновенно)**. Он **НЕ ждет** завершения промисов на каждой итерации.

Весь процесс можно разделить на два этапа:

Этап 1: Синхронное построение (Мгновенно)

Цикл `reduce` запускается и моментально "нанизывает" обработчики `.then()` друг на друга. Сами асинхронные функции `fn` в этот момент **еще не вызываются**.  
После завершения работы цикла `reduce` возвращает наружу один большой, длинный, но еще не выполнившийся промис. На этом работа самой функции `asyncReduceWithNative` полностью завершена.

Этап 2: Асинхронное выполнение (По очереди)

После того как текущий синхронный код завершился, в дело вступает Event Loop:

1. Разрешается самый первый базовый `Promise.resolve(initVal)`.
2. Это активирует колбэк в **первом** `.then()`. Запускается `fn` для первого элемента. Движок ждет его завершения.
3. Как только первый `fn` завершился, его результат передается во **второй** `.then()`. Запускается `fn` для второго элемента.
4. Процесс повторяется, как падение костяшек домино.

---

Вопросы для собеседования:
1. Что произойдет, если внутри микрозадачи рекурсивно вызывать другую микрозадачу (бесконечный цикл промисов)?
2. А если делать то же самое через setTimeout? Зависнет ли вкладка браузера в обоих случаях?

**Случай 1: Бесконечный цикл МИКРОзадач (Промисы)**

Представим такой код:

```js
function infiniteMicro() {
  Promise.resolve().then(infiniteMicro);
}
infiniteMicro();
```

- **Что происходит:** Первая функция выполняется и добавляет в очередь микрозадач саму себя. Синхронный стек пустеет. Event Loop заглядывает в очередь микрозадач, достает оттуда `infiniteMicro`, выполняет её... и она **снова** добавляет в очередь микрозадач себя.
- **Результат — Вкладка ЗАВИСЛА намертво:** Вспоминаем золотое правило: _Event Loop не перейдет к следующей макрозадаче и не отдаст управление браузеру, пока очередь микрозадач не опустеет на 100%_. Браузер физически не может перерисовать интерфейс (UI Render — это тоже разновидность макрозадачи), обработать клик мышки или скролл. Страница «умирает», анимации застывают, и через пару секунд браузер предлагает принудительно закрыть вкладку.

**Случай 2: Бесконечный цикл МАКРОзадач (`setTimeout`)**

Теперь посмотрим на этот код:

```js
function infiniteMacro() {
  setTimeout(infiniteMacro, 0);
}
infiniteMacro();
```

- **Что происходит:** Функция выполняется, регистрирует таймер в Web API и тут же завершается. Синхронный стек пустеет. Таймер на 0 мс мгновенно срабатывает, и колбэк падает в **очередь макрозадач**.
- **Поведение Event Loop:** Движок JS заглядывает в микрозадачи — там пусто. Тогда он берет **ровно одну** макрозадачу из очереди — нашу `infiniteMacro`. Она выполняется, планирует следующий `setTimeout` в Web API и завершается.
- **Результат — Вкладка РАБОТАЕТ:** Правило макрозадач гласит: _выполняется только ОДНА макрозадача за один тик (оборот) Event Loop_. После выполнения `infiniteMacro` движок обязан сделать полный круг: проверить микрозадачи, а затем дать браузеру возможность **сделать UI Render (перерисовать страницу)**. Новый таймаут в этот момент еще просто ждет своей очереди.

Интерфейс останется отзывчивым, анимации будут плавными, кнопки будут кликаться, хотя процессор и будет нагружен постоянным планированием таймаутов.

**Резюме для собеседования:**

- **Микрозадачи** выполняются непрерывным потоком «до упора». Бесконечные микрозадачи блокируют Event Loop и вешают интерфейс.
- **Макрозадачи** выполняются строго по одной за раз. Между ними Event Loop всегда успевает «глотнуть воздуха» — дать браузеру перерисовать страницу и обработать действия пользователя.

---

