# 如何在继续使用原主力系统的情况下，在物理机使用Qubes；一种解决方案 —— P2V(Physical to Virtual)
*示例：将主力物理 Debian / Windows 系统迁移到 Qubes OS HVM* 

*需要材料（稍后补充）：两块足够大的硬盘，一个足够大的U盘作为 Qubes 启动盘，原始电脑。

### 思路：
1. 在外接硬盘上安装 Qubes 作为测试环境，先测试硬件兼容。
2. 把 Debian 和 Windows 分别制作为P2V 备份/磁盘镜像，保存到另一块移动硬盘
3. 将 P2V 备份作为 Standalone HVM 导入SSD1外接硬盘的 Qubes OS中，目的是测试、调试与备份 （这时候主盘仍然没有任何改动，但我们获得了三重备份：主盘、SSD1 Qubes 上经过测试的 P2V HVM、SSD2上的P2V原始备份/磁盘镜像）
4. 使用Qubes 内置的 Qubes Backup 对已验证的HVM再次备份，保存到SSD2
5. 格式化主盘，安装 Qubes OS （此时我们仍然有双重备份：SSD1上的经过确认调试包含原系统 HVM 的完整 QubesOS, SSD2 上的原始备份/磁盘镜像+ Qubes Backup）
6.  启动安装在内置硬盘上的新 Qubes OS, 从 SSD2 Restore Qubes Backup恢复系统
7.  你获得了 QubesOS, 同时原系统作为 HVM 仍然可访问可使用。（此时我们有三重备份：SSD1上的经过确认调试包含原系统 HVM 的完整 Qubes OS、SSD上的系统原始备份+QubesVM、本机硬盘上的包含原系统P2V HVM的完整Qubes OS）



主包现在有啥

```
内置 2TB 硬盘
├── Windows
├── Debian
└── 其他/恢复分区

1TB SSD1
└── 空

1TB SSD2
└── 一些照片备份

USB stick
└── 空
```


### 1. 在SSD1安装Qubes,在物理机启动
测试硬件


### 2. 备份两个系统到SSD2
将会获得两个独立的虚拟磁盘：windows.raw，debian.raw

### 3. 把 Debian 和 Windows 分别 P2V 到 Qubes 里测试
大概这样

```
SSD1_Qubes
├── windows-old     StandaloneVM / HVM
└── debian-old      StandaloneVM / HVM
```



#### Windows
(https://doc.qubes-os.org/en/latest/user/templates/windows/qubes-windows.html)


#### Debian
Debian 比 Windows 好迁移多了！
施工中......

#### 在外置 Qubes 上启动两个 VM


### 4. Qubes Backup



### 5. 最thrilling的地方来了！格式化主盘，正式安装 Qubes



### 6. 把两个旧系统迁进最终内置硬盘上的 Qubes OS
(https://doc.qubes-os.org/en/r4.3/user/templates/windows/index.html)

### 7. 成功!
