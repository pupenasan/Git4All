## Синтаксис шалону (Template syntax)

### Кментарі

Все, що між послідовністю `$--` та кінцем рядка, буде розглядатися як коментар та опускатися з виводу.

### Роздільники (Delimiters)

Для позначення змінних та керуючих структур у шаблоні в якості роздільників можуть використовуватися `$`… `$` або `${` ... `}`. Стилі роздільників можуть бути змішані в одному шаблоні, але роздільник для відкривання та закриття повинен відповідати в кожному випадку. Відкриваючий роздільник може супроводжуватися одним або декількома пробілами чи табами, які будуть ігноровані. Роздільник, що закривається, може супроводжуватися одним або кількома пробілами чи табами, які будуть ігноровані.

Щоб включити літерал `$` в документ, використовуйте `$$`.

### Інтерпольовані змінні (Interpolated variables)

Слот для інтерпольованої змінної - це назва змінної, оточеної відповідними роздільниками. Імена змінних повинні починатися з літери і можуть містити літери, цифри, `_`,` -` і `.`. Ключові слова `it`,` if`, `else`,` endif`, `for`,` sep` і `endfor` не можуть використовуватися як імена змінних. Приклади:

```
$foo$
$foo.bar.baz$
$foo_bar.baz-bim$
$ foo $
${foo}
${foo.bar.baz}
${foo_bar.baz-bim}
${ foo }
```

Імена змінних з періодами використовуються для отримання структурованих змінних значень. Так, наприклад, `employee.salary`  поверне значення поля `salary` об'єкта, яке є значенням поля  `employee` .

- Якщо значення змінної є простим значенням, воно буде надано дослівно. (Зверніть увагу на те, що ніяких escaping не робиться; припущення полягає в тому, що програма, що викликає, буде escape з рядків відповідним чином для вихідного формату.)

- Якщо значення є списком (list), значення будуть об'єднані.
- Якщо значення є картою (map), буде виведений рядок `true` .
- Кожне інше значення відображатиметься як порожній рядок.

### Умови (Conditionals)

Умова починається з  `if(variable)` (укладається у відповідні роздільники) і закінчується  `endif`  (додається до збіжених роздільників). Він необов'язково може містити  `else` (укладений у відповідні роздільники). Розділ `if` використовується, якщо `variable` має значення, яке не є порожнім, інакше використовується розділ `else` (якщо він присутній). Приклади:

```
$if(foo)$bar$endif$

$if(foo)$
  $foo$
$endif$

$if(foo)$
part one
$else$
part two
$endif$

${if(foo)}bar${endif}

${if(foo)}
  ${foo}
${endif}

${if(foo)}
${ foo.bar }
${else}
no foo!
${endif}
```

Ключове слово `elseif` може використовуватися для спрощення складних вкладених умов:

```
$if(foo)$
XXX
$elseif(bar)$
YYY
$else$
ZZZ
$endif$
```

### Цикли For 

Цикл починається з `for(variable)` (укладений у відповідні роздільники) і закінчується на `endfor` (укладений у відповідні роздільники.

- Якщо `variable` є масив, матеріал всередині циклу буде оцінюватися повторно, при цьому `variable` буде встановлена для кожного значення масиву по черзі і об'єднана.

- Якщо `variable` є карта (map), матеріал всередині буде встановлений на карту (map).
- Якщо значенням асоційованої змінної не є масив чи карта, буде виконана одна ітерація її значення.

Приклади:

```
$for(foo)$$foo$$sep$, $endfor$

$for(foo)$
  - $foo.last$, $foo.first$
$endfor$

${ for(foo.bar) }
  - ${ foo.bar.last }, ${ foo.bar.first }
${ endfor }

$for(mymap)$
$it.name$: $it.office$
$endfor$
```

Ви можете опціонально вказати роздільник між послідовними значеннями, використовуючи `sep` (укладений у відповідні роздільники). Матеріал між `sep` та ` endfor` є роздільником.

```
${ for(foo) }${ foo }${ sep }, ${ endfor }
```

Замість використання `variable`  всередині циклу може використовуватися спеціальне анафоричне ключове слово `it`.

```
${ for(foo.bar) }
  - ${ it.last }, ${ it.first }
${ endfor }
```

### Partials

Партіали (підшаблони, що зберігаються в різних файлах) можуть бути включені за допомогою синтаксису

```
${ boilerplate() }
```

Парітали шукатимуться в каталозі, що містить основний шаблон, і вважатиметься, що він має те ж розширення, що і основний шаблон, якщо не вказане явне розширення. (Якщо тут не знайдено партіалу, він буде шукатисяу підкаталозі `templates` каталогу даних користувача)

Партіали можуть необов'язково застосовуватися до змінних за допомогою двокрапки:

```
${ date:fancy() }

${ articles:bibentry() }
```

Якщо `articles`  - масив, це повторюватиме його значення, застосовуючи партіальне ` bibentry() ` до кожного. Отже, другий приклад вище еквівалентний до

```
${ for(articles) }
${ it:bibentry() }
${ endfor }
```

Зауважимо, що анафоричне ключове слово `it` повинно використовуватися при повторенні над партіалами. У наведених вище прикладах партіал`bibentry` повинен містити ` it.title` (і так далі) замість `articles.title`.

Остаточні рядки опускаються з включених партіалів.

До партіалів можуть входити інші партіали.

Роздільник між значеннями масиву може бути вказаний у квадратних дужках, відразу після імені змінної або партіалу:

```
${months[, ]}$

${articles:bibentry()[; ]$
```

Розділювач у цьому випадку є літеральним (на відміну від `sep` в явному циклі ` for`) не може містити інтерпольовані змінні чи інші директиви шаблону.

### Вкладення (Nesting)

To ensure that content is “nested,” that is, subsequent lines indented, use the `^` directive:

Щоб переконатися, що вміст "вкладений", тобто є результатом рядків з відступом, використовуйте директиву `^`:

```
$item.number$  $^$$item.description$ ($item.price$)
```

In this example, if `item.description` has multiple lines, they will all be indented to line up with the first line:

```
00123  A fine bottle of 18-year old
       Oban whiskey. ($148)
```

To nest multiple lines to the same level, align them with the `^` directive in the template. For example:

```
$item.number$  $^$$item.description$ ($item.price$)
               (Available til $item.sellby$.)
```

will produce

```
00123  A fine bottle of 18-year old
       Oban whiskey. ($148)
       (Available til March 30, 2020.)
```

If a variable occurs by itself on a line, preceded by whitespace and  not followed by further text or directives on the same line, and the  variable’s value contains multiple lines, it will be nested  automatically.

### Breakable spaces

Normally, spaces in the template itself (as opposed to values of the  interpolated variables) are not breakable, but they can be made  breakable in part of the template by using the `~` keyword (ended with another `~`).

```
$~$This long line may break if the document is rendered
with a short line length.$~$
```

### Pipes

A pipe transforms the value of a variable or partial. Pipes are specified using a slash (`/`) between the variable name (or partial) and the pipe name. Example:

Труба (pipe) перетворює значення змінної або партіалу. Труби задаються за допомогою косої (`/`) між назвою змінної (або часткової) та назвою труби. Приклад:

```
$for(name)$
$name/uppercase$
$endfor$

$for(metadata/pairs)$
- $it.key$: $it.value$
$endfor$

$employee:name()/uppercase$
```

Pipes may be chained:

```
$for(employees/pairs)$
$it.key/alpha/uppercase$. $it.name$
$endfor$
```

Some pipes take parameters:

```
|----------------------|------------|
$for(employee)$
$it.name.first/uppercase/left 20 "| "$$it.name.salary/right 10 " | " " |"$
$endfor$
|----------------------|------------|
```

В даний час такі труби визначені заздалегідь:

- `pairs`: Перетворює карту (map) або масив у масив карт,  кожна з полями `key` та  `value` . Якщо вихідним значенням був масив, `key`  буде індексом масиву, починаючи з 1.
- `uppercase`: перетворює текст у верхній регістр 
- `lowercase`: перетворює текст у нижній регістр 
- `length`: Повертає довжину значення: кількість символів для текстового значення, кількість елементів для карти чи масиву.
- `reverse`: Повертає текстове значення або масив і не впливає на інші значення.
- `chomp`: Вилучає останні лінії (та нерозривний простір).
- `nowrap`:  Вимикає обгортання ліній на проривних місцях.
- `alpha`: Converts textual values that can be read as an integer into lowercase alphabetic characters `a..z` (mod 26). This can be used to get lettered enumeration from array indices. To get uppercase letters, chain with `uppercase`.
- `roman`: Converts textual  values that can be read as an integer into lowercase roman numerials.  This can be used to get lettered enumeration from array indices. To get  uppercase roman, chain with `uppercase`.
- `left n "leftborder" "rightborder"`: Renders a textual value in a block of width `n`, aligned to the left, with an optional left and right border. Has no  effect on other values. This can be used to align material in tables.  Widths are positive integers indicating the number of characters.  Borders are strings inside double quotes; literal `"` and `\` characters must be backslash-escaped.
- `right n "leftborder" "rightborder"`: Renders a textual value in a block of width `n`, aligned to the right, and has no effect on other values.
- `center n "leftborder" "rightborder"`: Renders a textual value in a block of width `n`, aligned to the center, and has no effect on other values.
