# 微信群消息同步

WeChat group message synchronization

## 10行代码实现微信群消息同步（wxpy）

目前，微信群成员的人数上限是500人。而现实中，有很多社群、校友群、同事群的成员数量已超过500人，只能分拆在不同的微信群里面。在聊天过程中，不同微信群的成员无法相互沟通。

去年，宏论工作室介绍了[基于itchat实现微信群消息同步机器人](https://www.jianshu.com/p/7aeadca0c9bd)的方法。现在，我们改用wxpy模块，以更简洁的代码实现微信群消息同步。wxpy模块是在itchat模块的基础上再次封装，所以使用起来更简便。利用本文介绍的代码，每个微信账号都可以变成“机器人”，在指定的微信群之间自动同步消息，打通了500人上限的阻碍，让不同微信群的成员互相沟通。

### 安装wxpy模块：

```bash
    pip install -U wxpy -i "https://pypi.doubanio.com/simple/" 
```
如果要使用该模块的其他功能，可以查看[wxpy官方文档](http://wxpy.readthedocs.io/zh/latest/)。

### 代码：

下面给出群消息同步的完整代码（去除注释文字，只有10行代码）：
```python
    from wxpy import *

    #导入wxpy模块的全部内容

    bot=Bot()

    # 初始化机器人，电脑弹出二维码，用手机微信扫码登陆

    bot.groups(update=True, contact_only=False)

    #微信登陆后，更新微信群列表（包括未保存到通讯录的群）

    my_groups=bot.groups().search('铲屎官')

    #找出名字包括“铲屎官”的群。假设我们有2个微信群，分别叫“铲屎官1群”、“铲屎官2群”。如果有3个或以上的铲屎群，上面这句代码也能全部找出来，并在后面的代码中实现多群同步。

    my_groups[0].update_group(members_details=True)

    #更新“铲屎官1群”的成员列表信息

    my_groups[1].update_group(members_details=True)

    #更新“铲屎官2群”的成员列表信息

    @bot.register(my_groups, except_self=False)

    #注册消息响应事件，一旦收到铲屎群的消息，就执行下面的代码同步消息。机器人自己在群里发布的信息也进行同步。

    def sync_my_groups(msg):

        sync_message_in_groups(msg, my_groups)

        #同步“铲屎官1群”和“铲屎官2群”的消息。包括文字、图片、视频、语音、文件、分享、普通表情、地图等。

    bot.join()

    #堵塞线程，让机器人保持运行
```


把上述10行代码保存为文件sync.py，然后在电脑运行，就能开始同步微信群消息了：
```bash
    python sync.py
```

### 个性化：

我们可以根据具体情境优化代码，以满足个性化要求：

1、在Linux服务器运行机器人，需要使用终端二维码。初始化机器人的代码改为：
```python
    bot=Bot(cache_path=True, console_qr=2)

    #console_qr=2，这个整数可以调整。如果终端底色是白色，则改为负数。
```    
2、如果需要同步的群名字不同，可以用以下命令进行指定：
```python
    my_groups[0]=bot.groups().search('铲屎官群')

    my_groups[1]=bot.groups().search('吃货群')

    #指定同步“铲屎官群”和“吃货群”的消息
```
3、wxpy在同步群消息时，会默认给发消息的群成员添加一个小图标作为临时头像。如果想使用更简洁的方式，可以改用以下代码：
```python
    @bot.register(my_groups, except_self=False)

    def sync_my_groups(msg):

        my_name=msg.member.name+':'

        #给转发的消息加上前缀，显示群成员名字和冒号。群成员名字从备注、群昵称、微信昵称里面按顺序自动获取。

        sync_message_in_groups(msg, my_groups, prefix=my_name)
```
4、在最后增加一条代码，给机器人发送消息，表示代码执行成功
```python
    bot.file_helper.send('Hello')

    #向机器人的文件传输助手发送消息“Hello”

    bot.join()
```

最后，建议用小号做“机器人”，并适当控制同步消息的数量和频率，以免对群成员造成不必要的骚扰，同时不影响个人大号的正常使用。
