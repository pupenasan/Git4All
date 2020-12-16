# Опис

[оригінал](https://pandoc.org/MANUAL.html#description)

Pandoc - це бібліотека [Haskell](https://www.haskell.org) для перетворення одного формату розмітки в інший та інструмент командного рядка, який використовує цю бібліотеку.

Pandoc може конвертувати між численними форматами розмітки та обробки текстів, включаючи, але не обмежуючись ними, різні смаки [Markdown](https://daringfireball.net/projects/markdown/), [HTML](https: //www.w3.org/html/), [LaTeX](https://www.latex-project.org/) та [Word docx](https://en.wikipedia.org/wiki/Office_Open_XML). Повні списки форматів введення та виведення див. [`--from`](https://pandoc.org/MANUAL.html#option--from) та [`--to`](https://pandoc.org/MANUAL.html#option--to) [опції нижче](https://pandoc.org/MANUAL.html#general-options). Pandoc може також публікувати в [PDF](https://www.adobe.com/pdf/): див [creating a PDF](https://pandoc.org/MANUAL.html#creating-a-pdf), нижче.

Удосконалена версія Pandoc Markdown включає синтаксис для  [таблиць](https://pandoc.org/MANUAL.html#tables), [definition lists](https://pandoc.org/MANUAL.html#definition-lists), [metadata blocks](https://pandoc.org/MANUAL.html#metadata-blocks), [footnotes](https://pandoc.org/MANUAL.html#footnotes), [citations](https://pandoc.org/MANUAL.html#citations), [math](https://pandoc.org/MANUAL.html#math), та ще багато вього. Див нижче в розділі [Pandoc’s Markdown](https://pandoc.org/MANUAL.html#pandocs-markdown).

Pandoc має модульну конструкцію: він складається з набору зчитувачів (readers), які розбирають текст у заданому форматі та створюють нативне представлення документа (*абстрактне синтаксичне дерево* або AST), і набору записувачв (writers), які перетворюють це нативне представлення у цільовому форматі. Таким чином, додавання формату введення або виведення вимагає лише додавання зчитувача чи записувача. Користувачі також можуть запускати користувацькі [фільтри для пандока](https://pandoc.org/filters.html) для зміни проміжного AST.

Оскільки проміжне представлення документа pandoc менш виразне, ніж багато форматів, в які вони перетворюються, не слід очікувати ідеальних перетворень між кожним форматом і будь-яким іншим. Pandoc намагається зберегти структурні елементи документа, але не форматуючи деталі, такі як розмір поля. А деякі елементи документа, такі як складні таблиці, можуть не входити до простої моделі документа pandoc. Незважаючи на те, що конвертування від Markdown Pandoc до всіх форматів прагнуть бути ідеальними, при конвертуванні з форматів, більш виразних, ніж Markdown Pandoc, можна очікувати втрати.

## Використання pandoc

Якщо не вказані *вхідні файли*, вхід зчитується з *stdin*. Вихід за замовчуванням виводиться до *stdout*. Для виводу у файл використовуйте опцію  [`-o`](https://pandoc.org/MANUAL.html#option--output):

```
pandoc -o output.html input.txt
```

За замовчуванням pandoc створює фрагмент документа. Щоб створити окремий документ (наприклад, дійсний файл HTML, включаючи `<head>` та `<body>`), використовуйте прапорці [`-s`](https://pandoc.org/MANUAL.html#option--standalone) або [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone):

```
pandoc -s -o output.html input.txt
```

Для отримання додаткової інформації про те, як створюються окремі документи, див. [Шаблони](https://pandoc.org/MANUAL.html#templates) нижче.

Якщо надано кілька вхідних файлів, `pandoc` об'єднає їх усіх (з порожніми рядками між ними) перед розбором. (Використовуйте  [`--file-scope`](https://pandoc.org/MANUAL.html#option--file-scope)  для розбору файлів окремо.)

## Вказівка форматів

Формат вводу та виводу можна точно вказати за допомогою параметрів командного рядка. Формат вводу можна задати за допомогою параметра [`-f/--from`](https://pandoc.org/MANUAL.html#option--from), формат виведення, використовуючи опцію  [`-t/--to`](https://pandoc.org/MANUAL.html#option--to) . Таким чином, для перетворення `hello.txt` з Markdown в LaTeX можна ввести:

```
pandoc -f markdown -t latex hello.txt
```

Для перетворення `hello.html` з HTML в Markdown:

```
pandoc -f html -t markdown hello.html
```

Підтримувані формати введення та виведення перелічені нижче в розділі  [Options](https://pandoc.org/MANUAL.html#options)  (див.  [`-f`](https://pandoc.org/MANUAL.html#option--from)  для вхідних форматів та  [`-t`](https://pandoc.org/MANUAL.html#option--to) для вихідних форматів). Ви також можете використовувати `pandoc --list-input-формати` та` pandoc --list-output-формати` для друку списків підтримуваних форматів.

Якщо формат вводу або виводу не вказаний явно, `pandoc` намагатиметься відгадати його з розширень назви файлів. Так, наприклад,

```
pandoc -o hello.tex hello.txt
```

перетворить `hello.txt`  з Markdown в LaTeX. Якщо не вказаний вихідний файл (таким чином вихід буде йти до *stdout*) або якщо розширення вихідного файлу невідоме, формат виводу за замовчуванням буде HTML. Якщо не введено жодного вхідного файлу (таким чином вхід буде від *stdin*), або якщо розширення файлів вводу невідомі, формат введення вважатиметься Markdown.

## Кодування символів

Pandoc використовує кодування символів UTF-8 і для вводу, і для виводу. Якщо ваше локальне кодування символів не є UTF-8, вам слід передавати введення та виведення через  [`iconv`](https://www.gnu.org/software/libiconv/):

```
iconv -t utf-8 input.txt | pandoc | iconv -f utf-8
```

Зауважте, що в деяких вихідних форматах (таких як HTML, LaTeX, ConTeXt, RTF, OPML, DocBook і Texinfo) інформація про кодування символів міститься у заголовку документа, який буде включений лише у тому випадку, якщо ви використовуєте опцію [`-s/--standalone`](https://pandoc.org/MANUAL.html#option--standalone) 

## Створення PDF

Щоб створити PDF, вкажіть вихідний файл із розширенням `.pdf`:

```
pandoc test.txt -o test.pdf
```

За замовчуванням pandoc використовуватиме LaTeX для створення PDF-файлу, який вимагає встановлення рушія LaTeX (див. [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine) нижче). Крім того, pandoc може використовувати ConTeXt, roff ms або HTML як проміжний формат. Для цього вкажіть вихідний файл із розширенням `.pdf`, як і раніше, але додайте до командного рядка опцію [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine) або [`-t context`](https://pandoc.org/MANUAL.html#option--to), [`-t html`](https://pandoc.org/MANUAL.html#option--to), або [`-t ms`](https://pandoc.org/MANUAL.html#option--to). Інструмент, що використовується для створення PDF з проміжного формату, може бути вказаний за допомогою [`--pdf-engine`](https://pandoc.org/MANUAL.html#option--pdf-engine).

Ви можете керувати стилем PDF за допомогою змінних, залежно від використовуваного проміжного формату: див. [variables for LaTeX](https://pandoc.org/MANUAL.html#variables-for-latex), [variables for ConTeXt](https://pandoc.org/MANUAL.html#variables-for-context), [variables for `wkhtmltopdf`](https://pandoc.org/MANUAL.html#variables-for-wkhtmltopdf), [variables for ms](https://pandoc.org/MANUAL.html#variables-for-ms). Коли HTML використовується як проміжний формат, вихід може бути стилізований за допомогою [`--css`](https://pandoc.org/MANUAL.html#option--css).

Для налагодження створення PDF-файлів може бути корисним переглянути проміжне представлення: замість  [`-o test.pdf`](https://pandoc.org/MANUAL.html#option--output), використовуйте, наприклад, [`-s -o test.tex`](https://pandoc.org/MANUAL.html#option--standalone) для виведення створеного LaTeX. Потім ви можете перевірити його за допомогою `pdflatex test.tex`.

Під час використання LaTeX повинні бути доступні наступні пакети (вони включені до всіх останніх версій [TeX Live](https://www.tug.org/texlive/)): [`amsfonts`](https://ctan.org/pkg/amsfonts), [`amsmath`](https://ctan.org/pkg/amsmath), [`lm`](https://ctan.org/pkg/lm), [`unicode-math`](https://ctan.org/pkg/unicode-math), [`ifxetex`](https://ctan.org/pkg/ifxetex), [`ifluatex`](https://ctan.org/pkg/ifluatex), [`listings`](https://ctan.org/pkg/listings) (якщо використовується опція [`--listings`](https://pandoc.org/MANUAL.html#option--listings)), [`fancyvrb`](https://ctan.org/pkg/fancyvrb), [`longtable`](https://ctan.org/pkg/longtable), [`booktabs`](https://ctan.org/pkg/booktabs), [`graphicx`](https://ctan.org/pkg/graphicx) (якщо документ містить рисунки), [`hyperref`](https://ctan.org/pkg/hyperref), [`xcolor`](https://ctan.org/pkg/xcolor), [`ulem`](https://ctan.org/pkg/ulem), [`geometry`](https://ctan.org/pkg/geometry) (з встановленням змінної `geometry`), [`setspace`](https://ctan.org/pkg/setspace) (з`linestretch`), та [`babel`](https://ctan.org/pkg/babel) (з`lang`). Використання `xelatex` або`lualatex` як рушія PDF потребує [`fontspec`](https://ctan.org/pkg/fontspec). `xelatex` використовує [`polyglossia`](https://ctan.org/pkg/polyglossia) (з`lang`), [`xecjk`](https://ctan.org/pkg/xecjk), та [`bidi`](https://ctan.org/pkg/bidi) (з встановленною змінною `dir`). Якщо встановлена змінна `mathspec`, `xelatex` буде використовувати [`mathspec`](https://ctan.org/pkg/mathspec) замість [`unicode-math`](https://ctan.org/pkg/unicode-math). Пакети [`upquote`](https://ctan.org/pkg/upquote) і [`microtype`](https://ctan.org/pkg/microtype) використовуються якщо доступні, і [`csquotes`](https://ctan.org/pkg/csquotes) будуть використовуватися для [typography](https://pandoc.org/MANUAL.html#typography) якщо змінна `csquotes` або поле метаданих будуть встановлені в true. Пакунки [`natbib`](https://ctan.org/pkg/natbib), [`biblatex`](https://ctan.org/pkg/biblatex), [`bibtex`](https://ctan.org/pkg/bibtex), та [`biber`](https://ctan.org/pkg/biber) можуть бути опціонально використані для [надання цитування](https://pandoc.org/MANUAL.html#citation-rendering). Наступні пакети будуть використані для поліпшення якості виводу, якщо вони є, але pandoc не вимагає їх присутності: [`upquote`](https://ctan.org/pkg/upquote) (для прямих цитат у дослівному середовищі), [`microtype`](https://ctan.org/pkg/microtype) (для кращого регулювання інтервалу), [`parskip`](https://ctan.org/pkg/parskip) (для кращих проміжків між абзацами), [`xurl`](https://ctan.org/pkg/xurl) (для кращих розривів рядків у URL-адресах), [`bookmark`](https://ctan.org/pkg/bookmark) (для кращих PDF-закладок), та [`footnotehyper`](https://ctan.org/pkg/footnotehyper) або [`footnote`](https://ctan.org/pkg/footnote) (щоб дозволити виноски в таблицях).

## Читання з Web

Замість вхідного файлу може бути заданий абсолютний URI. У цьому випадку pandoc зчитає вміст за допомогою HTTP:

```
pandoc -f html -t markdown https://www.fsf.org
```

При запиті документа з URL-адреси можна надати спеціальний рядок User-Agent або інший заголовок:

```
pandoc -f html -t markdown --request-header User-Agent:"Mozilla/5.0" \
  https://www.fsf.org
```