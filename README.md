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
    sudo mkdir -p /etc/alsa/conf.d
    sudo ln -sf /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d/
    sudo ln -sf /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d/
    ```

6. Cấp quyền cho người dùng:

    ```bash
    mkdir -p ~/.config/autostart
    ln -sf /usr/share/applications/pipewire.desktop ~/.config/autostart/
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

CẢNH BÁO: Tiềm ẩn nguy cơ bảo mật do không khởi chạy bằng runit và cấp quyền riêng biệt theo hướng dẫn gốc!

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

- Cài đặt các gói:

    ```bash
    sudo xbps-install acl acl-progs cmake extra-cmake-modules libfcitx5-devel libinput-devel eudev-libudev-devel gcc go gettext-devel pkg-config hicolor-icon-theme libX11-devel python3-QtPy python3-PyQt5 python3-pyqt6 python3-pyqt6-gui python3-pyqt6-widgets
    ```

- Clone source và build:

    ```bash
    git clone --recursive https://github.com/LotusInputMethod/fcitx5-lotus.git
    cd fcitx5-lotus
    mkdir build && cd build
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib -DINSTALL_RUNIT=ON -DRUNIT_SV_DIR=/etc/sv ..
    make
    sudo make install
    ```

- Tạo User và Group (thay thế systemd-sysusers):

    ```bash
    sudo groupadd -f input
    sudo useradd -M -g input -s /usr/bin/nologin -d / uinput_proxy
    ```

- Reload Udev Rules:

    ```bash
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```

- Bật tự khởi chạy cho fcitx-lotus-server

    ```bash
    nano ~/.config/autostart/fcitx5-lotus-server.desktop
    ```
    Thêm vào nội dung:
    
    ```bash
    [Desktop Entry]
    Name=Fcitx5 Lotus Server
    GenericName=Input Method Server
    Comment=Backend server for fcitx5-lotus
    Exec=/usr/bin/fcitx5-lotus-server
    Terminal=false
    Type=Application
    Categories=System;Utility;
    StartupNotify=false
    X-GNOME-Autostart-enabled=true
    ```

    Nhấn Ctrl + S để lưu và Ctrl + X để thoát.

- Nạp Kernel Module uinput:

    ```bash
    sudo modprobe uinput
    ```

- Đối với bash shell (mặc định), chạy:

    ```bash
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

## Các nguồn tham khảo

[1] https://docs.voidlinux.org/installation/live-images/guide.html

[2] https://lotusinputmethod.github.io

[3] https://gist.github.com/nerdyslacker/398671398915888f977b8bddb33ab1f1

[4] https://github.com/NetBeholder/VoidLinux-installation-guide