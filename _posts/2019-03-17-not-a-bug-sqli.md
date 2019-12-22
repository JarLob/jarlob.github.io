---
layout:     post
title:      "A story of one not-a-bug SQL injection"
date:       2019-03-17 12:00:00
categories: HackAndTell SQLi
cover:      
permalink:  /en/blog/not-a-bug-sqli
---
Recently I was auditing a website for security vulnerabilities. I found few good issues, but one part of the application looked solid - all interactions with database were done through [PHP-MySQLi-Database-Class](https://github.com/ThingEngineer/PHP-MySQLi-Database-Class) - a helper library that utilized parameterized statements. I wanted to get into the database really badly, so the only option I had was to find a vulnerability in the library.

After some time I found and successfully exploited a flaw in the wrapper, but when I tried to [disclose the vulnerability](https://github.com/ThingEngineer/PHP-MySQLi-Database-Class/issues/823) to developers I faced a denial :) *(Update 2019-08-24: Fixed in 2.9.3).* First I was told it is a fault of the user of the library - lack of validation, a "primary junior mistake", then I was told it is "not a bug" and "that mysqlidb doesn't support parameterized queries", it is "not a security vulnerability" and the "library never been advertising parameterized queries".

[Parameterized queries](https://www.databasejournal.com/features/mysql/a-guide-to-mysql-prepared-statements-and-parameterized-queries.html) are also known as [prepared](http://php.net/manual/en/mysqli.quickstart.prepared-statements.php) [statements](https://en.wikipedia.org/wiki/Prepared_statement). The safe injection-free technique is also often called parameter binding. The idea is that the application supplies (binds) values separately from the SQL expression that contains just placeholders like:
```php
    $mysqli = new mysqli('host', 'username', 'password', 'databaseName');
    $stmt = $mysqli->prepare('SELECT * FROM products WHERE name = ?');
    $stmt->bind_param('s', $_POST['userInput']);
    $stmt->execute();
```
The library does advertise using it. It is written in the description of the repository - "Wrapper for a PHP MySQL class, which utilizes MySQLi and prepared statements". Also it is not just a misleading statement, it is what they [actually do](https://github.com/ThingEngineer/PHP-MySQLi-Database-Class/blob/810ffe981519f04bdf4ff734bd43cf0be3c15757/MysqliDb.php#L1598).

So how come it is vulnerable? The library provides object interface, that in the end builds the prepared statement dynamically. The Select example above would look like:
```php
    $db = new MysqliDb('host', 'username', 'password', 'databaseName');
    $db->where('name', $_POST['userInput']);
    $db->get('products');
```
The issue is that because of special "forkaround" in `where` function if `$whereValue` happens to be an array, `$operator` value is extracted from it. The value is non-parametrerized, i.e. it gets into generated SQL statement without any validation.
```php
    public function where($whereProp, $whereValue = 'DBNULL', $operator = '=', $cond = 'AND')
    {
        // forkaround for an old operation api
        if (is_array($whereValue) && ($key = key($whereValue)) != "0") {
            $operator = $key;
            $whereValue = $whereValue[$key];
        }
        if (count($this->_where) == 0) {
            $cond = '';
        }
        $this->_where[] = array($cond, $whereProp, $operator, $whereValue);
        return $this;
    }
```
A malicious user may issue a request like this for example:
```
POST /page.php HTTP/1.1
...
userInput%5B%3D%20%3F%20or%201%3D1%20--%5D=0
```
That will inject `= ? or 1=1 --` into the previous SQL statement as:  
`SELECT * FROM products WHERE name = ? or 1=1 --?`

Notice that `where` function has an additional third parameter for `$operator`. Library creators were looking for troubles (or is it a backdoor?) by reusing second parameter for two purposes: value binding and SQL expression operator. Additional input validation by a user of the library could prevent that, but it is a defense in depth, not something you expect average programmer does when working with prepared statements that are known to be safe.