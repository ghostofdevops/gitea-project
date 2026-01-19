Сценарий автоматического развертывания Gitea на Debian 13.  
Установка выполняется с Control-машины (требуется Docker) на Target-машину.  
Требуется root-доступ к Target-машине. Должен быть включен логин по паролю (PermitRootLogin yes).

## Конфигурация портов

*   **Gitea Web:** 80
*   **Gitea Git SSH:** 22
*   **System SSH:** 2022 (переносится с 22 порта)

## Инструкция по запуску

Все команды выполняются из корня проекта.

### 1. Подготовка окружения

Клонировать репозиторий:
```bash
git clone https://github.com/ghostofdevops/gitea-project.git
cd gitea-project
```
ИЛИ
```bash
git clone git@github.com:ghostofdevops/gitea-project.git
cd gitea-project
```

Настроить доступ к серверу (SSH keys):
```bash
# 1. Генерация ключей (если отсутствуют)
ssh-keygen -t rsa -b 4096

# 2. Копирование ключа на сервер
ssh-copy-id root@<TARGET_IP>

# 3. Проверка подключения (должен быть вход без пароля)
ssh root@<TARGET_IP>

# 4. Добавить host key в known_hosts на control
ssh-keyscan -H 192.168.X.X >> ~/.ssh/known_hosts
```

### 2. Настройка инвентаря

Указать IP-адрес target-сервера в файле `ansible-deploy/inventory`:
```ini
[gitea_server]
192.168.X.X ansible_user=root
```

### 3. Сборка образа управления

Собрать Docker-образ с Ansible:
```bash
cd ansible-docker
docker build -t ansible-control:deb13 .
cd ..
```

### 4. Запуск деплоя

Запустить плейбук:
```bash
docker run --rm -it \
  -v "$(pwd)/ansible-deploy":/ansible \
  -v "$HOME/.ssh/id_rsa":/root/.ssh/id_rsa:ro \
  -v "$HOME/.ssh/known_hosts":/root/.ssh/known_hosts:ro \
  ansible-control:deb13 \
  ansible-playbook -i /ansible/inventory /ansible/deploy_gitea.yaml
```

## Проверка

1.  В браузере перейти по адресу `http://<TARGET_IP>`. Должна открыться страница установки Gitea.
2.  Системный SSH доступ к серверу теперь доступен по порту 2022:

```bash
ssh -p 2022 root@<TARGET_IP>
 ```
