# Комментарии к выполненному тестовому заданию

Привет. Представляю вашему вниманию решение тестового задания. Инструкция по
запуску в самом низу, но я бы просил прочесть комментарии к решению тоже, раз уж я их написал.
5 min read.

## Постановка задачи
Задание было сформулировано так: 
```
Тестовое задание. 
Реализовать веб приложение которое позволит авторизироваться, 
добавлять данные(как вариант можно реализовать таблицу для продаж 
товаров) в бд через страничку сайта и отображать данные из бд на 
отдельной странице. 
Качество верстки не важно, тип бд на выбор - mysql\postgres\sqlite. 
Должна быть инструкция по запуску приложения. 
Код залить на github и предоставить ссылку.
```

## Комментарии к решению  
1. Из задания не понятно однозначно, надо ли аутентифицировать пользователя для доступа ко всем страницам или только
для страницы добавления данных. Сделал аутентификацию по всем страницам. Исправляется на "показывать всем" удалением 
пары строчек. Разумеется, с реальной задачей надо было бы уточнить требования, на игрушечную не стал тратить время.

2. Для хранилища выбрана Sqlite3, чтобы было проще собрать и запустить приложение,
не поднимая полноценную базу. На целевой машине, соответственно, должна быть только sqlite3, а это буквально 
любой дистрибутив Linux или отсносительно свежая версия macOS. Библиортека для Sqlite - это, кстати,
единственная внешняя зависимость, а минимум внешних зависимостей я считаю в целом хорошей практикой. 
Хранилище полностью реализует CRUD, несмотря на то, что для решения этого не требуется. (Но при этом веб часть реализует 
только тот функционал, что был запрошен.)

3. Имена пользователей и пароли хранятся в plain-text файле тоже для простоты и чтобы 
не тащить в проект лишние библиотеки (для хранения хэша пароля в норме бы использовал golang.org/x/crypto/bcrypt).

4. Аутентификация пользователя на уровне HTTP организована с помощью отвечающего стандарту способа Authorization: Basic.
https://datatracker.ietf.org/doc/html/rfc7617 Опять же для того, чтобы не тащить в проект лишние библиотеки или JavaScript. 
Очевидно, что в реальности такой тип аутентификации можно использовать только с TLS (т.к. имя пользователя и пароль передаются 
в base64, т.е. в plain-text). Кроме того, это решение упростило код, т.к. стандартная библиотека Go знает про Basic, что удобно.
И да, программе можно указать файлы сертифика и ключа, и тогда будет TLS.

5. Маршрутизация по endpoints намеренно сделана без сторонних библиотек чисто на http.Handle, опять же чтобы избежать лишних зависимостей.
(Gorilla mux многие очень любят приносить для этого в проект, и это функциональный роутер). Соответственно, проверки методов запроса сделаны руками.

6. Код хендлеров http запросов не покрыт тестами. В норме надо было бы сделать тесты с помощью относительно недавно появившегося
в стандартной библиотеке модуля https://golang.org/pkg/net/http/httptest/ Я, честно говоря, просто пожалел времени. Код хранилища и аутентификации при этом покрыты базовыми тестами. 

7. Струтура проекта могла бы быть лучше. Я вынес package main в корень проекта, чтобы упростить вам задачу запуска программы и сэкономить время.
В норме main хорошо бы класть в _repo root_/cmd.

8. Такое понятие как "верстка" в моем решении остутсвует как класс, раз это было допустимо по условию. Честно не люблю делать пользовательские
интерфейсы, если речь не про CLI- утилиты (там люблю, да).


## Инструкция по запуску
Программа собирается компилятором Go версии 1.16.4. Более ранние версии (от 1.13) могут и должны работать, но это не проверялось.
Программа собирается на Linux и macOS. Windows у меня нет. 

В системе должны быть установлены sqlite3 и gcc (или clang на macOS) для сборки пакета **github.com/mattn/go-sqlite3**, т.к. библиотека
использует cgo.

Программе требуется модуль go-sqlite3. Инструкция по установке этой библиотеки находится здесь: https://github.com/mattn/go-sqlite3#installation.

После установки библиотеки:

git clone https://  
cd astral && go build 

В репозитории уже лежит "база данных" уже наполненная рандомными объектами - **sample.db**, чтобы сразу что-то показывать. Также в корне репозиотрия 
лежит файл **users.conf**, в который можно добававлять user credentials. На одной строке должно быть два слова - имя пользователя (alphanumeric characters) 
и пароль (любые символы). По любой из найденных в файле пар можно залогиниться.

Для запуска со значениями по умолчанию достаточно сказать **./astral**. Cервер будет слушать на **127.0.0.1:8083**. 

Имя пользователя: **user1**, пароль: **password1**. 

Чтобы изменить поведение по умолчанию - см. **./astral -- help**.

**Ctrl+C** - чтобы завершить программу.
