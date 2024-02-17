
[![pipeline status](https://gitlab.com/Finalproject2943838/ansible/badges/main/pipeline.svg)](https://gitlab.com/Finalproject2943838/ansible/-/commits/main) 

# Подготовка виртуальной машины

логин и пароль не используется. Авторизация по ключам из gitlab group variable


## Предварительная подготовка машины.

У timeweb по дефолту добавляется пользователь root с ключами. Делается это из web интерфейса timewed.

Что бы подготовить машину для раскатывания ролей нужно зайти на виртуальную машину, создать пользователя ansible, добавить его в группу sudo и отредактировать файл /etc/sudoers `%sudo   ALL=(ALL:ALL) NOPASSWD: ALL` для поднятия прав без ввода пароля.
