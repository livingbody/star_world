#  一、一步一步教你用wechaty+百度云主机打造一个带你穿越星际的微信机器人

* wechaty呢，就是一个神奇得能够沟通各种聊天工具得中间件包括但不限于微信（这样严谨不？）
* 百度云呢是用得它得主机，速度很快
* 走你！

![](https://ai-studio-static-online.cdn.bcebos.com/7b51cd5655604889bb9486f0fa8a1d3aa97a9a19d7cb4696a2aadcc3b2a60391)


# 二、搞定服务器
## 1.活动地址
百度云地址[https://cloud.baidu.com/campaign/bccdiscount/index.html?track=cp:nsem|pf:pc|pp:nsem-huodong-21bccerqi-bcc-124|pu:pinpaici|ci:|kw:10272805#2&bd_vid=8273677131822923008](https://cloud.baidu.com/campaign/bccdiscount/index.html?track=cp:nsem|pf:pc|pp:nsem-huodong-21bccerqi-bcc-124|pu:pinpaici|ci:|kw:10272805#2&bd_vid=8273677131822923008)

找到新人免费上云活动，然后开始！！！

![](https://ai-studio-static-online.cdn.bcebos.com/f7a136f7fddb492e894f2c4c066d82d31b7454d06b3246debbc9febfcfb9553e)

## 2.注册
到手！！！
![](https://ai-studio-static-online.cdn.bcebos.com/593fb5777f9f4b1686862a68663320bebab53db053cf4d86b04fc15ba4f984f1)

## 3.修改管理密码
控制台修改密码就不说了，改完后ssh连接


# 三、wechaty配置
## 1.tocken申请
免费token申请地址: http://pad-local.com
(温馨提示: 免费的token有效期为7天，如需使用有效期更长的token，请访问wechaty官网: https://wechaty.js.org/)

## 2.docker下载及配置
在终端里输入以下指令
```

 apt update

 apt install docker.io

 docker pull wechaty/wechaty:latest

 export WECHATY_LOG="verbose"

 export WECHATY_PUPPET="wechaty-puppet-wechat"

 export WECHATY_PUPPET_SERVER_PORT="8080"

 export WECHATY_TOKEN="puppet_padlocal_xxxxxx" # 这里输入你自己的token

 docker run -ti --name wechaty_puppet_service_token_gateway --rm -e WECHATY_LOG -e WECHATY_PUPPET -e WECHATY_TOKEN -e WECHATY_PUPPET_SERVER_PORT -p "$WECHATY_PUPPET_SERVER_PORT:$WECHATY_PUPPET_SERVER_PORT" wechaty/wechaty:latest
```
## 3.启动docker以后输出如下
![](https://ai-studio-static-online.cdn.bcebos.com/ebe723a559a64734a6bb1ae2e16b7c9935d79b81d4654881b09683fe12cb57b4)

微信扫描二维码即可登录，登陆成功有提示
## 4.注意事项
获取docker镜像比较慢，有两种办法
* 1.切换国内docker mirror点
* 2.直接 docker pull wechaty/wechaty:latest& 后台下载，不用管它

# 四、本地配置

## 1.wechaty环境配置


```python
!pip install wechaty==0.7dev17
```

## 2.demo示例
* 注意区分操作系统
* 注意token不用双引号（此坑踩了会）

## 3各平台配置
### 3.1 linux
```
export WECHATY_PUPPET=wechaty-puppet-service
export WECHATY_PUPPET_SERVICE_TOKEN=puppet_padlocal_*************
```
### 3.2 win
```
set WECHATY_PUPPET=wechaty-puppet-service
set WECHATY_PUPPET_SERVICE_TOKEN=puppet_padlocal_*************
```

```
iimport os
import asyncio
import paddlehub as hub
import cv2
import time
from PIL import Image

from wechaty import (
    Contact,
    FileBox,
    Message,
    Wechaty,
    ScanStatus,
)

model = hub.Module(name="humanseg_lite")


def img_koutu():
    img_path = r'dongman/dongman.jpg'
    # 图片转换后存放的路径
    img_new_path = os.path.join('humanseg_output', 'dongman' + '.png')
    print(img_new_path)
    res = model.segment(
        paths=[os.path.join(img_path)],
        visualization=True,
        output_dir='humanseg_output')
    # 返回新图片的路径
    while not os.path.exists(img_new_path):
        time.sleep(1)
    return img_new_path


def merge(back_img_path, img_path):
    # import cv2
    # cv2.namedWindow("logo")  # 定义一个窗口
    new_img_path = r'merge/result.png'
    frame = cv2.imread('3.jpg', cv2.IMREAD_COLOR)  # 捕获图像1
    # IMREAD_UNCHANGED  If set, return the loaded image as is (with alpha channel, otherwise it gets cropped).
    # 因此Png必须是4通道的IMREAD_UNCHANGED
    logo = cv2.imread('humanseg_output/dongmantime=1626272659.png', cv2.IMREAD_UNCHANGED)
    rows, cols, channels = logo.shape
    dx, dy = 120, 150
    roi = frame[dx:dx + rows, dy:dy + cols]
    for i in range(rows):
        for j in range(cols):
            if not (logo[i, j][3] == 0):  # 透明的意思
                roi[i, j][0] = logo[i, j][0]
                roi[i, j][1] = logo[i, j][1]
                roi[i, j][2] = logo[i, j][2]
    frame[dx:dx + rows, dy:dy + cols] = roi
    cv2.imwrite(new_img_path, frame)
    return new_img_path


def dongman(img_path, img_name):
    # 图片转换后存放的路径
    img_new_path = r'dongman/dongman.jpg'
    print(img_new_path)
    model = hub.Module(name='animegan_v2_shinkai_33')
    result = model.style_transfer(images=[cv2.imread(img_path)], visualization=True,
                                  output_dir='dongman')
    cv2.imwrite(img_new_path, result[0])
    # 返回新图片的路径
    # while not os.path.exists(img_new_path):
    #     time.sleep(1)
    return img_new_path


async def on_message(msg: Message):
    if msg.text() == 'ding':
        await msg.say('这是自动回复: dong dong dong')
    if msg.text() == 'hi' or msg.text() == '你好':
        await msg.say('这是自动回复: 机器人目前的功能是\n- 收到"ding", 自动回复"dong dong dong"\n- 收到"图片", 自动回复一张图片')
    if msg.text() == '图片':
        url = 'https://ai.bdstatic.com/file/403BC03612CC4AF1B05FB26A19D99BAF'
        # 构建一个FileBox
        file_box_1 = FileBox.from_url(url=url, name='xx.jpg')
        await msg.say(file_box_1)
    # 如果收到的message是一张图片
    if msg.type() == Message.Type.MESSAGE_TYPE_IMAGE:
        # 将Message转换为FileBox
        file_box_2 = await msg.to_file_box()
        # 获取图片名
        img_name = file_box_2.name
        # 图片保存的路径
        img_path = './image/' + img_name
        # 将图片保存为本地文件
        await file_box_2.to_file(file_path=img_path)
        # 调用图片风格转换的函数
        img_new_path = dongman(img_path, img_name)
        print(img_new_path)
        img_new_path = img_koutu()
        print(img_new_path)
        img_new_path = merge('3.jpg', img_new_path)
        print(img_new_path)
        # 从新的路径获取图片
        file_box_3 = FileBox.from_file(img_new_path)
        await msg.say(file_box_3)


async def on_scan(
        qrcode: str,
        status: ScanStatus,
        _data,
):
    print('Status: ' + str(status))
    print('View QR Code Online: https://wechaty.js.org/qrcode/' + qrcode)


async def on_login(user: Contact):
    print(user)


async def main():
    # 确保我们在环境变量中设置了WECHATY_PUPPET_SERVICE_TOKEN
    if 'WECHATY_PUPPET_SERVICE_TOKEN' not in os.environ:
        print('''
            Error: WECHATY_PUPPET_SERVICE_TOKEN is not found in the environment variables
            You need a TOKEN to run the Python Wechaty. Please goto our README for details
            https://github.com/wechaty/python-wechaty-getting-started/#wechaty_puppet_service_token
        ''')

    bot = Wechaty()

    bot.on('scan', on_scan)
    bot.on('login', on_login)
    bot.on('message', on_message)

    await bot.start()

    print('[Python Wechaty] Ding Dong Bot started.')


asyncio.run(main())

```

## 4.本地开发运行
`python main.py`

![](https://ai-studio-static-online.cdn.bcebos.com/252d32e4ad6e44a2b2708254e39126e19070fa5b08644c23b0cda6ef7e2bdf2b)


# 五、星际穿越的流程
计划在本机上开发赋能，程序测试完成OK后，再上服务器，敬请期待2天

![](https://ai-studio-static-online.cdn.bcebos.com/05b523c33aac4c7289227a80588c5fcfac45648729c14b5eaa66ea0158dc19d7)

## 1.动漫画
![](https://ai-studio-static-online.cdn.bcebos.com/a7207a9f598a4b2fa4d9cb91362206fe5c06d6f03cdf4ebd8cdc1d6ce974dabe)


## 2.抠图
![](https://ai-studio-static-online.cdn.bcebos.com/a9cfd57fa89145deb849496b94e180f7b370c07f71b943979e3a47df7ee7e2e1)


## 3.科幻海报合成
### 3.1 背景
![](https://ai-studio-static-online.cdn.bcebos.com/6a61e5520b95473ea1d2a692f2fd5727714b96acdac2420d93bb98a6cff7d15e)

### 3.2结果
![](https://ai-studio-static-online.cdn.bcebos.com/1e2169af364c4adca601ca2b6f5d59d5fb2e7b5e1835434ba9a8eb4ff8fdd372)



# 六、linux服务器部署
## 1.安装paddlepaddle
```
pip install paddlepaddle
```
## 2.安装paddlehub
```
pip install paddlehub
```
## 3.环境变量设置
```
export WECHATY_PUPPET=wechaty-puppet-service
export WECHATY_PUPPET_SERVICE_TOKEN = "puppet_padlocal_换成你的key"
```
## 4.启动docker
```
docker run -ti --name wechaty_puppet_service_token_gateway --rm -e WECHATY_LOG -e WECHATY_PUPPET -e WECHATY_TOKEN -e WECHATY_PUPPET_SERVER_PORT -p "$WECHATY_PUPPET_SERVER_PORT:$WECHATY_PUPPET_SERVER_PORT" wechaty/wechaty:latest
```

## 5.后台运行程序
```
python main.py &
```
