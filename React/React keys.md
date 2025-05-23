Атрибут `key` в React используется для идентификации элементов в списках. Он помогает React эффективно отслеживать, какие элементы были изменены, добавлены или удалены, при повторном рендеринге списка.

#### Основное предназначение `key`:

- **Оптимизация производительности**: React использует `key`, чтобы быстрее сравнивать старый и новый виртуальные DOM-элементы. Если `key` одинаковый — элемент сохраняется, если разный — пересоздаётся.

- **Корректное поведение компонентов**: Если `key` уникальный и стабильный, React не будет ошибочно «переносить» состояние одного элемента на другой при изменении списка.

#### Правила:

- `key` должен быть **уникальным** среди соседних элементов.

- Лучше использовать **уникальный ID** элемента, если он есть.

- **Не рекомендуется использовать индекс массива** как `key`, особенно если список может изменяться (удаление, вставка и т.п.), так как это может привести к неправильному поведению.

#### Пример работы keys с значением уникальных id

Предположим - есть массив из 3-х объектов, описывающих пользователей и состоящих из 3-х полей - id и name. Компонент контейнер, содержащий список пользователей и компонент PureComponent User (отображающий информацию о пользователе). Компонент User содержит 3 метода жизненного цикла - componentDidMount, componentDidUpdate и componentWillUnmount. Тело каждого метода будет содержать код вывода названия соответствующего метода. 

В исходный массив внесём след. изменения - у первого объекта изменим имя пользователя, у второго id, а третий оставим без изменений. 

Вывод в консоли покажет следующие результаты:

-> WillUnmount 

-> DidUpdate

-> DidMount

##### Пример использования key со значениями index массива

Тот же пример, только удалим второй элемент массива users. В этом случае в консоли мы увидим, что React удалил последний компонент, а второй переименовал (хотя, у нас удалился второй элемент). Т.к. у второго элемента key не изменился (index = 1), то второй элемент он не удаляет и, соотвественно, оставляет его состояние, если оно есть. Хотя вместо второго элемента должен быть третий.


Стоит использовать index массива в качестве значений keys, если выполняются три условия:

1. Список и элементы списка статичны и не вычисляются и не изменяются с течением времени
2. Элементы списка не имеют id
3. Сам список не фильтруется и не изменяется.
