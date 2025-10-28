[![en](https://img.shields.io/badge/lang-ru-red.svg)](https://github.com/Kylsha/kolba/blob/main/README.ru.md) 

## Kolba
![Table loook](https://pereverzev.info/easyPlugin/img/img_es.png)

This tool is made for testing Python script files. User should specify a direct path to directory with Python files. Then window will show a list of Python files while you user is able to edit them in some external code editor. So when user double-click a script in Scripter, the most up-to-date version of selected script will be launched. It also helps in a team work, when you have a shared folder between users, they don't need to constantly update script/plugin, they will have the latest version of tool made by someone.

>[!NOTE]
> Scripter tool was greeted by colleagues by its simplicity. From some moment I prefer it more than plugins with repository that should be updated manually sometimes. It is more convenient in case of issue fixes: colleagues have a same local network path to scripts and tell that some script works incorrectly. I fix the script and tell that it is ready to go. Another users don't have to have update something (plugin via repository or zip-file), they just re-run script and that's it.

Scripts which are run from Scripter should have all needed libraries imported in order to work. Otherwise Scripter will tell that something is wrong with selected script and an error will be printed in Python console of QGIS. So, despite the fact that some libraries are imported in QGIS from startup, they should be re-imported in local script file. 

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

This code snippet can be saved as a Python script file (for example, `my_widget.py`) and put in a folder selected as a script path in Scripter. In order to update contents of script list in Scripter widget, a blue refresh button should be pressed. Finally a double click on script will execute it. Same thing can be achieved in by selecting script in a list and pressing a ▶︎ button.

The right part of Scripter window is used to show a description of selected plugins. In order to do that, a file `descriptions.json` should be created in a folder which is selected as a script path. A content of this file should look like that:
```
{
    "test": "A sample Python code snippet.",
    "my_widget": "PyQt5 widget to show layers info"
}
```
where keys are filenames without extensions and values are script file descriptions.

Due to imports of all needed modules (like PyQt widgets and other) scripts can be used in plugin development.

Video guide provided below:

https://github.com/Kylsha/easyPlugin/assets/25682040/67390440-8ca9-4c46-9284-0677c74a3be9

