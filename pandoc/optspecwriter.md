## Options affecting specific writers

[оригінал](https://pandoc.org/MANUAL.html#options-affecting-specific-writers)

- `--self-contained`

  Produce a standalone HTML file with no external dependencies, using `data:` URIs to incorporate the contents of linked scripts, stylesheets, images, and videos. Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone). The resulting file should be “self-contained,” in the sense that it  needs no external files and no net access to be displayed properly by a  browser. This option works only with HTML output formats, including `html4`, `html5`, `html+lhs`, `html5+lhs`, `s5`, `slidy`, `slideous`, `dzslides`, and `revealjs`. Scripts, images, and stylesheets at absolute URLs will be downloaded;  those at relative URLs will be sought relative to the working directory  (if the first source file is local) or relative to the base URL (if the  first source file is remote). Elements with the attribute `data-external="1"` will be left alone; the documents they link to will not be incorporated in the document. Limitation: resources that are loaded dynamically  through JavaScript cannot be incorporated; as a result, [`--self-contained`](https://pandoc.org/MANUAL.html#option--self-contained) does not work with [`--mathjax`](https://pandoc.org/MANUAL.html#option--mathjax), and some advanced features (e.g. zoom or speaker notes) may not work in an offline “self-contained” `reveal.js` slide show.

- `--html-q-tags`

  Use `<q>` tags for quotes in HTML.

- `--ascii`

  Use only ASCII characters in output. Currently supported for XML  and HTML formats (which use entities instead of UTF-8 when this option  is selected), CommonMark, gfm, and Markdown (which use entities), roff  ms (which use hexadecimal escapes), and to a limited degree LaTeX (which uses standard commands for accented characters when possible). roff man output uses ASCII by default.

- `--reference-links`

  Use reference-style links, rather than inline links, in writing  Markdown or reStructuredText. By default inline links are used. The  placement of link references is affected by the [`--reference-location`](https://pandoc.org/MANUAL.html#option--reference-location) option.

- `--reference-location = block`|`section`|`document`

  Specify whether footnotes (and references, if `reference-links` is set) are placed at the end of the current (top-level) block, the current section, or the document. The default is `document`. Currently only affects the markdown writer.

- `--atx-headers`

  Use ATX-style headings in Markdown output. The default is to use  setext-style headings for levels 1 to 2, and then ATX headings. (Note:  for `gfm` output, ATX headings are always used.) This option also affects markdown cells in `ipynb` output.

- `--top-level-division=[default|section|chapter|part]`

  Treat top-level headings as the given division type in LaTeX,  ConTeXt, DocBook, and TEI output. The hierarchy order is part, chapter,  then section; all headings are shifted such that the top-level heading  becomes the specified type. The default behavior is to determine the  best division type via heuristics: unless other conditions apply, `section` is chosen. When the `documentclass` variable is set to `report`, `book`, or `memoir` (unless the `article` option is specified), `chapter` is implied as the setting for this option. If `beamer` is the output format, specifying either `chapter` or `part` will cause top-level headings to become `\part{..}`, while second-level headings remain as their default type.

- `-N`, `--number-sections`

  Number section headings in LaTeX, ConTeXt, HTML, or EPUB output. By default, sections are not numbered. Sections with class `unnumbered` will never be numbered, even if [`--number-sections`](https://pandoc.org/MANUAL.html#option--number-sections) is specified.

- `--number-offset=`*NUMBER*[`,`*NUMBER*`,`*…*]

  Offset for section headings in HTML output (ignored in other  output formats). The first number is added to the section number for  top-level headings, the second for second-level headings, and so on. So, for example, if you want the first top-level heading in your document  to be numbered “6”, specify [`--number-offset=5`](https://pandoc.org/MANUAL.html#option--number-offset). If your document starts with a level-2 heading which you want to be numbered “1.5”, specify [`--number-offset=1,4`](https://pandoc.org/MANUAL.html#option--number-offset). Offsets are 0 by default. Implies [`--number-sections`](https://pandoc.org/MANUAL.html#option--number-sections).

- `--listings`

  Use the [`listings`](https://ctan.org/pkg/listings) package for LaTeX code blocks. The package does not support multi-byte  encoding for source code. To handle UTF-8 you would need to use a custom template. This issue is fully documented here: [Encoding issue with the listings package](https://en.wikibooks.org/wiki/LaTeX/Source_Code_Listings#Encoding_issue).

- `-i`, `--incremental`

  Make list items in slide shows display incrementally (one by one). The default is for lists to be displayed all at once.

- `--slide-level=`*NUMBER*

  Specifies that headings with the specified level create slides (for `beamer`, `s5`, `slidy`, `slideous`, `dzslides`). Headings above this level in the hierarchy are used to divide the slide show into sections; headings below this level create subheads within a  slide. Note that content that is not contained under slide-level  headings will not appear in the slide show. The default is to set the  slide level based on the contents of the document; see [Structuring the slide show](https://pandoc.org/MANUAL.html#structuring-the-slide-show).

- `--section-divs`

  Wrap sections in `<section>` tags (or `<div>` tags for `html4`), and attach identifiers to the enclosing `<section>` (or `<div>`) rather than the heading itself. See [Heading identifiers](https://pandoc.org/MANUAL.html#heading-identifiers), below.

- `--email-obfuscation=none`|`javascript`|`references`

  Specify a method for obfuscating `mailto:` links in HTML documents. `none` leaves `mailto:` links as they are. `javascript` obfuscates them using JavaScript. `references` obfuscates them by printing their letters as decimal or hexadecimal character references. The default is `none`.

- `--id-prefix=`*STRING*

  Specify a prefix to be added to all identifiers and internal  links in HTML and DocBook output, and to footnote numbers in Markdown  and Haddock output. This is useful for preventing duplicate identifiers  when generating fragments to be included in other pages.

- `-T` *STRING*, `--title-prefix=`*STRING*

  Specify *STRING* as a prefix at the beginning of the title that appears in the HTML header (but not in the title as it appears at  the beginning of the HTML body). Implies [`--standalone`](https://pandoc.org/MANUAL.html#option--standalone).

- `-c` *URL*, `--css=`*URL*

  Link to a CSS style sheet. This option can be used repeatedly to  include multiple files. They will be included in the order specified. A stylesheet is required for generating EPUB. If none is provided using this option (or the `css` or `stylesheet` metadata fields), pandoc will look for a file `epub.css` in the user data directory (see [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). If it is not found there, sensible defaults will be used.

- `--reference-doc=`*FILE*

  Використовуйте вказаний файл як посилання на стиль при створенні файлу docx або ODT.

  **Docx** 

  Для найкращих результатів посилання docx має бути модифікованою версією docx-файлу, створеного за допомогою pandoc. Вміст посилального docx ігнорується, але його таблиці стилів та властивості документа (включаючи поля, розмір сторінки, заголовок та колонтитул) використовуються в новому docx. Якщо в командному рядку не вказано жодного довідкового docx, pandoc шукатиме файл `reference.docx` у каталозі даних користувачів (див [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). Якщо цього не буде знайдено, використовуватимуться значущі параметри за замовчуванням.

  Щоб створити власну `reference.docx`, спочатку отримайте копію за замовчуванням ` reference.docx`: 

  ```bash
  pandoc -o custom-reference.docx --print-default-data-file reference.docx
  ```

  Потім відкрийте "custom-reference.docx" у Word, змініть стилі за своїм бажанням і збережіть файл. Для найкращих результатів не вносьте змін у цей файл, окрім модифікації стилів, використовуваних pandoc:

  Paragraph styles: 

  - Normal 
  - Body Text 
  - First Paragraph 
  - Compact 
  - Title 
  - Subtitle 
  - Author 
  - Date 
  - Abstract 
  - Bibliography 
  - Heading 1 
  - Heading 2 
  - Heading 3 
  - Heading 4 
  - Heading 5 
  - Heading 6 
  - Heading 7 
  - Heading 8 
  - Heading 9 
  - Block Text 
  - Footnote Text 
  - Definition Term 
  - Definition Caption 
  - Table Caption 
  - Image Caption 
  - Figure 
  - Captioned Figure 
  - TOC Heading 

  Character styles: 

  - Default Paragraph Font 
  - Body Text Char 
  - Verbatim Char 
  - Footnote Reference 
  - Hyperlink 

  Table style: 

  - Table  

**ODT** 

For best results, the reference ODT should be a modified version  of an ODT produced using pandoc. The contents of the reference ODT are  ignored, but its stylesheets are used in the new ODT. If no reference  ODT is specified on the command line, pandoc will look for a file `reference.odt` in the user data directory (see [`--data-dir`](https://pandoc.org/MANUAL.html#option--data-dir)). If this is not found either, sensible defaults will be used. 

To produce a custom `reference.odt`, first get a copy of the default `reference.odt`: `pandoc -o custom-reference.odt --print-default-data-file reference.odt`. Then open `custom-reference.odt` in LibreOffice, modify the styles as you wish, and save the file. 

**PowerPoint** 

Templates included with Microsoft PowerPoint 2013 (either with `.pptx` or `.potx` extension) are known to work, as are most templates derived from these. The specific requirement is that the template should begin with the following first four layouts: Title Slide Title and Content Section Header Two Content All templates included with a recent version of MS PowerPoint will fit these criteria. (You can click on `Layout` under the `Home` menu to check.) You can also modify the default `reference.pptx`: first run `pandoc -o custom-reference.pptx --print-default-data-file reference.pptx`, and then modify `custom-reference.pptx` in MS PowerPoint (pandoc will use the first four layout slides, as mentioned above).

- `--epub-cover-image=`*FILE*

  Use the specified image as the EPUB cover. It is recommended that the image be less than 1000px in width and height. Note that in a  Markdown source document you can also specify `cover-image` in a YAML metadata block (see [EPUB Metadata](https://pandoc.org/MANUAL.html#epub-metadata), below).

- `--epub-metadata=`*FILE*

  Look in the specified XML file for metadata for the EPUB. The file should contain a series of [Dublin Core elements](https://www.dublincore.org/specifications/dublin-core/dces/). For example: ` <dc:rights>Creative Commons</dc:rights> <dc:language>es-AR</dc:language>` By default, pandoc will include the following metadata elements: `<dc:title>` (from the document title), `<dc:creator>` (from the document authors), `<dc:date>` (from the document date, which should be in [ISO 8601 format](https://www.w3.org/TR/NOTE-datetime)), `<dc:language>` (from the `lang` variable, or, if is not set, the locale), and `<dc:identifier id="BookId">` (a randomly generated UUID). Any of these may be overridden by elements in the metadata file. Note: if the source document is Markdown, a YAML metadata block in the document can be used instead. See below under [EPUB Metadata](https://pandoc.org/MANUAL.html#epub-metadata).

- `--epub-embed-font=`*FILE*

  Embed the specified font in the EPUB. This option can be repeated to embed multiple fonts. Wildcards can also be used: for example, `DejaVuSans-*.ttf`. However, if you use wildcards on the command line, be sure to escape  them or put the whole filename in single quotes, to prevent them from  being interpreted by the shell. To use the embedded fonts, you will need to add declarations like the following to your CSS (see [`--css`](https://pandoc.org/MANUAL.html#option--css)): `@font-face { font-family: DejaVuSans; font-style: normal; font-weight: normal; src:url("DejaVuSans-Regular.ttf"); } @font-face { font-family: DejaVuSans; font-style: normal; font-weight: bold; src:url("DejaVuSans-Bold.ttf"); } @font-face { font-family: DejaVuSans; font-style: italic; font-weight: normal; src:url("DejaVuSans-Oblique.ttf"); } @font-face { font-family: DejaVuSans; font-style: italic; font-weight: bold; src:url("DejaVuSans-BoldOblique.ttf"); } body { font-family: "DejaVuSans"; }`

- `--epub-chapter-level=`*NUMBER*

  Specify the heading level at which to split the EPUB into  separate “chapter” files. The default is to split into chapters at  level-1 headings. This option only affects the internal composition of  the EPUB, not the way chapters and sections are displayed to users. Some readers may be slow if the chapter files are too large, so for large  documents with few level-1 headings, one might want to use a chapter  level of 2 or 3.

- `--epub-subdirectory=`*DIRNAME*

  Specify the subdirectory in the OCF container that is to hold the EPUB-specific contents. The default is `EPUB`. To put the EPUB contents in the top level, use an empty string.

- `--ipynb-output=all|none|best`

  Determines how ipynb output cells are treated. `all` means that all of the data formats included in the original are preserved. `none` means that the contents of data cells are omitted. `best` causes pandoc to try to pick the richest data block in each output cell that is compatible with the output format. The default is `best`.

- `--pdf-engine=`*PROGRAM*

  Use the specified engine when producing PDF output. Valid values are `pdflatex`, `lualatex`, `xelatex`, `latexmk`, `tectonic`, `wkhtmltopdf`, `weasyprint`, `prince`, `context`, and `pdfroff`. If the engine is not in your PATH, the full path of the engine may be  specified here. If this option is not specified, pandoc uses the  following defaults depending on the output format specified using [`-t/--to`](https://pandoc.org/MANUAL.html#option--to): [`-t latex`](https://pandoc.org/MANUAL.html#option--to) or none: `pdflatex` (other options: `xelatex`, `lualatex`, `tectonic`, `latexmk`) [`-t context`](https://pandoc.org/MANUAL.html#option--to): `context` [`-t html`](https://pandoc.org/MANUAL.html#option--to): `wkhtmltopdf` (other options: `prince`, `weasyprint`) [`-t ms`](https://pandoc.org/MANUAL.html#option--to): `pdfroff`

- `--pdf-engine-opt=`*STRING*

  Use the given string as a command-line argument to the `pdf-engine`. For example, to use a persistent directory `foo` for `latexmk`’s auxiliary files, use [`--pdf-engine-opt=-outdir=foo`](https://pandoc.org/MANUAL.html#option--pdf-engine-opt). Note that no check for duplicate options is done.