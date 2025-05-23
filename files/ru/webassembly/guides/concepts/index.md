---
title: Основы WebAssembly
slug: WebAssembly/Guides/Concepts
---

{{WebAssemblySidebar}}

В этой статье объясняются концепции, лежащие в основе работы технологии WebAssembly, включая её цели, проблемы, которые она решает, и то, как она работает в движке рендеринга веб-браузера.

## Что такое WebAssembly?

WebAssembly - это технология, предоставляющая новый тип кода, который можно запускать в современных веб-браузерах, что обеспечивает новые функции и значительное повышение производительности. Код для WebAssembly не предназначен для написания вручную, скорее он спроектирован для эффективной компиляции из низкоуровневых исходных языков, таких как C, C++, Rust и.т.д.

WebAssembly позволяет запускать код, написанный на различных языках в web-приложениях почти c естественной скоростью. Это имеет огромное значение для веб-платформы, так как ранее это нельзя было сделать.

Более того, вам даже не нужно знать, как создавать код WebAssembly, чтобы его использовать. Модули WebAssembly можно импортировать в веб-приложение (или Node.js), и экспортировать из них функции для использования через JavaScript. JavaScript-фреймворки могут использовать модули WebAssembly для получения огромных преимуществ в производительности и новых функций, в то же время делая их функциональность легко доступной для веб-разработчиков.

## Цели WebAssembly

Технология WebAssembly создаётся как открытый стандарт внутри [W3C WebAssembly Community Group](https://www.w3.org/community/webassembly/) со следующими целями:

- Быть быстрым, эффективным и переносимым - код WebAssembly может выполняться практически на естественной скорости на разных платформах, используя преимущества [аппаратных возможностей](http://webassembly.org/docs/portability/#assumptions-for-efficient-execution).
- Быть читаемым и отлаживаемым - WebAssembly - это низкоуровневый ассемблерный язык, но он имеет читабельный текстовый формат (спецификация для которого ещё дорабатывается), который позволяет писать, просматривать и отлаживать код вручную.
- Поддерживать безопасность - код WebAssembly предназначен для запуска в безопасной, изолированной среде выполнения. Как и другой веб-код, он будет соблюдать политики безопасности браузера.
- Не разрушать текущий веб - технология WebAssembly разработана так, что она прекрасно сочетается с другими веб-технологиями и поддерживает обратную совместимость.

> [!NOTE]
> Технология WebAssembly также будет иметь возможность использования за пределами веб и JavaScript-сред (см. [Встраивание вне Web](http://webassembly.org/docs/non-web/)).

## Как WebAssembly встраивается в веб-платформу?

Веб-платформа может рассматриваться как состоящая из двух частей:

- Виртуальная машина (ВМ), на которой выполняется код веб-приложения, например код JavaScript, который обеспечивает работу ваших приложений.
- Набор [Web API](/ru/docs/Web/API), которые веб-приложение может вызывать для управления функциональными возможностями веб-браузера / устройства и выполнения задач ([DOM](/ru/docs/Web/API/Document_Object_Model), [CSSOM](/ru/docs/Web/API/CSS_Object_Model), [WebGL](/ru/docs/Web/API/WebGL_API), [IndexedDB](/ru/docs/Web/API/IndexedDB_API), [Web Audio API](/ru/docs/Web/API/Web_Audio_API), и т.д.)

Исторически ВМ могла загружать только JavaScript. Раньше нас это вполне устраивало, поскольку JavaScript достаточно мощный, чтобы решать большинство проблем, с которыми мы сталкивались в интернете. Однако мы столкнулись с проблемами производительности, когда попытались использовать JavaScript для более нагруженных сценариев использования, таких как 3D-игры, виртуальная и дополненная реальность, компьютерное зрение, редактирование изображений / видео и в ряде других применений, которые требуют повышенной производительности (см. [варианты использования WebAssembly](http://webassembly.org/docs/use-cases/) где описано больше идей).

Кроме того, длительность загрузки, анализа и компиляции очень больших приложений JavaScript может быть непомерно высокой. Мобильные и другие ограниченные в ресурсах платформы могут ещё более понизить производительность.

Язык WebAssembly отличается от языка JavaScript, но он не предназначен для его замены. Он предназначен для дополнения и работы вместе с JavaScript, что позволяет веб-разработчикам использовать преимущества обоих языков:

- JavaScript - это язык высокого уровня, гибкий и достаточно выразительный для написания веб-приложений. Он имеет много преимуществ - он динамически типизирован, не требует этапа компиляции и обладает огромной экосистемой, которая предоставляет мощные фреймворки, библиотеки и другие инструменты.
- WebAssembly - это низкоуровневый язык, похожий на ассемблер, с компактным двоичным форматом, который работает с почти естественной производительностью и предоставляет низкоуровневые модели памяти для таких языков как C++ и Rust, с целью компиляции, чтобы они могли работать в интернете. (Обратите внимание, что у WebAssembly на будущее есть [высокоуровневая цель](http://webassembly.org/docs/high-level-goals/) по поддержке языков со сборщиками мусора в модели памяти).

С появлением WebAssembly в браузерах виртуальная машина, о которой мы говорили ранее, теперь будет загружать и запускать два типа кода - JavaScript и WebAssembly.

Различные типы кода могут вызывать друг друга по мере необходимости - [WebAssembly JavaScript API](/ru/docs/WebAssembly/Reference/JavaScript_interface) оборачивает экспортированный код WebAssembly в функции JavaScript, которые можно вызывать обычным способом. А в код WebAssembly можно импортировать и синхронно вызывать обычные функции JavaScript. Фактически, базовая единица кода WebAssembly называется модулем, а модули WebAssembly во многом схожи с модулями ES2015.

### Ключевые понятия WebAssembly

Есть несколько ключевых понятий, необходимых для понимания того, как WebAssembly работает в браузере. Все эти понятия отражены 1:1 в [WebAssembly JavaScript API](/ru/docs/WebAssembly/Reference/JavaScript_interface).

- **Модуль**: Представляет двоичный файл WebAssembly, который был скомпилирован браузером в исполняемый машинный код. Модуль не имеет состояния и, таким образом, как [Blob](/ru/docs/Web/API/Blob), может быть явным образом разделён между windows и workers (через [`postMessage()`](/ru/docs/Web/API/MessagePort/postMessage)). В модуле есть объявление импорта и экспорта точно такое же, как и в модуле ES2015.
- **Память**: Массив ArrayBuffer изменяемого размера, который содержит линейный массив байтов, считываемых и записываемых инструкциями низкоуровневого доступа к памяти в WebAssembly.
- **Таблица**: Типизированный массив ссылок изменяемого размера (например, для функций), которые не могут быть размещены в памяти в виде байтов (по соображениям безопасности и переносимости).
- **Экземпляр**: Модуль в паре со своим состоянием, которое он использует во время выполнения, включая память, таблицу и набор импортируемых значений. Экземпляр модуля похож на модуль ES2015, который был загружен в определённую глобальную переменную с определённым набором импортов.

JavaScript API предоставляет разработчикам возможность создавать модули, объекты памяти, таблицы и экземпляры модулей. Получив экземпляр модуля WebAssembly, код JavaScript может синхронно вызывать его экспорты, которые представляются как обычные функции JavaScript. Любые функции JavaScript также могут синхронно вызываться кодом WebAssembly путём передачи этих функций в качестве импорта в экземпляр модуля WebAssembly.

Поскольку JavaScript полностью контролирует загрузку, компиляцию и запуск кода WebAssembly, разработчики JavaScript могут рассматривать технологию WebAssembly как расширение JavaScript для эффективной генерации высокопроизводительных функций.

В будущем модули WebAssembly будут загружаться так же, как и [модули ES2015](https://github.com/WebAssembly/design/issues/1087) (с использованием `<script type='module'>`), что означает, что JavaScript сможет извлекать, компилировать и импортировать модуль WebAssembly так же легко, как модуль ES2015.

## Как я могу использовать WebAssembly в своём приложении?

Выше мы говорили о низкоуровневых примитивах, которые WebAssembly добавляет к веб-платформе: двоичный формат для кода и API для его загрузки и запуска. Теперь давайте поговорим о том, как мы можем использовать эти примитивы на практике.

Экосистема WebAssembly находится на начальной стадии; больше инструментов, несомненно, появится в будущем. На данный момент есть три основные отправные точки:

- Портирование приложения C/C++ с [Emscripten](https://emscripten.org/).
- Написание или генерация WebAssembly кода непосредственно на уровне сборки.
- Написание приложения Rust и выбор WebAssembly в качестве типа вывода при построении.

Давайте поговорим об этих вариантах:

### Портирование из C/C++

Из множества вариантов создания кода WASM есть два наиболее популярных - это онлайн-сборщик wasm или [Emscripten](https://emscripten.org/). Существует ещё несколько вариантов сборки WASM, таких как:

- [WasmFiddle](https://wasdk.github.io/WasmFiddle/)
- [WasmFiddle++](https://anonyco.github.io/WasmFiddle/)
- [WasmExplorer](https://mbebenita.github.io/WasmExplorer/)

Это отличные ресурсы для людей, которые хотят начать, но у них нет инструментов и возможностей Emscripten.

Инструмент Emscripten способен взять практически любой исходный код C/C++ и скомпилировать его в модуль .wasm, добавив необходимый "связующий" JavaScript-код для загрузки и запуска модуля в HTML-документе и отображения результатов работы.

![](emscripten-diagram.png)

В двух словах, процесс работает следующим образом:

1. Сначала Emscripten передаёт код C/C++ в clang + LLVM - набор инструментов компилятора C/C++ с открытым исходным кодом, который поставляется, например, как часть XCode для OSX.
2. Emscripten преобразует скомпилированный результат clang + LLVM в двоичный файл .wasm.
3. Сам по себе код WebAssembly в настоящее время не может напрямую обращаться к DOM; он может вызывать только JavaScript, передавая целочисленные и числа с плавающей точкой примитивные типы данных. Таким образом, чтобы получить доступ к любому Web-API, WebAssembly модуль должен обращаться к JavaScript, который затем вызывает Web-API. Поэтому Emscripten создаёт необходимый для этого связующий код HTML и JavaScript.

> [!NOTE]
> В будущем планируется [позволить WebAssembly напрямую вызывать Web API](https://github.com/WebAssembly/gc/blob/master/README.md).

Связующий код JavaScript не так прост, как вы можете себе представить. Прежде всего Emscripten предоставляет популярные библиотеки C/C++, такие как [SDL](https://en.wikipedia.org/wiki/Simple_DirectMedia_Layer), [OpenGL](https://en.wikipedia.org/wiki/OpenGL), [OpenAL](https://en.wikipedia.org/wiki/OpenAL) и части [POSIX](https://en.wikipedia.org/wiki/POSIX). Эти библиотеки реализованы с точки зрения Web-API, и поэтому каждой из них требуется некоторый код JavaScript для соединения WebAssembly модуля с базовым Web-API.

Таким образом, часть связующего кода предоставляет функциональность каждой библиотеки, используемой кодом C/C++. Связующий код также содержит логику для вызова вышеупомянутых WebAssembly JavaScript API для извлечения, загрузки и запуска файла .wasm.

Сгенерированный HTML-документ загружает связующий файл JavaScript и может записать stdout в {{htmlelement("textarea")}}. Если приложение использует OpenGL, HTML документ будет содержать элемент {{htmlelement("canvas")}}, который будет использоваться для рендеринга. Очень легко изменить тип вывода компиляции Emscripten на любое веб-приложение, которое вам нужно.

Вы можете найти полную документацию по Emscripten на [emscripten.org](http://emscripten.org), а также руководство по созданию набора инструментов и компиляции вашего собственного приложения на C/C++ в wasm в разделе [Компилирование из C/C++ в WebAssembly](/ru/docs/WebAssembly/Guides/C_to_Wasm).

### Написание WebAssembly напрямую

Хотите создать свой собственный компилятор, собственные инструменты или библиотеку JavaScript, которая генерирует WebAssembly во время выполнения?

Аналогично другим ассемблерным языкам, двоичный формат WebAssembly имеет текстовое представление с которым имеет соответствие 1:1. Вы можете написать или сгенерировать этот формат вручную, а затем преобразовать его в двоичный формат с помощью любого из [инструментов преобразования текста в двоичные файлы WebAssembly](http://webassembly.org/getting-started/advanced-tools/).

Простое руководство о том, как это сделать, смотрите в нашей статье [Перевод из текстового формата WebAssembly в wasm](/ru/docs/WebAssembly/Guides/Text_format_to_Wasm).

### Написание приложения Rust с компиляцией в WebAssembly

Благодаря неустанной работе рабочей группы Rust WebAssembly, стало возможно писать Rust код и компилировать его в WebAssembly. В нашей статье [Компиляция из Rust в WebAssembly](/ru/docs/WebAssembly/Guides/Rust_to_Wasm) описан процесс установки необходимого набора инструментов, компиляция примера программы Rust в пакет npm для WebAssembly с целью использования его в качестве примера веб-приложения.

## Резюме

Эта статья дала вам объяснение того, что такое технология WebAssembly, чем она полезная, как она вписывается в веб и как вы можете её использовать.

## Смотрите также

- [Статьи WebAssembly в блоге Mozilla Hacks](https://hacks.mozilla.org/category/webassembly/)
- [WebAssembly на Mozilla Research](https://research.mozilla.org/webassembly/)
- [Загрузка и запуск кода WebAssembly](/ru/docs/WebAssembly/Guides/Loading_and_running) — узнайте, как загрузить свой собственный модуль WebAssembly на веб-страницу.
- [Использование WebAssembly JavaScript API](/ru/docs/WebAssembly/Guides/Using_the_JavaScript_API) — узнайте, как использовать другие основные функции WebAssembly JavaScript API.
