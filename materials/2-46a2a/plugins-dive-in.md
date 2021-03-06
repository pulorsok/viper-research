### plugins.py

#### 解說

```
plugins.py 裡面主要做了
1. 使用 pkgutil 模組，該模組提供實用 function 以支援 import 系統。
2. 使用 inspect 模組，針對所匯入之模組做一些基本檢查，例如：是否為 class 等。

首先看到 pkgutil 模組，該模組提供走訪特定路徑下 package 的功能。
此處用到的是 pkgutil.walk_packages()
https://docs.python.org/2/library/pkgutil.html

傳入參數通常只會有兩個 (手冊上共三個)
1. path: 走訪模組之路徑
2. prefix: 印出 module name 之前的 prefix，通常為 module.__name__ + '.'

此 function 回傳三個東西
1. module_loader: 尚不知為何物， plugins.py 中沒用到
2. module_name: 用在 __import__ 之參數
3. ispkg: 檢查是否為 package

若被檢查出是個 package，則 continue。
package 與 module 差別如下

A module is a single file (or files) that are imported under one import and used. e.g.
    import my_module

A package is a collection of modules in directories that give a package hierarchy.
    from my_package.timing.danger.internets import function_of_love

http://stackoverflow.com/questions/7948494/whats-the-difference-between-a-python-module-and-a-python-package

若不是，則使用 __import__, 將 module import。

__import__ 為 python 內建函式
傳入值有 name, globals, locals, fromlist, level

name: The function imports the module name

globals, locals: potentially using the given globals and locals 
                 to determine how to interpret the name in a package context.
                 
fromlist: The fromlist gives the names of objects or submodules 
          that should be imported from the module given by name.

level: specifies whether to use absolute or relative imports. 
       The default is -1 which indicates both absolute and relative imports will be attempted.

https://docs.python.org/2/library/functions.html#__import__
手冊範例：
from spam.ham import eggs, sausage as saus

等於
_temp = __import__('spam.ham', globals(), locals(), ['eggs', 'sausage'], -1)
eggs = _temp.eggs
saus = _temp.sausage

從定義與手冊上範例來看

fromlist 中寫 dummy 或許是可省略的？
解答：
是可省略，但得剛好 name 參數的值不是 package.module 形式，若是，則會 return package。
但若 fromlist 不為空值，則會 return module。
dummy 意思就是防範這樣的錯誤發生。

__import__ 後，用 inspect.getmembers() 取得 module 裡 all the members。
function 回傳 member_name 及 member_object。
member_object 會先被檢查是否為 class。
接著再被檢查是否繼承自 Module 且 member_object 不是 Module 本身。
若通過檢查，

就用

plugins[member_object.cmd] = dict(obj=member_object, description=member_object.description)
將相對應指令塞入 dict 的 key 值，並透過建立巢狀字典，將 obj=member_object。
開發者就可透過 module = __modules__[root]['obj']() 初始化模組的 instance。

參考文獻：https://docs.python.org/2/library/inspect.html
```
