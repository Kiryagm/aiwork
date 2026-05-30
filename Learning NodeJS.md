
https://chat.deepseek.com/share/c1tnpp2ilvib49gqpn


**1. Что такое Node.js и зачем он нужен (рантайм, событийная модель, неблокирующий ввод-вывод)**

Node.js — это не язык и не фреймворк. Это **среда выполнения (runtime)** для JavaScript, построенная на движке V8 от Google (используется в Chrome). Она позволяет запускать JS вне браузера — на сервере, в терминале, встроенных системах.

Ключевое отличие от классических серверов (Java, PHP, Ruby on Rails, Python с WSGI): Node.js работает в **одном потоке (single-threaded)**, но с **событийно-ориентированной (event-driven)** и **неблокирующей (non-blocking)** архитектурой.

**Стандартная блокирующая модель** (условный PHP под Apache):  
На каждый запрос сервер выделяет отдельный поток (или процесс). Если поток выполняет чтение файла или запрос к БД (операция I/O), он **ждёт** — останавливается до получения результата. Чем больше параллельных запросов, тем больше потоков и памяти.

**Модель Node.js**:
- Один главный поток принимает запросы.
- Вместо ожидания завершения I/O (чтение диска, сетевой вызов, запрос к БД) поток **отправляет операцию** и сразу переключается на следующий запрос.
- Когда операция завершается, в очередь событий (event queue) попадает уведомление, и **событийный цикл (event loop)** обрабатывает его — вызывает колбэк или Promise.

Результат:
- Один поток справляется с тысячами одновременных соединений.
- Нет накладных расходов на создание потоков и переключение контекста.

**Важно понимать ограничения**:
- Тяжёлые **CPU-операции** (обработка изображений, перебор массивов, шифрование) будут блокировать **весь** цикл событий — все остальные запросы встанут. Для таких задач используют Worker Threads или внешние микросервисы.
- Для типичных I/O-нагруженных приложений (REST API, чаты, прокси, обработка форм, работа с БД) Node.js даёт высокую производительность и отличную масштабируемость при малом потреблении памяти.

**Когда выбирать Node.js**:  
- Вы уже знаете JS и хотите использовать один язык на клиенте и сервере.
- Нужно много одновременных подключений с лёгкими запросами.
- Подходят real-time приложения (сокеты, стриминг).

**Когда НЕ стоит**:  
- Сложные математические вычисления, видео-кодирование, массовая трансформация изображений — эти задачи лучше отдать специализированным сервисам на Go, Rust, C++ или хотя бы Java/C#.



**2. Установка Node.js и npm (менеджеры версий: nvm)**

Node.js устанавливается вместе с **npm** (Node Package Manager) — стандартным менеджером пакетов для JavaScript. Но прямая установка из официального сайта — антипаттерн для профессиональной разработки.

**Проблема прямой установки**:  
Вы привязываетесь к одной глобальной версии Node.js. Разные проекты требуют разные версии (например, проект А на Node 18, проект Б на Node 22). Попытка переключать вручную или использовать символические ссылки приводит к хаосу и ошибкам.

**Решение — менеджеры версий**:

**nvm (Node Version Manager)** — стандарт де-факто для macOS и Linux.

Установка:
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
После установки перезагрузите терминал.

Базовые команды:
```bash
nvm install 20          # установить Node.js версии 20.x (последний LTS на момент написания)
nvm install 18          # установить версию 18
nvm ls                  # показать все установленные версии
nvm use 20              # переключиться на версию 20 в текущем терминале
nvm alias default 20    # установить версию по умолчанию для новых сессий
```

**Для Windows** — используйте **nvm-windows** (отдельный проект, синтаксис похож) или **fnm** (быстрый, написан на Rust, кроссплатформенный).

**Проверка после установки**:
```bash
node --version   # ожидается v20.x.x
npm --version    # версия npm поставляется с Node
```

**Важное о npm**:
- npm устанавливает пакеты локально (в `node_modules` проекта) или глобально (ключ `-g`).
- **Никогда** не используйте `sudo npm install -g` — это нарушает права файловой системы и создаёт проблемы с правами доступа. nvm решает эту проблему, так как все версии хранятся в домашней директории пользователя.

**Файл `.nvmrc` в проекте**:
Фиксируете нужную версию:
```bash
echo "20" > .nvmrc
```
Теперь любой разработчик выполняет `nvm use` внутри папки проекта — nvm автоматически переключится на версию 20.

**Альтернативы**:
- **fnm** — быстрее nvm, поддерживает `.node-version` файлы (стандарт от Volta/NodeSource)
- **Volta** — автоматическое переключение версий при входе в директорию проекта (без ручного `nvm use`)

**Практический совет**:  
Всегда устанавливайте **LTS-версию** (Long-Term Support) для продакшен-проектов. Текущие major-релизы хороши для экспериментов, но не для стабильной работы.



**3. Ваш первый скрипт Node.js (запуск JavaScript вне браузера)**

Создайте файл `app.js` в любой директории. В браузере JS работает с `window`, `document`, DOM-элементами. В Node.js этого нет — вместо этого глобальные объекты `global`, `process`, `__dirname`, `__filename`.

**Простейший пример**:
```javascript
// app.js
console.log('Hello from Node.js');
console.log('Текущая директория:', __dirname);
console.log('Имя файла:', __filename);
console.log('Аргументы командной строки:', process.argv);
```

**Запуск**:
```bash
node app.js
```

**Ключевое отличие от браузера**:
- Нет `alert()`, `prompt()`, `confirm()` — используйте `console.log()` и чтение из терминала (будет позже).
- Нет объекта `window` — есть `global` или `globalThis` (современный стандарт).
- Код выполняется **синхронно** сверху вниз, но вы можете асинхронно читать файлы или делать сетевые запросы.

**Чтение аргументов командной строки**:
```javascript
// args.js
const args = process.argv.slice(2); // первые два элемента — путь к node и путь к файлу
console.log('Переданные аргументы:', args);

if (args[0] === '--help') {
  console.log('Доступные команды: ...');
}
```

Запуск с аргументами:
```bash
node args.js --help --port 3000
```

**Работа с пользовательским вводом (базовый интерфейс)**:
```javascript
// input.js
process.stdout.write('Как вас зовут? ');

process.stdin.on('data', (data) => {
  const name = data.toString().trim();
  console.log(`Привет, ${name}!`);
  process.exit(); // завершаем процесс
});
```

**Выход из процесса**:
```javascript
process.exit(0);   // успешное завершение
process.exit(1);   // ошибка — сигнал системе, что скрипт упал
```

**Переменные окружения** (важно для конфигурации):
```javascript
// env.js
const port = process.env.PORT || 3000;
const mode = process.env.NODE_ENV || 'development';
console.log(`Сервер запустится на порту ${port} в режиме ${mode}`);
```

Запуск с переменными:
```bash
PORT=5000 NODE_ENV=production node env.js
```

**Использование глобальных объектов осторожно**:
```javascript
// в браузере это создаст window.myVar, в Node.js — global.myVar
global.myVar = 'это глобально'; 
// так делать не рекомендуется — нарушает модульную изоляцию
```

**Практический пример: утилита для работы с файловой системой**:
```javascript
// cat.js — аналог команды cat (вывод содержимого файла)
const fs = require('fs');
const filePath = process.argv[2];

if (!filePath) {
  console.error('Укажите путь к файлу');
  process.exit(1);
}

fs.readFile(filePath, 'utf8', (err, data) => {
  if (err) {
    console.error(`Ошибка чтения файла: ${err.message}`);
    process.exit(1);
  }
  console.log(data);
});
```

Запуск:
```bash
node cat.js app.js
```

**Что дальше**: этот скрипт уже использует встроенный модуль `fs` — следующий шаг к модульной системе Node.js. Научитесь создавать модули, экспортировать функции и подключать их через `require`.



**4. Среда REPL в Node.js**

**REPL** (Read-Eval-Print Loop) — интерактивная консоль Node.js для быстрого тестирования кода, изучения API и отладки. Запускается командой `node` без аргументов.

**Запуск и базовое использование**:
```bash
node
> 2 + 2
4
> const name = 'Node.js'
undefined
> console.log(`Hello ${name}`)
Hello Node.js
undefined
> .exit
```

**Ключевые особенности**:

- **Автоматический вывод результата** — не нужно писать `console.log()` для последнего выражения:
```javascript
> Math.random()
0.7234567891234567
> [1, 2, 3].map(x => x * 2)
[2, 4, 6]
```

- **Сохранение истории** — стрелки вверх/вниз для доступа к предыдущим командам (файл `.node_repl_history` в домашней директории).

- **Подчёркивание `_`** — содержит результат последней операции:
```javascript
> 5 * 10
50
> _ + 2
52
```

**Специальные команды (с точки)**:
```javascript
.help             // показать все доступные команды
.editor           // открыть редактор (для многострочного кода)
.save filename.js // сохранить сессию в файл
.load filename.js // загрузить и выполнить код из файла
.clear            // очистить контекст
.exit             // выйти из REPL
```

**Использование `.editor`**:
```
> .editor
// Входим в режим редактора (Ctrl+D для выполнения, Ctrl+C отмена)
function greet(name) {
  return `Hello ${name}!`
}
greet('Node.js')
// Нажать Ctrl+D
'Hello Node.js!'
```

**Доступ к встроенным модулям**:
```javascript
> const fs = require('fs')
undefined
> fs.readdirSync('.')
['app.js', 'node_modules', 'package.json']
```

**Переменные уровня REPL**:
```javascript
> global.myVar = 'доступно везде'
'доступно везде'
> .exit
$ node
> global.myVar  // новая сессия — переменной нет
undefined
```

**Табуляция для автодополнения**:
```javascript
> fs.    // дважды нажать Tab — показать все методы fs
fs.access    fs.appendFile  fs.chmod     fs.chown     ...
> global.    // автодополнение глобальных объектов
```

**Запуск REPL с загруженным файлом**:
```bash
node -i -e "const config = require('./config.js')"
> config.port
3000
```
Флаг `-i` запускает REPL после выполнения кода, `-e` выполняет строку.

**Практические сценарии использования**:

1. **Быстрое тестирование фрагментов кода** — вместо создания временного файла.
2. **Изучение API модулей** — `require('crypto')` и Tab для просмотра методов.
3. **Отладка асинхронного кода**:
```javascript
> setTimeout(() => console.log('done'), 1000)
Timeout {...}
> done   // через 1 секунду
```
4. **Калькулятор** — для простых математических операций.

**Настройка REPL через запуск**:
```bash
NODE_REPL_HISTORY=~/.custom_repl_history node
```

**Продвинутые возможности — кастомный REPL в коде**:
```javascript
// custom-repl.js
const repl = require('repl');
const local = repl.start('> ');
local.context.db = require('./database');
local.context.helpers = { formatDate: (d) => new Date(d).toISOString() };
```
Запуск даёт REPL с предзагруженными объектами `db` и `helpers`.

**Отличие от браузерной консоли**:
- Нет DOM API (`document`, `window`).
- Доступны все модули Node.js (`fs`, `path`, `http`).
- Полноценная работа с файловой системой — можно создавать, удалять файлы прямо из REPL.

**Выход из REPL**:
- Ctrl+C дважды (или один раз в пустой строке)
- Команда `.exit`
- Ctrl+D (Linux/macOS) / Ctrl+Z (Windows)



**5. Глобальный объект `global` vs браузерный `window`**

В браузере глобальный объект — `window`. В Node.js — `global`. Они служат одной цели (предоставляют глобальную область видимости), но ведут себя принципиально по-разному из-за модульной системы Node.js.

**Базовое различие: объявление переменных**:

В браузере (глобальная область):
```javascript
var x = 10;      // создаёт window.x
y = 20;          // создаёт window.y (строгий режим запретит)
console.log(window.x); // 10
```

В Node.js (модуль):
```javascript
var x = 10;      // НЕ попадает в global — локально для модуля
y = 20;          // создаёт global.y (антипаттерн!)
console.log(global.x); // undefined
console.log(global.y); // 20 — но так делать нельзя
```

**Почему так**? Каждый файл в Node.js — изолированный модуль. Переменные, объявленные через `var`, `let`, `const` на верхнем уровне, принадлежат только этому модулю, а не глобальному объекту.

**Правильные способы создать глобальную переменную**:
```javascript
global.myApp = { version: '1.0.0' };   // явное добавление в global
// или
globalThis.myApp = { version: '1.0.0' }; // универсальный стандарт ES2020
```

**Что реально лежит в `global`**:
```javascript
console.log(Object.keys(global));
// ['global', 'clearImmediate', 'setImmediate', 'clearInterval', 
//  'clearTimeout', 'setInterval', 'setTimeout', 'queueMicrotask', 
//  'structuredClone', 'performance', 'atob', 'btoa', ...]
```

**Кросс-платформенный `globalThis`** (ES2020):
```javascript
// Работает везде: браузер (window), Node.js (global), Web Workers (self)
globalThis.setTimeout(() => {}, 1000); // один стиль для всех сред
```

**Ключевые различия в API**:

| Браузер (`window`) | Node.js (`global`) |
|-------------------|-------------------|
| `document`, `window`, `alert()` | `process`, `Buffer`, `__dirname` |
| `localStorage`, `sessionStorage` | `require()`, `module`, `exports` |
| `fetch()`, `XMLHttpRequest` | `http`, `https`, `fs` (через require) |
| `requestAnimationFrame` | `setImmediate()`, `process.nextTick()` |

**Опасный пример: засорение глобальной области**:
```javascript
// module-a.js
global.config = { db: 'localhost' };

// module-b.js
console.log(global.config.db); // 'localhost' — неявная зависимость
```
Это антипаттерн: модуль B не заявляет явно, что ему нужен config. Правильный путь — передача через параметры или импорт из отдельного конфигурационного модуля.

**Когда использовать `global` легитимно**:
1. **Полифиллы для встроенных объектов**:
```javascript
if (!globalThis.TextEncoder) {
  globalThis.TextEncoder = require('util').TextEncoder;
}
```

2. **Отладочные флаги** (только для разработки):
```javascript
if (process.env.NODE_ENV === 'development') {
  global.__DEBUG__ = true;
}
```

3. **Встроенные глобальные конструкции Node.js**:
```javascript
console.log(__dirname);     // путь к текущей директории (НЕ в global, но глобально доступен)
console.log(__filename);    // путь к текущему файлу
console.log(module);        // текущий модуль
console.log(require);       // функция подключения модулей
```

**Проверка окружения: где выполняется код**:
```javascript
if (typeof window !== 'undefined' && window.document) {
  console.log('Браузер');
} else if (typeof global !== 'undefined' && global.process) {
  console.log('Node.js');
} else {
  console.log('Другая среда (Deno, Bun, Worker)');
}
```

**Современная альтернатива `global`**: используйте ES-модули с явным импортом/экспортом, а не глобальные переменные. `global` существует в основном для:
- Внутренних механизмов Node.js
- Обратной совместимости со старым кодом
- Редких кейсов, где глобальное состояние действительно оправдано (логирование, трейсинг)



**6. Понимание модульной системы (CommonJS: `require` и `module.exports`)**

Node.js по умолчанию использует **CommonJS** — модульную систему, которая изолирует код каждого файла. Браузеры не имеют встроенной поддержки CommonJS (поэтому используют сборщики вроде Webpack).

**Базовый экспорт и импорт**:

Файл `math.js` (экспорт):
```javascript
// приватная переменная — не видна снаружи
const PI = 3.14159;

// публичный экспорт
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = { add, subtract };
```

Файл `app.js` (импорт):
```javascript
const math = require('./math.js'); // .js можно опустить
console.log(math.add(5, 3));      // 8
console.log(math.subtract(10, 4)); // 6
```

**Разные способы экспорта**:

```javascript
// Способ 1: module.exports = объект (полная замена)
module.exports = { add, subtract };

// Способ 2: добавление свойств к существующему exports
exports.add = add;
exports.subtract = subtract;

// Способ 3: module.exports = функция (один экспорт)
module.exports = function(a, b) { return a + b; };
```

**Важное различие: `exports` vs `module.exports`**:
```javascript
// exports — это ссылка на module.exports
console.log(exports === module.exports); // true

// Так работает:
exports.hello = () => console.log('hi'); // корректно добавляет свойство

// Так НЕ работает:
exports = { hello: () => console.log('hi') }; // разрывает ссылку, module.exports остаётся пустым

// Правильно заменять только через module.exports:
module.exports = { hello: () => console.log('hi') };
```

**Кэширование модулей** (важное поведение):
```javascript
// counter.js
console.log('Модуль загружен');
let count = 0;
module.exports.increment = () => ++count;
module.exports.getCount = () => count;

// app.js
const counter1 = require('./counter.js');
const counter2 = require('./counter.js'); // тот же экземпляр!
counter1.increment();
console.log(counter2.getCount()); // 1 — модуль закеширован
```
Node.js кэширует модули после первого `require`. Одинаковые пути возвращают один и тот же объект.

**Пути в require**:
```javascript
require('./local-file');        // файл в текущей директории (./ или ../)
require('/absolute/path/file'); // абсолютный путь
require('fs');                  // встроенный модуль Node.js
require('express');             // сторонний модуль из node_modules
require('./folder');            // ищет folder/index.js или folder/package.json main
```

**Циклические зависимости** (как Node.js их разрешает):
```javascript
// a.js
const b = require('./b');
console.log('a.js: b =', b);
module.exports = { name: 'A' };

// b.js
const a = require('./a');
console.log('b.js: a =', a);
module.exports = { name: 'B' };

// порядок вывода:
// b.js: a = {} (неполный экспорт)
// a.js: b = { name: 'B' }
```
Node.js возвращает частично сформированный модуль, избегая бесконечной рекурсии.

**Условный require** (антипаттерн, но иногда необходим):
```javascript
let logger;
if (process.env.NODE_ENV === 'production') {
  logger = require('./logger-prod');
} else {
  logger = require('./logger-dev');
}
```

**Структура модуля после подключения**:
```javascript
const fs = require('fs');
console.log(module); // {
//   id: '.',
//   path: '/path/to',
//   exports: {},
//   parent: null,
//   filename: '/path/to/app.js',
//   loaded: false,
//   children: [...],
//   paths: [...]
// }
```

**Паттерн "один экспорт — функция-конструктор"**:
```javascript
// user.js
function User(name) {
  this.name = name;
  this.sayHi = () => console.log(`Hi, ${this.name}`);
}
module.exports = User;

// app.js
const User = require('./user.js');
const user = new User('Alice');
user.sayHi(); // Hi, Alice
```

**Паттерн "синглтон через экспорт экземпляра"**:
```javascript
// db.js
class Database {
  constructor() { this.connected = false; }
  connect() { this.connected = true; }
}
module.exports = new Database(); // один экземпляр на всё приложение
```

**__dirname и __filename в модулях**:
```javascript
console.log(__dirname);  // /home/user/project/src
console.log(__filename); // /home/user/project/src/app.js

const path = require('path');
const filePath = path.join(__dirname, 'data', 'config.json');
```

**Переход на ES-модули** (современный стандарт):
Добавьте `"type": "module"` в `package.json` и используйте `import/export`:
```javascript
// math.mjs или .js с "type": "module"
export const add = (a, b) => a + b;
export default function multiply(a, b) { return a * b; }

// app.mjs
import multiply, { add } from './math.mjs';
```

**Практический совет**: для новых проектов рассмотрите ES-модули (синтаксис яснее, поддерживается во всех современных средах). CommonJS остаётся стандартом для экосистемы npm (многие пакеты до сих пор поставляются в CommonJS), поэтому понимать его необходимо в любом случае.



**7. ES-модули в Node.js (`import`/`export` с `"type": "module"`)**

Начиная с Node.js 12 (стабильно с 14), поддерживаются нативные ES-модули — тот же синтаксис `import/export`, что и в браузере. Это официальный стандарт ECMAScript, в отличие от CommonJS.

**Активация ES-модулей**:

Способ 1 — `package.json`:
```json
{
  "name": "my-app",
  "type": "module",
  "dependencies": {}
}
```
Теперь все `.js` файлы в проекте интерпретируются как ES-модули.

Способ 2 — расширение `.mjs`:
```
file.mjs  // всегда ES-модуль, независимо от package.json
file.cjs  // всегда CommonJS
```

**Базовый экспорт и импорт**:

```javascript
// math.js
// именованные экспорты
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }

// экспорт по умолчанию (один на модуль)
export default function multiply(a, b) { return a * b; }
```

```javascript
// app.js
import multiply, { add, subtract, PI } from './math.js';

console.log(add(5, 3));      // 8
console.log(subtract(10, 4)); // 6
console.log(multiply(2, 3));  // 6
console.log(PI);              // 3.14159
```

**Импорт всего пространства имён**:
```javascript
import * as math from './math.js';
console.log(math.add(2, 3));     // 5
console.log(math.default(2, 3)); // 6 (экспорт по умолчанию)
```

**Переименование при импорте/экспорте**:
```javascript
// экспорт с псевдонимом
export { add as sum, subtract as diff };

// импорт с псевдонимом
import { sum as addFunction, diff as subtractFunction } from './math.js';
```

**Реэкспорт (агрегация модулей)**:
```javascript
// index.js — собирает API из нескольких модулей
export { add, subtract } from './math.js';
export { default as User } from './user.js';
export * from './helpers.js';
```

**Динамический импорт** (ленивая загрузка):
```javascript
// загружается только при вызове, возвращает Promise
const modulePath = './math.js';
const math = await import(modulePath);
console.log(math.add(5, 3));

// с деструктуризацией
const { default: multiply, add } = await import('./math.js');
```

**Особенности ES-модулей в Node.js**:

1. **Строгий режим всегда включён** — не нужно писать `'use strict'`.

2. **Расширения обязательны** (кроме случаев с `--experimental-specifier-resolution`):
```javascript
import { readFile } from 'fs';           // встроенный модуль — OK
import './local-file';                   // ОШИБКА — нужно расширение
import './local-file.js';                // OK
```

3. **__dirname и __filename не доступны**:
```javascript
// вместо них:
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// или используйте import.meta.url напрямую
console.log(import.meta.url); // file:///home/user/project/app.js
```

4. **require, module, exports не определены**:
```javascript
console.log(typeof require);  // "undefined" в ES-модуле
```

5. **Только Promise-топ-уровня (top-level await)**:
```javascript
// Можно использовать await вне async функции
const data = await fetch('https://api.example.com/data');
export default data;
```

**Совместимость CommonJS и ES-модулей**:

**Импорт CommonJS из ES-модуля** (работает):
```javascript
// es-module.mjs
import { createServer } from 'http';     // встроенный CommonJS
import express from 'express';            // сторонний CommonJS
const packageJson = require('./package.json'); // ОШИБКА — require не доступен

// правильный импорт CommonJS-модуля:
import packageJson from './package.json' assert { type: 'json' };
// или для старых версий:
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const packageJson = require('./package.json');
```

**Импорт ES-модуля из CommonJS** (только динамический):
```javascript
// commonjs.cjs
async function loadESModule() {
  const esModule = await import('./es-module.mjs');
  console.log(esModule.default);
}
```

**Различия в кэшировании**:
- CommonJS: `require()` кэширует по полному пути
- ES-модули: `import` кэширует по URL, но с большей детерминированностью

**Файлы JSON**:
```javascript
// ES-модули требуют явный assert
import config from './config.json' assert { type: 'json' };
console.log(config.port);

// или через readFile (для динамической загрузки)
import { readFile } from 'fs/promises';
const config = JSON.parse(await readFile('./config.json', 'utf8'));
```

**Практические рекомендации**:

- **Новые проекты** — используйте ES-модули: синтаксис современнее, топ-level await удобен, анализ зависимостей проще.
- **Существующие проекты** — оставайтесь на CommonJS, если у вас много зависимостей, плохо поддерживающих ES-модули.
- **Библиотеки для npm** — публикуйте с поддержкой обоих форматов (поля `main` и `module` в `package.json`).
- **Не смешивайте** стили в одном проекте без необходимости — это усложняет отладку.

**Проверка типа модуля из кода**:
```javascript
// в ES-модуле
console.log(import.meta.url); // определён

// в CommonJS
console.log(typeof __dirname !== 'undefined'); // true для CommonJS
```



**8. Модуль `fs`: чтение и запись файлов (синхронный vs асинхронный)**

Модуль `fs` (File System) — один из ключевых в Node.js. Позволяет взаимодействовать с файловой системой: читать, писать, удалять, переименовывать файлы и директории.

**Три стиля работы с `fs`** (важно понимать разницу):

| Стиль | Пример метода | Блокирует поток? | Возвращает |
|-------|--------------|------------------|------------|
| Синхронный | `readFileSync()` | **Да** | данные напрямую |
| Асинхронный (колбэк) | `readFile()` | **Нет** | `undefined` (результат в колбэке) |
| Асинхронный (Promise) | `readFile()` из `fs/promises` | **Нет** | `Promise` |

**1. Синхронное чтение и запись (блокирующее)** — только для скриптов инициализации:

```javascript
const fs = require('fs');

// Чтение
try {
  const data = fs.readFileSync('./file.txt', 'utf8');
  console.log(data);
} catch (err) {
  console.error('Ошибка чтения:', err.message);
}

// Запись
fs.writeFileSync('./output.txt', 'Привет, мир!', 'utf8');

// Дозапись
fs.appendFileSync('./log.txt', `Новая запись ${Date.now()}\n`);

// Проверка существования
if (fs.existsSync('./config.json')) {
  const config = JSON.parse(fs.readFileSync('./config.json', 'utf8'));
}
```

**Проблема синхронных методов** — во время чтения файла на 100 МБ весь сервер не обрабатывает другие запросы.

**2. Асинхронный стиль с колбэками (классический Node.js)** :

```javascript
const fs = require('fs');

// Чтение
fs.readFile('./file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Ошибка:', err);
    return;
  }
  console.log(data);
});

// Запись
fs.writeFile('./output.txt', 'Содержимое', 'utf8', (err) => {
  if (err) throw err;
  console.log('Файл сохранён');
});

// Цепочка операций (ад колбэков)
fs.readFile('./source.txt', 'utf8', (err, data) => {
  if (err) return console.error(err);
  fs.writeFile('./dest.txt', data.toUpperCase(), 'utf8', (err) => {
    if (err) return console.error(err);
    console.log('Готово');
  });
});
```

**3. Promise-версия (современный стандарт, рекомендуется)** :

```javascript
const fs = require('fs/promises'); // обратите внимание на /promises

async function workWithFiles() {
  try {
    const data = await fs.readFile('./file.txt', 'utf8');
    console.log(data);
    
    await fs.writeFile('./output.txt', data.toUpperCase(), 'utf8');
    console.log('Записано');
    
    // Чтение бинарного файла (без указания кодировки)
    const imageBuffer = await fs.readFile('./photo.jpg');
    console.log('Размер:', imageBuffer.length);
    
  } catch (err) {
    console.error('Ошибка:', err.message);
  }
}

workWithFiles();
```

**Ключевые методы `fs` (Promise-версия)** :

```javascript
// Проверка существования (альтернатива existsSync)
async function fileExists(path) {
  try {
    await fs.access(path);
    return true;
  } catch {
    return false;
  }
}

// Информация о файле
const stats = await fs.stat('./file.txt');
console.log({
  size: stats.size,           // размер в байтах
  isFile: stats.isFile(),     // это файл?
  isDirectory: stats.isDirectory(),
  mtime: stats.mtime,         // дата изменения
  birthtime: stats.birthtime  // дата создания
});

// Копирование
await fs.copyFile('./source.txt', './destination.txt');

// Переименование / перемещение
await fs.rename('./old.txt', './new.txt');

// Удаление
await fs.unlink('./tmp.txt');

// Работа с директориями
await fs.mkdir('./new-folder', { recursive: true }); // создаёт вложенные папки
const files = await fs.readdir('./');                 // список файлов
await fs.rmdir('./empty-folder');                     // удалить пустую папку
await fs.rm('./folder', { recursive: true, force: true }); // удалить папку с содержимым
```

**Работа с потоками (для больших файлов)** — не загружает весь файл в память:

```javascript
const fs = require('fs');
const readStream = fs.createReadStream('./large-file.txt', { encoding: 'utf8' });
const writeStream = fs.createWriteStream('./copy.txt');

readStream.on('data', (chunk) => {
  console.log(`Получено ${chunk.length} байт`);
  writeStream.write(chunk);
});

readStream.on('end', () => {
  writeStream.end();
  console.log('Копирование завершено');
});

// Или через pipe (проще)
readStream.pipe(writeStream);
```

**Обработка ошибок** — критическое различие:

```javascript
// Синхронный — try/catch
try {
  const data = fs.readFileSync('./missing.txt', 'utf8');
} catch (err) {
  console.log('Файл не найден');
}

// Колбэк — проверка err первым аргументом
fs.readFile('./missing.txt', 'utf8', (err, data) => {
  if (err.code === 'ENOENT') {
    console.log('Файл не существует');
  }
});

// Promise — try/catch или .catch
try {
  await fs.readFile('./missing.txt', 'utf8');
} catch (err) {
  if (err.code === 'ENOENT') console.log('Нет файла');
}
```

**Коды ошибок файловой системы** (часто встречаются):
- `ENOENT` — файл или директория не существует
- `EACCES` — недостаточно прав
- `EISDIR` — ожидался файл, а это директория
- `ENOTDIR` — ожидалась директория, а это файл
- `EEXIST` — файл уже существует

**Когда что использовать**:
| Сценарий | Рекомендация |
|----------|--------------|
| Загрузка конфига при старте | `readFileSync` (один раз при запуске) |
| Обработка HTTP-запроса | `fs/promises` или потоки |
| Большие файлы (>100 МБ) | **потоки** (`createReadStream`) |
| Консольные утилиты | `readFileSync` (простота) |
| Миграции данных | `fs/promises` с `Promise.all` для параллельности |

**Параллельное чтение нескольких файлов**:

```javascript
const files = ['./a.txt', './b.txt', './c.txt'];
const contents = await Promise.all(
  files.map(file => fs.readFile(file, 'utf8').catch(err => `Ошибка в ${file}: ${err.message}`))
);
console.log(contents);
```

**Важное правило**: в продакшен-сервере никогда не используйте синхронные методы `fs` в обработчиках запросов — каждый такой вызов блокирует весь event loop. Исключение — инициализация приложения до старта сервера.



**9. Работа со Streams и Buffers**

**Buffer** — временное хранилище бинарных данных в памяти. Появляется там, где Node.js работает с файлами, сетью, криптографией. **Stream** — абстракция для последовательной обработки данных частями (чанками), не загружая всё в память целиком.

**Buffer: когда и зачем**

Буфер нужен для работы с сырыми бинарными данными: изображения, видео, аудио, двоичные протоколы, TCP-пакеты.

```javascript
// Создание буферов
const buf1 = Buffer.from('Hello');              // из строки
const buf2 = Buffer.from([72, 101, 108, 108, 111]); // из массива байт
const buf3 = Buffer.alloc(10);                  // пустой буфер 10 байт (заполнен нулями)
const buf4 = Buffer.allocUnsafe(10);            // быстрее, но содержит "мусор"

// Чтение и запись
const buf = Buffer.from('Node.js');
console.log(buf[0]);        // 78 ('N' в ASCII)
console.log(buf.toString()); // 'Node.js'
console.log(buf.toString('hex')); // 4e6f64652e6a73

buf[0] = 77;                // замена байта
console.log(buf.toString()); // 'Mode.js'

// Размер и срезы
console.log(buf.length);    // 7 байт
const slice = buf.subarray(1, 4); // 'ode' (без копирования данных)
```

**Кодировки** (важно для преобразований):
```javascript
Buffer.from('Привет', 'utf8');      // UTF-8 (по умолчанию)
Buffer.from('Привет', 'ucs2');      // UTF-16
Buffer.from('Hello', 'ascii');      // только латиница
Buffer.from('Zm9v', 'base64');      // декодирование Base64
buf.toString('base64');              // кодирование в Base64
```

**Stream: четыре типа**

| Тип | Примеры | Направление |
|-----|---------|-------------|
| Readable | `fs.createReadStream`, HTTP request | чтение данных |
| Writable | `fs.createWriteStream`, HTTP response | запись данных |
| Duplex | TCP socket, crypto stream | чтение и запись |
| Transform | `zlib.createGzip`, `crypto.createCipher` | преобразование на лету |

**Readable Stream (чтение файла по частям)** :

```javascript
const fs = require('fs');
const stream = fs.createReadStream('./large-file.txt', { 
  encoding: 'utf8',
  highWaterMark: 64 * 1024  // размер чанка: 64 КБ (по умолчанию)
});

stream.on('data', (chunk) => {
  console.log(`Получено ${chunk.length} байт`);
  // пауза для контроля обратного давления (backpressure)
  stream.pause();
  setTimeout(() => stream.resume(), 100);
});

stream.on('end', () => console.log('Файл прочитан полностью'));
stream.on('error', (err) => console.error(err));
```

**Writable Stream (запись потоком)** :

```javascript
const fs = require('fs');
const stream = fs.createWriteStream('./output.log', { flags: 'a' }); // 'a' = append

stream.write('Первая строка\n');
stream.write('Вторая строка\n');
stream.end('Последняя строка');  // закрывает поток

stream.on('finish', () => console.log('Запись завершена'));
stream.on('error', (err) => console.error(err));
```

**Pipe — соединение потоков** (основной паттерн Node.js):

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Копирование файла с сжатием (не загружая в память)
const readStream = fs.createReadStream('./input.txt');
const gzipStream = zlib.createGzip();
const writeStream = fs.createWriteStream('./input.txt.gz');

readStream.pipe(gzipStream).pipe(writeStream);
// или цепочкой: readStream.pipe(gzipStream).pipe(writeStream)

writeStream.on('finish', () => console.log('Файл сжат'));
```

**Кастомный Transform поток (обработка данных на лету)** :

```javascript
const { Transform } = require('stream');

const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    // chunk — Buffer или строка
    const result = chunk.toString().toUpperCase();
    this.push(result);
    callback(); // важно вызывать после обработки
  }
});

process.stdin.pipe(upperCaseTransform).pipe(process.stdout);
// echo "hello" | node script.js -> HELLO
```

**Потоки в HTTP (почему Node.js эффективен)** :

```javascript
const http = require('http');
const fs = require('fs');

http.createServer((req, res) => {
  // ПЛОХО: читает весь файл в память
  // const data = fs.readFileSync('./video.mp4');
  // res.end(data);
  
  // ХОРОШО: потоковая передача
  const stream = fs.createReadStream('./video.mp4');
  stream.pipe(res); // автоматически управляет backpressure
  
  stream.on('error', () => res.statusCode = 500);
}).listen(3000);
```

**Обратное давление (backpressure)** — механизм контроля, когда потребитель не успевает за производителем:

```javascript
const fs = require('fs');
const readStream = fs.createReadStream('./bigfile.txt');
const writeStream = fs.createWriteStream('./copy.txt');

readStream.on('data', (chunk) => {
  const canWrite = writeStream.write(chunk);
  if (!canWrite) {
    readStream.pause();           // приостанавливаем чтение
    writeStream.once('drain', () => readStream.resume()); // когда запишет
  }
});
// pipe() делает это автоматически
```

**Полезные утилиты для потоков**:

```javascript
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

// Современный способ с правильной обработкой ошибок
try {
  await pipeline(
    fs.createReadStream('./input.txt'),
    zlib.createGzip(),
    fs.createWriteStream('./input.txt.gz')
  );
  console.log('Готово');
} catch (err) {
  console.error('Ошибка в pipeline:', err);
}
```

**Стриминг больших JSON-массивов** (паттерн для API):

```javascript
// server.js — потоковая генерация JSON
const { Readable } = require('stream');

app.get('/users', (req, res) => {
  res.setHeader('Content-Type', 'application/json');
  res.write('[');
  
  let first = true;
  const stream = Readable.from(async function*() {
    for (let i = 0; i < 1000000; i++) {
      yield first ? `{"id":${i}}` : `,{"id":${i}}`;
      first = false;
    }
  }());
  
  stream.pipe(res, { end: false });
  stream.on('end', () => res.end(']'));
});
```

**Buffer vs строка — практическое правило**:

| Данные | Использовать |
|--------|--------------|
| Текст (JSON, HTML, CSV) | строки + `'utf8'` кодировка |
| Изображения, видео, аудио | Buffer |
| Сетевые протоколы (TCP, WebSocket) | Buffer |
| Криптография (хэши, шифрование) | Buffer |
| Бинарные форматы (PDF, ZIP, Excel) | Buffer |

**Память и производительность**:

```javascript
// Плохо: загружает гигабайтный файл в память
const data = fs.readFileSync('./1gb-file.log');
res.end(data);

// Хорошо: использует ~64 КБ памяти
fs.createReadStream('./1gb-file.log').pipe(res);
```

**Типичные ошибки**:
- Забыть вызвать `callback()` в кастомном Transform
- Не обработать `error` события — поток упадёт молча
- Использовать `end` на Writable потоке, когда данные ещё пишутся
- Смешивать `pipe` и ручную запись `write()`



**10. Модуль `path`: работа с путями к файлам на разных ОС**

Модуль `path` решает главную проблему — различия в форматах путей между Windows (обратные слеши `\`) и Unix-системами (прямые слеши `/`). Никогда не склеивайте пути через `+` или шаблонные строки — используйте `path`.

**Базовые методы (используйте везде)** :

```javascript
const path = require('path');

// path.join() — склеивает части с правильным разделителем
const fullPath = path.join('/users', 'john', 'docs', 'file.txt');
console.log(fullPath);
// Linux/macOS: /users/john/docs/file.txt
// Windows: \users\john\docs\file.txt

// path.resolve() — строит абсолютный путь из относительного
console.log(path.resolve('src/index.js'));  
// /home/user/project/src/index.js (от текущей рабочей директории)

console.log(path.resolve('/etc', 'hosts'));
// /etc/hosts

console.log(path.resolve('data', '../config.json'));
// /home/user/project/config.json (нормализует ..)
```

**Ключевое различие: `join` vs `resolve`** :

```javascript
// join — просто склеивает с разделителем
path.join('a', 'b', 'c');     // a/b/c (или a\b\c на Windows)

// resolve — строит АБСОЛЮТНЫЙ путь, обрабатывая ..
path.resolve('a', 'b', 'c');  // /current/working/dir/a/b/c
path.resolve('/a', 'b', 'c'); // /a/b/c
```

**Разбор пути на компоненты** :

```javascript
const filePath = '/home/user/project/src/app.js';

console.log(path.parse(filePath));
// {
//   root: '/',
//   dir: '/home/user/project/src',
//   base: 'app.js',
//   ext: '.js',
//   name: 'app'
// }

console.log(path.dirname(filePath));   // /home/user/project/src
console.log(path.basename(filePath));  // app.js
console.log(path.basename(filePath, '.js')); // app (без расширения)
console.log(path.extname(filePath));   // .js
```

**Сборка пути из компонентов** :

```javascript
const parsed = path.parse('/data/logs/error.log');
const rebuilt = path.format({
  root: parsed.root,
  dir: parsed.dir,
  base: parsed.base
});
console.log(rebuilt); // /data/logs/error.log
```

**Нормализация путей** :

```javascript
console.log(path.normalize('/foo/bar//baz/asdf/quux/..'));
// /foo/bar/baz/asdf (убирает лишние слеши и обрабатывает ..)

console.log(path.normalize('C:\\temp\\\\foo\\..\\bar'));
// Windows: C:\temp\bar
```

**Определение абсолютности пути** :

```javascript
console.log(path.isAbsolute('/home/user'));     // true (Unix)
console.log(path.isAbsolute('C:\\Windows'));    // true (Windows)
console.log(path.isAbsolute('./file.txt'));     // false
```

**Работа с относительными путями** :

```javascript
// Вычисление относительного пути от одного к другому
const fromPath = '/var/www/html/index.html';
const toPath = '/var/www/images/logo.png';

const relative = path.relative(fromPath, toPath);
console.log(relative); // '../../images/logo.png'
```

**Разделители (зависят от ОС)** :

```javascript
console.log(path.sep);     // '/' на Unix, '\' на Windows
console.log(path.delimiter); // ':' на Unix, ';' на Windows

// Пример разбора PATH переменной
const envPath = process.env.PATH;
const directories = envPath.split(path.delimiter);
console.log(directories);
```

**Кроссплатформенные трюки** :

```javascript
// Принудительно получить Unix-стиль (для Docker, URLs)
const toUnixPath = (filePath) => filePath.split(path.sep).join('/');
console.log(toUnixPath('C:\\Users\\john\\file.txt')); // C:/Users/john/file.txt

// Проверка, является ли путь поддиректорией
function isSubdirectory(parent, child) {
  const relative = path.relative(parent, child);
  return relative && !relative.startsWith('..') && !path.isAbsolute(relative);
}
```

**Частые ошибки** :

```javascript
// ❌ НЕПРАВИЛЬНО
const wrong = __dirname + '/data/config.json';  // на Windows будет \data/config.json
const wrong2 = `./data/${fileName}`;            // нет нормализации

// ✅ ПРАВИЛЬНО
const correct = path.join(__dirname, 'data', 'config.json');
const correct2 = path.join('.', 'data', fileName);
```

**Практический пример: сканер файлов** :

```javascript
const fs = require('fs/promises');
const path = require('path');

async function findFiles(dir, extension) {
  const entries = await fs.readdir(dir, { withFileTypes: true });
  
  const files = await Promise.all(entries.map(async (entry) => {
    const fullPath = path.join(dir, entry.name);
    
    if (entry.isDirectory()) {
      return findFiles(fullPath, extension);
    } else if (entry.isFile() && path.extname(entry.name) === extension) {
      return fullPath;
    }
    return [];
  }));
  
  return files.flat();
}

// Использование
const jsFiles = await findFiles('./src', '.js');
console.log(jsFiles);
```

**Работа с `__dirname` в ES-модулях** :

```javascript
// В ES-модулях __dirname не определён
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const configPath = join(__dirname, 'config', 'app.json');
```

**Безопасность: защита от path traversal** :

```javascript
const safeJoin = (base, userPath) => {
  const resolved = path.resolve(base, userPath);
  // Проверка, что результат внутри base
  if (!resolved.startsWith(path.resolve(base))) {
    throw new Error('Path traversal detected');
  }
  return resolved;
};

// Пример
try {
  const userInput = '../../../etc/passwd';
  const filePath = safeJoin('/app/storage', userInput);
  console.log(filePath); // Ошибка: Path traversal detected
} catch (err) {
  console.error(err.message);
}
```

**Мнемоническое правило**: если работаете с путями в Node.js — всегда используйте `path`. Даже если кажется, что ваш код никогда не запустят на Windows. Однажды это случится, и конкатенация через `+` рухнет.



**11. Модуль `os`: информация о системе**

Модуль `os` предоставляет доступ к операционной системе, на которой выполняется Node.js. Используется для адаптации поведения приложения под среду, мониторинга ресурсов и отладки.

**Основные методы (без импорта - встроенный модуль)** :

```javascript
const os = require('os');

// Архитектура процессора
console.log(os.arch());        // 'x64', 'arm64', 'ia32'

// Платформа
console.log(os.platform());    // 'linux', 'win32', 'darwin' (macOS)

// Тип ОС
console.log(os.type());        // 'Linux', 'Windows_NT', 'Darwin'

// Версия ядра
console.log(os.release());     // '5.15.0-91-generic' (Linux), '10.0.19045' (Windows)

// Имя хоста
console.log(os.hostname());    // 'my-server'

// Время работы системы (в секундах)
console.log(`Система работает ${Math.floor(os.uptime() / 3600)} часов`);
```

**Информация о пользователе** :

```javascript
const os = require('os');

// Текущий пользователь
console.log(os.userInfo());
// {
//   uid: 1000,
//   gid: 1000,
//   username: 'john',
//   homedir: '/home/john',
//   shell: '/bin/bash'
// }

// Домашняя директория (быстрый доступ)
console.log(os.homedir());     // '/home/john'

// Временная директория
console.log(os.tmpdir());      // '/tmp' (Linux), 'C:\Users\john\AppData\Local\Temp' (Windows)

// Конец строки (зависит от ОС)
console.log(os.EOL);           // '\n' (Unix), '\r\n' (Windows)
```

**Информация о процессоре** :

```javascript
const os = require('os');

// Количество логических ядер (включая Hyper-Threading)
console.log(os.cpus().length);    // 8 (например)

// Детальная информация о каждом ядре
const cpus = os.cpus();
cpus.forEach((cpu, index) => {
  console.log(`Ядро ${index}: ${cpu.model}`);
  console.log(`  Скорость: ${cpu.speed} MHz`);
  console.log(`  Время: user=${cpu.times.user}s, sys=${cpu.times.sys}s, idle=${cpu.times.idle}s`);
});

// Загрузка процессора (средняя за 1, 5, 15 минут) — только Unix
console.log(os.loadavg());     // [2.34, 1.89, 1.56] (Linux/macOS)
// На Windows возвращает [0, 0, 0] (не поддерживается)
```

**Память и приоритет** :

```javascript
const os = require('os');

// Общая и свободная память (в байтах)
const totalMem = os.totalmem();
const freeMem = os.freemem();

console.log(`RAM: ${(totalMem / 1024 ** 3).toFixed(2)} GB всего`);
console.log(`Свободно: ${(freeMem / 1024 ** 3).toFixed(2)} GB`);
console.log(`Используется: ${((totalMem - freeMem) / 1024 ** 3).toFixed(2)} GB`);

// Приоритет процесса (Unix: -20..19, Windows: другой диапазон)
try {
  os.setPriority(0, 10);      // установить приоритет для PID 0 (текущий процесс)
  console.log('Приоритет:', os.getPriority());
} catch (err) {
  console.log('Недостаточно прав для изменения приоритета');
}
```

**Сетевые интерфейсы** :

```javascript
const os = require('os');

const networkInterfaces = os.networkInterfaces();
console.log(networkInterfaces);

// Фильтрация активных IPv4 адресов
const activeIPv4 = [];
for (const [name, interfaces] of Object.entries(networkInterfaces)) {
  for (const iface of interfaces) {
    if (iface.family === 'IPv4' && !iface.internal) {
      activeIPv4.push({ name, address: iface.address, mac: iface.mac });
    }
  }
}
console.log('Активные сетевые интерфейсы:', activeIPv4);
```

**Определение окружения (практический кейс)** :

```javascript
const os = require('os');

function getEnvironmentInfo() {
  const platform = os.platform();
  const isWindows = platform === 'win32';
  const isMac = platform === 'darwin';
  const isLinux = platform === 'linux';
  
  return {
    isWindows,
    isMac,
    isLinux,
    isProduction: process.env.NODE_ENV === 'production',
    isCI: !!process.env.CI,
    tempDir: os.tmpdir(),
    eol: os.EOL,
    shell: os.userInfo().shell || 'unknown'
  };
}

// Пример использования: разные команды для разных ОС
const env = getEnvironmentInfo();
const clearCommand = env.isWindows ? 'cls' : 'clear';
const pathSeparator = env.isWindows ? ';' : ':';
```

**Мониторинг ресурсов в реальном времени** :

```javascript
const os = require('os');

function getSystemStats() {
  const totalMem = os.totalmem();
  const freeMem = os.freemem();
  const usedMem = totalMem - freeMem;
  
  // Загрузка CPU (приблизительная)
  const cpus = os.cpus();
  let idle = 0, total = 0;
  cpus.forEach(cpu => {
    for (const type in cpu.times) {
      total += cpu.times[type];
    }
    idle += cpu.times.idle;
  });
  
  return {
    memoryUsagePercent: ((usedMem / totalMem) * 100).toFixed(1),
    cpuUsagePercent: (100 - (idle / total) * 100).toFixed(1),
    uptimeHours: (os.uptime() / 3600).toFixed(1),
    loadAvg: os.loadavg()
  };
}

// Мониторинг каждые 5 секунд
setInterval(() => {
  console.log(new Date().toISOString(), getSystemStats());
}, 5000);
```

**Константы и утилиты** :

```javascript
const os = require('os');

// Количество свободных слотов (endianness)
console.log(os.endianness());     // 'LE' (Little-Endian) или 'BE'

// Приоритет процесса
console.log(os.constants.priority);
// { PRIORITY_LOW: 19, PRIORITY_BELOW_NORMAL: 10, PRIORITY_NORMAL: 0, ... }

// Сигналы (только Unix)
if (os.platform() !== 'win32') {
  console.log(os.constants.signals.SIGINT);  // 2
  console.log(os.constants.signals.SIGTERM); // 15
}
```

**Практический пример: проверка системных требований** :

```javascript
const os = require('os');

function checkSystemRequirements() {
  const requirements = {
    minMemoryGB: 2,
    minCores: 2,
    supportedPlatforms: ['linux', 'darwin'] // исключаем Windows для продакшена
  };
  
  const totalMemGB = os.totalmem() / 1024 ** 3;
  const cpuCores = os.cpus().length;
  const platform = os.platform();
  
  const issues = [];
  
  if (totalMemGB < requirements.minMemoryGB) {
    issues.push(`Недостаточно памяти: ${totalMemGB.toFixed(1)} GB < ${requirements.minMemoryGB} GB`);
  }
  
  if (cpuCores < requirements.minCores) {
    issues.push(`Мало ядер CPU: ${cpuCores} < ${requirements.minCores}`);
  }
  
  if (!requirements.supportedPlatforms.includes(platform)) {
    issues.push(`Неподдерживаемая платформа: ${platform}`);
  }
  
  return {
    passed: issues.length === 0,
    issues,
    specs: {
      memoryGB: totalMemGB,
      cpuCores,
      platform,
      hostname: os.hostname()
    }
  };
}

console.log(checkSystemRequirements());
```

**Когда использовать `os`** :
- Адаптация путей к временным файлам (`os.tmpdir()`)
- Определение количества ядер для пула воркеров (`os.cpus().length`)
- Логирование окружения при старте приложения
- Автоматический выбор конфигурации в зависимости от ОС
- Системные метрики для мониторинга (Prometheus, Grafana)

**Важное замечание** : некоторые методы (`os.loadavg()`, `os.setPriority()`) работают не на всех платформах или требуют прав администратора. Всегда проверяйте поведение в целевой среде.



**12. Модуль `events`: эмиттеры событий и слушатели**

Модуль `events` — основа событийной архитектуры Node.js. Многие встроенные объекты (HTTP-сервер, потоки, сокеты) наследуют от `EventEmitter`. Этот модуль реализует паттерн "наблюдатель" (Observer) внутри процесса.

**Базовое использование EventEmitter**:

```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();

// Добавление слушателя
emitter.on('user:login', (username) => {
  console.log(`Пользователь ${username} вошёл в систему`);
});

// Добавление одноразового слушателя (сработает только раз)
emitter.once('app:start', () => {
  console.log('Приложение запущено (только при первом запуске)');
});

// Генерация события
emitter.emit('user:login', 'alice');   // Пользователь alice вошёл в систему
emitter.emit('app:start');              // Приложение запущено...
emitter.emit('app:start');              // (ничего не произойдёт, once сработал один раз)
```

**Цепочки и множественные слушатели**:

```javascript
const emitter = new EventEmitter();

// Несколько слушателей на одно событие
emitter.on('data', (chunk) => {
  console.log(`Первый слушатель: ${chunk.length} байт`);
});

emitter.on('data', (chunk) => {
  console.log(`Второй слушатель: ${chunk.toString().toUpperCase()}`);
});

// Порядок вызова — в порядке регистрации
emitter.emit('data', Buffer.from('hello'));
// Первый слушатель: 5 байт
// Второй слушатель: HELLO
```

**Управление слушателями**:

```javascript
const emitter = new EventEmitter();

function handler1() { console.log('Handler 1'); }
function handler2() { console.log('Handler 2'); }

emitter.on('event', handler1);
emitter.on('event', handler2);

// Удаление конкретного слушателя
emitter.off('event', handler1);  // или removeListener
emitter.emit('event');            // только Handler 2

// Удаление всех слушателей события
emitter.removeAllListeners('event');

// Удаление всех слушателей ВСЕХ событий (осторожно!)
emitter.removeAllListeners();

// Проверка наличия слушателей
console.log(emitter.listenerCount('event'));        // 0
console.log(emitter.eventNames());                  // []
```

**Передача аргументов**:

```javascript
const emitter = new EventEmitter();

// Можно передавать любое количество аргументов
emitter.on('order', (id, items, total, callback) => {
  console.log(`Заказ ${id}: ${items.length} товаров на сумму ${total}`);
  callback('processed');
});

emitter.emit('order', 123, ['book', 'pen'], 45.99, (status) => {
  console.log(`Статус: ${status}`);
});
```

**Обработка ошибок** (важное правило):

```javascript
const emitter = new EventEmitter();

// Слушатель на событие 'error' — обязателен, иначе процесс упадёт
emitter.on('error', (err) => {
  console.error('Перехвачена ошибка:', err.message);
});

// Генерация ошибки
emitter.emit('error', new Error('Что-то пошло не так'));

// Если нет слушателя 'error' — Node.js выбросит исключение и процесс завершится
```

**Создание собственных классов с EventEmitter** (паттерн наследования):

```javascript
const EventEmitter = require('events');

class FileUploader extends EventEmitter {
  upload(filePath) {
    this.emit('start', filePath);
    
    // Симуляция загрузки
    let progress = 0;
    const interval = setInterval(() => {
      progress += 10;
      this.emit('progress', { file: filePath, percent: progress });
      
      if (progress >= 100) {
        clearInterval(interval);
        this.emit('complete', filePath);
      }
    }, 100);
  }
}

// Использование
const uploader = new FileUploader();
uploader.on('start', (file) => console.log(`Начало: ${file}`));
uploader.on('progress', (data) => console.log(`Прогресс: ${data.percent}%`));
uploader.on('complete', (file) => console.log(`Завершено: ${file}`));
uploader.on('error', (err) => console.error(err));

uploader.upload('photo.jpg');
```

**Асинхронные слушатели и порядок выполнения**:

```javascript
const emitter = new EventEmitter();

// Слушатели выполняются синхронно ПО УМОЛЧАНИЮ
emitter.on('sync', () => {
  console.log('1. Синхронный');
});

emitter.on('sync', () => {
  console.log('2. Синхронный');
});

emitter.emit('sync');
// Вывод: 1, 2 (строго последовательно)

// Для асинхронного выполнения используйте setImmediate или process.nextTick
emitter.on('async', () => {
  setImmediate(() => console.log('Асинхронный'));
});
```

**Максимальное количество слушателей** (предотвращение утечек памяти):

```javascript
const emitter = new EventEmitter();

// По умолчанию — 10 слушателей на событие
console.log(emitter.getMaxListeners()); // 10

// Добавляем 11 слушателей — будет предупреждение
for (let i = 0; i < 11; i++) {
  emitter.on('event', () => {});
}
// Warning: Possible EventEmitter memory leak detected. 11 listeners added.

// Увеличиваем лимит
emitter.setMaxListeners(20);
console.log(emitter.getMaxListeners()); // 20
```

**Метод `prependListener`** (добавление в начало очереди):

```javascript
const emitter = new EventEmitter();

emitter.on('test', () => console.log('Второй'));
emitter.prependListener('test', () => console.log('Первый'));

emitter.emit('test');
// Вывод: Первый, Второй

// Аналогично для once
emitter.prependOnceListener('test', () => console.log('Выполнится первым, но один раз'));
```

**Практический пример: система событий в приложении**:

```javascript
const EventEmitter = require('events');

class AppEvents extends EventEmitter {
  constructor() {
    super();
    this.setMaxListeners(50); // Увеличиваем для крупного приложения
  }
  
  logEvent(event, ...args) {
    console.log(`[${new Date().toISOString()}] ${event}:`, ...args);
    this.emit(event, ...args);
  }
}

const appEvents = new AppEvents();

// Модуль логирования
appEvents.on('user:registered', (user) => {
  console.log(`Новый пользователь: ${user.email}`);
});

// Модуль отправки email
appEvents.on('user:registered', async (user) => {
  // await sendWelcomeEmail(user.email);
  console.log(`Отправлено приветствие на ${user.email}`);
});

// Модуль аналитики
appEvents.on('user:registered', (user) => {
  // trackEvent('user_registered', { plan: user.plan });
  console.log(`Событие отправлено в аналитику`);
});

// Генерация события
appEvents.logEvent('user:registered', {
  id: 123,
  email: 'user@example.com',
  plan: 'premium'
});
```

**Сравнение EventEmitter с браузерными событиями**:

| Особенность | Node.js EventEmitter | Браузерный EventTarget |
|-------------|---------------------|------------------------|
| Синтаксис | `on()`, `emit()` | `addEventListener()`, `dispatchEvent()` |
| Цепочки вызовов | Да (`emitter.on().on()`) | Нет |
| Приоритет слушателей | `prependListener` | опция `once` |
| Производительность | Высокая (нативный код) | Средняя |

**Распространённые ошибки**:

```javascript
// ❌ Забыть убрать слушатели (утечка памяти)
function leakyFunction() {
  const emitter = new EventEmitter();
  emitter.on('data', () => { /* обработчик */ });
  // emitter не удаляется, слушатель остаётся
}

// ✅ Удалять слушатели или использовать once
function correctFunction() {
  const emitter = new EventEmitter();
  emitter.once('data', handler); // самоудалится
  // или
  const handler = () => {};
  emitter.on('data', handler);
  // позже: emitter.off('data', handler);
}

// ❌ Синхронный emit внутри асинхронного обработчика
emitter.on('request', (req) => {
  fs.readFile('file.txt', (err, data) => {
    emitter.emit('file:loaded', data); // сработает, но порядок непредсказуем
  });
});
```

**Наследование от EventEmitter (классический Node.js стиль)**:

```javascript
const EventEmitter = require('events');
const util = require('util');

function Logger(prefix) {
  EventEmitter.call(this);
  this.prefix = prefix;
}

util.inherits(Logger, EventEmitter); // старый способ

Logger.prototype.log = function(message) {
  const formatted = `[${this.prefix}] ${message}`;
  this.emit('log', formatted);
  return formatted;
};

// Современный способ с class
class ModernLogger extends EventEmitter {
  constructor(prefix) {
    super();
    this.prefix = prefix;
  }
  
  log(message) {
    const formatted = `[${this.prefix}] ${message}`;
    this.emit('log', formatted);
    return formatted;
  }
}
```

**Когда использовать EventEmitter**:
- Реализация паттерна Observer в одном процессе
- Создание API, которое уведомляет о прогрессе или состоянии
- Декаппликация модулей (модуль А генерирует события, модуль Б реагирует)
- Расширение встроенных объектов (потоков, серверов)

**Что нужно запомнить**: EventEmitter работает только в пределах одного процесса Node.js. Для событий между разными процессами или серверами используйте очереди сообщений (Redis, RabbitMQ, Kafka).



**13. Понимание цикла событий (Event Loop) в Node.js**

Цикл событий — механизм, который позволяет Node.js выполнять неблокирующие I/O операции, несмотря на однопоточность. Это не "магия", а строгая конечная машина состояний с определёнными фазами и приоритетами.

**Как выглядит упрощённо**:

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← системные операции (TCP, TLS)
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← внутреннее использование
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │         poll              │  ← получение новых I/O событий
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │         check             │  ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  └      close callbacks      │  ← socket.on('close')
└──────────────────────────────┘
```

**Фазы цикла событий (от наиболее важного)** :

**1. Фаза timers** — выполняет колбэки `setTimeout` и `setInterval`, чей таймаут истёк.

```javascript
setTimeout(() => {
  console.log('Выполнится в фазе timers');
}, 0);

setInterval(() => {
  console.log('Выполняется каждые 100 мс');
}, 100);
```

**2. Фаза poll (самая важная)** — сердце цикла:
- Выполняет колбэки I/O операций (чтение файлов, сетевые запросы)
- Если очередь пуста, ждёт новые события или переходит к check

```javascript
const fs = require('fs');

fs.readFile('file.txt', (err, data) => {
  console.log('Колбэк выполнится в фазе poll');
});
```

**3. Фаза check** — колбэки `setImmediate()`

**4. Фаза close** — события закрытия (сокеты, соединения)

**Порядок выполнения: макро- и микро-задачи** (критическое различие):

```javascript
// Микро-задачи (выполняются между фазами, имеют высший приоритет)
Promise.resolve().then(() => console.log('Promise microtask'));
process.nextTick(() => console.log('nextTick microtask')); // самый высокий приоритет

// Макро-задачи (выполняются в фазах цикла)
setTimeout(() => console.log('setTimeout macro'), 0);
setImmediate(() => console.log('setImmediate macro'));

console.log('Синхронный код');

// Вывод:
// Синхронный код
// nextTick microtask
// Promise microtask
// setTimeout macro
// setImmediate macro
```

**`process.nextTick()` vs `setImmediate()`** — путаница в названиях:

| Метод | Фаза выполнения | Приоритет | Использовать для |
|-------|----------------|-----------|------------------|
| `process.nextTick()` | После текущей операции, **до** следующей фазы | Самый высокий | Срочный перехват ошибок, продолжение выполнения |
| `setImmediate()` | Фаза `check` (после `poll`) | Ниже, чем nextTick | Отложить операцию, но выполнить до таймеров |

```javascript
process.nextTick(() => console.log('1. nextTick'));
setImmediate(() => console.log('3. setImmediate'));
setTimeout(() => console.log('4. setTimeout'), 0);
Promise.resolve().then(() => console.log('2. Promise'));

// Вывод: 1, 2, 3, 4 (но 3 и 4 могут меняться местами)
```

**Почему `setTimeout(fn, 0)` не гарантирует выполнение через 0 мс**:

```javascript
const start = Date.now();

setTimeout(() => {
  console.log(`Таймер сработал через ${Date.now() - start} мс`);
}, 0);

// Если выполнять тяжёлую синхронную операцию
for (let i = 0; i < 1e9; i++) {} // ~500 мс

// Таймер сработает через ~500 мс, а не 0
```

**Блокировка цикла событий (главная опасность)** :

```javascript
const http = require('http');

http.createServer((req, res) => {
  if (req.url === '/block') {
    // ПЛОХО: блокирует весь сервер на 5 секунд
    const end = Date.now() + 5000;
    while (Date.now() < end) {}
    res.end('OK');
  }
  res.end('Fast');
}).listen(3000);

// Второй запрос во время блокировки будет ждать
```

**Как освободить цикл событий**:

```javascript
// Плохо: синхронная обработка большого массива
const largeArray = Array(1e6).fill(0).map((_, i) => i);
const sum = largeArray.reduce((a, b) => a + b, 0);

// Хорошо: разбивка на микро-задачи
function processLargeArrayAsync(array, callback) {
  let index = 0;
  let sum = 0;
  const chunkSize = 1000;
  
  function processChunk() {
    const end = Math.min(index + chunkSize, array.length);
    for (let i = index; i < end; i++) {
      sum += array[i];
    }
    index = end;
    
    if (index < array.length) {
      setImmediate(processChunk); // даём циклу обработать другие события
    } else {
      callback(sum);
    }
  }
  
  processChunk();
}
```

**Проверка, заблокирован ли цикл (эксперимент)** :

```javascript
let lastCheck = Date.now();

setInterval(() => {
  const now = Date.now();
  const delay = now - lastCheck;
  if (delay > 105) { // интервал 100 мс + погрешность
    console.log(`Цикл заблокирован на ${delay - 100} мс`);
  }
  lastCheck = now;
}, 100);

// Заблокируем цикл
setTimeout(() => {
  const end = Date.now() + 500;
  while (Date.now() < end) {}
}, 50);
```

**Пример из реального сервера (порядок выполнения)** :

```javascript
const fs = require('fs');

console.log('1. Синхронный старт');

setTimeout(() => console.log('2. setTimeout'), 0);
setImmediate(() => console.log('3. setImmediate'));

fs.readFile(__filename, () => {
  console.log('4. fs.readFile (I/O)');
  
  setTimeout(() => console.log('5. setTimeout внутри I/O'), 0);
  setImmediate(() => console.log('6. setImmediate внутри I/O'));
  process.nextTick(() => console.log('7. nextTick внутри I/O'));
});

Promise.resolve().then(() => console.log('8. Promise'));

process.nextTick(() => console.log('9. nextTick'));

console.log('10. Синхронный конец');

// Вывод (стабильный):
// 1. Синхронный старт
// 10. Синхронный конец
// 9. nextTick
// 8. Promise
// 2. setTimeout  (или 3, зависит от производительности)
// 3. setImmediate (или 2)
// 4. fs.readFile
// 7. nextTick внутри I/O
// 5. setTimeout внутри I/O
// 6. setImmediate внутри I/O
```

**Когда ставить CPU-интенсивные задачи**:

```javascript
const { Worker } = require('worker_threads');

// ❌ Блокируем цикл
function calculatePrimes(limit) {
  // сложные вычисления...
}

// ✅ Выносим в воркер
const worker = new Worker(`
  const { parentPort } = require('worker_threads');
  // вычисления...
  parentPort.postMessage(result);
`, { eval: true });
```

**Практические правила**:

| Операция | Где выполнять | Почему |
|----------|--------------|--------|
| Чтение файла | Нативный I/O (не блокирует) | `fs.readFile` внутри C++ слоя |
| HTTP запрос | Нативный I/O | `http.get` не блокирует |
| JSON.parse большого объекта | **Блокирует** | Выполняется в главном потоке |
| Шифрование (crypto) | **Блокирует** | В главном потоке, используйте `crypto.pbkdf2` с асинхронным API |
| Рендеринг отчётов | Вынести в воркер | CPU-intensive |

**Как наблюдать за циклом событий**:

```javascript
const CLS = require('cls-hooked'); // или использовать perf_hooks

// Встроенный мониторинг
const { performance, PerformanceObserver } = require('perf_hooks');

const obs = new PerformanceObserver((items) => {
  console.log(items.getEntries()[0].duration);
  performance.clearMarks();
});
obs.observe({ entryTypes: ['function'] });

// Измерение времени между тиками
let lastTick = performance.now();
setInterval(() => {
  const now = performance.now();
  const delta = now - lastTick;
  if (delta > 50) console.warn(`Long tick: ${delta}ms`);
  lastTick = now;
}, 10);
```

**Запоминайте**: Event loop — это не бесконечный while(true), а управляемая очередь. Пока в очередях есть задачи — цикл работает. Как только все задачи выполнены — Node.js завершает процесс (если не осталось активных слушателей, например, HTTP сервер удерживает процесс).



**14. Блокирующий vs Неблокирующий код (асинхронные паттерны)**

Блокирующий код останавливает выполнение программы до завершения операции. Неблокирующий код запускает операцию и продолжает выполнение, получая уведомление о результате позже. Node.js спроектирован для неблокирующих операций — это его главное преимущество и одновременно источник сложности.

**Наглядное сравнение**:

```javascript
const fs = require('fs');

// === БЛОКИРУЮЩАЯ (синхронная) ВЕРСИЯ ===
console.log('1. Начало');
const data = fs.readFileSync('./file.txt', 'utf8'); // останавливается здесь
console.log('2. Файл прочитан:', data.length);
console.log('3. Конец');

// Вывод:
// 1. Начало
// 2. Файл прочитан: 1024
// 3. Конец

// === НЕБЛОКИРУЮЩАЯ (асинхронная) ВЕРСИЯ ===
console.log('1. Начало');
fs.readFile('./file.txt', 'utf8', (err, data) => {
  console.log('2. Файл прочитан:', data.length);
});
console.log('3. Конец');

// Вывод:
// 1. Начало
// 3. Конец
// 2. Файл прочитан: 1024
```

**Три паттерна асинхронного кода в Node.js**:

```javascript
// 1. КОЛБЭКИ (классический, Callback Hell)
fs.readFile('a.txt', 'utf8', (err, a) => {
  if (err) return console.error(err);
  fs.readFile('b.txt', 'utf8', (err, b) => {
    if (err) return console.error(err);
    fs.readFile('c.txt', 'utf8', (err, c) => {
      console.log(a, b, c);
    });
  });
});

// 2. PROMISES (лучше)
const fsPromises = require('fs').promises;
fsPromises.readFile('a.txt', 'utf8')
  .then(a => fsPromises.readFile('b.txt', 'utf8').then(b => [a, b]))
  .then(([a, b]) => fsPromises.readFile('c.txt', 'utf8').then(c => [a, b, c]))
  .then(([a, b, c]) => console.log(a, b, c))
  .catch(console.error);

// 3. ASYNC/AWAIT (современный, самый читаемый)
async function readAllFiles() {
  try {
    const [a, b, c] = await Promise.all([
      fsPromises.readFile('a.txt', 'utf8'),
      fsPromises.readFile('b.txt', 'utf8'),
      fsPromises.readFile('c.txt', 'utf8')
    ]);
    console.log(a, b, c);
  } catch (err) {
    console.error(err);
  }
}
```

**Какие операции блокируют цикл событий**:

| Тип операции | Блокирует? | Пример |
|-------------|-----------|--------|
| Чтение файла (синхронное) | **Да** | `fs.readFileSync()` |
| Чтение файла (асинхронное) | Нет | `fs.readFile()` |
| Сетевой запрос | Нет | `http.get()`, `fetch()` |
| Таймеры | Нет | `setTimeout()`, `setInterval()` |
| JSON.parse() | **Да** | `JSON.parse(largeString)` |
| Циклы и вычисления | **Да** | `for`, `while`, `reduce` |
| Криптография (синхронная) | **Да** | `crypto.createHash().update().digest()` |
| Криптография (асинхронная) | Нет | `crypto.pbkdf2()` |
| Запрос к БД (через драйвер) | Зависит от драйвера | `mongoose.exec()` обычно не блокирует |

**Почему "асинхронный" не значит "параллельный"**:

```javascript
const fs = require('fs/promises');

// Эти три операции выполняются ПОСЛЕДОВАТЕЛЬНО (общее время = сумма)
async function sequentialRead() {
  const a = await fs.readFile('a.txt', 'utf8');
  const b = await fs.readFile('b.txt', 'utf8');
  const c = await fs.readFile('c.txt', 'utf8');
  // Время: t(a) + t(b) + t(c)
}

// Эти три операции выполняются ПАРАЛЛЕЛЬНО (общее время = max)
async function parallelRead() {
  const [a, b, c] = await Promise.all([
    fs.readFile('a.txt', 'utf8'),
    fs.readFile('b.txt', 'utf8'),
    fs.readFile('c.txt', 'utf8')
  ]);
  // Время: max(t(a), t(b), t(c))
}
```

**Паттерн "Callback Hell" и его решение**:

```javascript
// 🔴 АД КОЛБЭКОВ
doSomething(param1, (err, result1) => {
  if (err) handleError(err);
  doSomethingElse(result1, (err, result2) => {
    if (err) handleError(err);
    doThirdThing(result2, (err, result3) => {
      if (err) handleError(err);
      finalCallback(result3);
    });
  });
});

// 🟢 РЕШЕНИЕ 1: Именованные функции
function step3(err, result3) {
  if (err) return handleError(err);
  finalCallback(result3);
}

function step2(err, result2) {
  if (err) return handleError(err);
  doThirdThing(result2, step3);
}

function step1(err, result1) {
  if (err) return handleError(err);
  doSomethingElse(result1, step2);
}

doSomething(param1, step1);

// 🟢 РЕШЕНИЕ 2: Промисыфикация
const { promisify } = require('util');
const doSomethingAsync = promisify(doSomething);
const doSomethingElseAsync = promisify(doSomethingElse);

async function betterWay() {
  const result1 = await doSomethingAsync(param1);
  const result2 = await doSomethingElseAsync(result1);
  const result3 = await doThirdThingAsync(result2);
  finalCallback(result3);
}
```

**Распространённая ошибка: забытый `await`**:

```javascript
// ❌ ОШИБКА: Promise не выполнен
async function getData() {
  const data = fs.readFile('file.txt', 'utf8'); // забыли await
  console.log(data); // Promise { <pending> }
  return data; // возвращает Promise, а не строку
}

// ✅ ПРАВИЛЬНО
async function getData() {
  const data = await fs.readFile('file.txt', 'utf8');
  console.log(data); // строка
  return data;
}
```

**Преобразование колбэк-функций в промисы**:

```javascript
const { promisify } = require('util');
const fs = require('fs');

// Старый способ с колбэком
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Новый способ через promisify
const readFileAsync = promisify(fs.readFile);
const data = await readFileAsync('file.txt', 'utf8');

// Собственная функция с колбэком
function waitAndCallback(ms, callback) {
  setTimeout(() => callback(null, `Ждал ${ms} мс`), ms);
}

const waitAsync = promisify(waitAndCallback);
console.log(await waitAsync(1000));
```

**Проверка: блокирующий или нет**:

```javascript
// Эксперимент: измеряем влияние на цикл событий
const http = require('http');

let requestCount = 0;
http.createServer((req, res) => {
  requestCount++;
  
  // Блокирующая операция
  if (req.url === '/block') {
    const end = Date.now() + 1000;
    while (Date.now() < end) {} // блок на 1 секунду
    res.end(`Blocked. Total requests: ${requestCount}`);
  }
  
  // Неблокирующая операция
  if (req.url === '/nonblock') {
    setTimeout(() => {
      res.end(`Non-blocked. Total requests: ${requestCount}`);
    }, 1000);
  }
}).listen(3000);

// Тест:
// curl http://localhost:3000/block & curl http://localhost:3000/nonblock
// Второй запрос будет ждать завершения первого при /block
```

**CPU-интенсивные задачи: выход из потока**:

```javascript
// ❌ ПЛОХО: блокирует весь сервер
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
app.get('/fib/:n', (req, res) => {
  const result = fibonacci(parseInt(req.params.n)); // блок!
  res.json({ result });
});

// ✅ ХОРОШО: разбиваем на куски
async function fibonacciAsync(n, callback) {
  let result = 0;
  let a = 0, b = 1;
  
  for (let i = 2; i <= n; i++) {
    result = a + b;
    a = b;
    b = result;
    
    // Даём циклу событий поработать каждые 1000 итераций
    if (i % 1000 === 0) {
      await new Promise(resolve => setImmediate(resolve));
    }
  }
  return result;
}

// ✅ ЛУЧШЕ: выносим в Worker Thread
const { Worker } = require('worker_threads');

app.get('/fib/:n', (req, res) => {
  const worker = new Worker(`
    const { parentPort } = require('worker_threads');
    function fib(n) { return n <= 1 ? n : fib(n - 1) + fib(n - 2); }
    parentPort.postMessage(fib(${req.params.n}));
  `, { eval: true });
  
  worker.on('message', result => res.json({ result }));
  worker.on('error', err => res.status(500).json({ error: err.message }));
});
```

**Таблица выбора асинхронного паттерна**:

| Сценарий | Рекомендация | Пример |
|----------|--------------|--------|
| Простые скрипты, инициализация | Синхронный код | `readFileSync` в конфиге |
| HTTP сервер | Async/await | `app.get('/users', async (req, res) => {})` |
| Несколько независимых I/O операций | `Promise.all()` | Параллельное чтение файлов |
| Последовательные операции с зависимостью | `await` в цикле | `for (const id of ids) { await process(id) }` |
| Старый код с колбэками | `util.promisify()` | Обёртка legacy API |
| Обработка потока событий | EventEmitter | Чтение большого файла по частям |

**Золотое правило Node.js**: если операция связана с I/O (диск, сеть, база данных), используйте асинхронный вариант. Если с CPU (расчёты, парсинг, шифрование) — блокирующий, и выносите в отдельный поток или разбивайте на микро-задачи.



**15. Работа с колбэками и колбэками с первым аргументом-ошибкой**

**Колбэк (callback)** — функция, переданная в другую функцию и вызванная после завершения асинхронной операции. **Error-first callback** — соглашение в Node.js, где первым аргументом колбэка всегда идёт ошибка (или `null` при успехе), а последующими — результаты.

**Стандартный паттерн error-first callback**:

```javascript
const fs = require('fs');

// Сигнатура: (err, result) => {}
fs.readFile('./file.txt', 'utf8', (err, data) => {
  if (err) {
    // Первый аргумент — ошибка
    console.error('Ошибка чтения:', err.message);
    return;
  }
  // Второй аргумент — данные
  console.log('Содержимое:', data);
});
```

**Почему именно error-first**:
- Единый стандарт для всех встроенных модулей Node.js
- Легко проверить наличие ошибки одной веткой `if (err)`
- Ошибка не может быть "проглочена" — она явно передаётся

**Создание функций с error-first колбэками**:

```javascript
// Функция, которая принимает колбэк
function divide(a, b, callback) {
  // Валидация
  if (b === 0) {
    // Ошибка — первый аргумент
    callback(new Error('Деление на ноль'));
    return;
  }
  
  // Успех — первый аргумент null, второй результат
  callback(null, a / b);
}

// Использование
divide(10, 2, (err, result) => {
  if (err) {
    console.error(err.message);
    return;
  }
  console.log('Результат:', result); // 5
});

divide(10, 0, (err, result) => {
  if (err) {
    console.error('Ошибка:', err.message); // Ошибка: Деление на ноль
    return;
  }
  console.log(result); // не выполнится
});
```

**Несколько результатов в колбэке**:

```javascript
function getUserData(userId, callback) {
  // Симуляция асинхронной операции
  setTimeout(() => {
    if (userId !== 123) {
      callback(new Error('Пользователь не найден'));
      return;
    }
    
    // Несколько результатов
    callback(null, {
      id: 123,
      name: 'Alice',
      email: 'alice@example.com'
    });
  }, 100);
}

getUserData(123, (err, user) => {
  if (err) throw err;
  console.log(user.name, user.email);
});
```

**Вложенные колбэки (Callback Hell) и его проблемы**:

```javascript
const fs = require('fs');

// 🔴 Антипаттерн: глубокое вложение
fs.readFile('users.json', 'utf8', (err, usersData) => {
  if (err) return console.error(err);
  
  fs.readFile('posts.json', 'utf8', (err, postsData) => {
    if (err) return console.error(err);
    
    fs.readFile('comments.json', 'utf8', (err, commentsData) => {
      if (err) return console.error(err);
      
      // Обработка всех данных
      const users = JSON.parse(usersData);
      const posts = JSON.parse(postsData);
      const comments = JSON.parse(commentsData);
      
      console.log('Данные загружены');
    });
  });
});
```

**Способы борьбы с Callback Hell**:

**1. Именованные функции**:

```javascript
function handleComments(err, commentsData) {
  if (err) return console.error(err);
  const comments = JSON.parse(commentsData);
  console.log('Все данные загружены');
}

function handlePosts(err, postsData) {
  if (err) return console.error(err);
  const posts = JSON.parse(postsData);
  fs.readFile('comments.json', 'utf8', handleComments);
}

function handleUsers(err, usersData) {
  if (err) return console.error(err);
  const users = JSON.parse(usersData);
  fs.readFile('posts.json', 'utf8', handlePosts);
}

fs.readFile('users.json', 'utf8', handleUsers);
```

**2. Модуль `async` (библиотека для контроля потоков)**:

```javascript
const async = require('async');

async.parallel({
  users: (cb) => fs.readFile('users.json', 'utf8', cb),
  posts: (cb) => fs.readFile('posts.json', 'utf8', cb),
  comments: (cb) => fs.readFile('comments.json', 'utf8', cb)
}, (err, results) => {
  if (err) return console.error(err);
  
  const users = JSON.parse(results.users);
  const posts = JSON.parse(results.posts);
  const comments = JSON.parse(results.comments);
  
  console.log('Все данные загружены параллельно');
});
```

**3. Промисыфикация (современный стандарт)**:

```javascript
const { promisify } = require('util');
const fs = require('fs');

const readFileAsync = promisify(fs.readFile);

async function loadAllData() {
  try {
    const [usersData, postsData, commentsData] = await Promise.all([
      readFileAsync('users.json', 'utf8'),
      readFileAsync('posts.json', 'utf8'),
      readFileAsync('comments.json', 'utf8')
    ]);
    
    const users = JSON.parse(usersData);
    const posts = JSON.parse(postsData);
    const comments = JSON.parse(commentsData);
    
    console.log('Данные загружены');
  } catch (err) {
    console.error(err);
  }
}
```

**Типичные ошибки при работе с колбэками**:

```javascript
// ❌ Забыли обработать ошибку
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log(data); // если err !== null, data будет undefined
});

// ✅ Всегда проверяйте err
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Файл не найден, используем значения по умолчанию');
    return;
  }
  console.log(data);
});

// ❌ Вызов колбэка несколько раз
function asyncOperation(callback) {
  setTimeout(() => {
    callback(null, 'First');
    callback(null, 'Second'); // Второй вызов — ошибка
  }, 100);
}

// ✅ Вызывайте колбэк ровно один раз
function asyncOperation(callback) {
  let called = false;
  setTimeout(() => {
    if (called) return;
    called = true;
    callback(null, 'Result');
  }, 100);
}

// ❌ Синхронный вызов колбэка внутри асинхронной функции
function readConfig(callback) {
  if (process.env.CONFIG) {
    // Синхронный вызов
    callback(null, process.env.CONFIG);
  } else {
    // Асинхронный вызов
    fs.readFile('config.json', callback);
  }
}
// Проблема: поведение функции непредсказуемо (синхронное или асинхронное)

// ✅ Делаем поведение единообразным через setImmediate
function readConfig(callback) {
  if (process.env.CONFIG) {
    setImmediate(() => callback(null, process.env.CONFIG));
    return;
  }
  fs.readFile('config.json', callback);
}
```

**Паттерн "колбэк с результатом" для нескольких вызовов**:

```javascript
function parallelRequests(urls, callback) {
  const results = [];
  let completed = 0;
  let hasError = false;
  
  if (urls.length === 0) {
    return callback(null, []);
  }
  
  urls.forEach((url, index) => {
    makeRequest(url, (err, data) => {
      if (hasError) return;
      
      if (err) {
        hasError = true;
        callback(err);
        return;
      }
      
      results[index] = data;
      completed++;
      
      if (completed === urls.length) {
        callback(null, results);
      }
    });
  });
}

// Вспомогательная функция для запроса
function makeRequest(url, callback) {
  setTimeout(() => {
    callback(null, `Data from ${url}`);
  }, Math.random() * 100);
}

// Использование
parallelRequests(['url1', 'url2', 'url3'], (err, results) => {
  if (err) console.error(err);
  else console.log(results);
});
```

**Преобразование колбэк-функции в Promise (ручное)**:

```javascript
function waitAndCallback(ms, callback) {
  setTimeout(() => {
    if (ms < 0) {
      callback(new Error('Отрицательное время'));
    } else {
      callback(null, `Ждал ${ms} мс`);
    }
  }, ms);
}

// Ручная промисфикация
function waitAsync(ms) {
  return new Promise((resolve, reject) => {
    waitAndCallback(ms, (err, result) => {
      if (err) reject(err);
      else resolve(result);
    });
  });
}

// Использование
async function example() {
  try {
    const result = await waitAsync(1000);
    console.log(result);
  } catch (err) {
    console.error(err);
  }
}
```

**Когда колбэки всё ещё лучше промисов**:

```javascript
// Сценарий 1: Несколько последовательных операций с ранним выходом
function processStream(input, callback) {
  let buffer = '';
  
  input.on('data', (chunk) => {
    buffer += chunk;
    if (buffer.includes('END')) {
      input.removeAllListeners('data');
      callback(null, buffer);
    }
  });
  
  input.on('error', callback);
  input.on('end', () => callback(null, buffer));
}

// Сценарий 2: Высокопроизводительные библиотеки (потоки, сокеты)
const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    socket.write('Echo: ' + data);
  });
});
```

**Проверка: сигнатура error-first колбэка**:

```javascript
function isErrorFirstCallback(fn) {
  const fnStr = fn.toString();
  const params = fnStr.match(/\(([^)]*)\)/)?.[1] || '';
  const firstParam = params.split(',')[0]?.trim();
  return firstParam === 'err' || firstParam === 'error';
}

// Примеры
console.log(isErrorFirstCallback((err, data) => {})); // true
console.log(isErrorFirstCallback((error, result) => {})); // true
console.log(isErrorFirstCallback((data, err) => {})); // false
console.log(isErrorFirstCallback((result) => {})); // false
```

**Правила хорошего тона для колбэков**:

1. **Всегда обрабатывайте ошибку** в первой строке колбэка
2. **Не вызывайте колбэк больше одного раза** (используйте флаг)
3. **Не смешивайте синхронный и асинхронный вызов** одного колбэка
4. **Передавайте ошибку вверх** по цепочке, не проглатывайте её
5. **Для трёх и более вложенных колбэков** — рефакторинг в промисы

```javascript
// Пример соответствия правилам
function fetchUser(userId, callback) {
  if (typeof userId !== 'number') {
    // Синхронная ошибка — но через setImmediate для единообразия
    setImmediate(() => callback(new TypeError('userId должен быть числом')));
    return;
  }
  
  let called = false;
  database.query('SELECT * FROM users WHERE id = ?', [userId], (err, rows) => {
    if (called) return;
    called = true;
    
    if (err) return callback(err);
    if (rows.length === 0) {
      return callback(new Error('Пользователь не найден'));
    }
    
    callback(null, rows[0]);
  });
}
```



**16. Использование Promise и `async/await` в Node.js**

Promise — объект, представляющий результат асинхронной операции, который может быть выполнен (resolved) или отклонён (rejected). `async/await` — синтаксический сахар над Promise, делающий асинхронный код похожим на синхронный.

**Состояния Promise**:

```javascript
const promise = new Promise((resolve, reject) => {
  // pending (ожидание)
  
  if (/* операция успешна */) {
    resolve('результат'); // fulfilled (выполнен)
  } else {
    reject(new Error('ошибка')); // rejected (отклонён)
  }
});
```

**Создание Promise**:

```javascript
// 1. Ручное создание
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// 2. Promise.resolve() — мгновенно выполненный Promise
const resolved = Promise.resolve({ name: 'Alice' });
resolved.then(data => console.log(data.name));

// 3. Promise.reject() — мгновенно отклонённый Promise
const rejected = Promise.reject(new Error('Ошибка'));
rejected.catch(err => console.error(err.message));

// 4. util.promisify (преобразование error-first функций)
const { promisify } = require('util');
const readFileAsync = promisify(fs.readFile);
```

**Базовые методы Promise**:

```javascript
const promise = new Promise((resolve) => {
  setTimeout(() => resolve('Данные'), 1000);
});

// then — обработка успешного выполнения
promise.then((data) => {
  console.log(data); // 'Данные'
  return data.toUpperCase();
});

// catch — обработка ошибок
promise.catch((err) => {
  console.error('Ошибка:', err);
});

// finally — выполняется всегда (очистка ресурсов)
promise.finally(() => {
  console.log('Операция завершена (успех или ошибка)');
});

// Цепочки then
readFileAsync('a.txt')
  .then(data => JSON.parse(data))
  .then(obj => obj.id)
  .then(id => fetchUser(id))
  .then(user => console.log(user))
  .catch(err => console.error('Где-то в цепочке произошла ошибка'));
```

**Promise API: комбинация промисов**:

```javascript
const p1 = readFileAsync('file1.txt');
const p2 = readFileAsync('file2.txt');
const p3 = readFileAsync('file3.txt');

// Promise.all — ждёт ВСЕ или падает при первой ошибке
const allResults = await Promise.all([p1, p2, p3]);
console.log('Все файлы загружены:', allResults);

// Promise.allSettled — ждёт ВСЕ, но не падает (возвращает статус каждого)
const settled = await Promise.allSettled([p1, p2, p3]);
settled.forEach(result => {
  if (result.status === 'fulfilled') {
    console.log('Успех:', result.value);
  } else {
    console.log('Ошибка:', result.reason.message);
  }
});

// Promise.race — первый выполнившийся (успех или ошибка)
const fastResult = await Promise.race([
  fetch('/api/slow'),
  fetch('/api/fast')
]);

// Promise.any — первый УСПЕШНЫЙ (игнорирует ошибки, пока хоть один не выполнится)
const firstSuccess = await Promise.any([
  Promise.reject(new Error('fail')),
  Promise.resolve('success'),
  Promise.reject(new Error('another fail'))
]);
console.log(firstSuccess); // 'success'
```

**async/await — основной паттерн в современном Node.js**:

```javascript
// Функция async всегда возвращает Promise
async function getUserData(userId) {
  // await "разворачивает" Promise в значение
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  const posts = await db.query('SELECT * FROM posts WHERE user_id = ?', [userId]);
  
  return { user, posts }; // автоматически оборачивается в Promise.resolve()
}

// Использование
try {
  const data = await getUserData(123);
  console.log(data.user.name, data.posts.length);
} catch (err) {
  console.error('Ошибка загрузки:', err);
}
```

**Обработка ошибок в async/await**:

```javascript
// Способ 1: try/catch (рекомендуется)
async function fetchData() {
  try {
    const data = await fetch('/api/data');
    const json = await data.json();
    return json;
  } catch (err) {
    console.error('Сетевая ошибка:', err.message);
    return null; // значение по умолчанию
  }
}

// Способ 2: .catch() на результате await
async function fetchData() {
  const data = await fetch('/api/data').catch(err => {
    console.error(err);
    return null;
  });
  if (!data) return defaultData;
  return data.json();
}

// Способ 3: try/catch с повторной отправкой ошибки выше
async function controller(req, res) {
  try {
    const result = await serviceLayer.process(req.body);
    res.json(result);
  } catch (err) {
    // Логируем, но отдаём клиенту понятную ошибку
    logger.error(err);
    res.status(500).json({ error: 'Внутренняя ошибка сервера' });
  }
}
```

**Параллельное выполнение с async/await**:

```javascript
// ❌ ПОСЛЕДОВАТЕЛЬНО (медленно)
async function loadSequential() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { user, posts, comments };
  // Время: t(user) + t(posts) + t(comments)
}

// ✅ ПАРАЛЛЕЛЬНО (быстро)
async function loadParallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
  // Время: max(t(user), t(posts), t(comments))
}

// Частичный параллелизм (группировка)
async function mixedLoad() {
  const userPromise = fetchUser(); // запускаем сразу
  
  const [posts, comments] = await Promise.all([
    fetchPosts(),
    fetchComments()
  ]);
  
  const user = await userPromise; // дожидаемся, если ещё не готов
  return { user, posts, comments };
}
```

**Практический пример: HTTP запросы с async/await**:

```javascript
const https = require('https');
const { promisify } = require('util');

// Промис-обёртка для HTTP запросов
function httpsGet(url) {
  return new Promise((resolve, reject) => {
    https.get(url, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => {
        try {
          resolve(JSON.parse(data));
        } catch (err) {
          reject(err);
        }
      });
    }).on('error', reject);
  });
}

// API клиент с async/await
class GitHubAPI {
  async getUser(username) {
    const user = await httpsGet(`https://api.github.com/users/${username}`);
    return user;
  }
  
  async getRepos(username, maxRepos = 5) {
    const repos = await httpsGet(`https://api.github.com/users/${username}/repos`);
    return repos.slice(0, maxRepos).map(r => r.name);
  }
  
  async getUserWithRepos(username) {
    const [user, repos] = await Promise.all([
      this.getUser(username),
      this.getRepos(username)
    ]);
    
    return { ...user, repos };
  }
}

// Использование
const api = new GitHubAPI();
try {
  const data = await api.getUserWithRepos('nodejs');
  console.log(`${data.login}: ${data.repos.join(', ')}`);
} catch (err) {
  console.error('API ошибка:', err.message);
}
```

**Распространённые ошибки с async/await**:

```javascript
// ❌ Забытый await в параллельном выполнении
async function badExample() {
  const user = fetchUser(); // Promise, не данные
  const posts = fetchPosts(); // Promise
  
  console.log(user.id); // undefined (user — это Promise)
  return { user, posts };
}

// ✅ Правильно
async function goodExample() {
  const userPromise = fetchUser();
  const postsPromise = fetchPosts();
  
  const user = await userPromise;
  const posts = await postsPromise;
  
  return { user, posts };
}

// ❌ await в цикле (последовательно, медленно)
async function processFilesSequential(files) {
  const results = [];
  for (const file of files) {
    results.push(await readFileAsync(file)); // ждём каждый
  }
  return results;
}

// ✅ параллельно с Promise.all
async function processFilesParallel(files) {
  const promises = files.map(file => readFileAsync(file));
  return await Promise.all(promises);
}

// ❌ try/catch вокруг всего кода, но забыли обработать конкретные ошибки
async function noErrorHandling() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    return posts;
  } catch (err) {
    // Непонятно, кто упал: fetchUser или fetchPosts
    console.log('Что-то пошло не так');
  }
}

// ✅ Раздельная обработка
async function withErrorHandling() {
  try {
    const user = await fetchUser();
    try {
      const posts = await fetchPosts(user.id);
      return posts;
    } catch (err) {
      console.error('Ошибка загрузки постов:', err);
      return [];
    }
  } catch (err) {
    console.error('Ошибка загрузки пользователя:', err);
    throw err;
  }
}
```

**Promise без await (fire-and-forget)**:

```javascript
// Ситуация: отправка уведомления, которое не должно блокировать ответ
app.post('/order', async (req, res) => {
  const order = await saveOrder(req.body);
  
  // Отправляем email в фоне (не ждём)
  sendEmailNotification(order.email, 'Заказ создан')
    .catch(err => logger.error('Email не отправлен:', err));
  
  res.json({ id: order.id }); // ответ возвращается сразу
});

// Использование void оператора (TypeScript) или просто без await
void sendEmailNotification(order.email, 'Заказ создан');
```

**Преобразование колбэков в async/await (полная схема)**:

```javascript
// Исходная колбэк-функция
function legacyApi(param, callback) {
  setTimeout(() => {
    if (param === 'error') callback(new Error('Fail'));
    else callback(null, `Result: ${param}`);
  }, 100);
}

// Шаг 1: Промисфикация
function legacyApiPromise(param) {
  return new Promise((resolve, reject) => {
    legacyApi(param, (err, result) => {
      if (err) reject(err);
      else resolve(result);
    });
  });
}

// Шаг 2: Использование с async/await
async function modernUse() {
  try {
    const result = await legacyApiPromise('test');
    console.log(result);
  } catch (err) {
    console.error(err);
  }
}

// Утилита для массовой промисфикации
function promisifyAll(obj) {
  for (const key in obj) {
    if (typeof obj[key] === 'function') {
      obj[`${key}Async`] = promisify(obj[key]);
    }
  }
  return obj;
}
```

**Таймауты для Promise**:

```javascript
function timeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error(`Timeout after ${ms}ms`)), ms)
    )
  ]);
}

// Использование
try {
  const result = await timeout(fetch('/api/slow'), 1000);
  console.log(result);
} catch (err) {
  console.log('Запрос слишком долгий');
}
```

**Ретаи (повторные попытки) с Promise**:

```javascript
async function retry(fn, maxAttempts = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === maxAttempts) throw err;
      console.log(`Попытка ${attempt} не удалась, повтор через ${delay}мс`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Использование
const data = await retry(
  () => fetch('/api/unreliable'),
  5,
  500
);
```

**Золотое правило**: в новом коде всегда используйте `async/await` вместо прямых `.then()` и тем более колбэков. Исключения — редкие случаи, когда нужно динамически строить цепочки промисов или использовать методы вроде `Promise.all()` внутри цикла.



**17. Метод `util.promisify()`**

`util.promisify()` — встроенная функция Node.js, преобразующая функции с error-first колбэками в функции, возвращающие Promise. Это мост между старым (колбэки) и новым (async/await) стилями кода.

**Базовое использование**:

```javascript
const util = require('util');
const fs = require('fs');

// Преобразуем колбэк-функцию в Promise-версию
const readFileAsync = util.promisify(fs.readFile);
const writeFileAsync = util.promisify(fs.writeFile);
const statAsync = util.promisify(fs.stat);

// Теперь можно использовать async/await
async function example() {
  try {
    const data = await readFileAsync('./file.txt', 'utf8');
    await writeFileAsync('./copy.txt', data);
    const stats = await statAsync('./copy.txt');
    console.log(`Размер: ${stats.size} байт`);
  } catch (err) {
    console.error(err);
  }
}
```

**Как это работает внутри**:

```javascript
// Ручная реализация promisify (упрощённо)
function manualPromisify(originalFunction) {
  return function promisified(...args) {
    return new Promise((resolve, reject) => {
      // Добавляем колбэк в конец аргументов
      originalFunction(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

// Пример использования
const readFileAsync = manualPromisify(fs.readFile);
```

**Промисфикация методов с `this`**:

```javascript
const util = require('util');

class Database {
  constructor(connection) {
    this.conn = connection;
  }
  
  query(sql, callback) {
    // Симуляция запроса
    setTimeout(() => {
      if (sql === 'SELECT * FROM users') {
        callback(null, [{ id: 1, name: 'Alice' }]);
      } else {
        callback(new Error('Invalid SQL'));
      }
    }, 100);
  }
}

const db = new Database('connected');

// Промисфикация метода с привязкой контекста
const queryAsync = util.promisify(db.query.bind(db));

async function getUsers() {
  const users = await queryAsync('SELECT * FROM users');
  console.log(users); // [{ id: 1, name: 'Alice' }]
}

// Альтернатива: промисфикация всего класса
class AsyncDatabase extends Database {
  constructor(conn) {
    super(conn);
    this.queryAsync = util.promisify(this.query.bind(this));
  }
}
```

**Промисфикация функций с несколькими результатами**:

```javascript
const util = require('util');

// Функция, возвращающая несколько значений через колбэк
function divideWithRemainder(a, b, callback) {
  if (b === 0) {
    callback(new Error('Division by zero'));
    return;
  }
  callback(null, Math.floor(a / b), a % b);
}

// promisify по умолчанию возвращает только первый результат
const divideAsync = util.promisify(divideWithRemainder);
const result = await divideAsync(10, 3);
console.log(result); // 3 (только quotient, remainder потерян)

// Решение: кастомная промисфикация с получением всех результатов
function customPromisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, ...results) => {
        if (err) reject(err);
        else resolve(results.length === 1 ? results[0] : results);
      });
    });
  };
}

const divideFullAsync = customPromisify(divideWithRemainder);
const [quotient, remainder] = await divideFullAsync(10, 3);
console.log({ quotient, remainder }); // { quotient: 3, remainder: 1 }
```

**Использование `util.promisify.custom`**:

```javascript
const util = require('util');

// Функция, которая уже может возвращать Promise
function myAsyncFunction(arg, callback) {
  // Если callback не передан — возвращаем Promise
  if (typeof callback !== 'function') {
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(`Result: ${arg}`), 100);
    });
  }
  
  // Старый стиль с колбэком
  setTimeout(() => callback(null, `Result: ${arg}`), 100);
}

// Указываем promisify, какую функцию использовать для Promise-версии
myAsyncFunction[util.promisify.custom] = (arg) => {
  return Promise.resolve(`Custom promise result: ${arg}`);
};

const promisified = util.promisify(myAsyncFunction);
console.log(await promisified('test')); // 'Custom promise result: test'
```

**Практический пример: промисфикация API модуля `crypto`**:

```javascript
const util = require('util');
const crypto = require('crypto');

// Старый стиль с колбэками
crypto.randomBytes(32, (err, buffer) => {
  if (err) throw err;
  console.log('Random bytes:', buffer.toString('hex'));
});

// Современный стиль
const randomBytesAsync = util.promisify(crypto.randomBytes);
const pbkdf2Async = util.promisify(crypto.pbkdf2);

async function generatePasswordHash(password, salt) {
  const hash = await pbkdf2Async(password, salt, 100000, 64, 'sha256');
  return hash.toString('hex');
}

async function generateToken() {
  const buffer = await randomBytesAsync(32);
  return buffer.toString('hex');
}
```

**Промисфикация целого модуля**:

```javascript
const util = require('util');
const fs = require('fs');

// Функция для массовой промисфикации
function promisifyModule(module, exclude = ['exists']) {
  const result = {};
  
  for (const key of Object.keys(module)) {
    const original = module[key];
    
    // Пропускаем не-функции и исключённые методы
    if (typeof original !== 'function' || exclude.includes(key)) {
      result[key] = original;
      continue;
    }
    
    // Проверяем сигнатуру (последний аргумент — callback?)
    const fnStr = original.toString();
    const hasCallback = fnStr.includes('callback') || 
                       fnStr.includes('(err') ||
                       original.length === 0;
    
    if (hasCallback && key !== 'exists') { // exists — особый случай
      result[`${key}Async`] = util.promisify(original);
    }
    result[key] = original;
  }
  
  return result;
}

const fsAsync = promisifyModule(fs);
// Теперь есть fs.readFileAsync, fs.writeFileAsync и т.д.

await fsAsync.writeFileAsync('./test.txt', 'Hello');
const content = await fsAsync.readFileAsync('./test.txt', 'utf8');
```

**Особые случаи и ограничения**:

```javascript
const util = require('util');

// 1. Функции без колбэка (не промисфицируются)
function syncFunction() {
  return 'sync result';
}
const badAsync = util.promisify(syncFunction);
console.log(await badAsync()); // 'sync result' (работает, но бессмысленно)

// 2. Функции с колбэком, но не error-first
function nonStandardCallback(arg, callback) {
  callback('result'); // нет проверки ошибки
}
const promisifiedNonStandard = util.promisify(nonStandardCallback);
// ОШИБКА: callback должен быть (err, result)

// 3. Колбэк с ошибкой как вторым аргументом (не поддерживается)
function weirdCallback(arg, callback) {
  callback('result', null); // обратный порядок
}

// Решение: кастомная обёртка
function wrapWeird(fn) {
  return function(...args) {
    return new Promise((resolve) => {
      fn(...args, (result, err) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

// 4. Метод fs.exists (устаревший, не соответствует error-first)
// fs.exists не промисфицируется корректно, используйте fs.access или fs.stat
```

**Промисфикация в реальном проекте**:

```javascript
const util = require('util');
const redis = require('redis');

class RedisClient {
  constructor(config) {
    this.client = redis.createClient(config);
    
    // Промисфицируем нужные методы
    this.getAsync = util.promisify(this.client.get).bind(this.client);
    this.setAsync = util.promisify(this.client.set).bind(this.client);
    this.delAsync = util.promisify(this.client.del).bind(this.client);
    this.keysAsync = util.promisify(this.client.keys).bind(this.client);
  }
  
  async getOrSet(key, fetchFn, ttl = 3600) {
    const cached = await this.getAsync(key);
    if (cached !== null) {
      return JSON.parse(cached);
    }
    
    const data = await fetchFn();
    await this.setAsync(key, JSON.stringify(data), 'EX', ttl);
    return data;
  }
}

// Использование
const cache = new RedisClient({ host: 'localhost' });
const user = await cache.getOrSet('user:123', async () => {
  return await db.query('SELECT * FROM users WHERE id = 123');
});
```

**Сравнение подходов к промисфикации**:

| Подход | Код | Плюсы | Минусы |
|--------|-----|-------|--------|
| util.promisify | `util.promisify(fs.readFile)` | Стандарт, быстрый, нативный | Работает только с error-first |
| Ручной Promise | `new Promise((res,rej)=> fn(...))` | Полный контроль | Много шаблонного кода |
| Модуль `fs/promises` | `require('fs/promises')` | Современный, без обёрток | Только для встроенных модулей |
| Библиотека `bluebird` | `Promise.promisifyAll(fs)` | Промисфикация всех методов разом | Лишняя зависимость |

**Проверка, можно ли промисфицировать функцию**:

```javascript
function isPromisifiable(fn) {
  if (typeof fn !== 'function') return false;
  
  // Получаем количество параметров
  const params = fn.length;
  
  // Функции без параметров или с 1 параметром могут быть промисфицированы
  // (последний параметр — колбэк)
  return params <= 3; // упрощённая проверка
}

// Более точная проверка через анализ строки
function isErrorFirstCallback(fn) {
  const fnStr = fn.toString();
  const lastParam = fnStr.match(/\([^)]*\)/)?.[0]?.split(',').pop()?.trim();
  return lastParam === 'callback' || lastParam === 'cb' || lastParam === 'next';
}
```

**Рекомендации по использованию**:

1. **Для встроенных модулей** используйте нативные Promise-версии (`fs/promises`, `dns/promises`)
2. **Для сторонних библиотек** с колбэками — `util.promisify`
3. **Для своих функций** пишите сразу Promise или async/await, не создавая колбэк-версий
4. **В новых проектах** избегайте колбэков, используйте async/await с самого начала
5. **При поддержке старого кода** промисфицируйте колбэк-функции на границах модулей

```javascript
// Паттерн "адаптер" для устаревшего API
class LegacyAPIAdapter {
  constructor() {
    this.legacy = new LegacyAPI();
    
    // Промисфицируем на границе
    this.getUserAsync = util.promisify(this.legacy.getUser.bind(this.legacy));
    this.saveUserAsync = util.promisify(this.legacy.saveUser.bind(this.legacy));
  }
  
  async getUser(id) {
    return await this.getUserAsync(id);
  }
}
```



**18. Создание HTTP-сервера с модулем `http`**

Модуль `http` — встроенная основа для создания веб-серверов в Node.js. Без фреймворков (Express, Fastify) он даёт полный контроль над обработкой запросов и ответов.

**Минимальный HTTP-сервер**:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Сервер запущен на http://localhost:3000');
});
```

**Объекты request и response**:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // Request — входящий поток данных
  console.log('Метод:', req.method);        // GET, POST, PUT, DELETE
  console.log('URL:', req.url);             // /users/123?sort=asc
  console.log('HTTP версия:', req.httpVersion);
  console.log('Заголовки:', req.headers);   // { host: 'localhost', 'user-agent': '...' }
  
  // Response — исходящий поток данных
  res.statusCode = 200;
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('X-Powered-By', 'Node.js');
  
  res.end(JSON.stringify({ message: 'OK' }));
});

server.listen(3000);
```

**Разбор URL и query-параметров**:

```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  // Парсинг URL (true для преобразования query в объект)
  const parsedUrl = url.parse(req.url, true);
  
  console.log('Путь:', parsedUrl.pathname);     // '/users/profile'
  console.log('Параметры:', parsedUrl.query);   // { id: '123', sort: 'asc' }
  
  // Ручное извлечение параметров
  const { id, sort = 'desc' } = parsedUrl.query;
  
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ id, sort, path: parsedUrl.pathname }));
});

server.listen(3000);
// Запрос: http://localhost:3000/users?id=123&sort=asc
```

**Обработка разных HTTP методов и маршрутов**:

```javascript
const http = require('http');
const url = require('url');

const users = {
  1: { name: 'Alice', age: 30 },
  2: { name: 'Bob', age: 25 }
};

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const pathname = parsedUrl.pathname;
  const method = req.method;
  
  // GET /users
  if (method === 'GET' && pathname === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(users));
    return;
  }
  
  // GET /users/:id
  const match = pathname.match(/^\/users\/(\d+)$/);
  if (method === 'GET' && match) {
    const id = match[1];
    const user = users[id];
    
    if (!user) {
      res.writeHead(404, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'User not found' }));
      return;
    }
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(user));
    return;
  }
  
  // POST /users
  if (method === 'POST' && pathname === '/users') {
    let body = '';
    
    req.on('data', chunk => {
      body += chunk.toString();
    });
    
    req.on('end', () => {
      try {
        const newUser = JSON.parse(body);
        const id = Date.now();
        users[id] = newUser;
        
        res.writeHead(201, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ id, ...newUser }));
      } catch (err) {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Invalid JSON' }));
      }
    });
    return;
  }
  
  // 404 для всех остальных
  res.writeHead(404, { 'Content-Type': 'text/plain' });
  res.end('Not Found');
});

server.listen(3000);
```

**Чтение тела запроса (POST/PUT)**:

```javascript
// Утилита для парсинга тела запроса
function parseRequestBody(req) {
  return new Promise((resolve, reject) => {
    let body = '';
    
    req.on('data', chunk => {
      body += chunk.toString();
      
      // Защита от слишком больших запросов
      if (body.length > 10 * 1024 * 1024) { // 10 MB лимит
        req.destroy();
        reject(new Error('Request body too large'));
      }
    });
    
    req.on('end', () => {
      try {
        const contentType = req.headers['content-type'];
        
        if (contentType === 'application/json') {
          resolve(JSON.parse(body));
        } else if (contentType === 'application/x-www-form-urlencoded') {
          const params = new URLSearchParams(body);
          resolve(Object.fromEntries(params));
        } else {
          resolve(body);
        }
      } catch (err) {
        reject(err);
      }
    });
    
    req.on('error', reject);
  });
}

// Использование
const server = http.createServer(async (req, res) => {
  if (req.method === 'POST') {
    try {
      const data = await parseRequestBody(req);
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ received: data }));
    } catch (err) {
      res.writeHead(400);
      res.end(err.message);
    }
  }
});
```

**Работа с заголовками**:

```javascript
const server = http.createServer((req, res) => {
  // Чтение заголовков запроса
  const authHeader = req.headers.authorization;
  const userAgent = req.headers['user-agent'];
  const contentType = req.headers['content-type'];
  
  // Установка заголовков ответа
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Cache-Control', 'no-cache, no-store, must-revalidate');
  res.setHeader('Access-Control-Allow-Origin', '*'); // CORS
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  
  // Обработка preflight запросов (OPTIONS)
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    res.end();
    return;
  }
  
  res.writeHead(200);
  res.end(JSON.stringify({ status: 'ok' }));
});
```

**Потоковая передача больших данных**:

```javascript
const http = require('http');
const fs = require('fs');

// ❌ Плохо: загружает весь файл в память
server.on('request', (req, res) => {
  fs.readFile('./large-video.mp4', (err, data) => {
    res.end(data); // 500 MB файл убьёт память
  });
});

// ✅ Хорошо: потоковая передача
server.on('request', (req, res) => {
  const stream = fs.createReadStream('./large-video.mp4');
  
  // Поддержка частичных запросов (Range)
  const range = req.headers.range;
  if (range) {
    const positions = range.replace(/bytes=/, '').split('-');
    const start = parseInt(positions[0], 10);
    const end = positions[1] ? parseInt(positions[1], 10) : Infinity;
    
    res.writeHead(206, {
      'Content-Range': `bytes ${start}-${end}/${stat.size}`,
      'Content-Type': 'video/mp4'
    });
    stream.pipe(res);
  } else {
    stream.pipe(res);
  }
});

// ✅ Ещё лучше: сжатие на лету
const zlib = require('zlib');
server.on('request', (req, res) => {
  const acceptEncoding = req.headers['accept-encoding'];
  const readStream = fs.createReadStream('./data.json');
  
  if (acceptEncoding && acceptEncoding.includes('gzip')) {
    res.setHeader('Content-Encoding', 'gzip');
    readStream.pipe(zlib.createGzip()).pipe(res);
  } else {
    readStream.pipe(res);
  }
});
```

**Обработка ошибок и таймауты**:

```javascript
const server = http.createServer((req, res) => {
  // Таймаут запроса (30 секунд)
  req.setTimeout(30000, () => {
    res.writeHead(408);
    res.end('Request Timeout');
    req.destroy();
  });
  
  // Обработка ошибок запроса
  req.on('error', (err) => {
    console.error('Request error:', err);
    if (!res.headersSent) {
      res.writeHead(400);
      res.end('Bad Request');
    }
  });
  
  // Обработка ошибок ответа
  res.on('error', (err) => {
    console.error('Response error:', err);
  });
  
  // Основная логика
  try {
    // опасная операция
    throw new Error('Database error');
  } catch (err) {
    console.error(err);
    if (!res.headersSent) {
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'Internal Server Error' }));
    }
  }
});

// Глобальная обработка ошибок сервера
server.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error('Порт уже используется');
    process.exit(1);
  } else {
    console.error('Server error:', err);
  }
});
```

**Создание HTTPS-сервера**:

```javascript
const https = require('https');
const fs = require('fs');

// Загрузка сертификатов
const options = {
  key: fs.readFileSync('./private.key'),
  cert: fs.readFileSync('./certificate.crt'),
  ca: fs.readFileSync('./ca_bundle.crt'), // опционально
  passphrase: 'your-passphrase' // если сертификат защищён паролем
};

const server = https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('Secure Hello World');
});

server.listen(443, () => {
  console.log('HTTPS сервер запущен на порту 443');
});
```

**Сервер с балансировкой и keep-alive**:

```javascript
const http = require('http');
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  // Мастер-процесс: создаём воркеры по числу ядер CPU
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} умер, создаём нового`);
    cluster.fork();
  });
} else {
  // Воркер: обрабатывает запросы
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Handled by worker ${process.pid}`);
  });
  
  server.listen(3000, () => {
    console.log(`Worker ${process.pid} запущен`);
  });
  
  // Keep-alive настройки
  server.keepAliveTimeout = 65000; // 65 секунд для HTTP/1.1
  server.headersTimeout = 66000;    // чуть больше keepAliveTimeout
}
```

**Прокси-сервер на базе `http`**:

```javascript
const http = require('http');
const url = require('url');

const proxy = http.createServer((clientReq, clientRes) => {
  const targetUrl = url.parse(clientReq.url);
  targetUrl.host = 'api.example.com';
  targetUrl.protocol = 'http';
  
  const proxyReq = http.request(targetUrl, (proxyRes) => {
    // Копируем статус и заголовки
    clientRes.writeHead(proxyRes.statusCode, proxyRes.headers);
    proxyRes.pipe(clientRes);
  });
  
  proxyReq.on('error', (err) => {
    clientRes.writeHead(502);
    clientRes.end('Bad Gateway');
  });
  
  // Пересылаем тело запроса
  clientReq.pipe(proxyReq);
});

proxy.listen(8080);
console.log('Прокси запущен на http://localhost:8080');
```

**Практические советы**:

| Сценарий | Рекомендация |
|----------|--------------|
| Простой API или статика | `http` модуль достаточен |
| Сложная маршрутизация, middleware | Используйте Express, Fastify |
| WebSocket | `ws` библиотека поверх `http` |
| Высокая производительность | `http` + кластеризация |
| GraphQL | Apollo Server (на основе `http`) |

**Метрики и мониторинг**:

```javascript
const server = http.createServer((req, res) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} ${res.statusCode} - ${duration}ms`);
    
    // Отправка метрик (Prometheus, Datadog, etc.)
    recordMetric('http.request.duration', duration, {
      method: req.method,
      path: req.url,
      status: res.statusCode
    });
  });
  
  // обработка запроса...
});
```

**Запуск с graceful shutdown**:

```javascript
const server = http.createServer(app);

server.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM received, closing server...');
  
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
  
  // Принудительное закрытие через 10 секунд
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 10000);
});
```



**19. Обработка запросов и ответов (основы маршрутизации)**

Маршрутизация (routing) — механизм сопоставления URL и HTTP-метода с конкретным обработчиком. Встроенный модуль `http` не имеет встроенного роутера — его нужно реализовывать вручную или использовать фреймворк.

**Базовая маршрутизация без сторонних библиотек**:

```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const pathname = parsedUrl.pathname;
  const method = req.method;
  
  // Маршрутизация через if/else
  if (method === 'GET' && pathname === '/') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } 
  else if (method === 'GET' && pathname === '/about') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>About Us</h1>');
  }
  else if (method === 'GET' && pathname === '/api/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, name: 'Alice' }]));
  }
  else if (method === 'POST' && pathname === '/api/users') {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const user = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ id: Date.now(), ...user }));
    });
  }
  else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('404 Not Found');
  }
});

server.listen(3000);
```

**Паттерн "маршрутная таблица" (более структурированный)**:

```javascript
const http = require('http');
const url = require('url');

// Таблица маршрутов
const routes = {
  GET: {},
  POST: {},
  PUT: {},
  DELETE: {}
};

// Вспомогательные функции для регистрации
function get(path, handler) {
  routes.GET[path] = handler;
}

function post(path, handler) {
  routes.POST[path] = handler;
}

function put(path, handler) {
  routes.PUT[path] = handler;
}

function del(path, handler) {
  routes.DELETE[path] = handler;
}

// Регистрация маршрутов
get('/', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end('<h1>Home</h1>');
});

get('/api/users', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ users: [] }));
});

post('/api/users', async (req, res) => {
  const body = await parseBody(req);
  res.writeHead(201, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ created: body }));
});

// Основной обработчик
const server = http.createServer(async (req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const pathname = parsedUrl.pathname;
  const method = req.method;
  
  const handler = routes[method]?.[pathname];
  
  if (handler) {
    try {
      // Добавляем parsedUrl в req для доступа к query
      req.query = parsedUrl.query;
      await handler(req, res);
    } catch (err) {
      console.error(err);
      if (!res.headersSent) {
        res.writeHead(500, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Internal Server Error' }));
      }
    }
  } else {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
});
```

**Динамические маршруты с параметрами**:

```javascript
const http = require('http');
const url = require('url');

// Хранилище маршрутов с паттернами
const routePatterns = [];

function addRoute(method, pattern, handler) {
  routePatterns.push({ method, pattern, handler });
}

// Преобразование паттерна в регулярное выражение
function patternToRegex(pattern) {
  const paramNames = [];
  const regexPattern = pattern.replace(/:([a-zA-Z_][a-zA-Z0-9_]*)/g, (_, paramName) => {
    paramNames.push(paramName);
    return '([^/]+)';
  });
  return { regex: new RegExp(`^${regexPattern}$`), paramNames };
}

// Регистрация маршрутов с параметрами
addRoute('GET', '/users/:id', (req, res, params) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ userId: params.id, message: `User ${params.id} found` }));
});

addRoute('GET', '/posts/:postId/comments/:commentId', (req, res, params) => {
  res.end(JSON.stringify({ postId: params.postId, commentId: params.commentId }));
});

addRoute('GET', '/products/:category/:productId', (req, res, params) => {
  res.end(JSON.stringify({ category: params.category, productId: params.productId }));
});

// Матчер маршрутов
function matchRoute(method, pathname) {
  for (const route of routePatterns) {
    if (route.method !== method) continue;
    
    const { regex, paramNames } = patternToRegex(route.pattern);
    const match = pathname.match(regex);
    
    if (match) {
      const params = {};
      paramNames.forEach((name, index) => {
        params[name] = match[index + 1];
      });
      return { handler: route.handler, params };
    }
  }
  return null;
}

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const pathname = parsedUrl.pathname;
  
  const matched = matchRoute(req.method, pathname);
  
  if (matched) {
    req.query = parsedUrl.query; // query параметры тоже доступны
    matched.handler(req, res, matched.params);
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

// Примеры запросов:
// GET /users/123 → { userId: '123' }
// GET /posts/42/comments/7 → { postId: '42', commentId: '7' }
```

**Минимальный роутер с middleware поддержкой**:

```javascript
class Router {
  constructor() {
    this.middlewares = [];
    this.routes = { GET: [], POST: [], PUT: [], DELETE: [] };
  }
  
  // Регистрация middleware (выполняются перед маршрутом)
  use(middleware) {
    this.middlewares.push(middleware);
  }
  
  // Регистрация маршрутов
  add(method, path, handler) {
    this.routes[method].push({ path, handler });
  }
  
  get(path, handler) { this.add('GET', path, handler); }
  post(path, handler) { this.add('POST', path, handler); }
  put(path, handler) { this.add('PUT', path, handler); }
  delete(path, handler) { this.add('DELETE', path, handler); }
  
  // Поиск подходящего маршрута
  findRoute(method, pathname) {
    const routes = this.routes[method];
    
    for (const route of routes) {
      if (route.path === pathname) {
        return { handler: route.handler, params: {} };
      }
      
      // Поддержка параметров :id
      const routeParts = route.path.split('/');
      const pathParts = pathname.split('/');
      
      if (routeParts.length !== pathParts.length) continue;
      
      const params = {};
      let isMatch = true;
      
      for (let i = 0; i < routeParts.length; i++) {
        if (routeParts[i].startsWith(':')) {
          params[routeParts[i].slice(1)] = pathParts[i];
        } else if (routeParts[i] !== pathParts[i]) {
          isMatch = false;
          break;
        }
      }
      
      if (isMatch) return { handler: route.handler, params };
    }
    
    return null;
  }
  
  // Обработчик для http.createServer
  handler() {
    return async (req, res) => {
      const parsedUrl = url.parse(req.url, true);
      const pathname = parsedUrl.pathname;
      
      // Добавляем query к req
      req.query = parsedUrl.query;
      req.params = {};
      
      // Выполняем middleware
      let index = 0;
      const next = async () => {
        if (index < this.middlewares.length) {
          const middleware = this.middlewares[index++];
          await middleware(req, res, next);
        }
      };
      
      await next();
      
      // Если ответ уже отправлен в middleware — выходим
      if (res.headersSent) return;
      
      // Ищем маршрут
      const route = this.findRoute(req.method, pathname);
      
      if (route) {
        req.params = route.params;
        await route.handler(req, res);
      } else {
        res.writeHead(404, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Route not found' }));
      }
    };
  }
}

// Использование роутера
const router = new Router();

// Middleware
router.use(async (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

router.use(async (req, res, next) => {
  // Аутентификация (пример)
  const token = req.headers.authorization;
  if (req.url.startsWith('/api') && !token) {
    res.writeHead(401);
    res.end('Unauthorized');
    return;
  }
  next();
});

// Маршруты
router.get('/', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end('<h1>Home</h1>');
});

router.get('/api/users/:id', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ userId: req.params.id, query: req.query }));
});

router.post('/api/users', async (req, res) => {
  const body = await parseBody(req);
  res.writeHead(201, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ created: body }));
});

// Запуск сервера
const server = http.createServer(router.handler());
server.listen(3000);
```

**Обработка query-параметров и body**:

```javascript
const url = require('url');

// Утилиты для парсинга
function parseQuery(req) {
  const parsed = url.parse(req.url, true);
  req.query = parsed.query;
  req.pathname = parsed.pathname;
}

function parseBody(req) {
  return new Promise((resolve, reject) => {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const contentType = req.headers['content-type'];
      
      if (contentType === 'application/json') {
        try {
          resolve(JSON.parse(body));
        } catch (err) {
          reject(new Error('Invalid JSON'));
        }
      } else if (contentType === 'application/x-www-form-urlencoded') {
        const params = new URLSearchParams(body);
        resolve(Object.fromEntries(params));
      } else {
        resolve(body);
      }
    });
    req.on('error', reject);
  });
}

// Использование в маршруте
router.post('/api/search', async (req, res) => {
  parseQuery(req);
  const body = await parseBody(req);
  
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    query: req.query,
    body: body,
    searchTerm: body.term || req.query.q
  }));
});
```

**Разделение маршрутов по файлам (модульная структура)**:

```javascript
// routes/users.js
module.exports = (router) => {
  router.get('/users', (req, res) => {
    res.end(JSON.stringify({ users: [] }));
  });
  
  router.get('/users/:id', (req, res) => {
    res.end(JSON.stringify({ userId: req.params.id }));
  });
  
  router.post('/users', async (req, res) => {
    const body = await parseBody(req);
    res.end(JSON.stringify({ created: body }));
  });
};

// routes/products.js
module.exports = (router) => {
  router.get('/products', (req, res) => {
    res.end(JSON.stringify({ products: [] }));
  });
  
  router.get('/products/:id', (req, res) => {
    res.end(JSON.stringify({ productId: req.params.id }));
  });
};

// app.js
const router = new Router();
require('./routes/users')(router);
require('./routes/products')(router);

const server = http.createServer(router.handler());
```

**Статические файлы и маршрутизация**:

```javascript
const fs = require('fs');
const path = require('path');
const mime = require('mime-types'); // сторонняя библиотека

router.get('/static/*', (req, res) => {
  const filePath = req.params[0]; // получаем часть после /static/
  const fullPath = path.join(__dirname, 'public', filePath);
  
  fs.stat(fullPath, (err, stats) => {
    if (err || !stats.isFile()) {
      res.writeHead(404);
      res.end('File not found');
      return;
    }
    
    const mimeType = mime.lookup(fullPath) || 'application/octet-stream';
    const stream = fs.createReadStream(fullPath);
    
    res.writeHead(200, { 'Content-Type': mimeType });
    stream.pipe(res);
  });
});
```

**Редиректы и условные ответы**:

```javascript
router.get('/old-path', (req, res) => {
  // Постоянный редирект (301)
  res.writeHead(301, { 'Location': '/new-path' });
  res.end();
});

router.get('/temp-redirect', (req, res) => {
  // Временный редирект (302)
  res.writeHead(302, { 'Location': '/target' });
  res.end();
});

router.get('/conditional', (req, res) => {
  const ifNoneMatch = req.headers['if-none-match'];
  const etag = '"abc123"';
  
  if (ifNoneMatch === etag) {
    res.writeHead(304); // Not Modified
    res.end();
    return;
  }
  
  res.setHeader('ETag', etag);
  res.writeHead(200);
  res.end('New content');
});
```

**Рекомендации по организации маршрутов**:

| Размер проекта | Подход |
|----------------|--------|
| 1-5 маршрутов | if/else в одном файле |
| 5-20 маршрутов | Таблица маршрутов + параметры |
| 20+ маршрутов | Переход на Express или Fastify |
| REST API | Группировка по ресурсам (users, posts) |
| Вложенные ресурсы | `/users/:userId/posts/:postId` |

**Что запомнить**: ручная маршрутизация даёт понимание работы HTTP, но для реальных проектов используйте фреймворки (Express, Fastify, Koa), которые решают проблемы производительности, безопасности и удобства поддержки.



**20. Модули `url` и `querystring`**

Модули `url` и `querystring` — встроенные инструменты для парсинга и форматирования URL-адресов и query-параметров. В новых версиях Node.js функциональность `querystring` частично вытеснена глобальным `URL` и `URLSearchParams`, но оба подхода важны для поддержки старого кода.

**Модуль `url` (классический метод)**:

```javascript
const url = require('url');

// Разбор URL строки
const urlString = 'https://user:pass@example.com:8080/path/to/file?search=value&sort=asc#hash';
const parsed = url.parse(urlString, true); // true = парсить query в объект

console.log(parsed);
// {
//   protocol: 'https:',
//   slashes: true,
//   auth: 'user:pass',
//   host: 'example.com:8080',
//   port: '8080',
//   hostname: 'example.com',
//   hash: '#hash',
//   search: '?search=value&sort=asc',
//   query: { search: 'value', sort: 'asc' }, // благодаря true
//   pathname: '/path/to/file',
//   path: '/path/to/file?search=value&sort=asc',
//   href: 'https://user:pass@example.com:8080/path/to/file?search=value&sort=asc#hash'
// }

// Форматирование объекта в строку
const urlObject = {
  protocol: 'https',
  hostname: 'api.example.com',
  port: 443,
  pathname: '/v1/users',
  query: { limit: 10, offset: 20 }
};

const formatted = url.format(urlObject);
console.log(formatted); // 'https://api.example.com:443/v1/users?limit=10&offset=20'
```

**Модуль `querystring` (классический)**:

```javascript
const querystring = require('querystring');

// Парсинг query строки в объект
const queryString = 'name=Alice&age=30&city=New%20York&tags=js&tags=node';
const parsed = querystring.parse(queryString);

console.log(parsed);
// {
//   name: 'Alice',
//   age: '30',
//   city: 'New York',
//   tags: ['js', 'node']  // множественные значения становятся массивом
// }

// Парсинг с указанием разделителей
const custom = 'name:Alice;age:30';
const parsedCustom = querystring.parse(custom, ';', ':');
console.log(parsedCustom); // { name: 'Alice', age: '30' }

// Форматирование объекта в query строку
const obj = { name: 'Bob', age: 25, interests: ['coding', 'reading'] };
const stringified = querystring.stringify(obj);
console.log(stringified); // 'name=Bob&age=25&interests=coding&interests=reading'

// Экранирование и декодирование
const encoded = querystring.escape('Привет мир');
console.log(encoded); // '%D0%9F%D1%80%D0%B8%D0%B2%D0%B5%D1%82%20%D0%BC%D0%B8%D1%80'

const decoded = querystring.unescape(encoded);
console.log(decoded); // 'Привет мир'
```

**Современный API: `URL` и `URLSearchParams` (рекомендуется)**:

```javascript
// URL — WHATWG стандарт (работает также в браузерах)
const url = new URL('https://user:pass@example.com:8080/path/to/file?search=value&sort=asc#hash');

console.log(url.protocol);    // 'https:'
console.log(url.username);    // 'user'
console.log(url.password);    // 'pass'
console.log(url.hostname);    // 'example.com'
console.log(url.port);        // '8080'
console.log(url.pathname);    // '/path/to/file'
console.log(url.search);      // '?search=value&sort=asc'
console.log(url.hash);        // '#hash'
console.log(url.origin);      // 'https://example.com:8080'

// Чтение и изменение параметров через searchParams
console.log(url.searchParams.get('search'));  // 'value'
console.log(url.searchParams.has('sort'));    // true
console.log(url.searchParams.get('missing')); // null

// Итерация по параметрам
for (const [key, value] of url.searchParams) {
  console.log(`${key}: ${value}`);
}
// search: value
// sort: asc

// Изменение параметров
url.searchParams.set('limit', '50');
url.searchParams.delete('sort');
url.searchParams.append('tags', 'node');
url.searchParams.append('tags', 'js');

console.log(url.toString());
// 'https://user:pass@example.com:8080/path/to/file?search=value&limit=50&tags=node&tags=js#hash'
```

**URLSearchParams детально**:

```javascript
// Создание из строки
const params1 = new URLSearchParams('name=Alice&age=30');
console.log(params1.get('name')); // 'Alice'

// Создание из объекта
const params2 = new URLSearchParams({
  name: 'Bob',
  age: '25',
  tags: ['js', 'node']  // массив преобразуется в 'js,node'
});
console.log(params2.toString()); // 'name=Bob&age=25&tags=js%2Cnode'

// Создание из массива пар
const params3 = new URLSearchParams([
  ['name', 'Charlie'],
  ['age', '35'],
  ['hobby', 'coding'],
  ['hobby', 'reading']
]);
console.log(params3.getAll('hobby')); // ['coding', 'reading']

// Методы для работы
const params = new URLSearchParams('a=1&b=2&a=3');

console.log(params.has('a'));        // true
console.log(params.get('a'));        // '1' (первое значение)
console.log(params.getAll('a'));     // ['1', '3']
console.log(params.get('missing'));  // null

params.set('a', '10');               // заменяет все значения a на '10'
params.append('c', '4');             // добавляет параметр
params.delete('b');                  // удаляет все b

console.log(params.toString());      // 'a=10&c=4'

// Преобразование в объект
const asObject = Object.fromEntries(params);
console.log(asObject); // { a: '10', c: '4' } (теряет множественные значения)
```

**Сравнение подходов**:

| Операция | Классический (`url.parse`) | Современный (`URL`) |
|----------|---------------------------|--------------------|
| Парсинг строки | `url.parse(str, true)` | `new URL(str)` |
| Доступ к параметрам | `parsed.query.param` | `url.searchParams.get('param')` |
| Изменение параметров | Нужно пересобрать строку | `url.searchParams.set()` |
| Безопасность | Меньше (не проверяет протокол) | Больше (валидирует URL) |
| Браузерная совместимость | Нет | Да |

**Практические примеры работы с URL в HTTP сервере**:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // Современный способ (требует полный URL)
  const fullUrl = `http://${req.headers.host}${req.url}`;
  const urlObj = new URL(fullUrl);
  
  console.log('Путь:', urlObj.pathname);
  console.log('Параметры:');
  for (const [key, value] of urlObj.searchParams) {
    console.log(`  ${key} = ${value}`);
  }
  
  // Получение параметров
  const page = parseInt(urlObj.searchParams.get('page') || '1');
  const limit = parseInt(urlObj.searchParams.get('limit') || '10');
  const sort = urlObj.searchParams.get('sort') || 'desc';
  
  // Построение пагинации
  const response = {
    page,
    limit,
    sort,
    data: [`Item ${(page - 1) * limit + 1}`, `Item ${(page - 1) * limit + 2}`]
  };
  
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify(response));
});

server.listen(3000);
// GET /api/users?page=2&limit=5&sort=asc
```

**Разбор относительных путей**:

```javascript
const url = require('url');

// resolve — разрешает относительный путь относительно базового
const base = 'https://example.com/docs/';
const relative = '../images/logo.png';
const resolved = url.resolve(base, relative);
console.log(resolved); // 'https://example.com/images/logo.png'

// Современная альтернатива (new URL)
const resolvedModern = new URL(relative, base);
console.log(resolvedModern.href); // 'https://example.com/images/logo.png'

// Нормализация пути
const weird = 'https://example.com/./docs/../files///script.js';
const normalized = new URL(weird);
console.log(normalized.pathname); // '/files/script.js'
```

**Формирование URL для API запросов**:

```javascript
function buildApiUrl(endpoint, params = {}) {
  const url = new URL(endpoint, 'https://api.example.com');
  
  // Добавляем параметры
  Object.entries(params).forEach(([key, value]) => {
    if (Array.isArray(value)) {
      value.forEach(v => url.searchParams.append(key, v));
    } else if (value !== undefined && value !== null) {
      url.searchParams.set(key, String(value));
    }
  });
  
  // Добавляем временную метку против кэширования
  url.searchParams.set('_t', Date.now());
  
  return url.toString();
}

// Использование
const url1 = buildApiUrl('/v1/users', { limit: 10, fields: ['id', 'name'] });
console.log(url1);
// 'https://api.example.com/v1/users?limit=10&fields=id&fields=name&_t=1704067200000'

const url2 = buildApiUrl('/v1/posts', { userId: 123, sort: 'desc', includeComments: true });
console.log(url2);
```

**Парсинг строки запроса из тела POST запроса**:

```javascript
const querystring = require('querystring');

function parseFormData(req) {
  return new Promise((resolve, reject) => {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const contentType = req.headers['content-type'];
      
      if (contentType === 'application/x-www-form-urlencoded') {
        // Классический querystring
        const parsed = querystring.parse(body);
        resolve(parsed);
      } 
      else if (contentType === 'application/json') {
        resolve(JSON.parse(body));
      }
      else {
        resolve(body);
      }
    });
    req.on('error', reject);
  });
}

// Использование в маршруте
app.post('/login', async (req, res) => {
  const formData = await parseFormData(req);
  const { username, password } = formData;
  
  console.log(`Login attempt: ${username}`);
  res.end(`Welcome ${username}`);
});
```

**Экранирование и безопасность**:

```javascript
const url = require('url');
const querystring = require('querystring');

// Опасные символы в URL
const userInput = 'Alice & Bob? "great" > test';

// Классический querystring.escape
const escaped1 = querystring.escape(userInput);
console.log(escaped1); // 'Alice%20%26%20Bob%3F%20%22great%22%20%3E%20test'

// Современный URLSearchParams (рекомендуется)
const params = new URLSearchParams();
params.set('name', userInput);
console.log(params.toString()); // 'name=Alice+%26+Bob%3F+%22great%22+%3E+test'

// Декодирование
const decoded = querystring.unescape(escaped1);
console.log(decoded); // 'Alice & Bob? "great" > test'

// Защита от path traversal через decodeURIComponent
function safeJoin(base, userPath) {
  const decoded = decodeURIComponent(userPath);
  const resolved = new URL(decoded, base);
  
  if (!resolved.href.startsWith(base)) {
    throw new Error('Path traversal attempt');
  }
  return resolved.pathname;
}
```

**Извлечение параметров из сложных URL**:

```javascript
// Пользовательский роутер с поддержкой параметров пути и query
class RouterWithParams {
  constructor() {
    this.routes = [];
  }
  
  add(method, pattern, handler) {
    this.routes.push({ method, pattern, handler });
  }
  
  match(req) {
    const fullUrl = new URL(`http://${req.headers.host}${req.url}`);
    const pathname = fullUrl.pathname;
    
    for (const route of this.routes) {
      if (route.method !== req.method) continue;
      
      // Разбиваем паттерн на сегменты
      const patternSegments = route.pattern.split('/');
      const pathSegments = pathname.split('/');
      
      if (patternSegments.length !== pathSegments.length) continue;
      
      const params = {};
      let isMatch = true;
      
      for (let i = 0; i < patternSegments.length; i++) {
        if (patternSegments[i].startsWith(':')) {
          // Параметр :id
          const paramName = patternSegments[i].slice(1);
          params[paramName] = decodeURIComponent(pathSegments[i]);
        } 
        else if (patternSegments[i] !== pathSegments[i]) {
          isMatch = false;
          break;
        }
      }
      
      if (isMatch) {
        return {
          handler: route.handler,
          params,
          query: Object.fromEntries(fullUrl.searchParams)
        };
      }
    }
    
    return null;
  }
}

// Использование
const router = new RouterWithParams();
router.add('GET', '/users/:userId/posts/:postId', (req, res, context) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    params: context.params,   // { userId: '123', postId: '456' }
    query: context.query       // { includeComments: 'true', ... }
  }));
});
```

**Практические рекомендации**:

| Сценарий | Что использовать |
|----------|-----------------|
| Новый проект | `URL` и `URLSearchParams` |
| Поддержка старого кода | `url.parse()` и `querystring` |
| Парсинг query в HTTP сервере | `new URL(req.url, 'http://base')` |
| Формирование query строки | `new URLSearchParams(obj).toString()` |
| Разрешение относительных путей | `new URL(relative, base)` |
| Декодирование URL-encoded строк | `decodeURIComponent()` |

**Важное примечание**: `querystring` более толерантен к некорректным входным данным, но `URLSearchParams` строже следует стандарту WHATWG. В долгосрочной перспективе — используйте `URLSearchParams`.



**21. Введение в NPM: установка, обновление и удаление пакетов**

NPM (Node Package Manager) — стандартный менеджер пакетов для Node.js. Поставляется вместе с Node.js. Управляет зависимостями, запускает скрипты, публикует пакеты. Работает через файл `package.json` в корне проекта.

**Инициализация проекта**:

```bash
# Интерактивная инициализация (задаст вопросы)
npm init

# Быстрая инициализация со значениями по умолчанию
npm init -y
npm init --yes

# Полная ручная настройка (создайте package.json вручную)
```

**Структура `package.json` (минимальная)**:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "Пример приложения",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  },
  "keywords": ["example", "tutorial"],
  "author": "Your Name",
  "license": "MIT"
}
```

**Установка пакетов**:

```bash
# Установка в production зависимости (сохраняется в dependencies)
npm install express
npm i express                    # сокращение
npm add express                  # альтернативный синтаксис

# Установка конкретной версии
npm install express@4.17.1
npm install express@^4.18.0
npm install express@~4.17.0

# Установка в dev-зависимости (только для разработки)
npm install --save-dev nodemon
npm install -D nodemon           # сокращение
npm install --save-dev jest@29.0.0

# Установка глобально (для утилит командной строки)
npm install -g nodemon
npm install --global pm2

# Установка из package.json (все зависимости)
npm install
npm i                            # сокращение

# Установка только production зависимостей (без devDependencies)
npm install --production
npm install --omit=dev

# Установка пакета из GitHub
npm install git+https://github.com/expressjs/express.git
npm install expressjs/express#4.18.0

# Установка из локального пути
npm install ../local-package
npm install ./my-package-1.0.0.tgz
```

**Обновление пакетов**:

```bash
# Проверка устаревших пакетов
npm outdated

# Пример вывода:
# Package    Current  Wanted  Latest  Location
# express    4.17.1   4.18.2  4.18.2  my-app
# nodemon    2.0.15   2.0.22  3.0.0   my-app

# Обновление конкретного пакета до версии по правилам semver (Wanted)
npm update express
npm up express                    # сокращение

# Обновление до последней версии (Latest)
npm install express@latest
npm i express@latest

# Обновление всех пакетов до Wanted версий
npm update

# Обновление глобального пакета
npm update -g nodemon
npm install -g nodemon@latest

# Полное обновление с игнорирование semver (опасно)
npm install express@*
```

**Удаление пакетов**:

```bash
# Удаление пакета (из dependencies/devDependencies)
npm uninstall express
npm un express                    # сокращение
npm remove express
npm rm express

# Удаление с сохранением в package.json (удаляет запись)
npm uninstall --save express
npm uninstall -S express

# Удаление dev-зависимости
npm uninstall --save-dev nodemon
npm uninstall -D nodemon

# Удаление глобального пакета
npm uninstall -g nodemon

# Очистка node_modules и переустановка (решение проблем)
rm -rf node_modules package-lock.json
npm install

# Удаление только из node_modules без изменения package.json
npm uninstall --no-save express
```

**Типы зависимостей**:

```json
{
  "dependencies": {
    "express": "^4.18.0"      // Основные зависимости (production)
  },
  "devDependencies": {
    "nodemon": "^3.0.0"       // Только для разработки
  },
  "peerDependencies": {
    "react": "^18.0.0"        // Ожидается от родительского проекта
  },
  "optionalDependencies": {
    "fsevents": "^2.3.0"      // Опционально (не критично)
  },
  "bundledDependencies": [     // Включаются в сборку пакета
    "my-helper-lib"
  ]
}
```

**Семантическое версионирование (semver)**:

```bash
# Формат: major.minor.patch (breaking.feature.fix)

^1.2.3   # Любая совместимая версия (>=1.2.3 <2.0.0)
~1.2.3   # Примерно эквивалентная (>=1.2.3 <1.3.0)
1.2.x    # Любой patch (1.2.0, 1.2.1, 1.2.999)
*        # Любая версия (опасно)
1.2.3    # Точная версия
>=1.2.3  # Минимальная версия
<2.0.0   # Максимальная версия
1.2.3 - 2.1.0  # Диапазон

# Установка с точным соответствием (без каретки/тильды)
npm install --save-exact express
npm install -E express

# Сохранение диапазона с префиксом
npm config set save-prefix '~'
```

**Просмотр информации о пакетах**:

```bash
# Информация об установленном пакете
npm list
npm ls                          # сокращение
npm ls --depth=0                # только верхний уровень
npm ls express                  # информация о конкретном пакете
npm ls --prod                   # только production зависимости
npm ls --dev                    # только dev зависимости

# Информация о пакете в реестре (без установки)
npm view express
npm v express                   # сокращение
npm view express version        # конкретное поле
npm view express dependencies   # зависимости пакета
npm view express versions       # все доступные версии

# Поиск пакетов
npm search express
npm search "web framework"

# Статистика пакета
npm star express                # добавить в избранное
npm stars                       # показать избранные
```

**NPM скрипты (автоматизация)**:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "build": "webpack --mode production",
    "lint": "eslint .",
    "format": "prettier --write .",
    "precommit": "npm run lint && npm run test",
    "postinstall": "node scripts/setup.js",
    "custom": "echo 'Hello' && node script.js"
  }
}
```

```bash
# Запуск скриптов
npm start                       # специальный скрипт (можно без run)
npm run dev
npm run test
npm run build

# Встроенные хуки (pre/post)
npm run precommit               # выполнится перед commit
npm run postinstall             # выполнится после установки

# Передача аргументов
npm run test -- --coverage      # -- отделяет аргументы для скрипта
npm run build -- --watch

# Запуск нескольких скриптов
npm run lint & npm run test     # параллельно (Unix)
npm run lint && npm run test    # последовательно
```

**NPM конфигурация**:

```bash
# Просмотр настроек
npm config list
npm config get registry         # текущий реестр
npm config get prefix           # глобальный путь установки

# Установка настроек
npm config set init-author-name "Your Name"
npm config set init-license "MIT"
npm config set save-prefix "~"

# Установка реестра (например, для частного npm)
npm config set registry https://registry.npmjs.org/
npm config set @my-scope:registry https://private-registry.com/

# Редактирование конфигурации в редакторе
npm config edit

# Переменные окружения для npm
npm_config_registry=https://custom-registry.com/ npm install
```

**Управление кэшем**:

```bash
# Проверка размера кэша
npm cache verify

# Очистка кэша (при проблемах с установкой)
npm cache clean --force
npm cache clean -f              # сокращение

# Просмотр кэшированных пакетов
npm cache ls
```

**Аудит безопасности**:

```bash
# Проверка уязвимостей в зависимостях
npm audit

# Автоматическое исправление уязвимостей
npm audit fix
npm audit fix --force           # агрессивное исправление (может сломать)

# Детальный отчёт
npm audit --json
```

**Работа с локальными пакетами (npm link)**:

```bash
# В директории разрабатываемого пакета
cd ~/my-library
npm link                        # создаёт глобальную ссылку

# В проекте, использующем пакет
cd ~/my-project
npm link my-library             # создаёт символическую ссылку

# Удаление ссылки
npm unlink my-library           # удалить из проекта
cd ~/my-library && npm unlink   # удалить глобальную ссылку
```

**Публикация собственного пакета**:

```bash
# Подготовка
npm init --yes
# Добавьте необходимые поля в package.json

# Вход в аккаунт (один раз)
npm adduser
npm login                       # современная команда

# Проверка перед публикацией
npm pack                        # создаёт .tgz файл (проверить содержимое)

# Публикация
npm publish

# Публикация с областью видимости (scoped package)
npm publish --access public
npm publish --access restricted

# Обновление версии и публикация
npm version patch               # 1.0.0 -> 1.0.1
npm version minor               # 1.0.0 -> 1.1.0
npm version major               # 1.0.0 -> 2.0.0
npm publish

# Удаление опубликованной версии (в течение 72 часов)
npm unpublish my-package@1.0.0
```

**package-lock.json (фиксация зависимостей)**:

```bash
# Создаётся автоматически при npm install
# Фиксирует точные версии и дерево зависимостей

# Обновление lock файла (при изменении package.json)
npm install

# Игнорирование lock файла (не рекомендуется)
npm install --no-package-lock

# Создание lock файла без установки
npm install --package-lock-only
```

**Практические советы**:

| Ситуация | Команда |
|----------|---------|
| Новый проект | `npm init -y` |
| Добавить пакет | `npm i express` |
| Добавить dev-пакет | `npm i -D nodemon` |
| Обновить всё | `npm update` |
| Удалить пакет | `npm un express` |
| Проверить уязвимости | `npm audit` |
| Очистить node_modules | `rm -rf node_modules && npm i` |

**Типичные ошибки и решения**:

```bash
# Ошибка: EACCES (недостаточно прав)
# Решение: не используйте sudo, настройте права или используйте nvm

# Ошибка: version not found
npm view express versions --json  # посмотреть доступные версии

# Ошибка: package.json не найден
npm init -y                       # создать package.json

# Ошибка: ENOENT при установке
rm -rf node_modules package-lock.json && npm cache clean --force && npm install

# Ошибка: peer dependency conflict (React, Webpack)
npm install --legacy-peer-deps    # игнорировать конфликты peer
npm install --force               # принудительная установка
```

**Альтернативы NPM**:
- **Yarn** — более быстрый, имеет lock файл другого формата
- **pnpm** — экономит дисковое пространство через жёсткие ссылки
- **Bun** — новый менеджер пакетов (очень быстрый)

Для большинства проектов стандартного NPM достаточно. Переходить на альтернативы имеет смысл при проблемах с производительностью или дисковым пространством в монорепозиториях.



**22. `package.json` и `package-lock.json`: объяснение и различия**

`package.json` — манифест проекта, декларация зависимостей с диапазонами версий. `package-lock.json` — автоматически генерируемый файл, фиксирующий точное дерево зависимостей для воспроизводимой установки.

**package.json: основные поля**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "Описание проекта",
  "main": "index.js",
  "type": "commonjs",
  "private": true,
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "~16.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0",
    "jest": "29.0.0"
  },
  "peerDependencies": {
    "react": "^18.0.0"
  },
  "optionalDependencies": {
    "fsevents": "^2.3.2"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "os": ["linux", "darwin"],
  "cpu": ["x64", "arm64"],
  "keywords": ["example", "tutorial"],
  "author": "Your Name <email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/user/repo.git"
  },
  "bugs": {
    "url": "https://github.com/user/repo/issues"
  },
  "homepage": "https://github.com/user/repo#readme"
}
```

**Диапазоны версий в package.json (semver)**:

| Синтаксис | Значение | Пример: 1.2.3 |
|-----------|----------|---------------|
| `1.2.3` | Точная версия | только 1.2.3 |
| `^1.2.3` | Совместимая major | >=1.2.3 <2.0.0 |
| `~1.2.3` | Примерно эквивалентная | >=1.2.3 <1.3.0 |
| `1.2.x` | Любой patch | 1.2.0, 1.2.1, ... |
| `*` | Любая версия | опасно, не используйте |
| `>=1.2.3` | Минимальная версия | 1.2.3 или выше |
| `<2.0.0` | Максимальная версия | меньше 2.0.0 |
| `1.2.3 - 2.1.0` | Диапазон | между версиями |
| `1.2.3 || 2.0.0` | ИЛИ | одна из указанных |

**package-lock.json: структура и назначение**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "node_modules/express": {
      "version": "4.18.2",
      "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
      "integrity": "sha512-...",
      "dependencies": {
        "accepts": "~1.3.8",
        "body-parser": "1.20.1"
      },
      "engines": {
        "node": ">= 0.10.0"
      }
    },
    "node_modules/accepts": {
      "version": "1.3.8",
      "resolved": "https://registry.npmjs.org/accepts/-/accepts-1.3.8.tgz",
      "integrity": "sha512-...",
      "dependencies": {
        "mime-types": "~2.1.34",
        "negotiator": "0.6.3"
      }
    }
  }
}
```

**Ключевые различия**:

| Характеристика | package.json | package-lock.json |
|----------------|--------------|-------------------|
| **Назначение** | Декларация зависимостей (диапазоны) | Фиксация точных версий |
| **Редактирование** | Вручную или через `npm install --save` | Автоматически, не редактировать вручную |
| **Семантика** | "Нужен express версии ^4.18.0" | "express установлен версии 4.18.2" |
| **Вложенные зависимости** | Не фиксируются | Фиксируются все (включая зависимости зависимостей) |
| **Версионирование в Git** | Всегда коммитить | Обязательно коммитить |
| **Воспроизводимость** | Нет (разные машины могут установить разные patch-версии) | Да (точное дерево) |

**Почему package-lock.json критически важен**:

```bash
# Без package-lock.json (только package.json)
# Разработчик А устанавливает express@^4.18.0 -> получает 4.18.2
# Разработчик Б устанавливает через месяц -> получает 4.19.0 (minor)
# Возможны несовместимости

# С package-lock.json
# Оба разработчика получают абсолютно одинаковое дерево версий
# CI/CD сборка будет идентична локальной
```

**Как работает разрешение версий**:

```bash
# Исходный package.json
{
  "dependencies": {
    "express": "^4.18.0",
    "lodash": "~4.17.0"
  }
}

# При npm install:
# 1. Находим последнюю версию, удовлетворяющую ^4.18.0 (например, 4.18.2)
# 2. Находим последнюю версию, удовлетворяющую ~4.17.0 (например, 4.17.21)
# 3. Устанавливаем их
# 4. Записываем точные версии в package-lock.json
```

**Практические сценарии**:

```bash
# Сценарий 1: Новая установка проекта
git clone my-project
cd my-project
npm install   # использует package-lock.json для точных версий

# Сценарий 2: Обновление зависимостей
npm update express   # обновляет express в пределах диапазона из package.json
                     # автоматически обновляет package-lock.json

# Сценарий 3: Установка новой версии вне диапазона
npm install express@5.0.0-beta.1
# package.json обновляется: "^5.0.0-beta.1"
# package-lock.json обновляется: фиксируется бета-версия

# Сценарий 4: Удаление node_modules и переустановка
rm -rf node_modules
npm install  # установит версии из package-lock.json (если он есть)
             # если lock файла нет — разрешит заново
```

**Игнорирование package-lock.json (когда можно)**:

```bash
# Для библиотек (npm-пакетов)
# Обычно package-lock.json исключают через .gitignore
# Потребители будут разрешать зависимости по своему

# Для приложений (веб-серверы, CLI-утилиты)
# package-lock.json ОБЯЗАТЕЛЬНО коммитить в Git

# Пример .gitignore для библиотеки
node_modules/
package-lock.json  # исключаем

# Пример .gitignore для приложения
node_modules/
# package-lock.json НЕ исключаем
```

**Миграция между версиями lock-файла**:

```bash
# lockfileVersion: 1 (npm v5-v6)
# lockfileVersion: 2 (npm v7)
# lockfileVersion: 3 (npm v8+)

# Автоматическое обновление формата
npm install --package-lock-only  # обновит до текущей версии npm

# Принудительное создание lock-файла заново
rm package-lock.json
npm install  # создаст новый lock-файл с текущими диапазонами
```

**Проверка различий между package.json и package-lock.json**:

```bash
# Проверка синхронизации
npm install --dry-run  # покажет, что будет установлено без реальной установки

# Проверка несоответствий
npm ls --depth=0       # показывает установленные версии
npm outdated           # показывает устаревшие пакеты

# Сравнение требуемого и установленного
npm ls express
# my-app@1.0.0
# └── express@4.18.2 (в установке)
# В package.json: "^4.18.0" (требование)
```

**Обработка конфликтов package-lock.json в Git**:

```bash
# При merge конфликтах в package-lock.json
# 1. Принимаем изменения из ветки
git checkout --ours package-lock.json
# 2. Переустанавливаем зависимости
npm install
# 3. Фиксируем обновлённый lock-файл
git add package-lock.json
git commit --amend --no-edit

# Или стратегия "accept theirs"
git checkout --theirs package-lock.json
npm install
git add package-lock.json
```

**CI/CD рекомендации**:

```yaml
# GitHub Actions пример
- name: Install dependencies
  run: npm ci  # использует package-lock.json для точной установки
               # быстрее, строже, не обновляет lock-файл

# Вместо npm install в CI используйте npm ci:
# - Проверяет, что package-lock.json существует
# - Удаляет node_modules перед установкой
# - Устанавливает точно по lock-файлу
# - Завершается ошибкой, если lock-файл не синхронизирован
```

**Продвинутые настройки package.json**:

```json
{
  "files": [
    "dist/",
    "lib/",
    "!dist/tests/"
  ],
  "bin": {
    "my-cli": "./bin/cli.js"
  },
  "man": "./man/doc.1",
  "directories": {
    "lib": "src/",
    "test": "tests/"
  },
  "config": {
    "port": "3000"
  },
  "workspaces": [
    "packages/*"
  ],
  "overrides": {
    "lodash": "4.17.21"  // принудительная версия для всех вложенных зависимостей
  }
}
```

**npm overrides (форсирование версий)**:

```json
{
  "overrides": {
    "lodash": "4.17.21",
    "express": {
      "cookie-parser": "^1.4.6"  // переопределить зависимость зависимости
    },
    "**/glob": "7.2.3"  // переопределить во всех уровнях дерева
  }
}
```

**Типичные ошибки и решения**:

```bash
# Ошибка: package-lock.json не синхронизирован
npm install  # обновит lock-файл

# Ошибка: lockfileVersion не поддерживается
npm install -g npm@latest  # обновить npm
rm package-lock.json && npm install

# Ошибка: различия между package.json и lock-файлом
npm install --package-lock-only  # синхронизировать lock с package.json

# Предупреждение: deprecated package
npm audit        # проверить альтернативы
npm update       # может обновить до не-deprecated версии
```

**Золотое правило**: 

- **Приложения** (серверы, CLI, веб-сайты): коммитите `package-lock.json` в Git, используйте `npm ci` в CI/CD.
- **Библиотеки** (npm-пакеты): исключите `package-lock.json` из Git, оставляйте только `package.json` с диапазонами.
- **Никогда не редактируйте** `package-lock.json` вручную.
- **Всегда коммитите** `package-lock.json` в команде — это страховка от рассинхронизации версий у разных разработчиков.



**23. Запуск скриптов с `npm run`**

`npm run` — команда для выполнения пользовательских скриптов, определённых в поле `scripts` файла `package.json`. Это основной способ автоматизации задач в Node.js-проектах: запуск сервера, сборка, тестирование, линтинг, миграции БД.

**Базовое использование**:

```json
{
  "name": "my-app",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "lint": "eslint .",
    "build": "webpack --mode production"
  }
}
```

```bash
# Запуск скрипта (обязательно через npm run, кроме start/test)
npm run dev
npm run test
npm run build

# Специальные скрипты (можно без run)
npm start        # эквивалентно npm run start
npm test         # эквивалентно npm run test
npm stop         # эквивалентно npm run stop
npm restart      # выполняет stop, start (если определены)

# Для всех остальных скриптов нужен run
npm run lint
```

**Встроенные хуки: pre и post**:

```json
{
  "scripts": {
    "prebuild": "npm run lint && npm test",
    "build": "webpack --mode production",
    "postbuild": "npm run compress-assets",
    
    "precommit": "npm run lint",
    "commit": "git commit -m",
    "postcommit": "echo 'Commit done'",
    
    "preserve": "npm run build",
    "serve": "node dist/server.js"
  }
}
```

```bash
npm run build
# Порядок выполнения:
# 1. prebuild
# 2. build
# 3. postbuild
```

**Передача аргументов в скрипты**:

```bash
# Передача аргументов (разделитель --)
npm run test -- --coverage
npm run test -- --watch --verbose

# В package.json
{
  "scripts": {
    "test": "jest"
  }
}
# Фактически выполнится: jest --coverage

# Передача аргументов в позицию внутри скрипта
{
  "scripts": {
    "build:env": "webpack --mode $NODE_ENV"
  }
}
NODE_ENV=production npm run build:env

# Использование npm_config_ переменных
{
  "scripts": {
    "serve": "node server.js --port=$npm_config_port"
  }
}
npm run serve --port=3000
```

**Использование переменных окружения**:

```json
{
  "scripts": {
    "dev": "NODE_ENV=development nodemon server.js",
    "prod": "NODE_ENV=production node server.js",
    "cross": "cross-env NODE_ENV=production node server.js"
  }
}
```

```bash
# Unix (Linux/macOS)
npm run dev

# Windows (прямая установка переменных не работает)
# Решение 1: использовать cross-env пакет
npm install --save-dev cross-env

# package.json
{
  "scripts": {
    "dev:cross": "cross-env NODE_ENV=development nodemon server.js"
  }
}
```

**Ссылки на другие скрипты**:

```json
{
  "scripts": {
    "clean": "rm -rf dist/",
    "build:css": "sass src/styles:dist/css",
    "build:js": "webpack",
    "build": "npm run clean && npm run build:css && npm run build:js",
    "dev": "npm run build:css -- --watch & npm run build:js -- --watch",
    "all": "npm run lint && npm run test && npm run build"
  }
}
```

**Использование пакетов из node_modules/.bin**:

```bash
# npm автоматически добавляет node_modules/.bin в PATH при запуске скриптов

# Без npm run (пришлось бы указывать полный путь)
./node_modules/.bin/eslint .

# С npm run (достаточно имени)
npm run lint

# В package.json
{
  "scripts": {
    "lint": "eslint .",           # eslint найден в node_modules/.bin
    "prettier": "prettier --write .",
    "webpack": "webpack --config webpack.config.js"
  }
}
```

**Группировка и параллельное выполнение**:

```json
{
  "scripts": {
    "dev:server": "nodemon server.js",
    "dev:client": "vite",
    "dev:css": "sass --watch src:dist",
    
    "dev:parallel": "npm run dev:server & npm run dev:client & npm run dev:css",
    "dev:concurrently": "concurrently \"npm run dev:server\" \"npm run dev:client\" \"npm run dev:css\""
  }
}
```

```bash
# Параллельно через & (Unix) или start (Windows)
npm run dev:server & npm run dev:client

# Кроссплатформенное решение: concurrently
npm install --save-dev concurrently

# В package.json
{
  "scripts": {
    "dev": "concurrently \"npm:dev:server\" \"npm:dev:client\" \"npm:dev:css\""
  }
}
```

**Скрипты с условиями**:

```json
{
  "scripts": {
    "dev": "if-env NODE_ENV=development && npm run dev:start || npm run prod:start",
    "dev:start": "nodemon server.js",
    "prod:start": "node server.js",
    
    "postinstall": "node -e \"console.log('Installed dependencies')\"",
    "prepare": "husky install",
    
    "version": "npm run build && git add -A dist",
    "postversion": "git push && git push --tags"
  }
}
```

**Просмотр всех доступных скриптов**:

```bash
# Список всех скриптов
npm run

# С подробным описанием (если есть в package.json)
npm run --loglevel silent

# Просмотр содержимого скрипта
npm run config:list

# Через package.json
{
  "scripts": {
    "help": "echo 'Available scripts: start, dev, test, build, lint'"
  }
}
```

**Скрипты для работы с Git**:

```json
{
  "scripts": {
    "precommit": "lint-staged",
    "commit": "git-cz",
    "prepush": "npm test",
    "preversion": "npm test",
    "version": "npm run build && git add -A dist",
    "postversion": "git push && git push --tags"
  }
}
```

**Скрипты для Docker**:

```json
{
  "scripts": {
    "docker:build": "docker build -t my-app .",
    "docker:run": "docker run -p 3000:3000 my-app",
    "docker:push": "docker push my-app:latest",
    "deploy": "npm run docker:build && npm run docker:push && kubectl rollout restart deployment/my-app"
  }
}
```

**Скрипты для работы с базами данных**:

```json
{
  "scripts": {
    "db:migrate": "node-pg-migrate up",
    "db:migrate:down": "node-pg-migrate down",
    "db:seed": "node scripts/seed.js",
    "db:reset": "npm run db:migrate:down && npm run db:migrate && npm run db:seed",
    "db:studio": "prisma studio"
  }
}
```

**Переменные окружения npm (доступны в скриптах)**:

```bash
# npm предоставляет переменные окружения
npm_package_name          # my-app
npm_package_version       # 1.0.0
npm_package_config_port   # 3000 (из поля config)
npm_config_foo            # значения переданные через --foo
npm_lifecycle_event       # имя текущего скрипта (build, test)

# Использование в скриптах
{
  "config": {
    "port": "3000"
  },
  "scripts": {
    "start": "node server.js --port=$npm_package_config_port",
    "echo": "echo $npm_lifecycle_event",
    "env": "node -e \"console.log(process.env.npm_package_name)\""
  }
}
```

**Продвинутые примеры скриптов**:

```json
{
  "scripts": {
    "new:component": "node scripts/generate-component.js",
    "new:module": "node scripts/generate-module.js",
    
    "audit:fix": "npm audit fix",
    "outdated": "npm outdated --long",
    
    "clean": "rm -rf node_modules dist coverage",
    "clean:install": "npm run clean && npm install",
    
    "rebuild": "npm run clean && npm install && npm run build",
    
    "stats": "webpack --profile --json > stats.json",
    
    "deploy:prod": "npm run test && npm run build && pm2 restart ecosystem.config.js",
    
    "logs": "pm2 logs my-app",
    "status": "pm2 status"
  }
}
```

**Кастомные хуки жизненного цикла**:

```json
{
  "scripts": {
    "install": "node scripts/postinstall.js",
    "uninstall": "node scripts/cleanup.js",
    "prepublishOnly": "npm run test && npm run build",
    "postpublish": "npm run deploy:docs"
  }
}
```

**npm-run-all (мощный инструмент для нескольких скриптов)**:

```bash
npm install --save-dev npm-run-all
```

```json
{
  "scripts": {
    "lint": "eslint src",
    "test": "jest",
    "build": "webpack",
    
    "check": "npm-run-all --parallel lint test",
    "build:all": "npm-run-all --serial clean build lint test",
    "dev": "npm-run-all --parallel watch:css watch:js --print-label"
  }
}
```

**Безопасное экранирование в скриптах**:

```json
{
  "scripts": {
    "dangerous": "echo \"User input: $1\"",
    "safe": "node -e \"console.log(process.argv[1])\" --"
  }
}
```

**Отладка скриптов**:

```bash
# Просмотр, какая команда реально выполняется
npm run dev --loglevel verbose

# Выполнить без вывода в stdout
npm run build --silent

# Игнорировать ошибки (не рекомендуется)
npm run test || true

# Выполнить с трейсом
npm run test --trace-warnings

# Использование nodemon для перезапуска при изменении скриптов
{
  "scripts": {
    "dev": "nodemon --exec 'npm run build && node dist/server.js'"
  }
}
```

**Типичные паттерны**:

| Сценарий | Скрипт |
|----------|--------|
| Локальная разработка | `"dev": "nodemon server.js"` |
| Production запуск | `"start": "node server.js"` |
| Тесты один раз | `"test": "jest"` |
| Тесты в watch-режиме | `"test:watch": "jest --watch"` |
| Сборка проекта | `"build": "webpack --mode production"` |
| Линтинг | `"lint": "eslint --fix ."` |
| Форматирование | `"format": "prettier --write ."` |
| Проверка типов | `"typecheck": "tsc --noEmit"` |
| Подготовка к коммиту | `"precommit": "lint-staged"` |

**Ошибки и их решения**:

```bash
# Ошибка: команда не найдена
# Причина: пакет не установлен
npm install --save-dev eslint

# Ошибка: неверный синтаксис для Windows
# Решение: использовать cross-env или npm-run-all

# Ошибка: слишком длинная команда
# Решение: разбить на несколько скриптов

# Ошибка: скрипт не завершается по Ctrl+C
# Решение: добавить обработку сигналов в коде
process.on('SIGINT', () => { process.exit(0); });
```

**Золотые правила**:

1. Всегда используйте `npm run` для автоматизации — даже для простых команд
2. Документируйте скрипты в `README.md`, если их назначение не очевидно
3. Используйте `pre`/`post` хуки для связанных операций
4. Для кросс-платформенности используйте `cross-env` и `npm-run-all`
5. В CI всегда используйте `npm ci` вместо `npm install`, а скрипты вызывайте через `npm run`
6. Не вкладывайте сложную логику в скрипты — выносите в отдельные `.js` файлы, которые вызываются из скриптов.



**24. Работа с переменными окружения (`process.env` и `dotenv`)**

Переменные окружения — механизм передачи конфигурации в приложение без жёсткой привязки к коду. В Node.js доступны через глобальный объект `process.env`. Пакет `dotenv` загружает переменные из файла `.env` в `process.env`.

**Базовое использование `process.env`**:

```javascript
// Чтение переменных окружения
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const nodeEnv = process.env.NODE_ENV || 'development';

console.log(`Сервер запущен на порту ${port}`);
console.log(`Режим: ${nodeEnv}`);

// Установка переменной (только для текущего процесса)
process.env.MY_VAR = 'значение';
console.log(process.env.MY_VAR);

// Проверка существования
if (process.env.DEBUG === 'true') {
  console.log('Режим отладки включён');
}

// Все переменные окружения
console.log(process.env);
```

**Запуск с переменными окружения**:

```bash
# Unix (Linux/macOS)
PORT=5000 NODE_ENV=production node app.js
DEBUG=true node app.js

# Несколько переменных
DB_HOST=localhost DB_PORT=5432 DB_NAME=mydb node app.js

# Windows (CMD)
set PORT=5000 && node app.js
set NODE_ENV=production && node app.js

# Windows (PowerShell)
$env:PORT=5000; node app.js
$env:NODE_ENV="production"; node app.js

# Кроссплатформенный способ с cross-env
npx cross-env PORT=5000 NODE_ENV=production node app.js
```

**dotenv: загрузка из `.env` файла**:

```bash
npm install dotenv
```

```javascript
// app.js — в самом начале файла
require('dotenv').config();

// Теперь переменные из .env доступны
console.log(process.env.DB_HOST);
console.log(process.env.API_KEY);

// С указанием пути к файлу
require('dotenv').config({ path: './config/.env.production' });
```

**Файл `.env` (не коммитить в Git\!)**:

```bash
# .env
# Базовая конфигурация
PORT=3000
NODE_ENV=development

# База данных
DB_HOST=localhost
DB_PORT=5432
DB_USER=admin
DB_PASSWORD=supersecret
DB_NAME=mydb

# API ключи
API_KEY=sk_live_abc123def456
SECRET_KEY=my-secret-key-2024

# Булевы значения (как строки)
DEBUG=true
ENABLE_CACHE=false

# Числа (как строки)
MAX_CONNECTIONS=100
TIMEOUT_MS=5000

# Массивы (JSON формат)
ALLOWED_ORIGINS=["http://localhost:3000","https://example.com"]

# Комментарии
# DATABASE_URL=postgresql://user:pass@localhost:5432/db  # закомментировано
```

**Продвинутые настройки dotenv**:

```javascript
const dotenv = require('dotenv');
const path = require('path');

// Базовый вариант
dotenv.config();

// С кастомным путём
dotenv.config({ path: './config/.env' });

// Переопределение существующих переменных
dotenv.config({ override: true });

// Без изменения process.env (получить объект)
const config = dotenv.config({ processEnv: false });
console.log(config.parsed); // { PORT: '3000', ... }

// Загрузка из нескольких файлов
dotenv.config({ path: '.env.default' });
dotenv.config({ path: '.env.local', override: true });

// С декодированием
dotenv.config({ encoding: 'latin1' });

// Расширенный пример с окружением
const envFile = `.env.${process.env.NODE_ENV || 'development'}`;
dotenv.config({ path: envFile });
dotenv.config({ path: '.env', override: true }); // значения по умолчанию
```

**Файлы окружения для разных сред**:

```bash
# .env (общий, коммитится с дефолтами)
PORT=3000
LOG_LEVEL=info

# .env.development (не коммитится)
PORT=4000
DEBUG=true

# .env.production (не коммитится)
PORT=80
LOG_LEVEL=error
API_URL=https://api.prod.com

# .env.test (не коммитится)
PORT=0
DATABASE_URL=sqlite::memory:

# .env.local (локальные переопределения, не коммитится)
DB_PASSWORD=my-local-password
```

```javascript
// Загрузка в зависимости от NODE_ENV
const env = process.env.NODE_ENV || 'development';
require('dotenv').config({ path: `.env.${env}` });
require('dotenv').config({ path: '.env' }); // переопределяет недостающие

// Или с помощью dotenv-flow (пакет)
// npm install dotenv-flow
require('dotenv-flow').config();
// Автоматически загружает .env, .env.development, .env.local
```

**Валидация переменных окружения**:

```javascript
// Ручная валидация
const requiredEnvVars = ['DATABASE_URL', 'API_KEY', 'PORT'];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}

// Использование пакета envalid
// npm install envalid
const envalid = require('envalid');
const { str, port, bool, num } = envalid;

const env = envalid.cleanEnv(process.env, {
  NODE_ENV: str({ choices: ['development', 'production', 'test'] }),
  PORT: port({ default: 3000 }),
  DATABASE_URL: str(),
  DEBUG: bool({ default: false }),
  MAX_CONNECTIONS: num({ default: 10 })
});

console.log(env.PORT); // гарантированно число
console.log(env.DEBUG); // гарантированно boolean
```

**Практический пример: конфигурация приложения**:

```javascript
// config.js
require('dotenv').config();

module.exports = {
  // Сервер
  port: parseInt(process.env.PORT, 10) || 3000,
  host: process.env.HOST || 'localhost',
  env: process.env.NODE_ENV || 'development',
  
  // База данных
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT, 10) || 5432,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    name: process.env.DB_NAME,
    get url() {
      return `postgresql://${this.user}:${this.password}@${this.host}:${this.port}/${this.name}`;
    }
  },
  
  // API ключи
  apiKeys: {
    stripe: process.env.STRIPE_SECRET_KEY,
    sendgrid: process.env.SENDGRID_API_KEY
  },
  
  // Особые случаи
  allowedOrigins: process.env.ALLOWED_ORIGINS ? 
    JSON.parse(process.env.ALLOWED_ORIGINS) : 
    ['http://localhost:3000'],
  
  features: {
    enableCaching: process.env.ENABLE_CACHING === 'true',
    logLevel: process.env.LOG_LEVEL || 'info'
  },
  
  // Проверка на production
  isProduction: process.env.NODE_ENV === 'production',
  isDevelopment: process.env.NODE_ENV === 'development',
  isTest: process.env.NODE_ENV === 'test'
};

// app.js
const config = require('./config');

const server = http.createServer(app);
server.listen(config.port, config.host, () => {
  console.log(`Server running on ${config.host}:${config.port}`);
  console.log(`Environment: ${config.env}`);
});
```

**Безопасность: защита секретов**:

```bash
# .gitignore — обязательно добавить
.env
.env.*
!.env.example

# .env.example (коммитится, содержит структуру без значений)
PORT=3000
NODE_ENV=development
DB_HOST=localhost
DB_USER=your_username
DB_PASSWORD=your_password
API_KEY=your_api_key_here
```

```javascript
// Проверка в production
if (process.env.NODE_ENV === 'production') {
  if (!process.env.API_KEY || process.env.API_KEY === 'your_api_key_here') {
    console.error('FATAL: Invalid API_KEY in production');
    process.exit(1);
  }
}
```

**Работа с зашифрованными секретами**:

```bash
# Использование пакета dotenv-vault
npm install -g dotenv-vault

dotenv-vault new
dotenv-vault encrypt
dotenv-vault push

# В коде
require('dotenv').config();
console.log(process.env.SECRET_KEY);
```

**Переменные окружения в npm скриптах**:

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "NODE_ENV=development nodemon app.js",
    "prod": "NODE_ENV=production node app.js",
    "test": "NODE_ENV=test jest",
    "docker:run": "docker run -e PORT=3000 -e DB_HOST=postgres my-app",
    "env:example": "cp .env.example .env && echo '.env created'"
  }
}
```

**Использование в Docker**:

```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

```bash
# Запуск с переменными окружения

# Через -e
docker run -e PORT=3000 -e DB_HOST=postgres my-app

# Через --env-file
docker run --env-file .env.production my-app

# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    environment:
      - PORT=3000
      - NODE_ENV=production
    env_file:
      - .env.production
    ports:
      - "3000:3000"
```

**Платформенные переменные окружения**:

```javascript
// Определение окружения
if (process.env.DYNO) {
  console.log('Running on Heroku');
}

if (process.env.VERCEL_ENV) {
  console.log('Running on Vercel');
}

if (process.env.AWS_EXECUTION_ENV) {
  console.log('Running on AWS Lambda');
}

if (process.env.GITHUB_ACTIONS) {
  console.log('Running on GitHub Actions');
}
```

**Типичные ошибки**:

```javascript
// ❌ Неправильно: значение всегда строка
const maxUsers = process.env.MAX_USERS;
if (maxUsers > 100) { // maxUsers — строка, > приведёт к неожиданностям
  // Никогда не выполнится, если maxUsers = "50"
}

// ✅ Правильно: приводим к числу
const maxUsers = parseInt(process.env.MAX_USERS, 10) || 100;
if (maxUsers > 100) {
  // Работает корректно
}

// ❌ Неправильно: булевы переменные
if (process.env.ENABLE_CACHE) { // "false" — истинно как строка
  enableCache(); // выполнится даже при ENABLE_CACHE=false
}

// ✅ Правильно: явное сравнение
const enableCache = process.env.ENABLE_CACHE === 'true';
if (enableCache) {
  enableCache();
}

// ❌ Неправильно: .env не загружен
console.log(process.env.SECRET); // undefined
require('dotenv').config(); // Загружаем после использования

// ✅ Правильно: загружаем в самом начале
require('dotenv').config();
const secret = process.env.SECRET;
```

**Работа с TypeScript**:

```typescript
// types/env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
    PORT: string;
    DATABASE_URL: string;
    API_KEY: string;
  }
}

// config.ts
import dotenv from 'dotenv';
dotenv.config();

export const config = {
  port: parseInt(process.env.PORT, 10),
  nodeEnv: process.env.NODE_ENV,
  databaseUrl: process.env.DATABASE_URL,
  apiKey: process.env.API_KEY
} as const;
```

**Лучшие практики**:

1. **Никогда не коммитьте** `.env` файлы с реальными секретами в Git
2. **Всегда коммитьте** `.env.example` с описанием структуры
3. **Используйте разные файлы** для разных окружений
4. **Валидируйте** переменные при старте приложения
5. **Не храните секреты в коде** — только в переменных окружения
6. **В production используйте** платформенные механизмы (AWS Secrets Manager, HashiCorp Vault)
7. **Для булевых значений** используйте строгое сравнение с `'true'`
8. **Для чисел** всегда применяйте `parseInt` или `parseFloat`

**Полезные пакеты**:

| Пакет | Назначение |
|-------|-------------|
| `dotenv` | Загрузка из `.env` файлов |
| `dotenv-expand` | Переменные внутри переменных (`${VAR}`) |
| `envalid` | Валидация и типизация |
| `dotenv-vault` | Шифрование .env файлов |
| `cross-env` | Кроссплатформенная установка переменных |



**25. Введение в Express.js (минималистичный веб-фреймворк)**

Express.js — минималистичный и гибкий веб-фреймворк для Node.js. Оборачивает встроенный модуль `http`, добавляя маршрутизацию, middleware, шаблонизацию и удобную обработку запросов/ответов. Де-факто стандарт для веб-приложений на Node.js.

**Установка**:

```bash
npm init -y
npm install express
npm install -D nodemon  # для разработки
```

**Минимальное приложение**:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000, () => {
  console.log('Сервер запущен на http://localhost:3000');
});
```

**Сравнение Express с чистым `http`**:

```javascript
// Чистый http модуль
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, name: 'Alice' }]));
  } else if (req.method === 'GET' && req.url === '/posts') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, title: 'Post' }]));
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

// Express — лаконичнее
const express = require('express');
const app = express();

app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.get('/posts', (req, res) => {
  res.json([{ id: 1, title: 'Post' }]);
});
```

**Основные методы HTTP**:

```javascript
const express = require('express');
const app = express();

// GET — получение данных
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

// POST — создание ресурса
app.post('/api/users', (req, res) => {
  res.status(201).json({ id: 123 });
});

// PUT — полное обновление
app.put('/api/users/:id', (req, res) => {
  res.json({ updated: true });
});

// PATCH — частичное обновление
app.patch('/api/users/:id', (req, res) => {
  res.json({ patched: true });
});

// DELETE — удаление
app.delete('/api/users/:id', (req, res) => {
  res.status(204).send();
});
```

**Объекты request и response (расширенные)**:

```javascript
app.get('/example/:id', (req, res) => {
  // REQUEST
  console.log(req.params);      // { id: '123' } — параметры маршрута
  console.log(req.query);       // { sort: 'asc', page: '1' } — query параметры
  console.log(req.headers);     // заголовки запроса
  console.log(req.method);      // 'GET'
  console.log(req.url);         // '/example/123?sort=asc'
  console.log(req.ip);          // IP клиента
  console.log(req.path);        // '/example/123'
  
  // RESPONSE
  res.status(200);              // установка статуса
  res.set('X-Custom-Header', 'value'); // установка заголовка
  res.type('json');             // Content-Type: application/json
  
  // Методы отправки ответа
  res.send('<h1>HTML</h1>');    // автоматически определяет тип
  res.json({ key: 'value' });   // JSON с правильным Content-Type
  res.sendFile('/path/to/file'); // отправка файла
  res.redirect('/new-location'); // редирект (302 по умолчанию)
  res.download('/path/to/file'); // скачивание файла
  
  res.end();                    // завершение без данных
});
```

**Работа с параметрами маршрутов**:

```javascript
// Статические маршруты
app.get('/about', (req, res) => res.send('About page'));
app.get('/contact', (req, res) => res.send('Contact page'));

// Динамические параметры
app.get('/users/:userId', (req, res) => {
  const { userId } = req.params;
  res.json({ userId });
});

// Несколько параметров
app.get('/posts/:postId/comments/:commentId', (req, res) => {
  const { postId, commentId } = req.params;
  res.json({ postId, commentId });
});

// Необязательные параметры (с ?)
app.get('/products/:category/:productId?', (req, res) => {
  const { category, productId } = req.params;
  res.json({ category, productId });
});

// Параметры с регулярными выражениями
app.get('/users/:userId(\\d+)', (req, res) => {
  // только цифры в userId
  res.json({ userId: parseInt(req.params.userId) });
});
```

**Query параметры**:

```javascript
// GET /api/search?q=express&limit=10&page=1
app.get('/api/search', (req, res) => {
  const { q, limit = 10, page = 1, sort = 'desc' } = req.query;
  
  res.json({
    searchTerm: q,
    limit: parseInt(limit),
    page: parseInt(page),
    sort
  });
});

// GET /api/users?fields=id,name&includePosts=true
app.get('/api/users', (req, res) => {
  const fields = req.query.fields ? req.query.fields.split(',') : [];
  const includePosts = req.query.includePosts === 'true';
  
  res.json({ fields, includePosts });
});
```

**Чтение тела запроса (body parsing)**:

```javascript
const express = require('express');
const app = express();

// Встроенные парсеры
app.use(express.json());                    // парсинг JSON
app.use(express.urlencoded({ extended: true })); // парсинг form-data (x-www-form-urlencoded)
app.use(express.text());                    // парсинг plain text
app.use(express.raw());                     // парсинг сырых данных

// POST /api/users
app.post('/api/users', (req, res) => {
  console.log(req.body); // данные уже распарсены
  const { name, email, age } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  res.status(201).json({ id: Date.now(), name, email, age });
});

// POST /api/upload (form-data с файлами — требует multer)
// npm install multer
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/api/upload', upload.single('file'), (req, res) => {
  console.log(req.file); // информация о загруженном файле
  console.log(req.body); // текстовые поля формы
  res.json({ filename: req.file.filename });
});
```

**Маршрутизация (routing)**:

```javascript
// Группировка маршрутов по пути
const userRouter = express.Router();

userRouter.get('/', (req, res) => {
  res.json({ users: [] });
});

userRouter.get('/:id', (req, res) => {
  res.json({ userId: req.params.id });
});

userRouter.post('/', (req, res) => {
  res.status(201).json({ created: true });
});

userRouter.put('/:id', (req, res) => {
  res.json({ updated: true });
});

userRouter.delete('/:id', (req, res) => {
  res.status(204).send();
});

app.use('/api/users', userRouter);

// Вложенные маршруты
const postRouter = express.Router();
postRouter.get('/', (req, res) => res.json({ posts: [] }));
postRouter.get('/:postId', (req, res) => res.json({ postId: req.params.postId }));

app.use('/api/users/:userId/posts', (req, res, next) => {
  req.userId = req.params.userId; // проброс параметра
  next();
}, postRouter);
```

**Обработка ошибок**:

```javascript
// Синхронные ошибки (express перехватывает автоматически)
app.get('/error', (req, res) => {
  throw new Error('Что-то пошло не так');
});

// Асинхронные ошибки — нужно передавать в next()
app.get('/async-error', async (req, res, next) => {
  try {
    const data = await someAsyncOperation();
    res.json(data);
  } catch (err) {
    next(err); // передаём в обработчик ошибок
  }
});

// Обёртка для асинхронных обработчиков
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/safe-async', asyncHandler(async (req, res) => {
  const data = await someAsyncOperation();
  res.json(data);
}));

// 404 обработчик (когда маршрут не найден)
app.use((req, res, next) => {
  res.status(404).json({ error: 'Route not found' });
});

// Центральный обработчик ошибок (должен быть ПОСЛЕ всех маршрутов)
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  const status = err.status || 500;
  const message = process.env.NODE_ENV === 'production' 
    ? 'Internal Server Error' 
    : err.message;
  
  res.status(status).json({ error: message });
});
```

**Middleware — цепочки обработки**:

```javascript
// Кастомные middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // переход к следующему middleware или маршруту
};

const auth = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token || token !== 'Bearer secret-token') {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  req.user = { id: 1, name: 'Alice' };
  next();
};

const timing = (req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`Request took ${duration}ms`);
  });
  next();
};

// Подключение middleware
app.use(logger);      // глобальный
app.use(timing);
app.use('/api/admin', auth); // только для определённого пути

// Middleware для конкретного маршрута
app.get('/protected', auth, (req, res) => {
  res.json({ user: req.user });
});

// Массив middleware
const validateUser = (req, res, next) => {
  if (!req.body.name) {
    return res.status(400).json({ error: 'Name required' });
  }
  next();
};

const checkEmail = (req, res, next) => {
  if (!req.body.email?.includes('@')) {
    return res.status(400).json({ error: 'Invalid email' });
  }
  next();
};

app.post('/users', [validateUser, checkEmail], (req, res) => {
  res.json({ created: true });
});
```

**Статические файлы**:

```javascript
// Обслуживание статики
app.use(express.static('public'));
// http://localhost:3000/css/style.css → ./public/css/style.css

// Виртуальный префикс
app.use('/static', express.static('public'));
// http://localhost:3000/static/css/style.css

// Несколько директорий
app.use(express.static('public'));
app.use(express.static('uploads'));

// С кастомными опциями
app.use(express.static('public', {
  index: false,                 // отключить index.html
  maxAge: '1d',                 // кэширование на 1 день
  setHeaders: (res, path) => {
    if (path.endsWith('.html')) {
      res.set('Cache-Control', 'no-cache');
    }
  }
}));
```

**Полный пример реального приложения**:

```javascript
const express = require('express');
const app = express();
require('dotenv').config();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// Логирование
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path} - ${req.ip}`);
  next();
});

// Данные (имитация БД)
let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
];

// Маршруты API
app.get('/api/users', (req, res) => {
  const { limit = 10, offset = 0 } = req.query;
  const result = users.slice(offset, offset + limit);
  res.json({ users: result, total: users.length });
});

app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  res.json(user);
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

app.put('/api/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === id);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users[userIndex] = { ...users[userIndex], ...req.body };
  res.json(users[userIndex]);
});

app.delete('/api/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === id);
  
  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users.splice(userIndex, 1);
  res.status(204).send();
});

// Обработка ошибок
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something broke!' });
});

// Запуск
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

**Express vs альтернативы**:

| Фреймворк | Особенность | Когда выбрать |
|-----------|-------------|---------------|
| Express | Минималистичный, гибкий, огромная экосистема | Стандарт для большинства проектов |
| Fastify | Быстрый, валидация схем, низкий оверхед | Высокая производительность |
| Koa | Современный, async/await без колбэков | Современные приложения |
| NestJS | Модульный, TypeScript-first | Крупные корпоративные проекты |

**Полезные пакеты для Express**:

```bash
npm install helmet          # защита заголовков безопасности
npm install cors            # CORS поддержка
npm install compression     # gzip сжатие
npm install morgan          # логирование запросов
npm install rate-limit      # ограничение количества запросов
npm install express-session # сессии
npm install cookie-parser   # парсинг кук
```

```javascript
// Пример с популярными middleware
const helmet = require('helmet');
const cors = require('cors');
const compression = require('compression');
const morgan = require('morgan');
const rateLimit = require('express-rate-limit');

app.use(helmet());
app.use(cors());
app.use(compression());
app.use(morgan('combined'));
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
```

**Золотые правила**:

1. Всегда обрабатывайте ошибки (try/catch в async handler + next)
2. Разделяйте маршруты на модули через `express.Router()`
3. Используйте middleware для сквозной функциональности (логирование, аутентификация)
4. Не выполняйте тяжёлые CPU-задачи в обработчиках — выносите в worker threads
5. В production используйте reverse proxy (Nginx) и запускайте через PM2
6. Для валидации входных данных используйте Joi, Yup или express-validator



**26. Создание простого REST API с Express**

REST API (Representational State Transfer) — архитектурный стиль, где ресурсы идентифицируются URL, а операции — HTTP методами. Express — идеальный инструмент для быстрого создания REST API.

**Структура проекта**:

```
rest-api/
├── package.json
├── .env
├── server.js          # точка входа
├── routes/            # маршруты
│   ├── users.js
│   └── posts.js
├── controllers/       # бизнес-логика
│   ├── userController.js
│   └── postController.js
├── middleware/        # промежуточные обработчики
│   ├── auth.js
│   └── validation.js
├── models/            # работа с данными
│   └── User.js
└── data/              # имитация БД (JSON файлы)
    └── users.json
```

**Базовая настройка сервера**:

```javascript
// server.js
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

// Глобальные middleware
app.use(helmet());                    // безопасность
app.use(cors());                      // CORS
app.use(express.json());              // парсинг JSON
app.use(express.urlencoded({ extended: true })); // парсинг form data
app.use(morgan('dev'));               // логирование запросов

// Подключение маршрутов
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// Health check
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'OK', timestamp: new Date() });
});

// 404 обработчик
app.use('*', (req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// Глобальный обработчик ошибок
app.use((err, req, res, next) => {
  console.error(err.stack);
  const status = err.status || 500;
  const message = process.env.NODE_ENV === 'production' 
    ? 'Internal Server Error' 
    : err.message;
  res.status(status).json({ error: message });
});

app.listen(PORT, () => {
  console.log(`REST API running on http://localhost:${PORT}`);
});
```

**Модель данных (имитация базы данных)**:

```javascript
// models/User.js
const fs = require('fs').promises;
const path = require('path');

const DATA_PATH = path.join(__dirname, '../data/users.json');

class UserModel {
  // Инициализация данных
  static async init() {
    try {
      await fs.access(DATA_PATH);
    } catch {
      await fs.writeFile(DATA_PATH, JSON.stringify([], null, 2));
    }
  }
  
  // Чтение всех пользователей
  static async findAll() {
    const data = await fs.readFile(DATA_PATH, 'utf8');
    return JSON.parse(data);
  }
  
  // Поиск по ID
  static async findById(id) {
    const users = await this.findAll();
    return users.find(user => user.id === id);
  }
  
  // Поиск по email
  static async findByEmail(email) {
    const users = await this.findAll();
    return users.find(user => user.email === email);
  }
  
  // Создание пользователя
  static async create(userData) {
    const users = await this.findAll();
    const newUser = {
      id: Date.now(),
      ...userData,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    };
    users.push(newUser);
    await fs.writeFile(DATA_PATH, JSON.stringify(users, null, 2));
    return newUser;
  }
  
  // Обновление пользователя
  static async update(id, userData) {
    const users = await this.findAll();
    const index = users.findIndex(user => user.id === id);
    
    if (index === -1) return null;
    
    users[index] = {
      ...users[index],
      ...userData,
      updatedAt: new Date().toISOString()
    };
    
    await fs.writeFile(DATA_PATH, JSON.stringify(users, null, 2));
    return users[index];
  }
  
  // Удаление пользователя
  static async delete(id) {
    const users = await this.findAll();
    const filtered = users.filter(user => user.id !== id);
    
    if (filtered.length === users.length) return false;
    
    await fs.writeFile(DATA_PATH, JSON.stringify(filtered, null, 2));
    return true;
  }
}

module.exports = UserModel;
```

**Контроллеры (бизнес-логика)**:

```javascript
// controllers/userController.js
const UserModel = require('../models/User');

// GET /api/users
const getUsers = async (req, res, next) => {
  try {
    const { limit = 10, offset = 0, sort = 'desc' } = req.query;
    
    let users = await UserModel.findAll();
    
    // Сортировка
    users.sort((a, b) => {
      return sort === 'desc' 
        ? b.createdAt.localeCompare(a.createdAt)
        : a.createdAt.localeCompare(b.createdAt);
    });
    
    // Пагинация
    const paginatedUsers = users.slice(
      parseInt(offset), 
      parseInt(offset) + parseInt(limit)
    );
    
    res.json({
      success: true,
      data: paginatedUsers,
      pagination: {
        total: users.length,
        limit: parseInt(limit),
        offset: parseInt(offset),
        hasMore: parseInt(offset) + parseInt(limit) < users.length
      }
    });
  } catch (err) {
    next(err);
  }
};

// GET /api/users/:id
const getUserById = async (req, res, next) => {
  try {
    const id = parseInt(req.params.id);
    const user = await UserModel.findById(id);
    
    if (!user) {
      return res.status(404).json({ 
        success: false, 
        error: 'User not found' 
      });
    }
    
    res.json({ success: true, data: user });
  } catch (err) {
    next(err);
  }
};

// POST /api/users
const createUser = async (req, res, next) => {
  try {
    const { name, email, age, role = 'user' } = req.body;
    
    // Валидация
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        error: 'Name and email are required'
      });
    }
    
    // Проверка уникальности email
    const existingUser = await UserModel.findByEmail(email);
    if (existingUser) {
      return res.status(409).json({
        success: false,
        error: 'Email already exists'
      });
    }
    
    const newUser = await UserModel.create({
      name,
      email,
      age: age || null,
      role
    });
    
    res.status(201).json({ success: true, data: newUser });
  } catch (err) {
    next(err);
  }
};

// PUT /api/users/:id (полное обновление)
const updateUser = async (req, res, next) => {
  try {
    const id = parseInt(req.params.id);
    const { name, email, age, role } = req.body;
    
    const user = await UserModel.findById(id);
    if (!user) {
      return res.status(404).json({ 
        success: false, 
        error: 'User not found' 
      });
    }
    
    // При PUT все поля обязательны
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        error: 'Name and email are required for full update'
      });
    }
    
    const updatedUser = await UserModel.update(id, {
      name,
      email,
      age: age || user.age,
      role: role || user.role
    });
    
    res.json({ success: true, data: updatedUser });
  } catch (err) {
    next(err);
  }
};

// PATCH /api/users/:id (частичное обновление)
const patchUser = async (req, res, next) => {
  try {
    const id = parseInt(req.params.id);
    const updates = req.body;
    
    const user = await UserModel.findById(id);
    if (!user) {
      return res.status(404).json({ 
        success: false, 
        error: 'User not found' 
      });
    }
    
    // Не позволяем обновлять id
    delete updates.id;
    
    const updatedUser = await UserModel.update(id, updates);
    res.json({ success: true, data: updatedUser });
  } catch (err) {
    next(err);
  }
};

// DELETE /api/users/:id
const deleteUser = async (req, res, next) => {
  try {
    const id = parseInt(req.params.id);
    const deleted = await UserModel.delete(id);
    
    if (!deleted) {
      return res.status(404).json({ 
        success: false, 
        error: 'User not found' 
      });
    }
    
    res.status(204).send();
  } catch (err) {
    next(err);
  }
};

module.exports = {
  getUsers,
  getUserById,
  createUser,
  updateUser,
  patchUser,
  deleteUser
};
```

**Маршруты**:

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { validateUser, validateUserId } = require('../middleware/validation');
const { authenticate, authorize } = require('../middleware/auth');

// Публичные маршруты (без аутентификации)
router.get('/', userController.getUsers);
router.get('/:id', validateUserId, userController.getUserById);

// Защищённые маршруты (требуют аутентификации)
router.post('/', 
  authenticate, 
  validateUser, 
  userController.createUser
);

router.put('/:id', 
  authenticate, 
  validateUserId, 
  validateUser, 
  userController.updateUser
);

router.patch('/:id', 
  authenticate, 
  validateUserId, 
  userController.patchUser
);

router.delete('/:id', 
  authenticate, 
  authorize('admin'), 
  validateUserId, 
  userController.deleteUser
);

module.exports = router;
```

**Middleware для валидации**:

```javascript
// middleware/validation.js
const validateUser = (req, res, next) => {
  const { name, email, age } = req.body;
  const errors = [];
  
  if (!name || typeof name !== 'string' || name.length < 2) {
    errors.push('Name must be a string with at least 2 characters');
  }
  
  if (!email || !email.includes('@') || !email.includes('.')) {
    errors.push('Valid email is required');
  }
  
  if (age !== undefined) {
    const ageNum = parseInt(age);
    if (isNaN(ageNum) || ageNum < 0 || ageNum > 150) {
      errors.push('Age must be a number between 0 and 150');
    }
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ 
      success: false, 
      errors 
    });
  }
  
  next();
};

const validateUserId = (req, res, next) => {
  const id = parseInt(req.params.id);
  
  if (isNaN(id) || id <= 0) {
    return res.status(400).json({
      success: false,
      error: 'Invalid user ID'
    });
  }
  
  req.params.id = id; // преобразуем в число
  next();
};

module.exports = { validateUser, validateUserId };
```

**Middleware для аутентификации**:

```javascript
// middleware/auth.js
const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      success: false,
      error: 'No token provided'
    });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    // В реальном проекте — верификация JWT
    // const decoded = jwt.verify(token, process.env.JWT_SECRET);
    // req.user = decoded;
    
    // Имитация для примера
    if (token === 'secret-token') {
      req.user = { id: 1, role: 'admin' };
      next();
    } else {
      throw new Error('Invalid token');
    }
  } catch (err) {
    res.status(401).json({ 
      success: false, 
      error: 'Invalid or expired token' 
    });
  }
};

const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required'
      });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        error: 'Insufficient permissions'
      });
    }
    
    next();
  };
};

module.exports = { authenticate, authorize };
```

**Пример запросов к API**:

```bash
# Получение всех пользователей
curl http://localhost:3000/api/users
curl http://localhost:3000/api/users?limit=5&offset=10&sort=asc

# Получение конкретного пользователя
curl http://localhost:3000/api/users/123

# Создание пользователя (POST)
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer secret-token" \
  -d '{"name":"Alice","email":"alice@example.com","age":30}'

# Полное обновление (PUT)
curl -X PUT http://localhost:3000/api/users/123 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer secret-token" \
  -d '{"name":"Alice Smith","email":"alice.smith@example.com","age":31}'

# Частичное обновление (PATCH)
curl -X PATCH http://localhost:3000/api/users/123 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer secret-token" \
  -d '{"age":32}'

# Удаление пользователя (DELETE)
curl -X DELETE http://localhost:3000/api/users/123 \
  -H "Authorization: Bearer secret-token"
```

**Обработка ошибок и статус-коды**:

```javascript
// Статус-коды HTTP в REST API
// 200 OK — успешный GET, PUT, PATCH
// 201 Created — успешный POST
// 204 No Content — успешный DELETE
// 400 Bad Request — ошибка валидации
// 401 Unauthorized — нет аутентификации
// 403 Forbidden — недостаточно прав
// 404 Not Found — ресурс не найден
// 409 Conflict — конфликт (например, duplicate email)
// 500 Internal Server Error — ошибка сервера
```

**Логирование запросов**:

```javascript
// middleware/logger.js
const logger = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const status = res.statusCode;
    const logLevel = status >= 500 ? 'ERROR' : status >= 400 ? 'WARN' : 'INFO';
    
    console.log(
      `${logLevel} | ${req.method} ${req.originalUrl} | ${status} | ${duration}ms | ${req.ip}`
    );
    
    // Отправка метрик (Prometheus, DataDog и т.д.)
    if (global.metrics) {
      global.metrics.recordHttpRequest(req.method, req.path, status, duration);
    }
  });
  
  next();
};

app.use(logger);
```

**Расширение API: связанные ресурсы**:

```javascript
// routes/users.js — вложенные маршруты
// GET /api/users/:userId/posts
router.get('/:userId/posts', async (req, res, next) => {
  try {
    const userId = req.params.userId;
    const posts = await PostModel.findByUserId(userId);
    res.json({ success: true, data: posts });
  } catch (err) {
    next(err);
  }
});

// GET /api/users/:userId/posts/:postId
router.get('/:userId/posts/:postId', async (req, res, next) => {
  try {
    const { userId, postId } = req.params;
    const post = await PostModel.findByIdAndUserId(postId, userId);
    
    if (!post) {
      return res.status(404).json({ success: false, error: 'Post not found' });
    }
    
    res.json({ success: true, data: post });
  } catch (err) {
    next(err);
  }
});
```

**Тестирование API**:

```javascript
// tests/api.test.js (с использованием supertest)
const request = require('supertest');
const app = require('../server');

describe('User API', () => {
  describe('GET /api/users', () => {
    it('should return list of users', async () => {
      const res = await request(app)
        .get('/api/users')
        .expect('Content-Type', /json/)
        .expect(200);
      
      expect(res.body.success).toBe(true);
      expect(Array.isArray(res.body.data)).toBe(true);
    });
    
    it('should support pagination', async () => {
      const res = await request(app)
        .get('/api/users?limit=2&offset=0')
        .expect(200);
      
      expect(res.body.pagination.limit).toBe(2);
      expect(res.body.data.length).toBeLessThanOrEqual(2);
    });
  });
  
  describe('POST /api/users', () => {
    it('should create new user', async () => {
      const newUser = {
        name: 'Test User',
        email: 'test@example.com',
        age: 25
      };
      
      const res = await request(app)
        .post('/api/users')
        .set('Authorization', 'Bearer secret-token')
        .send(newUser)
        .expect(201);
      
      expect(res.body.data.name).toBe(newUser.name);
      expect(res.body.data.email).toBe(newUser.email);
    });
    
    it('should reject missing fields', async () => {
      const res = await request(app)
        .post('/api/users')
        .set('Authorization', 'Bearer secret-token')
        .send({ name: 'Only Name' })
        .expect(400);
      
      expect(res.body.error).toBeDefined();
    });
  });
});
```

**Рекомендации по проектированию REST API**:

| Принцип | Пример |
|---------|--------|
| Используйте существительные для ресурсов | `/users` вместо `/getUsers` |
| HTTP методы отражают действия | `GET /users/123` — чтение, `DELETE /users/123` — удаление |
| Версионируйте API | `/v1/users`, `/v2/users` |
| Используйте множественное число для коллекций | `/users`, `/posts` |
| Фильтрация через query параметры | `GET /users?role=admin&active=true` |
| Пагинация | `GET /users?limit=20&offset=40` |
| Сортировка | `GET /users?sort=-createdAt` |
| Выбор полей | `GET /users?fields=id,name,email` |
| Статус-коды имеют смысл | 201 для создания, 204 для удаления |

**Готовый REST API можно расширять**:
- Подключение реальной БД (PostgreSQL, MongoDB)
- JWT аутентификация
- Swagger/OpenAPI документация
- Rate limiting
- Caching (Redis)
- WebSocket для real-time
- GraphQL для сложных запросов



**27. Middleware в Express (встроенные, сторонние, кастомные)**

Middleware (промежуточный обработчик) — функции, имеющие доступ к объектам `req` (запрос) и `res` (ответ), а также к следующей функции `next()` в цикле обработки. Могут выполнять любой код, изменять `req/res`, завершать цикл или передавать управление дальше.

**Принцип работы middleware**:

```
Запрос → Middleware 1 → Middleware 2 → Middleware 3 → Маршрут → Ответ
           ↓              ↓              ↓
        next()         next()         next()
           ↓              ↓              ↓
        Ошибка ←────── Ошибка ←────── Ошибка (если есть)
```

**Базовый пример**:

```javascript
const express = require('express');
const app = express();

// Простейшее middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next(); // передаём управление дальше
});

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(3000);
```

**Жизненный цикл middleware**:

```javascript
// Middleware может:
// 1. Выполнить код
// 2. Изменить req/res
// 3. Завершить запрос (res.send(), res.json())
// 4. Вызвать next() для передачи следующему middleware
// 5. Вызвать next(err) для передачи ошибки

const middlewareExample = (req, res, next) => {
  // Добавляем данные в req
  req.requestTime = Date.now();
  
  // Модифицируем заголовки
  res.setHeader('X-Processed-By', 'Express');
  
  // Условная логика
  if (req.headers['x-block'] === 'true') {
    return res.status(403).json({ error: 'Blocked by middleware' });
  }
  
  // Передаём управление
  next();
  
  // Код после next() выполнится после всех последующих middleware
  console.log('Response уже отправлен или будет отправлен');
};
```

**Типы middleware**:

```javascript
// 1. Application-level middleware (применяется ко всем запросам)
app.use((req, res, next) => next());

// 2. Router-level middleware (применяется к группе маршрутов)
const router = express.Router();
router.use((req, res, next) => next());

// 3. Error-handling middleware (4 аргумента — ОБЯЗАТЕЛЬНО)
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send('Something broke!');
});

// 4. Built-in middleware
app.use(express.json());
app.use(express.static('public'));

// 5. Third-party middleware
const morgan = require('morgan');
app.use(morgan('dev'));
```

**Встроенные middleware Express**:

```javascript
const express = require('express');
const app = express();

// 1. express.json() — парсинг JSON тела запроса
app.use(express.json({ limit: '10mb' }));
app.post('/api/data', (req, res) => {
  console.log(req.body); // распарсенный JSON
  res.json(req.body);
});

// 2. express.urlencoded() — парсинг form-data
app.use(express.urlencoded({ extended: true, limit: '10mb' }));
app.post('/api/form', (req, res) => {
  console.log(req.body); // { name: 'Alice', age: '30' }
  res.json(req.body);
});

// 3. express.text() — парсинг plain text
app.use(express.text({ type: 'text/plain' }));
app.post('/api/text', (req, res) => {
  console.log(req.body); // строка
  res.send('Received');
});

// 4. express.raw() — парсинг сырых данных (Buffer)
app.use(express.raw({ type: 'application/octet-stream' }));
app.post('/api/binary', (req, res) => {
  console.log(req.body instanceof Buffer); // true
  res.send('Binary received');
});

// 5. express.static() — раздача статических файлов
app.use('/static', express.static('public'));
app.use(express.static('uploads', { maxAge: '1d' }));
```

**Сторонние middleware (популярные)**:

```javascript
// Установка: npm install morgan helmet cors compression

const morgan = require('morgan');      // логирование
const helmet = require('helmet');      // безопасность (заголовки)
const cors = require('cors');          // CORS поддержка
const compression = require('compression'); // gzip сжатие

// Логирование в разных форматах
app.use(morgan('tiny'));    // GET / 304 - - 0.933 ms
app.use(morgan('combined')); // ::1 - - [10/Dec/2024:12:00:00] "GET /" 304 -
app.use(morgan(':method :url :status :response-time ms'));

// Безопасность
app.use(helmet()); // добавляет заголовки: X-Content-Type-Options, X-Frame-Options и др.

// CORS — настройка доступа с других доменов
app.use(cors({
  origin: ['http://localhost:3000', 'https://example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

// Сжатие ответов
app.use(compression({ level: 6 })); // уровень сжатия 1-9
```

**Популярные сторонние middleware (продолжение)**:

```javascript
// npm install express-rate-limit
const rateLimit = require('express-rate-limit');

// Ограничение количества запросов
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100, // макс 100 запросов с одного IP
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false
});
app.use('/api', limiter);

// npm install express-session
const session = require('express-session');
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: process.env.NODE_ENV === 'production' }
}));

// npm install cookie-parser
const cookieParser = require('cookie-parser');
app.use(cookieParser());
app.get('/set-cookie', (req, res) => {
  res.cookie('user', 'Alice', { maxAge: 900000, httpOnly: true });
  res.send('Cookie set');
});

// npm install express-fileupload
const fileupload = require('express-fileupload');
app.use(fileupload({
  limits: { fileSize: 5 * 1024 * 1024 },
  abortOnLimit: true
}));
```

**Кастомные middleware: примеры**:

```javascript
// 1. Middleware для логирования времени выполнения
const timingMiddleware = (req, res, next) => {
  const start = Date.now();
  
  // Сохраняем оригинальный метод end
  const originalEnd = res.end;
  res.end = function(...args) {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} - ${duration}ms`);
    originalEnd.apply(res, args);
  };
  
  next();
};

app.use(timingMiddleware);

// 2. Middleware для аутентификации
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  try {
    // В реальности — jwt.verify(token, SECRET)
    const decoded = { id: 1, role: 'user' };
    req.user = decoded;
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

app.use('/api/protected', authMiddleware);

// 3. Middleware для валидации
const validateUser = (req, res, next) => {
  const { name, email } = req.body;
  const errors = [];
  
  if (!name || name.length < 2) {
    errors.push('Name must be at least 2 characters');
  }
  
  if (!email || !email.includes('@')) {
    errors.push('Valid email is required');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
};

app.post('/api/users', validateUser, (req, res) => {
  res.json({ created: true });
});

// 4. Middleware для обработки ошибок
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/api/async-error', asyncHandler(async (req, res) => {
  const data = await someAsyncOperation(); // если ошибка — next(err)
  res.json(data);
}));

// 5. Middleware с параметрами
const requireRole = (role) => {
  return (req, res, next) => {
    if (req.user?.role !== role && req.user?.role !== 'admin') {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

app.delete('/api/admin/users/:id', requireRole('admin'), (req, res) => {
  res.json({ deleted: true });
});
```

**Порядок middleware критичен**:

```javascript
// ❌ Неправильный порядок
app.get('/api/users', (req, res) => {
  res.json(users);
});

app.use(express.json()); // Поздно — тело запроса уже не распарсится

// ✅ Правильный порядок
app.use(express.json()); // Сначала парсинг
app.use(morgan('dev'));  // Потом логирование
app.use(helmet());       // Потом безопасность
app.use('/api', limiter); // Потом rate limiting
app.use('/api', authMiddleware); // Потом аутентификация
app.use('/api/users', userRoutes); // Потом маршруты
app.use(errorMiddleware); // В самом конце — обработка ошибок
```

**Middleware для конкретных путей**:

```javascript
// Только для /api маршрутов
app.use('/api', (req, res, next) => {
  console.log('API request');
  next();
});

// Для всех маршрутов, начинающихся с /admin
app.use('/admin', authMiddleware);
app.use('/admin', requireRole('admin'));

// Для конкретного метода и пути
app.get('/special', middleware1, middleware2, middleware3, (req, res) => {
  res.send('Multiple middleware');
});

// Массив middleware
const middlewares = [authMiddleware, validateUser, logRequest];
app.post('/api/users', middlewares, (req, res) => {
  res.json({ created: true });
});
```

**Обработка ошибок в middleware**:

```javascript
// Синхронные ошибки — Express перехватывает автоматически
app.use((req, res, next) => {
  throw new Error('Something went wrong');
});

// Асинхронные ошибки — нужно передавать в next()
app.use((req, res, next) => {
  someAsyncOperation((err, result) => {
    if (err) return next(err);
    res.json(result);
  });
});

// Promise ошибки
app.get('/api/data', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    next(err); // передаём в error-handling middleware
  }
});

// Специальный error-handling middleware (4 аргумента!)
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  // Определяем статус
  const status = err.status || err.statusCode || 500;
  const message = process.env.NODE_ENV === 'production'
    ? (status === 500 ? 'Internal Server Error' : err.message)
    : err.message;
  
  res.status(status).json({
    error: message,
    ...(process.env.NODE_ENV !== 'production' && { stack: err.stack })
  });
});
```

**Практический пример: комплекс middleware**:

```javascript
const express = require('express');
const app = express();

// 1. Глобальные middleware (порядок важен!)
app.use(helmet());                                    // безопасность
app.use(cors());                                      // CORS
app.use(compression());                               // сжатие
app.use(express.json({ limit: '10mb' }));            // парсинг JSON
app.use(express.urlencoded({ extended: true }));     // парсинг форм
app.use(cookieParser());                              // парсинг кук
app.use(morgan('combined'));                          // логирование

// 2. Кастомные middleware для каждого запроса
app.use((req, res, next) => {
  req.startTime = Date.now();
  res.setHeader('X-Powered-By', 'Express-Middleware');
  next();
});

// 3. Rate limiting для API
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  skipSuccessfulRequests: true
});
app.use('/api', apiLimiter);

// 4. Аутентификация для защищённых маршрутов
const requireAuth = (req, res, next) => {
  const token = req.cookies.token || req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid or expired token' });
  }
};

// 5. Логирование для админки
app.use('/admin', requireAuth);
app.use('/admin', (req, res, next) => {
  console.log(`Admin access by user ${req.user.id} at ${new Date()}`);
  next();
});

// 6. Маршруты
app.get('/api/public', (req, res) => {
  res.json({ message: 'Public data' });
});

app.get('/api/private', requireAuth, (req, res) => {
  res.json({ user: req.user });
});

// 7. 404 middleware (если ни один маршрут не сработал)
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// 8. Error-handling middleware (последним!)
app.use((err, req, res, next) => {
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';
  
  console.error(`[ERROR] ${status}: ${message}`);
  console.error(err.stack);
  
  res.status(status).json({
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});

app.listen(3000);
```

**Тестирование middleware**:

```javascript
// middleware/logger.js
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
};

module.exports = logger;

// tests/middleware.test.js
const request = require('supertest');
const express = require('express');
const logger = require('../middleware/logger');

describe('Logger Middleware', () => {
  let app;
  
  beforeEach(() => {
    app = express();
    app.use(logger);
    app.get('/test', (req, res) => res.send('OK'));
  });
  
  it('should log request method and url', async () => {
    const consoleSpy = jest.spyOn(console, 'log');
    await request(app).get('/test');
    expect(consoleSpy).toHaveBeenCalledWith('GET /test');
    consoleSpy.mockRestore();
  });
});
```

**Золотые правила middleware**:

1. **Порядок имеет значение** — регистрируйте middleware в правильной последовательности
2. **Не забывайте `next()`** — иначе запрос зависнет
3. **Вызывайте `next(err)`** для передачи ошибок в error-handler
4. **Error-handling middleware** должен иметь 4 аргумента и быть последним
5. **Не выполняйте тяжёлые синхронные операции** — они заблокируют цикл событий
6. **Для асинхронных операций** используйте try/catch с next(err)
7. **Модифицируйте `req`** для передачи данных между middleware (например, `req.user`)
8. **Завершайте запрос** только один раз (не вызывайте `res.send()` дважды)

**Когда использовать каждый тип**:

| Тип | Сценарий |
|-----|----------|
| Built-in | Парсинг тел запросов, статика |
| Third-party | Логирование, безопасность, сжатие, лимиты |
| Application-level | Аутентификация, логирование, CORS |
| Router-level | Группировка маршрутов с общей логикой |
| Error-handling | Централизованная обработка ошибок |



**28. Отладка Node.js приложений (встроенный отладчик, VS Code, `--inspect`)**

Отладка Node.js — процесс поиска и исправления ошибок с использованием контрольных точек, пошагового выполнения, инспекции переменных и анализа стека вызовов. Node.js предоставляет встроенный отладчик через протокол Chrome DevTools.

**Встроенный отладчик Node.js (базовый)**:

```bash
# Запуск с флагом debug (старый стиль, не рекомендуется)
node debug app.js

# Современный способ — инспектор
node --inspect app.js
node --inspect-brk app.js  # Останавливается на первой строке
```

```javascript
// app.js
function calculate(a, b) {
  const result = a + b;
  return result * 2;
}

const x = 10;
const y = 20;
const z = calculate(x, y);
console.log(z);
```

```bash
# Вывод при запуске:
# Debugger listening on ws://127.0.0.1:9229/abc123
# For help, see: https://nodejs.org/en/docs/inspector
```

**Chrome DevTools (графический интерфейс)**:

```bash
# 1. Запустите приложение с инспектором
node --inspect app.js

# 2. Откройте Chrome и перейдите по адресу:
chrome://inspect

# 3. Нажмите "Open dedicated DevTools for Node"
```

```javascript
// Пример для отладки
function fetchUser(id) {
  // Установите breakpoint на этой строке в DevTools
  console.log(`Fetching user ${id}`);
  return { id, name: 'Alice' };
}

function processOrder(orderId) {
  const user = fetchUser(123);
  const total = calculateTotal(orderId);
  return { user, total };
}

function calculateTotal(orderId) {
  // Точка останова
  let total = 0;
  for (let i = 0; i < 100; i++) {
    total += i;
  }
  return total;
}

processOrder(456);
```

**Флаги инспектора**:

```bash
# Базовый запуск с инспектором
node --inspect app.js

# Остановка на первой строке (полезно для ранней отладки)
node --inspect-brk app.js

# Указать порт (по умолчанию 9229)
node --inspect=9230 app.js

# Указать хост и порт
node --inspect=192.168.1.100:9230 app.js

# Отключение инспектора (для production)
node --inspect=0 app.js

# Только ожидание подключения без выполнения кода
node --inspect-brk --inspect-port=0 app.js
```

**VS Code отладка**:

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "program": "${workspaceFolder}/app.js",
      "cwd": "${workspaceFolder}",
      "args": ["--port=3000"],
      "env": {
        "NODE_ENV": "development"
      },
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Process",
      "port": 9229,
      "restart": true,
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Jest Tests",
      "program": "${workspaceFolder}/node_modules/jest/bin/jest",
      "args": ["--runInBand", "--coverage=false"],
      "console": "integratedTerminal"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Nodemon",
      "runtimeExecutable": "nodemon",
      "program": "${workspaceFolder}/app.js",
      "restart": true,
      "console": "integratedTerminal",
      "env": {
        "NODE_ENV": "development"
      }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug with Args",
      "program": "${workspaceFolder}/app.js",
      "args": ["--debug", "test.txt"],
      "outputCapture": "std"
    }
  ]
}
```

**VS Code отладка (продвинутые конфигурации)**:

```json
{
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Express App",
      "runtimeArgs": ["--inspect-brk", "bin/www"],
      "cwd": "${workspaceFolder}",
      "outputCapture": "std",
      "sourceMaps": true,
      "resolveSourceMapLocations": [
        "${workspaceFolder}/**",
        "!**/node_modules/**"
      ]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug TypeScript",
      "runtimeArgs": ["-r", "ts-node/register"],
      "args": ["${workspaceFolder}/src/index.ts"],
      "sourceMaps": true,
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Worker Threads",
      "program": "${workspaceFolder}/worker.js",
      "runtimeArgs": ["--inspect=9230"]
    }
  ]
}
```

**Точки останова (Breakpoints) в коде**:

```javascript
// Статическая точка останова (работает в любом отладчике)
debugger;

function processData(data) {
  debugger; // Остановка здесь
  const result = data.map(item => item * 2);
  debugger; // И здесь
  return result;
}

// Условная точка останова (только в DevTools/VS Code)
// В интерфейсе: правый клик по строке → Conditional Breakpoint
// Условие: i === 50

// Лог-точки (Logpoint) — вывод в консоль без остановки
// В VS Code: правый клик → Add Logpoint
// Сообщение: "Value: {value}"
```

**Отладка в терминале (команды отладчика)**:

```bash
# Запуск в режиме отладки
node inspect app.js

# Команды в консоли отладчика
# cont, c - продолжить выполнение
# next, n - шаг с обходом (не заходит в функции)
# step, s - шаг с заходом в функции
# out, o - выйти из текущей функции
# pause - приостановить выполнение
# watch('expression') - добавить выражение в наблюдение
# unwatch('expression') - удалить из наблюдения
# watchers - показать все наблюдения
# repl - открыть REPL в текущем контексте
# exec expression - выполнить выражение
# list(n) - показать n строк кода вокруг текущей позиции
# backtrace - показать стек вызовов
# setBreakpoint()/sb() - установить точку останова
# clearBreakpoint()/cb() - удалить точку останова
# restart - перезапустить скрипт
# .exit - выйти
```

```javascript
// Пример для терминальной отладки
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

function calculateFactorial(n) {
  if (n <= 1) return 1;
  return n * calculateFactorial(n - 1);
}

const result = fibonacci(5);
const fact = calculateFactorial(5);
console.log({ result, fact });
```

```bash
node inspect app.js
# < Debugger listening...
# < ok
debug> sb(8)               # точка останова на строке 8
debug> cont                # запустить
debug> repl                # войти в REPL
> n
5
> .exit
debug> exec result
undefined
debug> next                # шаг
debug> backtrace           # стек вызовов
debug> watch('n')          # наблюдать переменную n
debug> watchers            # показать наблюдения
debug> restart             # перезапустить
```

**Отладка асинхронного кода**:

```javascript
// Проблема: отладка асинхронных операций
async function fetchData(url) {
  debugger; // Точка останова
  const response = await fetch(url);
  const data = await response.json();
  debugger; // Остановится после await
  return data;
}

function processUsers() {
  debugger;
  return fetchData('/api/users')
    .then(users => {
      debugger; // В колбэке Promise
      return users.map(u => u.name);
    })
    .catch(err => {
      debugger; // Ошибка
      console.error(err);
    });
}

// Решение: использовать асинхронный стек в DevTools
// В Chrome DevTools включите "Async stack traces"
```

**Отладка ошибок и исключений**:

```javascript
// Настройка обработки исключений
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  // Логирование ошибки перед выходом
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  // Продолжаем работу, но логируем
});

// Запуск с отладкой исключений
// node --inspect --inspect-brk app.js
// В Chrome DevTools: включите "Pause on exceptions"
```

**Продвинутые техники отладки**:

```javascript
// 1. Мониторинг производительности
const inspector = require('inspector');
const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    // Ваш код
    setTimeout(() => {
      session.post('Profiler.stop', (err, { profile }) => {
        // Анализ профиля
        console.log('Profiling completed');
      });
    }, 5000);
  });
});

// 2. Кастомное логирование с контекстом
class DebugLogger {
  constructor(namespace) {
    this.namespace = namespace;
  }
  
  log(...args) {
    console.log(`[${this.namespace}]`, ...args);
  }
  
  trace(message) {
    const stack = new Error().stack?.split('\n')[2];
    console.log(`[TRACE] ${this.namespace}: ${message}\n  at ${stack?.trim()}`);
  }
  
  time(label) {
    console.time(`${this.namespace}:${label}`);
  }
  
  timeEnd(label) {
    console.timeEnd(`${this.namespace}:${label}`);
  }
}

const logger = new DebugLogger('API');
logger.log('Starting request');
logger.time('database-query');
// ... операция
logger.timeEnd('database-query');

// 3. Условный отладочный вывод
const DEBUG = process.env.DEBUG === 'true';

function debug(message, data) {
  if (DEBUG) {
    console.error(`[DEBUG] ${message}`, data || '');
  }
}

debug('User login', { userId: 123, timestamp: Date.now() });
```

**Отладка в production (без остановки)**:

```javascript
// Использование модуля debug (npm install debug)
const debug = require('debug');
const httpDebug = debug('http');
const dbDebug = debug('database');

// Запуск: DEBUG=http,database node app.js
// Запуск всего: DEBUG=* node app.js

app.get('/api/users', async (req, res) => {
  httpDebug('GET /api/users', req.query);
  
  try {
    dbDebug('Executing query: SELECT * FROM users');
    const users = await db.query('SELECT * FROM users');
    dbDebug('Query returned %d rows', users.length);
    
    res.json(users);
  } catch (err) {
    httpDebug('Error: %O', err);
    res.status(500).send();
  }
});

// Логирование с уровнями (pino, winston)
const pino = require('pino');
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: { colorize: true }
  }
});

logger.debug({ userId: 123 }, 'Processing user');
logger.info('Server started');
logger.error({ err }, 'Database connection failed');
```

**Отладка утечек памяти**:

```bash
# Запуск с отслеживанием памяти
node --inspect --max-old-space-size=4096 app.js

# В Chrome DevTools:
# - Перейдите в Memory вкладку
# - Сделайте Heap Snapshot
# - Выполните действия
# - Сделайте второй снимок
# - Сравните (Comparison view) для поиска утечек
```

```javascript
// Мониторинг памяти в коде
function checkMemory() {
  const used = process.memoryUsage();
  console.log({
    rss: `${Math.round(used.rss / 1024 / 1024)} MB`,
    heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
    heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
    external: `${Math.round(used.external / 1024 / 1024)} MB`
  });
}

setInterval(checkMemory, 30000);
```

**Отладка сетевых запросов**:

```javascript
// Логирование всех HTTP запросов
const http = require('http');
const originalRequest = http.request;

http.request = function(...args) {
  const options = args[0];
  console.log(`HTTP Request: ${options.method || 'GET'} ${options.protocol}//${options.hostname}:${options.port}${options.path}`);
  
  const req = originalRequest.apply(this, args);
  
  req.on('response', (res) => {
    console.log(`HTTP Response: ${res.statusCode}`);
  });
  
  return req;
};

// Использование отладчика для network в DevTools
// Откройте chrome://inspect
// Вкладка Network покажет все запросы из Node.js
```

**Отладка многопроцессных приложений (cluster)**:

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  // Запуск каждого воркера с отдельным портом для отладки
  const numWorkers = os.cpus().length;
  const debugPort = 9229;
  
  for (let i = 0; i < numWorkers; i++) {
    const worker = cluster.fork({
      NODE_OPTIONS: `--inspect=${debugPort + i}`
    });
    
    worker.on('message', (msg) => {
      console.log(`Worker ${worker.id}: ${msg}`);
    });
  }
} else {
  // Воркер запущен с отладчиком на своём порту
  console.log(`Worker ${cluster.worker.id} debug on port ${9229 + cluster.worker.id - 1}`);
  
  // Приложение
  const express = require('express');
  const app = express();
  
  app.get('/', (req, res) => {
    res.send(`Worker ${cluster.worker.id}`);
  });
  
  app.listen(3000);
}
```

**Лучшие практики отладки**:

| Сценарий | Инструмент |
|----------|-----------|
| Локальная разработка | VS Code + `--inspect` |
| Быстрая проверка | `console.log()` + `util.inspect()` |
| Сложная логика | Chrome DevTools с точками останова |
| Асинхронный код | Async stack traces в DevTools |
| Производительность | `--inspect` + Performance вкладка |
| Память | Heap snapshots + Comparison |
| Production | Модуль `debug` + структурированное логирование |
| CI/CD | `node --inspect --inspect-brk` + ожидание подключения |

```javascript
// Утилита для форматированного вывода объектов
const util = require('util');

console.log(util.inspect(complexObject, {
  showHidden: false,
  depth: null,
  colors: true,
  maxArrayLength: 100,
  breakLength: 80
}));
```

**Золотые правила**:

1. **Не используйте `console.log()` в production** без уровня логирования
2. **Всегда обрабатывайте `unhandledRejection`**
3. **Включите `--inspect` только в development** — в production это риск безопасности
4. **Используйте `debugger`** для явных точек останова в коде
5. **Для долгих сессий** используйте `nodemon --inspect` для автоматического перезапуска
6. **Храните отладочные флаги** в переменных окружения (`DEBUG=*`)
7. **Не коммитьте `debugger`** в production код (используйте линтеры)



**29. Обработка ошибок в Node.js (Try/Catch, события процесса, глобальные обработчики)**

Обработка ошибок — механизм перехвата и реагирования на исключительные ситуации без падения всего приложения. В Node.js ошибки бывают синхронные, асинхронные (колбэки, промисы, события) и системные (ошибки потоков, памяти).

**Синхронные ошибки (try/catch)**:

```javascript
// Простой пример
function parseJSON(jsonString) {
  try {
    const data = JSON.parse(jsonString);
    return { success: true, data };
  } catch (err) {
    return { success: false, error: err.message };
  }
}

console.log(parseJSON('{"name": "Alice"}')); // { success: true, data: ... }
console.log(parseJSON('invalid json'));      // { success: false, error: 'Unexpected token i' }

// Try/catch с finally
function processFile(filePath) {
  let file = null;
  try {
    file = fs.openSync(filePath, 'r');
    const data = fs.readFileSync(file, 'utf8');
    return data;
  } catch (err) {
    console.error(`Ошибка: ${err.message}`);
    throw err; // проброс дальше
  } finally {
    if (file) fs.closeSync(file);
    console.log('Ресурсы освобождены');
  }
}

// Вложенные try/catch
function complexOperation() {
  try {
    try {
      riskyOperation();
    } catch (err) {
      if (err.code === 'SPECIFIC_ERROR') {
        // Обрабатываем специфичную ошибку
        return fallbackValue();
      }
      throw err; // Пробрасываем остальные
    }
  } catch (err) {
    console.error('Общая ошибка:', err);
    return defaultResult();
  }
}
```

**Асинхронные ошибки (колбэки)**:

```javascript
const fs = require('fs');

// ❌ Неправильно: try/catch не работает с асинхронным кодом
try {
  fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err; // Не перехватится в try/catch!
    console.log(data);
  });
} catch (err) {
  console.log('Эта строка никогда не выполнится');
}

// ✅ Правильно: проверка ошибки в колбэке
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Ошибка чтения:', err.message);
    return;
  }
  console.log(data);
});

// Паттерн с ранним возвратом
function readConfig(callback) {
  fs.readFile('config.json', 'utf8', (err, data) => {
    if (err) return callback(err);
    
    try {
      const config = JSON.parse(data);
      callback(null, config);
    } catch (parseErr) {
      callback(parseErr);
    }
  });
}
```

**Асинхронные ошибки (Promises и async/await)**:

```javascript
const fs = require('fs').promises;

// Promise .catch()
fs.readFile('file.txt', 'utf8')
  .then(data => console.log(data))
  .catch(err => console.error('Ошибка:', err.message));

// async/await + try/catch (рекомендуется)
async function readFileAsync() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Ошибка:', err.message);
  }
}

// Обработка конкретных типов ошибок
async function handleSpecificErrors() {
  try {
    const data = await fs.readFile('missing.txt', 'utf8');
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.log('Файл не найден, создаю новый');
      await fs.writeFile('missing.txt', 'default content');
    } else if (err.code === 'EACCES') {
      console.error('Нет прав доступа');
    } else {
      throw err; // Неизвестная ошибка — пробрасываем
    }
  }
}

// Promise.all ошибки
async function parallelOperations() {
  try {
    const results = await Promise.all([
      fetchUser(),
      fetchPosts(),
      fetchComments()
    ]);
    console.log('Все успешно:', results);
  } catch (err) {
    // Если любая операция упадёт — попадём сюда
    console.error('Одна из операций не удалась:', err);
  }
}

// Promise.allSettled — обрабатывает все результаты независимо
async function handleAllResults() {
  const results = await Promise.allSettled([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  
  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`Операция ${index} успешна:`, result.value);
    } else {
      console.log(`Операция ${index} не удалась:`, result.reason);
    }
  });
}
```

**EventEmitter ошибки (EventEmitter)**:

```javascript
const EventEmitter = require('events');
const fs = require('fs');

// ❌ Неправильно: ошибка без обработчика убьёт процесс
const stream = fs.createReadStream('missing.txt');
// stream.on('error', (err) => console.error(err)); // Забыли обработчик
// Процесс упадёт с 'error' event

// ✅ Правильно: всегда обрабатывайте error события
const safeStream = fs.createReadStream('missing.txt');
safeStream.on('error', (err) => {
  console.error('Stream error:', err.message);
  // Можем восстановиться или логировать
});

// Кастомный EventEmitter с обработкой ошибок
class DatabaseConnection extends EventEmitter {
  connect() {
    // Симуляция асинхронного подключения
    setTimeout(() => {
      const isError = Math.random() > 0.5;
      if (isError) {
        this.emit('error', new Error('Connection failed'));
      } else {
        this.emit('connected', { host: 'localhost', port: 5432 });
      }
    }, 100);
  }
}

const db = new DatabaseConnection();
db.on('error', (err) => {
  console.error('DB error:', err);
  // Попытка переподключения
  setTimeout(() => db.connect(), 1000);
});
db.on('connected', (config) => console.log('Connected to', config));
db.connect();
```

**Глобальные обработчики ошибок**:

```javascript
// 1. Uncaught Exception (синхронные ошибки без try/catch)
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION:', err);
  console.error('Stack trace:', err.stack);
  
  // Логирование в файл
  fs.appendFileSync('crash.log', `${new Date()}\n${err.stack}\n`);
  
  // ❌ Не делайте: process.exit(1) — убивает процесс
  // ✅ Лучше: попытаться gracefully завершить
  // server.close(() => process.exit(1));
  
  // В идеале: перезапустить процесс через PM2/forever
  process.exit(1); // Всё равно придётся выйти, состояние нестабильно
});

// 2. Unhandled Rejection (Promise ошибки без .catch)
process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION:', reason);
  console.error('Promise:', promise);
  
  // Логирование
  logger.error({ reason, promise }, 'Unhandled rejection');
  
  // В продакшене: можно продолжить, но лучше перезапустить
  // process.exit(1);
});

// 3. Warning (предупреждения Node.js)
process.on('warning', (warning) => {
  console.warn('WARNING:', warning.name, warning.message);
  console.warn(warning.stack);
});

// 4. Multiple Resolves (Promise резолвлен несколько раз)
process.on('multipleResolves', (type, promise, reason) => {
  console.error('MULTIPLE RESOLVES:', type, promise, reason);
});
```

**Практический пример: глобальный обработчик**:

```javascript
// error-handler.js
const fs = require('fs');

class GlobalErrorHandler {
  constructor(options = {}) {
    this.exitOnUncaught = options.exitOnUncaught !== false;
    this.logFile = options.logFile || 'errors.log';
    this.enableConsole = options.enableConsole !== false;
  }
  
  setup() {
    process.on('uncaughtException', this.handleUncaughtException.bind(this));
    process.on('unhandledRejection', this.handleUnhandledRejection.bind(this));
    process.on('warning', this.handleWarning.bind(this));
    
    // Graceful shutdown
    process.on('SIGTERM', () => this.gracefulShutdown('SIGTERM'));
    process.on('SIGINT', () => this.gracefulShutdown('SIGINT'));
  }
  
  handleUncaughtException(err) {
    const errorInfo = {
      type: 'uncaughtException',
      message: err.message,
      stack: err.stack,
      timestamp: new Date().toISOString()
    };
    
    this.logError(errorInfo);
    
    if (this.exitOnUncaught) {
      this.gracefulShutdown('uncaughtException', 1);
    }
  }
  
  handleUnhandledRejection(reason, promise) {
    const errorInfo = {
      type: 'unhandledRejection',
      reason: reason?.message || String(reason),
      stack: reason?.stack,
      timestamp: new Date().toISOString()
    };
    
    this.logError(errorInfo);
    
    // Promise rejections менее критичны
    if (this.exitOnUncaught && reason?.fatal) {
      this.gracefulShutdown('unhandledRejection', 1);
    }
  }
  
  handleWarning(warning) {
    const errorInfo = {
      type: 'warning',
      name: warning.name,
      message: warning.message,
      stack: warning.stack,
      timestamp: new Date().toISOString()
    };
    
    this.logError(errorInfo);
  }
  
  logError(errorInfo) {
    const logEntry = JSON.stringify(errorInfo) + '\n';
    
    // Запись в файл
    fs.appendFile(this.logFile, logEntry, (err) => {
      if (err) console.error('Failed to write to log file:', err);
    });
    
    // Вывод в консоль
    if (this.enableConsole) {
      console.error('\n========== ERROR ==========');
      console.error(errorInfo.type.toUpperCase());
      console.error(errorInfo.message || errorInfo.reason);
      console.error(errorInfo.stack);
      console.error('===========================\n');
    }
  }
  
  gracefulShutdown(signal, exitCode = 0) {
    console.log(`Received ${signal}, shutting down gracefully...`);
    
    // Закрыть сервер, закрыть соединения с БД
    // closeDatabase();
    // closeServer();
    
    setTimeout(() => {
      console.log('Forced exit');
      process.exit(exitCode);
    }, 10000);
    
    process.exit(exitCode);
  }
}

module.exports = GlobalErrorHandler;

// app.js — подключение в самом начале
const GlobalErrorHandler = require('./error-handler');
const errorHandler = new GlobalErrorHandler({ exitOnUncaught: false });
errorHandler.setup();

// Остальной код приложения...
```

**Обработка ошибок в Express**:

```javascript
const express = require('express');
const app = express();

// Синхронные ошибки — Express перехватывает автоматически
app.get('/sync-error', (req, res) => {
  throw new Error('Sync error'); // Express отправит 500
});

// Асинхронные ошибки — нужно передавать в next()
app.get('/async-error', async (req, res, next) => {
  try {
    const data = await riskyOperation();
    res.json(data);
  } catch (err) {
    next(err); // Передаём в обработчик ошибок
  }
});

// Обёртка для async маршрутов
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/safe-route', asyncHandler(async (req, res) => {
  const data = await riskyOperation();
  res.json(data);
}));

// Кастомные классы ошибок
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

app.get('/users/:id', async (req, res, next) => {
  const user = await db.findUser(req.params.id);
  if (!user) {
    return next(new AppError('User not found', 404));
  }
  res.json(user);
});

// 404 обработчик
app.use((req, res, next) => {
  next(new AppError(`Route ${req.url} not found`, 404));
});

// Глобальный обработчик ошибок Express (4 аргумента!)
app.use((err, req, res, next) => {
  const status = err.statusCode || err.status || 500;
  const message = process.env.NODE_ENV === 'production' && status === 500
    ? 'Internal Server Error'
    : err.message;
  
  console.error(`[ERROR] ${status}: ${err.message}`);
  console.error(err.stack);
  
  res.status(status).json({
    error: message,
    ...(process.env.NODE_ENV !== 'production' && { stack: err.stack })
  });
});
```

**Создание кастомных классов ошибок**:

```javascript
// Ошибка валидации
class ValidationError extends Error {
  constructor(errors) {
    super('Validation failed');
    this.name = 'ValidationError';
    this.statusCode = 400;
    this.errors = errors;
  }
}

// Ошибка аутентификации
class AuthenticationError extends Error {
  constructor(message = 'Authentication required') {
    super(message);
    this.name = 'AuthenticationError';
    this.statusCode = 401;
  }
}

// Ошибка доступа
class AuthorizationError extends Error {
  constructor(message = 'Insufficient permissions') {
    super(message);
    this.name = 'AuthorizationError';
    this.statusCode = 403;
  }
}

// Ошибка базы данных
class DatabaseError extends Error {
  constructor(originalError, operation) {
    super(`Database error during ${operation}: ${originalError.message}`);
    this.name = 'DatabaseError';
    this.originalError = originalError;
    this.operation = operation;
  }
}

// Использование
app.post('/api/users', (req, res, next) => {
  const { name, email } = req.body;
  const errors = [];
  
  if (!name) errors.push('Name is required');
  if (!email) errors.push('Email is required');
  
  if (errors.length > 0) {
    return next(new ValidationError(errors));
  }
  
  next();
});

app.use((err, req, res, next) => {
  if (err instanceof ValidationError) {
    return res.status(400).json({ errors: err.errors });
  }
  if (err instanceof AuthenticationError) {
    return res.status(401).json({ error: err.message });
  }
  if (err instanceof AuthorizationError) {
    return res.status(403).json({ error: err.message });
  }
  next(err);
});
```

**Обработка ошибок в потоках (Streams)**:

```javascript
const fs = require('fs');
const readStream = fs.createReadStream('file.txt');
const writeStream = fs.createWriteStream('output.txt');

// Обработка ошибок в потоках
readStream.on('error', (err) => {
  console.error('Read stream error:', err);
  writeStream.end(); // Закрываем write stream
});

writeStream.on('error', (err) => {
  console.error('Write stream error:', err);
  readStream.destroy(); // Уничтожаем read stream
});

// Использование pipeline (рекомендуется)
const { pipeline } = require('stream/promises');

async function copyFileWithPipeline() {
  try {
    await pipeline(
      fs.createReadStream('source.txt'),
      fs.createWriteStream('destination.txt')
    );
    console.log('Copy successful');
  } catch (err) {
    console.error('Pipeline failed:', err);
  }
}
```

**Логирование ошибок**:

```javascript
// Winston пример
const winston = require('winston');

const logger = winston.createLogger({
  level: 'error',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Использование
try {
  riskyOperation();
} catch (err) {
  logger.error({
    message: err.message,
    stack: err.stack,
    context: { userId: 123, operation: 'payment' }
  });
}
```

**Рекомендации по обработке ошибок**:

| Тип операции | Стратегия |
|-------------|-----------|
| Синхронный код | try/catch |
| Колбэки | Проверка err первым аргументом |
| Promises | .catch() или try/catch с await |
| EventEmitter | Слушатель 'error' |
| Express маршруты | next(err) |
| Критические ошибки | process.exit(1) + перезапуск |
| Неожиданные ошибки | Логировать и восстановиться |
| Ожидаемые ошибки | Обработать локально (404, валидация) |

**Золотые правила**:

1. **Всегда обрабатывайте ошибки** — никогда не оставляйте их неперехваченными
2. **Логируйте ошибки** с достаточным контекстом
3. **Не используйте throw в асинхронном коде** без try/catch
4. **Для Express всегда вызывайте next(err)**
5. **В production не показывайте пользователю стек ошибок**
6. **Различайте ожидаемые и неожиданные ошибки**
7. **Устанавливайте глобальные обработчики** (`uncaughtException`, `unhandledRejection`)
8. **Используйте кастомные классы ошибок** для разных типов ситуаций
9. **Не игнорируйте ошибки** пустыми catch блоками
10. **В критических системах** реализуйте graceful shutdown и автоматический перезапуск



**30. Следующие шаги: базы данных (MongoDB/PostgreSQL), аутентификация и деплой**

Вы прошли основы Node.js. Дальнейший путь — подключение баз данных, защита приложений аутентификацией и развёртывание в продакшен. Ниже — дорожная карта с конкретными инструментами и примерами.

## 1. БАЗЫ ДАННЫХ

### MongoDB (NoSQL) с Mongoose

```bash
npm install mongoose
```

```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  password: { type: String, required: true },
  age: { type: Number, min: 0, max: 150 },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  createdAt: { type: Date, default: Date.now }
});

userSchema.methods.toJSON = function() {
  const user = this.toObject();
  delete user.password;
  return user;
};

module.exports = mongoose.model('User', userSchema);

// connection.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });
    console.log('MongoDB connected');
  } catch (err) {
    console.error('MongoDB connection error:', err);
    process.exit(1);
  }
};

// CRUD операции
const User = require('./models/User');

// Create
const user = await User.create({ name: 'Alice', email: 'alice@example.com', password: 'hashed' });

// Read
const users = await User.find({ role: 'user' }).limit(10).sort('-createdAt');
const singleUser = await User.findById(userId);
const userByEmail = await User.findOne({ email: 'alice@example.com' });

// Update
await User.findByIdAndUpdate(userId, { age: 31 }, { new: true });

// Delete
await User.findByIdAndDelete(userId);

// Агрегации
const stats = await User.aggregate([
  { $group: { _id: '$role', count: { $sum: 1 } } }
]);
```

### PostgreSQL (SQL) с Prisma (рекомендуется) или Knex

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  password  String
  age       Int?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
}
```

```bash
npx prisma migrate dev --name init
npx prisma generate
```

```javascript
// db.js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// CRUD операции
// Create
const user = await prisma.user.create({
  data: { email: 'alice@example.com', name: 'Alice', password: 'hashed' }
});

// Read
const users = await prisma.user.findMany({
  where: { age: { gte: 18 } },
  include: { posts: true },
  take: 10,
  orderBy: { createdAt: 'desc' }
});

const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true }
});

// Update
const updated = await prisma.user.update({
  where: { id: userId },
  data: { age: 31 }
});

// Delete
await prisma.user.delete({ where: { id: userId } });

// Транзакции
const result = await prisma.$transaction([
  prisma.user.create({ data: { email: 'new@example.com', name: 'New' } }),
  prisma.user.update({ where: { id: 1 }, data: { age: 30 } })
]);
```

## 2. АУТЕНТИФИКАЦИЯ

### JWT (JSON Web Tokens) + bcrypt

```bash
npm install jsonwebtoken bcrypt
```

```javascript
// auth/utils.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const SALT_ROUNDS = 10;
const JWT_SECRET = process.env.JWT_SECRET;
const JWT_EXPIRES_IN = '7d';

// Хэширование пароля
const hashPassword = async (password) => {
  return await bcrypt.hash(password, SALT_ROUNDS);
};

// Сравнение паролей
const comparePassword = async (password, hash) => {
  return await bcrypt.compare(password, hash);
};

// Генерация токена
const generateToken = (payload) => {
  return jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
};

// Верификация токена
const verifyToken = (token) => {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (err) {
    return null;
  }
};

// middleware/auth.js
const authenticate = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  const token = authHeader.split(' ')[1];
  const decoded = verifyToken(token);
  
  if (!decoded) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
  
  req.user = decoded;
  next();
};

// auth/routes.js
const express = require('express');
const router = express.Router();

// Регистрация
router.post('/register', async (req, res, next) => {
  try {
    const { email, password, name } = req.body;
    
    // Проверка существования пользователя
    const existingUser = await prisma.user.findUnique({ where: { email } });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already registered' });
    }
    
    const hashedPassword = await hashPassword(password);
    const user = await prisma.user.create({
      data: { email, name, password: hashedPassword }
    });
    
    const token = generateToken({ id: user.id, email: user.email });
    
    res.status(201).json({
      user: { id: user.id, email: user.email, name: user.name },
      token
    });
  } catch (err) {
    next(err);
  }
});

// Логин
router.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;
    
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const isValid = await comparePassword(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const token = generateToken({ id: user.id, email: user.email });
    
    res.json({
      user: { id: user.id, email: user.email, name: user.name },
      token
    });
  } catch (err) {
    next(err);
  }
});

// Защищённый маршрут
router.get('/me', authenticate, async (req, res, next) => {
  try {
    const user = await prisma.user.findUnique({
      where: { id: req.user.id },
      select: { id: true, email: true, name: true, createdAt: true }
    });
    res.json(user);
  } catch (err) {
    next(err);
  }
});

module.exports = router;
```

### OAuth2 (Google, GitHub) с Passport.js

```bash
npm install passport passport-google-oauth20 passport-github2 express-session
```

```javascript
// passport-config.js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20');

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
  let user = await prisma.user.findUnique({ where: { googleId: profile.id } });
  if (!user) {
    user = await prisma.user.create({
      data: { email: profile.emails[0].value, name: profile.displayName, googleId: profile.id }
    });
  }
  done(null, user);
}));

passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser(async (id, done) => {
  const user = await prisma.user.findUnique({ where: { id } });
  done(null, user);
});

// app.js
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
app.get('/auth/google/callback', passport.authenticate('google', { failureRedirect: '/login' }), (req, res) => {
  res.redirect('/dashboard');
});
```

## 3. ДЕПЛОЙ (РАЗВЁРТЫВАНИЕ)

### Подготовка к деплою

```json
// package.json
{
  "scripts": {
    "start": "node server.js",
    "build": "npm run build:frontend",
    "postinstall": "npm run build",
    "pm2:start": "pm2 start ecosystem.config.js",
    "pm2:stop": "pm2 stop ecosystem.config.js"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}
```

```javascript
// config/production.js
module.exports = {
  port: process.env.PORT || 3000,
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  corsOrigin: process.env.CORS_ORIGIN?.split(',') || [],
  trustProxy: true,
  rateLimit: { windowMs: 15 * 60 * 1000, max: 100 }
};
```

### PM2 (Production process manager)

```bash
npm install -g pm2
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-app',
    script: 'server.js',
    instances: 'max', // по числу ядер CPU
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    max_memory_restart: '1G',
    kill_timeout: 5000,
    listen_timeout: 5000
  }]
};
```

```bash
# Команды PM2
pm2 start ecosystem.config.js
pm2 list
pm2 logs my-app
pm2 monit
pm2 restart my-app
pm2 reload my-app   # zero-downtime reload
pm2 stop my-app
pm2 delete my-app
pm2 save
pm2 startup         # автозапуск при загрузке системы
```

### Деплой на VPS (Ubuntu + Nginx + PM2 + GitHub Actions)

```nginx
# /etc/nginx/sites-available/my-app
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /var/www/my-app
            git pull origin main
            npm ci --production
            npx prisma migrate deploy
            pm2 reload my-app
```

### Контейнеризация (Docker)

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/app
    depends_on:
      - db
    restart: always

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=app
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: always

volumes:
  pgdata:
```

### Облачные платформы

```bash
# Heroku
heroku create my-app
heroku config:set NODE_ENV=production JWT_SECRET=xxx
git push heroku main
heroku logs --tail

# Railway
# Подключить GitHub репозиторий -> автоматический деплой

# Vercel (для API + фронтенд)
vercel --prod

# AWS Elastic Beanstalk
eb init -p node.js my-app
eb create production
eb deploy

# Render
# Подключить GitHub -> авто-деплой из main
```

## 4. ДОПОЛНИТЕЛЬНЫЕ ТЕМЫ ДЛЯ ИЗУЧЕНИЯ

| Тема | Инструменты | Зачем |
|------|-------------|-------|
| GraphQL | Apollo Server, GraphQL Yoga | Гибкие запросы вместо REST |
| WebSocket | Socket.io, ws | Реал-тайм (чаты, уведомления) |
| Кэширование | Redis, Node-cache | Высокая производительность |
| Очереди | Bull, RabbitMQ | Фоновые задачи (email, обработка) |
| Тестирование | Jest, Supertest, Mocha | Надёжность и регрессия |
| TypeScript | TS-Node, TypeORM | Типизация, масштабируемость |
| Monitoring | PM2, Prometheus, Grafana | Отслеживание состояния |
| Логирование | Winston, Pino | Централизованные логи |
| Rate Limiting | express-rate-limit, Redis | Защита от DDoS |

## 5. КНИГИ И РЕСУРСЫ

- **Node.js Design Patterns** — Mario Casciaro
- **Node.js in Practice** — Alex Young
- **Официальная документация** — nodejs.dev
- **Express документация** — expressjs.com
- **Prisma документация** — prisma.io
- **PM2 документация** — pm2.keymetrics.io

## 6. ПРАКТИЧЕСКИЙ ПРОЕКТ ДЛЯ ЗАКРЕПЛЕНИЯ

```bash
# Полноценный проект для портфолио
real-world-api/
├── src/
│   ├── config/          # конфигурация (env, db, redis)
│   ├── models/          # Prisma/Mongoose модели
│   ├── controllers/     # бизнес-логика
│   ├── routes/          # маршруты Express
│   ├── middleware/      # auth, rate-limit, validation
│   ├── services/        # email, upload, queue
│   ├── utils/           # helpers, logger
│   └── app.js
├── tests/               # unit + integration
├── prisma/              # схема БД
├── docker-compose.yml
├── ecosystem.config.js
└── .github/workflows/   # CI/CD
```

**Функционал проекта**:
- JWT аутентификация + refresh токены
- Ролевая модель (user/admin)
- CRUD для ресурсов с пагинацией/фильтрацией
- Загрузка файлов (multer)
- Фоновые задачи (отправка email через Bull + Redis)
- Кэширование (Redis)
- Swagger документация
- Rate limiting и защита
- Unit + интеграционные тесты
- Docker + docker-compose
- CI/CD (GitHub Actions)
- Деплой на VPS с Nginx + PM2

Этот проект закроет 90% задач реальной коммерческой разработки на Node.js. После его реализации вы станете уверенным mid-level разработчиком.
