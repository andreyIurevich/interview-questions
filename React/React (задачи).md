По нажатию кнопки список переворачивается.
Если в функцию обновляющую состояние передать старую ссылку на объект(массив) перерендера не произойдёт. Нужно создавать новый массив.

```tsx
type ListItem = {  
  id: number,  
  name: string,  
}  
  
function App() {  
  const [list, setList] = useState<ListItem[]>([  
    {  
      id: 1,  
      name: 'item 1',  
    },  
    {  
      id: 2,  
      name: 'item 2',  
    }  
  ])  
  
  const handleClick = () => {  
    // setList((old) => old.reverse()) если будем использовать старую ссылку на
    // массив перерендера компонента не произойдёт
    setList((old) => [...old].reverse())
  }  
  
  return (  
    <>  
      <button onClick={handleClick}>Reverse List</button>  
      <ul>        
	      {list.map(item => <li key={item.id}>{item.name}</li>)}  
      </ul>  
    </>  
	)  
}
```

Порядок вызова функций:

```jsx
import { useState, useEffect } from 'react';

export default function App() {
  console.log('Hack Frontend 1');

  useEffect(() => {
    console.log('Hack Frontend 2');
  }, []);

  return <Child />;
}

function Child() {
  console.log('Hack Frontend 3');

  useEffect(() => {
    console.log('Hack Frontend 4');
  }, []);

  return <div>Hack Frontend</div>;
}
```

1. `Hack Frontend 1`: React начинает фазу рендера с родительского компонента `App` и выполняет его тело.
2. `Hack Frontend 3`: В процессе чтения JSX компонента `App` React видит `<Child />` и сразу переходит к рендеру дочернего компонента, выполняя его тело.
3. `Hack Frontend 4`: Фаза рендера завершена, начинается фаза фиксации (commit) и монтирования (mount). React идет снизу вверх по дереву компонентов. Сначала срабатывают эффекты `useEffect` у дочерних элементов.
4. `Hack Frontend 2`: В последнюю очередь срабатывают эффекты `useEffect` родительского компонента `App`.

Вызовется ли `useEffect` в дочернем компоненте при перерендере родителя — **зависит от массива зависимостей `useEffect`**.

В твоем примере у `Child` указан пустой массив зависимостей `[]`. Это значит, что при перерендере родителя `useEffect` в `Child` **не вызовется**.

Что произойдет при перерендере `App`

Если `App` перерендерится, порядок логов будет следующим:

1. **`Hack Frontend 1`** — тело `App` выполняется заново.
2. **`Hack Frontend 3`** — `Child` автоматически перерендеривается, так как обновился его родитель, и его тело тоже выполняется заново.

Логи `Hack Frontend 4` и `Hack Frontend 2` **не появятся**.

Почему эффекты не сработают?

- **Фаза рендера** (`console.log` в теле функции) запускается при **каждом** перерендере.
- **Фаза эффектов** (`useEffect`) с пустым массивом `[]` запускается **только один раз** — при монтировании (добавлении компонента на экран).

Таймер:

```jsx
export default function App() {  
  const [elapsedTime, setElapsedTime] = useState<number>(0);  
  const [inProgress, setInProgress] = useState<boolean>(false);  
  const startTime = useRef<number | null>(null);  
  const intervalId = useRef<number | null>(null);  
  
  useEffect(() => {  
    if (inProgress) {  
      intervalId.current = setInterval(() => {  
        if (!startTime.current) startTime.current = Date.now();  
        const timePassed = Date.now() - startTime.current;  
        setElapsedTime(Math.floor(timePassed / 1000));  
      }, 1000);  
    }  
  
    return () => {  
      if (intervalId.current) clearInterval(intervalId.current);  
    };  
  }, [inProgress]);  
  
  const handleProgressChange = () => {  
    if (!inProgress && startTime.current)  
      startTime.current = Date.now() - elapsedTime * 1000;  
    setInProgress(prev => !prev);  
  };  
  
  const handleClear = () => {  
    if (intervalId.current) clearInterval(intervalId.current);  
    setElapsedTime(0);  
    startTime.current = null;  
    setInProgress(false);  
  };  
  
  const formattedTime = (time: number): string => {  
    const minutes = Math.trunc(time / 60);  
    const seconds = time % 60;  
    return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;  
  };  
  
  return (  
    <div style={{ width: '200px' }}>  
      <button        onClick={handleProgressChange}  
      >  
        {inProgress ? 'Стоп' : 'Старт'}  
      </button>  
      <button        onClick={handleClear}  
      >  
        Сбросить  
      </button>  
      <p>секунд: {formattedTime(elapsedTime)}</p>  
    </div>  );  
}
```

Главная причина, почему `setInterval` не гарантирует точность, кроется в ==однопоточной архитектуре JavaScript и механизме работы Event Loop (цикла событий)==.

Браузер не умеет выполнять несколько задач одновременно в одном потоке. Вот основные факторы, которые ломают точность `setInterval`:

## 1. Очередь задач (Task Queue) и блокировка потока

Когда вы пишете `setInterval(fn, 1000)`, это не означает, что функция выполнится ровно через 1000 мс. Это означает лишь то, что через 1000 мс браузер попытается добавить функцию в очередь задач.

- Если главный поток (Call Stack) в этот момент занят тяжелыми вычислениями (например, обработкой большого массива или сложным рендером), ваша функция будет покорно ждать своей очереди.
- Время ожидания в очереди добавляется к задержке.

## 2. Замораживание фоновых вкладок (Throttling)

Современные браузеры (Chrome, Safari, Firefox) жестко экономят батарею ноутбука и ресурсы процессора.

- Как только пользователь переключается на другую вкладку, браузер искусственно снижает частоту срабатывания таймеров.
- В фоновом режиме интервал может принудительно замедлиться до 1 раза в секунду (1000 мс) или даже до 1 раза в минуту для «спящих» вкладок. Для секундомера это означает мгновенную потерю точности.

## 3. Накопление ошибки (Дрейф времени)

`setInterval` не учитывает, сколько времени выполнялся код внутри самой функции.  
Если интервал настроен на 10 мс, а код внутри функции выполняется 4 мс, то из-за микрозадержек Event Loop реальный шаг может составить 11 или 12 мс. Со временем эти миллисекунды суммируются, и уже через пару минут ваш таймер начнет отставать от реального времени на несколько секунд. Это называется дрейфом таймера.

---

## Наглядная иллюстрация работы Event Loop:

```unset
Время (мс):   0 -------- 1000 -------- 2000 -------- 3000
Интервал:    [Старт]      [Таск 1]     [Таск 2]     [Таск 3]

                          |            |
Тяжелый код: ------------|БЛОК ПОТОКА|------------------------>

                          |            |
Реальный вызов:           Опоздал! ->  Вызвался вовремя
```

Именно поэтому для любых точных измерений (секундомеры, часы, фитнес-трекеры) всегда берут разницу во времени через `Date.now()` или `performance.now()`, так как системные часы компьютера не зависят от загруженности JavaScript-потока.