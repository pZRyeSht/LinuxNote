# Prometheus + Grafana + Alertmanager

## node_exporter 安装

1.在所有需要prometheus监控的计算机上下载并安装Node Exporter：`https://github.com/prometheus/node_exporter/releases`
```shell
wget https://github.com/prometheus/node_exporter/releases/download/v0.15.2/node_exporter-0.15.2.linux-amd64.tar.gz
```

2.解压缩下载的文档：（选定对应版本下载）
```shell
tar -xf node_exporter-0.15.2.linux-amd64.tar.gz
```

3.将node_exporter二进制文件移至/usr/local/bin：
```shell
sudo mv node_exporter-0.15.2.linux-amd64/node_exporter /usr/local/bin
```

4.移除相关无用文件（非必要，仅保留服务运行所需二进制文件）：
```shell
rm -r node_exporter-0.15.2.linux-amd64*
```

5.为node_exporter创建用户和服务文件：
出于安全原因，始终建议在单独的帐户中运行任何服务/守护程序。
node_exporter创建一个用户帐户。
-r标志，以表明它是一个系统帐户，默认的shell设置/bin/false使用-s，以防止登录。
```shell
sudo useradd -rs /bin/false node_exporter
```

6.创建systemd service：
```shell
sudo vim /etc/systemd/system/node_exporter.service
 
[Unit]                                      # 这个项目与此 unit 的解释、执行服务相依性有关
Description=Node Exporter                   # 服务描述说明
After=network.target                        # 服务启动的顺序
 
[Service]                                   # 服务指令参数
User=node_exporter
Group=node_exporter
Type=simple                                 # daemon 启动的方式
ExecStart=/usr/local/bin/node_exporter      # 执行此 daemon 的指令或脚本程序
 
[Install]
WantedBy=multi-user.target                  # 附挂在哪一个 target unit 下面
```

7.重载进程服务并启动node_expoeter：
```shell
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

## Prometheus 安装

1.在Prometheus服务器上下载并安装Prometheus：`https://github.com/prometheus/prometheus/releases`
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.1.0/prometheus-2.1.0.linux-amd64.tar.gz
```

2.提取Prometheus存档：
```shell
tar -xf prometheus-2.1.0.linux-amd64.tar.gz
```

3.将二进制文件移到/usr/local/bin：
```shell
sudo mv prometheus-2.1.0.linux-amd64/prometheus prometheus-2.1.0.linux-amd64/promtool /usr/local/bin
```

4.创建配置文件和其他普prometheus数据目录：
```shell
sudo mkdir /etc/prometheus /var/lib/prometheus
```

5.将移动配置文件到对应的创建的目录中：
```shell
sudo mv prometheus-2.1.0.linux-amd64/consoles prometheus-2.1.0.linux-amd64/console_libraries /etc/prometheus
```

6.移除相关无用文件（非必要，仅保留二进制脚本）：
```shell
rm -r prometheus-2.1.0.linux-amd64*
```

7.配置Prometheus监控相关服务器：
```shell
sudo vim /etc/hosts
 
x.x.x.x prometheus-target-1   # x.x.x.x 为监控服务器ip
x.x.x.x prometheus-target-2
...
x.x.x.x prometheus-target-n
```

8.配置Prometheus yml文件
```yaml
sudo vim /etc/prometheus/prometheus.yml
 
global:
  scrape_interval: 10s
 
scrape_configs:
  - job_name: 'prometheus_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100','prometheus-target-1:9100','prometheus-target-2:9100']  # localhost:9100 为本地node_exporter, prometheus-target-1:9100 为配置相关监控服务器
 
# Prometheus使用端口9090，node_exporter使用端口9100提供其指标
```

9.更改Prometheus将使用的文件的所有权：
```shell
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
```

10.创建service进程服务：
```shell
sudo vim /etc/systemd/system/prometheus.service
 
[Unit]
Description=Prometheus
After=network.target
 
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
 
[Install]
WantedBy=multi-user.target
```

11.重新加载systemd：
```shell
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

12.访问Prometheus：
`http://<your_server_IP>:9090/`

## 配置Grafana

1.获取.deb安装包：https://grafana.com/grafana/download
```shell
sudo apt-get install -y adduser libfontconfig
wget <.deb package url>  # 访问Grafana Download获取链接
sudo dpkg -i grafana<edition>_<version>_amd64.deb
```

2.重载守护进程并启动grafana：
```shell
sudo systemctl daemon-reload && sudo systemctl enable grafana-server && sudo systemctl start grafana-server.service
```

3.访问：
```
url：http://your.server.ip:3000
user：admin       # 默认用户
password：admin   # 默认密码
```

4.为Grafana配置Prometheus data source：


## 配置Prometheus Alertmanager

1.下载AlertManager：`https://github.com/prometheus/alertmanager/releases`
```shell
cd /opt/wget https://github.com/prometheus/alertmanager/releases/download/v0.11.0/alertmanager-0.11.0.linux-amd64.tar.gz
tar -xvzf alertmanager-0.11.0.linux-amd64.tar.gz
mv alertmanager-0.11.0.linux-amd64/alertmanager /usr/local/bin/
```

2.创建.yml配置文件：
```shell
mkdir /etc/alertmanager/
sudo vim /etc/alertmanager/alertmanager.yml
```

```yaml
# 全局配置项
global:
  resolve_timeout: 5m #处理超时时间，默认为5min
  smtp_smarthost: 'smtp.exmail.qq.com:465' # 邮箱smtp服务器代理
  smtp_from: 'xxxxxxxx@ppiaas.cn' # 发送邮箱名称
  smtp_auth_username: 'xxxxxxx@ppiaas.cn' # 邮箱名称
  smtp_auth_password: '********' # 邮箱密码或授权码
# 定义页面模版信息
templates:
  - '/etc/alertmanager/template/*.tmpl'
# 定义路由树信息
route:
  group_by: ['alertname'] # 报警分组依据
  group_wait: 3s # 最初即第一次等待多久时间发送一组警报的通知
  group_interval: 5s # 在发送新警报前的等待时间
  repeat_interval: 1h # 发送重复警报的周期 对于email配置中，此项不可以设置过低，否则将会由于邮件发送太多频繁，被smtp服务器拒绝
  receiver: 'email' # 发送警报的接收者的名称，以下receivers name的名称
# 定义警报接收者信息
receivers:
  - name: 'email' # 警报
    email_configs: # 邮箱配置
    - to: '742642743@qq.com' # 接收警报的email配置
      send_resolved: true
```

3.创建AlertManager systemd服务：
```shell
sudo vim /etc/systemd/system/alertmanager.service
 
[Unit]
Description=AlertManager Server Service
Wants=network-online.target
After=network-online.target
 
[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file /etc/alertmanager/alertmanager.yml -web.external-url=http://x.x.x.x:9093
# 使用-web.external-url=http://x.x.x.x:9093允许将通知URL重定向到Prometheus AlertManager Web界面。 x.x.x.x 为prometheus 服务器ip
 
[Install]
WantedBy=multi-user.target
```

4. 重载并开启进程：
```shell
systemctl daemon-reload
systemctl start alertmanager
systemctl enable alertmanager
systemctl status alertmanager
```

5.在Grafana中安装Prometheus Alertmanager插件:
```
grafana-cli plugins install camptocamp-prometheus-alertmanager-datasource

service grafana-server restart
```

6.配置AlertManager Prometheus数据源，安装仪表板：`https://grafana.com/grafana/dashboards/8010`

7.AlertManager集成到Prometheus，建立一个警报规则文件，该文件定义触发警报所需的所有规则：
```yaml
sudo vim /etc/prometheus/prometheus.yml
 
rule_files:
  - '/etc/prometheus/alert.rules.yml'
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 'localhost:9093'
```

8.配置alert rules：
```yaml
sudo vim /etc/prometheus/alert.rules.yml
 
groups:
- name: alert.rules
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: "critical"
    annotations:
      summary: "Endpoint {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

9.检查警报文件在语法上是否正确：
`promtool check rules alert.rules.yml`

10.重启所有相关服务：
```shell
sudo systemctl stop node_exporter &&
sudo systemctl start node_exporter &&
sudo systemctl stop prometheus &&
sudo systemctl start prometheus &&
sudo systemctl stop alertmanager &&
sudo systemctl start alertmanager
```

## WechatRobot Webhook

1.下载二进制webhook post request脚本：`https://github.com/EscAlice/Alertmanager-Webhook-Wechat`

2.上传至服务器对应路径下：
```shell
mv alertmanager-webhook-wechat /usr/local/bin
chmod +x alertmanager-webhook-wechat
```

3.创建systemd service 服务：
```shell
sudo vim /etc/systemd/system/alertmanager-webhook-wechat.service
 
[Unit]
Description=Alertmanager Webhook Wervice
After=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager-webhook-wechat --RobotKey=xxxxxx-xxxxx-xxxxx-xxxxxx-xxxxxxx # RobotKey为wechat robot key
 
 
[Install]
WantedBy=multi-user.target
```

4.重载并开启alertmanager-webhook-wechat服务：
```shell
systemctl daemon-reload
systemctl start alertmanager-webhook-wechat
systemctl enable alertmanager-webhook-wechat
systemctl status alertmanager-webhook-wechat
```

5.修改alertmanager.yml：
```yaml
sudo vim /etc/alertmanager/alertmanager.yml
 
global:
  resolve_timeout: 5m #处理超时时间，默认为5min
route:
  group_by: ['alertname']
  group_wait: 3s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
    - url: 'http://localhost:6666/webhook?key=xxxxxx-xxxxx-xxxxx-xxxxxx-xxxxxxx'
```

6.重启所有相关服务：
```shell
sudo systemctl stop node_exporter &&
sudo systemctl start node_exporter &&
sudo systemctl stop prometheus &&
sudo systemctl start prometheus &&
sudo systemctl stop alertmanager &&
sudo systemctl start alertmanager &&
sudo systemctl stop alertmanager-webhook-wechat &&
sudo systemctl start alertmanager-webhook-wechat
```