# WeatherSense 部署指南

## 第一步：把代码传到 GitHub（免费，网页操作即可）

1. 打开 https://github.com ，注册/登录
2. 右上角 `+` → `New repository`，随便起个名字，比如 `weathersense`
3. 选 "Add a README file" 不用勾（我们已经有了）
4. 建好仓库后，点 `Add file` → `Upload files`
5. 把这个文件夹里的 4 个文件都拖进去上传：
   - `server.py`
   - `requirements.txt`
   - `Procfile`
   - `README.md`
6. 点 `Commit changes`

## 第二步：部署到 Render.com（免费）

1. 打开 https://render.com ，用 GitHub 账号登录（一键关联）
2. 点 `New` → `Web Service`
3. 选择你刚才那个 `weathersense` 仓库
4. 配置：
   - **Name**：随便起
   - **Runtime**：Python 3
   - **Build Command**：`pip install -r requirements.txt`
   - **Start Command**：`gunicorn server:app`
   - **Instance Type**：选 Free
5. 往下翻到 **Environment Variables**，加两个（改成你自己的）：
   - `HOME_LAT` = 你家的纬度（比如 `31.2304`）
   - `HOME_LON` = 你家的经度（比如 `121.4737`）
   - `CITY_NAME` = 你所在的城市英文名（比如 `Shanghai`）

   > 不知道自己家经纬度？在手机地图 App 里长按你家位置，一般会显示坐标。

6. 点 `Create Web Service`，等 2-3 分钟部署完成
7. 部署好之后，页面顶部会显示一个网址，类似：
   `https://weathersense-xxxx.onrender.com`
   这就是你的服务器地址，记下来。

   打开这个网址，如果看到 `{"status": "WeatherSense server is running", ...}` 就说明成功了。

> 免费版有个小限制：15 分钟没人访问会"睡着"，下次访问要等几秒钟醒过来。如果不想要这个延迟，之后可以升级付费版，或者换成 Railway / Fly.io。

## 第三步：配置 iPhone 上的 SensorLogger

1. App Store 搜索并安装 **SensorLogger**（免费）
2. 打开 App，进入设置（齿轮图标）
3. 找到 **Data Streaming**（HTTP 推送）
4. **Push URL** 填：`https://你的网址.onrender.com/upload`
5. 勾选要上报的传感器：**Location**（位置）、**Battery**（电量）、**Microphone**（麦克风，如果有这个选项）
6. 设置推送频率，比如每 8 秒一次
7. 点开始录制（Start Recording）

之后 SensorLogger 就会自动把数据发到你的服务器了。

## 第四步：给你的 AI 用

你的 AI（比如通过一个自动化脚本、或者你自己复制粘贴）访问：

`https://你的网址.onrender.com/current`

会拿到类似这样的 JSON：

```json
{
  "此刻体感": "开着窗的午后，风一下一下从纱窗钻进来，凉丝丝地舒服。",
  "季节": "夏天",
  "时段": "下午",
  "地点": "在家",
  "温度_c": 30,
  "电量": 45,
  "电量提醒": null
}
```

把 `此刻体感` 这句话，作为一段上下文喂给你的 AI（比如放进系统提示词，或者每隔一段时间发给它），它就"知道"你此刻的环境了。

## 关于苹果的环境光

iPhone 系统不开放光线传感器给 App，这一项目前拿不到。如果之后想要，
教程原文建议：买一块 ESP32（几十块钱）+ BH1750 光照传感器（几块钱），
放在房间里，通过 WiFi 把数据发到你这个服务器的 `/upload` 接口（需要再加一小段接收代码，
到时候可以再来找我加）。

## 关于体感句子库

现在代码里 `SEED_SENTENCES` 只写了 5 个例子。想让它更有灵魂，
可以按第五节的"6轴"，自己往这个字典里加更多你亲手写的句子——
写身体的感觉，不要写天气预报，一句话 35~50 字最好。
没写到的组合，会用一套简单规则自动拼一句朴素的话兜底。
