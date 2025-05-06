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

