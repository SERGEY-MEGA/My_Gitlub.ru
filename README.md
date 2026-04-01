# Как поднять свой GitLab дома и открыть к нему доступ из интернета

Это простой человеческий конспект: что купить, где взять GitLab, как связать домашний mini-PC, домен и VPS, и как в итоге получить свой рабочий GitLab по обычной ссылке.

Мой рабочий пример:

- сайт: [https://gitlub.ru](https://gitlub.ru)
- страница входа: [https://gitlub.ru/users/sign_in](https://gitlub.ru/users/sign_in)
- тестовый проект: [https://gitlub.ru/MEGA/deveps-mega-coder](https://gitlub.ru/MEGA/deveps-mega-coder)

## Зачем вообще поднимать свой GitLab

Если коротко:

- свои репозитории лежат у вас
- вы сами решаете, кто может регистрироваться
- можно открыть доступ извне по своему домену
- можно работать через браузер, HTTPS и SSH
- не нужно зависеть только от публичного SaaS

## Почему `gitlub.ru`, а не `gitlab.ru`

Потому что `gitlab.ru` уже был занят.

Для своего сервера нужен свободный домен, поэтому был выбран `gitlub.ru`.

## Почему не просто пользоваться `gitlab.com`

Если вам удобно, можно пользоваться и `gitlab.com`.

Но у self-hosted варианта есть свои плюсы:

- полный контроль над сервером
- свой домен
- свои правила регистрации
- своя почта и уведомления
- свои бэкапы и обновления

Плюс у GitLab.com есть отдельная тема с `identity verification`. В официальной документации GitLab указано, что в разных случаях могут требоваться дополнительные проверки, включая `email`, `phone` и иногда `credit card verification`.

Официальная страница:

- [GitLab Identity verification](https://docs.gitlab.com/security/identity_verification/)

## Что нужно купить или подготовить

Нужны 4 вещи:

1. mini-PC или любой домашний Linux-сервер
2. домен
3. VPS с белым IP
4. GitLab CE

## Где взять домен

Домен нужен для красивого постоянного адреса.

Где можно взять:

- [Beget Domains](https://beget.com/ru/domains)
- [REG.RU Domains](https://www.reg.ru/domain/new/)
- [RU-CENTER Domains](https://www.nic.ru/catalog/domains/)

Где брал я:

- домен был зарегистрирован через [Beget](https://beget.com/ru/domains)

Что делать:

1. покупаете свободный домен
2. ждете, пока он появится в панели управления
3. позже направляете `A`-запись домена на VPS

## Где взять VPS

VPS нужен как внешняя точка входа.

Зачем он нужен:

- у домашнего интернета часто нет белого IP
- не хочется пробрасывать лишние порты на роутере
- через VPS удобно поднять HTTPS и проксирование

Где можно взять:

- [JustHost VPS](https://justhost.ru/)
- [Timeweb Cloud VPS](https://timeweb.cloud/)
- [Beget Cloud VPS](https://beget.com/ru/vps)
- [Selectel Cloud Servers](https://selectel.ru/services/cloud/servers/)

Где брал я:

- VPS был взят у [JustHost](https://justhost.ru/)

Для такого сценария обычно хватает:

- 1 vCPU
- 1 GB RAM
- Linux Ubuntu

Если VPS нужен только как входная точка и reverse proxy, большой тариф не обязателен.

## Где взять GitLab

GitLab лучше брать только из официальных источников.

Основные ссылки:

- [GitLab Install Docs](https://docs.gitlab.com/install/)
- [GitLab Docker installation](https://docs.gitlab.com/install/docker/)
- [GitLab CE Docker image](https://hub.docker.com/r/gitlab/gitlab-ce)

Если хотите повторить путь попроще, самый практичный вариант для дома:

- `GitLab CE` в `Docker Compose`

## Как устроена схема

Схема простая:

1. GitLab работает дома на mini-PC
2. VPS смотрит в интернет
3. домен ведет на VPS
4. VPS проксирует запросы в домашний GitLab через обратный туннель
5. HTTPS выдается на VPS через Let's Encrypt

Итог:

- GitLab физически остается дома
- снаружи открывается обычный сайт по домену
- пользователям не нужен VPN

## Пошагово

### Шаг 1. Подготовить mini-PC

Нужен Linux-сервер дома.

Подойдет:

- mini-PC
- старый ПК
- NUC
- маленький домашний сервер

Главное:

- Linux
- Docker
- стабильный интернет
- достаточно места под репозитории и контейнеры

### Шаг 2. Поднять GitLab локально

Самый удобный путь для дома:

1. установить Docker и Docker Compose
2. поднять `gitlab/gitlab-ce`
3. проверить, что локально GitLab открывается

Нужные материалы:

- [Install Docker Engine](https://docs.docker.com/engine/install/)
- [GitLab Docker installation](https://docs.gitlab.com/install/docker/)

После запуска нужно убедиться, что GitLab отвечает локально, например на своем порту.

### Шаг 3. Купить домен

После покупки домена:

1. оставляете его в панели регистратора
2. создаете `A`-запись
3. позже направляете ее на IP VPS

В моем случае доменом стал `gitlub.ru`.

### Шаг 4. Купить VPS

На VPS нужно:

1. установить `Nginx` или `Caddy`
2. открыть `80/443`
3. выпустить сертификат Let's Encrypt

VPS здесь не хранит сами репозитории.

Он нужен только как:

- внешний вход
- HTTPS
- reverse proxy
- точка для туннеля

### Шаг 5. Связать VPS и домашний GitLab

Есть два основных пути:

- через Cloudflare Tunnel
- через VPS и reverse tunnel

В моем случае финально сработала схема:

- `домашний mini-PC -> reverse SSH tunnel -> VPS -> Nginx -> домен`

Почему это удобно:

- GitLab остается дома
- VPS почти не нагружается
- домен работает стабильно

### Шаг 6. Направить домен на VPS

Когда VPS готов:

1. в панели домена меняете `A`-запись
2. домен начинает смотреть на VPS
3. VPS уже отдает ваш GitLab наружу

### Шаг 7. Включить HTTPS

После того как домен уже смотрит на VPS:

1. ставите `certbot`
2. выпускаете сертификат
3. включаете HTTPS

Полезные ссылки:

- [Certbot](https://certbot.eff.org/)
- [Nginx](https://nginx.org/)

### Шаг 8. Проверить вход

Финально должны открываться:

- главная страница GitLab
- страница входа
- конкретный проект

У меня это сейчас:

- [https://gitlub.ru](https://gitlub.ru)
- [https://gitlub.ru/users/sign_in](https://gitlub.ru/users/sign_in)
- [https://gitlub.ru/MEGA/deveps-mega-coder](https://gitlub.ru/MEGA/deveps-mega-coder)

## Как давать доступ другим людям

Есть два варианта:

- создавать пользователей вручную
- включить регистрацию

Если включена регистрация, пользователь может подать заявку на аккаунт, а администратор потом одобрит его вручную.

Официальные ссылки:

- [Sign-up restrictions](https://docs.gitlab.com/administration/settings/sign_up_restrictions/)
- [Moderate users](https://docs.gitlab.com/administration/moderate_users/)
- [Email confirmation](https://docs.gitlab.com/security/user_email_confirmation/)

## Можно ли сделать так, чтобы пользователь регистрировался только после моего подтверждения

Да.

Для этого нужен режим с ручным подтверждением новых пользователей.

Смысл такой:

- человек заполняет регистрацию
- аккаунт не получает полный доступ сразу
- администратор заходит в панель GitLab и подтверждает пользователя

## Можно ли получать письмо, что кто-то хочет зарегистрироваться

Что можно сделать точно:

- включить SMTP
- включить ограничения на регистрацию
- вручную подтверждать новых пользователей

Официальная настройка почты:

- [GitLab SMTP settings](https://docs.gitlab.com/omnibus/settings/smtp/)

Что важно:

- в официальной документации GitLab я не нашел отдельной стандартной галочки формата "отправить админу письмо на каждую новую заявку"

То есть штатно лучше рассчитывать на:

- `Require admin approval for new sign-ups`
- рабочую почту GitLab
- ручную проверку пользователей в админке

Если захотите, уведомление админу можно потом добрать отдельно через:

- GitLab API
- cron
- маленький скрипт

## Для тех, кто хочет повторить коротко

1. Берете mini-PC с Linux.
2. Ставите Docker.
3. Поднимаете GitLab CE.
4. Покупаете домен.
5. Покупаете недорогой VPS.
6. На VPS ставите Nginx.
7. Связываете дом и VPS через туннель.
8. Направляете домен на VPS.
9. Включаете HTTPS.
10. Настраиваете регистрацию и почту.

## Полезные ссылки

- [GitLab Install Docs](https://docs.gitlab.com/install/)
- [GitLab Docker installation](https://docs.gitlab.com/install/docker/)
- [GitLab CE Docker image](https://hub.docker.com/r/gitlab/gitlab-ce)
- [GitLab Admin Area](https://docs.gitlab.com/administration/admin_area/)
- [GitLab Sign-up restrictions](https://docs.gitlab.com/administration/settings/sign_up_restrictions/)
- [GitLab Moderate users](https://docs.gitlab.com/administration/moderate_users/)
- [GitLab Email confirmation](https://docs.gitlab.com/security/user_email_confirmation/)
- [GitLab Identity verification](https://docs.gitlab.com/security/identity_verification/)
- [GitLab SMTP settings](https://docs.gitlab.com/omnibus/settings/smtp/)
- [Beget Domains](https://beget.com/ru/domains)
- [JustHost VPS](https://justhost.ru/)
- [Certbot](https://certbot.eff.org/)
