# OmarBots — скрипты

Пиши свои фичи на **JavaScript** (всегда) или **Java** (если клиент запущен на JDK).
Скрипты лежат в папке `omarobots/scripts/`. Сохранил файл — он загрузился сам; сохранил ещё раз — перезагрузился; удалил — выгрузился. Ошибка в скрипте показывается уведомлением и **не роняет клиент**.

---

## Команды

| Команда | Что делает |
|---|---|
| `.script list` | загруженные скрипты |
| `.script reload [имя]` | перезагрузить один или все |
| `.script load` | подхватить новые файлы |
| `.script unload <имя>` | выгрузить |
| `.script new <имя> js\|java` | создать файл-заготовку |
| `.script eval <код>` | выполнить JS один раз |
| `.script folder` | открыть папку со скриптами |

---

## Быстрый старт

Создай `omarobots/scripts/test.js`:

```js
log("привет");

commands.add("hi", "поздороваться", args => chat.print("хай " + args));

let m = modules.create("AutoHi", "Misc", "пишет хай");
let delay = m.number("Задержка сек", 5, 1, 60, 1);
let last = 0;
m.onTick(() => {
    if (Date.now() - last >= delay.getValue() * 1000) {
        last = Date.now();
        chat.print("хай");
    }
});
```

Сохрани. Модуль `AutoHi` появится в ClickGUI → Misc, команда `.hi` заработает.

---

## Глобалы (JS)

Объекты: `captcha`, `modules`, `commands`, `events`, `bots`, `http`, `chat`, `storage`, `render`, `api`.
Функции: `log(x)`, `notify(x)`, `after(ms, fn)`, `every(ms, fn)`, `onUnload(fn)`, `console.log(x)`.

В Java-скрипте то же самое через `api`: `api.captcha()`, `api.modules()`, `api.chat()` и т.д.

---

## captcha — свои решалки капчи

Решалка сразу появляется в модуле **Bots → Captcha Solver → Solver mode** под своим именем и участвует в режиме Auto.

```js
captcha.solver("MySolver", input => {
    // input.pngBase64(), input.width(), input.height(), input.botName()
    let r = http.postJson("http://localhost:8000/solve", { image: input.pngBase64() });
    if (!r.ok) return;              // ничего не вернул = не решил, попробуется следующая
    return r.json().text;           // текст, который бот напишет
});

captcha.acceptWords("верно", "captcha ok");   // слова сервера = приняли
captcha.rejectWords("неверно", "try again");  // = не приняли, повтор
```

Вернуть можно строку, число или объект с полем `text`/`answer`/`code`.

---

## modules — свои модули в ClickGUI

```js
let m = modules.create("Имя", "Combat", "описание");   // категория: Combat/Movement/Player/Render/World/Misc/Bots/Hud/Crashers
let speed = m.number("Скорость", 1.0, 0.1, 5, 0.1);
let on    = m.bool("Флаг", true);
let mode  = m.mode("Режим", "A", "A", "B", "C");
let name  = m.string("Текст", "", 64);
let col   = m.color("Цвет", 0xFF3568FF);
let key   = m.bind("Клавиша", 0);

m.onEnable(() => { ... });
m.onDisable(() => { ... });
m.onTick(() => { ... speed.getValue() ... });
```

Настройки и вкл/выкл **сохраняются** между запусками.

---

## commands — свои команды

```js
commands.add("пинг", "тест", args => chat.print("понг " + args));
```

`args` — текст после имени команды.

---

## events — события

```js
events.on("tick", e => { ... });                               // каждый тик
events.on("render", e => { render.text("hi", 10, 10, 9, 0xFFFFFFFF); });
events.on("chatreceive", e => { if (e.text().includes("спам")) e.cancel(); });  // отмена прячет сообщение
events.on("death", e => notify("умер: " + e.text()));
events.on("packetreceive", e => { log(e.getPacket().getClass().getSimpleName()); e.cancel(); });
events.on("bot", e => log(e.type() + " " + e.bot().username()));
events.on("captcha", e => log(e.type() + " " + e.bot()));
events.on(dev.omarobots.core.event.events.EventKey, e => log(e.getKey()));  // любой класс события
```

**Имена:** `tick, update, render, key, packet, packetsend, packetreceive, chatreceive, death, title, subtitle, actionbar, module, bot, captcha, world, motion`.

Отменяемые (`e.cancel()`): `packet*`, `chatreceive`, `motion`, закрытие инвентаря.
`packetsend` / `packetreceive` ловят почти всё серверное (чат, GUI, звуки, титлы) — это универсальный хук.

**Свои события** (любое имя):

```js
events.on("моёсобытие", data => log(data));
events.emit("моёсобытие", { x: 1 });
```

---

## bots — боты

```js
bots.chatAll("привет");       // все боты в игре пишут
log(bots.count());            // сколько в игре
bots.list();                  // список
bots.manager().connect(5, 200);   // полный BotManager для всего остального
```

---

## http — запросы

Блокирующий, можно звать из решалки капчи (она на отдельном потоке).

```js
let r = http.get("https://example.com");
log(r.status + " " + r.ok);
log(r.text);

let j = http.postJson("http://host/api", { a: 1 }).json();   // .json() → объект
```

У ответа: `.status`, `.text`, `.ok`, `.json()`.

---

## chat — чат

```js
chat.send("привет");     // твой игрок пишет на сервер (или /команда)
chat.print("локально");  // строка только у тебя
chat.notify("попап");    // уведомление
```

---

## storage — хранилище (сохраняется)

```js
let n = storage.getNumber("count", 0) + 1;
storage.set("count", n);
storage.get("key", "по умолчанию");
storage.getBool("flag", false);
```

Хранит строки / числа / булевы.

---

## render — рисование (2D)

Рисовать только внутри `events.on("render", ...)` или в таймере. Цвета `0xAARRGGBB`. Координаты — экранные единицы (экран 800×450 на 16:9).

```js
events.on("render", e => {
    render.roundedRect(6, 6, 120, 26, 5, 0xCC101418);
    render.text("хп " + hp, 14, 15, 9, 0xFFFFFFFF);
    render.rect(6, 40, 100, 4, 0xFF3568FF);
});
```

Методы: `rect, roundedRect, outline, circle, verticalGradient, horizontalGradient, shadow, blur, text, boldText, centeredText, rightText, gradientText, textWidth, scissor, width(), height()`.
Цвет: `render.color(r,g,b[,a])`, `render.fade(цвет, 0..1)`.

---

## таймеры

```js
after(1000, () => chat.print("через секунду"));
every(5000, () => chat.print("каждые 5 сек"));
```

Коллбэки идут на игровом потоке, останавливаются при выгрузке скрипта.

---

## Java-скрипты

Только на JDK. Имя `public class` = имя файла.

```java
import dev.omarobots.script.api.Script;
import dev.omarobots.script.api.ScriptApi;

public class MySolver implements Script {
    public void onLoad(ScriptApi api) {
        api.captcha().solver("MySolver", input -> "1234");
        api.commands().add("пинг", "тест", args -> api.chat().print("понг"));
    }
    public void onUnload() { }
}
```

---

## Чего нельзя / особенности

- **Главное:** вклиниться в чисто клиентскую логику, которая не пакет и не кидает событие (например середина метода рендера). Нет точки → нет события
- Отменить тик/рендер нельзя (рисовать поверх — да).
- JS — это Rhino: **нет `class`**, нет `async/await`, `import`, npm. http блокирующий.
- Бесконечный цикл в JS прибьётся защитой (~это норма).
- Один файл = один скрипт; импорта одного скрипта в другой нет.

---

## Что можно (коротко)

Читать мир / игрока / сущности, слать свои пакеты, двигать ботов, решать капчу, рисовать HUD, свои команды и модули, http, таймеры, хранилище, перехватывать и отменять пакеты и входящий чат, свои события между скриптами.
