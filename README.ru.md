[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/pavelpereverzev/kolba/blob/main/README.md) 

# Колба

<p align="center">
<img src="https://gisworks.ru/qgis_tools/img/kolba_window.png?new=true" 
       width="auto">
</p>

Инструмент для запуска и тестирования Python-скриптов в QGIS.

## Концепция

Колба — это альтернатива классическому репозиторию QGIS-плагинов.  
Вместо постоянной пересборки, обновления и рассылки новых версий плагинов она позволяет запускать Python-скрипты напрямую из локальной/общей папки внутри QGIS. Благодаря этому  увеличивается скорость разработки и обновления инструментов: после изменения скрипта все пользователи автоматически работают с его актуальной версией без ручного обновления.

<details>
  <summary>О проекте</summary>
  
Колба представляет собой альтернативу использованию репозитория QGIS-плагинов, в том числе локального, развернутого внутри корпоративной сети.
Ранее я разрабатывал плагины для коллег и распространял их через репозиторий в локальной сети офиса. Плагины активно использовались, и по мере работы в них находили ошибки. Процесс поддержки плагинов со временем становился неудобным. После сбора обратной связи нужно было исправлять накопленные ошибки, пересобрать плагин, обновить xml-файл в репозитории, разослать коллегам письмо об обновлении. Последнее было особенно мучительно для всех, подобные рассылки часто терялись в потоке писем и коллегам наверняка было неудобно каждый раз вручную обновлять плагины.

Чтобы упростить этот процесс, я создал Колбу — виджет-обёртку для запуска Python-скриптов напрямую из QGIS. По сути, он работает как файловый браузер, показывая только Python-файлы и позволяя запускать их. Все скрипты я складывал в общую сетевую папку, к которой имели доступ коллеги. Они также указали путь к этой папке в Колбе и видели набор скриптов, с которыми могли работать. Как только в скрипте возникала ошибка, мне об этом сообщали, я делал исправления и все: другим коллегам не нужно было ничего обновлять — они просто запускают скрипт заново и сразу получают его актуальную версию. В дальнейшем это сильно упрощало процесс разработки, учитывая то, что у коллег могут быть разные версии QGIS и другие особенности в системе: можно было гибко подогнать скрипты под каждого.

Стоит отметить, что коллеги тепло приняли такой подход :).
</details>

## Интерфейс 

Интерфейс Колбы довольно простой: в верхней части находится панель управления и строка пути к папке со скриптами.

В строке пути есть кнопки:
* ![Table look](https://gisworks.ru/qgis_tools/img/line_dropdown.png) - выбор сохраненных путей
* ![Table look](https://gisworks.ru/qgis_tools/img/line_refresh.png) - обновление списка скриптом в текущей папке 
* ![Table look](https://gisworks.ru/qgis_tools/img/line_webscript.png) - загрузка скриптов из интернета
* ![Table look](https://gisworks.ru/qgis_tools/img/line_folder_select.png) - выбор папки

Ниже расположены два блока: слева — список скриптов, справа — описание выбранного скрипта.
В дополнение к стандартным (открепить/закрепить, закрыть виджет) кнопкам в заголовке:

* ![Table look](https://gisworks.ru/qgis_tools/img/icon_folder_on.png) - показать/скрыть строку пути
* ![Table look](https://gisworks.ru/qgis_tools/img/path_list.png) - открыть настройки Колба


Закладки - представлены в версии 1.5, позволяют оперативно переключаться между сохраненными путями в Колбе

# Начало работы

Укажите путь к папке с .py-файлами в строке ввода пути и нажмите Enter.
Слева появится список скриптов. Двойной клик запустит скрипт. Также можно выбрать скрипт и нажать ▶︎ для запуска. 
Для разработчиков удобство заключается в том, что они могут редактировать файл скрипта в стороннем редакторе, сохранять изменения и после - сразу запускать скрипт из Колбы. Быстро и удобно для всех: и разработчиков, и пользователей.

Важно: в скрипте должны быть импортированы все необходимые модули (к примеру, если это виджет, то нужно импортировать все зависимости PyQt).
Даже если что-то уже доступно в среде QGIS — импорт всё равно должен быть явно прописан в файле.

Например, ниже скрипт, который может быть запущен консоли Python:

```
from qgis.utils import iface
from qgis.PyQt.QtWidgets import QWidget, QPushButton, QVBoxLayout

class TestWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.resize(250, 100)
        self.setWindowTitle("Test widget")
        layout = QVBoxLayout(self)
        button = QPushButton("Check active layer")
        layout.addWidget(button)
        button.clicked.connect(self.get_current_layer)
        self.show()

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')
app = TestWidget()
```
>[!NOTE]
> Здесь `iface` и `PyQt5` импортированы в начале, чтобы скрипт можно было запустить из Колбы.

Скопируйте и вставьте код в пустой файл с расширением .py и сохраните с названием `my_widget.py` в папку, которую указали в строке в Колбе. Для того, чтобы файл появился в списке скриптов, нажмите кнопку обновления списка ![Table loook](https://gisworks.ru/qgis_tools/img/line_refresh.png). Двойным кликом запустите скрипт.

Колба запускает скрипт тем же способом, что и QGIS, когда работа ведется в редакторе консоли. В дополнение в выполняемый код передаются дополнительные переменные:

* **wrapper** - экземпляр Колбы
* **project_folder** - текущий путь
* **script_name** - имя скрипта

Использование этих переменных может пригодится в некоторых случаях.
Например, чтобы предотвратить повторное открытие виджетов, Колба добавляет атрибут к `iface`, а именно `iface.kolba_plugin`. Это словарь, который используется для сбора информации об открытых виджетах.
Если требуется предотвратить повторное открытие виджета, добавьте строки:
```
(script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, self) # - в том месте, где виджет запущен и показан
...
(script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, None) # - в том месте, где виджет закрывается, например, в функции closeEvent 
```

Готовый пример:
```
from qgis.utils import iface
from qgis.PyQt.QtWidgets import QWidget, QPushButton, QVBoxLayout

class TestWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.resize(250, 100)
        self.setWindowTitle("Test widget")
        layout = QVBoxLayout(self)
        button = QPushButton("Check active layer")
        layout.addWidget(button)
        button.clicked.connect(self.get_current_layer)
        self.show()
        (script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, self) # виджет запиывается в словарь iface.kolba_plugin, поэтому не будет повторно открываться 

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        (script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, None) # виджет обнулен в словаре iface.kolba_plugin, и готов к повторному открытию

app = TestWidget()
```

Видео с демострацнией работы в Колбе (устаревшее, но отражает суть использования):

https://github.com/pavelpereverzev/easyPlugin/assets/25682040/67390440-8ca9-4c46-9284-0677c74a3be9

## Web Scripts

Начиная с версии 1.2 в Колбе можно загружать скрипты по прямым ссылкам из интернета. Найдите и нажмите кнопку ![Table loook](https://gisworks.ru/qgis_tools/img/line_webscript.png?new=true). 
Окно `WebScripts` позволяет загрузить скрипты по прямым ссылкам или из готовых наборов скриптов, определяемыми ссылкой в **Настройках**
Для загрузки определенного скрипта введите прямую ссылку к нему в строке **Custom script**. Например, `https://gisworks.ru/qgis_tools/my_widget.py`, после чего нажмите Enter. 

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_webscript.png?new=true)

Будет выполнен запрос по указанной ссылке и, если ссылка корректная, в окне `WebScripts` появится описание скрипта, а также станет активной кнопка `Save`. При нажатии этой кнопки скрипт сохранится в указанной папке и он же появится в списке скриптов в Колбе.

**Script set** - это набор готовых пользовательских скриптов, взятых по ссылке. Данный список исключает необходимость поиска скриптов вручную и позволяет просто выбрать и загрузить необходимый инструмент.  

## Описания

В правой части Колбы показывается описание скриптов. Описание скрипта хранится в его начале в виде многострочного комментария:
* **description** - само описание, начиная с версии 1.4, поддерживает HTML-верстку
* **version** - номер версии (необходимо для обновления скриптов)
* **reference** - ссылка с информацией по скрипту
* **author** - имя разработчика
* **author_mail** - e-mail или иной способ связи
* **original_url** - ссылка, по которой можно загрузить скрипт

Пример скрипта с описанием:
```
"""
description: My test tool
version: 1.0
reference: https://github.com/pavelpereverzev/kolba
author: Me
author_mail: me@mail.com
original_url: https://gisworks.ru/qgis_tools/my_widget.py
"""

from qgis.utils import iface
from qgis.PyQt.QtWidgets import QWidget, QPushButton, QVBoxLayout

class TestWidget(QWidget):
    def __init__(self):
        super().__init__()
        self.resize(250, 100)
        self.setWindowTitle("Test widget")
        layout = QVBoxLayout(self)
        button = QPushButton("Check active layer")
        layout.addWidget(button)
        button.clicked.connect(self.get_current_layer)
        self.show()
        (script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, self) = self # виджет запиывается в словарь iface.kolba_plugin, поэтому не будет повторно открываться 

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        (script_name:=globals().get("script_name")) and hasattr(iface,"kolba_plugin") and iface.kolba_plugin.__setitem__(script_name, None) # виджет обнулен в словаре iface.kolba_plugin, и готов к повторному открытию
```
Любой скрипт можно загрузить на веб-хостинг и предоставить возможность другим пользователям его скачать, а позже - запустить через Колбу.
Скрипт не будет запускаться сразу после загрузки, у пользователя будет возможность ознакомиться с кодом.

>[!NOTE]
> Ни одна из строк описания не является обязательной, но **description** помогает коллегам лучше понять инструмент, созданный кем-то другим.
> **version** и **original_url** следует указывать аккуратно, если разработчик решит обновить свой инструмент. Например, после изменения числа в **version** с `1.0` на `1.1` в скрипте, расположенном по адресу `https://gisworks.ru/qgis_tools/my_widget.py`, пользователь увидит кнопку `Update tool` внизу окна описания в Колбе.

## Настройки

Настройки содержат:
- список сохраненных путей: доступны добавление, удаление, редактирование путей и изменение их порядка
- список закладок: позволяет создать кнопки-закладки к путям для быстрой смены текущего пути в Колбе
- корневая папка скриптов WebScript и путь к пользовательскому списку скриптов
- импорт/экспорт настроек путей и закладок плагина
- настройки интерфейса: ориентация разделителя и тема 

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_settings.png?new=true)

**Paths manager** позволяет добавлять, удалять и редактировать пути в плагине. Например, в одной папке расположены стабильно рабочие инструменты, а в другой - скрипты в работе. После добавления путей в самой Колбе можно будет переключаться между папками по выпадающему списку. В списке можно менять порядок путей перетаскиванем.


**Bookmarks manager** позволяет представить путь к папке как кнопку для основного интерфейса Колбы. Чтобы не открывать выпадающий список в основном окне плагина, можно установить некоторые пути как закладки-кнопки и они сразу же появятся в пространстве между текущим путем Колбы и блоками со скриптами и описанием. Доступно добавление как новых путей, так и создание закладок из путей в **Paths manager**.

![Table look](https://gisworks.ru/qgis_tools/img/kolba_bookmarks.png)

**WebScript default location** представляет собой URL-адрес корневой папки скриптов, которые можно загрузить из интернета. URL-адрес по умолчанию — `https://gisworks.ru/qgis_tools`, что означает, что если пользователь введет `my_widget` в поиск WebScript, последний перейдет по пути `https://gisworks.ru/qgis_tools/my_widget.py`.

**WebScript custom set URL** - это ссылка на json-файл, в котором представлен список названий инструментов и ссылок на них. Эти данные потом отображаются в разделе **Script set** в окне **WebScripts**. Ссылка по умолчанию: `https://gisworks.ru/kolba_set.json`.

Образец структуры json для **Script set**:
```
[
    {"id": "script_name", "url": "https://some_site.com/script_name.py"},
    {"id": "another_script_name", "url": "https://some_another_site.com/another_script_name.py"}
]
```


**Import/Export config** - это кнопки для сохранения текущих путей, закладок и настроек WebScript в json-файл. Пользователь может сохранить конфигурацию на одном ПК и перенести ее в Колбу на другом. Может пригодиться в случе командной работы и быстрой настройки Колбы.

**Tools widget orientation** позволяет изменить отображение виджетов списка скриптов и окна описания.

**Theme** - это настройка персонализации Колбы. Пользователь может установить jpg/png-картику или gif-анимацию на фон плагина. Также доступна настройка прозрачности изображения.


![Table look](https://gisworks.ru/qgis_tools/img/kolba_theme.gif)

## История изменений
<details>
  <summary>Раскрыть список</summary>

  ### v1.4.1 
  - чистка кода
  - фикс багов для совместимости кода с новыми проверками плагинов в репозитории
  
  ### v1.4
  - поддержка Qt6
  - исправление вылета в случае некорректной ссылки в WebScript
  - добавлена ссылка по умолчанию в WebScript
  - поддержка HTML-форматирования в окне описания скрипта WebScript
 

  ### v1.3
  - фикс определения стандартного пути для плагина
  - фиксы для Linux и MacOS
  - обновления интерфейса (изменение компоновки виджетов, убраны неиспользуемые элементы, исправление функций)
  - чистка кода

  ### v1.2.1
  - исправление вылета при закрытии окна WebScript

  ### v1.2
  - первая публичная версия плагина
</details>

<p align="center">
  <img src="https://gisworks.ru/qgis_tools/img/kolba_glow_2.svg" 
       width="256">
</p>
