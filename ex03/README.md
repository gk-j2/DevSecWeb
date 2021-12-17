# Лабораторная работа №3

## Задание 1

В папке lab3 находится nodejs уязвимое приложение. Необходимо развернуть его, найти источники XSS и исправить. Модифицированное приложение загрузить в свой репозиторий GitHub.

Для выполнения лабораторной потребуется проделать следующие шаги (если вы их проделали для лабораторной 2, то повторять не нужно):

1. Установить PostgreSQL сервер любой версии.

2. Создать БД lib.

3. Применить к ней скрипты из папки db (либо создать объекты и вставить данные в таблицы руками). Скрипты выполнять в порядке указанном в имени файла. Восстановить данные из файла data.sql

4. Установить nodejs версии 14.

5. Перейти в папку lab3 и выполнить в ней команду npm install.

6. Запустить сайт через Visual Studio Code или через команду npm start.

7. Войти на сайт и увидеть список книг и авторов.

8. На странице со списком книг найти:

8.1. Reflected XSS в поиске книг

Если в поле поиска ввести строку:

```html
<img src=1 onerror='javascript:alert(document.cookie)'/>
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/8.1.png" />
</p>

8.2. Persisted (Stored) XSS при создании книги и отображении списка книг

При добавлении книги вставим пейлоад <script>alert('XSS')<script>.
Пейлоад добавился на страницу.

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/8.2.png" />
</p>

8.3. Потенциальную уязвимость через Cookie Injection

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/8.3.png" />
</p>

8.4. Некорректное создание сессионной cookie, которое приводит к захвату сессии (Session hijacking)

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/8.4.1.png" />
</p>
<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/8.4.2.png" />
</p>
9. Написать отчёт с описанием найденных уязвимости и примерами их эксплуатации

10. Исправить уязвимость. В отчёте привести пример того, что уязвимости больше не эксплуатируются.
Для того, чтобы исправить уязвимости необходимо:
1. Запретить метасимволы, например '<' или '>'.

```javascript
app.post('/addbook', async (req, res) => {
    console.log("adding new book");
    let aid = req.body.author;
    let bname = req.body.bookname.replace('<', '|').replace('>', '|');
    let bid = Math.floor(Math.random() * 1000);
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/10.1.png" />
</p>

1. Реализовать более сильную генерацию куки, установить HttpOnly флаг. (Для большей надежности можно установить Secure, но в данном случае не применимо.)

```javascript
app.post('/signin', async (req, res) => {
    let login = req.body.name;
    let pass = req.body.pass;
    let sql = {
        text: "SELECT name as result FROM users WHERE name = $1 AND pass = $2", 
        values: [login, pass]
    };
    try{
        let data = await client.query(sql);
        let userId = data.rows[0].result;
        if(data.rows.length>0 && userId){
            const oneDayToSeconds = 24 * 60 * 60;
            let salt = Math.floor(CryptoJS.random * 1000);
            let cookie_val = CryptoJS.MD5(salt + userId)
            res.cookie('userId', cookie_val, {maxAge: oneDayToSeconds, httpOnly: true});
            res.redirect('/books');
```

<p align="center">
  <img src="https://github.com/gk-j2/DevSecWeb/blob/main/ex03/pictures/10.2.png" />
</p>

