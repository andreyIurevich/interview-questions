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