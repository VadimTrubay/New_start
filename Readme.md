### Readme.md
# Налаштування ovpn(Для чайників)

## 1. 📁 Підготовка
Перед початком у тебе має бути файл docker-compose.yml, наприклад:
```bash
services:
  openvpn:
    image: kylemanna/openvpn
    container_name: openvpn
    restart: always
    ports:
      - "1194:1194/udp"
    volumes:
      - ./openvpn-data:/etc/openvpn
    cap_add:
      - NET_ADMIN
```
openvpn-data — це папка, в яку зберігатиметься конфігурація, сертифікати тощо.

## 🔧 Крок 2: Генерація базової конфігурації сервера
```bash
  docker compose run --rm openvpn ovpn_genconfig -u udp://VPN.SERVER.IP
```
Що робить:
Генерує базову конфігурацію VPN-сервера.
Вказує, що клієнти підключатимуться через UDP на адресу VPN.SERVER.IP.
Зберігає файли в openvpn-data.
Пояснення:
ovpn_genconfig — скрипт, який налаштовує /etc/openvpn/openvpn.conf.
-u udp://... — каже: "слухай цей порт для підключень клієнтів".

## 🛡️ Крок 3: Ініціалізація PKI (сертифікатів)
```bash
  docker compose run --rm openvpn ovpn_initpki
```
Що робить:
Ініціалізує PKI (Public Key Infrastructure):
Створює CA (сертифікат авторитету).
Створює сертифікат сервера.
Запитує пароль (для захисту ключів).
Це як створення "головного центру сертифікатів", який потім видаватиме сертифікати клієнтам.

## ▶️ Крок 3: Запуск сервера
```bash
  docker compose up -d  
```
Що робить:
Запускає OpenVPN-сервер у фоновому режимі (-d).
Контейнер використовує створені конфіги й сертифікати для роботи.

## 👤 Крок 4: Створення клієнта
```bash
  docker compose run --rm openvpn easyrsa build-client-full client1 nopass
```
Що робить:
Створює ключ + сертифікат для клієнта client1
nopass означає без пароля на приватний ключ (інакше буде кожен раз питати пароль при підключенні)

## 👤 Крок 5: Створення клієнтського сертифіката
```bash
  docker compose run --rm openvpn ovpn_getclient client1 > client1.ovpn
```
Що робить:
Створює:
Клієнтський сертифікат (client1).
Конфіг-файл client1.ovpn, який містить:
IP/домен VPN-сервера
сертифікат клієнта
CA
ключ
Виводить результат у client1.ovpn, який можна використовувати на телефоні або ноутбуці.

##  🔍 Крок 6: Перевірка виданих сертифікатів
```bash
  docker compose exec openvpn ls /etc/openvpn/pki/issued
```
Що робить:
Показує список виданих сертифікатів.
Там мають бути файли типу client1.crt, client2.crt і т.д.
