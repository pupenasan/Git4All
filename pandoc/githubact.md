# Using `pandoc` with [GitHub Actions](https://github.com/features/actions)

[Using `pandoc` with [GitHub Actions](https://github.com/features/actions)](https://github.com/pandoc/pandoc-action-example)

Ви можете використовувати [pandoc](https://pandoc.org/), універсальний конвертер розмітки, в GitHub Actions для перетворення документів.

GitHub Actions - це інфраструктура як послуга (IaaS) від GitHub, яка дозволяє автоматично запускати код на серверах GitHub на кожному пуші (або купі інших подій GitHub). Наприклад, ви можете використовувати дії GitHub для перетворення деякого `file.md` у` file.pdf` (через LaTeX) та завантаження результатів на веб-хост.

## Using `docker://pandoc` Images Directly

Тепер ви можете *безпосередньо* [посилатися на actions контейнера](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow#referencing-a-container-on-docker-hub) на GitHub Actions. Вам не потрібна  окрема GitHub Action.

Якщо вам потрібен LaTeX (тому що ви хочете перетворити його в PDF), слід використовувати образ`docker://pandoc/latex`. В іншому випадку потрібен менший `docker://pandoc/core`.

Варто вказати явно потрібну версію pandoc, наприклад `docker://pandoc/core:2.9`. Таким чином, будь-які майбутні порушення в системі pandoc не вплинуть на ваш робочий процес. Ви можете дізнатися, який найновіший випущений образ докера є на [docker hub](https://hub.docker.com/r/pandoc/core/tags). Вам варто вказувати версію і  уникати вказівки тегу `latest` - вони будуть вказувати на останній образ і піддаватимуть ваш робочий процес потенційним проблемам.

## Просте використання

Ви можете використовувати pandoc всередині GitHub actions так саме, як і в командному рядку. Рядок, переданий `args`, додається до команди  [`pandoc` command](https://pandoc.org/MANUAL.html).

Наведений нижче приклад еквівалентний запуску `pandoc --help`.

Ви можете бачити це в дії [тут](http://github.com/maxheld83/pandoc-example).

```
name: Simple Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - uses: docker://pandoc/core:2.9
        with:
          args: "--help" # gets appended to pandoc command
```

## Довгі виклики Pandoc 

Remember that as per the [GitHub Actions workflow syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswithargs), "an `array of strings` is not supported by the `jobs.<job_id>.steps.with.args` parameter. Pandoc commands can sometimes get quite long and unwieldy, but you must pass them as a *single* string. If you want to break up the string over several lines, you can use YAML's [block chomping indicator](http://www.yaml.org/spec/1.2/spec.html#id2794534):

Пам'ятайте, що відповідно до синтаксису [GitHub Actions workflow syntax](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepswithargs) ,`array of strings` не підтримується параметром`jobs.<job_id>.steps.with.args`  Команди Pandoc іноді можуть бути досить довгими і непростими, але ви повинні передавати їх як *одиночний* рядок. Якщо ви хочете розбити рядок на кілька рядків, ви можете скористатись YAML's [block chomping indicator](http://www.yaml.org/spec/1.2/spec.html#id2794534):

```
name: Long Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - run: echo "foo" > input.txt  # create an example file
      - uses: docker://pandoc/core:2.9
        with:
          args: |>  # allows you to break string into multiple lines
            --standalone
            --output=index.html
            input.txt
```

You can see it in action [here](http://github.com/maxheld83/pandoc-example).

## Advanced Usage

Ви також можете:

- створити вихідний каталог для компіляції; полегшує розгортання результатів.
- завантажте вихідний каталог у сховище артефактів [GitHub's artifact storage](https://help.github.com/en/articles/managing-a-workflow-run#downloading-logs-and-artifacts); Ви можете швидко завантажити результати з вкладки GitHub Actions у вашому репозиторії.

Пам’ятайте, що підміна підстановки (скажімо, `pandoc *.md`) або інші функції оболонок, які часто використовуються для pandoc, не працюють у полях ` args: ` файлів GitHub Actions yaml . Тут можна використовувати лише [GitHub Actions context and expression syntax](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions) . Якщо ви хочете скористатися такими функціями оболонки, вам потрібно виконати це окремим кроком у полі `run` і зберегти результат у контексті дій GitHub. Нижче наведений робочий процес містить приклад того, як це зробити для об'єднання декількох вхідних файлів.

You can see it in action (haha!) [here](http://github.com/maxheld83/pandoc-example).

```
name: Advanced Usage

on: push

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
      - run: |
          echo "Lorem ipsum" > lorem_1.md  # create two example files
          echo "dolor sit amet" > lorem_2.md
          mkdir output  # create output dir
          # this will also include README.md
          echo "::set-env name=FILELIST::$(printf '"%s" ' *.md)"
      - uses: docker://pandoc/latex:2.9
        with:
          args: --output=output/result.pdf ${{ env.FILELIST }}
      - uses: actions/upload-artifact@master
        with:
          name: output
          path: output
```