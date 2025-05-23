**React Reconciliation** — это механизм сравнения текущего виртуального DOM с предыдущей версией, чтобы понять, **что именно изменилось**, и **обновить только нужные части** реального DOM.

Алгоритм работы:
1. Монтирование (mount) вызывает рендеринг приложения.
2. Получившийся DOM вставляется в реальный DOM целиком, так как там еще ничего нет. А виртуальный DOM, в свою очередь, сохраняется внутри React для последующего обновления.
3. Изменение состояния приводит к вычислению нового виртуального DOM.
4. Вычисляется разница между старым виртуальным DOM и новым.
5. Разница применяется к реальному DOM.

React Fiber — это переписанный алгоритм согласования, представленный в React 16, который позволяет управлять рендерингом эффективней старого стекового алгоритма.

До React 16 использовался стековый рекурсивный алгоритм. Проблемы заключалась в том, что, если один компонент рендерился долго, UI мог «зависнуть». Отсутствовали приоритеты рендеринга, а также не было возможности прервать рендеринг, например, если пользователь вводил текст, то React мог продолжать рендеринг и не реагировать на ввод.

**Fiber решил все эти проблемы и теперь у нас есть**:

- **Приоритизация задач**. Сначала перерендериваем более важные части, потом менее.
- **Разбиение рендеринга на чанки** (React может рендерить части дерева отдельно).
- **Асинхронный рендеринг** (React может прерывать выполнение и продолжать позже).

#### Концепции Fiber

**Каждый узел VDOM — это Fiber Node**  
В отличие от старого алгоритма, React теперь хранит в памяти две версии дерева. Первая — это текущее (current) дерево — то, что сейчас в DOM, вторая — рабочее (work‑in‑progress) дерево — новая версия, которую React готовит.

**Рендеринг разбивается на этапы**  
Рендеринг разбивается на этапы с использованием «chunk‑based rendering». React может приостанавливать рендеринг для выполнения более важных задач, таких как пользовательский ввод, и продолжить его позже.

#### Фазы работы Fiber

**Render Phase (Diffing, вычисление изменений)**

- Проходит асинхронно и может быть прервано.    
- Создаётся новое Work-In-Progress (WIP) Fiber-дерево.

**Commit Phase (Обновление DOM)**

- Выполняется **синхронно** и **не может быть прервано**.
- Применяются изменения к DOM и вызываются componentDidMount, componentDidUpdate, useEffect.

[Подробнее](https://habr.com/ru/companies/gnivc/articles/899878/)
