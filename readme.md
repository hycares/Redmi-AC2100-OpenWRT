# Xiaomi Redmi AC2100 刷机指北

如果没有特殊需求，请不要随便刷机，刷机有风险！！刷机有风险！！刷机有风险！！

刷机原因：

1. 校园网客户端
2. 限制广告
3. airport...

网络上有大量教程但感觉有些过时，有些还有大坑，为避免大家重蹈覆辙，我记录下整个刷机过程供大家参考:）

## 刷入Breed

Breed是一个第三方的启动引导工具，刷入此工具后我们能够通过它再刷入不同的固件比如openwrt，padavan等。

刷入步骤：

1. 下载固件
2. 降级当前路由器固件（可选）
3. 利用漏洞获取ssh
4. 刷入breed
5. reset重启进入breed恢复后台

### 1. 下载对应固件

[ac2100 breed](https://breed.hackpascal.net/breed-mt7621-xiaomi-r3g.bin)

### 2. 降级路由器当前固件

为了将breed固件写入路由器，我们需要上传固件并且烧写，这需要登录进路由器，低版本的固件存在一些漏洞（2.0.7）。

在路由器管理页面手动降级到对应[版本固件](./assets/2.0.7/miwifi_rm2100_firmware_d6234_2.0.7.bin)后，等待重启。

### 3. 开启ssh

降级完成后再次登录管理界面，登录的url大致是这样的：

```url
// ip可能会不一样，<stok>是一长串字符每次登录后都不一样
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/web/home#router
```

使用下述代码进行注入，将`<stok>`替换为浏览器地址栏显示的值后再输入进地址栏

```url
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B
```

执行完成后会出现`{"code":0}`

退出再刷新浏览器重新进入管理界面，此时`<stok>`会发生变化，再通过下述代码修改root密码：

```url
http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B
```

此时能够使用ssh登录进路由器，账号为：root 密码为：admin

### 4. 刷入breed

ssh登录路由器后，将第一步得到的固件上传到路由器后执行下述代码：

```shell
mtd -r write <YOURPATHTO>/breed-mt7621-xiaomi-r3g.bin Bootloader
```

### 5. 进入breed控制台

如果电脑重新获取到IP后说明刷写完成并且breed引导了官方固件，断电。按住reset键（背面圆孔，用取卡针插进去）后插电等待system的蓝灯闪烁后松开reset键。

用浏览器访问`192.168.1.1`进入breed恢复界面，点击固件更新即可上传自己的固件。

## 刷入OpenWRT

准备OpenWRT的[固件](https://downloads.openwrt.org/releases/21.02.2/targets/ramips/mt7621/openwrt-21.02.2-ramips-mt7621-xiaomi_redmi-router-ac2100-initramfs-kernel.bin)，也可以找其他的版本，通过breed上传该固件，点击更新并等待路由器重启，这里建议用有线连接电脑，测试时该固件烧入后不会开启WIFI。

![image-20220222185832288](assets/img/image-20220222185832288.png)

烧写完成后breed将会引导OpenWRT，如果要进入恢复控制台按照上述`进入breed控制台`操作即可。

## 刷入升级包

OpenWRT烧写完成后用浏览器访问`192.168.1.1`进入管理界面，默认无密码。

![image-20220222190608778](assets/img/image-20220222190608778.png)

点击导航栏的[`System`的`Backup/Flash Firmware`](http://192.168.1.1/cgi-bin/luci/admin/system/flash)点击最下方的Flash image刷入[升级包](https://downloads.openwrt.org/releases/21.02.2/targets/ramips/mt7621/openwrt-21.02.2-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-sysupgrade.bin)，等待重启。

在[Network中点击Wireless](http://192.168.1.1/cgi-bin/luci/admin/network/wireless)启用WIFI

![image-20220222191106039](assets/img/image-20220222191106039.png)

## 安装插件

### scutclient

### OpenClash

## 参考

[恩山大佬提供的开启SSH方法](https://www.right.com.cn/forum/thread-4032490-1-1.html)

