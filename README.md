# sing-box

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
uname -an
Linux (none) 5.10.0 #1 SMP Mon Dec 19 16:03:09 CST 2022 armv7l armv7l armv7l GNU/Linux
exe infomation:
ELF 32-bit LSB pie executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-arm.so.1, stripped

  git clone https://github.com/SagerNet/sing-box.git
  cd sing-box
  git checkout v1.11.9    ##1.12.0+ using new format of config.
  export CC=arm-euler-linux-musleabi-gcc
  export CXX=arm-euler-linux-musleabi-g++
  CGO_ENABLED=0 GOOS=linux GOARCH=arm GOARM=5 go build  -ldflags "-s -w"   -tags "with_quic,with_utls,with_wireguard,with_gvisor,with_clash_api,with_v2ray_api,with_ech,with_dns_cache"   -o singbox-armv7  ./cmd/sing-box
  or using GOARM=7


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
