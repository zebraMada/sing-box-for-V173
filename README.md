# sing-box for 华为星光v173 光猫
利用光猫里的eaiapp 的启动脚本来启用singbox。

```
huawei home gateway v173
processor       : 0
model name      : ARMv7 Processor rev 1 (v7l)
BogoMIPS        : 1594.16
Features        : half thumb fastmult edsp thumbee tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x4
CPU part        : 0xc09
CPU revision    : 1
CPU physical    : 0

processor       : 1
model name      : ARMv7 Processor rev 1 (v7l)
BogoMIPS        : 1594.16
Features        : half thumb fastmult edsp thumbee tls
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x4
CPU part        : 0xc09
CPU revision    : 1
CPU physical    : 1

Hardware        : Hisilicon A9
Revision        : 0000
Serial          : 0000000000000000
```
```
uname -an
Linux (none) 5.10.0 #1 SMP Mon Dec 19 16:03:09 CST 2022 armv7l armv7l armv7l GNU/Linux
exe infomation:
ELF 32-bit LSB pie executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-arm.so.1, stripped
```

```
  git clone https://github.com/SagerNet/sing-box.git
  cd sing-box
  git checkout v1.11.9    ##1.12.0+ using new format of config.
  export CC=arm-euler-linux-musleabi-gcc
  export CXX=arm-euler-linux-musleabi-g++
  CGO_ENABLED=0 GOOS=linux GOARCH=arm GOARM=5 go build  -ldflags "-s -w"   -tags "with_quic,with_utls,with_wireguard,with_gvisor,with_clash_api,with_v2ray_api,with_ech,with_dns_cache"   -o singbox-armv7  ./cmd/sing-box
  or using GOARM=7
```
手头上的光猫是移动光猫，恢复出厂设置，用默认管理员密码登陆即可。
自己补全shell，这样就可以tftp 来上传文件。


编译完符合v173上使用singbox-armv7.
tftp32做tftpd server， 华为v173 需要补全shell，
在shell 下 吧singbox-armv7 get到 /mnt/jffs2/plug/app/cplugin/cplugin1/apps/eaiapp/MyPlugin/bin目录下。

```
cat /mnt/jffs2/plug/app/cplugin/cplugin1/apps/eaiapp/MyPlugin/appstart.sh
#!/bin/sh
# Copyright Huawei Technologies Co., Ltd. 2021-2021. All rights reserved.

# 依赖核心插件的libplugin_agent_api.so
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$(pwd)/Lib:$(pwd)/../../../MyPlugin/Lib

chmod +x $(pwd)/bin/*
chmod +x ./start.sh
sh ./start.sh

cd $(pwd)/bin
#./singbox-armv7 run -c config.json -D ./ >/dev/null 2>&1 &

echo "start eaiapp!"
trap 'killall -9 eaiapp; exit 1;' 15
while true ; do
  NUM=$(ps | grep eaiapp | grep -v grep |wc -l)
  if [[ "${NUM}" -lt "1" ]]; then
        ./eaiapp &
  fi

  PID=$(ps | grep eaiapp | awk -F: NR==${NUM} | awk '{print $1}' )
  USEMEM=$(cat /proc/${PID}/status | grep VmRSS |  awk '{print $2}' )
  if [[ "${USEMEM}" -ge "21432" ]]; then
        killall -9 eaiapp
  fi

  sleep 8
done
```
```
 cat /mnt/jffs2/plug/app/cplugin/cplugin1/apps/eaiapp/MyPlugin/start.sh
#!/bin/sh

# 自启脚本 - 放在 jffs2，确保开机可执行

# 等待 USB 挂载完成（关键！）
echo "Waiting for USB mount..."
umount /dev/sda1
sleep 2
mount /dev/sda1 /mnt/usb
sleep 2

iptables -I INPUT -p tcp --dport 2080 -j ACCEPT
iptables -I INPUT -p udp --dport 2080 -j ACCEPT

#while [ ! -d "/mnt/usb/tmp/bin" ]; do
#    sleep 3
#done
sleep 5  # 再等 5 秒确保文件系统稳定
cd $(pwd)/bin
# 检查程序是否存在
if [ ! -f "./singbox-armv7" ]; then
    echo "Error: singbox not found in ./bin/" >/tmp/sing.log
    exit 1
fi

# 启动 sing-box（后台运行）
echo "Starting sing-box..."
./singbox-armv7 run -c ./config.json -D ./ >/dev/null 2>&1 &

# 可选：记录 PID
#echo $! > /mnt/jffs2/app/plugins/work/myapp/MyPlugin/bin/singbox.pid
cd ../
echo "sing-box started."

```
```
WAP(Dopra Linux) # pwd
/mnt/jffs2/plug/app/cplugin/cplugin1/apps/eaiapp/MyPlugin/bin
WAP(Dopra Linux) # ls $pwd -alt
-rwxrwxrwx    1 osgi_pro osgi        413188 Oct  5 12:29 config.json
drwxrwxrwx    3 osgi_pro osgi          1040 Aug 12 01:11 dashboard
-rwxrwxrwx    1 osgi_pro osgi       5215701 Aug 10 05:13 geoip.db
-rwxrwxrwx    1 osgi_pro osgi       3742377 Aug 10 05:12 geosite.db
-rwxrwxrwx    1 osgi_pro osgi      31391896 Aug 10 05:10 singbox-armv7
WAP(Dopra Linux) # ls /mnt/jffs2/plug/app/cplugin/cplugin1/apps/eaiapp/MyPlugin -alt
drwxr-x---    4 osgi_pro osgi           992 Oct 26 03:10 config
drwxr-x---    3 osgi_pro osgi           888 Oct  5 11:36 bin
-r-xr-x---    1 osgi_pro osgi           753 Aug 12 00:50 appstart.sh
-rwxrwxrwx    1 osgi_pro osgi           806 Aug 12 00:50 start.sh
drwxr-x---    3 osgi_pro osgi           304 Aug 10 05:18 ..
drwxr-x---    7 osgi_pro osgi           696 Aug 10 04:48 .
drwxr-x---    2 osgi_pro osgi           320 Jan  1  1981 model
drwxr-x---    2 osgi_pro osgi           224 Jan  1  1981 kernel
drwxr-x---    2 osgi_pro osgi          1464 Jan  1  1981 Lib
-rw-r-----    1 osgi_pro osgi           158 Jan  1  1981 BuildInfo

```
```
singbox-armv7文件的属主属性按上图显示来更改即可。
config.json是配置文件，tunnel方式会报错，要去掉。
geoip.db geosite.db 需手动下载在板子上。
clash的面板要手动添加上去，127.0.0.1改成0.0.0.0
   "experimental": {
     "clash_api": {
            "external_controller": "0.0.0.0:9090",
            "external_ui": "dashboard"
        }
    }
```


The universal proxy platform.

[![Packaging status](https://repology.org/badge/vertical-allrepos/sing-box.svg)](https://repology.org/project/sing-box/versions)

## Documentation

https://sing-box.sagernet.org

## License

```
Copyright (C) 2022 by nekohasekai <contact-sagernet@sekai.icu>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <http://www.gnu.org/licenses/>.

In addition, no derivative work may use the name or imply association
with this application without prior consent.
```
