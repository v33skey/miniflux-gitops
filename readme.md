# miniflux-ansible

## About
Автоматическое разворачивание Miniflux на VPS с помощью Ansible. Один прогон плейбука с локальной машины поднимает на сервере пользователей, Docker, сам Miniflux с базой Postgres

## Стек
- **Ansible** - оркестрация и конфигурация удаленного хоста
- **Docker** - рантайм для приложения
- **Miniflux** - RSS-читалка
- **Ansible-vault** - хранение секретов

## Как это работает

```site.yml``` последовательно запускает три плейбука:

1. **bootstrap.yml** - работает от root, создает пользователей deploy(без пароля, доступ по ssh-ключу) и администратора с паролем из vault и дополнительным ключом.

2. **docker.yml** - работает от ```deploy```, добавляет официальный APT-репозиторий Docker, устанавливает ```docker-ce```, ```docker-ce-cli```, ```docker-compose-plugin```, ```containerd.io``` и включает ```deploy``` в группу ```docker```.
3. **miniflux.yml** - работает от ```deploy```, создает директорию под ```docker-compose.yml``` файл, который копируется с локальной машины, ```db.env``` рендерится из шаблона ```db.env.j2```.

## Требования к запуску

Локально:

- Ansible
- Пара SSH-ключей для пользователей
- Создать пароль от ansible-vault

На хосте:

- Свежий Ubuntu/Debian с доступом по SSH под root

Установить необходимые ansible коллекции
```
ansible-galaxy collection install community.docker ansible.posix
```

## Быстрый старт

1. Указать свой сервер в ```ansible/inventory/hosts.yml```.
2. Создать свой файл с паролем от vault:
```
echo "vault_pass" > vaultpass.ini
```
3. Зашифровать пароли для vault с именами ```db_password``` и ```admin_password```
4. Заменить пароли в ```miniflux_vars.yml```
5. Для изменения админ-пароля необходимо:
```
mkpasswd --method=sha-512
```
Копируем вывод
```
ansible-vault encrypt_string --vault-password-file vaultpass.ini '$6$...(сюда вставить строку из вывода прошлой команды)' --name 'name_passwd'
```
Заменить ```v33skey_passwd``` на свой пароль
6. Запустить playbook:
```
ansible-playbook ansible/site.yml --vault-password-file vaultpass.ini
```

## Доступ к Miniflux
- URL: ```http://<IP_сервера>```
- Логин: ```admin```
- Пароль: ```сгенерирован ранее```