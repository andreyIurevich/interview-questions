Event Loop (Событийный цикл) - это механизм JavaScript-окружения, который позволяет однопоточному JavaScript выполнять асинхронный код, не блокируя основной поток.

Во внутренней реализации данного механизма можно выделить следующие основные части: call stack, web api и очередь задач (task queue), которая в свою очередь делится на macrotask queue и microstask queue.

К микротаскам относятся - промисы, queueMicrotask и mutationObserver
К макротаскам относятся - Таймеры (setInterval, setTimeout), события.

После выполнения синхронного кода event loop **сначала выполняет все microtasks**, затем браузер может сделать перерисовку, и только потом берётся следующая macrotask.

Пример:

```javascript
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

console.log('end');
```

Результат:

```txt
start
end
promise
timeout
```

Благодаря этому JavaScript остаётся отзывчивым, а UI не блокируется.