# Linux-Armbian-CPU
Linux  玩客云 Armbian使用脚本根据温度调整CPU主频防止矿渣死机

创建所需要的文件：
touch /usr/sbin/cpu-control.sh
touch /home/cpu_freq.log
touch /etc/systemd/system/cpu-governor.service
------------------------------------------------------

vi /usr/sbin/cpu-control.sh
--------------------------------------------------------------------------------------------------
#!/bin/bash
# 定义日志文件路径
LOG_FILE="/home/cpu_freq.log"

# 无限循环，持续检查 CPU 温度和频率
while true
do
    # 读取 CPU 温度，单位为毫度 (m°C)，例如 42000 表示 42°C
    TEMP=$(cat /sys/class/thermal/thermal_zone0/temp)
    # 将温度转换为°C
    TEMP_C=$(($TEMP / 1000))
    # 读取当前 CPU 的频率调节器（Governor），例如 ondemand
    GOVERNOR=$(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor)
    # 获取当前时间
    DATE=$(date "+%Y-%m-%d %H:%M:%S")
    # 记录温度和 governor 状态
    echo "$DATE - CPU 温度: $TEMP_C°C, Governor: $GOVERNOR" >> $LOG_FILE
    # 如果温度低于等于 42°C 并且当前 governor 是 ondemand，设置 CPU 最低频率为 400MHz，最高频率为 1200MHz
    if [ "$TEMP" -le 42000 ] && [ "$GOVERNOR" = "ondemand" ]; then
        if ! sudo cpufreq-set -d 400m -u 1200m; then
            echo "$DATE - 设置 CPU 频率失败: 400m-1200m" >> $LOG_FILE
        else
            echo "$DATE - 设置 CPU 频率: 400m-1200m" >> $LOG_FILE
        fi
    fi
    # 如果温度高于等于 45°C 并且当前 governor 是 ondemand，设置 CPU 最低频率为 300MHz，最高频率为 600MHz
    if [ "$TEMP" -ge 45000 ] && [ "$GOVERNOR" = "ondemand" ]; then
        if ! sudo cpufreq-set -d 300m -u 600m; then
            echo "$DATE - 设置 CPU 频率失败: 300m-600m" >> $LOG_FILE
        else
            echo "$DATE - 设置 CPU 频率: 300m-600m" >> $LOG_FILE
        fi
    fi
    # 如果温度高于等于 49°C 并且当前 governor 是 ondemand，设置 CPU 最低频率为 200MHz，最高频率为 280MHz
    if [ "$TEMP" -ge 49000 ] && [ "$GOVERNOR" = "ondemand" ]; then
        if ! sudo cpufreq-set -d 200m -u 280m; then
            echo "$DATE - 设置 CPU 频率失败: 200m-280m" >> $LOG_FILE
        else
            echo "$DATE - 设置 CPU 频率: 200m-280m" >> $LOG_FILE
        fi
    fi
    # 每隔 10 秒检查一次温度和频率
    sleep 10
done
--------------------------------------------------------------------------------------------------
写成服务，开机运行
vi /etc/systemd/system/cpu-governor.service
-------------------------------------------------
[Unit]
Description=CPU Governor Control by Temperature
[Service]
Type=simple
ExecStart=/bin/sh /usr/sbin/cpu-control.sh
[Install]
WantedBy=multi-user.target
--------------------------------------------------
重新加载 systemd 配置：
sudo systemctl daemon-reload

再次尝试启动服务
sudo systemctl start cpu-governor.service

验证服务是否已启用
sudo systemctl is-enabled cpu-governor.service

确保 cpu-governor.service 文件有足够的权限，通常它应该是可读的。可以通过以下命令设置文件的权限
sudo chmod 644 /etc/systemd/system/cpu-governor.service

开机启动
sudo systemctl enable cpu-governor.service

-----------------二选一--------------------------
重启服务
sudo systemctl restart cpu-governor.service

重新启动测试
sudo reboot
-----------------二选一--------------------------

查看服务状态
sudo systemctl status cpu-governor.service


编写清空文件内容的脚本

vi /home/cpu_freq.log

编辑 cron 定时任务
crontab -e

----------------------------------------------
*/15 * * * * > /home/cpu_freq.log
----------------------------------------------
*/15 * * * *：每 15 分钟执行一次。
> /home/cpu_freq.log：用空内容覆盖 /home/cpu_freq.log 文件，相当于清空文件内容。
