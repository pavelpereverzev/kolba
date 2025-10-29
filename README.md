[![en](https://img.shields.io/badge/lang-ru-red.svg)](https://github.com/pavelpereverzev/kolba/blob/main/README.ru.md) 

# Kolba
![Table loook](https://gisworks.ru/qgis_tools/img/kolba.png)

A tool made for running and testing Python script files in QGIS. 

# Concept

This tool is an alternative to maintaining a local QGIS plugin repository. Previously, I developed plugins for my colleagues and distributed them through a shared repository on our office network. The plugins were used actively, which naturally led to frequent bug reports. The support workflow became burdensome: collecting feedback, fixing issues, repackaging the plugin into a ZIP file, updating the repository, and notifying everyone about the new version. The update notifications, in particular, were inconvenient for users.

To simplify this process, I created Kolba — a wrapper widget for running Python scripts directly in QGIS. It acts as a file browser that displays only Python scripts and allows users to execute them in one click. All scripts are stored in a shared network folder, so if someone reports an issue, I simply fix the script in that folder. Users do not need to update anything — they just run the script again and immediately get the latest version.

This approach has been well-received by colleagues due to its simplicity and minimal maintenance effort.

# Quickstart

The interface of Kolba is simple for understanding: on the top there is a control panel and path to folder with script files. 
Path line contains buttons on the right side:
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_dropdown.png) - dropdown list of saved paths
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_refresh.png) - updates script list by scanning current path 
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_webscript.png) - loader of scripts from web
* ![Table loook](https://gisworks.ru/qgis_tools/img/line_folder_select.png) - folder selector

Below path line there are two sections: the left is script list, the right is a window with description when script is selected.
In addition to two standard icons (pin/unpin, close widget) there are another two buttons:

* ![Table loook](https://gisworks.ru/qgis_tools/img/icon_folder_on.png) - turn on/off path line
* ![Table loook](https://gisworks.ru/qgis_tools/img/path_list.png) - open Kolba settings

Kolba settings contains detailed settings of paths which user can manage: add new folders, delete another, change order of them. 
Also a theme can be set by using a background image for Kolba widget. User should check a `Theme` checkbox and then select an image which can be jpg/png/gif format. Transparency is also can be set.

User should specify a direct path to directory with Python files in a Kolba's header text line box and hit Enter. Then on the left side of Kolba there will be shown a list of Python files. Users can run them while developers are able to edit them in some external code editor. When user double-click a script in Kolba, the most up-to-date version of selected script will be launched. 

Scripts which are run from Kolba should have all needed libraries imported in order to work. Otherwise Kolba will tell that something is wrong with selected script and an error will be printed in Python console of QGIS. So, despite the fact that some libraries are imported in QGIS from startup, they should be re-imported in local script file. 

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
> Here `iface` and `PyQt5` elements are imported in order to make script run from Kolba.

Copy and paste it to a blank Python script file and name it like `my_widget.py`. In order to update contents of script list in Kolba widget, a blue refresh button should be pressed. Finally, a double click on script will execute it. Same thing can be achieved in by selecting script in a list and pressing a ▶︎ button.

Kolba is actually runs a Python script file the same way like in QGIS Python console editor. But it also passes additional variables with running code. They are:
* **wrapper** - Kolba instance
* **project_folder** - current folder in Kolba
* **script_name** - script filename

This variables can be used in some cases. 
For example, in a way to prevent widgets from opening them as multiple instances. Kolba is adding a new attribute for an `iface` object: `iface.kolba_plugin`. It is basically a dictionary which is used for keeping instances of running widgets. 
So if you need to run only single instance of specific widget, add strings:

```
iface.kolba_plugin[script_name] = self # - where widget is shown
...
iface.kolba_plugin[script_name] = None # - where widget is closing, i.e. closeEvent function
```

Completed example:
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

# Web Scripts

From version 1.2 scripts can be downloaded from web URLs. Find a green arrow button in Kolba path line and press it.
In `WebScripts` window enter the direct URL to Python script file. 
For example, `https://gisworks.ru/qgis_tools/my_widget.py`

![Table loook](https://gisworks.ru/qgis_tools/img/kolba_webscript.png)

After that, press search button. Some time later there will be answer whether script found or not. If found, is there description of it.
If URL is valid script can be saved with a button `Save`. After that script will be appeared in a list of Kolba.

# Descriptions

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

You can also upload script to some hosting and let other users to users to download it directly from Kolba.
Script file will not run right after download is completed so you will have a time to check what script actually does.


Video guide provided below:

https://github.com/Kylsha/easyPlugin/assets/25682040/67390440-8ca9-4c46-9284-0677c74a3be9

