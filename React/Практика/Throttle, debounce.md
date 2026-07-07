`Throttle` и `Debounce` решают задачи оптимизации.

`Throttle` - пропускает вызовы функции с определённой периодичностью.  
`Debounce` - откладывает вызов функции до того момента, когда с последнего вызова пройдёт определённое количество времени.

Пример использования throttle:
1. Если пользователь изменяет размер окна браузера и нам необходимо изменять содержимое сайта
2. Показ пользователю количества процентов прокрутки страницы

Пример использования debounce:
1. Обработка данных поискового запроса пользователя
2. Отправка данных аналитики на сервер

Реализация

```js
const throttle = (fn, delay) => {
	let timer = null;
	
	return (...args) => {
		if (timer) {
			return;
		}
		
		timer = setTimeout(() => {
			fn(args);
			timer = null;
		}, delay);
	};
};
```

```js
function debounce(originalFn, timeoutMs) {
	let timeout;
	
	return (...args) => {
		clearTimeout(timeout); 
		timeout = setTimeout(() => originalFn(...args), timeoutMs); 
	};
}
```

```jsx
function useDebounceValue(value: string, delay: number) {  
  const [debouncedValue, setDebouncedValue] = useState<string>('');  
  
  useEffect(() => {  
    const handler = setTimeout(() => {  
      setDebouncedValue(value);  
    }, delay);  
  
    return () => {  
      if (handler) clearTimeout(handler);  
    }  
  }, [value, delay])  
  
  return debouncedValue;  
}

import { useCallback, useEffect, useRef } from 'react';

/**
 * Хук для дебаунса самого вызова функции.
 * @param callback — Функция, вызов которой нужно задержать
 * @param delay — Задержка в миллисекундах
 */
export function useDebounceCallback<Args extends any[]>(
  callback: (...args: Args) => void,
  delay: number = 500
) {
  // 1. Сохраняем актуальную функцию в ref, чтобы асинхронный setTimeout 
  // всегда видел её последнюю версию, не перезапуская таймер.
  const callbackRef = useRef(callback);
  
  // 2. Храним id таймера, чтобы иметь к нему доступ между рендерами
  const timeoutRef = useRef<ReturnType<typeof setTimeout> | null>(null);

  // Каждый рендер обновляем ref актуальной функцией [3]
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  // 3. Очищаем таймер при размонтировании компонента (чтобы не было утечек памяти)
  useEffect(() => {
    return () => {
      if (timeoutRef.current) clearTimeout(timeoutRef.current);
    };
  }, []);

  // 4. Оборачиваем возвращаемую функцию в useCallback, 
  // чтобы ссылка на неё не менялась при перерендере компонента
  return useCallback((...args: Args) => {
    // Если таймер уже запущен — сбрасываем его
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }

    // Запускаем новый таймер
    timeoutRef.current = setTimeout(() => {
      callbackRef.current(...args);
    }, delay);
  }, [delay]);
}
```

Использование `useRef` для сохранения функции (паттерн Latest Ref) — это важнейший трюк для оптимизации асинхронного кода в React.

Переменная `callbackRef` решает фундаментальную проблему: как дать функции внутри `setTimeout` доступ к самым свежим переменным компонента и при этом не сбрасывать таймер дебаунса на каждый чих.

Давай разберем это на пальцах, посмотрев, что произошло бы БЕЗ использования `useRef`.

---

## Что было бы БЕЗ `useRef`? (Проблема сломанного замыкания)

Представь, что мы написали хук наивно, передав `callback` напрямую в зависимости `useCallback`:

```typescript
// ТАК ДЕЛАТЬ НЕЛЬЗЯ — ЭТО НАИВНЫЙ ВАРИАНТ С БАГАМИ
export function useDebounceCallback(callback, delay) {
  return useCallback((...args) => {
    // Каждое нажатие клавиши сбрасывает таймер
    if (timeoutRef.current) clearTimeout(timeoutRef.current);

    timeoutRef.current = setTimeout(() => {
      callback(...args); // <-- Вызываем напрямую из замыкания
    }, delay);
  }, [callback, delay]); // <-- callback в зависимостях!
}
```

А теперь посмотрим на компонент инпута, который использует этот наивный хук:

```tsx
function Search() {
  const [text, setText] = useState("");

  // При каждом изменении text функция пересоздается и видит АКТУАЛЬНЫЙ text
  const sendToServer = useDebounceCallback(() => {
    console.log("Отправляем на сервер:", text);
  }, 1000);

  return <input value={text} onChange={(e) => setText(e.target.value)} />;
}
```

Ловушка, в которую мы попали:

1. Пользователь вводит букву «R». Стейт `text` меняется. Компонент перерендеривается.
2. Функция `sendToServer` пересоздается, потому что `callback` изменился (он теперь видит букву «R»).
3. Из-за того, что изменилась ссылка на `sendToServer`, во всех дочерних компонентах (если они есть) ломается оптимизация, но главное — если бы мы использовали этот колбэк внутри других эффектов, они бы бесконечно перезапускались.

А если убрать `callback` из зависимостей `useCallback`?  
Тогда `sendToServer` создастся один раз при старте. Внутри `setTimeout` функция `callback` навсегда закроется (замкнется) на самом первом рендере, где `text` был пустой строкой `""`. Сколько бы пользователь ни писал, спустя секунду на сервер улетит пустая строка. Это классический баг «устаревшего замыкания» (stale closure).

---

## Как `useRef` идеально решает эту дилемму?

Реф (`useRef`) — это коробка, которая хранит ссылку на объект. Изменение содержимого этой коробки (`.current`) не вызывает перерендер, но самое главное — коробка всегда одна и та же между всеми рендерами.

Посмотри, как элегантно работает правильный код:

```typescript
// 1. Создаем коробку при старте
const callbackRef = useRef(callback);

// 2. На КАЖДОМ рендере компонента мы молча обновляем содержимое коробки
// Теперь внутри .current ВСЕГДА лежит самая свежая функция со всеми актуальными стейтами
useEffect(() => {
  callbackRef.current = callback;
}); // без массива зависимостей — выполняется каждый раз

// 3. Возвращаем стабильную функцию
return useCallback((...args) => {
  if (timeoutRef.current) clearTimeout(timeoutRef.current);

  timeoutRef.current = setTimeout(() => {
    // В момент срабатывания таймера мы лезем в коробку .current
    // и берем оттуда самую последнюю версию функции!
    callbackRef.current(...args); 
  }, delay);
}, [delay]); // <-- Функция СТАБИЛЬНА, она пересоздается ТОЛЬКО если изменится delay
```

## Итог:

Переменная `callbackRef` нужна для того, чтобы:

1. Сохранить ссылку на `useDebounceCallback` стабильной. Она не меняется при изменении стейта компонента, её можно смело передавать в дочерние компоненты, не боясь лишних рендеров.
2. Гарантировать актуальность данных. Функция внутри `setTimeout` гарантированно вызовется с самыми последними значениями переменных, избегая бага устаревшего замыкания.

**useThrottleCallback**

```jsx
import { useCallback, useEffect, useRef } from 'react';

export function useThrottleCallback<Args extends any[]>(
  fn: (...args: Args) => void,
  delay: number = 500
) {
  // 1. Избавляемся от багов замыкания через ref
  const fnRef = useRef(fn);
  useEffect(() => {
    fnRef.current = fn;
  });

  const timerRef = useRef<ReturnType<typeof setTimeout> | null>(null);
  
  // Хранилище для последних переданных аргументов
  const lastArgsRef = useRef<Args | null>(null);

  // Очищаем таймер при размонтировании
  useEffect(() => {
    return () => {
      if (timerRef.current) clearTimeout(timerRef.current);
    };
  }, []);

  return useCallback((...args: Args) => {
    // Запоминаем самые свежие аргументы при каждом вызове
    lastArgsRef.current = args;

    // Если таймер уже запущен — просто выходим, блокируя моментальный вызов
    if (timerRef.current) {
      return;
    }

    // Трюк: первый вызов в серии выполняем СИНХРОННО и сразу (так работает классический троттлинг)
    fnRef.current(...lastArgsRef.current);
    lastArgsRef.current = null; // Сбрасываем аргументы, так как уже выполнили

    // Запускаем таймер «периода охлаждения»
    timerRef.current = setTimeout(() => {
      timerRef.current = null;

      // На выходе из интервала проверяем: были ли новые вызовы за это время?
      // Если да — выполняем последний накопившийся вызов
      if (lastArgsRef.current) {
        fnRef.current(...lastArgsRef.current);
        lastArgsRef.current = null;
      }
    }, delay);
  }, [delay]);
}
```


Подробнее:
1. [Теория, пример функций](https://dev.to/andreysm/ispolzuiem-throttle-i-debounce-v-react-3cn9)
2. [Пример реализации функций](https://medium.com/@vinchik/throttle-debounce-%D1%80%D0%B0%D0%B7%D0%BD%D0%B8%D1%86%D0%B0-%D1%81-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%B0%D0%BC%D0%B8-%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8-%D0%B8-%D0%B4%D0%B5%D0%BC%D0%BE-5133b40c01d2)
3. [Пример реализации useThrottleValue](https://dev.to/loonywizard/react-usethrottle-hook-87h)
4. [Пример хука](https://learnersbucket.com/examples/interview/usethrottle-hook-in-react/)
5. [Пример хука 2](https://javascript.plainenglish.io/debounce-search-feature-in-react-46dfc31aede2)
