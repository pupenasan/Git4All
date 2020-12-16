## Створення PDF

[Не надто короткий вступ до LaTex](http://mirror.datacenter.by/pub/mirrors/CTAN/info/lshort/ukrainian/lshort-ukr.pdf)

Щоб створити PDF, вкажіть вихідний файл із розширенням `.pdf`:

```
pandoc test.txt -o test.pdf
```

За замовчуванням pandoc використовуватиме LaTeX для створення PDF-файлу, який вимагає встановлення рушія LaTeX (див. [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine) нижче). Крім того, pandoc може використовувати ConTeXt, roff ms або HTML як проміжний формат. Для цього вкажіть вихідний файл із розширенням `.pdf`, як і раніше, але додайте до командного рядка опцію [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine) або [`-t context`](https://pandoc.org/MANUAL.html#option--to), [`-t html`](https://pandoc.org/MANUAL.html#option--to), або [`-t ms`](https://pandoc.org/MANUAL.html#option--to). Інструмент, що використовується для створення PDF з проміжного формату, може бути вказаний за допомогою [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine).

Ви можете керувати стилем PDF за допомогою змінних, залежно від використовуваного проміжного формату: див. [variables for LaTeX](https://pandoc.org/MANUAL.html#variables-for-latex), [variables for ConTeXt](https://pandoc.org/MANUAL.html#variables-for-context), [variables for `wkhtmltopdf`](https://pandoc.org/MANUAL.html#variables-for-wkhtmltopdf), [variables for ms](https://pandoc.org/MANUAL.html#variables-for-ms). Коли HTML використовується як проміжний формат, вихід може бути стилізований за допомогою [`--css`](https://pandoc.org/MANUAL.html#option--css).

Для налагодження створення PDF-файлів може бути корисним переглянути проміжне представлення: замість  [`-o test.pdf`](https://pandoc.org/MANUAL.html#option--output), використовуйте, наприклад, [`-s -o test.tex`](https://pandoc.org/MANUAL.html#option--standalone) для виведення створеного LaTeX. Потім ви можете перевірити його за допомогою `pdflatex test.tex`.

Під час використання LaTeX повинні бути доступні наступні пакети (вони включені до всіх останніх версій [TeX Live](https://www.tug.org/texlive/)): [`amsfonts`](https://ctan.org/pkg/amsfonts), [`amsmath`](https://ctan.org/pkg/amsmath), [`lm`](https://ctan.org/pkg/lm), [`unicode-math`](https://ctan.org/pkg/unicode-math), [`ifxetex`](https://ctan.org/pkg/ifxetex), [`ifluatex`](https://ctan.org/pkg/ifluatex), [`listings`](https://ctan.org/pkg/listings) (якщо використовується опція [`--listings`](https://pandoc.org/MANUAL.html#option--listings)), [`fancyvrb`](https://ctan.org/pkg/fancyvrb), [`longtable`](https://ctan.org/pkg/longtable), [`booktabs`](https://ctan.org/pkg/booktabs), [`graphicx`](https://ctan.org/pkg/graphicx) (якщо документ містить рисунки), [`hyperref`](https://ctan.org/pkg/hyperref), [`xcolor`](https://ctan.org/pkg/xcolor), [`ulem`](https://ctan.org/pkg/ulem), [`geometry`](https://ctan.org/pkg/geometry) (з встановленням змінної `geometry`), [`setspace`](https://ctan.org/pkg/setspace) (з `linestretch`), та [`babel`](https://ctan.org/pkg/babel) (з`lang`). 

Використання `xelatex` або `lualatex` як рушія PDF потребує [`fontspec`](https://ctan.org/pkg/fontspec). `xelatex` використовує [`polyglossia`](https://ctan.org/pkg/polyglossia) (з`lang`), [`xecjk`](https://ctan.org/pkg/xecjk), та [`bidi`](https://ctan.org/pkg/bidi) (з встановленною змінною `dir`). Якщо встановлена змінна `mathspec`, `xelatex` буде використовувати [`mathspec`](https://ctan.org/pkg/mathspec) замість [`unicode-math`](https://ctan.org/pkg/unicode-math). Пакети [`upquote`](https://ctan.org/pkg/upquote) і [`microtype`](https://ctan.org/pkg/microtype) використовуються якщо доступні, і [`csquotes`](https://ctan.org/pkg/csquotes) будуть використовуватися для [typography](https://pandoc.org/MANUAL.html#typography) якщо змінна `csquotes` або поле метаданих будуть встановлені в true. Пакунки [`natbib`](https://ctan.org/pkg/natbib), [`biblatex`](https://ctan.org/pkg/biblatex), [`bibtex`](https://ctan.org/pkg/bibtex), та [`biber`](https://ctan.org/pkg/biber) можуть бути опціонально використані для [надання цитування](https://pandoc.org/MANUAL.html#citation-rendering). Наступні пакети будуть використані для поліпшення якості виводу, якщо вони є, але pandoc не вимагає їх присутності: [`upquote`](https://ctan.org/pkg/upquote) (для прямих цитат у дослівному середовищі), [`microtype`](https://ctan.org/pkg/microtype) (для кращого регулювання інтервалу), [`parskip`](https://ctan.org/pkg/parskip) (для кращих проміжків між абзацами), [`xurl`](https://ctan.org/pkg/xurl) (для кращих розривів рядків у URL-адресах), [`bookmark`](https://ctan.org/pkg/bookmark) (для кращих PDF-закладок), та [`footnotehyper`](https://ctan.org/pkg/footnotehyper) або [`footnote`](https://ctan.org/pkg/footnote) (щоб дозволити виноски в таблицях).

### Metadata variables

- `title`, `author`, `date`

  allow identification of basic aspects of the document. Included  in PDF metadata through LaTeX and ConTeXt. These can be set through a [pandoc title block](https://pandoc.org/MANUAL.html#extension-pandoc_title_block), which allows for multiple authors, or through a [YAML metadata block](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block): `--- author: - Aristotle - Peter Abelard ...` Note that if you just want to set PDF or HTML metadata, without including a title block in the document itself, you can set the `title-meta`, `author-meta`, and `date-meta` variables. (By default these are set automatically, based on `title`, `author`, and `date`.)

- `subtitle`

  document subtitle, included in HTML, EPUB, LaTeX, ConTeXt, and docx documents

- `abstract`

  document summary, included in LaTeX, ConTeXt, AsciiDoc, and docx documents

- `keywords`

  list of keywords to be included in HTML, PDF, ODT, pptx, docx and AsciiDoc metadata; repeat as for `author`, above

- `subject`

  document subject, included in ODT, PDF, docx and pptx metadata

- `description`

  document description, included in ODT, docx and pptx metadata. Some applications show this as `Comments` metadata.

- `category`

  document category, included in docx and pptx metadata

Additionally, any root-level string metadata, not included in ODT, docx or pptx metadata is added as a *custom property*. The following [YAML](https://yaml.org/spec/1.2/spec.html) metadata block for instance:

```
---
title:  'This is the title'
subtitle: "This is the subtitle"
author:
- Author One
- Author Two
description: |
    This is a long
    description.

    It consists of two paragraphs
...
```

will include `title`, `author` and `description` as standard document properties and `subtitle` as a custom property when converting to docx, ODT or pptx.

### Language variables

- `lang`

  identifies the main language of the document using IETF language tags (following the [BCP 47](https://tools.ietf.org/html/bcp47) standard), such as `en` or `en-GB`. The [Language subtag lookup](https://r12a.github.io/app-subtags/) tool can look up or verify these tags. This affects most formats, and  controls hyphenation in PDF output when using LaTeX (through [`babel`](https://ctan.org/pkg/babel) and [`polyglossia`](https://ctan.org/pkg/polyglossia)) or ConTeXt. 

  Use native pandoc [Divs and Spans](https://pandoc.org/MANUAL.html#divs-and-spans) with the `lang` attribute to switch the language: `--- lang: en-GB ... Text in the main document language (British English). ::: {lang=fr-CA} > Cette citation est écrite en français canadien. ::: More text in English. ['Zitat auf Deutsch.']{lang=de}`

- `dir`

  the base script direction, either `rtl` (right-to-left) or `ltr` (left-to-right). For bidirectional documents, native pandoc `span`s and `div`s with the `dir` attribute (value `rtl` or `ltr`) can be used to override the base direction in some output formats. This may not always be necessary if the final renderer (e.g. the browser,  when generating HTML) supports the [Unicode Bidirectional Algorithm](https://www.w3.org/International/articles/inline-bidi-markup/uba-basics). When using LaTeX for bidirectional documents, only the `xelatex` engine is fully supported (use [`--pdf-engine=xelatex`](https://pandoc.org/MANUAL.html#option--pdf-engine)).

### Variables for LaTeX

Pandoc використовує ці змінні при [створенні PDF](https://pandoc.org/MANUAL.html#creating-a-pdf) з рушієм LaTeX.

#### Layout

- `block-headings`

  make `\paragraph` and `\subparagraph` (fourth- and fifth-level headings, or fifth- and sixth-level with book  classes) free-standing rather than run-in; requires further formatting  to distinguish from `\subsubsection` (third- or fourth-level headings). Instead of using this option, [KOMA-Script](https://ctan.org/pkg/koma-script) can adjust headings more extensively: `--- documentclass: scrartcl header-includes: |  \RedeclareSectionCommand[    beforeskip=-10pt plus -2pt minus -1pt,    afterskip=1sp plus -1sp minus 1sp,    font=\normalfont\itshape]{paragraph}  \RedeclareSectionCommand[    beforeskip=-10pt plus -2pt minus -1pt,    afterskip=1sp plus -1sp minus 1sp,    font=\normalfont\scshape,    indent=0pt]{subparagraph} ...`

- `classoption`

  option for document class, e.g. `oneside`; repeat for multiple options: `--- classoption: - twocolumn - landscape ...`

- `documentclass`

  клас документу: зазвичай один із стандартних класів, [`article`](https://ctan.org/pkg/article), [`book`](https://ctan.org/pkg/book), and [`report`](https://ctan.org/pkg/report); the [KOMA-Script](https://ctan.org/pkg/koma-script) equivalents, `scrartcl`, `scrbook`, and `scrreprt`, which default to smaller margins; or [`memoir`](https://ctan.org/pkg/memoir)

- `geometry`

  option for [`geometry`](https://ctan.org/pkg/geometry) package, e.g. `margin=1in`; repeat for multiple options: `--- geometry: - top=30mm - left=20mm - heightrounded ...`

- `hyperrefoptions`

  option for [`hyperref`](https://ctan.org/pkg/hyperref) package, e.g. `linktoc=all`; repeat for multiple options: `--- hyperrefoptions: - linktoc=all - pdfwindowui - pdfpagemode=FullScreen ...`

- `indent`

  uses document class settings for indentation (the default LaTeX  template otherwise removes indentation and adds space between  paragraphs)

- `linestretch`

  adjusts line spacing using the [`setspace`](https://ctan.org/pkg/setspace) package, e.g. `1.25`, `1.5`

- `margin-left`, `margin-right`, `margin-top`, `margin-bottom`

  sets margins if `geometry` is not used (otherwise `geometry` overrides these)

- `pagestyle`

  control `\pagestyle{}`: the default article class supports `plain` (default), `empty` (no running heads or page numbers), and `headings` (section titles in running heads)

- `papersize`

  paper size, e.g. `letter`, `a4`

- `secnumdepth`

  numbering depth for sections (with [`--number-sections`](https://pandoc.org/MANUAL.html#option--number-sections) option or `numbersections` variable)

#### Fonts

- `fontenc`

  allows font encoding to be specified through `fontenc` package (with `pdflatex`); default is `T1` (see [LaTeX font encodings guide](https://ctan.org/pkg/encguide))

- `fontfamily`

  font package for use with `pdflatex`: [TeX Live](https://www.tug.org/texlive/) includes many options, documented in the [LaTeX Font Catalogue](https://tug.org/FontCatalogue/). The default is [Latin Modern](https://ctan.org/pkg/lm).

- `fontfamilyoptions`

  options for package used as `fontfamily`; repeat for multiple options. For example, to use the Libertine font  with proportional lowercase (old-style) figures through the [`libertinus`](https://ctan.org/pkg/libertinus) package: `--- fontfamily: libertinus fontfamilyoptions: - osf - p ...`

- `fontsize`

  font size for body text. The standard classes allow 10pt, 11pt, and 12pt. To use another size, set `documentclass` to one of the [KOMA-Script](https://ctan.org/pkg/koma-script) classes, such as `scrartcl` or `scrbook`.

- `mainfont`, `sansfont`, `monofont`, `mathfont`, `CJKmainfont`

  font families for use with `xelatex` or `lualatex`: take the name of any system font, using the [`fontspec`](https://ctan.org/pkg/fontspec) package. `CJKmainfont` uses the [`xecjk`](https://ctan.org/pkg/xecjk) package.

- `mainfontoptions`, `sansfontoptions`, `monofontoptions`, `mathfontoptions`, `CJKoptions`

  options to use with `mainfont`, `sansfont`, `monofont`, `mathfont`, `CJKmainfont` in `xelatex` and `lualatex`. Allow for any choices available through [`fontspec`](https://ctan.org/pkg/fontspec); repeat for multiple options. For example, to use the [TeX Gyre](http://www.gust.org.pl/projects/e-foundry/tex-gyre) version of Palatino with lowercase figures: `--- mainfont: TeX Gyre Pagella mainfontoptions: - Numbers=Lowercase - Numbers=Proportional ...`

- `microtypeoptions`

  options to pass to the microtype package

#### Links

- `colorlinks`

  add color to link text; automatically enabled if any of `linkcolor`, `filecolor`, `citecolor`, `urlcolor`, or `toccolor` are set

- `linkcolor`, `filecolor`, `citecolor`, `urlcolor`, `toccolor`

  color for internal links, external links, citation links, linked  URLs, and links in table of contents, respectively: uses options allowed by [`xcolor`](https://ctan.org/pkg/xcolor), including the `dvipsnames`, `svgnames`, and `x11names` lists

- `links-as-notes`

  causes links to be printed as footnotes

#### Front matter

- `lof`, `lot`

  include list of figures, list of tables

- `thanks`

  contents of acknowledgments footnote after document title

- `toc`

  include table of contents (can also be set using [`--toc/--table-of-contents`](https://pandoc.org/MANUAL.html#option--toc))

- `toc-depth`

  level of section to include in table of contents

#### BibLaTeX Bibliographies

These variables function when using BibLaTeX for [citation rendering](https://pandoc.org/MANUAL.html#citation-rendering).

- `biblatexoptions`

  list of options for biblatex

- `biblio-style`

  bibliography style, when used with [`--natbib`](https://pandoc.org/MANUAL.html#option--natbib) and [`--biblatex`](https://pandoc.org/MANUAL.html#option--biblatex).

- `biblio-title`

  bibliography title, when used with [`--natbib`](https://pandoc.org/MANUAL.html#option--natbib) and [`--biblatex`](https://pandoc.org/MANUAL.html#option--biblatex).

- `bibliography`

  bibliography to use for resolving references

- `natbiboptions`

  list of options for natbib

# Шаблони (Templates)

Коли використовується параметр [`-s/--standalone`](https://pandoc.org/MANUAL.html#option--standalone), pandoc використовує шаблон, щоб додати матеріал заголовка та нижнього колонтитула, який необхідний для самостійного документу. Щоб побачити шаблон, що використовується за замовчуванням, просто введіть

```bash
pandoc -D *FORMAT*
```

де *FORMAT* - ім'я вихідного формату. За допомогою параметра [`--template`](https://pandoc.org/MANUAL.html#option--template)  можна вказати  спеціальний шаблон. Також можна змінити системні шаблони за замовчуванням для заданого формату виводу *FORMAT*, помістивши файл  `templates/default.*FORMAT*`  у каталог даних користувачів (див. [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir), вище). *Винятки:*

- Для виходу `odt` , налаштуйте шаблон `default.opendocument`.
- Для виходу`pdf` , налаштуйте шаблон `default.latex` (або: шаблон `default.context`, якщо ви використовуєте [`-t context`](https://pandoc.org/MANUAL.html#option--to); шаблон `default.ms` якщо використовуєте  [`-t ms`](https://pandoc.org/MANUAL.html#option--to); шаблон  `default.html` якщо використовуєте [`-t html`](https://pandoc.org/MANUAL.html#option--to)).
- `docx` and `pptx` не має шаблону (одак для налаштування ви можете використовувати [`--reference-doc`](https://pandoc.org/MANUAL.html#option--reference-doc) ).

Шаблони містять *змінні*, які дозволяють включати довільну інформацію в будь-якій точці файлу. Вони можуть бути встановлені в командному рядку, використовуючи параметр [`-V/--variable`](https://pandoc.org/MANUAL.html#option--variable) . Якщо змінна не встановлена, pandoc шукатиме ключ (key) у метаданих документа, який можна встановити за допомогою  [YAML metadata blocks](https://pandoc.org/MANUAL.html#extension-yaml_metadata_block) або за допомогою параметру [`-M/--metadata`](https://pandoc.org/MANUAL.html#option--metadata). Крім того, деяким змінним pandoc надаються значення за замовчуванням. Див.  [Variables](https://pandoc.org/MANUAL.html#variables)  нижче для переліку змінних, які використовуваних у шаблонах за замовчуванням pandoc.

Якщо ви використовуєте власні шаблони, можливо, вам знадобиться переглянути їх при зміні pandoc. Ми рекомендуємо відстежувати зміни в шаблонах за замовчуванням та відповідно змінювати власні шаблони. Простий спосіб зробити це - зробити форк сховища[pandoc-templates](https://github.com/jgm/pandoc-templates) та об'єднатись (merge) ці зміни після кожного випуску pandoc.

`--template=`*FILE*|*URL*

Use the specified file as a custom template for the generated document. Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone). See [Templates](https://pandoc.org/MANUAL.html#templates), below, for a description of template syntax. If no extension is  specified, an extension corresponding to the writer will be added, so  that [`--template=special`](https://pandoc.org/MANUAL.html#option--template) looks for `special.html` for HTML output. If the template is not found, pandoc will search for it in the `templates` subdirectory of the user data directory (see [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). If this option is not used, a default template appropriate for the output format will be used (see [`-D/--print-default-template`](https://pandoc.org/MANUAL.html#option--print-default-template)).

`-V` *KEY*[`=`*VAL*], `--variable=`*KEY*[`:`*VAL*]

Set the template variable *KEY* to the value *VAL* when rendering the document in standalone mode. If no *VAL* is specified, the key will be given the value `true`.

`-D` *FORMAT*, `--print-default-template=`*FORMAT*

Print the system default template for an output *FORMAT*. (See [`-t`](https://pandoc.org/MANUAL.html#option--to) for a list of possible *FORMAT*s.) Templates in the user data directory are ignored. This option may be used with [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) to redirect output to a file, but [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) must come before [`--print-default-template`](https://pandoc.org/MANUAL.html#option--print-default-template) on the command line.

Note that some of the default templates use partials, for example `styles.html`. To print the partials, use [`--print-default-data-file`](https://pandoc.org/MANUAL.html#option--print-default-data-file): for example, [`--print-default-data-file=templates/styles.html`](https://pandoc.org/MANUAL.html#option--print-default-data-file).

## Синтаксис шаблону (Template syntax)

### Коментарі

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