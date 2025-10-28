[![en](https://img.shields.io/badge/lang-ru-red.svg)](https://github.com/pavelpereverzev/kolba/blob/main/README.ru.md) 

## Kolba
![Table loook](https://gisworks.ru/qgis_tools/img/kolba.png)

Kolba is a tool made for running and testing Python script files. User should specify a direct path to directory with Python files. Then window will show a list of Python files while you user is able to edit them in some external code editor. So when user double-click a script in Kolba, the most up-to-date version of selected script will be launched. It also helps in a team work, when you have a shared folder between users, they don't need to constantly update script/plugin, they will have the latest version of tool made by someone.

>[!NOTE]
> Kolba tool was greeted by colleagues by its simplicity. From some moment I prefer it more than plugins with repository that should be updated manually sometimes. It is more convenient in case of issue fixes: colleagues have a same local network path to scripts and tell that some script works incorrectly. I fix the script and tell that it is ready to go. Another users don't have to have update something (plugin via repository or zip-file), they just re-run script and that's it.

Scripts which are run from Kolba should have all needed libraries imported in order to work. Otherwise Kolba will tell that something is wrong with selected script and an error will be printed in Python console of QGIS. So, despite the fact that some libraries are imported in QGIS from startup, they should be re-imported in local script file. 

### Quickstart

For example there is a script which can be run from Python console:

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
> Here `iface` and `PyQt5` elements are imported in order to make script run from Scripter.

There is also a way to prevent widgets from opening them as multiple instances. Kolba is adding a new attribute for an `iface` object: `iface.kolba_plugin`. It is basically a dict which is used for keeping instances of running widgets. 
So if you need to run only single instance of specific widget, add strings:

```
iface.kolba_plugin[script_name] = self # - where widget is shown
...
iface.kolba_plugin[script_name] = None # - where widget is closing, i.e. closeEvent function
```

Edited example:
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
        iface.kolba_plugin[script_name] = self # widget is stored in Kolba dict, so it won't open more than one time

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        iface.kolba_plugin[script_name] = None # widget is reset in Kolba dict, so it is ready for re-run

app = TestWidget()
```

This code snippet can be saved as a Python script file (for example, `my_widget.py`) and put in a folder selected as a script path in Scripter. In order to update contents of script list in Kolba widget, a blue refresh button should be pressed. Finally a double click on script will execute it. Same thing can be achieved in by selecting script in a list and pressing a ▶︎ button.

### Web Scripts

From version 1.2 scripts can be downloaded from web urls. Find a green arrow button in Kolba path line and press it.
In WebScripts window enter the direct URL to Pyhton script file. 
For example, `https://gisworks.ru/qgis_tools/my_widget.py`

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_webscript.png)

After that, press search button. Some time later there will be answer wether script found or not. If found, is there description of it.
If URL is valid script can be saved with a button `Save`. After that script will be appeared in a list of Kolba.

### Descriptions

The right part of Kolba window is used to show a description of selected plugins. 
From version 1.2 all description data stores in script file at the beginning.
Description contain rows like:
* **description** - description itself
* **version** - number of version (needed for updates check)
* **reference** - URL containing information or guide to plugin
* **author** - developer name
* **author_mail** - e-mail of author
* **original_url** - URL where script stores

Check an example:
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
        iface.kolba_plugin[script_name] = self # widget is stored in Kolba dict, so it won't open more than one time

    def get_current_layer(self):
        active_layer = iface.activeLayer()
        if active_layer:
            print(active_layer.name())
        else:
            print('no layers in project')

    def closeEvent(self, event):
        iface.kolba_plugin[script_name] = None # widget is reset in Kolba dict, so it is ready for re-run
```



Video guide provided below:

https://github.com/Kylsha/easyPlugin/assets/25682040/67390440-8ca9-4c46-9284-0677c74a3be9

