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
  .then(p => p + '1', p => p + '2')
  .catch(p => p + 'b')
  .catch(p => p + 'c')
  .then(p => p + 'd1')
  .then('d2')
  .then(p => p + 'd3')
  .finally(p => p + 'e')
  .then(p => console.log(p));
  
// Вывод: a2d1d3
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
  let currentResolve = null;
  let currentReject = null;

  return function (...args) {
    if (timeoutId) {
      clearTimeout(timeoutId);
      if (currentReject) {
        currentReject('Cancelled');
      }
    }

    return new Promise((resolve, reject) => {
      currentResolve = resolve;
      currentReject = reject;

      setTimeout(async () => {
        try {
          const res = await fn(...args);
          resolve(res);
        } catch (err) {
          reject(err);
        } finally {
          timeoutId = null;
        }
      }, delay);
    });
  }
}

const search = debounceAsync(async () => {
  return await promise1;
}, 500);

const searchRes = await search();

console.log('-> searchRes ', searchRes);
```

