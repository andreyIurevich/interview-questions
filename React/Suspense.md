`Suspense` в React — это механизм **декларативной обработки ожидания**: загрузки кода, данных или других асинхронных ресурсов. Он позволяет **описывать состояние “пока ждём” прямо в JSX**, а не вручную через флаги `isLoading`.

### Идея Suspense (ключевая мысль)

Компонент **может “приостановить” рендер**, если нужные данные ещё не готовы, а React в это время покажет `fallback`.

```jsx
<Suspense fallback={<Spinner />}>
  <MyComponent />
</Suspense>
```

**Самый распространённый кейс — Code Splitting**

React.lazy + Suspense
```jsx
const Profile = React.lazy(() => import('./Profile'));

function App() {
  return (
    <Suspense fallback={<div>Загрузка…</div>}>
      <Profile />
    </Suspense>
  );
}
```

Что происходит:

1. Бандл `Profile` ещё не загружен
2. React “приостанавливает” рендер
3. Показывает `fallback`
4. Когда код загружен — рендерит компонент

✅ Это **production-ready** и основной сценарий сегодня

Suspense для данных (важное уточнение ⚠️)

```jsx
useEffect(() => {
  fetch(...)
}, []);
```

`Suspense` **не работает с обычными `useEffect`**

### ✅ Работает только с:

- библиотеками, которые **интегрированы с Suspense**
- или с **throw Promise** (низкоуровневый API)

Примеры:
- Relay
- Apollo (experimental)
- TanStack Query (`suspense: true`)
- React Server Components

**Suspense + ErrorBoundary**

```jsx
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Loader />}>
    <Profile />
  </Suspense>
</ErrorBoundary>
```

