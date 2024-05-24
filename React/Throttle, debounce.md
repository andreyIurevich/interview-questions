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

```jsx
function useThrottle(fn, delay) {
	const timerRef = useRef<number | null>(null);
	
	const throttledFn = useCallback((...args) => {
		if (timerRef.current) {
			return;
		}
		timerRef.current = setTimeout(() => {
			fn(args);
			timerRef.current = null;
		}, delay);
	}, []);
	
	return throttledFn;
}
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

Подробнее:
1. [Теория, пример функций](https://dev.to/andreysm/ispolzuiem-throttle-i-debounce-v-react-3cn9)
2. [Пример реализации функций](https://medium.com/@vinchik/throttle-debounce-%D1%80%D0%B0%D0%B7%D0%BD%D0%B8%D1%86%D0%B0-%D1%81-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%B0%D0%BC%D0%B8-%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8-%D0%B8-%D0%B4%D0%B5%D0%BC%D0%BE-5133b40c01d2)
3. [Пример реализации useThrottleValue](https://dev.to/loonywizard/react-usethrottle-hook-87h)
4. [Пример хука](https://learnersbucket.com/examples/interview/usethrottle-hook-in-react/)