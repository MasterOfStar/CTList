# CTList (Golang)
- 支持多账户
- 支持显示文件夹大小
- 支持每天自动签到
- 支持异步缓存
- 支持隐藏指定文件夹和文件
- 支持整个目录,单层目录或单文件访问加密
- 支持展示任意目录,自定义根目录

# 配置文件
#### 无特殊需要,只需要填写账号密码即可 (前4项). 修改配置后需重新启动 `CTList` .
#### `CaptchaMode` 填写为 `"https://api.moeclub.org/SampleCode"` 用于识别登陆验证码. 默认: "0"。
```
[
    {
        "Enable": 1,                                    # 0: Disable, 1: Enbale.
                                                        ## 0: 关闭, 1: 打开
                                                        
        "UserName": "",                                 # Input Phone Number.
                                                        ## 天翼云网盘登陆用户名.
                                                        
        "Password": "",                                 # Input Password.
                                                        ## 天翼云网盘登陆密码
                                                        
        "CaptchaMode": "0",                             # Captcha Mode. 0: Auto Reject, 1: Manual Input, other: API URL. 
                                                        ## 验证码. 0: 遇到验证码拒绝登陆, 1: 手动输入验证, 其他: 自动识别验证码的API.
                                                        
        "RefreshToken": "",                             # Token. * Do Not Modify It.
                                                        ## 天翼网盘会话. 保持默认, 如果出现异常, 请将该值清空.
                                                        
        "SubPath": "/CTList",                           # Index Path. * Unique Per Account.
                                                        ## 指定某账户挂载在网站的某个目录, 多账户时每个目录值必须唯一.
                                                        
        "RootPathId": "-11",                            # Default Root: -11
                                                        ## 设置展示天翼网盘目录的ID, 根目录为 -11.
                                                        
        "HideItemId": "0|-16",                          # Allow Folder and File.
                                                        ## 不展示某个目录或文件, 填写其ID. 每项用"|"分隔.
                                                        
        "AuthItemId": "",                               # HTTP 401.
                                                        ## 加密某个目录或文件. "<文件或者目录的ID>?<加密模式>?<用户名>:<密码>"
                                                        
        "RefreshURL": 189,                              # Min: 180, Max: 1800; Allow Max: 2329
                                                        ## 下载直链缓存的秒数. 超时则被动更新.
                                                        
        "RefreshInterval": 1800                         # Max: Null, Min: 300
                                                        ## 刷新目录结构,如果不常更新,建议设置更长时间.
    }
]
```

# 准备工作
#### `CTList`皮肤文件与`OneList`皮肤文件完全兼容.
#### 可实现在线浏览图片,在线观看视频等其他功能 [点此前往下载](https://github.com/MoeClub/OneList/tree/master/Rewrite/@Theme)
- **授权码** [32位小写字母和数字,一个授权码可以绑定多个用户名,用于启动多账户.] **[*[获取授权码](https://api.moeclub.org/CTListRegister/)*]**
- **主程序** (CTList)
- **配置文件** (config.json)
- **皮肤文件** (index.html)

# 使用示例
#### 将`CTList`, `config.json`, `index.html`三个文件放在同一目录下即可
```
# 默认启动监听 127.0.0.1:5189
# ./CTList -a "<AUTH_TOKEN_32>"
# 直接监听公网.
# ./CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80
```

# 后台运行及开机自启
#### * `/path/to/CTList` 为CTList的绝对路径
```
# 直接运行
/path/to/CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80

# 不打印HTTP访问日志
/path/to/CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80 -l

# 后台运行 (Windows操作系统无效)
/path/to/CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80 -d

# 以下开机自启方式二选一, 推荐 SYSTEMD 方式.
# 开机自启 (CRON)
编辑 /etc/crontab 文件, 并在文件末尾多添加几个空行. (有些系统不留空行会出现不能自启动)
@reboot root /path/to/CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80 -d

# 开机自启 (SYSTEMD)
cat > /etc/systemd/system/ctlist.service <<EOF
[Unit]
Description=ctlist
After=network.target

[Service]
Type=simple
ExecStart=/path/to/CTList -a "<AUTH_TOKEN_32>" -bind 0.0.0.0 -port 80 -l
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
# 设置服务为开机自启
systemctl enable ctlist
# 启动服务
systemctl start ctlist
```

# 寻找目录ID
#### 用于 `RootPathId`, `HideItemId`, `AuthItemId` 配置项
#### 登陆 https://cloud.189.cn ;进入需要操作的目录,查看地址栏最后的数字就是这个目录的ID.
#### 文件ID需要浏览器F12查看请求项.
```
RootPathId: 列表展示的根目录对应的天翼网盘文件夹ID, 天翼网盘根目录ID为 -11 
HideItemId: 在展示目录中隐藏天翼网盘内的文件或文件夹,填写其ID,使用 "|" 分隔
AuthItemId: 在展示目录中加密天翼网盘内的文件或文件夹,使用 "|" 分隔
```

# 加密目录
#### `AuthItemId` 配置项 采用 HTTP 401 认证方式加密
```
# 单个写法
"AuthItemId": "-11?0?UserName:Password"
# 多个写法
"AuthItemId": "-11?0?UserName:Password|-16?1?UserName:Password"

# 字段解析
<文件或者目录的ID>?<加密模式>?<用户名>:<密码>

# 加密模式
0: 只加密这一层文件夹,可以直接访问这层文件夹内部的内容.
1: 加密这个文件夹的所有子项目.
注意: 加密文件选0和1效果相同.
```

# 刷新策略
```
# 4个刷新逻辑完全异步,互不影响.
Token(登陆保活): 60 * 60 * 10
Cookie(会话授权): 60 * 30
RefreshURL(真实下载链接): 189 （配置文件可改 <RefreshURL>）
RefreshInterval(刷新目录结构): 60 * 15  （配置文件可改, 全局最小值生效 <RefreshInterval>）
```

# 使用说明
```
Usage of CTList:
  -bind string
        Bind Address (default "127.0.0.1")
  -port string
        Port (default "5189")
  -a string
        Auth Token.
  -c string
        Config file. (default "config.json")
  -t string
        Index file. (default "index.html")
  -json
        Output json.
  -d
        Run in the background.
  -l
        Less output.
```

# 目录访问
#### `SubPath` 配置项 控制目录访问 
```
# 多账户时,确保 SubPath 项唯一.

当 SubPath 配置为空("")或者为单斜杆("/")时
访问路径为 http://0.0.0.0

当 SubPath 配置为具体字段("/CTList")时, "/CTList" 可以修改成自己喜欢的字段.
访问路径为 http://0.0.0.0/CTList

```

# 反向代理
```
    location ^~ /CTList {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://127.0.0.1:5189;
    }
```

# 天翼云网盘登陆验证码识别API(基于开源OCR识别)
```
# 目前去噪算法只支持天翼云网盘登陆的验证码(其他类型根本识别不出来)
# 使用时可多次尝试下载不同的验证图片进行提交(目前准确率不算高,但可用.)
# 下载天翼云验证码需要添加请求头 "Referer: https://open.e.189.cn/"

# 接口: https://api.moeclub.org/SampleCode
# 方式: POST
# 参数: Base64=<IMAGE_BASE64_CODE>&Type=CTCloud
# 返回: 状态码:200, 显示识别结果. 状态码:404, 识别错误或结果不符合预设规则, 显示为空.
```
