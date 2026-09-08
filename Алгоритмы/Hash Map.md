**Условие:**  
Дан массив строк `strs`. Сгруппируйте **анаграммы** вместе. Вы можете вернуть ответ в любом порядке.

_Анаграмма_ — это слово, полученное путем перестановки букв другого слова, обычно с использованием всех исходных букв ровно один раз (например, `"eat"`, `"tea"` и `"ate"` — это анаграммы).

**Пример:**

- **Вход:** `strs = ["eat", "tea", "tan", "ate", "nat", "bat"]`
- **Выход:** `[["bat"], ["nat", "tan"], ["ate", "eat", "tea"]]`

O(N * M * log M)

```js
function groupAnagrams(strs) {
  const map = new Map();

  for (const str of strs) {
    // 1. Формируем ключ сортировкой
    const key = str.split('').sort().join('');
    
    // 2. Инициализируем массив, если ключа еще нет
    if (!map.has(key)) {
      map.set(key, []);
    }
    
    // 3. Просто пушим элемент за O(1) без копирования массива
    map.get(key).push(str);
  }

  // Возвращаем массив массивов
  return Array.from(map.values());
}
```

O(N * M)

```js
function groupAnagramsFaster(strs) {
  const map = new Map();

  for (const str of strs) {
    // Создаем массив счетчиков для 26 букв (от 'a' до 'z')
    const count = new Array(26).fill(0);
    
    for (let i = 0; i < str.length; i++) {
      // Получаем код символа относительно 'a'
      const charCode = str.charCodeAt(i) - 97; // 97 — это код буквы 'a'
      count[charCode]++;
    }
    
    // Превращаем массив в строку-ключ, например: "1,0,2,0,0..."
    const key = count.join(',');

    if (!map.has(key)) {
      map.set(key, []);
    }
    map.get(key).push(str);
  }

  return Array.from(map.values());
}
```

Задача: Сумма двух чисел (Two Sum)

**Условие:**  
Дан массив целых чисел `nums` и целое число `target`. Найдите **индексы** двух чисел в этом массиве таких, чтобы их сумма была равна `target`.

**Важные условия:**

- Каждый входной массив имеет **ровно одно решение**.
- Вы не можете использовать один и тот же элемент дважды (то есть индексы двух чисел должны быть разными).
- Порядок индексов в ответе не имеет значения.

**Примеры для проверки:**

- `nums = [2, 7, 11, 15]`, `target = 9` → Ответ: `[0, 1]` (так как `nums[0] + nums[1] === 9`).
- `nums = [3, 2, 4]`, `target = 6` → Ответ: `[1, 2]` (так как `nums[1] + nums[2] === 6`).
- `nums = [3, 3]`, `target = 6` → Ответ: `[0, 1]`.

```js
function twoSum(nums, target) {
  const numsMap = new Map();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    // Если мы уже встречали нужное нам дополнение — решение найдено!
    if (numsMap.has(complement)) {
      return [numsMap.get(complement), i];
    }

    // Если не нашли, просто запоминаем текущее число и его индекс
    numsMap.set(nums[i], i);
  }

  return null; // На случай, если решения нет (хотя по условию оно всегда есть)
}
```