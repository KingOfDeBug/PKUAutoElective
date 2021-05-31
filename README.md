# PKUAutoElective

## 由本人修改，支持暑校选课


北大选课网 **补退选** 阶段自动选课小工具 v6.0.0 (2021.03.12)

目前支持 `本科生（含辅双）` 和 `研究生` 选课

## 停更说明

感谢大家两年多来对这个项目的支持！我已经大四了，这学期结束后我将毕业，那之后我将不再更新这个项目。由于测试较为困难，这个项目一直以来基本都没有接受过 PR，如果有更改和扩充功能的需要，建议你 fork 一个自己的分支。停更后该项目不会 archive，有任何问题仍然可以在 Issue 里共同讨论，如果你想宣传自己的改版后的分支，也可以在 Issue 里分享

小白兔 写于 2021.03.12

## 注意事项

特地将一些重要的说明提前写在这里，希望能得到足够的重视

1. 不要使用过低的刷新间隔，以免对选课网服务器造成压力，建议时间间隔不小于 4 秒
2. 选课网存在 IP 级别的限流，访问过于频繁可能会导致 IP 被封禁

## 特点

- 运行过程中不需要进行任何人为操作，且支持同时通过其他设备、IP 访问选课网
- 利用专门训练的 CNN 模型自动识别验证码，识别准确率为 99.16%，详细见 [PKUElectiveCaptcha2021Spring](https://github.com/zhongxinghong/PKUElectiveCaptcha2021Spring)
- 具有较为完善的错误捕获机制，程序鲁棒性好
- 提供额外的监视器线程，开启后可以通过端口监听进程运行状况，为服务器上部署提供可能
- 支持多进程下的多账号/多身份选课
- 可以自定义额外的选课规则，目前支持互斥规则和延迟规则

## 安装

### Python 3

该项目至少需要 Python 3，可以从 [Python 官网](https://www.python.org/) 下载并安装（项目开发环境为 Python 3.6.8）

例如在 Linux 下运行：
```console
$ apt-get install python3
```

### Repo

下载这个 repo 至本地。点击右上角的 `Code -> Download ZIP` 即可下载

对于 git 命令行：
```console
$ git clone https://github.com/zhongxinghong/PKUAutoElective.git
```

### Packages

安装 PyTorch 外的依赖包（该示例中使用清华镜像源以加快下载速度）
```console
$ pip3 install requests lxml Pillow opencv-python numpy flask -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装 PyTorch，从 [PyTorch 官网](https://pytorch.org/) 中选择合适的条件获得下载命令，然后复制粘贴到命令行中运行即可下载安装 (CUDA 可以为 None)，PyTorch 版本必须要大于 1.4.x，否则无法读取 CNN 模型

示例选项：

- `PyTorch Build`:  Stable (1.8.0)
- `Your OS`: Windows
- `Package`: Pip
- `Language`: Python
- `CUDA`: None

复制粘贴所得命令在命令行中运行：
```console
pip3 install torch==1.8.0+cpu torchvision==0.9.0+cpu torchaudio===0.8.0 -f https://download.pytorch.org/whl/torch_stable.html
```

该项目不依赖 torchvision 和 torchaudio，因此你可以只安装 torch
```console
pip3 install torch==1.8.0+cpu -f https://download.pytorch.org/whl/torch_stable.html
```

PyTorch 安装时间可能比较长，需耐心等待

### 验证码识别模块测试

这个测试旨在检查与验证码识别模块相关的依赖包是否正确安装，尤其是 PyTorch, OpenCV
```console
$ cd test/
$ python3 test_cnn.py

Captcha('er47') True
Captcha('rskh') True
Captcha('uesg') True
Captcha('skwc') True
Captcha('mmfk') True
```

## 基本用法

1. 复制 `config.sample.ini` 文件，所得的新文件重命名为 `config.ini`
    - 直接复制文件，不要新建一个文件叫 `config.ini`，然后复制粘贴内容，否则可能会遇到编码问题
2. 用文本编辑器打开 `config.ini` （建议用代码编辑器，当然记事本一类的系统工具也可以）
3. 配置 `[user]`，详细见注释
    - 如果是双学位账号，设置 `dual_degree` 为 `true`，同时需要设置登录身份 `identity`，非双学位账号两者均保持默认即可
4. 在选课网上，将待选课程手动添加到选课网的 `补退选` 页 `选课计划` 中，并确保它们处在 `补退选` 页 `选课计划` 列表的 **同一页**
    - 如果想刷的课处在不同页，可以参考 [多进程选课](#多进程选课)
    - 该项目无法事前检查选课计划的合理性，只会根据选课的提交结果来判断某门课是否能够被选上，所以请自行 **确保填写的课程在有名额的时候可以被选上**，以免浪费时间。选课失败引发的常见错误可参见 [异常处理](#异常处理)
5. 配置 `[course]` 定义待选课程，详细见注释
6. 如果有需要，可以配置 `[mutex]`,`[delay]`，详细见注释。如要使用，请务必仔细阅读 [自定义选课规则](#自定义选课规则)
7. 配置 `[client]`，详细见注释（如果不理解选项的含义，建议不要修改）
    - `supply_cancel_page` 指定实际刷新第几页，确保这个值等于 (4) 中待选课程所处的页数
    - `refresh_interval / random_deviation` 设置刷新的时间间隔（如果有需要），切记 **不要将刷新间隔改得过短**，以免对选课网服务器造成太大压力
8. 进入项目根目录，运行 `python3 main.py`，即可开始自动选课。


## 高级用法

### 命令行参数

输入 `python3 main.py -h` 查看帮助
```console
$ python3 main.py -h

Usage: main.py [options]

PKU Auto-Elective Tool v6.0.0 (2021.03.12)

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -c FILE, --config=FILE
                        custom config file encoded with utf8
  -m, --with-monitor    run the monitor thread simultaneously
```

### 多进程选课

如果你有多个账号需要选课，那么可以为每一个账号单独配置一个 `config.ini` 然后以不同的配置文件运行多个进程，即可实现多账号同时刷课

例如你为 Alice 和 Bob 同学创建了这两个文件，假设它们处在 `config/` 文件夹中（手动创建）
```console
$ ls

config  main.py

$ ls config/

config.alice.ini  config.bob.ini
```

接下来在两个终端中分别运行下面两个命令，即可实现多账号刷课
```console
$ python3 main.py -c ./config/config.alice.ini
$ python3 main.py -c ./config/config.bob.ini
```

由于选课网单 IP 下存在会话数上限，开启多进程时还需更改 `[client]` 中的 `elective_client_pool_size` 项，合理分配各个进程的会话数。同一 IP 下所有进程的会话总数不能超过 5。建议值： 单进程 3~4; 两进程 2+2; 三进程 1+1+2 ... （保留一个会话给浏览器正常访问选课网）

如果教学网的 `选课计划` 列表很长，想刷的课处在不同页，也可以通过类似的方法实现多页选课。例如：要同时刷第 1 页和第 2 页的课程，那么分别将两页的课配置成两个 `config.ini`，修改相应的 `supply_cancel_page`，然后按照上法运行即可。
```console
$ ls config/

config.p1.ini  config.p2.ini
```

### 监视器

如果你拥有可以同时连上 `elective.pku.edu.cn` 和 `iaaa.pku.edu.cn` 的服务器，你可以在服务器上部署这个项目，并且开启监听线程，配置相应的路由，之后就可以通过外网访问服务器来查看当前运行状态。

示例：

1. 配置 `[monitor]`，修改需要绑定的 `host/port`
2. 在运行时指定额外 `-m` 参数，即 `python3 main.py -m`
3. 利用 `nginx` 进行反向代理。假设监视器线程监听 `http://127.0.0.1:7074`，相应的配置示例如下：
```nginx
# filename: nginx.autoelective.conf
# coding: utf-8

server {

    listen       12345;
    server_name  10.123.124.125;
    charset      UTF-8;

    location / {
        proxy_pass  http://127.0.0.1:7074;
    }
}
```
在这个示例中，通过访问 `http://10.123.124.125:12345` 即可以查看运行状态


该项目为监视器注册了如下路由：
```
GET  /              查看该路由规则
GET  /rules         同 /
GET  /stat          同 /
GET  /stat/course   查看与选课相关的状态
GET  /stat/error    查看与错误相关的状态
GET  /stat/loop     查看与 loop 线程相关的状态
```

例如，请求 `http://10.123.124.125:12345/stat/course` 可以查看与选课相关的状态


### 自定义选课规则

#### 互斥规则

假设你有多个备选方案，它们在选课规则上并不矛盾，可以同时被选上。例如你在考虑选 A 院的概率统计（B）或是 B 院开的概率统计（B），你希望在选上其中两者其一时就不再考虑选另一门，那么你可以定义这两门课为互斥的，之后在上述情境发生时，另一门课会被程序自动忽略，这样就不会发生两者同时被选上的问题。详细见 [Issue #8](https://github.com/zhongxinghong/PKUAutoElective/issues/8)

务必注意的是，如果互斥的几门课在同一回合内同时出现空位，优先级高的课会被首先提交，而低优先级的课会被忽略（关于课程优先级的概念，参看 `config.ini` 中 `[course]` 下的相关注释），虽然多门课同时出现空位的情况比较罕见，但是为了确保在这种情况发生的时候你能选到更加心仪的课，如果你使用了互斥规则，记得将你更希望选上的课定义成更高的优先级

另外，没有必要为本身就是互斥的课定义互斥规则，比如学校一学期只能选一门体育课，如果你现在有十几门体育课候选，你不必额外声明一个互斥规则来指定这些体育课互斥，因为一旦有一门体育课选上，选课网就会拒绝你提交另一门体育课的选课请求，然后其他的体育课在自然的选课失败后就会被自动忽略

#### 延迟规则

假设你有一门课（不妨设为马原）有多个班可以选，但是每个班的受欢迎程度不一样，你希望选择某个比较火爆的班（比如林锋老师的班，不妨设为 A 班），但是它已经被选满了，而另外一个冷门的班（不妨设为 B 班）仍然有很多的名额，你只希望把它作为你的第二志愿，因此你想尝试抢 A 班。但是你发现当你选了 B 班后，出于某些原因（比如 A 班与 B 班时间冲突、不够学分），你将无法再选 A 班，因此你只好把 B 班先撂在一边，去抢 A 班，但是你并不知道 B 班什么时候会被选满，为此你还得不时地监控 B 班当前的选课情况。这个时候你就可以考虑同时添加 A 班和 B 班到选课计划中，并为 B 班定义一个延迟规则。延迟规则使得触发 B 班选课请求的提交必须要满足 B 班的当前空余人数小于等于某个特定的阈值，我们不妨假设这个阈值是 10，那么程序将会在 B 班剩余的空位小于等于 10 的时候，才会提交 B 班的选课请求，否则它将对此视而不见。详细见 [Issue #28](https://github.com/zhongxinghong/PKUAutoElective/issues/28)

如果你要使用这个规则，你必须要注意以下几件事情，以预知一些可能存在的风险：

1. 我已经对相关逻辑做了简单测试，但是考虑到这个规则出错的后果可能是十分严重的，如果你不放心，还是建议你先拿 B 班测试一下，以确保这个规则能够被程序正确实现

2. 情况并不一定总和你想象得那么美好： B 班人少的时候，程序肯定会帮我补选上，因此我一定会有课上的。你必须要考虑一些特殊情况下程序可能会对此无能为力，例如在某个选课时间节点的时候，选课网访问量骤增，程序可能会连不上服务器，然后一直放着不选的 B 班一瞬间就被选满，那么这学期你可能就没有这个课上了。举一个我假想的案例：假设你是一名信科大一的学生，这学期想选程设，同理可以假设有上述情境中的 A 班和 B 班，你试着抢 A 班并给 B 班设置了延迟规则，但是在跨院系选课名额开放的那个下午 5 点，很多想要选修程设的外院系同学涌进选课系统，一瞬间把 B 班的名额抢完了，而你的程序一直卡着没法登录，完美错过了 B 班的选课，于是你这学期可能就没有程设课可以上了。对于这种风险，你必须要权衡一下，是否有必要在跨院系选课名额开放前就率先抢下 B 班。如果你打算见好就收，是不是下午 4:50 这样我就把 B 班给选了呢？并不是！因为这个时候系统已经在维护了，你需要在跨院系选课名额开放的那天的中午 12 点前，趁着系统还没开始维护，就把 B 班给选了，否则你还是有可能会错过 B 班的选课

3. 不要以为空余人数的缩减速率一定是线性的。并不是说平时观察到 B 班每小时少 10 个名额，那么从 20 -> 10 和从 10 -> 0 都将花掉一小时。当延迟规则出现以后，阈值的设置有可能就会变成一次博弈。例如有 30 个程序都在抢 A 班，并均为 B 班设置了延迟规则，假设大家的阈值都是 10，那么当 B 班还剩 10 个名额的时候，它可能在接下来的一瞬间就没了，至少会有 20 个程序将抢不到 B 班。因此，爆发式抢课的情况仍然是有可能发生的

题外话：我个人觉得有课上就已经很好啦，所以最省心的事情是马上选下 B 班，然后当做 A 班不存在 :)

### 自定义 User-Agent 池

在 `user_agents.txt.gz` 中提供了默认的 User-Agent 池，可以通过 `gzip` 工具解压得到 `user_agents.txt` 加以查看

每次从 IAAA 登录时，都会从中随机选择一个 User-Agent 并设置到 IAAA 客户端和 Elective 客户端上，在下次重新登录前将不再更换

如果你需要自定义 User-Agent 池，可以在根目录下创建一个 `user_agents.user.txt`，在其中对其进行定义。格式为一行一条 User-Agent，具体格式可参考 `user_agents.txt`，需要确保文件为 `utf-8` 编码。程序会优先选择读入用户自定义的 User-Agent 池

### DEBUG 相关

在 `config.ini` 的 `[client]` 中：

- `debug_print_request` 如果设置为 `true`，会将与请求相关的重要信息打印到终端
- `debug_dump_request` 会用 `pickle+gzip` 保存请求的 `Response` 对象，如果发生未知错误，仍然可以重新导入当时的请求。关于未知错误，详见 [未知错误警告](#未知错误警告)

### 其他配置项

在 `config.ini` 的 `[client]` 中：

- `iaaa_client_timeout` IAAA 客户端的最长请求超时
- `elective_client_timeout` Elective 客户端的最长请求超时
- `login_loop_interval` IAAA 登录循环每两回合的时间间隔
- `elective_client_max_life` 设置 Elective 客户端的存活时间。超过存活时间的 Elective 客户端会主动登出并自动重登
- `print_mutex_rules` 是否在每次循环时打印完整的互斥规则列表。如果你定义了很复杂的互斥规则，你可以将这个值设为 `False` 以避免每次循环都将整个列表重复打印一遍

## 异常处理

### 系统异常 `SystemException`

对应于 `elective.pku.edu.cn` 的各种系统异常页，目前可识别：

- **请不要用刷课机刷课：** 请求头未设置 `Referer` 字段，或者未事先提交验证码校验请求，就提交选课请求（比如在 Chrome 的开发者工具中，直接找到 “补选” 按钮在 DOM 中对应的链接地址并单击访问）
- **Token无效：** token 失效
- **尚未登录或者会话超时：** cookies 中的 session 信息过期
- **不在操作时段：** 例如，在预选阶段试图打开补退选页
- **索引错误：** 貌似是因为在其他客户端操作导致课程列表中的索引值变化
- **验证码不正确：** 在补退选页填写了错误验证码后刷新页面
- **无验证信息：** 辅双登录时可能出现，原因不明
- **你与他人共享了回话，请退出浏览器重新登录：** 同一浏览器内登录了第二个人的账号，则原账号选课页会报此错误（由于共用 cookies）
- **只有同意选课协议才可以继续选课！** 第一次选课时需要先同意选课协议

### 提示框反馈 `TipsException`

对应于 `补退选页` 各种提交操作（补选、退选等）后的提示框反馈，目前可识别：

- **补选课程成功：** 成功选课后的提示
- **您已经选过该课程了：** 已经选了相同课号的课程（可能是别的院开的相同课，也可能是同一门课的不同班）
- **上课时间冲突：** 上课时间冲突
- **考试时间冲突** 考试时间冲突
- **超时操作，请重新登录：** 貌似是在 cookies 失效时提交选课请求（比如在退出登录或清空 `session.cookies` 的情况下，直接提交选课请求）
- **该课程在补退选阶段开始后的约一周开放选课：** 跨院系选课阶段未开放时，试图选其他院的专业课
- **您本学期所选课程的总学分已经超过规定学分上限：** 选课超学分
- **选课操作失败，请稍后再试：** 未知的操作失败，貌似是因为请求过快
- **只能选其一门：** 已选过与待选课程性质互斥的课程（例如：高代与线代）
- **学校规定每学期只能修一门英语课：** 一学期试图选修多门英语课
- **学校规定每学期只能修一门体育课：** 一学期试图选修多门体育课
- **该课程选课人数已满：** 试图选一门已经满人的课，这本是不允许的操作，但有的时候选课网会偶然出现某门课已选人数突然为 0 的情况，这时提交选课请求会遇到这个错误


## 补充说明

1. 一直遇到 `[310] 您尚未登录或者会话超时,请重新登录` 错误，可能是因为你是双学位账号，但是没有在 `config.ini` 中设置 `dual_degree = true`
2. 不要修改 `config.ini` 的编码，确保它能够以 `utf-8-sig` 编码被 Python 解析。如果遇到编码问题，请重新创建一个 `config.ini`，之后不要使用 `记事本 Notepad` 进行编辑，应改用更加专业的文本编辑工具或者代码编辑器，例如 `NotePad ++`, `Sublime Text`, `VSCode` 等，并以 `无 BOM 的 UTF-8` 编码保存文件
3. 该项目适用于：课在有空位的时候可以选，但是当前满人无法选上，需要长时间不断刷新页面。对于有名额但是网络拥堵的情况（比如到达某个特定的选课时段节点时），程序选课 **不一定比手选快**，因为该项目每次启动前都会先登录一次 IAAA，这个请求在网络阻塞时可能很难完成，如果你已经通过浏览器提前登入了选课网，那么手动选课可能是个更好的选择
4. 不要使用过低的刷新间隔，以免对选课网服务器造成压力，建议时间间隔不小于 4 秒
5. 选课网存在 IP 级别的限流，访问过于频繁可能会导致 IP 被封禁


## 未知错误警告

1. 在 2019.02.22 下午 5:00 跨院系选课名额开放的时刻，有人使用该项目试图抢 `程设3班`，终端日志表明，程序运行时发现 `程设3班` 存在空位，并成功选上，但人工登录选课网后发现，实际选上了 `程设4班（英文班）` 。使用者并未打算选修英文班，且并未将 `程设4班` 加入到 `course.csv` （从 v3.0.0 起已合并入 `config.ini`） 中，而仅仅将其添加到教学网　“选课计划”　中，在网页中与 `程设3班` 相隔一行。从本项目的代码逻辑上我可以断定，网页的解析部分是不会出错的，对应的提交选课链接一定是 `程设3班` 的链接。可惜没有用文件日志记录网页结构，当时的请求结果已无从考证。从这一极其奇怪的现象中我猜测，北大选课网的数据库或服务器有可能存在 **线程不安全** 的设计，也有可能在高并发时会偶发 **Race condition** 漏洞。因此，我在此 **强烈建议： (1) 不要把同班号、有空位，但是不想选的课放在选课计划内； (2) 不要在学校服务器遭遇突发流量的时候拥挤选课。** 否则很有可能遭遇 **未知错误！**

## 历史更新信息

详见 [Realease History](/HISTORY.md)

## 版本迁移指南

详见 [Migration Guide](/MIGRATION_GUIDE.md)

## 责任须知

- 你可以修改和使用这个项目，但请自行承担由此造成的一切后果
- 严禁在公共场合扩散这个项目，以免给你我都造成不必要的麻烦

## 证书

- PKUElectiveCaptcha2021Spring [MIT LICENSE](https://github.com/zhongxinghong/PKUElectiveCaptcha2021Spring/blob/master/LICENSE)
- PKUAutoElective [MIT LICENSE](https://github.com/zhongxinghong/PKUAutoElective/blob/master/LICENSE)
