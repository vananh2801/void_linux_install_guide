# Hướng dẫn cài đặt Gnome trên Void Linux (06/2026)

Hướng dẫn này dùng để  cài Gnome và một số gói khác sau khi cài Void Linux bằng base ISO.

Khi cài nên nên chọn srouce cài từ Internet để có các gói mới nhất dùng cho ổn định, đỡ mất thời gian update lại sau.

Chi tiết ở [trang hướng dẫn chính thức](https://docs.voidlinux.org/installation/live-images/guide.html) của Void Linux.

## Kết nối Wi-fi ở lần đầu khởi động

1. Mở khoá Wi-fi

    ```bash
    rfkill unblock all
    ```

2. Kiểm tra tên của interface:

    ```bash
    ip link
    ```

    Lúc này máy sẽ hiện danh sách các interface và trạng thái. Như máy tôi là tên interface là **wlp2s0**.

3. Cấu hình Wi-fi:

    ```bash
    wpa_passphrase "SSID" "PASSWORD" >> /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
    ```

    Trong đó:
    - **SSID** là tên Wi-fi (không có dấu tiếng Việt hoặc kí tự lạ, nếu có thì lên mạng chuyển string thành hex để nhập)
    - **PASSWORD** là mật khẩu 
    - **wlp2s0** nên thay thế cho thích hợp.

    Chẳng hạn:

    ```bash
    wpa_passphrase "Anh Đẹp Trai" "PASSWORD" >> /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
    ```

4. Khởi chạy dịch vụ wpa_supplicant và dhcpcd:

    ```bash
    sudo ln -s /etc/sv/wpa_supplicant /var/service/
    sudo sv start wpa_supplicant
    sudo ln -s /etc/sv/dhcpcd /var/service/
    sudo sv start dhcpcd
    ```

5. Cấu hình wpa_supplicant để chạy file cấu hình vừa tạo

    ```bash
    sudo wpa_supplicant -B -i wlp2s0 -c /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
    ```

6. Cấu hình lớp IP:

    ```bash
    sudo dhcpcd wlp2s0
    ```

7. Kiểm tra mạng:

    ```bash 
    ping google.com
    ```

    Nhấn Ctrl + C để dừng việc kiểm tra.

## Cập nhật hệ thống

- 
    ```bash
    sudo xbps-install -Suv
    ```

## Cài một số phần mềm khác

-
    ```bash
    sudo xbps-install nano git zip unzip curl wget xz vim
    ```

## Thêm non-free-repo và cài linux-firmware

-
    ```bash
    sudo xbps-install void-repo-nonfree linux-firmware
    ```

## Cài đặt Gnome

1. Cài đặt Xorg (Void Linux vẫn đang chạy Gnome 48 ở 06/2026):

    ```bash
    sudo xbps-install xorg
    ```

2. Cài đặt Gnome:

    ```bash
    sudo xbps-install gnome
    ```

3. Bật các dịch vụ cho gnome:

    ```bash
    sudo ln -s \etc\sv\gdm \var\servie
    sudo ln -s \etc\sv\dbus \var\servie
    sudo ln -s \etc\sv\power-profiles-daemon \var\servie
    ```

## Cài đặt NetworkManager dùng cho Gnome

1. Cài đặt:

    ```bash
    sudo xbps-install NetworkManager iwd
    ```

    Ở đây, tôi dùng iwd làm backend cho NetworkManager.

2. Cấu hình NetworkManager:

    ```bash
    sudo nano /etc/NetworkManager/NetworkManager.conf
    ```

    Chỉnh lại nội dung file như sau:

    ```bash                 
    [main]
    plugins=keyfile,iwd

    [device]
    wifi.backend=iwd
    ```

    Nhấn Ctrl + S để lưu và Ctrl + X để thoát.

3. Bật NetworkManager và tắt wpa_supplicant

    ```bash
    sudo sv stop wpa_supplicant
    sudo sv stop dhcpcd
    sudo rm /var/service/wpa_supplicant
    sudo rm /var/service/dhcpcd

    sudo ln -sv /etc/sv/NetworkManager /var/service
    sudo ln -sv /etc/sv/iwd /var/service
    sudo sv start NetworkManager
    sudo sv start iwd
    ```

## Cài gói PipeWire để có âm thanh và quay màn hình trên Gnome

1. Cài đặt các gói:

    ```bash
    sudo xbps-install -S pipewire wireplumber alsa-pipewire pipewire-pulse libspa-bluetooth pulseaudio-utils pulsemixer
    ```
2. Cấu hình WirePlumber:

    ```bash
    sudo mkdir -p /etc/pipewire/pipewire.conf.d
    sudo ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/
    ```

3. Cấu hình pipewire-pulse:

    ```bash
    sudo mkdir -p /etc/pipewire/pipewire.conf.d
    sudo ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/
    ```
4. Cấu hình ALSA:

    ```bash
    sudo mkdir -p /etc/alsa/conf.d
    sudo ln -sf /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d/
    sudo ln -sf /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d/
    ```

5. Bật tự khởi chạy cho PipeWire:

    ```bash
    mkdir -p ~/.config/autostart
    ln -sf /usr/share/applications/pipewire.desktop ~/.config/autostart/
    ```

6. Cấp quyền cho người dùng:

    ```bash
    sudo usermod -a -G audio <username>
    ```

## Cài đặt Bluetooth cho Gnome

1. Cài đặt bluez (thường thì đã có sẵn khi cài Gnome):

    ```bash
    sudo xbps-install bluez
    ```

2. Bật dịch vụ:

    ```bash
    sudo ln -s \etc\sv\bluetoothd \var\servie
    ```

3. Cấp quyền:

    ```bash
    sudo useradd -G bluetooth ${USER}
    ```

## Cài đặt snooze (tuỳ chọn)

Đây là trình quản lý dùng để chạy các hoạt động dựa trên lịch trình tự tạo. Chi tiết thì xem trên mạng :v

-
    ```bash
    sudo xbps-install snooze
    ```

## Cài một số phông chữ

1. Cài Noto fonts:

    ```bash
    sudo xbps-install noto-fonts-emoji noto-fonts-ttf noto-fonts-ttf-extra
    ```

2. Cài Ubuntu fonts:

    ```bash
    sudo xbps-install ttf-ubuntu-font-family
    ```

    Dùng Gnome Tweaks chỉnh sang loại này cho dễ nhìn.

3. Cài một số phông từ Microsoft (ví dụ Arial, Times New Roman,... ):

    Trước hết, ta cài xbps-src:

    ```bash
    git clone https://github.com/void-linux/void-packages
    cd void-packages
    ./xbps-src binary-bootstrap
    echo "XBPS_ALLOW_RESTRICTED=yes" >> etc/conf
    ```

    Sau đó, ta dùng xbps-src để cài phông:

    ```bash
    ./xbps-src pkg -f msttcorefonts
    xi msttcorefonts
    ```

## Cài đặt một số phần mềm khác

1. Cài đặt LibreOffce (tuỳ chọn):

    ```bash
    sudo xbps-install libreoffice-writer libreoffice-calc libreoffice-impress libreoffice-draw libreoffice-math libreoffice-base libreoffice-gnome libreoffice-i18n-en-US
    ```

2. Cài đặt Firefox:

    ```bash
    sudo xbps-install firefox firefox-i18n-en-US
    ```

3. Cài đặt Flatpak và cửa hàng ứng dụng:

- Cài đặt Flatpak

    ```bash
    sudo xbps-install flatpak
    ```

- Thêm rep của Flathub vào:
    ```bash
    flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    ```

- Cài đặt Cửa hàng (tuỳ chọn, tôi thấy chạy ngầm hơi phí RAM)

    ```bash
    sudo xbps-install gnome-software
    ```

- Cài thêm Flatseal để cấu hình quyền cho các phần mềm cài bằng Flatpak.

4. Cài Gnome Extensions (có thể cài bằng Flatpak):

- Cài đặt:

    ```bash
    sudo xbps-install gnome-shell-extensions gnome-browser-connector
    ```

- Một số extension nên cài thêm:

    ```bash
    AppIndicator and KStatusNotifierItem Support
    Dash to Dock
    Input Method Panel
    ```

6. Cài Gnome Tweaks

    ```bash
    sudo xbps-install gnome-tweaks
    ```

## Cài đặt Fcitx5-Lotus:

1. Cài đặt fcitx5:

    ```bash
    sudo xbps-install fcitx5 fcitx5-configtool fcitx5-gtk fcitx5-qt
    ```

2. Bật tự khởi chạy cho fcitx5

    ```bash
    mkdir -p ~/.config/autostart
    ln -s /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
    ```

3. Cài đặt fcitx5-lotus


    ```bash
    sudo xbps-install acl acl-progs cmake extra-cmake-modules libfcitx5-devel libinput-devel eudev-libudev-devel gcc go gettext-devel pkg-config hicolor-icon-theme libX11-devel python3-QtPy python3-PyQt5 python3-pyqt6 python3-pyqt6-gui python3-pyqt6-widgets

    git clone --recursive https://github.com/LotusInputMethod/fcitx5-lotus.git
    cd fcitx5-lotus
    mkdir build && cd build
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib -DINSTALL_RUNIT=ON -DRUNIT_SV_DIR=/etc/sv ..
    make
    sudo make install
    ```

4. Tạo User và Group (thay thế systemd-sysusers):

    ```bash
    sudo groupadd -f input
    sudo useradd -M -g input -s /usr/bin/nologin -d / uinput_proxy
    ```

5. Reload Udev Rules:

    ```bash
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```

6. Chỉnh sửa file runit cho Void Linux (file gốc lỗi):

    ```bash 
    sudo nano /etc/sv/fcitx5-lotus/run
    ```

    Thay tất cả nội dung như sau:
    
    ```sh
    #!/bin/sh -e
    # Copyright (c) 2026 Contributors to the LotusInputMethod project
    # SPDX-License-Identifier: GPL-3.0-or-later

    exec 2>&1
    setfacl -m u:uinput_proxy:rw /dev/uinput
    exclude="run supervise log conf"
    while true; do
        for user in *; do
            case " $exclude " in *" $user "*) continue ;; esac
            [ ! -f "$user" ] && continue
            if ! pgrep -f "/usr/bin/fcitx5-lotus-server -u $user" > /dev/null; then
                echo "Starting server for: $user"
                (
                    exec setpriv --reuid=uinput_proxy --regid=input --init-groups \
                        --bounding-set -all,+sys_nice,+sys_ptrace \
                        --inh-caps +sys_nice,+sys_ptrace \
                        --ambient-caps +sys_nice,+sys_ptrace \
                        /usr/bin/fcitx5-lotus-server -u "$user"
                ) &
            fi
        done
        sleep 10
    done
    ```

7. Bật tự khởi chạy cho fcitx-lotus-server

    ```bash
    sudo chmod +x /etc/sv/fcitx5-lotus/run
    sudo ln -s /etc/sv/fcitx5-lotus /var/service
    sudo touch /var/service/fcitx5-lotus/$(whoami)
    sudo sv start fcitx5-lotus
    ```

8. Nạp Kernel Module uinput:

    ```bash
    sudo modprobe uinput
    ```

9. Thêm biến môi trường:

- Đối với bash shell (mặc định), chạy:

    ```
    cat <<EOF >> ~/.bash_profile
    export XMODIFIERS=@im=fcitx
    export QT_IM_MODULE=fcitx
    export QT_IM_MODULES="wayland;fcitx"
    export GLFW_IM_MODULE=ibus
    EOF
    ```

- Đối với fish shell, chạy:

    ```
    echo 'if status is-login
        set -Ux XMODIFIERS @im=fcitx
        set -Ux QT_IM_MODULE fcitx
        set -Ux QT_IM_MODULES "wayland;fcitx"
        set -Ux GLFW_IM_MODULE ibus
     ~/.config/fish/config.fish
    ```

- Đối với zsh shell, chạy:

    ```
    cat <<EOF >> ~/.zprofile
    export XMODIFIERS=@im=fcitx
    export QT_IM_MODULE=fcitx
    export QT_IM_MODULES="wayland;fcitx"
    export GLFW_IM_MODULE=ibus
    EOF
    ```

## Bật zRAM để có trải nghiệm tốt hơn ( ĐỌC KĨ )

1. zRam là gì?  
    Đối với các máy có lượng RAM thấp, nếu RAM chạm mức tối đa sẽ bị treo máy. Theo cách truyền thống, ta có thể ghi tạm dữ liệu quá tải vào ổ cứng (HDD/SSD), gọi là vùng Swap vật lý. Lâu ngày việc này làm ổ cứng bị đọc ghi rất nhiều.

    zRAM có thể giải quyết vấn đề đó. Khi dung lượng RAM đạt đến ngưỡng nhất định thì hệ thống sẽ nén dữ liệu trên RAM lại. Tất nhiên dữ liệu trước và sau khi nén đều nằm trên RAM, không đọc ghi vào ổ cứng.

    Ví dụ. máy tôi có 8 GB, tôi cần mở các phần mềm để làm việc. Nếu mở hết thì tôi cần dùng tới 9 GB. Điều này là quá sức. Khi bắt đầu đạt ngưỡng 7,5 GB, hệ thống sẽ nén. Đến khi mở hết chương trình thì 9GB này đã nén thành 7,5 GB. Tất nhiên 7,5 GB dữ liệu này vẫn nằm trên RAM, ta đã tiết kiệm 1,5 GB. Như vậy 1,5 GB này dung lượng của zRAM (swap).

    Ưu điểm: 

    - Hạn chế việc phải đọc ghi trên ổ cứng.
    - Mặc dù cần tốn thời gian để nén giải nhưng tốc độ vẫn hơn rất nhiều so với việc đọc ghi trên ổ cứng.

    Nhược điểm:

    - Tốn một chút % CPU.

    Nên dùng khi bạn có lượng RAM tương đối thấp (thường thì 8GB trở xuống). Nhiều distro cũng bật chức năng này mặc định.

2. Cài gói:

    ```bash
    sudo xbps-install zramen
    ```

3. Khởi chạy:

    ```bash
    sudo ln -s /etc/sv/zramen /var/service/
    sudo sv start zramen
    ```

4. Cấu hình:

    ```bash
    sudo nano /etc/sv/zramen/conf
    ```

    Nội dung mặc đinh như sau, bỏ dấu # đầu dòng để bật thuộc tính:

    ```bash
    #export ZRAM_COMP_ALGORITHM=ls4
    #export ZRAM_PRIORITY=32767
    #export ZRAM_SIZE=25
    #export ZRAM_MAX_SIZE=4096
    #export ZRAM_STREAMS=1
    #export ZRAMEN_SWAPON_DISCARD=both
    #export ZRAMEN_QUIET=0
    ```

    Trong đó:
    
    - ZRAM_COMP_ALGORITHM: dùng lz4 cho hiệu suất cao nhưng độ nén vừa phải, dùng zstd cho độ nén cao nhưng hiệu suất thấp hơn. Nhu cầu tôi không nhiều nên dùng mặc định lz4. Theo tôi biết thì Fedora mặc định dùng zstd.

    - ZRAM_PRIORITY: độ ưu tiên, dòng này để mặc định là giá trị lớn nhất, không có gì để chỉnh sửa.

    - ZRAM_SIZE: dung lượng zRAM sẽ bằng bao nhiêu % so với RAM thực tế.

    - ZRAM_MAX_SIZE: dung lượng zRAM tối đa, tính bằng MiB. Chỗ này tôi thấy để ZRAM_SIZE=100 và ZRAM_MAX_SIZE=8192 là ổn. Máy 4 GB RAM thì cũng chạy tối đa 4 GB, các máy từ 8 GB RAM trở lên thì chạy 8 GB tối đa.

    - ZRAM_STREAMS: số luồng của CPU dùng để nén (không set thì mặc định là 1). Ví dụ máy tôi là AMD R7 7730U 8 nhân 16 luồng khá mạnh nên tôi set giá trị 10.

    ZRAMEN_SWAPON_DISCARD: để mặc định là ổn rồi.

    - ZRAMEN_QUIET: giá trị 0 là bật log đầy đủ, giá trị 1 là chỉ log khi có lỗi nghiệm trọng. Chỗ này tôi thấy để giá trị 0 ổn hơn, dễ theo dõi mà cũng không ảnh hưởng hiệu năng mấy.

    Cấu hình tôi dùng như sau:

    ```bash
    export ZRAM_COMP_ALGORITHM=ls4
    export ZRAM_PRIORITY=32767
    export ZRAM_SIZE=100
    export ZRAM_MAX_SIZE=8192
    export ZRAM_STREAMS=10
    export ZRAMEN_SWAPON_DISCARD=both
    export ZRAMEN_QUIET=0
    ```

    Nhấn Ctrl + S để lưu và Ctrl + X để thoát.

    Khởi động lại zramen:

    ```bash
    sudo sv restart zramen
    ```

5. Đặt thêm một số thông số để nén tiến trình ngầm tốt hơn:

- Tạo file:

    ```bash
    sudo mkdir -p /etc/sysctl.d/
    sudo nano /etc/sysctl.d/99-zram.conf
    ```

- Thêm nội dung:

    ```bash
    vm.swappiness=100
    vm.page-cluster=0
    ```

    Nhấn Ctrl + S để lưu và Ctrl + X để thoát.

- Chạy lệnh: 

    ```bash
    sudo sysctl --system
    ```

## Tổng hợp một số hướng dẫn sửa lỗi trên Gnome

### 1. Lỗi không lưu độ sáng màn hình sau khi khởi động lại.

- Do Void Linux sửa dụng runit nên không có systemd-backlight để lưu độ sáng. Mỗi khi khởi động lại thì độ sáng màn hình sẽ reset về một mức nào đó. Ta sẽ tạo một service để runit chạy nó thay thế cho systemd-backlight.

- Tạo thư mục chứa service

    ```bash
    sudo mkdir -p /etc/sv/backlight
    ```

- Tạo file run (Khôi phục độ sáng khi mở máy)

    ```bash
    sudo nano /etc/sv/backlight/run
    ```

    Với nội dung:

    ```sh
    #!/bin/sh
    # Source: https://github.com/madand/runit-services

    [ -d '/var/cache/backlight/' ] || mkdir -p '/var/cache/backlight'

    for card in $(find /sys/class/backlight/ -type l); do
        device_name=$(basename "$card")
        storage_file="/var/cache/backlight/${device_name}-brightness-old"

        if [ -r "$storage_file" ]; then
            cat "$storage_file" > "$card/brightness" 2>/dev/null
        fi
    done

    exec pause
    ``` 

- Tạo file finish (Lưu độ sáng khi tắt máy)

    ```bash
    sudo nano /etc/sv/backlight/finish
    ```

    Với nội dung:

    ```bash
    #!/bin/sh
    # Source: https://github.com/madand/runit-services

    [ -d '/var/cache/backlight/' ] || mkdir -p '/var/cache/backlight'

    for card in $(find /sys/class/backlight/ -type l); do
        device_name=$(basename "$card")
        cat "$card/brightness" > "/var/cache/backlight/${device_name}-brightness-old"
    done

    ```

- Cấp quyền và khởi chạy:

    ```
    sudo chmod +x /etc/sv/backlight/run
    sudo chmod +x /etc/sv/backlight/finish
    sudo ln -s /etc/sv/backlight /var/service
    sudo sv start backlight
    ```

## Các nguồn tham khảo

[1] https://docs.voidlinux.org/installation/live-images/guide.html

[2] https://lotusinputmethod.github.io

[3] https://gist.github.com/nerdyslacker/398671398915888f977b8bddb33ab1f1

[4] https://github.com/NetBeholder/VoidLinux-installation-guide

[5] https://github.com/madand/runit-services
