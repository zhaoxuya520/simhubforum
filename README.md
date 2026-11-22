# SIMHub 云端接入与翻译中转交接

适用版本为 SIMHub Android 25.6。把整个压缩包交给下一位执行者，从本文开始。当前授权只涵盖云端客户端准备和入网。用户要求先不登录 Oracle，已有的 nginx、Headscale、FlClash、443 和 8080 服务须原样保留。

## 交给下一位执行者的话

> 先在你自己的 Linux 执行环境运行下方的阶段一。只在本机查看组网状态，把 BackendState、本机内网 IP、是否看见 100.64.0.1 告诉我。成功后停在这里。即使入网成功，也先不要 SSH 登录服务器。阶段二仅在我另行说可以部署翻译服务后执行。凭据请通过私密输入提供。

## 目前已经具备什么

| 项目 | 配置或可验证的状态 |
| --- | --- |
| Headscale 控制入口 | `http://161.153.97.110:8080`。用户说此处由现有 nginx 分发组网与订阅请求。 |
| Oracle 内网地址 | `100.64.0.1`。用户报告曾从香港测试节点入网并看到 SSH banner，测试节点已删除。 |
| 当前云客户端 | Linux amd64，Tailscale 1.104.7，用户空间模式。现有本机目录为 `/workspace/tools/simhub-tailnet`，控制地址和节点名 `simhub-codex` 已保存。 |
| 本机观察结果 | 本机能取得控制端公钥。注册阶段经当前云出口返回 HTTP 403，直连 8080 也失败。最后一次状态为 `NeedsLogin`，无本机内网 IP，已停止自动重试。无法仅凭这个 403 指定是哪一层拒绝了升级。 |
| 翻译中转 | 包内已备独立 HTTPS 服务。手机连 TCP 8443，服务通过出站 TCP 443 访问 Google。真实域名、受信任证书、Google 项目密钥和 Oracle 上的端口占用情况仍待现场确认。 |
| 本地测试 | 项目 5 项中转测试通过，使用模拟 Google 服务。另外实际启动了本机 HTTPS 8443 监听，用临时证书验证 TLS，未配对的 POST /v1/test 返回预期的 401，并验证了带端口的配对链接。 |

## 阶段一 · 云客户端入网

在解压目录执行。脚本需要 Linux、Bash、Python 3、curl、tar 和 sha256sum。新环境首次安装需要访问 pkgs.tailscale.com。它只启动当前用户的专用 Tailscale 实例，使用私有目录和专用套接字，不接管本机 DNS 或子网路由。已有系统 Tailscale 可继续使用自己的套接字。该方式不需要 root 或 /dev/net/tun。

```bash
bash cloud-client.sh prepare
bash cloud-client.sh join
bash cloud-client.sh status
```

第二行会在终端隐藏输入，粘贴用户单独提供的 Headscale 入网密钥。无人值守环境可以通过 `SIMHUB_AUTHKEY_FILE` 指向平台私密挂载的文件。文件只放密钥本身，限当前账号读取。密钥、SSH 私钥、Google Key 和现有节点的 state 均不随包分发。

在全新的云机，默认目录是 `~/.local/state/simhub-tailnet`（若设置了 XDG_STATE_HOME 则遵循它）。可设置 `SIMHUB_TS_DIR` 指定自己的持久私有目录。多个同时存在的节点可用 `SIMHUB_NODE_NAME` 设置可辨认的名称。云容器重启后可以重跑 prepare 检查或启动本地守护进程。勿在多个主机间复制节点状态文件。

看到 `BackendState = Running` 和分配给本机的 IP 表示已入网。`Oracle listed = True` 表示协调服务器下发的节点列表中含有 100.64.0.1；它不验证 SSH、远端端口或业务可用性。若为 False，记录结果并检查控制面的可见性规则。此处就是本次操作的停止点。

**若仍是 403。** 从专用目录的 `state/daemon.log` 查看最后一条 register/upgrade 错误，只交付脱敏摘要。取到公钥仅表示普通 HTTP 请求可达，Tailscale 注册还需要控制通道的协议升级。请换用用户允许、支持该协议的运行环境，或由云平台管理员确认出口支持，再用同一套脚本。若提示密钥无效或过期，改从用户处取得有效凭据。普通系统 SSH 不会自动获得这个用户空间实例的路由；待用户授权管理时再选用该实例提供的连接方式。

## 阶段二 · 用户另行授权后，部署独立翻译中转

下列操作仅供获准管理 Oracle 的执行者使用。压缩包中的 `relay/` 自成一套部署文件。入口 `serve_tls.py` 使用 25.6 对应的原中转应用，直接在 IPv4 TCP 8443 终结 TLS。它不依赖现有反向代理，也不需要额外开放 80、443、8080 或 8871。手机走公网时，云安全组及主机防火墙须允许来自手机的入站 TCP 8443，服务器需能访问 Google 的 HTTPS 和 WSS。按现有运维方式单独核对这一条入站规则。

**1. 现场检查。** 需要支持 DynamicUser 和 LoadCredential 的 systemd（LoadCredential 要求 systemd 247+）、Python 3.10+ 及 venv。确认手机能解析到服务器的 DNS 名，并准备与该名字匹配、Android 信任的证书完整链和私钥。可以用既有的独立证书交付流程或 DNS 验证。不要为了取证书停现有网站。本文绑定 IPv4；如果域名还发布了 AAAA，需先确认相应 IPv6 路径可达。

```bash
systemctl --version | head -1
python3 --version
sudo ss -ltnp '( sport = :8443 )'
systemctl list-unit-files simhub-translation-8443.service
sudo ls -ld /opt/simhub-translation-8443 /etc/simhub-translation-8443
```

若已监听或已有同名文件、服务，先核实其归属；以下为首次安装步骤。有同名部署时应审查现状，避免覆盖。空闲检查只在 Oracle 本机才有意义。

**2. 安装到独立目录。** 在解压包根目录执行。

```bash
sudo install -d -m 0755 /opt/simhub-translation-8443
sudo install -m 0644 relay/relay.py relay/serve_tls.py relay/requirements.txt /opt/simhub-translation-8443/
sudo python3 -m venv /opt/simhub-translation-8443/.venv
sudo /opt/simhub-translation-8443/.venv/bin/pip install -r /opt/simhub-translation-8443/requirements.txt
sudo install -d -m 0700 /etc/simhub-translation-8443
```

**3. 交付凭据。** 设好下列三个变量，指向服务器上已有的私密文件。Google Key 文件只放 Google 项目签发的一行密钥。通话开关里的配对令牌由本服务另行生成。证书文件使用 PEM 格式，私钥应可供服务无人值守读取。每次证书续期都要把新证书交付到这个专用目录，再重启本服务。

```bash
sudo install -m 0600 "${GOOGLE_KEY_FILE:?请设置文件路径}" /etc/simhub-translation-8443/gemini.key
sudo install -m 0600 "${TLS_CHAIN_FILE:?请设置完整链路径}" /etc/simhub-translation-8443/fullchain.pem
sudo install -m 0600 "${TLS_PRIVATE_KEY_FILE:?请设置证书私钥路径}" /etc/simhub-translation-8443/privkey.pem
sudo install -m 0644 relay/simhub-translation-8443.service /etc/systemd/system/
sudo systemd-analyze verify /etc/systemd/system/simhub-translation-8443.service
sudo systemctl daemon-reload
sudo systemctl enable --now simhub-translation-8443.service
systemctl is-active simhub-translation-8443.service
sudo ss -ltnp '( sport = :8443 )'
```

**4. 验证公网 TLS 和身份认证。** 从手机所在网络或另一台外部主机执行。把 RELAY_HOST 设为真实域名，仅填主机名。curl 默认校验证书；保留此校验。预期状态为 401，因为请求没有设备令牌。超时先查 DNS、入站规则和监听；证书错误查主机名、有效期和完整链。401 表明访问到了受保护接口，Google 权限另测。

```bash
curl -sS --max-time 12 -o /dev/null -w '%{http_code}\n' -X POST "https://${RELAY_HOST:?设置域名}:8443/v1/test"
```

**5. 配对及手机验收。** 在运行中的 Oracle 服务所在主机执行。下面的临时单元与服务共享受保护的设备记录目录，并以同一服务账号生成一部手机专用的令牌。输出的完整 URL 私下交付给用户。把 `/#` 后的一段也一并粘贴到 SIMHub → 通话翻译 → 服务器配对链接，点「保存」。

```bash
sudo systemd-run --quiet --wait --pipe --collect \
  -p DynamicUser=yes -p User=simhub-translation-8443 \
  -p StateDirectory=simhub-translation-8443 -p StateDirectoryMode=0700 -p UMask=0077 \
  /opt/simhub-translation-8443/.venv/bin/python /opt/simhub-translation-8443/relay.py \
  --devices /var/lib/simhub-translation-8443/devices --pair "https://${RELAY_HOST:?设置域名}:8443"
```

点「测试密钥」，分别记录字幕和同传的结果。随包源码请求的字幕模型为 `gemini-3.5-flash-lite`，同传模型为 `gemini-3.5-live-translate-preview`。Google 项目是否有权限、地区是否可用和剩余配额，以实际请求为准。最后用一通经双方同意的测试电话确认字幕出现、译音可听、两种目标语言可以分别保存。交接验收时注明哪些在模拟环境通过、哪些在手机和真实 Google 项目通过。

要暂停本次新增服务，可执行 `sudo systemctl disable --now simhub-translation-8443.service`。设备配对记录保存在该服务独立的 StateDirectory；保留它以便恢复配对。
