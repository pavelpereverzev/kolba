[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/pavelpereverzev/kolba/blob/main/README.ru.md) 

# Колба
![Table loook](https://gisworks.ru/qgis_tools/img/kolba_1_3.png)

Инструмент для запуска и тестирования Python-скриптов в QGIS.

# Концепция

Колба представляет собой альтернативу использованию репозитория QGIS-плагинов, в том числе локального, развернутого внутри корпоративной сети.
Ранее я разрабатывал плагины для коллег и распространял их через репозиторий в локальной сети офиса. Плагины активно использовались, и по мере работы в них находили ошибки. Процесс поддержки плагинов со временем становился неудобным. После сбора обратной связи нужно было исправлять накопленные ошибки, пересобрать плагин, обновить xml-файл в репозитории, разослать коллегам письмо об обновлении. Последнее было особенно мучительно для всех, подобные рассылки часто терялись в потоке писем и коллегам наверняка было неудобно каждый раз вручную обновлять плагины.

Чтобы упростить этот процесс, я создал Колбу — виджет-обёртку для запуска Python-скриптов напрямую из QGIS. По сути, он работает как файловый браузер, показывая только Python-файлы и позволяя запускать их. Все скрипты я складывал в общую сетевую папку, к которой имели доступ коллеги. Они также указали путь к этой папке в Колбе и видели набор скриптов, с которыми могли работать. Как только в скрипте возникала ошибка, мне об этом сообщали, я делал исправления и все: другим коллегам не нужно было ничего обновлять — они просто запускают скрипт заново и сразу получают его актуальную версию. В дальнейшем это сильно упрощало процесс разработки, учитывая то, что у коллег могут быть разные версии QGIS и другие особенности в системе: можно было гибко подогнать скрипты под каждого.

Стоит сказать, что коллеги тепло приняли такой подход :).

# Начало работы

Интерфейс Колбы довольно простой: в верхней части находится панель управления и строка пути к папке со скриптами.

В строке пути есть кнопки:
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_dropdown.png) - выбор сохраненных путей
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_refresh.png) - обновление списка скриптом в текущей папке 
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_webscript.png) - загрузка скриптов из интернета
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_folder_select.png) - выбор папки

Ниже расположены два блока: слева — список скриптов, справа — описание выбранного скрипта.
В дополнение к стандартным (открепить/закрепить, закрыть виджет) кнопкам в заголовке:

* ![Table loook](https://gisworks.ru/qgis_tools/img/icon_folder_on.png) - показать/скрыть строку пути
* ![Table loook](https://gisworks.ru/qgis_tools/img/path_list.png) - открыть настройки Колба

В настройках можно управлять списком путей: добавлять, удалять, менять порядок. 

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_settings.png)

Также можно установить `тему` — фоновое изображение (jpg/png/gif), с регулируемой прозрачностью.

## Запуск скриптов

Укажите путь к каталогу с .py-файлами в заголовке и нажмите Enter.
Слева появится список скриптов. Двойной клик запустит скрипт. Также можно выбрать скрипт и нажать ▶︎ для запуска. 
Для разработчиков удобство заключается в том, что они могут редактировать файл скрипта в стороннем редакторе, сохранять изменения и после - сразу запускать скрипт из Колбы. Быстро и удобно для всех: и разработчиков, и пользователей.

Важно: в скрипте должны быть импортированы все необходимые модули (к примеру, если это виджет, то нужно импортировать все зависимости PyQt).
Даже если что-то уже доступно в среде QGIS — импорт всё равно должен быть явно прописан в файле.

Например, ниже скрипт, который может быть запущен консоли Python:

```
from qgis.utils import iface
from PyQt5.QtWidgets import QWidget, QPushButton, QVBoxLayout

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
iface.kolba_plugin[script_name] = self # - в том месте, где виджет запущен и показан
...
iface.kolba_plugin[script_name] = None # - в том месте, где виджет закрывается, например, в функции closeEvent 
```

Готовый пример:
```
from qgis.utils import iface
from PyQt5.QtWidgets import QWidget, QPushButton, QVBoxLayout

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
        iface.kolba_plugin[script_name] = self # виджет запиывается в словарь iface.kolba_plugin, поэтому не будет повторно открываться 

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        iface.kolba_plugin[script_name] = None # виджет обнулен в словаре iface.kolba_plugin, и готов к повторному открытию

app = TestWidget()
```

# Web Scripts

Начиная с версии 1.2 можно загружать скрипты по прямым ссылкам. Нажмите кнопку ![Table loook](https://gisworks.ru/qgis_tools/img/line_webscript.png) и в окне `WebScripts` введите ссылку, например, `https://gisworks.ru/qgis_tools/my_widget.py`, нажмите Enter.

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_webscript.png)

Будет выполнен запрос по указанной ссылке и, если ссылка валидная, в окне `WebScripts` появится описание скрипта, а также станет активной кнопка `Save`. При нажатии этой кнопки скрипт сохранится в указанной папке и он же появится в списке скриптов в Колбе.

# Описания

В правой части Колбы показывается описание скриптов. Описание скрипта хранится в его начале в виде многострочного комментария:
* **description** - само описание
* **version** - номер версии
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
from PyQt5.QtWidgets import QWidget, QPushButton, QVBoxLayout

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
        iface.kolba_plugin[script_name] = self # виджет запиывается в словарь iface.kolba_plugin, поэтому не будет повторно открываться 

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        iface.kolba_plugin[script_name] = None # виджет обнулен в словаре iface.kolba_plugin, и готов к повторному открытию
```
Любой скрипт можно загрузить на веб-хостинг и предоставить возможность другим пользователям его скачать, а позже - запустить через Колбу.
Скрипт не будет запускаться сразу после загрузки, у пользователя будет возможность ознакомиться с кодом.



Видео с демострацнией работы в Колбе (устаревшее, но отражает суть использования):

https://github.com/pavelpereverzev/easyPlugin/assets/25682040/67390440-8ca9-4c46-9284-0677c74a3be9

