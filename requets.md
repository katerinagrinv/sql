## Создание и наполнение таблиц
1. Таблица genre

```SQL
CREATE TABLE genre(
    genre_id INT PRIMARY KEY AUTO_INCREMENT, 
    name_genre VARCHAR(30)
);
```
```SQL
INSERT INTO genre (name_genre) 
VALUES 
    ('Роман'),
	('Поэзия'),
	('Приключения');
```

2. Таблица book
```SQL
CREATE TABLE book(
    book_id	INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(50),
    author	VARCHAR(30),
    price	DECIMAL(8, 2),
    amount	INT    
);
```
```SQL
INSERT INTO book (title, author, price, amount) 
VALUES 
    ('Белая гвардия', 'Булгаков М.А.', '540.50', '5'),
    ('Мастер и Маргарита', 'Булгаков М.А.', '670.99', '3'),
    ('Идиот', 'Достоевский Ф.М.', '460.00', '10'),
    ('Братья Карамазовы', 'Достоевский Ф.М.', '799.01', '2'),
    ('Стихотворения и поэмы','Есенин С.А.','650.00','15');
```

3. Таблица supply
```SQL
CREATE TABLE supply(
    supply_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(50),
    author	VARCHAR(30),
    price	DECIMAL(8, 2),
    amount	INT
);
```
```SQL
INSERT INTO supply (title, author, price, amount) 
VALUES
    ('Лирика','Пастернак Б.Л.','518.99','2'),
    ('Черный человек','Есенин С.А.','570.20','6'),
    ('Белая гвардия','Булгаков М.А.','540.50','7'),
    ('Идиот','Достоевский Ф.М.','360.80','3');
```
4. Таблица author
```SQL
CREATE TABLE author(
    author_id	INT PRIMARY KEY AUTO_INCREMENT,
    name_author	VARCHAR(50)
);
```
```SQL
INSERT author(name_author)
VALUES  
    ('Булгаков М.А.'),
    ('Достоевский Ф.М.'),
    ('Есенин С.А.'),
    ('Пастернак Б.Л.');
```

## Выборка данных

#### Задание 1
В конце года цену всех книг на складе пересчитывают – снижают ее на 30%. Написать SQL запрос, который из таблицы book выбирает названия, авторов, количества и вычисляет новые цены книг. Столбец с новой ценой назвать new_price, цену округлить до 2-х знаков после запятой.
```SQL
SELECT title, author, amount, ROUND((price*0.7),2) AS new_price
    FROM book;
```
#### Задание 2
При анализе продаж книг выяснилось, что наибольшей популярностью пользуются книги Михаила Булгакова, на втором месте книги Сергея Есенина. Исходя из этого решили поднять цену книг Булгакова на 10%, а цену книг Есенина - на 5%. Написать запрос, куда включить автора, название книги и новую цену, последний столбец назвать new_price. Значение округлить до двух знаков после запятой.
```SQL
SELECT 
    author, 
    title,
    ROUND(IF(author='Булгаков М.А.', price + price*0.1, IF(author='Есенин С.А.', price + price*0.05, price)),2) 
    AS new_price
FROM book;
```
#### Задание 3
Вывести название и автора тех книг, название которых состоит из двух и более слов, а инициалы автора содержат букву «С». Считать, что в названии слова отделяются друг от друга пробелами и не содержат знаков препинания, между фамилией автора и инициалами обязателен пробел, инициалы записываются без пробела в формате: буква, точка, буква, точка. Информацию отсортировать по названию книги в алфавитном порядке.
```SQL
SELECT title, author
FROM book
WHERE title LIKE '_% %'
    AND (author LIKE '% С._.' OR author LIKE '% _.С.' )
ORDER BY title;
```
#### Задание 4
Посчитать стоимость всех экземпляров каждого автора без учета книг «Идиот» и «Белая гвардия». В результат включить только тех авторов, у которых суммарная стоимость книг (без учета книг «Идиот» и «Белая гвардия») более 5000 руб. Вычисляемый столбец назвать Стоимость. Результат отсортировать по убыванию стоимости.
```SQL
SELECT author, SUM(price*amount) AS Стоимость
FROM book
WHERE title <> 'Идиот' AND title <> 'Белая гвардия'
GROUP BY author
HAVING SUM(price*amount) >5000
ORDER BY SUM(price*amount) DESC;
```
#### Задание 5
Вывести информацию (автора, название и цену) о тех книгах, цены которых превышают минимальную цену книги на складе не более чем на 150 рублей в отсортированном по возрастанию цены виде.
```SQL
SELECT author,title, price
FROM book
WHERE (price - ( SELECT MIN(price)FROM book)) <= 150
ORDER BY price ASC;
```
## Запросы корректировки
#### Задание 1
Занести из таблицы supply в таблицу book только те книги, авторов которых нет в  book.
```SQL
INSERT INTO book (title, author, price, amount) 
SELECT title, author, price, amount 
FROM supply
WHERE author NOT IN (
    SELECT author 
    FROM book
);
```
#### Задание 2
Для тех книг в таблице book , которые есть в таблице supply, не только увеличить их количество в таблице book ( увеличить их количество на значение столбца amountтаблицы supply), но и пересчитать их цену (для каждой книги найти сумму цен из таблиц book и supply и разделить на 2).
```SQL
UPDATE book,supply
SET book.amount = book.amount + supply.amount,
	book.price = (book.price + supply.price)/2
WHERE book.title = supply.title AND book.author = supply.author;

SELECT * FROM book;
```
#### Задание 3
Удалить из таблицы supply книги тех авторов, общее количество экземпляров книг которых в таблице book превышает 10.
```SQL
DELETE FROM supply
WHERE author IN (SELECT author
                FROM book
                WHERE amount >=10);

SELECT * FROM supply;
```

## Соединение таблиц

1) удаляем таблицу book
```SQL
DROP TABLE book;
```
2) создаем и заполняем новую таблицу book
```SQL
CREATE TABLE book (
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50), 
    author_id INT NOT NULL, 
    genre_id INT,
    price DECIMAL(8,2), 
    amount INT, 
    FOREIGN KEY (author_id)  REFERENCES author (author_id) ON DELETE CASCADE,
    FOREIGN KEY (genre_id)  REFERENCES genre (genre_id) ON DELETE SET NULL
);
```
```SQL
INSERT INTO book (title,author_id,genre_id,price,amount)
VALUES 
    ('Белая гвардия', 1, 1, '540.50', '5'),
    ('Мастер и Маргарита', 1, 1, '670.99', '3'),
    ('Идиот', 2, 1, '460.00', '10'),
    ('Братья Карамазовы', 2, 1, '799.01', '2'),
    ('Лирика', 4, 2, '518.99', '2'),
    ('Игрок', 2, 1, '480.50', '10'),
    ('Черный человек', 3, 1, '570.20', '6'),
    ('Стихотворения и поэмы',3, 2,'650.00','15');
```
#### Задание 1
Вывести название, жанр и цену тех книг, количество которых больше 8, в отсортированном по убыванию цены виде.
```SQL
SELECT title, name_genre, price
FROM book
INNER JOIN genre
ON genre.genre_id = book.genre_id
WHERE amount>8
ORDER BY price DESC;
```
#### Задание 2
Вывести все жанры, которые не представлены в книгах на складе.
```SQL
SELECT name_genre
FROM genre
LEFT JOIN book
ON genre.genre_id = book.genre_id
WHERE book.genre_id is NULL;
```
#### Задание 3
Посчитать количество экземпляров  книг каждого автора из таблицы author.  Вывести тех авторов,  количество книг которых меньше 10, в отсортированном по возрастанию количества виде. Последний столбец назвать Количество
```SQL
SELECT name_author,SUM(amount) AS Количество
FROM author
LEFT JOIN book
    ON author.author_id = book.author_id
GROUP BY name_author
HAVING SUM(amount) < 10 OR SUM(amount) IS NULL
ORDER BY Количество;
```
#### Задание 4
 Вывести информацию о книгах (название книги, фамилию и инициалы автора, название жанра, цену и количество экземпляров книг), написанных в самых популярных жанрах, в отсортированном в алфавитном порядке по названию книг виде. Самым популярным считать жанр, общее количество экземпляров книг которого на складе максимально.
```SQL
SELECT book.title, author.name_author, genre.name_genre, book.price, book.amount
FROM book
INNER JOIN author USING(author_id)
INNER JOIN genre USING(genre_id)
WHERE genre.genre_id IN                      
       ( SELECT genre_id
        FROM book
        GROUP BY genre_id
        HAVING SUM(amount)= (
                SELECT sum(amount) AS sum_amount
                FROM book
                GROUP BY genre_id
                HAVING sum_amount
                LIMIT 1))
ORDER BY book.title;
```
## Запросы на обновление. Связанные таблицы

#### Задание 1
Для книг, которые уже есть на складе (в таблице book), но по другой цене, чем в поставке (supply),  необходимо в таблице book увеличить количество на значение, указанное в поставке,  и пересчитать цену. А в таблице  supply обнулить количество этих книг.
```SQL
UPDATE book
INNER JOIN author ON book.author_id = author.author_id
INNER JOIN supply ON book.title = supply.title 
                  and supply.author = author.name_author
SET book.amount = book.amount + supply.amount,
    book.price = (book.price*book.amount + supply.price*supply.amount )/(book.amount + supply.amount),
    supply.amount = 0
WHERE book.price != supply.price;


SELECT * FROM book;

SELECT * FROM supply;
```
#### Задание 2
Включить новых авторов в таблицу author с помощью запроса на добавление, а затем вывести все данные из таблицы author.  Новыми считаются авторы, которые есть в таблице supply, но нет в таблице author.
```SQL
INSERT INTO author(name_author)
SELECT  supply.author
FROM 
    author 
    RIGHT JOIN supply ON author.name_author = supply.author
WHERE author.name_author IS NULL;


SELECT * FROM author;
```
#### Задание 3
Удалить всех авторов, которые пишут в жанре "Поэзия". Из таблицы book удалить все книги этих авторов. В запросе для отбора авторов использовать полное название жанра, а не его id.
```SQL
DELETE FROM author
USING author
INNER JOIN book ON book.author_id = author.author_id
WHERE genre_id IN (SELECT genre_id
                    FROM genre
                    WHERE name_genre = 'Поэзия');

SELECT * FROM author;

SELECT * FROM book;
```