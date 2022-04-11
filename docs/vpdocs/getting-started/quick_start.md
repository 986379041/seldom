# 快速开始

### 下载浏览器驱动

和Selenium一样，在使用seldom运行自动化测试之前，需要先配置浏览器驱动，这一步非常重要。

__自动下载__

seldom 集成 [webdriver_manager](https://github.com/SergeyPirogov/webdriver_manager) ，提供了`chrome/firefox/ie/edge/opera`浏览器驱动的自动下载。

```shell
> seldom -install chrome
> seldom -install firefox
> seldom -install ie
> seldom -install edge
> seldom -install opera
```

1. 默认下载到当前的`C:\Users\username\.wdm\drivers\` 目录下面。
2. Chrome: `chromedriver` （众所周知的原因，使用的taobao的镜像）。
3. Safari: `safaridriver` （macOS系统自带，默认路径:`/usr/bin/safaridriver`）


### `main()` 方法

`main()`方法是seldom运行测试的入口, 它提供了一些最基本也是重要的配置。

```python
import seldom

# ...

if __name__ == '__main__':

    seldom.main(path="./",
                browser="chrome",
                base_url=None,
                report=None,
                title="百度测试用例",
                tester="虫师",
                description="测试环境:chrome",
                debug=False,
                rerun=0,
                save_last_run=False,
                language="en",
                timeout=None,
                whitelist=[],
                blacklist=[]
    )
```

__参数说明__


* path : 指定测试目录或文件。
* browser : 针对Web UI测试需要指定浏览器（"chrome"、"firefox" 等）。
* base_url : 针对HTTP接口测试的参数，设置全局的URL。
* report : 自定义测试报告的名称，默认格式为`2020_04_04_11_55_20_result.html`。
* title : 指定测试报告标题。
* tester : 指定测试人员, 默认`Anonymous`。
* description : 指定测试报告描述。
* debug : debug模式，设置为True不生成测试HTML测试，默认为`False`。
* rerun : 设置失败重新运行次数，默认为 `0`。
* save_last_run : 设置只保存最后一次的结果，默认为`False`。
* language : 设置HTML报告中英文，默认`en`, 中文`zh-CN`。
* timeout : 设置超时时间，默认`10`秒。
* whitelist :  用例标签（label）设置白名单。
* blacklist :  用例标签（label）设置黑名单。

### 运行测试

__在终端下运行（推荐）__

在终端下运行（推荐）

创建 `run.py` 文件，在要文件中引用`main()`方法，如下：

```py
import seldom

seldom.main()    # 默认运行当前文件中的用例。
```

`main()`方法默认运行当前文件中的所有用例。

```shell
> python run.py      # 通过python命令运行
> seldom -r run.py   # 通过seldom命令运行
```

__设置运行目录、文件__

可以通过`path`参数指定要运行的目录或文件。

```py
import seldom

seldom.main(path="./")  # 指定当前文件所在目录下面的用例。
seldom.main(path="./test_dir/")  # 指定当前目录下面的test_dir/ 目录下面的用例。
seldom.main(path="./test_dir/test_sample.py")  # 指定测试文件中的用例。
seldom.main(path="D:/seldom_sample/test_dir/test_sample.py")  # 指定文件的绝对路径。
```

__运行类或方法__

`seldom -m`命令可以提供更细粒度的运行。

```shell
> seldom -m test_sample # 运行 test_sample.py 文件
> seldom -m test_sample.SampleTest # 运行 SampleTest 测试类
> seldom -m test_sample.SampleTest.test_case # 运行 test_case 测试方法
```
> 这种模式有两个问题：
> 1. 不支持poium，如果要使用，必须手动给`Seldom.driver` 赋值浏览器驱动。
> 2. 如果是Web UI自动化测试，无法自动关闭浏览器，需要手动关闭浏览器`self.close()`


### 失败重跑 & 截图

Seldom支持失败重跑，以及截图功能。

```python
# test_sample.py
import seldom


class YouTest(seldom.TestCase):

    def test_case(self):
        """a simple test case """
        self.open("https://www.baidu.com")
        self.type(id_="kw", text="seldom")
        self.click(css="#su_error")
        #...


if __name__ == '__main__':
    seldom.main(rerun=3, save_last_run=False)
```

__说明__

* rerun: 指定重跑的次数，默认为 `0`。
* save_last_run: 设置是否只保存最后一次运行结果，默认为`False`。

__运行日志__

```shell
> python test_sample.py


              __    __
   ________  / /___/ /___  ____ ____
  / ___/ _ \/ / __  / __ \/ __ ` ___/
 (__  )  __/ / /_/ / /_/ / / / / / /
/____/\___/_/\__,_/\____/_/ /_/ /_/  v2.6.0
-----------------------------------------
                             @itest.info



====== WebDriver manager ======
Current google-chrome version is
Current google-chrome version is 99.0.4844
Get LATEST chromedriver version for 99.0.4844 google-chrome
Driver [C:\Users\fnngj\.wdm\drivers\chromedriver\win32\99.0.4844.35\chromedriver.exe] found in cache

DevTools listening on ws://127.0.0.1:58294/devtools/browser/59f02afe-8c7a-4b20-b8f4-ff20fac07e08
.\ztest_sync.py

XTestRunner Running tests...

----------------------------------------------------------------------
2022-03-18 21:44:23 [INFO] 📖 https://www.baidu.com
2022-03-18 21:44:26 [INFO] ✅ Find 1 element: id=kw , input 'seldom'.
ERetesting... test_case (ztest_sync.YouTest)..1
2022-03-18 21:44:36 [INFO] 📖 https://www.baidu.com
2022-03-18 21:44:38 [INFO] ✅ Find 1 element: id=kw , input 'seldom'.
ERetesting... test_case (ztest_sync.YouTest)..2
2022-03-18 21:44:48 [INFO] 📖 https://www.baidu.com
2022-03-18 21:44:50 [INFO] ✅ Find 1 element: id=kw , input 'seldom'.
ERetesting... test_case (ztest_sync.YouTest)..3
2022-03-18 21:45:01 [INFO] 📖 https://www.baidu.com
2022-03-18 21:45:02 [INFO] ✅ Find 1 element: id=kw , input 'seldom'.
Generating HTML reports...
2022-03-18 21:45:12 [PRINT] generated html file: file:///D:\github\seldom\reports\2022_03_18_21_44_20_result.html
2022-03-18 21:45:12 [PRINT] generated log file: file:///D:\github\seldom\reports\2022_03_18_21_44_20_log.log
E
```

__测试报告__

![](/image/report.png)

点击报告中的`show`按钮可以查看截图。


### 测试报告

seldom 默认生成HTML测试报告，在运行测试文件下自动创建`reports`目录。

* 运行测试用例前

```shell
mypro/
└── test_sample.py
```

* 运行测试用例后

```shell
mypro/
├── reports/
│   ├── 2020_01_01_11_20_33_result.html
└── test_sample.py
```

通过浏览器打开 `2020_01_01_11_20_33_result.html` 测试报告，查看测试结果。

__debug模式__

如果不想每次运行都生成HTML报告，可以打开`debug`模式。

```py
import seldom

seldom.main(debug=True)
```

__定义测试报告__


```py
import seldom

seldom.main(report="report.html",
            title="百度测试用例",
            tester="虫师",
            description="测试环境：windows 10/ chrome")
```

* report: 配置报告名称和路径。
* title: 自定义报告的标题。
* description: 添加报告信息，支持列表, 例如：["OS: windows","Browser: chrome"]。

__XML测试报告__

如果需要生成XML格式的报告，只需要修改报告的后缀名为`.xml`即可。

```py
import seldom

seldom.main(report="report.xml")
```

