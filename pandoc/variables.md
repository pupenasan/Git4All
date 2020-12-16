## Variables

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

  identifies the main language of the document using IETF language tags (following the [BCP 47](https://tools.ietf.org/html/bcp47) standard), such as `en` or `en-GB`. The [Language subtag lookup](https://r12a.github.io/app-subtags/) tool can look up or verify these tags. This affects most formats, and  controls hyphenation in PDF output when using LaTeX (through [`babel`](https://ctan.org/pkg/babel) and [`polyglossia`](https://ctan.org/pkg/polyglossia)) or ConTeXt. Use native pandoc [Divs and Spans](https://pandoc.org/MANUAL.html#divs-and-spans) with the `lang` attribute to switch the language: `--- lang: en-GB ... Text in the main document language (British English). ::: {lang=fr-CA} > Cette citation est écrite en français canadien. ::: More text in English. ['Zitat auf Deutsch.']{lang=de}`

- `dir`

  the base script direction, either `rtl` (right-to-left) or `ltr` (left-to-right). For bidirectional documents, native pandoc `span`s and `div`s with the `dir` attribute (value `rtl` or `ltr`) can be used to override the base direction in some output formats. This may not always be necessary if the final renderer (e.g. the browser,  when generating HTML) supports the [Unicode Bidirectional Algorithm](https://www.w3.org/International/articles/inline-bidi-markup/uba-basics). When using LaTeX for bidirectional documents, only the `xelatex` engine is fully supported (use [`--pdf-engine=xelatex`](https://pandoc.org/MANUAL.html#option--pdf-engine)).

### Variables for HTML math

- `classoption`

  when using [KaTeX](https://pandoc.org/MANUAL.html#option--katex), you can render display math equations flush left using [YAML metadata](https://pandoc.org/MANUAL.html#layout) or with [`-M classoption=fleqn`](https://pandoc.org/MANUAL.html#option--metadata).

### Variables for HTML slides

These affect HTML output when [producing slide shows with pandoc](https://pandoc.org/MANUAL.html#producing-slide-shows-with-pandoc).

All [reveal.js configuration options](https://github.com/hakimel/reveal.js#configuration) are available as variables. To turn off boolean flags that default to true in reveal.js, use `0`.

- `revealjs-url`

  base URL for reveal.js documents (defaults to `reveal.js`)

- `s5-url`

  base URL for S5 documents (defaults to `s5/default`)

- `slidy-url`

  base URL for Slidy documents (defaults to `https://www.w3.org/Talks/Tools/Slidy2`)

- `slideous-url`

  base URL for Slideous documents (defaults to `slideous`)

- `title-slide-attributes`

  additional attributes for the title slide of reveal.js slide shows. See [background in reveal.js and beamer](https://pandoc.org/MANUAL.html#background-in-reveal.js-and-beamer) for an example.

### Variables for Beamer slides

These variables change the appearance of PDF slides using [`beamer`](https://ctan.org/pkg/beamer).

- `aspectratio`

  slide aspect ratio (`43` for 4:3 [default], `169` for 16:9, `1610` for 16:10, `149` for 14:9, `141` for 1.41:1, `54` for 5:4, `32` for 3:2)

- `beamerarticle`

  produce an article from Beamer slides

- `beameroption`

  add extra beamer option with `\setbeameroption{}`

- `institute`

  author affiliations: can be a list when there are multiple authors

- `logo`

  logo image for slides

- `navigation`

  controls navigation symbols (default is `empty` for no navigation symbols; other valid values are `frame`, `vertical`, and `horizontal`)

- `section-titles`

  enables “title pages” for new sections (default is true)

- `theme`, `colortheme`, `fonttheme`, `innertheme`, `outertheme`

  beamer themes

- `themeoptions`

  options for LaTeX beamer themes (a list).

- `titlegraphic`

  image for title slide

### Variables for PowerPoint

These variables control the visual aspects of a slide show that are not easily controlled via templates.

- `monofont`

  font to use for code.

### Variables for LaTeX

Pandoc використовує ці змінні при [створенні PDF](https://pandoc.org/MANUAL.html#creating-a-pdf) з рушієм LaTeX.

#### Layout

- `block-headings`

  make `\paragraph` and `\subparagraph` (fourth- and fifth-level headings, or fifth- and sixth-level with book  classes) free-standing rather than run-in; requires further formatting  to distinguish from `\subsubsection` (third- or fourth-level headings). Instead of using this option, [KOMA-Script](https://ctan.org/pkg/koma-script) can adjust headings more extensively: `--- documentclass: scrartcl header-includes: |  \RedeclareSectionCommand[    beforeskip=-10pt plus -2pt minus -1pt,    afterskip=1sp plus -1sp minus 1sp,    font=\normalfont\itshape]{paragraph}  \RedeclareSectionCommand[    beforeskip=-10pt plus -2pt minus -1pt,    afterskip=1sp plus -1sp minus 1sp,    font=\normalfont\scshape,    indent=0pt]{subparagraph} ...`

- `classoption`

  option for document class, e.g. `oneside`; repeat for multiple options: `--- classoption: - twocolumn - landscape ...`

- `documentclass`

  document class: usually one of the standard classes, [`article`](https://ctan.org/pkg/article), [`book`](https://ctan.org/pkg/book), and [`report`](https://ctan.org/pkg/report); the [KOMA-Script](https://ctan.org/pkg/koma-script) equivalents, `scrartcl`, `scrbook`, and `scrreprt`, which default to smaller margins; or [`memoir`](https://ctan.org/pkg/memoir)

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

### Variables for ConTeXt

Pandoc uses these variables when [creating a PDF](https://pandoc.org/MANUAL.html#creating-a-pdf) with ConTeXt.

- `fontsize`

  font size for body text (e.g. `10pt`, `12pt`)

- `headertext`, `footertext`

  text to be placed in running header or footer (see [ConTeXt Headers and Footers](https://wiki.contextgarden.net/Headers_and_Footers)); repeat up to four times for different placement

- `indenting`

  controls indentation of paragraphs, e.g. `yes,small,next` (see [ConTeXt Indentation](https://wiki.contextgarden.net/Indentation)); repeat for multiple options

- `interlinespace`

  adjusts line spacing, e.g. `4ex` (using [`setupinterlinespace`](https://wiki.contextgarden.net/Command/setupinterlinespace)); repeat for multiple options

- `layout`

  options for page margins and text arrangement (see [ConTeXt Layout](https://wiki.contextgarden.net/Layout)); repeat for multiple options

- `linkcolor`, `contrastcolor`

  color for links outside and inside a page, e.g. `red`, `blue` (see [ConTeXt Color](https://wiki.contextgarden.net/Color))

- `linkstyle`

  typeface style for links, e.g. `normal`, `bold`, `slanted`, `boldslanted`, `type`, `cap`, `small`

- `lof`, `lot`

  include list of figures, list of tables

- `mainfont`, `sansfont`, `monofont`, `mathfont`

  font families: take the name of any system font (see [ConTeXt Font Switching](https://wiki.contextgarden.net/Font_Switching))

- `margin-left`, `margin-right`, `margin-top`, `margin-bottom`

  sets margins, if `layout` is not used (otherwise `layout` overrides these)

- `pagenumbering`

  page number style and location (using [`setuppagenumbering`](https://wiki.contextgarden.net/Command/setuppagenumbering)); repeat for multiple options

- `papersize`

  paper size, e.g. `letter`, `A4`, `landscape` (see [ConTeXt Paper Setup](https://wiki.contextgarden.net/PaperSetup)); repeat for multiple options

- `pdfa`

  adds to the preamble the setup necessary to generate PDF/A of the type specified, e.g. `1a:2005`, `2a`. If no type is specified (i.e. the value is set to True, by e.g. [`--metadata=pdfa`](https://pandoc.org/MANUAL.html#option--metadata) or `pdfa: true` in a YAML metadata block), `1b:2005` will be used as default, for reasons of backwards compatibility. Using [`--variable=pdfa`](https://pandoc.org/MANUAL.html#option--variable) without specified value is not supported. To successfully generate  PDF/A the required ICC color profiles have to be available and the  content and all included files (such as images) have to be standard  conforming. The ICC profiles and output intent may be specified using  the variables `pdfaiccprofile` and `pdfaintent`. See also [ConTeXt PDFA](https://wiki.contextgarden.net/PDF/A) for more details.

- `pdfaiccprofile`

  when used in conjunction with `pdfa`, specifies the ICC profile to use in the PDF, e.g. `default.cmyk`. If left unspecified, `sRGB.icc` is used as default. May be repeated to include multiple profiles. Note  that the profiles have to be available on the system. They can be  obtained from [ConTeXt ICC Profiles](https://wiki.contextgarden.net/PDFX#ICC_profiles).

- `pdfaintent`

  when used in conjunction with `pdfa`, specifies the output intent for the colors, e.g. `ISO coated v2 300\letterpercent\space (ECI)` If left unspecified, `sRGB IEC61966-2.1` is used as default.

- `toc`

  include table of contents (can also be set using [`--toc/--table-of-contents`](https://pandoc.org/MANUAL.html#option--toc))

- `whitespace`

  spacing between paragraphs, e.g. `none`, `small` (using [`setupwhitespace`](https://wiki.contextgarden.net/Command/setupwhitespace))

- `includesource`

  include all source documents as file attachments in the PDF file

### Variables for `wkhtmltopdf`

Pandoc uses these variables when [creating a PDF](https://pandoc.org/MANUAL.html#creating-a-pdf) with [`wkhtmltopdf`](https://wkhtmltopdf.org). The [`--css`](https://pandoc.org/MANUAL.html#option--css) option also affects the output.

- `footer-html`, `header-html`

  add information to the header and footer

- `margin-left`, `margin-right`, `margin-top`, `margin-bottom`

  set the page margins

- `papersize`

  sets the PDF paper size

### Variables for man pages

- `adjusting`

  adjusts text to left (`l`), right (`r`), center (`c`), or both (`b`) margins

- `footer`

  footer in man pages

- `header`

  header in man pages

- `hyphenate`

  if `true` (the default), hyphenation will be used

- `section`

  section number in man pages

### Variables for ms

- `fontfamily`

  font family (e.g. `T` or `P`)

- `indent`

  paragraph indent (e.g. `2m`)

- `lineheight`

  line height (e.g. `12p`)

- `pointsize`

  point size (e.g. `10p`)

### Variables set automatically

Pandoc sets these variables automatically in response to [options](https://pandoc.org/MANUAL.html#options) or document contents; users can also modify them. These vary depending on the output format, and include the following:

- `body`

  body of document

- `date-meta`

  the `date` variable converted to ISO 8601 YYYY-MM-DD, included in all HTML based formats (dzslides,  epub, html, html4, html5, revealjs, s5, slideous, slidy). The recognized formats for `date` are: `mm/dd/yyyy`, `mm/dd/yy`, `yyyy-mm-dd` (ISO 8601), `dd MM yyyy` (e.g. either `02 Apr 2018` or `02 April 2018`), `MM dd, yyyy` (e.g. `Apr. 02, 2018` or `April 02, 2018),`yyyy[mm[dd]]]`(e.g.`20180402, `201804` or `2018`).

- `header-includes`

  contents specified by [`-H/--include-in-header`](https://pandoc.org/MANUAL.html#option--include-in-header) (may have multiple values)

- `include-before`

  contents specified by [`-B/--include-before-body`](https://pandoc.org/MANUAL.html#option--include-before-body) (may have multiple values)

- `include-after`

  contents specified by [`-A/--include-after-body`](https://pandoc.org/MANUAL.html#option--include-after-body) (may have multiple values)

- `meta-json`

  JSON representation of all of the document’s metadata. Field values are transformed to the selected output format.

- `numbersections`

  non-null value if [`-N/--number-sections`](https://pandoc.org/MANUAL.html#option--number-sections) was specified

- `sourcefile`, `outputfile`

  source and destination filenames, as given on the command line. `sourcefile` can also be a list if input comes from multiple files, or empty if  input is from stdin. You can use the following snippet in your template  to distinguish them: `$if(sourcefile)$ $for(sourcefile)$ $sourcefile$ $endfor$ $else$ (stdin) $endif$` Similarly, `outputfile` can be `-` if output goes to the terminal. If you need absolute paths, use e.g. `$curdir$/$sourcefile$`.

- `curdir`

  working directory from which pandoc is run.

- `toc`

  non-null value if [`--toc/--table-of-contents`](https://pandoc.org/MANUAL.html#option--toc) was specified

- `toc-title`

  title of table of contents (works only with EPUB, HTML, opendocument, odt, docx, pptx, beamer, LaTeX)