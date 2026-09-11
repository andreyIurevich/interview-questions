Контейнер с наибольшим количеством воды (Container With Most Water)

Вам дан массив целых чисел `height` длины n, где `height[i]` представляет собой высоту вертикальной линии, проведенной в точке с координатами (i, height[i]).

Найдите две линии, которые вместе с осью абсцисс образуют контейнер, содержащий **наибольшее количество воды**. Функция должна вернуть **максимальный объем воды**, который может вместить этот контейнер.

Решение без оптимизации (рассчёт площади на каждом шаге цикла):

```js
function maxArea(height) {
	let minHeight,left = 0,
	right = height.length - 1,
	maxArea = 0;

	while (left < right) {
		minHeight = Math.min(height[left], height[right]);
		maxArea = Math.max(maxArea, minHeight * (right - left));
		
		if (height[right] < height[left]) {
			right--;
		} else {
			left++;
		}
	}

return maxArea;
}
```

Решение с оптимизацией, пропуск рассчёта площадей для линий меньших чем пред-я:

```js
function maxAreaOptimized(height) {
  let left = 0, right = height.length - 1;
  let maxArea = 0;

  while (left < right) {
    const leftVal = height[left];
    const rightVal = height[right];
    
    // Считаем площадь текущего контейнера
    const currentArea = Math.min(leftVal, rightVal) * (right - left);
    if (currentArea > maxArea) maxArea = currentArea;

    // Вместо простого инкремента/декремента пропускаем все линии, 
    // которые не смогут дать большую высоту
    if (leftVal < rightVal) {
      while (left < right && height[left] <= leftVal) left++;
    } else {
      while (left < right && height[right] <= rightVal) right--;
    }
  }

  return maxArea;
}
```

Наибольшая подстрока без повторяющихся символов (Longest Substring Without Repeating Characters)

**Условие:**  
Дана строка `s`. Найдите длину **самой длинной подстроки**, которая не содержит повторяющихся символов.

**Примеры для проверки:**

- `s = "abcabcbb"` → Ответ: `3` (подстрока `"abc"`)
- `s = "bbbbb"` → Ответ: `1` (подстрока `"b"`)
- `s = "pwwkew"` → Ответ: `3` (подстрока `"wke"`, обрати внимание, что `"pwke"` не подходит, так как это подпоследовательность, а нам нужна непрерывная _подстрока_).

```js
function lengthOfLongestSubstring(s) {
	const charMap = new Map();
	let left = 0, right = 0, maxLen = 0;

	while (right < s.length) {
		if (charMap.has(s[right])) {
			left = Math.max(left, charMap.get(s[right]) + 1);
		}

		charMap.set(s[right], right);
		maxLen = Math.max(maxLen, right - left + 1);

		right++;
	}

	return maxLen;
}
```

Задача: Сумма трех чисел (3Sum)

**Условие:**  
Дан массив целых чисел `nums`. Найдите все **уникальные триплеты** (тройки чисел) `[nums[i], nums[j], nums[k]]` такие, чтобы:

1. `i !== j`, `i !== k`, `j !== k` (индексы элементов разные).
2. `nums[i] + nums[j] + nums[k] === 0` (сумма равна нулю).

**Важное требование:**  
В ответе не должно быть дублирующихся троек чисел. Например, если ты нашел `[-1, 0, 1]`, то тройка `[0, -1, 1]` или еще одна `[-1, 0, 1]` в ответе присутствовать не должна. Порядок троек и элементов внутри них значения не имеет.

**Пример:**

- **Вход:** `nums = [-1, 0, 1, 2, -1, -4]`
- **Выход:** `[[-1, -1, 2], [-1, 0, 1]]`

```js
function threeSum(nums) {
  let left, right, currSum, res = [];

  // 1. Сортируем массив по возрастанию
  nums.sort((a, b) => a - b);

  for (let i = 0; i < nums.length - 2; i++) {
    // ЗАЩИТА 1: Если текущее число такое же, как предыдущее, 
    // мы его уже полностью обработали на прошлом шаге. Пропускаем!
    if (i > 0 && nums[i] === nums[i - 1]) {
      continue;
    }

    left = i + 1;
    right = nums.length - 1;

    while (left < right) {
      currSum = nums[i] + nums[left] + nums[right];

      if (currSum === 0) {
        res.push([nums[i], nums[left], nums[right]]);
        
        // ЗАЩИТА 2: Пропускаем дубликаты для левого указателя
        while (left < right && nums[left] === nums[left + 1]) {
          left++;
        }
        // ЗАЩИТА 3: Пропускаем дубликаты для правого указателя
        while (left < right && nums[right] === nums[right - 1]) {
          right--;
        }

        // Вместо break просто сдвигаем оба указателя к центру 
        // и продолжаем искать другие пары для текущего nums[i]
        left++;
        right--;

      } else if (currSum > 0) {
        right--;
      } else {
        left++;
      }
    }
  }

  return res;
}
```