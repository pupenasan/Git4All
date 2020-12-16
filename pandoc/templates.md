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

- [Template syntax](templatesyntax.md)
- [variables](variables.md)

