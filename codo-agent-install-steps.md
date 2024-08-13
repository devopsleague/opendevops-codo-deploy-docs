CentOS 7 及以上版本手动部署文档：

### 步骤 1：下载、解压并设置执行权限

1. 创建目标目录，下载并解压 codo-agent，然后重命名文件并添加执行权限：
   ```bash
   mkdir -p /opt/apps/codo-agent/ && wget -O - https://ops-public.huanle.com/agent/agent.gz | tar -xz -C /opt/apps/codo-agent/ && mv /opt/apps/codo-agent/codo-agent-linux-x86 /opt/apps/codo-agent/codo-agent && chmod +x /opt/apps/codo-agent/codo-agent
   ```

### 步骤 2：创建并配置 systemd 服务文件
>【注意点】根据实际情况修改 `ExecStart` 行中的 `{server addr}` `{ip}:8888` 和 `{biz_id}`：
   ```bash
   cat <<EOF > /usr/lib/systemd/system/codo-agent.service
   [Unit]
   Description=codo-agent
   After=network-online.target
   Wants=network-online.target
   
   [Service]
   User=root
   Group=root
   LimitCORE=infinity
   LimitNOFILE=655350
   LimitNPROC=655350
   KillMode=process
   
   Type=simple
   ExecStart=/opt/apps/codo-agent/codo-agent --url ws://{server addr}:8081/api/v1/codo/agent?clientId={ip}:8888 -s --log-dir /opt/apps/codo-agent --client-type normal --row-limit 4000 --biz-id={biz_id}
   WorkingDirectory=/opt/apps/codo-agent
   
   Restart=always
   RestartSec=10s
   StartLimitInterval=0
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

```bash
# 生产数据参考
# /opt/apps/codo-agent/codo-agent --url wss://agent-server.codo.com/api/v1/codo/agent?clientId=10.60.16.5:8888 -s --log-dir /opt/apps/codo-agent --row-limit 4000 --biz-id=504
```

### 步骤 3：加载并启动服务

1. 重载 systemd 守护进程：
   ```bash
   systemctl daemon-reload
   ```

2. 启用并启动 `codo-agent` 服务：
   ```bash
   systemctl enable codo-agent.service
   systemctl restart codo-agent.service
   systemctl status codo-agent.service
   ```

### 步骤 4：查看日志

使用以下命令实时查看 `codo-agent` 的日志：

```bash
tail -f /opt/apps/codo-agent/logs/codo-agent.log
```