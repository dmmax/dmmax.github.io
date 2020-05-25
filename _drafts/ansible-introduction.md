---
title: "Введение в Ansible"
description: ""
tags: [ansible, devops]
---
## Задача
Рассказать что такое Ansible, установка и попробовать простые командные строки для демонстрации работы.

## Что это такое

*Ansible* – это система управления конфигурациями и она помогает оркестрировать вашими серверами. Чаще всего, Ansible используется для 
управления Linux-узлами, но также возможно и управление Windows (через WinRM соединение).
Ansible может работать в двух режимах. Раздавать команды серверам (режим ***push***) или сервера будут отслеживать изменения с репозитория и 
обновляться свою конфигурацию, используя ansible агент (режим ***pull***).

## Установка

1. [Установка python](https://opensource.com/article/19/5/python-3-default-mac). Предпочтительно ставить 3 версию, т.к. некоторые модули
поддерживают только 3 версию, но большинство работает и со второй и третьей версиями.
2. Установка Ansible [официальная документация](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip)

## Ansible в действии

Создадим папку для проекта + файл hosts для указания серверов, к которым будем подключаться

```bash
mkdir ansible-example
cd ansible-example
nano hosts
``` 
Файл hosts – Inventory file. Его нужно заполнить информацией о серверах. Для примера я поднял ec2 машинку.
```
[webservers]
web1     ansible_host=18.204.128.127     ansible_user=ec2-user 
```
Мой публичный ключ лежит на машинке, если у вас его там нет, то вам необходимо указать путь до приватного сертификата сервера
```
[webservers]
web1     ansible_host=18.204.128.127     ansible_user=ec2-user    ansible_ssh_private_key_file=/etc/ec2-keys/web1.pem
```
Убеждаемся, что указали верно host и пробуем подключиться к серверу
```bash
ansible -i hosts webservers -m ping
``` 
Команда *ansible* используется для выполнения одной команды, `-i hosts` указывает путь до inventory файла. `-m ping` указывает команду, 
которую вы хотите выполнить, `webservers` – название группы из inventory файла, но в данном случае можно было бы использовать `web1`, чтобы
указать конкретный сервер. Указание группы позволяет выполнить одну и ту же команду на нескольких серверах сразу.

Результат от предыдущей команды должен быть таким
```bash
web1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
Попробуем использовать другой полезный модуль `setup`
```bash
ansible -i hosts webservers -m setup
```
Данный модуль позволяет получить полезную информацию о сервере (ниже представлен результат с ее малой частью)
```bash
web1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.137.245.38"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::1058:16ff:fee7:6f1b"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "08/24/2006",
        "ansible_bios_version": "4.2.amazon",
...
```
Список всех возможных модулей находится на [официальной документации] (https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html)
 