#Тестовое задание: Развертывание Foreman, управление хостами и интеграция с Ansible

## Описание

Инфраструктура стенда развернута локально с помощью  
**Vagrant** - утилита для создания и конфигурирования виртуальной среды. Вся конфигурация серверов описана в конфигурационном файле [`Vagrantfile`](./Vagrantfile)

| Роль | ОС | vCPU | RAM | IP-адрес | FQDN |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Foreman Server** | AlmaLinux 9 | 2 | 4 GB | `192.168.56.10` | `foreman.example.local` |
| **Managed Host** | AlmaLinux 9 | 1 | 1 GB | `192.168.56.20` | `client.example.local` |

---

## Выполненные задачи

### 1. Развертывание Foreman
Установка выполнена с помощью инсталлятора `foreman-installer` с активацией плагинов для поддержки интеграции с Ansible:

```bash
foreman-installer \
  --enable-foreman-plugin-ansible \
  --enable-foreman-proxy-plugin-ansible
```
см. скриншот [`screenshots/01_foreman_dashboard.png`](./screenshots/01_foreman_dashboard.png).
### 2. Регистрация управляемого хоста (Managed Host)
Регистрация хоста `client.example.local` выполнена с помощью сгенерированного Foreman скрипта глобальной регистрации:

```bash
set -o pipefail && curl -sS --insecure 'https://foreman.example.local/register?hostgroup_id=1&location_id=2&operatingsystem_id=1&organization_id=1&update_packages=false' -H 'Authorization: Bearer <SECURE_TOKEN>' | bash
```
см. скриншот [`screenshots/02_all_hosts.png`](./screenshots/02_all_hosts.png).
### 3. Создание Host Group и Provisioning Template (NTP)
* Создан шаблон инициализации `Custom_NTP_setup` (код доступен в [`templates/custom_ntp_setup.erb`](./templates/custom_ntp_setup.erb)), выполняющий установку и базовую конфигурацию сервиса точного времени `chrony`.
* Создана группа хостов `Linux_Servers_Base`. 
* Хост `client.example.local` успешно включен в данную группу (см. скриншот [`screenshots/03_host_group.png`](./screenshots/03_host_group.png) [`screenshots/03_host_group1.png`](./screenshots/03_host_group1.png) [`screenshots/03_host_group2.png`](./screenshots/03_host_group2.png)).

### 4. Интеграция Ansible и применение роли запрета входа root
* Реализована кастомная Ansible-роль `disable_root_login` (исходный код в [`ansible/roles/disable_root_login/`](./ansible/roles/disable_root_login/)). Роль отключает параметр `PermitRootLogin` в конфигурации `/etc/ssh/sshd_config` и атомарно перезапускает службу `sshd`.
* Роль импортирована в веб-интерфейс Foreman и назначена на группу хостов `Linux_Servers_Base`.
* Для корректного применения настроен беспарольный доступ по SSH через публичный ключ.
* Успешный результат выполнения Ansible Job на скриншоте: [`screenshots/04_ansible.png`](./screenshots/04_ansible.png).

### 5. Сравнение подходов: Foreman vs Docker + Nginx + Samba
* **Docker + Nginx + Samba** - решают в основном задачу доставки файлов / образов / конфигурационных артефактов. Они не являются системой управления жизненным циклом машин. Чтобы после установки что-то настроить, отдельно нужен shell/Ansible
* **Foreman** - объединяет inventory, provisioning, шаблоны установки, привязку хостов к группам и интеграцию с configuration management. Host Group становится единым источником общих настроек.
### Плюсы и минусы решений

| Решение | Плюсы | Минусы |
| :--- | :--- | :--- |
| **Foreman** | • Динамические ERB-шаблоны (нет дублирования)<br>• Единый инвентарь оборудования (CMDB)<br>• Встроенная интеграция с Ansible и планировщик<br>• Масштабирование через Smart Proxy | • Высокий порог входа и сложная настройка<br>• Требователен к ресурсам |
| **Docker + Nginx + Samba** | • Предельно прост<br>• Ест немного памяти<br>• Samba удобна для сетевой раскатки Windows | • Статика: под каждый хост нужно вручную править файлы<br>• Нет обратной связи (успешно ли встала ОС неизвестно)<br>• Нет управления после установки (Ansible настраивается отдельно) |

---


