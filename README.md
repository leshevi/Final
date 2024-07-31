Добрый день.
1 Userstory
Мы, ООО “Вектор” - начинаем новое для себя направление деятельности,
связанное с общественной деятельностью. Для реализации нашей программы нам
нужен инструмент в виде веб-сайта.
У нас есть inhouse команда управления контентом, которая выбрала в качестве
инструмента CMS WordPress. На внутреннем тестовом стенде развернут WP, в котором
сверстаны несколько страниц и размещено какое-то количество контента. Мы
планируем выйти в продуктив со своим продуктом, и для этого нам нужна боевая
инфраструктура.
Решили делать проект по этой схеме, но после пришлось изменить в некоторых местах нашу
задумку, но то что наработали мы оставили в пректе для будущей работы.
Идём по пунктам/ первым пунктом, чтобы не ждать квот от VK Cloud, я зарегистрировалась
на TimeWeb(создала логин и пароль). Сразу раздала сотрудникам в группе, чтобы ознакомились с возможностями платформы. Паралельно создала группу в Gitlab и раздала
доступ всем участникам группы. Но нам так не понравилось и мы сделали один доступ на
всех кроме меня (это нам приглянулось больше), потому что делали всё в telegram c
расшаренным экраном и писали видео, кто не присутствовал мог посмотреть в свободное
время и быть в курсе событий на проекте.
Первое что сделали в gitlab прописали переменные на весь проект:
APP0_PUB 80.68.156.78
APP0_PRIV 192.168.0.7
APP1_PUB 185.247.16.175
APP1_PRIV 192.168.0.6
DB_PUB 185.247.16.236
DB_PRIV 192.168.0.5
MON_PUB 185.247.16.17
MON_PRIV 192.168.0.4
CI_SSH_USERNAME ansible
SSH_ROOT_PRIVATE_KEY
SSH_ROOT_PUBLIC_KEY
Использовали ranner «sysbox-runner». Так как проект не большой ip серверов прописали в
переменных.
Второе начали писать роли и плейбуки в ansible, для того чтобы подготовить сервера для
нашего проекта.
В проекте Ansible в .gitlab-ci.yml roli плеелись так
1.ansible-playbook playbooks/prepare.yml
2.ansible-playbook playbooks/prepare_nfs.yml
3.ansible-playbook playbooks/wordpress_nfs.yml
4.ansible-playbook playbooks/docker.yml
В первом плейбуке отработали две роли:
в первой установили софт
- net-tools
- tcpdump
- vim
- ncdu
- htop
- curl
- wget
- tree
- rsync
во второй отключили подключение по ssh через протокол ipv6, отключили swap и заменили
99-sysctl.conf.
Во втором плейбуке отработала одна роль по созданию nfs server.
В третьем плейбуке создали дирректорию под wordpress.
В четвёртом плейбуке установили, настроили docker и добавили пользователя в
соответствующую группу . Одним словом подготовили наши сервера для работы с
контейнерами.
Всё теперь наши машины готовы для реализации нашего проекта.
База данных. Завели проект mysql, в него написали docker-compose.yml и .gitlab-ci.yml. В
docker compose описан сервис который по CI разворачивается на ноде которую выбрали в
качестве базы дванных. В файле .gitlab-ci.yml написан pipeliпe который разворачивает базу данных из переменных gitlab и docker compose.
Wordpress разворачивается так же как и база данных из docker compose, но на двух нодах.
Так как у нас точная копия wordpress разворачиваем с одного файла docker compose. NFS
сервер развернули ansible ролью на ноде базы данных. При старте сервисов wopdpress
монтируем persistent volume. Wordpress сервисы абсолютно одинаковые на разных ip для
отказоустойчивости.
Keepalived. Отказоустойчивость выполнили за счет сервиса loadbalancer в облаке timeweb.
Timeweb не предоставел нам технической возможности использовать keepalived, но docker
сервисы для него были написаны. Keepalived - это программный комплекс обеспечивающий
высокую доступность и балансировку нагрузки. Завели проект Keepalived в группе проектов.
В нём два docker-compose.yml для двух серверов соответственно.
Мониторинг.
Prometheus получает метрики из разных сервисов и собирает их в одном месте. В нашем
случае Prometheus будет собирать данные из Node Exporter и Cadvisor. Node
exporter собирает метрики операционной системы и через HTTP предоставляет к ним доступ.
Cadvisor делает тоже самое, но для docker. Grafana отображает данные, полученные из
Prometheus. Эти данные можно отобразить в виде диаграмм и графиков, объединив в
дашборды.
Мониторинг проекта начали с установки node-exporter(мониторинг наших серверов) так же с
помощью docker-compose.yml и .gitlab-ci.yml в которых прописали наши глобальные
переменные и порты 9100:9100 и там же установили кроме этого ещё cadvisor(мониторинг
контейнеров) с портом 8080:8080.
После установили Prometheus который собирает метрики с наших сервисов используя Node
Exporter и cadvisor и пересылает всё в Grafana на каторой установили два самых удобных по нашему дашборда. Вот какие скрины из Grafana у нас получились.
Уведомления о том что в проекте на gitlab начались какие то изменения в telegram приходят с помощью telegram-bota. Оповещение приходит в канал где мы как пользователи все прописаны.
ку ку