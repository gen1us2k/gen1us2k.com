+++
author = "Gen1us2k"
date = "2016-03-31T17:58:02+06:00"
description = ""
tags = ["Python", "Monitoring" ]
title = "Как мы домены мониторили"

+++


Всем привет! Все началось с того, что у нас в компании развелось очень много доменов которые нужно вовремя продлевать. И вот, после одного провала с продлением доменов, было принято решение начать мониторить дату истечения домена и выводить его в мониторинге Nagios.

#### Как мониторить?

У нас было несколько доменов в зонах kz, ru, kg, ge, com.

Самый простой способ узнать всю нужную информацию о домене — запустить whois. Это по идее должен знать каждый. Но как все это дело внедрить в мониторинг?

#### Чем мониторить?

Покопавшись по просторам интернета, был найден модуль [python-whois](https://pypi.python.org/pypi/python-whois).
Он хорошо выполнял свою работу для доменов com, net и кучи других доменов, описанных в описании к модулю.

Не хватало функционала для нескольких доменов в зонах kg.

В итоге появился форк проекта [python-whois-extended](https://github.com/gen1us2k/python-whois), который расширяет функционал для больших TLD.

Ок, как внедрить в нагиос?

Все просто, пишем простой чек
```
#!/usr/bin/env python#
# Usage:
# python check_domain.py -d DOMAIN
import whois
from datetime import datetime

from sys import exit
from optparse import OptionParser


def check_domain(domain):
    q = whois.query(domain)
    if (q.expiration_date - datetime.now()).days <= 30:
        print "CRITICAL: Domain: {0} expires on {1}".format(domain, q.expiration_date)
        exit(2)
    print "OK: Domain: {0} expires on {1}".format(domain, q.expiration_date)

if __name__ == '__main__':
    parser = OptionParser()
    parser.add_option("-d", "--domain", dest="domain", help="Domain to monitor expiry date")
    (options, args) = parser.parse_args()
    if not options.domain:
        print parser.print_help()
        exit(0)

    check_domain(options.domain)
```
Что он делает? Светит красным в мониторинге за месяц до истечения домена.

Что интересно, появился еще один мейнтейнер, который добавил поддержку для hk, cn и kr TLD.

Текущий список поддерживаемых доменов такой:

com, net, org, uk, pl, ru, lv, jp, co_jp, de, at, eu, biz, info, name, us, co, me, be, nz, cz, it, fr, kg, vc, fm, tv, edu, ca, cn, kr, hk

Пул-реквесты, запросы фич привествуются!

Надеюсь, мой опыт поможет избавиться от подобной проблемы
