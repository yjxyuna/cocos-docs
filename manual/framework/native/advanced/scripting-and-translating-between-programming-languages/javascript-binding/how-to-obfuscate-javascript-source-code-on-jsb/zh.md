# 如何在JSB中调用Java脚本源代码

Obfuscation of javascript sources on the top of cocos2d JSB is available since cocos2d-x v2.1.3
自Cocos2d-x v2.1.3版本以来便支持在Cocos2d JSB中模糊（obfuscation）Java脚本源代码。
Related modifications 相关修订文件
- [https://github.com/cocos2d/cocos2d-x/pull/2245/files](https://github.com/cocos2d/cocos2d-x/pull/2245/files)- [https://github.com/cocos2d/cocos2d-js-tests/pull/140/files](https://github.com/cocos2d/cocos2d-js-tests/pull/140/files)
## 背景知识
### Google Closure Compiler
我们使用**Google Cloure Compiler**来模糊Cocos2d-html5及Cocos2d-x上的java脚本源代码。你可以从[https://code.google.com/p/closure-compiler/downloads/list](https://code.google.com/p/closure-compiler/downloads/list)下载其jar资源包，本机位置为“cocos2d-x/tools/closure-compiler/compiler.jar”。
在使用Closure Compiler之前，需要用到“ant”工具，可从[http://ant.apache.org/](http://ant.apache.org/)下载。安装后你可以运行

```ant -buildfile myconfig.xml```
来生成模糊的java脚本代码，这主要是因为“myconfig.xml”文件的作用。该文件位于“cocos2d-x/tools/closure-compiler/template.xml”，此外“obfuscate.py”文件会扫描“-input”（输入）文件夹，然后将所有文件列表都填入模板中，接着创建“obfuscate.xml”文件。
### 排除（exclude）类名字和函数
模糊cocos2d-html5及cocos2d-x JSB游戏的不同点在于，在cocos2d-x JSB游戏中，从C++导出的函数一定不可以被模糊。本人在以下文件中编写了排除（exclude）函数。
- cocos2d-x/scripting/javascript/bindings/obfuscate/obfuscate_exclude_cocos2d.js- cocos2d-x/scripting/javascript/bindings/obfuscate/obfuscate_exclude_chipmunk.js这些文件在“obfuscate.xml”中如下所示：```<externs dir="${basedir}/scripting/javascript/bindings/obfuscate">	<file name="obfuscate_exclude_cocos2d.js"/>	<file name="obfuscate_exclude_chipmunk.js"/></externs>
```如果你从C/C++导出一些Java脚本绑定API至Java脚本，不要忘了在这里增加排除类（exclude）名字和函数，否则游戏代码在模糊后无法找到C++函数。
### 在Java脚本代码中不要使用“require”
你可以看到，我把所有“require”指令只放到一个文件，这个文件会被包含到调试编译文件（debug build）中。因为调试编译文件为了方便调试不会模糊Java脚本代码。但是在发布编译文件（release build）中，模糊操作会将所有java脚本代码融合到一个文件“game.js”中。
Google Closure Compiler不会模糊java脚本关键字，所以“require(xxx.js)”指令会留在“game.js”文件中。这将会导致运行时期出现“Can't find xxx.js file”错误。实际上，“xxx.js”文件的内容已经在你模糊的“game.js”大文件中。
## 开始工作
**步骤1**：使用obfuscate.py生成obfuscate.xml
本人编写cocos2d-x/tools/closure-compiler/obfuscate.py是为了在“-input”文件夹扫描Java脚本文件，从template.xml文件中生成obfuscate.xml，然后调用ant工具生成game.js。
例如在“samples/javascript/TestJavascript”脚本运行如下。```
Walzers-mbpr:TestJavascript walzer$ /workspace/cocos2d-x/tools/closure-compiler/obfuscate.py -input ../Shared/tests/ -output ./ -cocos2d /workspace/cocos2d-x/preparing configs...generating obfuscate.xml for google closure compilerSuccessful! obfuscate.xml is generateNote: Please reoder the files sequence in obfuscate.xml, keep it the same order as javascript "requrie" instruction,then call "ant -buildfile obfuscate.xml" to obfuscate your js codes.Walzers-mbpr:TestJavascript walzer$ ```**步骤2**：修改obfuscate.xml中的文件顺序
在步骤1中，python脚本会扫描输入文件夹生成obfuscate.xml，该文件被ant工具及Google Closure Compiler利用。但是python脚本还不够聪明，无法识别java脚本代码中的“require”顺序。例如在[https://github.com/cocos2d/cocos2d-js-tests/blob/master/tests/tests-boot-jsb.js](https://github.com/cocos2d/cocos2d-js-tests/blob/master/tests/tests-boot-jsb.js)中存在一个require顺序。现在你需要将这个顺序应用到自己的obfuscate.xml文件中。
**步骤3**：配置项目
你可以以TestJavascript.xcodeproj为例。本人创建了两个目标（target），一个是“TestJavascript”，另一个是“TestJavascriptObfuscated”。
在“TestJavascriptObfuscated”中，“Copy Bundle Resources”阶段前会增加一个“Build Phases”阶段，这个阶段会运行一个脚本文件，每次运行时都会重新从obfuscate.xml中生成game.js文件。而在“Copy Bundle Resources”阶段，只有“game.js”这个java脚本代码会被涉及。