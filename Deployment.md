# Enflame Radar部署文档
官方链接：https://github.com/thoughtworks/build-your-own-radar


## 服务端部署-Docker镜像版
1. Setup：
   * 系统：Ubuntu 22.04
   * 资源配置：4vCPU 8GB内存 30GB存储
   * IP地址：10.9.231.163
   * 域名：radar.enflame.cn
   * 已联外网
   * 可提供HTTP服务
      ![1-7](https://github.com/shenxiyue/Enflame-Technology-Radar/blob/master/figure_deployment/1-7.jpg?raw=true)
   * 拥有可远程访问的图形化界面
      ![1-8](figure_deployment\1-8.png)
2. 拉取Docker镜像：```docker pull wwwthoughtworks/build-your-own-radar```
   ![1-1](figure_deployment\1-1.jpg)
3. 获取Google客户端ID：
   * 进入```https://console.developers.google.com/apis/credentials```并登录
   * 创建凭据
    ![1-2](figure_deployment\1-2.jpg)
   * 获得客户端ID
    ![1-3](figure_deployment\1-3.jpg)
   **注意：** 测试发现不使用客户端ID也能启用服务。
4. 启动容器以持续开放端口：```docker run --rm -p 8080:8080 -e CLIENT_ID="[Google Client ID]" wwwthoughtworks/build-your-own-radar```
   ![1-10](figure_deployment\1-10.jpg)
   **注意：** 若要停止，可先通过```docker ps```查看运行的容器，再根据容器ID进行关闭。
   ![1-6](figure_deployment\1-6.jpg)
5. 进入浏览器：```http://localhost:8080```
   ![1-9](figure_deployment\1-9.png)
   即可在服务端打开build-your-own-radar页面，输入符合要求的URL可生成雷达图。若要在本地打开，则在浏览器进入```http://[ServerIP]:8080```（**注意：** 如果本机与服务器不在一个内网内，则需要配置端口映射）。


## 服务端部署-本地源码版
1. 下载官方源码：```git clone https://github.com/thoughtworks/build-your-own-radar.git```
   ![6-1](figure_deployment\6-1.jpg)
2. 进入文件夹：```cd build-your-own-radar/```
3. 安装：```npm install```
   ![6-5](figure_deployment\6-5.jpg)
4. 运行：```npm run quality```
   ![6-4](figure_deployment\6-4.jpg)
5. 运行服务：```npm run dev```
   ![6-2](figure_deployment\6-2.jpg)
   ![6-3](figure_deployment\6-3.jpg)
6. 进入浏览器：本机```http://localhost:8080```或者远程```http://10.9.231.163:8080/```
   ![6-6](figure_deployment\6-6.jpg)

## Enflame-radar.csv文件维护
1. （初始）进入Gitlab创建仓库：```http://git.enflame.cn/```
   ![2-1](figure_deployment\2-1.jpg)
   已创建用于维护radar.csv的仓库：```http://git.enflame.cn/int.xiyue.shen/radar.git```
2. 添加本地SSH密钥：
   首先生成本地SSH密钥并获取：
   ![2-4](figure_deployment\2-4.jpg)
   然后添加至Gitlab：
   ![2-3](figure_deployment\2-3.jpg)
3. 拉取仓库并进行后续开发：```git clone git@git.enflame.cn:int.xiyue.shen/radar.git```
   ![2-5](figure_deployment\2-5.jpg)
   例如将在本地修改过的Enflame-radar.csv传至远程仓库：
   ![2-6](figure_deployment\2-6.jpg)
4. raw文件的URL获取：```http://git.enflame.cn/int.xiyue.shen/radar/raw/master/Enflame-radar.csv```
   ![2-7](figure_deployment\2-7.jpg)
   

## Enflame Radar生成
1. 在build-your-own-radar网页中输入URL并执行生成：
   ![3-3](figure_deployment\3-3.png)
2. 查看详细内容：
   ![3-4](figure_deployment\3-4.png)


## Enflame-radar.csv文件监测

### 自动监测<font color="#dd0000">（暂不需要）</font><br/>
1. 服务端新建radar_monitor.py文件：
   ```python
    import requests
    import webbrowser
    import time

    # the URL of .csv file
    api = 'http://git.enflame.cn/int.xiyue.shen/radar/raw/master/Enflame-radar.csv'
    web_page = 'http://localhost:8080'
    last_update = None

    while True:
        info = requests.get(api).json()
        cur_update = info['pushed_at']
    
        if not last_update:
            last_update = cur_update
        # check
        if last_update < cur_update:
            webbrowser.open(web_page) # the command
        time.sleep(600) # check every 600s
   ```
2. 服务端持续运行radar_monitor.py：```python radar_monitor.py```
3. 若监测到更新，自动打开网页：
   ![1-5](figure_deployment\1-5.jpg)
4. 输入URL生成雷达图并保持页面打开：
   ![3-1](figure_deployment\3-1.jpg)

### 人工监测
1. 用户修改Enflame-radar.csv文件，维护人员得到提示。
2. 维护人员在服务器上手动开启服务，使容器持续运行。
3. 用户通过访问指定链接查询雷达图信息。


## 用户访问
1. 用户访问```http://radar.enflame.cn:8080```可进入build-your-own-radar网页：
   ![5-1](figure_deployment\5-1.jpg)
2. 用户访问```http://radar.enflame.cn:8080/?sheetId=https%3A%2F%2Fraw.githubusercontent.com%2Fshenxiyue%2FEnflame-Technology-Radar%2Fmaster%2Fradar.csv```可查看当前的雷达图：
   ![5-2](figure_deployment\5-2.jpg)


# 问题
目前在公司虚拟机上部署的服务只能识别并生成github链接的雷达图，google sheet、本机文件和gitlab都不行（连技术雷达官网也是只有github可以），也不生成图，也不报错，如下两图：
![7-1](figure_deployment\7-1.jpg)
![7-2](figure_deployment\7-2.jpg)
|              | 虚拟机+docker镜像 | 虚拟机+本地源码 | 虚拟机+旧版源码 | 虚拟机+官网网页 | 服务器+docker镜像 | 本机+本地源码 | 本机+官网网页 |
|:------------:|:----------------:|:--------------:|:--------------:|:--------------:|:----------------:|:------------:|:------------:|
| Google sheet |         ×       |        ×        |         ×       |         ×       |         ×       |         ×       |        ✔       |
|  Github .csv |         ✔       |        ✔       |        ✔        |        ✔        |        ✔       |        ✔       |        ✔        |
|  Gitlab .csv |         ×       |         ×       |         ×       |         ×       |         ×       |         ×       |         ×       |
