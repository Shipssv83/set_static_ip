Hi, ![](https://user-images.githubusercontent.com/18350557/176309783-0785949b-9127-417c-8b55-ab5a4333674e.gif)my name is Serhii Shypylov
=========================================================================================================================================

-------------------------------

### Socials
<p align="left">
  <a href="https://t.me/oneitpro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/telegram-app.png" alt="Telegram" width="30" height="30" />
  </a>
  <a href="https://www.linkedin.com/in/sergey-shipilov-7262a31b4/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/linkedin.png" alt="LinkedIn" width="30" height="30" />
  </a>
  <a href="https://www.instagram.com/shipssvpl/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/instagram-new.png" alt="Instagram" width="30" height="30" />
  </a>
  <a href="https://www.facebook.com/profile.php?id=100083345006373">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/facebook.png" alt="Facebook" width="30" height="30" />
  </a>
  <a href="https://discord.com/invite/6z5EyagDyW?ref=1it.pro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/discord.png" alt="Discord" width="30" height="30" />
  </a>
  <a href="mailto:admin@1it.pro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/new-post.png" alt="Mail" width="30" height="30" />
  </a>
  <a href="https://1it.pro/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/domain.png" alt="Website" width="30" height="30" />
  </a>
</p>
---

```bash
#!/bin/bash
echo "
 SSSSSSS  EEEEEEE  RRRRRR   HHH   HHH IIIIIIII IIIIIIII
 SSS      EEE      RRR  RR  HHH   HHH    III     III
 SSSSSS   EEEEE    RRRRRR   HHHHHHHHH    III     III
     SSS  EEE      RRR  RR  HHH   HHH    III     III
 SSSSSSS  EEEEEEE  RRR   RR HHH   HHH IIIIIIII IIIIIIII

 SSSSSSS  HHH   HHH YYY   YYY PPPPPP   YYY   YYY LLL      OOOOOOO  VVV     VVV
 SSS      HHH   HHH  YYY YYY  PPP  PP   YYY YYY  LLL      OOO OOO   VVV   VVV
 SSSSSS   HHHHHHHHH   YYYYY   PPPPPP     YYYYY   LLL      OOO OOO    VVV VVV
     SSS  HHH   HHH    YYY    PPP        YYY     LLL      OOO OOO     VVVV
 SSSSSSS  HHH   HHH    YYY    PPP        YYY     LLLLLLL  OOOOOOO      VV

      11   iii tttttttt      pppppp  rrrrrrr   ooooooo
      11         ttt         pp   pp rr   rr  oo     oo
      11   iii   ttt   ..... pppppp  rr  rrr  oo     oo
      11   iii   ttt         pp      rr   rr  oo     oo
      11   iii   ttt         pp      rr    rr  ooooooo
"

# Функция для вывода сообщений с форматированием
fancy_echo() {
  local fmt="$1"; shift
  printf "\n$fmt\n" "$@"
}

# Запрос пароля администратора
sudo -v

# Обработка ошибок
trap 'ret=$?; test $ret -ne 0 && printf "failed\n\n" >&2; exit $ret' EXIT

set -e

# Проверка и добавление пользователя office-adm
create_office_adm_user() {
  if ! id -u office-adm > /dev/null 2>&1; then
    fancy_echo "Creating user office-adm..."
    read -s -p "Enter password for office-adm: " password
    echo
    sudo useradd -m -s /bin/bash office-adm
    echo "office-adm:$password" | sudo chpasswd
    sudo usermod -aG sudo office-adm
  else
    fancy_echo "User office-adm already exists. Skipping."
  fi
}

# Настройка sudo без пароля для пользователя $USER и office-adm
configure_sudo() {
  for user in $USER office-adm; do
    if ! sudo grep -q "^$user ALL=(ALL:ALL) NOPASSWD: ALL$" /etc/sudoers.d/$user; then
      fancy_echo "Configuring sudo for user $user..."
      echo "$user ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$user
    else
      fancy_echo "Sudo configuration for $user already exists. Skipping."
    fi
  done
}

# Установка необходимых пакетов
install_packages() {
  fancy_echo "Installing necessary packages..."
  sudo apt-get update
  sudo apt-get -y install vim python3 openssh-server ufw ansible \
    gzip p7zip-rar cabextract unace rar expect openjdk-8-jdk ecryptfs-utils \
    cryptsetup gimp google-chrome-stable vim python3-pip flameshot net-tools \
    speedtest-cli gnome-tweaks pritunl-client-electron dconf-editor flatpak \
    ubuntu-restricted-extras nautilus-admin exe-thumbnailer
}

# Установка пакетов из Snap
install_snap_packages() {
  fancy_echo "Installing snap packages..."
  sudo snap install telegram-desktop eversticky standard-notes teams-for-linux \
    postman bitwarden
}

# Настройка репозиториев
configure_repositories() {
  fancy_echo "Adding necessary repositories..."
  # Добавление репозитория AnyDesk
  echo "deb http://deb.anydesk.com/ all main" | sudo tee /etc/apt/sources.list.d/anydesk.list
  wget -qO - https://keys.anydesk.com/repos/DEB-GPG-KEY | sudo apt-key add -

  # Добавление репозитория Google Chrome
  echo "deb [arch=amd64] https://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
  wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo tee /etc/apt/trusted.gpg.d/google.gpg

  # Добавление Flathub в Flatpak
  if [ ! -f /var/lib/flatpak/repo/flathub ]; then
    flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  fi
}

# Установка AnyDesk
install_anydesk() {
  fancy_echo "Installing AnyDesk..."
  sudo apt-get update
  sudo apt-get -y install anydesk
}

# Настройка SSH
setup_ssh() {
  local ssh_dir="/home/$USER/.ssh"
  local authorized_keys="$ssh_dir/authorized_keys"
  local ssh_keys="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA3vlPm27iD4PnRPb6G+mYhRlHR6HY+5x908fEk0O57V s.shipilov"

  fancy_echo "Setting up SSH for user $USER..."

  mkdir -p "$ssh_dir"
  echo "$ssh_keys" > "$authorized_keys"

  chmod 700 "$ssh_dir"
  chown $USER:$USER "$ssh_dir"

  chmod 600 "$authorized_keys"
  chown $USER:$USER "$authorized_keys"

  cat "$authorized_keys"
}

# Настройка конфигурации SSHD
configure_sshd() {
  fancy_echo "Configuring SSH daemon..."

  sudo sed -i 's/^#\?PermitEmptyPasswords.*/PermitEmptyPasswords no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?PermitUserEnvironment.*/PermitUserEnvironment no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?PasswordAuthentication .*/PasswordAuthentication no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?UsePAM.*/UsePAM no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?X11Forwarding.*/X11Forwarding no/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?ClientAliveInterval.*/ClientAliveInterval 300/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?ClientAliveCountMax.*/ClientAliveCountMax 0/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?LogLevel.*/LogLevel VERBOSE/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?MaxAuthTries.*/MaxAuthTries 4/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?IgnoreRhosts.*/IgnoreRhosts yes/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?Protocol.*/Protocol 2/' /etc/ssh/sshd_config
  sudo sed -i 's/^#\?Banner.*/Banner \/etc\/issue.net/' /etc/ssh/sshd_config
}

# Настройка UFW для порта SSH
configure_ufw() {
  local ssh_port=22  # Установите нужный порт SSH

  if ! dpkg -l | grep -q ufw; then
    fancy_echo "Installing UFW..."
    sudo apt-get -y install ufw
  fi

  fancy_echo "Configuring UFW for SSH port $ssh_port..."
  sudo ufw allow $ssh_port/tcp comment 'ssh port'
}

# Перезагрузка службы SSH
restart_ssh_service() {
  if grep -q '^DISTRIB_RELEASE=24.04' /etc/lsb-release; then
    fancy_echo "Ubuntu 24.04 detected. Reloading systemd and restarting ssh..."
    sudo systemctl daemon-reload
    sudo systemctl restart ssh
  else
    fancy_echo "Other Ubuntu version detected. Restarting sshd..."
    sudo systemctl restart sshd
  fi
}

# Вывод информации о сетевом интерфейсе
print_network_info() {
  fancy_echo "Network information:"
  ip -o -4 addr show | awk '{print $2, $4}'  # Вывод IP-адресов и сетевых интерфейсов
}

# Основная логика
install_packages
install_snap_packages
configure_repositories
install_anydesk
configure_sudo
setup_ssh
configure_sshd
configure_ufw
restart_ssh_service
print_network_info

fancy_echo "Done."
```
