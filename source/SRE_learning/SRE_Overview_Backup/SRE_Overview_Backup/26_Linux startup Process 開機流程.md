# Linux startup Process 開機流程

[TOC]

---

## 開機流程

先按開機鍵，
電腦第一個啟動的是BIOS或UEFI，兩者最大的差別是UEFI滑鼠可以動，然後會執行自我開機檢測，BIOS會來檢測硬體周邊的裝置，包含CPU,記憶體,顯卡,網卡和硬碟...等，假如記憶體插錯，開機就有可能會聽到連續逼的聲音，如果裝置都正常就可以聽到逼一聲，代表開機正常．

接下來會去讀取可開機裝置的MBR，可開機裝置指的是光碟,硬碟和隨身碟...等那些，只要他能開機，在BIOS有設定，他就會嘗試去找可開機裝置的第一個磁區，他的大小512byte，其中446byte是Boot loader(儲存開機管理程式)，像alpine linux用的是syslinux，ubuntu linux用的是GRUB2，tinycore用的是isolinux

而Boot Loader最重要的是把kernel以及initramfs載入到記憶體裡面，把initramfs的記憶體位址告訴kernel，kernel才能找到，之後把控制權給linux kernel

linux kernel拿到控制權後，一樣會做一件事情，就是檢測電腦週邊的硬體是不是都正常，之後會將initramfs的system解壓縮，並且掛載起來，目的是為了要取得真正的檔案系統的驅動程式，因為假如我們是用sda硬碟，就會要有sda硬碟的驅動程式，因為kernel一開始不知道這些驅動程式，所以要先掛載臨時的檔案系統，在取得真正的檔案系統後，再把真正的檔案系統掛載上來，

確定都ok後，他就會啟動第一支程式，叫做init的process，在ubuntu用的是systemd，而alpine用的是openrc

---

## Bash Startup

![](https://i.imgur.com/7KBCCfG.png)

使用者登入成功後，以bash shell為例

1. 系統會先判斷是交互式的貝殼程式還是非交互式，交互式的貝殼程式（interactive ）就是可以讓我們打命令去執行的貝殼程式，而非交互式(non-interactive )就是像我們寫的bash script ，當系統判定我們是交互式的shell 之後，

2. 就會繼續判斷是login shell 還是 non-login shell ，意思就是說要不要登入帳號密碼，這邊我們是一定會登入，

	>以上就會有4種shell，
	- interactive shell
	- non-interactive shell
	- login shell
	- non-login shel

3. 系統會去看bash這個指令後面有沒有加參數  `-- noprofile` ，如果有加這個參數，系統就不會讀取/etc/profile 這個檔案，也不會讀取~/.bash_profile, ~/.bash_login, ~/.profile 這些檔案

	> `--noprofile`
不會讀取系統範圍的啟動文件 /etc/profile 或任何個人初始化文件 `~/.bash_profile`、`~/.bash_login` 或 `~/.profile`。 
默認情況下，bash 在作為登錄 shell 調用時會讀取這些文件（請參閱下面的 INVOCATION）。

4. 正常情況下，一定會讀取/etc/profile

5. 來到使用者的家目錄，按照順序讀取`~/.bash_profile`，`~/.bash_login`和`~/.profile`，通常是最後一個`~/.profile` 這個檔案會有

6. 如果最後還有存在~/.bashrc 一樣會去讀取。

---


###### tags: `Linux`














