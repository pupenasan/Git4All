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

## Building images with Ti*k*Z

This filter converts raw LaTeX Ti*k*Z environments into images. It works with both PDF and HTML output. The Ti*k*Z code is compiled to an image using `pdflatex`, and the image is converted from pdf to svg format using [`pdf2svg`](https://github.com/dawbarton/pdf2svg), so both of these must be in the system path. Converted images are  cached in the working directory and given filenames based on a hash of  the source, so that they need not be regenerated each time the document  is built. (A more sophisticated version of this might put these in a  special cache directory.)

```lua
local function tikz2image(src, filetype, outfile)
    local tmp = os.tmpname()
    local tmpdir = string.match(tmp, "^(.*[\\/])") or "."
    local f = io.open(tmp .. ".tex", 'w')
    f:write("\\documentclass{standalone}\n\\usepackage{xcolor}\n\\usepackage{tikz}\n\\begin{document}\n\\nopagecolor\n")
    f:write(src)
    f:write("\n\\end{document}\n")
    f:close()
    os.execute("pdflatex -output-directory " .. tmpdir  .. " " .. tmp)
    if filetype == 'pdf' then
        os.rename(tmp .. ".pdf", outfile)
    else
        os.execute("pdf2svg " .. tmp .. ".pdf " .. outfile)
    end
    os.remove(tmp .. ".tex")
    os.remove(tmp .. ".pdf")
    os.remove(tmp .. ".log")
    os.remove(tmp .. ".aux")
end

extension_for = {
    html = 'svg',
    html4 = 'svg',
    html5 = 'svg',
    latex = 'pdf',
    beamer = 'pdf' }

local function file_exists(name)
    local f = io.open(name, 'r')
    if f ~= nil then
        io.close(f)
        return true
    else
        return false
    end
end

local function starts_with(start, str)
   return str:sub(1, #start) == start
end


function RawBlock(el)
    if starts_with("\\begin{tikzpicture}", el.text) then
        local filetype = extension_for[FORMAT] or "svg"
        local fname = pandoc.sha1(el.text) .. "." .. filetype
        if not file_exists(fname) then
            tikz2image(el.text, filetype, fname)
        end
        return pandoc.Para({pandoc.Image({}, fname)})
    else
       return el
    end
end
```

Example of use:

```lua
pandoc --lua-filter tikz.lua -s -o cycle.html <<EOF
Here is a diagram of the cycle:

\begin{tikzpicture}

\def \n {5}
\def \radius {3cm}
\def \margin {8} % margin in angles, depends on the radius

\foreach \s in {1,...,\n}
{
  \node[draw, circle] at ({360/\n * (\s - 1)}:\radius) {$\s$};
  \draw[->, >=latex] ({360/\n * (\s - 1)+\margin}:\radius)
    arc ({360/\n * (\s - 1)+\margin}:{360/\n * (\s)-\margin}:\radius);
}
\end{tikzpicture}
EOF
```

# Lua type reference

У цьому розділі описані типи об'єктів, доступних фільтрам Lua. Див [pandoc module](https://pandoc.org/lua-filters.html#module-pandoc) для функцій для створення цих об'єктів.

## Загальні властивості 

### `clone`

```lua
clone ()
```

Усі екземпляри перелічених тут типів, за винятком об'єктів лише для читання, можуть бути клоновані методом `clone ()`.

Використання:

```lua
local emph = pandoc.Emph {pandoc.Str 'important'}
local cloned_emph = emph:clone()  -- note the colon
```

## Pandoc

Доумент Pandoc. Значення цього типу можна створити за допомогою конструктору [`pandoc.Pandoc`](https://pandoc.org/lua-filters.html#pandoc.pandoc) . Об'єктна рівність визначається через [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

- `blocks` - зміст документу - список блоків ([List](https://pandoc.org/lua-filters.html#type-list) of [Block](https://pandoc.org/lua-filters.html#type-block))

- `meta` - мета-інформація про документ ([Meta](https://pandoc.org/lua-filters.html#type-meta) object)

## Meta

Мета-інформація про документ; строково-індексована колекція обєктів [MetaValues](https://pandoc.org/lua-filters.html#type-metavalue). Значення цього типу можна створити за допомогою конструктору [`pandoc.Meta`](https://pandoc.org/lua-filters.html#pandoc.meta) . Об'єктна рівність визначається через [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

## MetaValue

Елемент Мета-інформація про документ. Об'єктна рівність визначається через [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

### MetaBlocks

Список блоків, що використовуються як мета-значення - список блоків ([List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block)). Поля:

- `tag`, `t` - літерали типу `MetaBlocks` (string)

### MetaBool

Псевдонім для Lua boolean, наприклад значення `true` та`false`.

### MetaInlines

Список рядків (inlines) використовуваних в метаданих ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline)). Значення цього типу можна створити за допомогою конструктору [`pandoc.MetaInlines`](https://pandoc.org/lua-filters.html#pandoc.metainlines). Поля:

- `tag`, `t` - літерали типу `MetaInlines` (string)

### MetaList

A list of other metadata values ([List](https://pandoc.org/lua-filters.html#type-list) of [MetaValues](https://pandoc.org/lua-filters.html#type-metavalue)). Values of this type can be created with the [`pandoc.MetaList`](https://pandoc.org/lua-filters.html#pandoc.metalist) constructor. Fields:

- `tag`, `t` - the literal `MetaList` (string)

All methods available for [List](https://pandoc.org/lua-filters.html#type-list)s can be used on this type as well.

### MetaMap

A string-indexed map of meta-values. (table). Values of this type can be created with the [`pandoc.MetaMap`](https://pandoc.org/lua-filters.html#pandoc.metamap) constructor. Fields:

- `tag`, `t` - the literal `MetaMap` (string)

*Note*: The fields will be shadowed if the map contains a field with the same name as those listed.

### MetaString

Plain Lua string value (string).

## Block

Object equality is determined via [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

### BlockQuote

A block quote element. Values of this type can be created with the [`pandoc.BlockQuote`](https://pandoc.org/lua-filters.html#pandoc.blockquote) constructor. Fields:

- `content` - block content ([List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block))

- `tag`, `t` - the literal `BlockQuote` (string)

### BulletList

A bullet list. Values of this type can be created with the [`pandoc.BulletList`](https://pandoc.org/lua-filters.html#pandoc.bulletlist) constructor. Fields:

- `content` - list of items ([List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block))

- `tag`, `t` - the literal `BulletList` (string)

### CodeBlock

Block of code. Values of this type can be created with the [`pandoc.CodeBlock`](https://pandoc.org/lua-filters.html#pandoc.codeblock) constructor. Fields:

- `text` - code string (string)

- `attr` - element attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `CodeBlock` (string)

### DefinitionList

Definition list, containing terms and their explanation. Values of this type can be created with the [`pandoc.DefinitionList`](https://pandoc.org/lua-filters.html#pandoc.definitionlist) constructor. Fields:

- `content` - list of items

- `tag`, `t` - the literal `DefinitionList` (string)

### Div

Generic block container with attributes. Values of this type can be created with the [`pandoc.Div`](https://pandoc.org/lua-filters.html#pandoc.div) constructor. Fields:

- `content` - block content ([List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block))

- `attr` - element attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Div` (string)

### Header

Creates a header element. Values of this type can be created with the [`pandoc.Header`](https://pandoc.org/lua-filters.html#pandoc.header) constructor. Fields:

- `level` - header level (integer)

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `attr` - element attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Header` (string)

### HorizontalRule

A horizontal rule. Values of this type can be created with the [`pandoc.HorizontalRule`](https://pandoc.org/lua-filters.html#pandoc.horizontalrule) constructor. Fields:

- `tag`, `t` - the literal `HorizontalRule` (string)

### LineBlock

A line block, i.e. a list of lines, each separated from the next by a newline. Values of this type can be created with the [`pandoc.LineBlock`](https://pandoc.org/lua-filters.html#pandoc.lineblock) constructor. Fields:

- `content` - inline content

- `tag`, `t` - the literal `LineBlock` (string)

### Null

A null element; this element never produces any output in the target format. Values of this type can be created with the [`pandoc.Null`](https://pandoc.org/lua-filters.html#pandoc.null) constructor. 

- `tag`, `t` - the literal `Null` (string)

### OrderedList

An ordered list. Values of this type can be created with the [`pandoc.OrderedList`](https://pandoc.org/lua-filters.html#pandoc.orderedlist) constructor. Fields:

- `content` - list items ([List](https://pandoc.org/lua-filters.html#type-list) of [List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block))

- `listAttributes` - list parameters ([ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes))

- `start` - alias for `listAttributes.start` (integer)

- `style` - alias for `listAttributes.style` (string)

- `delimiter` - alias for `listAttributes.delimiter` (string)

- `tag`, `t` - the literal `OrderedList` (string)

### Para

A paragraph. Values of this type can be created with the [`pandoc.Para`](https://pandoc.org/lua-filters.html#pandoc.para) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Para` (string)

### Plain

Plain text, not a paragraph. Values of this type can be created with the [`pandoc.Plain`](https://pandoc.org/lua-filters.html#pandoc.plain) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Plain` (string)

### RawBlock

Raw content of a specified format. Values of this type can be created with the [`pandoc.RawBlock`](https://pandoc.org/lua-filters.html#pandoc.rawblock) constructor. Fields:

- `format` - format of content (string)

- `text` - raw content (string)

- `tag`, `t` - the literal `RawBlock` (string)

### Table

A table. Values of this type can be created with the [`pandoc.Table`](https://pandoc.org/lua-filters.html#pandoc.table) constructor. Fields:

- `caption` - table caption ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `aligns` - column alignments ([List](https://pandoc.org/lua-filters.html#type-list) of [Alignment](https://pandoc.org/lua-filters.html#type-alignment)s)

- `widths` - column widths (number)

- `headers` - header row ([List](https://pandoc.org/lua-filters.html#type-list) of [table cells](https://pandoc.org/lua-filters.html#type-table-cell))

- `rows` - table rows ([List](https://pandoc.org/lua-filters.html#type-list) of [List](https://pandoc.org/lua-filters.html#type-list)s of [table cells](https://pandoc.org/lua-filters.html#type-table-cell))

- `tag`, `t` - the literal `Table` (string)

A table cell is a list of blocks. *Alignment* is a string value indicating the horizontal alignment of a table column. `AlignLeft`, `AlignRight`, and `AlignCenter` leads cell content tob be left-aligned, right-aligned, and centered, respectively. The default alignment is `AlignDefault` (often equivalent to centered).

## Inline

Об'єктна рівність визначається через [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

### Cite

Citation. Values of this type can be created with the [`pandoc.Cite`](https://pandoc.org/lua-filters.html#pandoc.cite) constructor. Fields:

- `content` - ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `citations` - citation entries ([List](https://pandoc.org/lua-filters.html#type-list) of [Citations](https://pandoc.org/lua-filters.html#type-citation))

- `tag`, `t` - the literal `Cite` (string)

### Code

Inline code. Values of this type can be created with the [`pandoc.Code`](https://pandoc.org/lua-filters.html#pandoc.code) constructor.  Fields:

- `text` - code string (string)

- `attr` - attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Code` (string)

### Emph

Підкреслений текст. Значення цього типу можна створити за допомогою конструктору [`pandoc.Emph`](https://pandoc.org/lua-filters.html#pandoc.emph) . Поля:

- `content` - вбудований контент ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Emph` (string)

### Image

Image: alt text (list of inlines), target. Values of this type can be created with the [`pandoc.Image`](https://pandoc.org/lua-filters.html#pandoc.image) constructor. Fields:

- `attr` - attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `caption` - text used to describe the image ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `src` - path to the image file (string)

- `title` - brief image description

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Image` (string)

### LineBreak

Жорсткий розрив лінії. Значення цього типу можна створити за допомогою конструктора [`pandoc.LineBreak`](https://pandoc.org/lua-filters.html#pandoc.linebreak). Поля:

- `tag`, `t` - the literal `LineBreak` (string)

### Link

Hyperlink: alt text (list of inlines), target. Values of this type can be created with the [`pandoc.Link`](https://pandoc.org/lua-filters.html#pandoc.link) constructor. Fields:

- `attr` - attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `content` - text for this link ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `target` - the link target (string)

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Link` (string)

### Math

TeX math (literal). Values of this type can be created with the [`pandoc.Math`](https://pandoc.org/lua-filters.html#pandoc.math) constructor. Fields:

- `mathtype` - specifier determining whether the math content should be shown inline (`InlineMath`) or on a separate line (`DisplayMath`) (string)

- `text` - math content (string)

- `tag`, `t` - the literal `Math` (string)

### Note

Footnote or endnote. Values of this type can be created with the [`pandoc.Note`](https://pandoc.org/lua-filters.html#pandoc.note) constructor. Fields:

- `content` - ([List](https://pandoc.org/lua-filters.html#type-list) of [Blocks](https://pandoc.org/lua-filters.html#type-block))

- `tag`, `t` - the literal `Note` (string)

### Quoted

Quoted text. Values of this type can be created with the [`pandoc.Quoted`](https://pandoc.org/lua-filters.html#pandoc.quoted) constructor. Fields:

- `quotetype` - type of quotes to be used; one of `SingleQuote` or `DoubleQuote` (string)

- `content` - quoted text ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Quoted` (string)

### RawInline

Raw inline. Values of this type can be created with the [`pandoc.RawInline`](https://pandoc.org/lua-filters.html#pandoc.rawinline) constructor. Fields:

- `format` - the format of the content (string)

- `text` - raw content (string)

- `tag`, `t` - the literal `RawInline` (string)

### SmallCaps

Small caps text. Values of this type can be created with the [`pandoc.SmallCaps`](https://pandoc.org/lua-filters.html#pandoc.smallcaps) constructor. Fields:

- `content` - ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t`-  the literal `SmallCaps` (string)

### SoftBreak

Soft line break. Values of this type can be created with the [`pandoc.SoftBreak`](https://pandoc.org/lua-filters.html#pandoc.softbreak) constructor. Fields:

- `tag`, `t` - the literal `SoftBreak` (string)

### Space

Inter-word space. Values of this type can be created with the [`pandoc.Space`](https://pandoc.org/lua-filters.html#pandoc.space) constructor. Fields:

- `tag`, `t` - the literal `Space` (string)

### Span

Generic inline container with attributes. Values of this type can be created with the [`pandoc.Span`](https://pandoc.org/lua-filters.html#pandoc.span) constructor. Fields:

- `attr` - attributes ([Attr](https://pandoc.org/lua-filters.html#type-attr))

- `content` - wrapped content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `identifier` - alias for `attr.identifier` (string)

- `classes` - alias for `attr.classes` ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes` - alias for `attr.attributes` ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

- `tag`, `t` - the literal `Span` (string)

### Str

Текст. Значення цього типу можна створити за допомогою конструктору [`pandoc.Str`](https://pandoc.org/lua-filters.html#pandoc.str) constructor. Поля:

- `text` - content (string)

- `tag`, `t` - літерал `Str` (string)

### Strikeout

Strikeout text. Values of this type can be created with the [`pandoc.Strikeout`](https://pandoc.org/lua-filters.html#pandoc.strikeout) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Strikeout` (string)

### Strong

Strongly emphasized text. Values of this type can be created with the [`pandoc.Strong`](https://pandoc.org/lua-filters.html#pandoc.strong) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Strong` (string)

### Subscript

Subscripted text. Values of this type can be created with the [`pandoc.Subscript`](https://pandoc.org/lua-filters.html#pandoc.subscript) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Subscript` (string)

### Superscript

Superscripted text. Values of this type can be created with the [`pandoc.Superscript`](https://pandoc.org/lua-filters.html#pandoc.superscript) constructor. Fields:

- `content` - inline content ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `tag`, `t` - the literal `Superscript` (string)

## Element components

### Attr

A set of element attributes. Values of this type can be created with the [`pandoc.Attr`](https://pandoc.org/lua-filters.html#pandoc.attr) constructor. For convenience, it is usually not necessary to construct  the value directly if it is part of an element, and it is sufficient to  pass an HTML-like table. E.g., to create a span with identifier “text”  and classes “a” and “b”, on can write:

```lua
local span = pandoc.Span('text', {id = 'text', class = 'a b'})
```

This also works when using the `attr` setter:

```lua
local span = pandoc.Span 'text'
span.attr = {id = 'text', class = 'a b', other_attribute = '1'}
```

Object equality is determined via [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

Fields:

- `identifier`

  element identifier (string)

- `classes`

  element classes ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `attributes`

  collection of key/value pairs ([Attributes](https://pandoc.org/lua-filters.html#type-attributes))

### Attributes

List of key/value pairs. Values can be accessed by using keys as indices to the list table.

### Citation

Single citation entry

Values of this type can be created with the [`pandoc.Citation`](https://pandoc.org/lua-filters.html#pandoc.citation) constructor.

Object equality is determined via [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

Fields:

- `id`

  citation identifier, e.g., a bibtex key (string)

- `mode`

  citation mode, one of `AuthorInText`, `SuppressAuthor`, or `NormalCitation` (string)

- `prefix`

  citation prefix ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `suffix`

  citation suffix ([List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline))

- `note_num`

  note number (integer)

- `hash`

  hash (integer)

### ListAttributes

List attributes

Values of this type can be created with the [`pandoc.ListAttributes`](https://pandoc.org/lua-filters.html#pandoc.listattributes) constructor.

Object equality is determined via [`pandoc.utils.equals`](https://pandoc.org/lua-filters.html#pandoc.utils.equals).

Fields:

- `start`

  number of the first list item (integer)

- `style`

  style used for list numbers; possible values are `DefaultStyle`, `Example`, `Decimal`, `LowerRoman`, `UpperRoman`, `LowerAlpha`, and `UpperAlpha` (string)

- `delimiter`

  delimiter of list numbers; one of `DefaultDelim`, `Period`, `OneParen`, and `TwoParens` (string)

## ReaderOptions

Pandoc reader options

Fields:

- `abbreviations`

  set of known abbreviations (set of strings)

- `columns`

  number of columns in terminal (integer)

- `default_image_extension`

  default extension for images (string)

- `extensions`

  string representation of the syntax extensions bit field (string)

- `indented_code_classes`

  default classes for indented code blocks (list of strings)

- `standalone`

  whether the input was a standalone document with header (boolean)

- `strip_comments`

  HTML comments are stripped instead of parsed as raw HTML (boolean)

- `tab_stop`

  width (i.e. equivalent number of spaces) of tab stops (integer)

- `track_changes`

  track changes setting for docx; one of `AcceptChanges`, `RejectChanges`, and `AllChanges` (string)

## CommonState

The state used by pandoc to collect information and make it available to readers and writers.

Fields:

- `input_files`

  List of input files from command line ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `output_file`

  Output file from command line (string or nil)

- `log`

  A list of log messages in reverse order ([List](https://pandoc.org/lua-filters.html#type-list) of [LogMessage](https://pandoc.org/lua-filters.html#type-logmessage)s)

- `request_headers`

  Headers to add for HTTP requests; table with header names as keys and header contents as value (table)

- `resource_path`

  Path to search for resources like included images ([List](https://pandoc.org/lua-filters.html#type-list) of strings)

- `source_url`

  Absolute URL or directory of first source file (string or nil)

- `user_data_dir`

  Directory to search for data files (string or nil)

- `trace`

  Whether tracing messages are issued (boolean)

- `verbosity`

  Verbosity level; one of `INFO`, `WARNING`, `ERROR` (string)

## List

A list is any Lua table with integer indices. Indices start at one, so if `alist = {'value'}` then `alist[1] == 'value'`.

Lists, when part of an element, or when generated during marshalling, are made instances of the `pandoc.List` type for convenience. The `pandoc.List` type is defined in the [*pandoc.List*](https://pandoc.org/lua-filters.html#module-pandoc.list) module. See there for available methods.

Values of this type can be created with the [`pandoc.List`](https://pandoc.org/lua-filters.html#pandoc.list) constructor, turning a normal Lua table into a List.

## LogMessage

A pandoc log message. Objects have no fields, but can be converted to a string via `tostring`.

## Version

A version object. This represents a software version like “2.7.3”.  The object behaves like a numerically indexed table, i.e., if `version` represents the version `2.7.3`, then

```lua
version[1] == 2
version[2] == 7
version[3] == 3
#version == 3   -- length
```

Comparisons are performed element-wise, i.e.

```lua
Version '1.12' > Version '1.9'
```

Values of this type can be created with the [`pandoc.types.Version`](https://pandoc.org/lua-filters.html#pandoc.types.version) constructor.

### `must_be_at_least`

```lua
must_be_at_least(actual, expected [, error_message])
```

Raise an error message if the actual version is older than the  expected version; does nothing if actual is equal to or newer than the  expected version.

Parameters:

- `actual`

  actual version specifier ([Version](https://pandoc.org/lua-filters.html#type-version))

- `expected`

  minimum expected version ([Version](https://pandoc.org/lua-filters.html#type-version))

- `error_message`

  optional error message template. The string is used as format  string, with the expected and actual versions as arguments. Defaults to `"expected version %s or newer, got %s"`.

Usage:

```lua
PANDOC_VERSION:must_be_at_least '2.7.3'
PANDOC_API_VERSION:must_be_at_least(
  '1.17.4',
  'pandoc-types is too old: expected version %s, got %s'
)
```

# Module text

UTF-8 aware text manipulation functions, implemented in Haskell. The module is made available as part of the `pandoc` module via `pandoc.text`. The text module can also be loaded explicitly:

```lua
-- uppercase all regular text in a document:
text = require 'text'
function Str (s)
  s.text = text.upper(s.text)
  return s
end
```

### lower

```lua
lower (s)
```

Returns a copy of a UTF-8 string, converted to lowercase.

### upper

```lua
upper (s)
```

Returns a copy of a UTF-8 string, converted to uppercase.

### reverse

```lua
reverse (s)
```

Returns a copy of a UTF-8 string, with characters reversed.

### len

```lua
len (s)
```

Returns the length of a UTF-8 string.

### sub

```lua
sub (s)
```

Returns a substring of a UTF-8 string, using Lua’s string indexing rules.

# Module pandoc

Lua функції для pandoc scripts; включає конструктори для дерева елементів документу, функції для парсингу тексту в наданому форматі, і функції для фільтрації та модифікування піддерева.

## Pandoc

- `Pandoc (blocks[, meta])`

  Повний набір параметрів документа pandoc:
  
   `blocks`: document content `meta`: document meta data  Returns: [Pandoc](https://pandoc.org/lua-filters.html#type-pandoc) object

## Meta

- `Meta (table)`

  Create a new Meta object. Parameters: `table`: table containing document meta information  Returns: [Meta](https://pandoc.org/lua-filters.html#type-meta) object

## MetaValue

- `MetaBlocks (blocks)`

  Meta blocks Parameters: `blocks`: blocks  Returns: [MetaBlocks](https://pandoc.org/lua-filters.html#type-metablocks) object

- `MetaInlines (inlines)`

  Meta inlines Parameters: `inlines`: inlines  Returns: [MetaInlines](https://pandoc.org/lua-filters.html#type-metainlines) object

- `MetaList (meta_values)`

  Meta list Parameters: `meta_values`: list of meta values  Returns: [MetaList](https://pandoc.org/lua-filters.html#type-metalist) object

- `MetaMap (key_value_map)`

  Meta map Parameters: `key_value_map`: a string-indexed map of meta values  Returns: [MetaMap](https://pandoc.org/lua-filters.html#type-metamap) object

- `MetaString (str)`

  Creates string to be used in meta data. Parameters: `str`: string value  Returns: [MetaString](https://pandoc.org/lua-filters.html#type-metastring) object

- `MetaBool (bool)`

  Creates boolean to be used in meta data. Parameters: `bool`: boolean value  Returns: [MetaBool](https://pandoc.org/lua-filters.html#type-metabool) object

## Blocks

- `BlockQuote (content)`

  Creates a block quote element Parameters: `content`: block content  Returns: [BlockQuote](https://pandoc.org/lua-filters.html#type-blockquote) object

- `BulletList (content)`

  Creates a bullet (i.e. Parameters: `content`: list of items  Returns: [BulletList](https://pandoc.org/lua-filters.html#type-bulletlist) object

- `CodeBlock (text[, attr])`

  Creates a code block element Parameters: `text`: code string `attr`: element attributes  Returns: [CodeBlock](https://pandoc.org/lua-filters.html#type-codeblock) object

- `DefinitionList (content)`

  Creates a definition list, containing terms and their explanation. Parameters: `content`: list of items  Returns: [DefinitionList](https://pandoc.org/lua-filters.html#type-definitionlist) object

- `Div (content[, attr])`

  Creates a div element Parameters: `content`: block content `attr`: element attributes  Returns: [Div](https://pandoc.org/lua-filters.html#type-div) object

- `Header (level, content[, attr])`

  Creates a header element. Parameters: `level`: header level `content`: inline content `attr`: element attributes  Returns: [Header](https://pandoc.org/lua-filters.html#type-header) object

- `HorizontalRule ()` Creates a horizontal rule. Returns: [HorizontalRule](https://pandoc.org/lua-filters.html#type-horizontalrule) object

- `LineBlock (content)`

  Creates a line block element. Parameters: `content`: inline content  Returns: [LineBlock](https://pandoc.org/lua-filters.html#type-lineblock) object

- `Null ()`

  Creates a null element. Returns: [Null](https://pandoc.org/lua-filters.html#type-null) object

- `OrderedList (items[, listAttributes])`

  Creates an ordered list. Parameters: `items`: list items `listAttributes`: list parameters  Returns: [OrderedList](https://pandoc.org/lua-filters.html#type-orderedlist) object

- `Para (content)`

  Creates a para element. Parameters: `content`: inline content  Returns: [Para](https://pandoc.org/lua-filters.html#type-para) object

- `Plain (content)`

  Creates a plain element. Parameters: `content`: inline content  Returns: [Plain](https://pandoc.org/lua-filters.html#type-plain) object

- `RawBlock (format, text)`

  Creates a raw content block of the specified format. Parameters: `format`: format of content `text`: string content  Returns: [RawBlock](https://pandoc.org/lua-filters.html#type-rawblock) object

- `Table (caption, aligns, widths, headers, rows)`

  Creates a table element. Parameters: `caption`: table caption `aligns`: alignments `widths`: column widths `headers`: header row `rows`: table rows  Returns: [Table](https://pandoc.org/lua-filters.html#type-table) object

## Inline

- `Cite (content, citations)`

  Creates a Cite inline element Parameters: `content`: List of inlines `citations`: List of citations  Returns: [Cite](https://pandoc.org/lua-filters.html#type-cite) object

- `Code (text[, attr])`

  Creates a Code inline element Parameters: `text`: code string `attr`: additional attributes  Returns: [Code](https://pandoc.org/lua-filters.html#type-code) object

- `Emph (content)`

  Creates an inline element representing emphasised text. Parameters: `content`: inline content  Returns: [Emph](https://pandoc.org/lua-filters.html#type-emph) object

- `Image (caption, src[, title[, attr]])`

  Creates a Image inline element Parameters: `caption`: text used to describe the image `src`: path to the image file `title`: brief image description `attr`: additional attributes  Returns: [Image](https://pandoc.org/lua-filters.html#type-image) object

- `LineBreak ()`

  Create a LineBreak inline element Returns: [LineBreak](https://pandoc.org/lua-filters.html#type-linebreak) object

- `Link (content, target[, title[, attr]])`

  Creates a link inline element, usually a hyperlink. Parameters: `content`: text for this link `target`: the link target `title`: brief link description `attr`: additional attributes  Returns: [Link](https://pandoc.org/lua-filters.html#type-link) object

- `Math (mathtype, text)`

  Creates a Math element, either inline or displayed. Parameters: `mathtype`: rendering specifier `text`: Math content  Returns: [Math](https://pandoc.org/lua-filters.html#type-math) object

- `DisplayMath (text)`

  Creates a math element of type “DisplayMath” (DEPRECATED). Parameters: `text`: Math content  Returns: [Math](https://pandoc.org/lua-filters.html#type-math) object

- `InlineMath (text)`

  Creates a math element of type “InlineMath” (DEPRECATED). Parameters: `text`: Math content  Returns: [Math](https://pandoc.org/lua-filters.html#type-math) object

- `Note (content)`

  Creates a Note inline element Parameters: `content`: footnote block content  Returns: [Note](https://pandoc.org/lua-filters.html#type-note) object

- `Quoted (quotetype, content)`

  Creates a Quoted inline element given the quote type and quoted content. Parameters: `quotetype`: type of quotes to be used `content`: inline content  Returns: [Quoted](https://pandoc.org/lua-filters.html#type-quoted) object

- `SingleQuoted (content)`

  Creates a single-quoted inline element (DEPRECATED). Parameters: `content`: inline content  Returns: [Quoted](https://pandoc.org/lua-filters.html#type-quoted)

- `DoubleQuoted (content)`

  Creates a single-quoted inline element (DEPRECATED). Parameters: `content`: inline content  Returns: [Quoted](https://pandoc.org/lua-filters.html#type-quoted)

- `RawInline (format, text)`

  Creates a raw inline element Parameters: `format`: format of the contents `text`: string content  Returns: [RawInline](https://pandoc.org/lua-filters.html#type-rawinline) object

- `SmallCaps (content)`

  Creates text rendered in small caps Parameters: `content`: inline content  Returns: [SmallCaps](https://pandoc.org/lua-filters.html#type-smallcaps) object

- `SoftBreak ()`

  Creates a SoftBreak inline element. Returns: [SoftBreak](https://pandoc.org/lua-filters.html#type-softbreak) object

- `Space ()`

  Create a Space inline element Returns: [Space](https://pandoc.org/lua-filters.html#type-space) object

- `Span (content[, attr])`

  Creates a Span inline element Parameters: `content`: inline content `attr`: additional attributes  Returns: [Span](https://pandoc.org/lua-filters.html#type-span) object

- `Str (text)`

  Creates a Str inline element Parameters: `text`: content  Returns: [Str](https://pandoc.org/lua-filters.html#type-str) object

- `Strikeout (content)`

  Creates text which is striked out. Parameters: `content`: inline content  Returns: [Strikeout](https://pandoc.org/lua-filters.html#type-strikeout) object

- `Strong (content)`

  Creates a Strong element, whose text is usually displayed in a bold font. Parameters: `content`: inline content  Returns: [Strong](https://pandoc.org/lua-filters.html#type-strong) object

- `Subscript (content)`

  Creates a Subscript inline element Parameters: `content`: inline content  Returns: [Subscript](https://pandoc.org/lua-filters.html#type-subscript) object

- `Superscript (content)`

  Creates a Superscript inline element Parameters: `content`: inline content  Returns: [Superscript](https://pandoc.org/lua-filters.html#type-superscript) object

## Element components

- `Attr ([identifier[, classes[, attributes]]])`

  Create a new set of attributes (Attr). Parameters: `identifier`: element identifier `classes`: element classes `attributes`: table containing string keys and values  Returns: [Attr](https://pandoc.org/lua-filters.html#type-attr) object

- `Citation (id, mode[, prefix[, suffix[, note_num[, hash]]]])`

  Creates a single citation. Parameters: `id`: citation identifier (like a bibtex key) `mode`: citation mode `prefix`: citation prefix `suffix`: citation suffix `note_num`: note number `hash`: hash number  Returns: [Citation](https://pandoc.org/lua-filters.html#type-citation) object

- `ListAttributes ([start[, style[, delimiter]]])`

  Creates a set of list attributes. Parameters: `start`: number of the first list item `style`: style used for list numbering `delimiter`: delimiter of list numbers  Returns: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes) object

## Constants

- `AuthorInText`

  Author name is mentioned in the text. See also: [Citation](https://pandoc.org/lua-filters.html#type-citation)

- `SuppressAuthor`

  Author name is suppressed. See also: [Citation](https://pandoc.org/lua-filters.html#type-citation)

- `NormalCitation`

  Default citation style is used. See also: [Citation](https://pandoc.org/lua-filters.html#type-citation)

- `AlignLeft`

  Table cells aligned left. See also: [Table](https://pandoc.org/lua-filters.html#type-alignment)

- `AlignRight`

  Table cells right-aligned. See also: [Table](https://pandoc.org/lua-filters.html#type-alignment)

- `AlignCenter`

  Table cell content is centered. See also: [Table](https://pandoc.org/lua-filters.html#type-alignment)

- `AlignDefault`

  Table cells are alignment is unaltered. See also: [Table](https://pandoc.org/lua-filters.html#type-alignment)

- `DefaultDelim`

  Default list number delimiters are used. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `Period`

  List numbers are delimited by a period. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `OneParen`

  List numbers are delimited by a single parenthesis. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `TwoParens`

  List numbers are delimited by a double parentheses. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `DefaultStyle`

  List are numbered in the default style See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `Example`

  List items are numbered as examples. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `Decimal`

  List are numbered using decimal integers. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `LowerRoman`

  List are numbered using lower-case roman numerals. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `UpperRoman`

  List are numbered using upper-case roman numerals See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `LowerAlpha`

  List are numbered using lower-case alphabetic characters. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `UpperAlpha`

  List are numbered using upper-case alphabetic characters. See also: [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes)

- `sha1`

  Alias for [`pandoc.utils.sha1`](https://pandoc.org/lua-filters.html#pandoc.utils.sha1) (DEPRECATED).

## Helper functions

### pipe

```lua
pipe (command, args, input)
```

Runs command with arguments, passing it some input, and returns the output.

Returns:

- Output of command.

Raises:

- A table containing the keys `command`, `error_code`, and `output` is thrown if the command exits with a non-zero error code.

Usage:

```lua
local output = pandoc.pipe("sed", {"-e","s/a/b/"}, "abc")
```

### walk_block

```lua
walk_block (element, filter)
```

Застосовує фільтр всередині елемента блоку, просуваючи його вміст. 

Parameters:

- `element`: блочний елемент

- `filter`: фільтер Lua (таблиця функцій) підлягає застосуванню в межах блочного елемента 

Повертає: перетворений блочний елемент

### walk_inline

```lua
walk_inline (element, filter)
```

Застосовуйте фільтр всередині вбудованого елемента, просуваючи його вміст.

Parameters:

- `element`:

  the inline element

- `filter`:

  a Lua filter (table of functions) to be applied within the inline element

Returns: the transformed inline element

### read

```lua
read (markup[, format])
```

Parse the given string into a Pandoc document.

Parameters:

- `markup`:

  the markup to be parsed

- `format`:

  format specification, defaults to `"markdown"`.

Returns: pandoc document

Usage:

```lua
local org_markup = "/emphasis/"  -- Input to be read
local document = pandoc.read(org_markup, "org")
-- Get the first block of the document
local block = document.blocks[1]
-- The inline element in that block is an `Emph`
assert(block.content[1].t == "Emph")
```

# Module pandoc.utils

Цей модуль розкриває внутрішні функції pandoc та корисні функції.

Модуль завантажується як частина модуля `pandoc` і доступний як ` pandoc.utils`. У версіях, що включають pandoc 2.6, цей модуль потрібно було явно завантажувати. Приклад:

```lua
pandoc.utils = require 'pandoc.utils'
```

Використовуйте вищезазначене для зворотної сумісності.

### blocks_to_inlines

```lua
blocks_to_inlines (blocks[, sep])
```

Squash a list of blocks into a list of inlines. Parameters:

- `blocks`: List of [Blocks](https://pandoc.org/lua-filters.html#type-block) to be flattened.

- `sep`: List of [Inlines](https://pandoc.org/lua-filters.html#type-inline) inserted as separator between two consecutive blocks; defaults to `{ pandoc.Space(), pandoc.Str'¶', pandoc.Space()}`.

Returns:

- [List](https://pandoc.org/lua-filters.html#type-list) of [Inlines](https://pandoc.org/lua-filters.html#type-inline)

Usage:

```lua
local blocks = {
  pandoc.Para{ pandoc.Str 'Paragraph1' },
  pandoc.Para{ pandoc.Emph 'Paragraph2' }
}
local inlines = pandoc.utils.blocks_to_inlines(blocks)
-- inlines = {
--   pandoc.Str 'Paragraph1',
--   pandoc.Space(), pandoc.Str'¶', pandoc.Space(),
--   pandoc.Emph{ pandoc.Str 'Paragraph2' }
-- }
```

### equals

```lua
equals (element1, element2)
```

Test equality of AST elements. Elements in Lua are considered equal  if and only if the objects obtained by unmarshaling are equal. Parameters:

- `element1`, `element2`: Objects to be compared. Acceptable input types are [Pandoc](https://pandoc.org/lua-filters.html#type-pandoc), [Meta](https://pandoc.org/lua-filters.html#type-meta), [MetaValue](https://pandoc.org/lua-filters.html#type-metavalue), [Block](https://pandoc.org/lua-filters.html#type-block), [Inline](https://pandoc.org/lua-filters.html#type-inline), [Attr](https://pandoc.org/lua-filters.html#type-attr), [ListAttributes](https://pandoc.org/lua-filters.html#type-listattributes), and [Citation](https://pandoc.org/lua-filters.html#type-citation).

Returns:

- Whether the two objects represent the same element (boolean)

### make_sections

```lua
make_sections (number_sections, base_level, blocks)
```

Converst list of [Blocks](https://pandoc.org/lua-filters.html#type-block) into sections. `Div`s will be created beginning at each `Header` and containing following content until the next `Header` of comparable level. If `number_sections` is true, a `number` attribute will be added to each `Header` containing the section number. If `base_level` is non-null, `Header` levels will be reorganized so that there are no gaps, and so that the base level is the level specified.

Returns:

- List of [Blocks](https://pandoc.org/lua-filters.html#type-block).

Usage:

```lua
local blocks = {
  pandoc.Header(2, pandoc.Str 'first'),
  pandoc.Header(2, pandoc.Str 'second'),
}
local newblocks = pandoc.utils.make_sections(true, 1, blocks)
```

### run_json_filter

```lua
run_json_filter (doc, filter[, args])
```

Filter the given doc by passing it through the a JSON filter.

Parameters:

- `doc`: the Pandoc document to filter

- `filter`: filter to run

- `args`: list of arguments passed to the filter. Defaults to `{FORMAT}`.

Returns:

- ([Pandoc](https://pandoc.org/lua-filters.html#type-pandoc)) Filtered document

Usage:

```lua
-- Assumes `some_blocks` contains blocks for which a
-- separate literature section is required.
local sub_doc = pandoc.Pandoc(some_blocks, metadata)
sub_doc_with_bib = pandoc.utils.run_json_filter(
  sub_doc,
  'pandoc-citeproc'
)
some_blocks = sub_doc.blocks -- some blocks with bib
```

### normalize_date

```lua
normalize_date (date_string)
```

Parse a date and convert (if possible) to “YYYY-MM-DD” format. We  limit years to the range 1601-9999 (ISO 8601 accepts greater than or  equal to 1583, but MS Word only accepts dates starting 1601).

Returns:

- A date string, or nil when the conversion failed.

### sha1

```lua
sha1 (contents)
```

Returns the SHA1 has of the contents.

Returns:

- SHA1 hash of the contents.

Usage:

```lua
local fp = pandoc.utils.sha1("foobar")
```

### stringify

```lua
stringify (element)
```

Converts the given element (Pandoc, Meta, Block, or Inline) into a string with all formatting removed.

Returns:

- A plain string representation of the given element.

Usage:

```lua
local inline = pandoc.Emph{pandoc.Str 'Moin'}
-- outputs "Moin"
print(pandoc.utils.stringify(inline))
```

### to_roman_numeral

```lua
to_roman_numeral (integer)
```

Converts an integer < 4000 to uppercase roman numeral.

Returns:

- A roman numeral string.

Usage:

```lua
local to_roman_numeral = pandoc.utils.to_roman_numeral
local pandoc_birth_year = to_roman_numeral(2006)
-- pandoc_birth_year == 'MMVI'
```

# Module pandoc.mediabag

The `pandoc.mediabag` module allows accessing pandoc’s media storage. The “media bag” is used when pandoc is called with the `--extract-media` or (for HTML only) `--self-contained` option.

The module is loaded as part of module `pandoc` and can either be accessed via the `pandoc.mediabag` field, or explicitly required, e.g.:

```lua
local mb = require 'pandoc.mediabag'
```

### delete

```lua
delete (filepath)
```

Removes a single entry from the media bag.

Parameters:

- `filepath`:

  filename of the item to be deleted. The media bag will be left unchanged if no entry with the given filename exists.

### empty

```lua
empty ()
```

Clear-out the media bag, deleting all items.

### insert

```lua
insert (filepath, mime_type, contents)
```

Adds a new entry to pandoc’s media bag.

Parameters:

- `filepath`:

  filename and path relative to the output folder.

- `mime_type`:

  the file’s MIME type

- `contents`:

  the binary contents of the file.

Usage:

```lua
local fp = "media/hello.txt"
local mt = "text/plain"
local contents = "Hello, World!"
pandoc.mediabag.insert(fp, mt, contents)
```

### items

```lua
items ()
```

Returns an iterator triple to be used with Lua’s generic `for` statement. The iterator returns the filepath, MIME type, and content of a media bag item on each invocation. Items are processed one-by-one to  avoid excessive memory use.

This function should be used only when full access to all items, including their contents, is required. For all other cases, [`list`](https://pandoc.org/lua-filters.html#pandoc.mediabag.list) should be preferred.

Returns:

- The iterator function; must be called with the iterator state and the current iterator value.
- Iterator state – an opaque value to be passed to the iterator function.
- Initial iterator value.

Usage:

```lua
for fp, mt, contents in pandoc.mediabag.items() do
  -- print(fp, mt, contents)
end
```

### list

```lua
list ()
```

Get a summary of the current media bag contents.

Returns: A list of elements summarizing each entry in the media bag. The summary item contains the keys `path`, `type`, and `length`, giving the filepath, MIME type, and length of contents in bytes, respectively.

Usage:

```lua
-- calculate the size of the media bag.
local mb_items = pandoc.mediabag.list()
local sum = 0
for i = 1, #mb_items do
    sum = sum + mb_items[i].length
end
print(sum)
```

### lookup

```lua
lookup (filepath)
```

Lookup a media item in the media bag, and return its MIME type and contents.

Parameters:

- `filepath`:

  name of the file to look up.

Returns:

- the entry’s MIME type, or nil if the file was not found.
- contents of the file, or nil if the file was not found.

Usage:

```lua
local filename = "media/diagram.png"
local mt, contents = pandoc.mediabag.lookup(filename)
```

### fetch

```lua
fetch (source, base_url)
```

Fetches the given source from a URL or local file. Returns two  values: the contents of the file and the MIME type (or an empty string).

Returns:

- the entries MIME type, or nil if the file was not found.
- contents of the file, or nil if the file was not found.

Usage:

```lua
local diagram_url = "https://pandoc.org/diagram.jpg"
local mt, contents = pandoc.mediabag.fetch(diagram_url, ".")
```

# Module pandoc.List

The this module defines pandoc’s list type. It comes with useful methods and convenience functions.

## Constructor

- `pandoc.List([table])`

  Create a new List. If the optional argument `table` is given, set the metatable of that value to `pandoc.List`. This is an alias for [`pandoc.List:new([table\])`](https://pandoc.org/lua-filters.html#pandoc.list:new).

## Metamethods

- `pandoc.List:__concat (list)`

  Concatenates two lists. Parameters: `list`: second list concatenated to the first  Returns: a new list containing all elements from list1 and list2

## Methods

- `pandoc.List:clone ()`

  Returns a (shallow) copy of the list.

- `pandoc.List:extend (list)`

  Adds the given list to the end of this list. Parameters: `list`: list to appended

- `pandoc.List:find (needle, init)`

  Returns the value and index of the first occurrence of the given item. Parameters: `needle`: item to search for `init`: index at which the search is started  Returns: first item equal to the needle, or nil if no such item exists.

- `pandoc.List:find_if (pred, init)`

  Returns the value and index of the first element for which the predicate holds true. Parameters: `pred`: the predicate function `init`: index at which the search is started  Returns: first item for which `test` succeeds, or nil if no such item exists.

- `pandoc.List:filter (pred)`

  Returns a new list containing all items satisfying a given condition. Parameters: `pred`: condition items must satisfy.  Returns: a new list containing all items for which `test` was true.

- `pandoc.List:includes (needle, init)`

  Checks if the list has an item equal to the given needle. Parameters: `needle`: item to search for `init`: index at which the search is started  Returns: true if a list item is equal to the needle, false otherwise

- `pandoc.List:insert ([pos], value)`

  Inserts element `value` at position `pos` in list, shifting elements to the next-greater indix if necessary. This function is identical to [`table.insert`](https://www.lua.org/manual/5.3/manual.html#6.6). Parameters: `pos`: index of the new value; defaults to length of the list + 1 `value`: value to insert into the list

- `pandoc.List:map (fn)`

  Returns a copy of the current list by applying the given function to all elements. Parameters: `fn`: function which is applied to all list items.

- `pandoc.List:new([table])`

  Create a new List. If the optional argument `table` is given, set the metatable of that value to `pandoc.List`. Parameters: `table`: table which should be treatable as a list; defaults to an empty table  Returns: the updated input value

- `pandoc.List:remove ([pos])`

  Removes the element at position `pos`, returning the value of the removed element. This function is identical to [`table.remove`](https://www.lua.org/manual/5.3/manual.html#6.6). Parameters: `pos`: position of the list value that will be remove; defaults to the index of the last element  Returns: the removed element

- `pandoc.List:sort ([comp])`

  Sorts list elements in a given order, in-place. If `comp` is given, then it must be a function that receives two list elements  and returns true when the first element must come before the second in  the final order (so that, after the sort, `i < j` implies `not comp(list[j],list[i]))`. If comp is not given, then the standard Lua operator `<` is used instead. Note that the comp function must define a strict partial order over  the elements in the list; that is, it must be asymmetric and transitive. Otherwise, no valid sort may be possible. The sort algorithm is not stable: elements considered equal by the  given order may have their relative positions changed by the sort. This function is identical to [`table.sort`](https://www.lua.org/manual/5.3/manual.html#6.6). Parameters: `comp`: Comparison function as described above.

# Module pandoc.system

Access to system information and functionality.

## Static Fields

### arch

The machine architecture on which the program is running.

### os

The operating system on which the program is running.

## Functions

### environment

```lua
environment ()
```

Retrieve the entire environment as a string-indexed table.

Returns:

- A table mapping environment variables names to their string value (table).

### get_working_directory

```lua
get_working_directory ()
```

Obtain the current working directory as an absolute path.

Returns:

- The current working directory (string).

### with_environment

```lua
with_environment (environment, callback)
```

Run an action within a custom environment. Only the environment variables given by `environment` will be set, when `callback` is called. The original environment is restored after this function  finishes, even if an error occurs while running the callback action.

Parameters:

- `environment`

  Environment variables and their values to be set before running `callback`. (table with string keys and string values)

- `callback`

  Action to execute in the custom environment (function)

Returns:

- The result(s) of the call to `callback`

### with_temporary_directory

```lua
with_temporary_directory ([parent_dir,] templ, callback)
```

Create and use a temporary directory inside the given directory. The directory is deleted after the callback returns.

Parameters:

- `parent_dir`

  Parent directory to create the directory in (string). If this  parameter is omitted, the system’s canonical temporary directory is  used.

- `templ`

  Directory name template (string).

- `callback`

  Function which takes the name of the temporary directory as its first argument (function).

Returns:

- The result of the call to `callback`.

### with_working_directory

```lua
with_working_directory (directory, callback)
```

Run an action within a different directory. This function will change the working directory to `directory`, execute `callback`, then switch back to the original working directory, even if an error occurs while running the callback action.

Parameters:

- `directory`

  Directory in which the given `callback` should be executed (string)

- `callback`

  Action to execute in the given directory (function)

Returns:

- The result(s) of the call to `callback`

# Module pandoc.types

Constructors for types which are not part of the pandoc AST.

### Version

```lua
Version (version_specifier)
```

Creates a Version object.

Parameters:

- `version_specifier`:

  Version specifier: this can be a version string like `'2.7.3'`, a list of integers like `{2, 7, 3}`, a single integer, or a [Version](https://pandoc.org/lua-filters.html#type-version).

Returns:

- A new [Version](https://pandoc.org/lua-filters.html#type-version) object.