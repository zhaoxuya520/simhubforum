# SIMHub 账号同步与 eSIM 邮件提醒服务

SIMHub Android 26.1 的设置页提供「账号与云同步」。登录后重新打开应用进入独立空间。首次登录可主动选择将本机访客资料移入该账号。进入账号后，在同一页面设置到期提醒邮箱；清空邮箱即可停用。应用进程存活期间约每分钟同步一次，也可以手动同步。两个设备修改不同记录可合并，遇到同一条记录冲突可手动选择采用本机或云端。部分下载的设置和通话记录在下次启动时载入。

同步覆盖卡片、卡组、设备、绑定记录、到期时间、短信数据库、通话记录、壁纸及支持的用户设置。SIM 卡芯片内容、系统通讯录、设备密钥及本机配对凭据属于设备本身。单次同步请求上限 16 MiB。到期任务以最近一次成功同步的数据为准，服务器定时器会在手机离线时继续发信。

## 授权码注册与忘记密码（26.1）

在「设置 → 账号与云同步 → 注册账号」填写账号名、密码和管理员发放的授权码。账号名支持 3–64 位英文字母、数字、点、下划线或短横线，忽略大小写。密码长度为 12–256 个字符。注册成功后长按复制个人恢复码，保存到密码管理器，然后返回登录。

管理员在部署账号服务后可执行以下命令。每次运行会在终端显示一枚独立的注册授权码，7 天内有效，可注册一个账号。向获准注册的用户单独发放。

```sh
sudo -u simhub-account /opt/simhub-account/.venv/bin/python /opt/simhub-account/service.py --db /var/lib/simhub-account/accounts.sqlite invite
```

用户在登录页点「忘记密码」，输入原账号、个人恢复码及新密码。若已丢失恢复码，或账号在 26.0 创建，请由管理员核实账号归属后执行下列命令，将临时重置码私下交给账号持有人。将 USERNAME 换成原账号名。同一账号的新临时码取代上次生成的临时码，30 分钟内有效。

```sh
sudo -u simhub-account /opt/simhub-account/.venv/bin/python /opt/simhub-account/service.py --db /var/lib/simhub-account/accounts.sqlite reset-code USERNAME
```

密码重置后，账号资料、提醒邮箱与到期任务均保留，所有旧会话和原恢复码失效。页面会给出新的个人恢复码，请保存。已登录设备可从「账号与云同步」退出，重新打开应用并用新密码登录。个人恢复码持续有效，直至被使用或账号密码被重置。

从 26.0 升级时保留已有 accounts.sqlite 及其目录，更新账号服务的 service.py 和当前 8443 入口加载的 accounts_gateway.py，然后重启这两个服务。程序启动会自动添加授权码表。原账号可继续登录。

## 运行条件

服务用 SQLite 保存账号、会话、快照和提醒任务。API 默认监听 127.0.0.1:8872，由部署人通过手机预设的 https://161.153.97.110:8443 暴露 `/accounts/v1/`。请按现有服务器的运维安排选择入口。程序访问 Cloud Mail 只需要出站 HTTPS，发件协议为教程中的 HTTP API。手机不需要安装组网客户端。

安装 Python 3.10+，创建虚拟环境并安装 `requirements.txt`。systemd 文件假定程序位于 `/opt/simhub-account`，账号 `simhub-account` 可读取程序，并通过 StateDirectory 持有数据库。第一次启动 API 服务后，以同一服务账号运行以下命令创建用户。程序会安全提示输入并确认密码。

```sh
sudo -u simhub-account /opt/simhub-account/.venv/bin/python /opt/simhub-account/service.py \
  --db /var/lib/simhub-account/accounts.sqlite add-user USERNAME
```

由部署人把教程中的 Cloud Mail 发件账号、密码和 accountId 放进仅 root 可读的 `/etc/simhub-account/cloud-mail.json`。键名为 `email`、`password` 和 `accountId`。systemd 的 LoadCredential 会把文件交给定时任务。客户端只需要自己的 SIMHub 登录信息和提醒接收邮箱。

收件邮箱与登录账号独立，每个账号单独设置。服务器接入、HTTPS 路由和凭据安装完成后可启用定时器。它按五分钟轮询，在到期前七天进入发送窗口。测试账号请设置经本人确认的收件地址。测试可把记录到期时间设为未来六天，然后运行一次提醒服务并检查邮箱。

## API 约定

- POST `/accounts/v1/login` 使用 `username`、`password` 换取 30 天会话。后续请求使用 `Authorization: Bearer <会话>`。POST `/accounts/v1/logout` 立即撤销本次会话。
- GET/PUT `/accounts/v1/settings` 读写当前账号的 `recipient`。传 null 可停用邮件提醒。
- GET `/accounts/v1/snapshot` 返回 `revision` 和 `data`。PUT 接收这两个字段及完整的 `cards` 数组。卡片投影只包含 `id`、`title` 和 UTC Unix 秒级 `expires`，供提醒任务使用。客户端应将有期限的卡投影到该数组。
- PUT 的版本落后时返回 409。客户端应读取新快照并进行合并，保留未能自动合并的本地记录。客户端必须按账户分别持久化上一次共同版本和未上传的更改。
- 只要账号、卡片 ID、到期时间未变，已成功发送的记录会去重。Cloud Mail 未确认成功时，服务会延后重试。API 不提供幂等键，请求被发信端接收但响应丢失时仍可能重复送达。
- `data` 可承载客户端版本化的业务数据。服务以账号隔离访问，数据库含快照内容，生产环境需限制访问并加密保存备份。账号切换通过冷启动装载独立数据目录。服务器登录密码、会话令牌、发信凭据和设备配对凭据都应走各自的凭据存储。

## 测试

运行 `python -m unittest -v`。本地测试使用独立临时数据库和模拟发件端，覆盖账号隔离、版本冲突、注销、七天窗口、重试、同卡去重和两个账号各自的收件邮箱。

## 在既有 8443 翻译服务接入账号 API

随包的 accounts_gateway.py 与 serve_tls_accounts.py 对应之前的独立翻译中转部署。把两份文件安装到 /opt/simhub-translation-8443/（与 relay.py 同目录），文件模式 0644。通过该服务的 systemd drop-in 设置以下两行 ExecStart，然后 daemon-reload，重启 simhub-translation-8443。仅在获准部署时进行。443 和 8080 继续由现有服务使用。

    [Service]
    ExecStart=
    ExecStart=/opt/simhub-translation-8443/.venv/bin/python /opt/simhub-translation-8443/serve_tls_accounts.py

部署账号端的命令示例（首次部署时执行，先检查目录和 8872 端口）：

    sudo install -d -m755 /opt/simhub-account
    sudo install -m644 service.py requirements.txt /opt/simhub-account/
    sudo python3 -m venv /opt/simhub-account/.venv
    sudo /opt/simhub-account/.venv/bin/pip install -r /opt/simhub-account/requirements.txt
    id simhub-account || sudo useradd --system --home-dir /var/lib/simhub-account --shell /usr/sbin/nologin simhub-account
    sudo install -d -m700 /etc/simhub-account
    sudo install -m600 "${PRIVATE_MAIL_JSON:?设置私密凭据文件路径}" /etc/simhub-account/cloud-mail.json
    sudo install -m644 simhub-account.service simhub-account-reminders.service simhub-account-reminders.timer /etc/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl enable --now simhub-account.service simhub-account-reminders.timer

从外部设备访问 GET https://161.153.97.110:8443/accounts/v1/settings，应通过系统证书校验并返回 HTTP 401。使用本页的 add-user 命令创建用户后，在手机登录、保存收件邮箱、从首页添加一张 6 天后到期的测试卡，执行立即同步并检查收件。凭据从用户的私密教程准备，仅交付给服务器。

若这台服务器的 8443 空闲、只部署账号服务，可以独立启动 HTTPS 入口。把 accounts_gateway.py 和 serve_tls_accounts.py 放在 /opt/simhub-account/，将完整证书链及私钥按上面路径放入 /etc/simhub-account/fullchain.pem 和 privkey.pem，私钥权限 0600。证书需要覆盖 Oracle IP 并受手机信任。安装 simhub-account-https.service 并通过 systemctl enable --now 启动。已有翻译中转占用 8443 时使用上面的网关接入方式。
