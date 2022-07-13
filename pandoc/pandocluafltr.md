# Фільтри Pandoc Lua

[оригінал](https://pandoc.org/lua-filters.html)

# Короткий зміст

Фільтри Lua - це таблиці з іменами елементів у вигляді ключів і значень, що складаються з функцій, що діють на ці елементи. Є два рівнозначні способи їх означення:

```lua
return {
  { Strong = function (elem)
      return pandoc.SmallCaps(elem.c)
    end}
}
```

або еквівалентно,

```lua
function Strong(elem)
  return pandoc.SmallCaps(elem.c)
end
```

Тобто для об'єктів певного типу будуть шукатися функції з відповідними іменами у фіксованому порядку, пропускаючи ті, які відсутні: для [*Inline* elements](https://pandoc.org/lua-filters.html#type-inline), потім для [`Inlines`](https://pandoc.org/lua-filters.html#inlines-filter) фільтрів, потім для [*Block* elements](https://pandoc.org/lua-filters.html#type-block) , потім для  [`Blocks`](https://pandoc.org/lua-filters.html#inlines-filter) філтьтрів, потім для [`Meta`](https://pandoc.org/lua-filters.html#type-meta) фільтрації, потім для фільтрації [`Pandoc`](https://pandoc.org/lua-filters.html#type-pandoc) .

Очікується, що фільтри будуть поміщені в окремі файли та передані через аргумент командного рядка `--lua-filter`. Наприклад, якщо фільтр означений у файлі `current-date.lua`, він буде застосований так:

```bash
pandoc -s 1.md -t 1.docx --lua-filter=current-date.lua
```

Опція `--lua-filter` може поставлятися кілька разів. Pandoc застосовує всі фільтри (включаючи фільтри JSON, визначені через `--filter` та фільтри Lua, вказані через ` --lua-filter`) у порядку, який вони відображаються в командному рядку. Pandoc очікує, що кожен файл Lua поверне список фільтрів. Фільтри у цьому списку визиваються послідовно, кожен за результатами попереднього фільтра. 

Записи фільтру будуть викликані для кожного відповідного елемента документа, отримуючи відповідний елемент як вхідний. Повернення функції фільтра має бути одним із наступних:

- нуль: це означає, що об’єкт повинен залишатися незмінним.
- об'єкт pandoc: він повинен бути того ж типу, що і вхідний, і замінить початковий об'єкт.
- список об’єктів pandoc: вони замінять вихідний об'єкт; список об'єднується з сусідами оригінальних об'єктів (зрощений у список, до якого належить оригінальний об’єкт); повернення порожнього списку видаляє об'єкт.

Результат функції повинен мати результат того ж типу, що і вхід. Це означає, що функція фільтра, що діє на вбудований елемент, повинна повертати або нуль, рядок, або список рядків, а функція, що фільтрує блок-елемент, повинна повернути один нуль, блок або список блокових елементів. Pandoc видасть помилку, якщо ця умова буде порушена.

# Вступ

Pandoc вже давно підтримує фільтри, які дозволяють керувати деревом абстрактних синтаксисів pandoc (AST) між етапом синтаксичного розбору (парсингу) та записування. [Традиційні фільтри pandoc ](https://pandoc.org/filters.html) приймають JSON-представлення pandoc AST і створюють змінене JSON-представлення AST. Вони можуть бути написані будь-якою мовою програмування та викликатись з pandoc за допомогою опції `--filter`.

Хоча традиційні фільтри дуже гнучкі, вони мають кілька недоліків. По-перше, є деякий накладний характер написання JSON для stdout та читання його від stdin (двічі, один раз на кожній стороні фільтра). По-друге, чи буде працювати фільтр, буде залежати від деталей оточення користувача. Фільтр може вимагати наявності інтерпретатора для певної мови програмування, а також бібліотеки для маніпулювання pandoc AST у формі JSON. Не можна просто надати фільтр, який може використовувати кожен, хто має певну версію виконуваного файлу pandoc.

Починаючи з версії 2.0, pandoc дозволяє записувати фільтри в Lua без зовнішніх залежностей взагалі. Інтерпретатор Lua (версія 5.3) та бібліотека Lua для створення фільтрів pandoc вбудовані у виконуваний файл pandoc. Типи даних Pandoc безпосередньо піддаються Lua, уникаючи накладних витрат JSON на stdout та зчитування з stdin.

Ось приклад фільтра Lua, який перетворює напівжирний шрифт (strong emphasis) на зменшені великі літери (small caps):

```lua
return {
  {
    Strong = function (elem)
      return pandoc.SmallCaps(elem.c)
    end,
  }
}
```

або еквівалентно,

```lua
function Strong(elem)
  return pandoc.SmallCaps(elem.c)
end
```

Це говорить: перейдіть по AST, і коли ви знайдете елемент Strong, замініть його елементом SmallCaps з тим самим вмістом.

Щоб запустити його, збережіть його у файлі, скажімо  `smallcaps.lua`, і викличте

```bash
pandoc -s 1.md -o 1.docx --lua-filter=smallcaps.lua
```

Ось швидке порівняння продуктивності, перетворення посібника з pandoc (MANUAL.txt) в HTML, з версії того самого JSON фільтра записаного і скомпільованого в Haskell (`smallcaps`) і інтерпреторваного Python (`smallcaps.py`):

| Command                               | Time  |
| :------------------------------------ | :---- |
| `pandoc`                              | 1.01s |
| `pandoc --filter ./smallcaps`         | 1.36s |
| `pandoc --filter ./smallcaps.py`      | 1.40s |
| `pandoc --lua-filter ./smallcaps.lua` | 1.03s |

Як бачимо, фільтр Lua дозволяє уникнути значних накладних витрат, пов’язаних з маршируванням до і від JSON через конвеєр.

# Структура фільтрів Lua

Фільтри Lua - це таблиці з іменами елементів у вигляді ключів і значень, що складаються з функцій, що діють на ці елементи.

Очікується, що фільтри будуть поміщені в окремі файли та передані через аргумент командного рядка `--lua-filter`. Наприклад, якщо фільтр означений у файлі `current-date.lua`, він буде застосований так:

```bash
pandoc --lua-filter=current-date.lua -f markdown MANUAL.txt
```

Опція `--lua-filter` може поставлятися кілька разів. Pandoc застосовує всі фільтри (включаючи фільтри JSON, визначені через `--filter` та фільтри Lua, вказані через ` --lua-filter`) у порядку, який вони відображаються в командному рядку.

Pandoc очікує, що кожен файл Lua поверне список фільтрів. Фільтри у цьому списку визиваються послідовно, кожен за результатами попереднього фільтра. Якщо значення сценарію фільтра не повернеться, то pandoc спробує генерувати єдиний фільтр, зібравши всі функції верхнього рівня, назви яких відповідають назвам елементів pandoc (наприклад, `Str`,` Para`, `Meta`, або `Pandoc`). (Ось чому два вищевказані приклади є рівнозначними.)

Для кожного фільтра проходить документ і кожен елемент піддається фільтру. Елементи, для яких фільтр містить запис (тобто однойменна функція), передаються функції фільтрації елементів Lua. Іншими словами, записи фільтру будуть викликані для кожного відповідного елемента документа, отримуючи відповідний елемент як вхідний.

Повернення функції фільтра має бути одним із наступних:

- нуль: це означає, що об’єкт повинен залишатися незмінним.
- об'єкт pandoc: він повинен бути того ж типу, що і вхідний, і замінить початковий об'єкт.
- список об’єктів pandoc: вони замінять вихідний об'єкт; список об'єднується з сусідами оригінальних об'єктів (зрощений у список, до якого належить оригінальний об’єкт); повернення порожнього списку видаляє об'єкт.

Результат функції повинен мати результат того ж типу, що і вхід. Це означає, що функція фільтра, що діє на вбудований елемент, повинна повертати або нуль, рядок, або список рядків, а функція, що фільтрує блок-елемент, повинна повернути один нуль, блок або список блокових елементів. Pandoc видасть помилку, якщо ця умова буде порушена.

Якщо немає функції, що відповідає типу вузла елемента, система фільтрації шукатиме більш загальну функцію резервного запасу. Підтримуються дві резервні функції: `Inline` та `Block`. Кожен відповідає елементам відповідного типу.

Елементи без відповідних функцій залишаються недоторканими.

Див. [Документація модуля](https://pandoc.org/lua-filters.html#module-pandoc) для списку елементів pandoc.

## Фільтри за послідовностями елементів 

Для деяких завдань фільтрації необхідно знати порядок, в якому елементи зустрічаються в документі. Тоді недостатньо оглянути окремий елемент за раз.

Є дві назви спеціальних функцій, які можна використовувати для означення фільтрів у списках блоків або списках рядків.

- `Inlines (inlines)` - Якщо він присутній у фільтрі, ця функція буде викликана у всіх списках вбудованих елементів, як-от вміст блоку [Para](https://pandoc.org/lua-filters.html#type-para)  (paragraph), або  [Image](https://pandoc.org/lua-filters.html#type-image). Аргумент `inlines`, переданий функції, буде  [списком (List)](https://pandoc.org/lua-filters.html#type-list)  [рядків (Inlines)](https://pandoc.org/lua-filters.html#type-inline) для кожного виклику.

- `Blocks (blocks)` - Якщо він присутній у фільтрі, ця функція буде викликана у всіх списках елементів блоку, як-от уміст блоку мета-елементів [MetaBlocks](https://pandoc.org/lua-filters.html#type-metablocks) кожен елемент списку та основний зміст документа [Pandoc](https://pandoc.org/lua-filters.html#type-pandoc). Аргумент `blocks` , переданий функції, буде [списком(List)](https://pandoc.org/lua-filters.html#type-list) [блоків(Blocks)](https://pandoc.org/lua-filters.html#type-block) для кожного виклику.

Ці функції фільтра особливі тим, що результат повинен бути або нульовим, і в цьому випадку список залишається незмінним, або повинен бути списком правильного типу, тобто того ж типу, що і аргумент введення. Окремі елементи **не** дозволені як повернені значення, оскільки окремий елемент у цьому контексті зазвичай натякає на помилку.

See [“Remove spaces before normal citations”](https://pandoc.org/lua-filters.html#remove-spaces-before-citations) for an example.

## Порядок виконання

Функції фільтра елементів у наборі фільтрів викликаються у фіксованому порядку, пропускаючи ті, які відсутні:

1. функції для [*Inline* elements](https://pandoc.org/lua-filters.html#type-inline),
2. функції [`Inlines`](https://pandoc.org/lua-filters.html#inlines-filter) фільтрів,
3. функції для [*Block* elements](https://pandoc.org/lua-filters.html#type-block) ,
4. функції [`Blocks`](https://pandoc.org/lua-filters.html#inlines-filter) філтьтрів,
5. функції [`Meta`](https://pandoc.org/lua-filters.html#type-meta) фільтрації
6. функції фільтрації [`Pandoc`](https://pandoc.org/lua-filters.html#type-pandoc) .

Ще можна форсувати інший порядок, явно повертаючи кілька наборів фільтрів. Наприклад, якщо фільтр для *Meta* повинен запускатися до цього для *Str*, можна записати

```lua
-- ... filter definitions ...

return {
  { Meta = Meta },  -- (1)
  { Str = Str }     -- (2)
}
```

Набори фільтрів застосовуються в тому порядку, в якому вони повертаються. Таким чином, всі функції в наборі (1) виконуються перед функціями в (2), завдяки чому функція фільтра для *Meta* запускається до початку фільтрації елементів *Str*.

## Глобальні змінні

Pandoc передає додаткові дані фільтрам Lua, встановлюючи глобальні змінні.

- `FORMAT`

  The global `FORMAT` is set to the format of the pandoc writer being used (`html5`, `latex`, etc.), so the behavior of a filter can be made conditional on the eventual output format.

- `PANDOC_READER_OPTIONS`

  Table of the options which were provided to the parser.

- `PANDOC_VERSION`

  Contains the pandoc version as a [Version](https://pandoc.org/lua-filters.html#type-version) object which behaves like a numerically indexed table, most significant number first. E.g., for pandoc 2.7.3, the value of the variable is  equivalent to a table `{2, 7, 3}`. Use `tostring(PANDOC_VERSION)` to produce a version string. This variable is also set in custom writers.

- `PANDOC_API_VERSION`

  Contains the version of the pandoc-types API against which pandoc  was compiled. It is given as a numerically indexed table, most  significant number first. E.g., if pandoc was compiled against  pandoc-types 1.17.3, then the value of the variable will behave like the table `{1, 17, 3}`. Use `tostring(PANDOC_API_VERSION)` to produce a version string. This variable is also set in custom writers.

- `PANDOC_SCRIPT_FILE`

  The name used to involve the filter. This value can be used to find  files relative to the script file. This variable is also set in custom  writers.

- `PANDOC_STATE`

  The state shared by all readers and writers. It is used by pandoc to collect and pass information. The value of this variable is of type [CommonState](https://pandoc.org/lua-filters.html#type-commonstate) and is read-only.

# Pandoc Module

Модуль `pandoc` Lua завантажується в середовище Lua фільтра і забезпечує набір функцій і констант, щоб полегшити створення та маніпулювання елементами. Глобальна змінна `pandoc` пов'язана з модулем і, як правило, з цієї причини не повинна бути перезаписана.

Модуль забезпечується двома основними функціоналами: функція створення елементів та доступ до деяких основних функцій pandoc.

## Element creation

Функції для створення елементів, такі як `Str`,  `Para`, і `Pandoc` , призначені для легкого створення нових елементів, простих у використанні, і їх можна прочитати з середовища Lua. Внутрішньо, pandoc використовує ці функції для створення об'єктів Lua, які передаються функціям фільтра елементів. Це означає, що елементи, створені за допомогою цього модуля, будуть поводитись так само, як ті елементи, доступні через параметр функції фільтра.

## Exposed pandoc functionality

У Lua були доступні деякі функції pandoc:

- [`walk_block`](https://pandoc.org/lua-filters.html#pandoc.walk_block) and [`walk_inline`](https://pandoc.org/lua-filters.html#pandoc.walk_inline) дозволяють застосовувати фільтри всередині конкретних блокових або вбудованих елементів;
- [`read`](https://pandoc.org/lua-filters.html#pandoc.read) дозволяє фільтрам розбирати рядки в документи pandoc;
- [`pipe`](https://pandoc.org/lua-filters.html#pandoc.pipe) виконує зовнішню команду із введенням та виведенням до рядків;
- модуль [`pandoc.mediabag`](https://pandoc.org/lua-filters.html#module-pandoc.mediabag) дозволяє отримати доступ до медіа-сумки, в якій зберігається бінарний вміст, такий як зображення, які можуть бути включені до підсумкового документа;
- модуль [`pandoc.utils`](https://pandoc.org/lua-filters.html#module-pandoc.utils) містить різні функції утиліти.

# Lua interpreter initialization

Ініціалізацією інтерпретатора Lua можна управляти, розмістивши файл `init.lua` в каталозі даних pandoc. Загальним випадком використання буде завантаження додаткових модулів або навіть зміна модулів за замовчуванням.

Наступний фрагмент - приклад коду, який може бути корисним при додаванні до `init.lua`. Фрагмент додає всі функції, що знають унікод, визначені в модулі  [`text` module](https://pandoc.org/lua-filters.html#module-text) до модуля `string`  за замовчуванням, встановленим рядком `uc_ `.

```lua
for name, fn in pairs(require 'text') do
  string['uc_' .. name] = fn
end
```

Це дає можливість застосувати ці функції до рядків за допомогою синтаксису двокрапки  (`mystring:uc_upper()`).

# Приклади

Наступні фільтри представлені як приклади. Сховище корисних фільтрів Lua (яке також може слугувати хорошими прикладами) доступне за адресою  https://github.com/pandoc/lua-filters.

## Макро-замінник

Наступний фільтр перетворює рядок `{{helloworld}}` у підкреслений текст "Hello, World". 

```lua
return {
  {
    Str = function (elem) -- Str - це текстовий зміст 
      if elem.text == "{{helloworld}}" then -- text - це текстовий вміст
        return pandoc.Emph {pandoc.Str "Hello, World"} -- Emph - Підкреслений текст
      else
        return elem
      end
    end,
  }
}
```

## Центрування рисунків LaTeX і HTML виходу 

Для LaTeX оберніть зображення у фрагменти LaTeX, які спричиняють зображення по центру по горизонталі. У HTML атрибут стилю елемента зображення використовується для центрування.

```lua
-- Filter images with this function if the target format is LaTeX.
if FORMAT:match 'latex' then
  function Image (elem)
    -- Surround all images with image-centering raw LaTeX.
    return {
      pandoc.RawInline('latex', '\\hfill\\break{\\centering'),
      elem,
      pandoc.RawInline('latex', '\\par}')
    }
  end
end

-- Filter images with this function if the target format is HTML
if FORMAT:match 'html' then
  function Image (elem)
    -- Use CSS style to center image
    elem.attributes.style = 'margin:auto; display: block;'
    return elem
  end
end
```

## Встановлення дати в метаданих

Цей фільтр встановлює дату в метаданих документа до поточної дати:  

```lua
function Meta(m)
  m.date = os.date("%B %e, %Y")
  return m
end
```

## Отримання інформації про лінки

Цей фільтр друкує таблицю всіх URL-адрес, пов’язаних у документі, разом із кількістю посилань на цю URL-адресу.

```lua
links = {}

function Link (el)
  if links[el.target] then
    links[el.target] = links[el.target] + 1
  else
    links[el.target] = 1
  end
  return el
end

function Doc (blocks, meta)
  function strCell(str)
    return {pandoc.Plain{pandoc.Str(str)}}
  end
  local caption = {pandoc.Str "Link", pandoc.Space(), pandoc.Str "count"}
  local aligns = {pandoc.AlignDefault, pandoc.AlignLeft}
  local widths = {0.8, 0.2}
  local headers = {strCell "Target", strCell "Count"}
  local rows = {}
  for link, count in pairs(links) do
    rows[#rows + 1] = {strCell(link), strCell(count)}
  end
  return pandoc.Doc(
    {pandoc.Table(caption, aligns, widths, headers, rows)},
    meta
  )
end
```

## Видалення пробілів перед цитатами 

This filter removes all spaces preceding an “author-in-text” citation. In Markdown, author-in-text citations (e.g., `@citekey`), must be preceded by a space. If these spaces are undesired, they must be removed with a filter.

Цей фільтр видаляє всі пробіли, що передують цитуванню "авторський текст". У Markdown, цитатам авторського тексту (наприклад, `@citekey`) має передувати пробіл. Якщо ці пробіли небажані, їх потрібно видалити за допомогою фільтра.

```lua
local function is_space_before_author_in_text(spc, cite)
  return spc and spc.t == 'Space'
    and cite and cite.t == 'Cite'
    -- there must be only a single citation, and it must have
    -- mode 'AuthorInText'
    and #cite.citations == 1
    and cite.citations[1].mode == 'AuthorInText'
end

function Inlines (inlines)
  -- Go from end to start to avoid problems with shifting indices.
  for i = #inlines-1, 1, -1 do
    if is_space_before_author_in_text(inlines[i], inlines[i+1]) then
      inlines:remove(i)
    end
  end
  return inlines
end
```

## Заміна заповнювачів їх значенням метаданих 

Функції фільтра Lua виконуються в порядку 

> *Inlines → Blocks → Meta → Pandoc*.

Передача інформації з вищого рівня (наприклад, метадані) на нижчий (наприклад, рядки) все ще можлива за допомогою двох фільтрів, що живуть в одному файлі:

```lua
local vars = {}
function get_vars (meta)
  for k, v in pairs(meta) do
    if type(v) == 'table' and v.t == 'MetaInlines' then
      vars["%" .. k .. "%"] = {table.unpack(v)}
    end
  end
end
function replace (el)
  if vars[el.text] then
    return pandoc.Span(vars[el.text])
  else
    return el
  end
end
return {{Meta = get_vars}, {Str = replace}}
```

If the contents of file `occupations.md` is

```lua
---
name: Samuel Q. Smith
occupation: Professor of Phrenology
---

Name

:   %name%

Occupation

:   %occupation%
```

then running `pandoc --lua-filter=meta-vars.lua occupations.md` will output:

```lua
<dl>
<dt>Name</dt>
<dd><p><span>Samuel Q. Smith</span></p>
</dd>
<dt>Occupation</dt>
<dd><p><span>Professor of Phrenology</span></p>
</dd>
</dl>
```

## Модифкування pandoc’s `MANUAL.txt` для сторінок man

Це фільтр, який ми використовуємо при перетворенні `MANUAL.txt` на сторінки інструкції. Він перетворює заголовки рівня 1 у великі регістри (використовуючи `walk_block` для перетворення вбудованих елементів всередині заголовків), видаляє виноски та замінює посилання звичайним текстом.

```lua
-- ми використовуємо попередньо завантажений текст, щоб отримати UTF-8 названий функцією 'upper' 
local text = require('text')

function Header(el)
    if el.level == 1 then -- заголовки рівня 1
      return pandoc.walk_block(el, {
        Str = function(el)
            return pandoc.Str(text.upper(el.text))
        end })
    end
end

function Link(el)
    return el.content
end

function Note(el)
    return {}
end
```

## Створення роздаткового матеріалу з паперу 

Цей фільтр витягує всі нумеровані приклади, заголовки розділів, блоки цитат та рисунки з документа, а також будь-які divs з класом `handout`. (Зверніть увагу, що включені лише блоки на "зовнішньому рівні"; це ігнорує блоки всередині вкладених конструкцій, наприклад елементи списку.)

```lua
-- creates a handout from an article, using its headings,
-- blockquotes, numbered examples, figures, and any
-- Divs with class "handout"

function Pandoc(doc)
    local hblocks = {}
    for i,el in pairs(doc.blocks) do
        if (el.t == "Div" and el.classes[1] == "handout") or
           (el.t == "BlockQuote") or
           (el.t == "OrderedList" and el.style == "Example") or
           (el.t == "Para" and #el.c == 1 and el.c[1].t == "Image") or
           (el.t == "Header") then
           table.insert(hblocks, el)
        end
    end
    return pandoc.Pandoc(hblocks, doc.meta)
end
```

## Counting words in a document

Цей фільтр підраховує слова в тілі документа (опускаючи метадані, як назви та конспекти), включаючи слова в коді. Це повинно бути більш точним, ніж  `wc -w` ,  що виконується безпосередньо на документі Markdown, оскільки останній буде рахувати символи розмітки, як  `#`  перед заголовком ATX, або теги в документах HTML, як слова. Щоб запустити його,  `pandoc --lua-filter wordcount.lua myfile.md`.

```lua
-- counts words in a document

words = 0

wordcount = {
  Str = function(el)
    -- we don't count a word if it's entirely punctuation:
    if el.text:match("%P") then
        words = words + 1
    end
  end,

  Code = function(el)
    _,n = el.text:gsub("%S+","")
    words = words + n
  end,

  CodeBlock = function(el)
    _,n = el.text:gsub("%S+","")
    words = words + n
  end
}

function Pandoc(el)
    -- skip metadata, just count body:
    pandoc.walk_block(pandoc.Div(el.blocks), wordcount)
    print(words .. " words in body")
    os.exit(0)
end
```

## Converting ABC code to music notation

This filter replaces code blocks with class `abc` with images created by running their contents through `abcm2ps` and ImageMagick’s `convert`. (For more on ABC notation, see https://abcnotation.com.)

Images are added to the mediabag. For output to binary formats,  pandoc will use images in the mediabag. For textual formats, use `--extract-media` to specify a directory where the files in the mediabag will be written, or (for HTML only) use `--self-contained`.

```lua
-- Pandoc filter to process code blocks with class "abc" containing
-- ABC notation into images.
--
-- * Assumes that abcm2ps and ImageMagick's convert are in the path.
-- * For textual output formats, use --extract-media=abc-images
-- * For HTML formats, you may alternatively use --self-contained

local filetypes = { html = {"png", "image/png"}
                  , latex = {"pdf", "application/pdf"}
                  }
local filetype = filetypes[FORMAT][1] or "png"
local mimetype = filetypes[FORMAT][2] or "image/png"

local function abc2eps(abc, filetype)
    local eps = pandoc.pipe("abcm2ps", {"-q", "-O", "-", "-"}, abc)
    local final = pandoc.pipe("convert", {"-", filetype .. ":-"}, eps)
    return final
end

function CodeBlock(block)
    if block.classes[1] == "abc" then
        local img = abc2eps(block.text, filetype)
        local fname = pandoc.sha1(img) .. "." .. filetype
        pandoc.mediabag.insert(fname, mimetype, img)
        return pandoc.Para{ pandoc.Image({pandoc.Str("abc tune")}, fname) }
    end
end
```


