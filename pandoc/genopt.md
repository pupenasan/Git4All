## Загальні опції

[оригінал](https://pandoc.org/MANUAL.html#general-options)

##### вхідний формат

`-f` *FORMAT*, `-r` *FORMAT*, `--from=`*FORMAT*, `--read=`*FORMAT*

Вказує вхідний формат. *FORMAT* може бути: 

- `commonmark` ([CommonMark](https://commonmark.org) Markdown)

- `creole` ([Creole 1.0](http://www.wikicreole.org/wiki/Creole1.0)) 

- `csv` ([CSV](https://tools.ietf.org/html/rfc4180) table)

-  `docbook` ([DocBook](https://docbook.org))

-  `docx` ([Word docx](https://en.wikipedia.org/wiki/Office_Open_XML))

-  `dokuwiki` ([DokuWiki markup](https://www.dokuwiki.org/dokuwiki))
-  `epub` ([EPUB](http://idpf.org/epub))
-  `fb2` ([FictionBook2](http://www.fictionbook.org/index.php/Eng:XML_Schema_Fictionbook_2.1) e-book)
-  `gfm` ([GitHub-Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/)), or the deprecated and less accurate `markdown_github`; use [`markdown_github`](https://pandoc.org/MANUAL.html#markdown-variants) only if you need extensions not supported in [`gfm`](https://pandoc.org/MANUAL.html#markdown-variants).
-  `haddock` ([Haddock markup](https://www.haskell.org/haddock/doc/html/ch03s08.html))
-  `html` ([HTML](https://www.w3.org/html/))
-  `ipynb` ([Jupyter notebook](https://nbformat.readthedocs.io/en/latest/))
-  `jats` ([JATS](https://jats.nlm.nih.gov) XML)
-  `jira` ([Jira](https://jira.atlassian.com/secure/WikiRendererHelpAction.jspa?section=all) wiki markup)
-  `json` (JSON version of native AST)
-  `latex` ([LaTeX](https://www.latex-project.org/))
-  `markdown` ([Pandoc’s Markdown](https://pandoc.org/MANUAL.html#pandocs-markdown))
-  `markdown_mmd` ([MultiMarkdown](https://fletcherpenney.net/multimarkdown/))
-  `markdown_phpextra` ([PHP Markdown Extra](https://michelf.ca/projects/php-markdown/extra/))
-  `markdown_strict` (original unextended [Markdown](https://daringfireball.net/projects/markdown/))
-  `mediawiki` ([MediaWiki markup](https://www.mediawiki.org/wiki/Help:Formatting))
-  `man` ([roff man](https://man.cx/groff_man(7)))
-  `muse` ([Muse](https://amusewiki.org/library/manual))
-  `native` (native Haskell)
-  `odt` ([ODT](https://en.wikipedia.org/wiki/OpenDocument))
-  `opml` ([OPML](http://dev.opml.org/spec2.html))
-  `org` ([Emacs Org mode](https://orgmode.org))
-  `rst` ([reStructuredText](https://docutils.sourceforge.io/docs/ref/rst/introduction.html))
-  `t2t` ([txt2tags](https://txt2tags.org))
-  `textile` ([Textile](https://www.promptworks.com/textile))
-  `tikiwiki` ([TikiWiki markup](https://doc.tiki.org/Wiki-Syntax-Text#The_Markup_Language_Wiki-Syntax))
-  `twiki` ([TWiki markup](https://twiki.org/cgi-bin/view/TWiki/TextFormattingRules))
-  `vimwiki` ([Vimwiki](https://vimwiki.github.io))

Розширення можна включити або вимкнути індивідуально, додавши  `+EXTENSION` або `-EXTENSION` в ім'я формату.  Див [Extensions](https://pandoc.org/MANUAL.html#extensions) нижче, для переліку розширень та їх назв. Див [`--list-input-formats`](https://pandoc.org/MANUAL.html#option--list-input-formats) та [`--list-extensions`](https://pandoc.org/MANUAL.html#option--list-extensions), нижче.

##### вихідний формат

`-t` *FORMAT*, `-w` *FORMAT*, `--to=`*FORMAT*, `--write=`*FORMAT*

Вказує вихідний формат. *FORMAT* може бути: 

- `asciidoc` ([AsciiDoc](https://www.methods.co.nz/asciidoc/)) or `asciidoctor` ([AsciiDoctor](https://asciidoctor.org/))
- `beamer` ([LaTeX beamer](https://ctan.org/pkg/beamer) slide show) 
- `commonmark` ([CommonMark](https://commonmark.org) Markdown) 
- `context` ([ConTeXt](https://www.contextgarden.net/)) 
- `docbook` or `docbook4` ([DocBook](https://docbook.org) 4)
-  `docbook5` (DocBook 5)
-  `docx` ([Word docx](https://en.wikipedia.org/wiki/Office_Open_XML))
-  `dokuwiki` ([DokuWiki markup](https://www.dokuwiki.org/dokuwiki))
-  `epub` or `epub3` ([EPUB](http://idpf.org/epub) v3 book)
-  `epub2` (EPUB v2) 
- `fb2` ([FictionBook2](http://www.fictionbook.org/index.php/Eng:XML_Schema_Fictionbook_2.1) e-book)
-  `gfm` ([GitHub-Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/)), or the deprecated and less accurate `markdown_github`; use [`markdown_github`](https://pandoc.org/MANUAL.html#markdown-variants) only if you need extensions not supported in [`gfm`](https://pandoc.org/MANUAL.html#markdown-variants). 
- `haddock` ([Haddock markup](https://www.haskell.org/haddock/doc/html/ch03s08.html))
-  `html` or `html5` ([HTML](https://www.w3.org/html/), i.e. [HTML5](https://html.spec.whatwg.org/)/XHTML [polyglot markup](https://www.w3.org/TR/html-polyglot/))
-  `html4` ([XHTML](https://www.w3.org/TR/xhtml1/) 1.0 Transitional)
-  `icml` ([InDesign ICML](https://wwwimages.adobe.com/www.adobe.com/content/dam/acom/en/devnet/indesign/sdk/cs6/idml/idml-cookbook.pdf))
-  `ipynb` ([Jupyter notebook](https://nbformat.readthedocs.io/en/latest/))
-  `jats_archiving` ([JATS](https://jats.nlm.nih.gov) XML, Archiving and Interchange Tag Set)
-  `jats_articleauthoring` ([JATS](https://jats.nlm.nih.gov) XML, Article Authoring Tag Set)
-  `jats_publishing` ([JATS](https://jats.nlm.nih.gov) XML, Journal Publishing Tag Set)
-  `jats` (alias for `jats_archiving`)
-  `jira` ([Jira](https://jira.atlassian.com/secure/WikiRendererHelpAction.jspa?section=all) wiki markup)
-  `json` (JSON version of native AST)
-  `latex` ([LaTeX](https://www.latex-project.org/))
-  `man` ([roff man](https://man.cx/groff_man(7)))
-  `markdown` ([Pandoc’s Markdown](https://pandoc.org/MANUAL.html#pandocs-markdown))
-  `markdown_mmd` ([MultiMarkdown](https://fletcherpenney.net/multimarkdown/))
-  `markdown_phpextra` ([PHP Markdown Extra](https://michelf.ca/projects/php-markdown/extra/))
-  `markdown_strict` (original unextended [Markdown](https://daringfireball.net/projects/markdown/))
-  `mediawiki` ([MediaWiki markup](https://www.mediawiki.org/wiki/Help:Formatting))
-  `ms` ([roff ms](https://man.cx/groff_ms(7)))
-  `muse` ([Muse](https://amusewiki.org/library/manual)),
-  `native` (native Haskell),
-  `odt` ([OpenOffice text document](https://en.wikipedia.org/wiki/OpenDocument))
-  `opml` ([OPML](http://dev.opml.org/spec2.html))
-  `opendocument` ([OpenDocument](http://opendocument.xml.org))
-  `org` ([Emacs Org mode](https://orgmode.org))
-  `pdf` ([PDF](https://www.adobe.com/pdf/))
-  `plain` (plain text),
-  `pptx` ([PowerPoint](https://en.wikipedia.org/wiki/Microsoft_PowerPoint) slide show)
-  `rst` ([reStructuredText](https://docutils.sourceforge.io/docs/ref/rst/introduction.html))
-  `rtf` ([Rich Text Format](https://en.wikipedia.org/wiki/Rich_Text_Format))
-  `texinfo` ([GNU Texinfo](https://www.gnu.org/software/texinfo/))
-  `textile` ([Textile](https://www.promptworks.com/textile))
-  `slideous` ([Slideous](https://goessner.net/articles/slideous/) HTML and JavaScript slide show)
-  `slidy` ([Slidy](https://www.w3.org/Talks/Tools/Slidy2/) HTML and JavaScript slide show)
-  `dzslides` ([DZSlides](http://paulrouget.com/dzslides/) HTML5 + JavaScript slide show),
-  `revealjs` ([reveal.js](https://revealjs.com/) HTML5 + JavaScript slide show) 
- `s5` ([S5](https://meyerweb.com/eric/tools/s5/) HTML and JavaScript slide show)
-  `tei` ([TEI Simple](https://github.com/TEIC/TEI-Simple))
-  `xwiki` ([XWiki markup](https://www.xwiki.org/xwiki/bin/view/Documentation/UserGuide/Features/XWikiSyntax/))
-  `zimwiki` ([ZimWiki markup](https://zim-wiki.org/manual/Help/Wiki_Syntax.html)) the path of a custom Lua writer, see [Custom writers](https://pandoc.org/MANUAL.html#custom-writers) below  Note that `odt`, `docx`, `epub`, and `pdf` output will not be directed to *stdout* unless forced with [`-o -`](https://pandoc.org/MANUAL.html#option--output). 

Розширення можна включити або вимкнути індивідуально, додавши  `+EXTENSION` або `-EXTENSION` в ім'я формату.  Див [Extensions](https://pandoc.org/MANUAL.html#extensions) нижче, для переліку розширень та їх назв. Див [`--list-input-formats`](https://pandoc.org/MANUAL.html#option--list-input-formats) та [`--list-extensions`](https://pandoc.org/MANUAL.html#option--list-extensions), нижче.

##### виведення в файл

 `-o` *FILE*, `--output=`*FILE*

Записування в *FILE* замість *stdout*. Якщо *FILE*  `-`, вихід піде на *stdout*, навіть якщо вказаний нетекстовий формат (`docx`, `odt`, `epub2`, `epub3`).

##### каталог даних

`--data-dir=`*DIRECTORY*

Вкажіть каталог даних користувачів для пошуку файлів даних pandoc. Якщо ця опція не вказана, буде використаний каталог даних користувачів за замовчуванням. Для систем *nix та macOS це буде піддерикторія `pandoc` директорії даних XDG (за замовченням, `$HOME/.local/share`, перезапис за допомогою налаштування змінної середовища `XDG_DATA_HOME` ). Якщо цього каталогу не існує, буде використовуватися `$HOME/.pandoc` (для зворотної сумісності ). У Windows користувацька директорія за замовченням `C:\Users\USERNAME\AppData\Roaming\pandoc`.  Ви можете знайти каталог даних користувачів за замовчуванням у вашій системі, переглянувши вихідні дані  `pandoc --version`. Директорії  `reference.odt`, `reference.docx`, `epub.css`, `templates`, `slidy`, `slideous`, або `s5` що розміщені в цій директорії будуть перезаписані pandoc’s за замовченням.

##### файл налаштувань за замвоченням

`-d` *FILE*, `--defaults=`*FILE*

Вказує набір параметрів параметрів за замовчуванням. *FILE* є форматом YAML, поля якого відповідають налаштуванням параметра командного рядка. Усі параметри перетворення документів, включаючи вхідні та вихідні файли, можна встановити за допомогою файлу за замовчуванням. Спершу файл буде шукати у робочому каталозі, а потім у піддиректорії `defaults` користувацької директорії (див [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). Розширення `.yaml` може бути опущене. Див розділ [Default files](https://pandoc.org/MANUAL.html#default-files) для детальнішої інформації про формат файлу. Налаштування для файлу за замовченням можуть бути перезаписані або розширені наступними опціями в командному рядку.

##### файл сценарію завершення 

`--bash-completion`

Створіть сценарій завершення bash . Щоб увімкнути завершення bash за допомогою pandoc, додайте це до свого  `.bashrc`: `eval "$(pandoc --bash-completion)"`

##### деталізований вивід нааодження

 `--verbose`

Встановлює деталізований вихід налагодження. В даний час це впливає лише на вихід PDF.

##### не виводити попередження

 `--quiet`

Забороняє вивід попереджень.

- `--fail-if-warnings`

  Виходить зі статусом помилки, якщо є якісь попередження.

- `--log=`*FILE*

  Write log messages in machine-readable JSON format to *FILE*. All messages above DEBUG level will be written, regardless of verbosity settings ([`--verbose`](https://pandoc.org/MANUAL.html#option--verbose), [`--quiet`](https://pandoc.org/MANUAL.html#option--quiet)).

- `--list-input-formats`

  List supported input formats, one per line.

- `--list-output-formats`

  List supported output formats, one per line.

- `--list-extensions`[`=`*FORMAT*]

  List supported extensions for *FORMAT*, one per line, preceded by a `+` or `-` indicating whether it is enabled by default in *FORMAT*. If *FORMAT* is not specified, defaults for pandoc’s Markdown are given.

- `--list-highlight-languages`

  List supported languages for syntax highlighting, one per line.

- `--list-highlight-styles`

  List supported styles for syntax highlighting, one per line. See [`--highlight-style`](https://pandoc.org/MANUAL.html#option--highlight-style).

- `-v`, `--version`

  Print version.

- `-h`, `--help`

  Show usage message.

## Загальні опції для запису

[оригінал](https://pandoc.org/MANUAL.html#general-writer-options)

##### вивести комплектний файл

 `-s`, `--standalone`

Отримайте вихід із відповідним колонтитулом та колонтитулом (наприклад, окремий файл HTML, LaTeX, TEI або RTF, а не фрагмент). Цей параметр встановлюється автоматично для виходів `pdf`, `epub`, `epub3`, `fb2`, `docx`, та `odt` . Для виходу `native` , ця опція викликає включення метаданих; інакше метадані забороняються.

##### `--template=`*FILE*|*URL*

Використовує вказаний файл, як спеціальний шаблон для згенерованого документа. Наслідує [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone). Див [Templates](https://pandoc.org/MANUAL.html#templates), нижче, для опису синтаксису шаблону. Якщо не вказано розширення, буде додано розширення, що відповідає записувачу, так що  [`--template=special`](https://pandoc.org/MANUAL.html#option--template) шукатиме  наприклад `special.html` для HTML виходу. Якщо шаблон не знайдено, pandoc буде шукати його у підкаталозі `templates` каталогу користувача (див [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). Якщо ця опція не використовується, буде використаний шаблон за замовчуванням, відповідний формату виводу (see [`-D/--print-default-template`](https://pandoc.org/MANUAL.html#option--print-default-template)).

##### `-V` *KEY*[`=`*VAL*], `--variable=`*KEY*[`:`*VAL*]

При перетворенні документа в режимі standalone встановлює змінні *KEY* в значення *VAL*. Якщо значення *VAL* не вказані, значення буде взяте як `true`.

##### `-D` *FORMAT*, `--print-default-template=`*FORMAT*

Print the system default template for an output *FORMAT*. (See [`-t`](https://pandoc.org/MANUAL.html#option--to) for a list of possible *FORMAT*s.) Templates in the user data directory are ignored. This option may be used with [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) to redirect output to a file, but [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) must come before [`--print-default-template`](https://pandoc.org/MANUAL.html#option--print-default-template) on the command line. Note that some of the default templates use partials, for example `styles.html`. To print the partials, use [`--print-default-data-file`](https://pandoc.org/MANUAL.html#option--print-default-data-file): for example, [`--print-default-data-file=templates/styles.html`](https://pandoc.org/MANUAL.html#option--print-default-data-file).

##### `--print-default-data-file=`*FILE*

Print a system default data file. Files in the user data directory are ignored. This option may be used with [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) to redirect output to a file, but [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) must come before [`--print-default-data-file`](https://pandoc.org/MANUAL.html#option--print-default-data-file) on the command line.

##### `--eol=crlf`|`lf`|`native`

Manually specify line endings: `crlf` (Windows), `lf` (macOS/Linux/UNIX), or `native` (line endings appropriate to the OS on which pandoc is being run). The default is `native`.

##### `--dpi`=*NUMBER*

Specify the default dpi (dots per inch) value for conversion from pixels to inch/centimeters and vice versa. (Technically, the correct  term would be ppi: pixels per inch.) The default is 96dpi. When images  contain information about dpi internally, the encoded value is used  instead of the default specified by this option.

##### `--wrap=auto`|`none`|`preserve`

Determine how text is wrapped in the output (the source code, not the rendered version). With `auto` (the default), pandoc will attempt to wrap lines to the column width specified by [`--columns`](https://pandoc.org/MANUAL.html#option--columns) (default 72). With `none`, pandoc will not wrap lines at all. With `preserve`, pandoc will attempt to preserve the wrapping from the source document  (that is, where there are nonsemantic newlines in the source, there will be nonsemantic newlines in the output as well). Automatic wrapping does not currently work in HTML output. In `ipynb` output, this option affects wrapping of the contents of markdown cells.

##### `--columns=`*NUMBER*

Specify length of lines in characters. This affects text wrapping in the generated source code (see [`--wrap`](https://pandoc.org/MANUAL.html#option--wrap)). It also affects calculation of column widths for plain text tables (see [Tables](https://pandoc.org/MANUAL.html#tables) below).

##### `--toc`, `--table-of-contents`

Include an automatically generated table of contents (or, in the case of `latex`, `context`, `docx`, `odt`, `opendocument`, `rst`, or `ms`, an instruction to create one) in the output document. This option has no effect unless [`-s/--standalone`](https://pandoc.org/MANUAL.html#option--standalone) is used, and it has no effect on `man`, `docbook4`, `docbook5`, or `jats` output. Note that if you are producing a PDF via `ms`, the table of contents will appear at the beginning of the document,  before the title. If you would prefer it to be at the end of the  document, use the option [`--pdf-engine-opt=--no-toc-relocation`](https://pandoc.org/MANUAL.html#option--pdf-engine-opt).

##### `--toc-depth=`*NUMBER*

Specify the number of section levels to include in the table of  contents. The default is 3 (which means that level-1, 2, and 3 headings  will be listed in the contents).

##### `--strip-comments`

Strip out HTML comments in the Markdown or Textile source, rather than passing them on to Markdown, Textile or HTML output as raw HTML.  This does not apply to HTML comments inside raw HTML blocks when the `markdown_in_html_blocks` extension is not set.

##### `--no-highlight`

Disables syntax highlighting for code blocks and inlines, even when a language attribute is given.

##### `--highlight-style=`*STYLE*|*FILE*

Specifies the coloring style to be used in highlighted source code. Options are `pygments` (the default), `kate`, `monochrome`, `breezeDark`, `espresso`, `zenburn`, `haddock`, and `tango`. For more information on syntax highlighting in pandoc, see [Syntax highlighting](https://pandoc.org/MANUAL.html#syntax-highlighting), below. See also [`--list-highlight-styles`](https://pandoc.org/MANUAL.html#option--list-highlight-styles). Instead of a *STYLE* name, a JSON file with extension `.theme` may be supplied. This will be parsed as a KDE syntax highlighting theme and (if valid) used as the highlighting style. To generate the JSON version of an existing style, use [`--print-highlight-style`](https://pandoc.org/MANUAL.html#option--print-highlight-style).

##### `--print-highlight-style=`*STYLE*|*FILE*

Prints a JSON version of a highlighting style, which can be modified, saved with a `.theme` extension, and used with [`--highlight-style`](https://pandoc.org/MANUAL.html#option--highlight-style). This option may be used with [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) to redirect output to a file, but [`-o`](https://pandoc.org/MANUAL.html#option--output)/[`--output`](https://pandoc.org/MANUAL.html#option--output) must come before [`--print-highlight-style`](https://pandoc.org/MANUAL.html#option--print-highlight-style) on the command line.

##### `--syntax-definition=`*FILE*

Instructs pandoc to load a KDE XML syntax definition file, which  will be used for syntax highlighting of appropriately marked code  blocks. This can be used to add support for new languages or to use  altered syntax definitions for existing languages. This option may be  repeated to add multiple syntax definitions.

##### `-H` *FILE*, `--include-in-header=`*FILE*|*URL*

Include contents of *FILE*, verbatim, at the end of the  header. This can be used, for example, to include special CSS or  JavaScript in HTML documents. This option can be used repeatedly to  include multiple files in the header. They will be included in the order specified. Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone).

##### `-B` *FILE*, `--include-before-body=`*FILE*|*URL*

Include contents of *FILE*, verbatim, at the beginning of the document body (e.g. after the `<body>` tag in HTML, or the `\begin{document}` command in LaTeX). This can be used to include navigation bars or  banners in HTML documents. This option can be used repeatedly to include multiple files. They will be included in the order specified. Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone).

##### `-A` *FILE*, `--include-after-body=`*FILE*|*URL*

Include contents of *FILE*, verbatim, at the end of the document body (before the `</body>` tag in HTML, or the `\end{document}` command in LaTeX). This option can be used repeatedly to include  multiple files. They will be included in the order specified. Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone).

##### `--resource-path=`*SEARCHPATH*

List of paths to search for images and other resources. The paths should be separated by `:` on Linux, UNIX, and macOS systems, and by `;` on Windows. If [`--resource-path`](https://pandoc.org/MANUAL.html#option--resource-path) is not specified, the default resource path is the working directory. Note that, if [`--resource-path`](https://pandoc.org/MANUAL.html#option--resource-path) is specified, the working directory must be explicitly listed or it will not be searched. For example: [`--resource-path=.:test`](https://pandoc.org/MANUAL.html#option--resource-path) will search the working directory and the `test` subdirectory, in that order. [`--resource-path`](https://pandoc.org/MANUAL.html#option--resource-path) only has an effect if (a) the output format embeds images (for example, `docx`, `pdf`, or `html` with [`--self-contained`](https://pandoc.org/MANUAL.html#option--self-contained)) or (b) it is used together with [`--extract-media`](https://pandoc.org/MANUAL.html#option--extract-media).

##### `--request-header=`*NAME*`:`*VAL*

Set the request header *NAME* to the value *VAL*  when making HTTP requests (for example, when a URL is given on the  command line, or when resources used in a document must be downloaded).  If you’re behind a proxy, you also need to set the environment variable `http_proxy` to `http://...`.