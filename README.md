

## 一键部署

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

> 貌似在这个 repo 下 点击 一键部署貌似 heroku 提示违反某些原则，但是action 正常工作！！建议 fork 时候，项目名字，尽量不要带有 v2ray 关键字。

> 如果被heroku 提示错误，请用 github action 来部署。

> 部署成功后，可以先用浏览器访问 ***.herokuapp.com， 查看页面是否能正常访问。会显示一个随机的维基百科页面。

## Github Actions 管理

请 Fork 本项目到自己的账户下。 Actions 需要以下 Secrets 才能正常工作，这些 Secrets 会被 workflow 中的 [akhileshns/heroku-deploy](https://github.com/AkhileshNS/heroku-deploy) 使用。

具体实现细节，请查看 [workflow 配置文件](./.github/workflows/main.yml). 如何配置， 请查看，[Github Secrets](#github-secrets)

| Name              | Description                                |
| ----------------- | ------------------------------------------ |
| APP_NAME          | 就是你 heroku 项目的名字. 如果你是第一次创建APP，**请确保名字是唯一的**|
| EMAIL             | heroku 账户的 email                      |
| HEROKU_API_KEY    | heroku API key，在 account 设置下可以找到 |
| HEROKU_V2RAY_UUID | V2rayUUID                                |
| HEROKU_TUNNEL_TOKEN | **可选** cloudflare tunnel 的 token    |

> 这样Token一定必须是大写。。请在 heroku 网站创建app，来确保项目的名字的唯一性。

HEROKU_TUNNEL_TOKEN 是可选项，可以忽略. 详细说明，请查看章节 《建立-cloudflare-tunnel-（可选）》

> 请务必生成新的 UUID。使用已有的 UUID 会使自己 V2ray 暴露在危险之下。


PowerShell:

```powershell
PS C:\Users\> New-Guid
```

Shell:

```bash
xxx@xxx:/mnt/c/Users/$ uuidgen
```

### Github Secrets

路径

```text
项目Setting-->Secrets
```

![Secrets](./readme-data/GithubSecrets.gif)

### Heroku API key

路径

```text
heroku Account settings-->API key
```

![Secrets](./readme-data/herokuapikey.gif)

### Github Actions 界面

```text
Actions
```

![Actions](./readme-data/githubactions.gif)

### 重新部署

点击 `Run workflow`, 输入 deploy。 然后就会重新 deploy。

这里可以**选择区域**，但是请确保app没有被创建过。如果要切换区域，请先使用 destroy 删除应用。

![deploy](./readme-data/deploy.png)

### 停止

点击 `Run workflow`, 输入 stop。 然后就会 stop，不在计入小时数。
![stop](./readme-data/stop.jpg)

### 启动

点击 `Run workflow`, 输入 start。 然后就会启动。

![start](./readme-data/start.jpg)

### 删除

点击 `Run workflow`, 输入 destroy  然后就会删除。

![delete](./readme-data/delete-app.png)


## 建立 cloudflare worker （可选）

如果遇到创建的app在正常网络下不能访问，请尝试这个。

可以参考 开头的视频。代码如下。

```javascript
const targetHost = "xxx.herokuapp.com"; //你的heroku的hostname
addEventListener("fetch", (event) => {
  let url = new URL(event.request.url);
  url.hostname = targetHost;
  let request = new Request(url, event.request);
  event.respondWith(fetch(request));
});

// herokuapp 如果长时间不访问就会休眠。增加cron事件监听器以支持定时job访问herokuapp url。
addEventListener('scheduled', event => {
  event.waitUntil(
    handleSchedule(event)
  )
})

async function handleSchedule(event) {
  let url = new URL("https://" + targetHost);
  url.hostname = targetHost;
  let request = new Request(url);
  const resp = await fetch(request);
  //console.log(await resp.text());
}
```
然后添加Worker的触发器以定时访问herokuapp url：
![image](https://user-images.githubusercontent.com/78028446/174426881-cc16c91a-eab1-4900-ad7c-957ede42a67c.png)


如果 worker 不好用，请用自己域名代理 worker
https://owo.misaka.rest/cf-workers-ban-solution/

为 worker 选择速度更快的 IP。
https://github.com/badafans/better-cloudflare-ip

## 建立 cloudflare tunnel （可选）

项目集成 cloudflare tunnel， 在配置 Secrets `HEROKU_TUNNEL_TOKEN` 之后生效。具体怎么配置，请查看 [cloudflare tunnel](./cloudflared-tunnel.md)。

## 使用 Environments 实现 多账户/多app Secrets 管理

文档介绍： https://docs.github.com/en/actions/deployment/using-environments-for-deployment

### 建立 Environments, 并添加 Secrets

1. 创建 Environments
![Environments](./readme-data/Environments.png)
2. 添加 Secrets
![EnvironmentsSercet](./readme-data/EnvironmentsSercet.png)

### 输入环境名字
**一定要确保环境名字是对的，要不然就会用主的Secrets。**
![EnvironmentsDeploy](./readme-data/EnvironmentsDeploy.png)

## VLESS websocket 客户端配置

### JSON

```json
"outbounds": [
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "***.herokuapp.com", // heroku app URL 或者 cloudflare worker url/ip
                        "port": 443,
                        "users": [
                            {
                                "id": "", // 填写你的 UUID
                                "encryption": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "tls",
                "tlsSettings": {
                    "serverName": "***.herokuapp.com" // heroku app host 或者 cloudflare worker host
                }
              }
          }
    ]
```

### v2rayN


换成 [V2rayN](https://github.com/2dust/v2rayN)

别人的配置教程参考，https://v2raytech.com/v2rayn-config-tutorial/.

![v2rayN](/readme-data/V2rayN.jpg)

cloudflare worker ip 配置

![v2rayN1](/readme-data/V2rayN1.jpg)
