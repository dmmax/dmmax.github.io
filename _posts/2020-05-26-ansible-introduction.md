---
title: "Введение в Ansible"
description: "Описание Ansible. Установка Ansible и Python. Выполнение базовых Ansible ad-hoc команд."
excerpt: "Устанавливаем Python и Ansible. Выполнение Ansible ad-hoc команд."
tags: [ansible, devops]
toc: true
---
## Задача
Рассказать что такое Ansible, установка и попробовать простые командные строки для демонстрации работы.

## Описание Ansible

*Ansible* – это система управления конфигурациями и она помогает оркестрировать вашими серверами. Чаще всего, Ansible используется для 
управления Linux-узлами, но также возможно и управление Windows (через WinRM соединение).
Ansible может работать в двух режимах:
* Режим **push** – Master сервер раздает команды управляемым серверам
* Режим **pull** – Управляемые сервера обращаются к Master серверу для обновления настроек через ansible агент

## Установка Ansible

1. Установка Python. Советую ставить 3 версию, т.к. некоторые из модулей поддерживает только ее, но остальные модули отлично работают со 
второй и третьей. [Гайд на установку](https://opensource.com/article/19/5/python-3-default-mac).
2. Установка Ansible [официальная документация](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip)

## Ansible в действии (выполнение ad-hoc команд)

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
Команда `ansible` используется для выполнения одной команды, `-i hosts` указывает путь до inventory файла. `-m ping` указывает команду, 
которую вы хотите выполнить, `webservers` – название группы из inventory файла, но в данном случае можно было бы использовать `web1`, для
указания конкретного сервера. Указание группы позволяет выполнить одну и ту же команду на нескольких серверах сразу.

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
Попробуем использовать другой полезный модуль – `setup`
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
Также можно выполнить shell скрипт напрямую на машинке:
```bash
ansible -i hosts webservers -m shell -a "pwd"
``` 
Аргумент `-a` используется для указания аргументов. Результат команды:
```bash
web1 | CHANGED | rc=0 >>
/home/ec2-user
```
Сейчас можно сделать что-нибудь полезное: например установим apache.
```bash
ansible -i hosts webservers -m yum -a "name=httpd state=latest" -b
```
Аргумент `-b` от слова become, обозначает запуск команды под root пользователем. Результат выполнения:
```bash
web1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true, # свойство "changed" показывает, что выполненная команда внесла изменения на сервере
    "changes": {
        "installed": [
            "httpd"
        ],
        "updated": []
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins:
...
```
Если мы запустим ту же команду, то ansible не будет устанавливать apache второй раз
```bash
web1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false, # изменений не было
    "changes": {
        "installed": [],
        "updated": []
    },
    "msg": "",
    "rc": 0,
    "results": [
        "All packages providing httpd are up to date",
        ""
    ]
}
```
Это связано с тем, что ansible перед тем как выполнить задачу, проверяет состояние модуля и если его состояние такое же, как вы запрашиваете, 
то он пропускает выполнение.

Список всех возможных модулей находится в [официальной документации] (https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html)