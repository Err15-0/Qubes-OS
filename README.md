# 在继续使用原主力系统的情况下，在物理机使用Qubes
## 一种解决方案 —— P2V(Physical to Virtual)完整流程
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

现在我们有
```
内置 2TB NVMe
├── Windows
├── Debian
└── 其他/恢复分区

1TB SSD1
└── 当前可启动 Qubes 4.3.2

1TB SSD2
└── 其他备份

USB
└── Qubes 安装盘
```

### 2. 备份两个系统到SSD2
将会获得两个独立的虚拟磁盘：windows.raw，debian.raw

### 3. 把 Debian 和 Windows 分别 P2V 到 Qubes 里测试
大概这样

```
SSD1_Qubes
├── windows-old     StandaloneVM / HVM
└── debian-old      StandaloneVM / HVM
```


#### (1) 了解基本信息（此时我在debian）
```
┌─(~)─────────────────────────────────────────────────────────────────────────────────────────────(Err15@debian:pts/1)─┐ 
└─(20:39:24)──>lsblk -o NAME,SIZE,FSTYPE,FSUSED,FSAVAIL,LABEL,PARTLABEL,MOUNTPOINTS                      ──(Tue,Oct27)─┘
NAME          SIZE FSTYPE FSUSED FSAVAIL LABEL            PARTLABEL                    MOUNTPOINTS 
nvme0n1       1.9T                                                                      
├─nvme0n1p1   260M vfat    90.9M  165.1M SYSTEM           EFI system partition         /boot/efi 
├─nvme0n1p2    16M                                        Microsoft reserved partition  
├─nvme0n1p3 771.5G ntfs                  Windows          Basic data partition          
├─nvme0n1p4 842.2G ntfs                  Data                                            
├─nvme0n1p5   806M ntfs                  Windows RE tools Basic data partition          
├─nvme0n1p6  46.6G ext4    32.7G   10.5G                  Root/                         / 
├─nvme0n1p7   7.5G swap                                   Swap                         [SWAP] 
└─nvme0n1p8 229.2G ext4   152.4G   60.7G                  Home                         /home

┌─(~)─────────────────────────────────────────────────────────────────────────────────────────────(Err15@debian:pts/1)─┐ 
└─(20:39:24)──> sudo fdisk -l /dev/nvme0n1                                                               ──(Tue,Oct27)─┘
[sudo] password for root:  
Disk /dev/nvme0n1: 1.86 TiB, 2048408248320 bytes, 4000797360 sectors
Disk model: <disk model>                    
Units: sectors of 1 * 512 = 512 bytes 
Sector size (logical/physical): 512 bytes / 512 bytes 
I/O size (minimum/optimal): 512 bytes / 512 bytes 
Disklabel type: gpt 
Disk identifier: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX 
Device         Start      End        Sectors    Size   Type
/dev/nvme0n1p1       2048     534527     532480   260M EFI System 
/dev/nvme0n1p2     534528     567295      32768    16M Microsoft reserved 
/dev/nvme0n1p3     567296 1618487295 1617920000 771.5G Microsoft basic data 
/dev/nvme0n1p4 1638967296 3405213695 1766246400 842.2G Microsoft basic data 
/dev/nvme0n1p5 3999133696 4000784383    1650688   806M Windows recovery environment 
/dev/nvme0n1p6 3405213696 3502870527   97656832  46.6G Linux swap 
/dev/nvme0n1p7 3983509504 3999133695   15624192   7.5G Linux swap 
/dev/nvme0n1p8 3502870528 3983509503  480638976 229.2G Linux filesystem
Partition table entries are not in disk order. 
┌─(~)──────────────────────────────────────────────────────────────────────────────────────────────(Err15@debian:pts/1)─┐ 
└─(20:39:24)──> sudo du -xsh /                                                                            ──(Tue,Oct27)─┘
33G     /
┌─(~)──────────────────────────────────────────────────────────────────────────────────────────────(Err15@debian:pts/1)─┐
└─(20:39:24)──> sudo du -sh /home                                                                         ──(Tue,Oct27)─┘
153G    /home
```
施工中......

#### (2) 开始备份

#### Debian
Debian 比 Windows 好迁移多了！
待会写

#### Windows
https://doc.qubes-os.org/en/latest/user/templates/windows/qubes-windows.html

打开管理员Powershell, 确保Bitlocker关闭
```

PS C:\windows\system32> manage-bde -status
Disk volumes that can be protected with BitLocker Drive Encryption:
Volume C: [Windows]
[OS Volume]

Size:                     771.48 GB
BitLocker Version:        None
Conversion Status:        Fully Decrypted
Percentage Encrypted:     0.0%
Encryption Method:        None
Protection Status:        Protection Off
Lock Status:              Unlocked
Identification Field:     None
Key Protectors:           None Found

Volume D: [Data]
[Data Volume]

Size:                     842.21 GB
BitLocker Version:        None
Conversion Status:        Fully Decrypted
Percentage Encrypted:     0.0%
Encryption Method:        None
Protection Status:        Protection Off
Lock Status:              Unlocked
Identification Field:     None
Automatic Unlock:         Disabled
Key Protectors:           None Found
```
确保都是Fully Decrypted.

(Optional)缩小空间：
- 管理员Powershell检查
``` 
  Get-ChildItem <你的盘符>:\ -Force |
ForEach-Object {
    $p = $_.FullName
    $size = if ($_.PSIsContainer) {
        (Get-ChildItem $p -Force -File -Recurse -ErrorAction SilentlyContinue |
         Measure-Object Length -Sum).Sum
    } else {
        $_.Length
    }
    [PSCustomObject]@{
        Name = $_.Name
        SizeGB = [math]::Round($size / 1GB, 2)
    }
} | Sort-Object SizeGB -Descending
```

- 关闭休眠/Fast Startup，并删除 hiberfil.sys
```
powercfg /h off
```
- 运行 Windows 自带清理
```
cleanmgr
```
- 缩C盘
Win + R，输入：
```
diskmgmt.msc
```
C: --> 右键 --> Shrink Volume


#### （3）在外置 Qubes 上启动两个 VM


### 4. Qubes Backup



### 5. 最thrilling的地方来了！格式化主盘，正式安装 Qubes



### 6. 把两个旧系统迁进最终内置硬盘上的 Qubes OS
(https://doc.qubes-os.org/en/r4.3/user/templates/windows/index.html)

### 7. 成功!
