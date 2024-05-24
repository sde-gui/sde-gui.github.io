---
layout: default
title: Waterline 0.6.0. Руководство пользователя
---


# Статус версии программы и данного руководства

Версия 0.6.0 является нестабильным снимком разрабатываемой ветки и предназначена для тестирования опытными пользователями. В руководство включены сведения о известных на момент релиза ошибках и ограничениях. О любых ошибках и недоработках, не перечисленных в руководстве, просьба сообщать в баг-трекер по адресу <https://github.com/sde-gui/waterline/issues>.

Если вы читаете этот документ до фактического релиза 0.6.0, значит некоторые из описанных багов будут исправлены к релизу и исключены из документа, зато наверняка будут добавлены новые.

# Сборка программы

Для сборки программы из исходных кодов требуются следующие обязательные компоненты:

```
pkg_modules="gtk+-2.0 >= 2.24.0 \
             gio-unix-2.0 \
             gthread-2.0 \
             gmodule-2.0 \
             jansson >= 2.2 \
             sde-utils-0 >= 0.1 \
             sde-utils-x11-0 >= 0.1 \
             sde-utils-jansson-0 >= 0.1 \
             sde-utils-gtk-0 >= 0.1"
```

Компонент `menu-cache` является опциональным для сборки, однако рекомендуется его наличие, поскольку без него не будет работать меню приложений, апплет launchbar и другие существенные функции панели.

Если в системе установлена динамическая библиотека `libsmfm-gtk2.so.4` её возможности автоматически задействуются при работе программы. На этапе сборки их наличие не требуется.

Для простоты изложения, в этом руководстве предполагается, что вы соберёте `waterline` с установочным префиксом `/usr/`. Все файловые пути, указанные далее с этим префиксом, следует понимать как пути в том префиксе, с которым на самом деле будет собрана программа.

# Конфигурационные файлы

Waterline позволяет использовать произвольное количество именованных профилей настроек. При запуске программы имя профиля указывается при помощи ключа `-p`. Если имя профиля не указано, будет использован профиль `default`.

Waterline читает и сохраняет конфигурационные файлы в каталог `$XDG_CONFIG_HOME/sde/waterline/<профиль>/`. В случае, если файл отсутствует в этом каталоге Waterline выполняет поиск файла по следующему алгоритму:

* Пробует найти файл, используя в качестве конфигурационного префикса каталоги, перечисленные в `$XDG_CONFIG_DIRS`.
* Пробует найти файл, используя в качестве конфигурационного префикса путь `/usr/share/sde-config/`.
* Если имя профиля не равно `template`, подставляет вместо имени профиля значение `template` и повторяет алгоритм поиска, начиная от конфигурационного префикса `$XDG_CONFIG_HOME`.

В случае, если вы как мейнтейнер дистрибутива желаете добавить собственный общесистемный вариант настроек программы, мы рекомендуем установить их в `/usr/share/sde-config/sde/waterline/<имя профиля>/` и затем запускать Waterline с данным профилем. Если вы как администратор системы хотите добавить новый или модифицировать существующий профиль, рекомендуется расположить профиль в одном из каталогов списка `$XDG_CONFIG_DIRS` (по умолчанию `/etc/xdg/` на большинстве систем).

## Структура конфигурационных файлов

В каталоге профиля программа сохраняет следующие файлы:

* `config` — Файл в формате INI, содержащий настройки, общие для всей программы (а не отдельных панелей).
* `panels/` — Каталог, содержащий по одному файлу для каждой панели на экране. Waterline создаёт и удаляет эти файлы автоматически при создании и удалении панелей через пользовательский интерфейс. Файлы имеют формат JSON и суффикс `.js`. Файлы, которые не оканчиваются на `.js`, игнорируются.

Перед любым изменением файла в каталоге `panels/` программа делает резервную копию файла с расширением `.bak`. Если вы случайно удалили панель или потеряли содержимое файла панели из-за программного или аппаратного сбоя, у вас есть шанс восстановить его из резервной копии.

При удалении панели через пользовательcкий интерфейс, удаляется основной файл панели, но не резервный. Таким образом, со временем в профиле могут накапливаться файлы от созданных и затем удалённых панелей. Это поведение не является багом. Вы можете удалить лишние резервные копии вручную.

В дополнение к указанным файлам, программа читает настройки еще из нескольких файлов конфигурации, но никогда не модифицирует их. Содержимое и назначение этих файлов частично документированы ниже при описании апплетов. (**TODO: документировать остальные.**)

## Режим киоска

Waterline может работать в так называемом режиме киоска, что подразумевает использование программы на публично доступных компьютерах. В режиме киоска Waterline блокирует все способы изменения настроек через пользовательский интерфейс. Для включения блокировки в секции `[General]` файла config замените строку `KioskMode=false` на `KioskMode=true`.

# Пользовательский интерфейс

Основной интерфейс программы представляет собой панели с апплетами, прикрепленные к краям экрана. Настройка панелей и апплетов производится через контекстное меню, открываемое при щелчке правой кнопкой по панели. Пункт меню «Свойства» открывает диалог настроек указанного апплета, а пункт «Панель → Настройки панели» открывает диалог настроек панели в целом.

Апплеты могут добавлять собственные пункты в контекстное меню. Например апплет часов добавляет пункт для копирования текущего значения времени в буфер обмена.

Некоторые апплеты полностью переопределяют контекстное меню под свои потребности. В этом случае для доступа к контекстному меню панели удерживайте при щелчке правой кнопкой клавишу `Ctrl`.

**Примечание:** на апплете "Область уведомлений" щелчок правой кнопкой всегда обрабатывается приложением, которому принадлежит иконка уведомления. Это связано с техническими особенностями области уведомлений. Доступ к настройкам этого апплета можно получить только через вкладку «Апплеты панели» в диалоге «Настройки панели.

# Основные настройки панелей

## Положение и размеры

Вкладка «Размещение» диалога настроек панели позволяет задать положение и размеры панели. Панель может быть приклеплена в любому из краёв экрана. Также вы можете задать желаемые размеры и величину дополнительных отступов.

Размер вдоль того края, к которому прикреплена панель (т.е. ширина для панелей у верхнего и нижнего края и высота для панелей у левого и правого края) можно задать как в пикселях, так и в процентах от размера экрана. Также доступен вариант «Динамически», при котором панель будет автоматически менять размер в зависимости от потребностей апплетов. Мы исправили несколько неприятных багов с автоподгонкой размера, но не уверены, что исправили их все. Просьба сообщать о всех проблемах с этим режимом, если таковые возникнут.

Панель может отображаться поверх всех окон или ниже всех окон. Также имеется режим автоскрытия панели и режим «автоопускания» панели. Они отличаются друг от друга тем, что в первом случае неиспользуемая панель полностью скрывается с экрана, а во втором — опускается ниже всех прочих окон.

Около одного края можно разместить любое количество панелей. Если вы желаете, чтобы панели были расположены «рядами», «колонками» и т.п., вам придётся вручную настроить отступы и размеры панелей. Автоматически панели не будут выстраиваться подобным образом. В ходе тестирования программы, мы нашли, что подход с ручным размещением не только проще и надёжнее в реализации, но и концептуально гибче, позволяя вам получить в точности желаемую конфигурацию.

Положение панели на экране можно задать как в диалоге настроек, так и перетаскиваем при помощи мыши. Чтобы перетащить панель, удерживайте клавишу `Ctrl`, зажмите левую кнопку мыши и перетащите панель в желамое место на экране.

Если выбран режим отображения «Ниже всех окон», и выключена опция «Не закрывать окнами, развёрнутыми на весь экран», панели можно использовать как виджеты рабочего стола, так как они могут быть расположены в любом месте экрана и отображаются поверх рабочего стола, но ниже прочих окон.

Если вы желаете сообщить о багах размещения панели, обязательно при описании проблемы указывайте используемый оконный менеджер. Корректная работе Waterline возможна только с оконными менеджерами, совместимыми со спецификацией NETWM. (Включая, но не ограничиваясь: openbox, fluxbox, icewm, xfwm4.)

## Внешний вид

Waterline отображается с использованием темы оформления gtk.

Вкладка «Внешний вид» позволяет задать фон панели, шрифт, размер значков и внутренние отступы. В качестве фона может использоваться фон, предоставленный темой оформления GTK, вручную заданный цвет или вручную заданное изображение. Для того чтобы использовать прозрачный фон, требуется запущенный композитный менеджер. (Как часть функциональности оконного менеджера или в виде отдельного приложения.)

Раскрывающаяся группа «Paddings» позволяет задать отступы между аплетами и внутренние отступы от краёв панели до апплетов. С точки зрения удобства использования, отступы от краёв обычно имеет смысл устанавливать в ноль. Однако некоторые темы оформления выглядят намного лучше с отступами 2-3 пикселя.

Более тонкая настройка внешнего вида возможна только через механизмы тем оформления gtk.

## Список апплетов

Вкладка «Апплеты панели» позволяет удобавлять, удалять и переупорядочивать апплеты, а также изменять их настройки.

##. Интеграция с приложениями

На вкладке «Дополнительно» можно указать предпочитаемый файловый менеджер (будет использоваться для открытия любых файлов, не только каталогов) и эмулятор терминала (будет использоваться для запуска программ, которым требуется терминал). Также возможно указать команду вызова диалога выхода из системы (будет запускаться при нажатии на соответствующий пункт в меню приложений).

Если предпочтительные программы не указаны, waterline пытается автоматически определить предпочитаемые программы в зависимости от DE, в которой она запущена.

# Апплеты

## Апплеты для запуска приложений

### Меню приложений

Апплет представляет собой кнопку, при нажатии на которую открывается меню, содержащее список установленных приложений, список недавно использованных документов и пункты «Выполнить» и «Завершение работы».

Пункт «Выполнить» открывает диалог, позволяющий запустить произвольную команду операционной системы. Если вместо команды ввести URL, будет запущен браузер с указанным URL.

Диалог «Выполнить» также открывается и при щелчке средней кнопкой мыши по апплету.

Недоработка текущей версии: в связи с переходом на новое API, потеряна возможность модифицировать меню апплета. Данная недоработка будет исправлена в следующих выпусках. Мы планируем полностью перепиcать реализацию этого апплета, добавив возможность настраивать через конфиг произвольные выпадающие меню.

Структура меню считывается из конфигурационного файла `plugins/menu/application_menu.js`, который по умолчанию выглядит так:

```json
{
  "items": [
    { "type": "xdg_menu" },
    { "type": "separator" },
    { "type": "recent_documents_menu" },
    { "type": "separator" },
    { "type": "item", "action": "run" },
    { "type": "separator" },
    { "type": "item", "action": "logout" }
  ]
}
```

Самым важным здесь является пункт `{ "type": "xdg_menu" }`. Вместо этого пункта Waterline подставляет меню приложений, формируемое при помощи компонента `menu_cache`.

Назначение остальных пунктов очевидно из их названий.

### Структура каталогов (dirmenu)

Нажатие на этот апплет отображает в виде меню содержимое указанного в настройках каталога. Апплет имеет гибкие, но вполне очевидные настройки, которые не требуют детального описания.

Если библиотека `libsmfm-gtk2` присутствует в системе, щелчок правой кнопкой по пункту меню будет открывать контекстное меню соответствующего файла.

Вы можете использовать этот апплет как простое средство навигации по файловой системе и открытия файлов. Вы также можете поместить в отдельный каталог свои скрипты или символические ссылки на исполняемые файлы и получить на основе этого каталога собственное «меню приложений».

Известные недоработки апплета, которые мы планируем исправить:
* Файлы *.desktop обрабатываются как обычные файлы, а должны — как ярлыки запуска приложений.
* Нет графической индикации, что является символьной ссылкой, а что не является.
* Нет возможности открыть файл от имени суперпользователя.
* Возможно зависание при просмотре свойств файла.
* Спорадические зависания при открытии контекстного меню файла.

Недоработки, которые в обозримом будущем вряд ли удастся исправить (но мы всё равно будем пытаться) :

* Очень долгое открытие меню на каталогах с большим количеством элементов. (Проблема в медленном алгоритме построения меню в gtk.)
* Всплывающая подсказка каталога может перекрываться распахнувшимся подменю каталога.

### Панель запуска приложений (launchbar)

Панель запуска приложений — простой апплет, отображающий иконки для запуска зарегистрированных в системе приложений. В настройках апплета можно выбрать приложения из списка.

Если на одной панели присутствуют и меню приложений, и панель приложений, то иконки на панель приложений можно добавлять прямо из меню приложений: щелчок правой кнопкой по приложению, «Добавить на панель запуска».

### Кнопка (launchbutton)

Просто кнопка для запуска заданной команды — что может быть проще? На самом деле апплет «Кнопка» в Waterline не так-то прост. Вы можете настроить:

* Иконку.
* Отображаемый текст.
* Всплывающую подсказку.
* Команду, запускаемую по щелчку левой кнопкой мыши.
* Команду, запускаемую по щелчку средней кнопкой мыши.
* Команду, запускаемую по щелчку правой кнопкой мыши.
* Команду, запускаемую при прокрутке колесом мыши вверх.
* Команду, запускаемую при прокрутке колесом мыши вниз.

Чтобы указать апплету, что команду следует запускать в эмуляторе терминала, поставьте в начале команды символ амперсанд (&).

Апплетом «Кнопка» можно управлять из скрипта. На вкладке «Интерактивные обновления» укажите интервал обновления (в миллисекундах), команду, затем активируйте чек-бокс «Включить интерактивные обновления». Вот что будет происходить в этом случае:

* Кнопка запускает указанную команду и применяет к себе все инструкции, которые запущенная программа посылает через стандартный вывод.
* Когда программа завершилась, кнопка ждёт указанное количество миллисекунд. После чего снова запускает программу и читает посылаемые инструкции.

Инструкции читаются построчно: одна строка — одна инструкция. Первое слово строки задаёт собственно инструкцию, а остальная часть строки является аргументом инструкции. Поддерживаются следующие инструкции:

* `Title` — установить заголовок кнопки (с тегами разметки)
* `TitlePlainText` — установить заголовок кнопки (как простой текст)
* `Tooltip` — установить текст всплывающей подсказки (с тегами разметки)
* `TooltipPlainText` — установить текст всплывающей подсказки (как простой текст)
* `Icon` — установить иконку
* `Command1` — установить команду, запускаемую по щелчку левой кнопкой мыши
* `Command2` — установить команду, запускаемую по щелчку средней кнопкой мыши
* `Command3` — установить команду, запускаемую по щелчку правой кнопкой мыши
* `ScrollUpCommand` — установить команду, запускаемую при прокрутке колесом мыши вверх
* `ScrollDownCommand` — установить команду, запускаемую при прокрутке колесом мыши вниз
* `BgColor` — установить цвет фона

## Апплеты для управления окнами

### Панель задач

Панель задач отображает кнопки для управления окнами запущенных приложений. Панель задач имеет большое количество настроек и режимов работы и является наиболее сложным апплетом в составе Waterline.

Настройки панели задач можно разделить на следующие группы:

**Что именно отображать на кнопках:**

* Только иконку, только заголовок или иконку и заголовок вместе.
* Отображать ли в панели кнопку закрытия окна.

**Как именно отображать кнопки:**

* Показывать кнопки активных или неактивных окон плоскими или рельефными.
* Окрашивать ли фон кнопки в тон иконки окна.
* Осветлять ли иконки свёрнутых окон.
* Отображать ли вместо иконок миниатюры окон. (Требуется наличие композитного менеджера. Данная возможность является экспериментальной и может вызывать существенно снижение производительности.)
* Подсвечивать ли изменившиеся заголовки окон.
* Подсвечивать ли заголовок активного окна.
* Подсвечивать ли заголовок на кнопке при наведении мыши.
* Мигать, если окно сообщает, что требуется внимание пользователя.

**Какие окна отображать на панели задач:**

* Показывать окна со всех рабочих мест или только с текущего.
  * Во втором случае, если окно с другого рабочего стола сообщает, что требует внимания, показывать ли его кнопку.
* Классический режим без группировки по приложениям.
* Режим «только активное окно». (Превращает панель задач в «просто заголовок окна».)
* Режим с группировкой по приложениям.
* При группировке можно соединять все окна одного приложения в одну кнопку, а можно просто группировать в последовательность рядом расположенных кнопок. (Сворачивание и разворачивание групп.)
* При каких условиях группы будут сворачиваться и разворачиваться автоматически.
* Также возможно сворачивать и разворачивать группы вручную.

**В каком порядке отображать кнопки:**

* Сортировать по времени создания, времени активации, названию, состоянию или рабочему столу.
* Разрешать пользователю переупорядочивать кнопки вручную.

Баг в текущем релизе: переупорядочивание вручную работает как попало, можно сказать, что вообще не работает. Требуется серьёзный рефакторинг для исправления бага.

**Как управлять окнами:**

Панель задач позволяет привязать действия на следующие события: левая кнопка мыши, средняя кнопка мыши, правая кнопка мыши, прокрутка колесом мыши вверх, прокрутка колесом мыши вниз, а также аналогичные события, но с зажатой клавишей Shift. Действия могут быть таковы:

* Показать контекстное меню окна.
* Закрыть.
* Активировать/cвернуть.
* Распахнуть на всю рабочую область.
* Скрутить в заголовок. (Поддерживают не все оконные менеджеры.)
* Отключить декорации окна. (Поддерживают не все оконные менеджеры.)
* Распахнуть на полный экран.
* Закрепить на всех рабочих столах.
* Показать список всех окон.
* Показать список всех окон группы.
* Активировать следующее окно.
* Активировать предыдущее окно.
* Активировать следующее окно в активной группе.
* Активировать предыдущее окно в активной группе.
* Активировать следующее окно в указанной группе.
* Активировать предыдущее окно в указанной группе.
* Копировать заголовок окна в буфер обмена.

Также вы можете настроить, что будет происходить при наведении мыши на кнопку:

* Будет открываться список окон группы.
* Будет отображаться всплывающая панель с миниатюрами окон. (Требуется наличие композитного менеджера. Данная возможность является экспериментальной и может вызывать существенно снижение производительности.)

*Прочие возможности:*

* Если в оконном менеджере у вас настроено анимированное сворачивание окон, включение пункта `_NET_WM_ICON_GEOMETRY` на вкладке «Интеграция» позволит оконному менеджеру анимировать сворачивание окна в кнопку именно туда, где она находится на экране.
* Если у вас на общей панели находятся панель задач и панель запуска, возможно вам захочется включить опцию «Не отображать иконки видимых приложений в панели запуска» на вкладке «Интеграция».
* В контекстном меню кнопки окна имеется пункт для запуска еще одной копии приложения. Она работает не со всеми приложениями, но когда работает — очень полезен. В будущем мы планируем умную интеграцию приложений со средствами управления окнами, которая заменит этот хак.
* Структура меню окна хранится в конфигурационном файле `plugins/taskbar/task-menu`.

### Свернуть все окна

Щелчок левой кнопкой мыши по этому апплету приводит к сворачиванию всех окон, а щелчок средней кнопкой — к скручиванию всех окон в заголовки.

### Переключатель рабочих мест

Отображает в виде иконок рабочие столы и окна на них и позволяет переключаться между рабочими столами.

Известные недоработки:

* Не поддерживаются рабочие столы в стиле компиза (образованные сегментированием одной большой рабочей области). Аналогичная недоработка и у панели задач.

### Область уведомлений

Отображает иконки уведомлений по протоколу, описанному на сайте FDO.

В настройках можно задать наличие рамки и размер иконок.

## Апплеты для управления параметрами рабочей среды

### Регулятор громкости (ALSA)

Щелчок по апплету отображает полосу регулятора громкости. Также можно регулировать громкость прокруткой колеса мыши над апплетом. Щелчок средней кнопкой глушит/восстанавливает звук. Двойной щелчок запускает микшер. (По умолчанию программа выбирается на основе конфигурационного файла `applications/volume-control`.)

### Индикатор раскладки и переключатель раскладки

Просто работает, документации не требует.

## Индикаторы состояния «физического мира»

### Часы

Настройки апплета часов позволяют задать отображаемый формат времени, формат всплывающей подсказки, шрифт и часовой пояс. По щелчку левой кнопкой открывается встроенный календарь, либо запускается заданная пользователем команда. В контекстном меню апплета имеются пункты для копирования текущего значения времени в буфер обмена в различных форматах. Список форматов берётся из конфигурационного файла `plugins/dclock/formats`.

Формат отображения даты и времени может быть настроен в широких пределах. В качестве примера смотрите [скриншот по ссылке](../screenshots-0.6.0/2013-09-11-waterline-clock.png).

На скриншоте присутствует два очень разных варианта настройки часов — один в левом верхнем углу экрана, другой в правом нижнем.

## Индикаторы состояния компьютера

### Индикатор частоты процессора

Известные недоработки: неизвестно, работает ли он вообще, его никто не проверял.

### Монитор температуры

Отображает температуру с датчика ACPI. Датчик либо выбирается апплетом автоматически, либо задаётся вручную.

### Монитор батареи

Отображает уровень заряда батареи текстом в процентах либо в виде полосы. Цвет индикатора меняется в зависимости от уровня заряда.

### Монитор загрузки процессора

Отображает график загрузки процессора, на котором разными цветами отмечены состояния user, system, idle и iowait.

### Монитор статуса сети

Отображает иконку состояния сетевого интерфейса. По щелчку открывает диалог с подробной информацией.

## Разный мусор

### Регулятор громкости (OSS)

Никто не проверял его работоспособность.

Планируется объединение с регулятором ALSA в рамках рефакторинга обоих.

### Имя рабочего стола

Ненужный апплет, функциональность должна быть добавлена в переключатель рабочих столов.

### Клавиатурный индикатор

Показывает состояние lock-ов клавиатуры. Работает, но никому не нужен.

## Управление программой через waterlinectl

**FIXME: написать главу**

# Разные советы

Во всех полях ввода, которые указывают путь к какому-либо файлу, можно использовать сокращения: если путь начинается с тильды и слэша (`~/`), то подразумевается путь от домашнего каталога текущего пользователя. Если путь начинается с `~имя`, то подразумевается путь от домашнего каталога пользователя `имя`.

Во всех полях ввода, где указывается путь к иконке, можно указывать как полный путь, так и краткое имя. Краткое имя подразумевает, что поиск иконки будет происходить в наборе иконок, указанном в настройках gtk.