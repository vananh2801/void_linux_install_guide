# Void_linux_install_guide

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

    Lúc này máy sẽ hiện danh sách các interface và trạng thái. Như máy tôi là tên interface là **“wlp2s0”**.

3. Cấu hình Wi-fi:

    ```bash
    wpa_passphrase SSID PASSWORD >> /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
    ```

    Trong đó, **"SSID"** là tên Wi-fi, **"PASSWORD"** là mật khẩu.

4. Khởi chạy dịch vụ wpa_supplicant:

    ```bash
    sudo ln -s /etc/sv/wpa_supplicant /var/service/
    sudo sv start wpa_supplicant
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

## Cài gói pipewire để có âm thanh và streaming trên Gnome

1. Cài đặt các gói:

    ```bash
    sudo xbps-install -S pipewire wireplumber alsa-pipewire pipewire-pulse libspa-bluetooth pulseaudio-utils pulsemixer
    ```

2. Cấu hình ALSA:

    ```bash
    sudo mkdir -p /etc/alsa/conf.d
    sudo ln -sf /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d/
    sudo ln -sf /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d/
    ```

3. Cài đặt tự khởi động pipewire:
    
    ```bash
    mkdir -p ~/.config/autostart
    ln -sf /usr/share/applications/pipewire.desktop ~/.config/autostart/
    ln -sf /usr/share/applications/pipewire-pulse.desktop ~/.config/autostart/
    ln -sf /usr/share/applications/wireplumber.desktop ~/.config/autostart/
    ```

4. Cấp quyền cho người dùng:

    ```bash
    sudo usermod -aG audio,video $USER
    ```

## Cài đặt Bluetooth

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

2. Cài một số phông từ Microsoft (ví dụ Arial, Times New Roman,... ):

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

