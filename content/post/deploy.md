+++
date = "2016-03-31T15:23:31+06:00"
draft = false
title = "Используем Git в качестве инструмента для деплоя веб приложения"
tags = ["deploy",]
+++

### Что имеем

У нас есть веб приложение, которое мы разрабатываем. Нам нужно быстро и легко добавлять изменения на продакшн

#### Что предлагает нам Git

post-merge — Этот хук вызывается ‘git-merge’, после того, как мы выполнили 'git-pull’ или 'git-merge’ на локальном репозитории. Хук не выполнится, если у нас будут конфликты при мердже.
post-checkout — Этот хук вызывается 'git-checkout’, после того, как мы выполним 'git-clone’ или 'git-checkout’.

#### Что делаем

Механизм деплоя будет примерно таким:
1. Поднимаем git сервер. Переводим всю разработку на git. Протестированный код заливаем в репозиторий
2. Пишем хуки, которые будут делать то, что нам нужно сделать после деплоя. В данном случае хук post-merge.
3. Если хотим развернуть рабочую конфигурацию на чистом сервере, то пишем хук post-checkout

#### К примеру, нам после деплоя надо почистить какую-то папку и перезагрузить апач.
```
#!/bin/bash /etc/init.d/apache2 stop find /path/to/folder -type f -delete /etc/init.d/apache2 start
```

Положим этот скрипт в .git/hooks/post-merge.
Выполняем git pull и у нас выполняется этот скрипт

Аналогично и для хука post-checkout

Плюсы:

Быстрая доставка изменений на продакшн.
Автоматизация
Централизированное хранение рабочего веб проекта


Минусы:

Данная схема не будет работать, если мы будем создавать новые ветки, путем git checkout -b newBranch
Данная схема не будет работать, если мы будем вручную мерджить изменения в локальном хранилище.
