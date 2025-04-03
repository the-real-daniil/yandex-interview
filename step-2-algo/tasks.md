# Этап 2: базовые навыки программирования ex. алгоритмы

## Дигит пермутэйшен

### Задача

```js
/*
Дан массив целых неотрицательных чисел, нужно сгруппировать друг с другом числа,
которые можно получить путём перестановки цифр их составляющих,
нули при этом игнорируем, т. к. нет числа 011.
Решение должно быть максимально эффективно по памяти и времени.
*/

function digitPermutation(arr) {
  // your code here
}

console.clear();
console.log("start test");
console.log(
  digitPermutation([1230, 99, 23001, 123, 111, 300021, 101010, 90000009, 9])
);
// [[99, 90000009], [111, 101010], [1230, 23001, 123, 300021], [9]]
console.log(digitPermutation([11, 22])); // [[11], [22]]
console.log(digitPermutation([11111111112, 122222222222])); // [[11111111112], [122222222222]]
console.log("end test");
```

### Решение

```js
function digitPermutation(arr) {
  const groups = {};

  for (let num of arr) {
    const key = String(num).replace(/0/g, "").split("").sort().join("");

    if (!groups[key]) {
      groups[key] = [];
    }

    groups[key].push(num);
  }

  return Object.values(groups);
}

console.log(
  digitPermutation([1230, 99, 23001, 123, 111, 300021, 101010, 900009, 9])
); // [[99, 900009], [111, 101010], [23001, 123, 23001, 300021], [9]]
console.log(digitPermutation([11, 22])); // [[11], [22]]
console.log(digitPermutation([11111112, 12222222])); // [[11111112], [12222222]]
```

### Объяснение

1. Создаем объект, в котором будем группировать числа
2. Проходимся по всем числам и каждое число превращаем в ключ
   - Превращаем в строку
   - Удаляем все нули
   - Превращаем в массив
   - Сортируем цифры по порядку
   - Превращаем обратно в строку
3. Если в объекте еще нет этого ключа, то создаем пустой массив по этому ключу
4. Добавляем число в объект по сформированному ключу
5. Возвращаем все сгруппированные значения объекта в виде массива

## DFS-ка

### Задача

```js
/*
Дана древовидная структура следующего формата:

const tree = {
  type: 'nested',
  children: [
    { type: 'added', value: 42 },
    {
      type: 'nested',
      children: [
        { type: 'added', value: 43 },
      ],
    },
    { type: 'added', value: 44 },
    ...
  ]
}

Необходимо написать функцию `getNodes(tree, type)`, которая возвращает все ноды в порядке следования, соответствующие переданному типу.

Глубина вложенности любая.

Пример:

const addedItems = getNodes(tree, 'added');

// Результат:
[
  { type: 'added', value: 42 },
  { type: 'added', value: 43 },
  { type: 'added', value: 44 },
  ...
]
*/
```

### Решение

```js
function getNodes(tree, type) {
  const result = [];

  const stack = [tree];

  while (stack.length != 0) {
    const node = stack.pop();

    if (node.type === type) {
      result.push(node);
    }

    if (node.children) {
      stack.push(...node.children);
    }
  }

  return result.reverse();
}

// Тестирование

const tree = {
  type: "nested",

  children: [
    { type: "added", value: 42 },

    {
      type: "nested",

      children: [{ type: "added", value: 43 }],
    },

    { type: "added", value: 44 },
  ],
};

console.log(getNodes(tree, "added"));
// [
// { type: 'added', value: 42 },
// { type: 'added', value: 43 },
// { type: 'added', value: 44 }
// ]
```

### Объяснение

1. Необходимо обойти дерево в глубину (DFS)
2. Рекурсию использовать нельзя, так как по условию дерево может иметь любую глубину вложенности
3. Используем stack для хранения узлов.
4. Берём последний элемент (pop()), обрабатываем.
5. Если тип совпадает — добавляем в result.
6. Если есть children, добавляем их в stack (push(...children), LIFO).
7. При возвращении result делаем reverse, так как по условию нужно вернуть узлы в порядке следования
8. Обход гарантированно работает для любых деревьев, глубина не влияет.

## Тортилла

### Задача

```js
/**
 * throttle.
 *
 * Напишите функцию throttle(fn, delay, ctx) — «тормозилку», которая возвращает обёртку,
 * вызывающую fn не чаще, чем раз в delay миллисекунд.
 * В качестве контекста исполнения используется ctx.
 * Первый вызов fn всегда должен быть синхронным.
 * Если игнорируемый вызов оказался последним, то он должен выполниться.
 */

// пример для delay === 100
// . - вызовы throttledFn
// ! - вызовы fn
// ...............!
///   !         !
///0ms 100ms   200ms
//  .    .     .
//      !     !
///0ms 100ms 200ms

function throttle(fn, delay, ctx) {
  // code here
}

function test() {
  const start = Date.now();

  function log(text) {
    const msPassed = Date.now() - start;
    console.log(`${msPassed}: ${this.name} logged ${text}`);
  }

  const throttled = throttle(log, 100, { name: "me" });

  setTimeout(() => throttled("m"), 0);
  setTimeout(() => throttled("mo"), 22);
  setTimeout(() => throttled("mos"), 33);
  setTimeout(() => throttled("mosc"), 150);
  setTimeout(() => throttled("moscow"), 400);

  // Ожидаемый вывод:
  //  0ms: me logged m
  // 100ms: me logged mos
  // 200ms: me logged mosc
  // 400ms: me logged moscow
}
```

### Решение

```js
function throttle(fn, delay, ctx) {
  let lastCallArgs = null;
  let blocked = false;

  function setTimer() {
    blocked = true;

    setTimeout(() => {
      blocked = false;

      if (lastCallArgs) {
        fn.apply(ctx, lastCallArgs);
        lastCallArgs = null;
        blocked = true;
        setTimer();
      }
    }, delay);
  }

  return function (...args) {
    if (blocked) {
      lastCallArgs = args;
    } else {
      fn.apply(ctx, args);
      setTimer();
    }
  };
}

function test() {
  const start = Date.now();

  function log(text) {
    const mPassed = Date.now() - start;

    console.log(`${mPassed}: ${this.name} logged ${text}`);
  }

  const throttled = throttle(log, 100, { name: "me" });

  setTimeout(() => throttled("m"), 0); // сразу
  setTimeout(() => throttled("mo"), 22); // игнор
  setTimeout(() => throttled("mos"), 33); // сохраняем 'mos'
  setTimeout(() => throttled("mosc"), 150); // прошло 150мс — вызов
  setTimeout(() => throttled("moscow"), 400); // прошло ещё 250мс — вызов
}

test();

// 0ms: me logged m
// 100ms: me logged mos
// 200ms: me logged mosc
// 400ms: me logged moscow
```

### Объяснение

1. В lastCallArgs храним последний переданные в функцию аргументы
2. В blocked храним флаг, который указывает, заблокирован ли вызов функции
3. Создаем функцию setTimer, которая блокирует потом и запускает таймер с переданным delay и вызывает fn с переданным lastCallArgs и контекстом
4. Возвращаем функцию, которая либо устанавливает новый lastCallArgs (если поток заблокирован), либо вызывает функцию и ставит таймер

## Вконтактик

### Задача

```js
/*
Наше приложение-чат должно отображать новые сообщения, которые приходят с сервера, как можно быстрее.

Сообщение имеет формат:

interface Message {
  id: number
  text: string
}

Id самого первого сообщения = 1, а id каждого следующего сообщения на 1 больше, чем id предыдущего.
Нам нужно выводить сообщения в правильном порядке, однако сервер не гарантирует правильный порядок
сообщений, отправляемых в наше приложение.

Таймлайн:
// (приходит)  1  3  4  2     5
// (рисуем)    1     2  3  4  5

Сообщения от сервера приходят в обработчик функции connect:

connect((msg) => {
  ...
});

Отображать сообщения нужно с помощью функции `render`:
render(msg)
*/
```

### Решение

```js
function render(msg) {
  console.log(msg);
}

function createMessageHandler() {
  const buffer = new Map();

  let expectedId = 1;

  return function (msg) {
    buffer.set(msg.id, msg);

    while (buffer.has(expectedId)) {
      render(buffer.get(expectedId));

      buffer.delete(expectedId);

      ++expectedId;
    }
  };
}
const onMessage = createMessageHandler();
onMessage({ id: 1, text: "hi" }); // рендерится сразу
onMessage({ id: 3, text: "how are u" }); // в буфере
onMessage({ id: 2, text: "yo" }); // триггерит рендер 2 и 3
onMessage({ id: 5, text: "yo" }); // ждёт 4
onMessage({ id: 4, text: "yo" }); // триггерит рендер 4 и 5
```

### Объяснение

1. Создаем createMessageHandler, который будет хранить буффер сообщение и возвращать обработчик onMessage
2. Объявляем buffer, в котором будут храниться все сообщения по ключу id
3. Объявляем expectedId, в котором будем хранить айдишник сообщения, которое должно быть выведено следующим (мы знаем, что айдишники идут по порядку)
4. Возвращаем функцию onMessage, которая

   - Принимает сообщение msg
   - Записывает сообщение в buffer по id
   - Пока в буффере есть сообщения с ожидаемым id, мы их рендерим и инкрементируем expectedId

5. Таким образом, сообщения будут выведены в порядке, соответствующем id

## Папочки

### Задача

```js
// Дана вложенная структура файлов и папок.

const data = {
  name: "folder",
  children: [
    { name: "file1.txt" },
    { name: "file2.txt" },
    {
      name: "images",
      children: [
        { name: "image.png" },
        {
          name: "vacation",
          children: [{ name: "crocodile.png" }, { name: "penguin.png" }],
        },
      ],
    },
    { name: "shopping-list.pdf" },
  ],
};

/*
Нужно вывести в консоль файлы и папки с отступами, чтобы показать вложенность.
Решение должно учитывать любую вложенность элементов (т.е. не должно содержать рекурсивные вызовы).
*/

/*
Пример вывода:
folder
  file1.txt
  file2.txt
  images
    image.png
    vacation
      crocodile.png
      penguin.png
  shopping-list.pdf
*/

function printFileTree(root) {
  // todo
}

printFileTree(data);
```

### Решение

```js
const data = {
  name: "folder",
  children: [
    { name: "file1.txt" },
    { name: "file2.txt" },
    {
      name: "images",
      children: [
        { name: "image.png" },
        {
          name: "vacation",
          children: [{ name: "crocodile.png" }, { name: "penguin.png" }],
        },
      ],
    },
    { name: "shopping-list.pdf" },
  ],
};

function printFileTree(root) {
  const stack = [{ node: root, depth: 0 }];

  while (stack.length > 0) {
    const { node, depth } = stack.pop();

    console.log("  ".repeat(depth) + node.name);

    if (node.children) {
      for (let i = node.children.length - 1; i >= 0; i--) {
        stack.push({ node: node.children[i], depth: depth + 1 });
      }
    }
  }
}

printFileTree(data);
```

### Объяснение

1. Необходимо реализовать алгоритм DFS (обход дерева в глубину) без рекурсии, поэтому используем стек
2. `stack` — стек, где каждый элемент содержит:
3. `node` — текущая папка или файл
4. `depth` — сколько отступов печатать
5. `console.log(' '.repeat(depth) + node.name)` — вывод имени с нужным числом отступов.
6. Если у узла есть `children` — добавляем их в стек, увеличив `depth`.
7. Важно: добавляем детей в обратном порядке, чтобы в консоли они шли сверху вниз в правильной очередности.

## Генератор

### Задача

```js
/*
    Напишите функцию, которая будет возвращать комбинацию слов,
    для переданного массива массивов слов.
    Комбинации должны быть перечислены в порядке из примера.
    Если комбинации закончились, то функция возвращает `undefined`.
    При решении нельзя пользоваться генераторами.
*/

const nextSequence = allSequences([
  [0, 1, 2],
  ["a", "b"],
  ["?", "!", "."],
]);

console.log(nextSequence()); // "0 a ?"
console.log(nextSequence()); // "0 a !"
console.log(nextSequence()); // "0 a ."
console.log(nextSequence()); // "0 b ?"
// ...
console.log(nextSequence()); // "2 b ."
console.log(nextSequence()); // undefined
```

### Решение

```js
function allSequences(words) {
  const position = new Array(words.length).fill(0);
  let finished = false;

  return function nextSequence() {
    if (finished) return undefined;

    const current = position.map((val, i) => words[i][val]).join(" ");

    for (let i = words.length - 1; i >= 0; i--) {
      position[i]++;

      if (position[i] < words[i].length) break;

      position[i] = 0;

      if (i === 0) {
        finished = true;
      }
    }

    return current;
  };
}

const nextSequence = allSequences([
  [0, 1, 2],
  ["a", "b"],
  ["?", "!", "."],
]);

console.log(nextSequence()); // "0 a ?"
console.log(nextSequence()); // "0 a !"
console.log(nextSequence()); // "0 a ."
console.log(nextSequence()); // "0 b ?"
// ...
console.log(nextSequence()); // "2 b ."
console.log(nextSequence()); // undefined
```

### Объяснение

1. Инициализируем массив индексов `position`:
   - Используем для отслеживания текущей позиции в каждом подмассиве.
   - Например, `[0, 0, 0]` — первая комбинация:` words[0][0]` `words[1][0]` `words[2][0]`
2. `finished` — флаг, показывающий, что комбинации закончились
3. В функции `nextSequence`:
   - Формируем строку в соответствии с текущим значением `position`
   - Обновляем индексы справа налево
   - Возврщаем результат
