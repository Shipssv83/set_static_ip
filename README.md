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
# Устанавливаем строгий режим выполнения скрипта для повышения надежности
set -euo pipefail

# Функция для вывода логов с временной меткой
log() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*"
}

# Функция для вывода ошибок и выхода из скрипта
error_exit() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: ERROR: $*" >&2
  exit 1
}

# Проверка, что скрипт выполняется от имени root
if [[ "${EUID}" -ne 0 ]]; then
  error_exit "Этот скрипт должен быть запущен от имени root."
fi

# Проверка наличия необходимых команд
required_commands=(ip netplan)
for cmd in "${required_commands[@]}"; do
  if ! command -v "$cmd" &>/dev/null; then
    error_exit "Команда '$cmd' не найдена. Пожалуйста, установите её и повторите попытку."
  fi
done

# Функция для определения активного сетевого интерфейса
get_default_interface() {
  ip route | grep '^default' | awk '{print $5}' | head -n1
}

# Чтение входных данных от пользователя или установка значений по умолчанию
read -rp "Введите статический IP-адрес (например, 10.10.0.35/24): " STATIC_IP
read -rp "Введите шлюз (gateway) (например, 10.10.0.1): " GATEWAY
read -rp "Введите DNS-серверы через запятую (например, 8.8.8.8,10.10.0.251): " DNS_SERVERS
read -rp "Введите имя сетевого интерфейса (оставьте пустым для автоматического определения): " INTERFACE

# Если интерфейс не указан, определить его автоматически
if [[ -z "$INTERFACE" ]]; then
  INTERFACE=$(get_default_interface)
  if [[ -z "$INTERFACE" ]]; then
    error_exit "Не удалось определить сетевой интерфейс. Пожалуйста, укажите его вручную."
  fi
  log "Автоматически определён сетевой интерфейс: $INTERFACE"
else
  log "Используется указанный сетевой интерфейс: $INTERFACE"
fi

# Валидация введённых данных
# Проверка IP-адреса
if ! [[ "$STATIC_IP" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}/[0-9]{1,2}$ ]]; then
  error_exit "Неверный формат статического IP-адреса. Пример правильного формата: 192.168.1.100/24"
fi

# Проверка шлюза
if ! [[ "$GATEWAY" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
  error_exit "Неверный формат шлюза. Пример правильного формата: 192.168.1.1"
fi

# Проверка DNS-серверов
IFS=',' read -ra DNS_ARRAY <<< "$DNS_SERVERS"
for dns in "${DNS_ARRAY[@]}"; do
  if ! [[ "$dns" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
    error_exit "Неверный формат DNS-сервера: $dns. Пример правильного формата: 8.8.8.8"
  fi
done

# Определение путей к конфигурационным файлам Netplan
NETPLAN_DIR="/etc/netplan"
NETPLAN_MAIN_FILE="${NETPLAN_DIR}/01-netcfg.yaml"
NETPLAN_CLOUDINIT_FILE="${NETPLAN_DIR}/50-cloud-init.yaml"

# Проверка существования директории Netplan
if [[ ! -d "$NETPLAN_DIR" ]]; then
  error_exit "Директория Netplan не найдена: $NETPLAN_DIR"
fi

# Обработка файла 50-cloud-init.yaml
if [[ -f "$NETPLAN_CLOUDINIT_FILE" ]]; then
  read -rp "Найден файл $NETPLAN_CLOUDINIT_FILE, который может конфликтовать с настройками. Хотите удалить его? (y/N): " CONFIRM
  case "$CONFIRM" in
    [yY][eE][sS]|[yY])
      log "Удаление файла $NETPLAN_CLOUDINIT_FILE..."
      rm -f "$NETPLAN_CLOUDINIT_FILE"
      ;;
    *)
      log "Оставление файла $NETPLAN_CLOUDINIT_FILE неизменным."
      ;;
  esac
fi

# Резервное копирование существующего конфигурационного файла Netplan
if [[ -f "$NETPLAN_MAIN_FILE" ]]; then
  BACKUP_FILE="${NETPLAN_MAIN_FILE}.bak.$(date +%F_%T)"
  log "Создание резервной копии существующего файла Netplan: $BACKUP_FILE"
  cp "$NETPLAN_MAIN_FILE" "$BACKUP_FILE"
else
  log "Файл Netplan не найден. Будет создан новый файл: $NETPLAN_MAIN_FILE"
fi

# Генерация новой конфигурации Netplan
log "Создание новой конфигурации Netplan..."

cat <<EOF > "$NETPLAN_MAIN_FILE"
network:
  version: 2
  renderer: networkd
  ethernets:
    $INTERFACE:
      dhcp4: no
      addresses:
        - $STATIC_IP
      gateway4: $GATEWAY
      nameservers:
        addresses: [$(echo $DNS_SERVERS | sed 's/,/, /g')]
EOF

# Применение новой конфигурации Netplan
log "Применение новой конфигурации Netplan..."
netplan apply || error_exit "Не удалось применить конфигурацию Netplan."

log "Статический IP успешно настроен:"
log "Интерфейс: $INTERFACE"
log "IP-адрес: $STATIC_IP"
log "Шлюз: $GATEWAY"
log "DNS-серверы: $DNS_SERVERS"

exit 0

# Дополнительные команды для проверки настроек:
# ip addr show $INTERFACE
# ip route
# systemd-resolve --status

# Восстановление Резервной Копии (если потребуется):
# sudo cp /etc/netplan/01-netcfg.yaml.bak.YYYY-MM-DD_HH:MM:SS /etc/netplan/01-netcfg.yaml
# sudo netplan apply
```
