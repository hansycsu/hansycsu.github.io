# YC's Linux Mint Note

![MintLogo](https://upload.wikimedia.org/wikipedia/commons/3/3f/Linux_Mint_logo_without_wordmark.svg)
---

* [安裝字型](#install-font)
* [設定面版 (Panel) 字型](#panel-font)
* [VIM背景黑色顯示不正確(變灰色)](#bg-issue)
* [LibreOffice 字型使用紀錄](#libreoffice-font-history)
* [輸入法相關](#ime)
* [套件管理指令](#package-management)
* [讓Tarball也變成套件](#tarball-to-deb)
* [開機修復 (grub 2)](#repair-boot)
* [查看磁碟資訊](#disk-management)
* [關閉Swap](#dis-swap)
* [家目錄改英文](#home-locale)
* [截圖軟體 Shutter](#shutter)

---

<br><h2 id="install-font">安裝字型</h2>

微軟英文字體： ttf-mscorefonts-installer  
文泉驛正黑：fonts-wqy-zenhei  
系統字型路徑： /usr/share/fonts/truetype  
使用者字型路徑： ~/.local/share/fonts/

安裝步驟：
>1. 複製字型檔到系統字型路徑
>2. fc-cache -f -v

    apt install ttf-mscorefonts-installer
    apt install fonts-wqy-zenhei
    cd /usr/share/fonts/truetype
    sudo mkdir MyFonts
    cd MyFonts
    cp /path/to/fonts/* ./
    fc-cache -f -v

<br><h2 id="panel-font">設定面版 (Panel) 字型</h2>

將以下的***Mint-X***換成你正在使用的主題(可在 */usr/share/themes/* 查看)

    cp -r /usr/share/themes/Mint-X ~/.themes
    vim ~/.themes/Mint-X/cinnamon/cinnamon.css

In cinnamon.css:

    stage {
        /* replace DEFAULT_FONT with your preferred one */
        font-family: DEFAULT_FONT;
    }

<br><h2 id="bg-issue">VIM背景黑色顯示不正確(變灰色)</h2>

這是因為 GNOME Terminal 的 Palette 定義問題。
設定方式：Edit→Profile Preferences，選擇Colors分頁，修改下面的調色盤第一項
參考：[black backgrounds appear grey on gnome-terminal][ref_bgissue]

[ref_bgissue]: https://superuser.com/questions/142486/black-backgrounds-appear-grey-on-gnome-terminal

<br><h2 id="libreoffice-font-history">LibreOffice 字型使用紀錄</h2>

LibreOffice 的設定檔在 *~/.config/libreoffice/* 底下，而字型使用記錄在 *~/.config/libreoffice/4/user/config/fontnameboxmruentries*

    vim ~/.config/libreoffice/4/user/config/fontnameboxmruentries

<br><h2 id="ime">輸入法相關</h2>

安裝輸入法：

    sudo apt-get install fcitx
    sudo apt-get install fcitx-chewing
    sudo apt-get install fcitx-table-boshiamy
    sudo apt-get purge ibus
    im-config (Choose fcitx as default IME)

懶人全裝法：

    apt install fcitx fcitx-tools fcitx-config* fcitx-frontend* fcitx-module* fcitx-ui-* presage

無法顯示選字窗時：(這個套件僅在kde桌面正常運作)

    apt purge fcitx-module-kimpanel

<br><h2 id="package-management">套件管理指令</h2>

增加 / 移除 軟體源 (PPA)：

    sudo add-apt-repository ppa:fcitx-team/nightly
    sudo add-apt-repository --remove ppa:whatever/ppa

使用 apt 安裝 / 移除 / 更新：

    sudo apt install PACKAGE   #安裝
    sudo apt remove PACKAGE    #移除
    sudo apt purge PACKAGE     #完整移除(含設定檔)
    sudo apt autoremove        #清理未使用的套件
    sudo apt update            #更新套件資訊(常用於設定PPA後)
    sudo apt upgrade           #更新套件
    sudo apt full-upgrade      #更新套件，必要時自動移除套件(常用於系統更新)
    sudo apt edit-sources      #取代sudo vi /etc/apt/sources.list
    sudo apt-mark hold PACKAGE #將PACKAGE設定為不更新
    sudo apt-mark auto PACKAGE #將PACKAGE設定為自動更新/移除

套件查詢：

    apt search ‘REGEX’ #以Regex搜尋套件
    apt show PACKAGE #顯示PACKAGE詳細資訊
    apt list --installed #列出已安裝套件
    apt list --upgradable #列出可更新套件
    dpkg -l PACKAGE #顯示套件狀態
    dpkg -L PACKAGE #顯示套件安裝路徑

<br><h2 id="tarball-to-deb">讓Tarball也變成套件</h2>

**Checkinstall**除了可以追蹤`make install`的安裝，他還可以追蹤任何Script/Command的檔案修改。這個程式會在 */var/log/packages* 產生.deb檔，可由`dpkg -r`移除

    sudo apt install checkinstall

Example:

    tar jxvf squid-2.6.STABLE12.tar.bz2
    cd squid-2.6.STABLE12
    ./configure && make
    sudo checkinstall #instead of make install

參考：[Linux源碼安裝工具Checkinstall][checkinstall_1] ;   [Checkinstall Readme][checkinstall_2]

[checkinstall_1]: https://www.ibm.com/developerworks/cn/linux/l-cn-checkinstall/
[checkinstall_2]: http://www.asic-linux.com.mx/~izto/checkinstall/docs/README

<br><h2 id="repair-boot">開機修復 (grub 2)</h2>

首先以Live CD/USB開機，接著開啟Terminal執行以下指令：

    sudo fdisk -l #確認bootloader分割區，以下以sda1為例
    sudo mount /dev/sda1 /mnt
    for i in /dev /dev/pts /proc /sys; do sudo mount -B $i /mnt$i; done
    sudo chroot /mnt
    grub-install /dev/sda
    update-grub
    sudo reboot

完成，以sda1重新開機即可。
參考：[GRUB2中文指南第二版(上）][grub_1] ;   [Reinstalling GRUB 2][grub_2]

[grub_1]: http://wiki.ubuntu-tw.org/index.php?title=GRUB2%E4%B8%AD%E6%96%87%E6%8C%87%E5%8D%97%E7%AC%AC%E4%BA%8C%E7%89%88%28%E4%B8%8A%EF%BC%89#.E9.87.8D.E6.96.B0.E5.AE.89.E8.A3.9D_GRUB_2
[grub_2]: https://help.ubuntu.com/community/Grub2/Installing#Reinstalling_GRUB_2

<br><h2 id="disk-management">硬碟管理</h2>

查看硬碟資訊：

    sudo fdisk -l
    lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,UUID

<br><h2 id="dis-swap">關閉Swap</h2>

以下是如何在系統安裝完成後關閉Swap

    sudo swapoff -a
    free #確認swap是否為0
    sudo parted /dev/sdc
    (parted) unit b #設定單位為Byte
    (parted) print #確認分區編號
    (parted) rm 6 #移除Swap分區
    (parted) resizepart 5 100% #重新分配剩下的分區給sdc5

<br><h2 id="home-locale">家目錄改英文</h2>

### 方法一

設語系為英文，再執行`xdg-user-dirs-gtk-update`

    export LANG=en_US
    xdg-user-dirs-gtk-update

此後再執行`xdg-user-dirs-gtk-update`會沒有作用，若要改回來，需修改設定檔：

    echo zh_TW > ~/.config/user-dirs.locale

### 方法二

中文到英文：

    export LC_ALL=en_US.UTF-8
    xdg-user-dirs-update --force

英文到中文：

    export LC_ALL=zh_TW.UTF-8
    xdg-user-dirs-update --force

如果不想再被提示更新目錄，可以執行

    echo en_US > ~/.config/user-dirs.locale #目前為英文
    echo zh_TW > ~/.config/user-dirs.locale #目前為中文

<br><h2 id="shutter">截圖軟體 Shutter</h2>

    sudo apt-add-repository ppa:shutter/ppa
    apt update
    apt install shutter 
    apt install libgoo-canvas-perl #編輯功能
    apt install gnome-web-photo #網頁截圖
    apt install libimage-exiftool-perl #旋轉時修改metadata

