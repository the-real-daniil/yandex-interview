# Этап 1: платформа

## Цепь на шее блестит

### Задача

Что будет выведено в консоль?

```js
Promise.resolve(1)
  .then((x) => x + 1)
  .then((x) => {
    throw x;
  })
  .then((x) => console.log(x))
  .catch((err) => console.log(err))
  .then((x) => Promise.resolve(x))
  .catch((err) => console.log(err))
  .then((x) => console.log(x));
```

### Решение

Ответ:

```js
Promise.resolve(1)
  .then((x) => x + 1)
  .then((x) => {
    throw x;
  })
  .then((x) => console.log(x))
  .catch((err) => console.log(err)) // 2
  .then((x) => Promise.resolve(x))
  .catch((err) => console.log(err))
  .then((x) => console.log(x)); // undefined
```

### Объяснение

- `Promise.resolve(1)` — создаёт успешно выполненный промис с результатом 1.
- `.then(x => x + 1)` — принимает 1, возвращает 2, передаёт дальше.
- `.then(x => { throw x })` — выбрасывает исключение 2, промис переходит в состояние `rejected`.
- `.then(x => console.log(x))` — пропускается, так как предыдущий промис `rejected`.
- `.catch(err => console.log(err))` — ловит ошибку 2 и выводит её в консоль.
- `.then(x => Promise.resolve(x))` — catch вернул `undefined`, поэтому `x` равно `undefined`.
- `.catch(err => console.log(err))` — не выполняется, так как нет ошибки.
- `.then(x => console.log(x))` — `x === undefined`, выводится `undefined`.

## Курочка карри

### Задача

Реализовать функцию каррирования

```js
function curry(fn) {}

// Тестирование

function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);

console.log(curriedSum(1, 2, 3)); // 6
console.log(curriedSum(1, 2)(3)); // 6
console.log(curriedSum(1)(2, 3)); // 6
console.log(curriedSum(1)(2)(3)); // 6
```

### Решение

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    } else {
      return (...nextArgs) => curried(...args, ...nextArgs);
    }
  };
}

// Тестирование

function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);

console.log(curriedSum(1, 2, 3)); // 6
console.log(curriedSum(1, 2)(3)); // 6
console.log(curriedSum(1)(2, 3)); // 6
console.log(curriedSum(1)(2)(3)); // 6
```

### Объяснение

- Принимаем функцию fn и возвращаем новую обёртку.
- Собираем аргументы при каждом вызове.
- Когда аргументов достаточно, вызываем оригинальную функцию fn.
- Если аргументов недостаточно, возвращаем новую функцию для дальнейшего каррирования.

## Джонни, они на деревьях

### Задача

Написать функцию, которая склеивает переданные строки через разделитель

```js
function strjoin(separator, ...strings) {}

// Тестирование
console.log(strjoin(".", "a", "b", "c")); // 'a.b.c'
console.log(strjoin("-", "a", "b", "c", "d", "e", "f")); // 'a-b-c-d-e-f'
```

### Решение

```js
function strjoin(separator, ...strings) {
  return strings.join(separator);
}

// Тестирование
console.log(strjoin(".", "a", "b", "c")); // 'a.b.c'
console.log(strjoin("-", "a", "b", "c", "d", "e", "f")); // 'a-b-c-d-e-f'
```

### Объяснение

- Принимаем произвольное количество аргументов.
- Первый аргумент — разделитель.
- Оставшиеся аргументы — элементы для объединения.
- Используем `.join(separator)` для объединения.

## Асинк эбаут ит

### Задача

```js
import asyncAuth from ".";

/**
 * `asyncAuth()` function receives a callback into which
 * an error may be passed (argument 1) or
 * data from backend (argument 2).
 *
 * You need to implement an `auth()` function
 * which executes `asyncAuth()`, but returns Promise.
 *
 * @returns {Promise}
 */
function auth() {
  asyncAuth((error, data) => {});
}

/**
 * `tryAuth()` function uses `auth()` and, in case of errors,
 * makes N additional attempts.
 *
 * @returns {Promise}
 */
function tryAuth(n) {}
```

### Решение

```js
function asyncAuth(cb) {
  setTimeout(() => cb(undefined, { data: "text" }), 1000);
}

function asyncAuthError(cb) {
  setTimeout(() => cb(new Error("error")), 1000);
}

function auth() {
  return new Promise((resolve, reject) => {
    asyncAuth((error, data) => {
      if (error) reject(error);
      else resolve(data);
    });
  });
}

function tryAuth(n) {
  return new Promise((resolve, reject) => {
    function attempt(remainingTries) {
      auth()
        .then(resolve)
        .catch((error) => {
          if (remainingTries > 0) {
            attempt(remainingTries - 1);
          } else {
            reject(error);
          }
        });
    }

    attempt(n);
  });
}

// Тестирование
tryAuth(3)
  .then((data) => console.log("Success:", data))
  .catch((error) => console.log("Failed:", error));
```

### Объяснение

- `auth()` создаёт Promise, внутри вызывает `asyncAuth`.
- Если `asyncAuth` вызывает коллбэк с ошибкой → `reject(error)`.
- Если данных нет ошибки → `resolve(data)`.
- `tryAuth(n)` рекурсивно вызывает `auth()`, уменьшая `n`, пока не закончится лимит попыток.

## Преврати верблюда в змею

### Задача

```js
// Необходимо написать функцию, которая преобразует строку
// из CamelCase в snake_case.

function camelToSnake(text) {}

console.log(camelToSnake("updatedAt")); // updated_at
console.log(camelToSnake("XmlHttpRequest")); // xml_http_request
console.log(camelToSnake("XmlHttpRequestI")); // "xml_http_request_i"
```

### Решение (алгоритмическое)

```js
function camelToSnake(text) {
  let result = "";

  for (let i = 0; i < text.length; ++i) {
    const char = text[i];

    if (char === char.toUpperCase() && i !== 0) {
      result += "_";
    }

    result += char.toLowerCase();
  }

  return result;
}

// Тестирование
console.log(camelToSnake("updatedAt")); // "updated_at"
console.log(camelToSnake("XmlHttpRequest")); // "xml_http_request"
console.log(camelToSnake("XmlHttpRequestI")); // "xml_http_request_i"
```

### Объяснение

- Создаём пустую строку `result`.
- Проходим по каждому символу входной строки:
- Если символ заглавный и не первый — добавляем `_` перед ним.
- Добавляем символ в нижнем регистре к `result`.
- Возвращаем итоговую строку.

### Решение (через регулярные выражения)

```js
function camelToSnake(text) {
  return text.replace(/([a-z])([A-Z])/g, "$1_$2").toLowerCase();
}

// Тестирование
console.log(camelToSnake("updatedAt")); // "updated_at"
console.log(camelToSnake("XmlHttpRequest")); // "xml_http_request"
console.log(camelToSnake("XmlHttpRequestI")); // "xml_http_request_i"
```

### Объяснение

- `replace(/([a-z])([A-Z])/g, "$1\_$2")` — находит места, где маленькая буква ([a-z]) стоит перед заглавной буквой ([A-Z]).
- Вставляет подчёркивание между ними.
- `toLowerCase()` переводит всю строку в нижний регистр.

## группировОчка

### Задача

```js
// Необходимо реализовать метод groupBy, расширяющий стандартные методы массивов.
// Метод должен возвращать сгруппированную версию массива - объект,
// в котором каждый ключ является результатом выполнения переданной функции fn(arr[i]),
// а каждое значение - массивом, содержащим все элементы исходного массива с этим ключом.

// Пример 1
const array1 = [{ id: 1 }, { id: 1 }, { id: 2 }];

const fn = (item) => item.id;

array1.groupBy(fn);
// {
//   1: [{ id: 1 }, { id: 1 }],
//   2: [{ id: 2 }]
// }

// Пример 2
const array2 = [1, 2, 3];
array2.groupBy(String); // { "1": [1], "2": [2], "3": [3] }

// Пример 3
const array3 = [3.3, 0.5, 1.4];
array3.groupBy(Math.round); // { "3": [3.3], "1": [0.5], "1": [1.4] }
```

### Решение

```js
Array.prototype.groupBy = function (fn) {
  return this.reduce((acc, item) => {
    const key = fn(item);

    if (key in acc) {
      acc[key].push(item);
    } else {
      acc[key] = [item];
    }

    return acc;
  }, {});
};

const array1 = [{ id: 1 }, { id: 1 }, { id: 2 }];
console.log(array1.groupBy((item) => item.id)); // { 1: [{ id: 1 }, { id: 1 }], 2: [{ id: 2 }] }

const array2 = [1, 2, 3];
console.log(array2.groupBy(String)); // { "1": [1], "2": [2], "3": [3] }

const array3 = [3.3, 0.5, 1.4];
console.log(array3.groupBy(Math.round)); // { "3": [3.3], "1": [0.5, 1.4] }
```

### Объяснение

- Расширяем прототип массива, добавляя `groupBy().`
- Используем `reduce()` для накопления групп.
- Ключ вычисляется с помощью переданной функции `fn(item)`.
- Если ключ отсутствует в объекте, создаём массив.
- Добавляем текущий элемент в массив.

## Время ограничено

### Задача

```js
/*
Дана асинхронная функция fn и время t в миллисекундах, нужно вернуть новую версию этой функции,
выполнение которой ограничено заданным временем. Функция fn принимает аргументы,
переданные в эту новую функцию.

Возвращаемая функция работает по следующим правилам:

- если fn выполнится за заданное время t, то функция резолвит полученные данные
- если fn не выполнится за заданное время t, то функция реджектит строку "Time limit exceeded"
*/

/**
 * @param {Function} fn
 * @param {number} t
 * @return {Function}
 */
const timeLimited = function (fn, t) {
  // your code
};

const asyncF = () => {};
```

### Решение

```js
const timeLimited = function (fn, t) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      const timerPromise = new Promise((reject) =>
        setTimeout(() => reject("Time limit exceeded"), t)
      );

      const resultPromise = fn(...args);

      Promise.race([timerPromise, resultPromise])
        .then(resolve)
        .catch(reject)
        .finally(() => clearTimer(timerPromise));
    });
  };
};

// Тестирование
const asyncF = (x) =>
  new Promise((resolve) => setTimeout(() => resolve(x * 2), 1000));

const limitedFunc = timeLimited(asyncF, 500);

limitedFunc(5)
  .then(console.log) // Не успеет, будет "Time limit exceeded"
  .catch(console.error);

const limitedFunc2 = timeLimited(asyncF, 1500);

limitedFunc2(5)
  .then(console.log) // Успеет, выведет 10
  .catch(console.error);
```

### Объяснение

- Возвращаем асинхронную функцию, которая принимает аргументы и прокидывает их в `fn()`
- Создаем `Promise.race`, в котором соревнуются два промиса: `fn()` и таймер с заданным временем `t` и реджектом ошибки `"Time limit exceeded"`
- Если быстрее выполняется таймер, то функция реджектнет ошибку
- Если быстрее выполнится `fn`, то функция резолвнет результат

## Интерсекция

### Задача

```js
/*
Даны два отсортированных списка с интервалами присутствия
пользователей в онлайне в течение дня. Начало интервала строго меньше конца.
Нужно вычислить интервалы, когда оба пользователя были в онлайне.
Интервалы указаны в часах, считаем что могут быть часы от 0 до 24.
*/

intersection(
  [
    [8, 12],
    [17, 22],
  ],
  [
    [5, 11],
    [14, 18],
    [20, 23],
  ]
); // [[8, 11], [17, 18], [20, 22]]

intersection(
  [
    [9, 15],
    [18, 21],
  ],
  [
    [10, 14],
    [21, 22],
  ]
); // [[10, 14]]

function intersection(user1, user2) {}
```

### Решение

```js

function intersection(user1, user2) {
  let i = 0
  let j = 0
  const result = []

  while (i < user1.length && j < user2.length) {
    const [start1, end1] = user1[i]
    const [start2, end2] = user2[j]

    const start = Math.max(start1, start2)
    const end = Math.min(end1, end2)

    if (start < end) {
      result.push([start, end])
    }

    end1 < end2 ? ++i : ++j
  }

  return result
}

console.log(intersection(
  [
    [8, 12],
    [17, 22],
  ],
  [
    [5, 11],
    [14, 18],
    [20, 23],
  ];
)) // [8, 11], [17, 18], [20, 22]

console.log(intersection(
  [
    [9, 15],
    [18, 21],
  ],
  [
    [10, 14],
    [21, 22],
  ];
)) // [10, 14]

console.log(intersection(
  [
    [10, 14],
    [21, 24],
  ],
  [
    [9, 15],
    [18, 21],
    [22, 24],
  ];
)) // [10, 14], [22, 24]
```

### Объяснение

- Два указателя (`i` и `j`) проходят по двум спискам интервалов.
- Определяем пересечение интервалов:
- Начало пересечения: `max(start1, start2)`.
- Конец пересечения: `min(end1, end2)`.
- Если `start < end`, значит есть пересечение.
- Двигаем указатель:
  - Если `end1 < end2`, двигаем `i`.
  - Иначе двигаем `j`.

## Спокойной ночи

### Задача

```js
/**
 * Необходимо написать асинхронную функцию,
 * которая будет "спать" заданное количество миллисекунд,
 * а потом успешно завершаться
 */

function sleep(duration) {}

const startTime = Date.now();

sleep(100).then(() => console.log(Date.now() - startTime));
// Выведет примерно 100
```

### Решение

```js
function sleep(duration) {
  return new Promise((resolve) => setTimeout(resolve, duration));
}

// Тестирование
const startTime = Date.now();

sleep(100).then(() => {
  console.log(Date.now() - startTime); // Выведет примерно 100
});
```

### Объяснение

- Создаём `Promise`, который завершится через duration миллисекунд.
- Передаём `resolve` в `setTimeout`, чтобы промис завершился по таймеру.
- После `await` или `.then()` выполнение кода продолжается.

###

## Тайм сквэр чи шо?

### Задача

```js
const square = (x) => x * x;
const times2 = (x) => x * 2;
const sum = (a, b) => a + b;

console.log(compose(square, times2)(2) === square(times2(2)));
console.log(compose(square, times2, sum)(3, 4) === square(times2(sum(3, 4))));

function compose(...funcs) {}
```

### Решение (императивное)

```js
function compose(...funcs) {
  let result;

  return function (...args) {
    for (let i = funcs.length - 1; i >= 0; --i) {
      const f = funcs[i];

      const currentArgs = i === funcs.length - 1 ? args : [result];

      result = f(...currentArgs);
    }

    return result;
  };
}

// Тестирование
const square = (x) => x * x;
const times2 = (x) => x * 2;
const sum = (a, b) => a + b;

console.log(compose(square, times2)(2) === square(times2(2))); // true
console.log(compose(square, times2, sum)(3, 4) === square(times2(sum(3, 4)))); // true
```

### Объяснение

- Создаем переменную `result`, в которой будем накапливать результат
- Возвращаем функцию, которая принимает аргументы.
- Внутри функции проходимся циклом по переданным функциям в обратном порядке и вызываем их с нужным аргументом
- Возвращаем результат

### Решение (более декларативное)

```js
function compose(...funcs) {
  return (...args) => funcs.reduceRight((acc, f) => [f(...acc)], args)[0];
}

// Тестирование
const square = (x) => x * x;
const times2 = (x) => x * 2;
const sum = (a, b) => a + b;

console.log(compose(square, times2)(2) === square(times2(2))); // true
console.log(compose(square, times2, sum)(3, 4) === square(times2(sum(3, 4)))); // true
```

### Объяснение

- Возвращаем функцию, которая принимает аргументы.
- `reduceRight` проходит по функциям с конца к началу.
- `fn(...acc)` передаёт результат текущей функции в следующую.
- Используем массив `[fn(...acc)]`, чтобы поддерживать вариативное число аргументов.

## Панаграммчик

### Задача

```js
// Вам задана строка, состоящая из пробелов и латинских букв.
// Строка называется панграммой, если она содержит каждую из 26 латинских
// букв хотя бы раз. Определите является ли строка панграммой.
const LETTERS = [
  "A",
  "B",
  "C",
  "D",
  "E",
  "F",
  "G",
  "H",
  "I",
  "J",
  "K",
  "L",
  "M",
  "N",
  "O",
  "P",
  "Q",
  "R",
  "S",
  "T",
  "U",
  "V",
  "W",
  "X",
  "Y",
  "Z",
];

function isPangram(text) {
  // your code here
}

console.log(
  isPangram(`A pangram or holoalphabetic sentence is a sentence
using every letter of a given alphabet at least once.`)
); // => false

console.log(isPangram("Waltz, bad nymph, for quick jigs vex.")); // => true
```

### Решение

```js
const LETTERS = [
  "A",
  "B",
  "C",
  "D",
  "E",
  "F",
  "G",
  "H",
  "I",
  "J",
  "K",
  "L",
  "M",
  "N",
  "O",
  "P",
  "Q",
  "R",
  "S",
  "T",
  "U",
  "V",
  "W",
  "X",
  "Y",
  "Z",
];

function isPangram(text) {
  text = new Set(text.toLowerCase());

  return [...LETTERS].every((letter) => text.has(letter.toLowerCase()));
}

// Тестирование
console.log(
  isPangram(
    "A pangram or holoalphabetic sentence is a sentence using every letter of a given alphabet at least once."
  )
); // false
console.log(isPangram("Waltz, bad nymph, for quick jigs vex.")); // true
```

### Объяснение

- Приводим текст к нижнему регистру и преобразуем в множество, чтоб избавить от дубликатов и использовать проверку `.has()` сложностью О(1).
- Проверяем, встречается ли каждая буква из `LETTERS` в тексте.
- Если хотя бы одной буквы нет — возвращаем `false`.
- Если все есть — возвращаем `true`.
