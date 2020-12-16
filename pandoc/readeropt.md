## Опції зчитувача

[оригінал](https://pandoc.org/MANUAL.html#reader-options)

- `--shift-heading-level-by=`*NUMBER*

  Зсув рівнів заголовка на натуральне чи додатнє ціле число. Наприклад, з [`--shift-heading-level-by=-1`](https://pandoc.org/MANUAL.html#option--shift-heading-level-by), заголовки рівня 2 стають заголовками рівня 1, а заголовки рівня 3 - заголовками рівня 2. Заголовки не можуть мати рівень менший за 1, тому заголовок, який буде зміщений нижче рівня 1, стає звичайним абзацом. Exception: with a shift of -N, a level-N heading at the beginning of the document replaces the metadata title. [`--shift-heading-level-by=-1`](https://pandoc.org/MANUAL.html#option--shift-heading-level-by) is a good choice when converting HTML or Markdown documents that use an initial level-1 heading for the document title and level-2+ headings  for sections. [`--shift-heading-level-by=1`](https://pandoc.org/MANUAL.html#option--shift-heading-level-by) may be a good choice for converting Markdown documents that use level-1 headings for sections to HTML, since pandoc uses a level-1 heading to  render the document title.

- `--indented-code-classes=`*CLASSES*

  Укажіть класи, які слід використовувати для відступних блоків коду - наприклад, `perl,numberLines` або `haskell`. Кілька класів можуть бути розділені пробілами або комами.

- `--default-image-extension=`*EXTENSION*

  Вкажіть розширення за замовчуванням, яке слід використовувати, коли шляхи / URL-адреси зображень не мають розширення. Це дозволяє використовувати одне і те ж джерело для форматів, які потребують різних видів зображень. В даний час ця опція впливає лише на зчитувачі Markdown та LaTeX. 

- `--file-scope`

  Parse each file individually before combining for multifile  documents. This will allow footnotes in different files with the same  identifiers to work as expected. If this option is set, footnotes and  links will not work across files. Reading binary files (docx, odt, epub) implies [`--file-scope`](https://pandoc.org/MANUAL.html#option--file-scope).

- `-F` *PROGRAM*, `--filter=`*PROGRAM*

  Вказує виконуваний файл, який буде використовуватися як фільтр, що перетворює Pandoc AST після розбору входу та перед записом виводу. Виконаний файл повинен читати JSON від stdin і писати JSON в stdout. JSON повинен бути відформатований як власний вхід і вихід JSON pandoc.   Назва вихідного формату буде передано у фільтр як перший аргумент. Звідси, `pandoc --filter ./caps.py -t latex` еквівалентний `pandoc -t json | ./caps.py latex | pandoc -f json -t latex` Остання форма може бути корисною для налагодження фільтрів. Фільтри можуть бути написані будь-якою мовою.  `Text.Pandoc.JSON` експортує `toJSONFilter` для полегшення запису фільтрів у Haskell. Those who would prefer to write filters in python can use the module [`pandocfilters`](https://github.com/jgm/pandocfilters), installable from PyPI. There are also pandoc filter libraries in [PHP](https://github.com/vinai/pandocfilters-php), [perl](https://metacpan.org/pod/Pandoc::Filter), and [JavaScript/node.js](https://github.com/mvhenderson/pandoc-filter-node). In order of preference, pandoc will look for filters in a specified full or relative path (executable or non-executable) `$DATADIR/filters` (executable or non-executable) where `$DATADIR` is the user data directory (see [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir), above). `$PATH` (executable only) Filters and Lua-filters are applied in the order specified on the command line.

- `-L` *SCRIPT*, `--lua-filter=`*SCRIPT*

  Перетворює документ аналогічно фільтрам JSON (див [`--filter`](https://pandoc.org/MANUAL.html#option--filter)), але використовує вбудовану систему фільтрації Lua від Pandoc. Очікується, що даний сценарій Lua поверне список фільтрів Lua, які будуть застосовані в порядку. Кожен фільтр Lua повинен містити функції перетворення елементів, індексовані назвою елемента AST, щодо якого слід застосувати функцію фільтра. 

  Модуль `pandoc` Lua забезпечує допоміжні функції для створення елементів. Він завжди завантажується в середовище Lua сценарію. 

  Нижче наведено приклад сценарію Lua для розширення макросу:  

  ```lua
  function expand_hello_world(inline)
    if inline.c == '{{helloworld}}' then
      return pandoc.Emph{ pandoc.Str "Hello, World" }
    else
      return inline
    end
  end
  
  return {{Str = expand_hello_world}}
  ```

  У порядку переваги pandoc шукатиме фільтри Lua 

  - у визначеному повному або відносному шляху (виконуваний або невиконаний) 
  - `$DATADIR/filters` (виконуваний або невиконаний) де`$DATADIR` - користувацька директорія (див [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir), нижче).

- `-M` *KEY*[`=`*VAL*], `--metadata=`*KEY*[`:`*VAL*]

  Встановлює для поля метаданих *KEY* значення *VAL*. Значення, вказане в командному рядку, переозначує значення, вказане в документі за допомогою [YAML metadata blocks](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block). Значення будуть проаналізовані як булеві або строкові значення YAML. Якщо значення не вказане, це значення буде розглядатися як булева істина. Подібно [`--variable`](https://pandoc.org/MANUAL.html#option--variable), [`--metadata`](https://pandoc.org/MANUAL.html#option--metadata) викликає встановлення змінних шаблонів. But unlike [`--variable`](https://pandoc.org/MANUAL.html#option--variable), [`--metadata`](https://pandoc.org/MANUAL.html#option--metadata) affects the metadata of the underlying document (which is accessible  from filters and may be printed in some output formats) and metadata  values will be escaped when inserted into the template.

- `--metadata-file=`*FILE*

  Read metadata from the supplied YAML (or JSON) file. This option  can be used with every input format, but string scalars in the YAML file will always be parsed as Markdown. Generally, the input will be handled the same as in [YAML metadata blocks](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block). This option can be used repeatedly to include multiple metadata files;  values in files specified later on the command line will be preferred  over those specified in earlier files. Metadata values specified inside  the document, or by using [`-M`](https://pandoc.org/MANUAL.html#option--metadata), overwrite values specified with this option.

- `-p`, `--preserve-tabs`

  Preserve tabs instead of converting them to spaces. (By default,  pandoc converts tabs to spaces before parsing its input.) Note that this will only affect tabs in literal code spans and code blocks. Tabs in  regular text are always treated as spaces.

- `--tab-stop=`*NUMBER*

  Specify the number of spaces per tab (default is 4).

- `--track-changes=accept`|`reject`|`all`

  Specifies what to do with insertions, deletions, and comments produced by the MS Word “Track Changes” feature. `accept` (the default), inserts all insertions, and ignores all deletions. `reject` inserts all deletions and ignores insertions. Both `accept` and `reject` ignore comments. `all` puts in insertions, deletions, and comments, wrapped in spans with `insertion`, `deletion`, `comment-start`, and `comment-end` classes, respectively. The author and time of change is included. `all` is useful for scripting: only accepting changes from a certain  reviewer, say, or before a certain date. If a paragraph is inserted or  deleted, `track-changes=all` produces a span with the class `paragraph-insertion`/`paragraph-deletion` before the affected paragraph break. This option only affects the docx reader.

- `--extract-media=`*DIR*

  Extract images and other media contained in or linked from the source document to the path *DIR*, creating it if necessary, and adjust the images references in the  document so they point to the extracted files. If the source format is a binary container (docx, epub, or odt), the media is extracted from the  container and the original filenames are used. Otherwise the media is  read from the file system or downloaded, and new filenames are  constructed based on SHA1 hashes of the contents.

- `--abbreviations=`*FILE*

  Specifies a custom abbreviations file, with abbreviations one to a line. If this option is not specified, pandoc will read the data file `abbreviations` from the user data directory or fall back on a system default. To see the system default, use `pandoc --print-default-data-file=abbreviations`. The only use pandoc makes of this list is in the Markdown reader.  Strings ending in a period that are found in this list will be followed  by a nonbreaking space, so that the period will not produce  sentence-ending space in formats like LaTeX.