

#### 修改home目录为英文

```shell
cat ~/.config/user-dirs.dirs

# This file is written by xdg-user-dirs-update
# If you want to change or add directories, just edit the line you're
# interested in. All local changes will be retained on the next run.
# Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
# homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
# absolute path. No other format is supported.
# 
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Download"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"

```

#### [Archlinux 字体配置](http://blog.chinaunix.net/uid-25906175-id-3072940.html)

```shell
 pacman -S ttf-dejavu ttf-liberation wqy-zenhei ttf-arphic-ukai ttf-arphic-uming
```

网上很多都是说修改 /etc/fonts/local.conf  或是 /etc/fonts/conf.d/49-sansserif.conf 等系统文件，但是这样升级系统后，配置文件便会丢失。

其实只需要在 home 目录下创建 ～/.config/fontconfig/fonts.conf 文件，其编码为 UTF-8，其内容为：

```xml
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
    <match target="font">
        <edit name="antialias" mode="assign"><bool>true</bool></edit>
        <edit name="rgba" mode="append"><const>rgb</const></edit>
        <edit name="lcdfilter" mode="append"><const>lcddefault</const></edit>
        <edit name="autohint" mode="append"><bool>false</bool></edit>
        <edit name="hinting" mode="assign"><bool>true</bool></edit>
        <edit name="hintstyle" mode="assign"><const>hintslight</const></edit>
    </match>

    <alias binding="strong">
        <family>serif</family>
        <prefer>
            <family>DejaVu Serif</family>
            <family>WenQuanYi Zen Hei</family>
        </prefer>
    </alias>

    <alias binding="strong">
        <family>sans-serif</family>
        <prefer>
            <family>DejaVu Sans</family>
            <family>WenQuanYi Zen Hei</family>
        </prefer>
    </alias>

    <alias binding="strong">
        <family>monospace</family>
        <prefer>
            <family>DejaVu Sans Mono</family>
            <family>WenQuanYi Zen Hei Mono</family>
        </prefer>
    </alias>

    <!-- To substitute some famous Chinese fonts -->
    <match target="pattern">
        <test name="family">
            <string>宋体</string>
        </test>
        <edit name="family" mode="assign">
            <string>SimSun</string>
        </edit>
    </match>

    <match target="pattern">
        <test name="family">
            <string>新宋体</string>
        </test>
        <edit name="family" mode="assign">
            <string>SimSun</string>
        </edit>
    </match>

    <match target="pattern">
        <test name="family">
            <string>楷体</string>
        </test>
        <edit name="family" mode="assign">
            <string>KaiTi</string>
        </edit>
    </match>

    <match target="pattern">
        <test name="family">
            <string>楷体_GB2312</string>
        </test>
        <edit name="family" mode="assign">
            <string>KaiTi</string>
        </edit>
    </match>

    <match target="pattern">
        <test name="family">
            <string>黑体</string>
        </test>
        <edit name="family" mode="assign">
            <string>SimHei</string>
        </edit>
    </match>

    <match target="pattern">
        <test name="family">
            <string>微软雅黑</string>
        </test>
        <edit name="family" mode="assign">
            <string>SimHei</string>
        </edit>
    </match>

    <alias binding="strong">
        <family>SimSun</family>
        <accept>
            <family>AR PL UMing CN</family>
        </accept>
    </alias>

    <alias binding="strong">
        <family>KaiTi</family>
        <accept>
            <family>AR PL UKai CN</family>
        </accept>
    </alias>

    <alias binding="strong">
        <family>SimHei</family>
        <accept>
            <family>WenQuanYi Zen Hei</family>
        </accept>
    </alias>

    <!-- To substitute some English fonts -->
    <alias binding="strong">
        <family>BookAntiqua</family>
        <accept>
            <family>URW Palladio L</family>
        </accept>
    </alias>

    <alias binding="strong">
        <family>Georgia</family>
        <accept>
            <family>Liberation Serif</family>
        </accept>
    </alias>

    <alias binding="strong">
        <family>Verdana</family>
        <accept>
            <family>Liberation Sans</family>
        </accept>
    </alias>

    <alias binding="strong">
        <family>Calibri</family>
        <accept>
            <family>Liberation Sans</family>
        </accept>
    </alias>

</fontconfig>
```