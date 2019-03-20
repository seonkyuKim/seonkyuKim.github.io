---
title: "maria DB 기초: 설치부터 유저 생성까지"
categories: tech
tags:
- mariaDB
author_profile: true
---

### mariaDB 설치하기(mac 기준)

    brew install mariadb

### mariaDB 서버 실행하기

    mysql.server start

### mariaDB 로그인하기

    mysql -u root

### 데이터베이스 확인하기

    SHOW databases;

### 데이터베이스 만들기

    CREATE DATABASE bookstoredb;

### 데이터베이스 사용하기

    USE bookstoredb;

## 테이블 생성하기

    CREATE TABLE authorstbl;

다음과 같은 에러:

    ERROR 1113 (42000): A table must have at least 1 column

따라서 데이터베이스 칼럼과 같이 생성:

    CREATE TABLE authorstbl (
        -> AuthorID INT NOT NULL AUTO_INCREMENT,
        -> AuthorName VARCHAR(100),
        -> PRIMARY KEY(AuthorID)
        -> );

    CREATE TABLE bookstbl (
        -> BookID INT NOT NULL AUTO_INCREMENT,
        -> BookName VARCHAR(100) NOT NULL,
        -> AuthorID INT NOT NULL,
        -> BookPrice DECIMAL(6,2) NOT NULL,
        -> BookLastUpdated TIMESTAMP,
        -> BookIsAvailable BOOLEAN,
        -> PRIMARY KEY (BookID), 
        -> FOREIGN KEY (AuthorID) REFERENCES authorstbl(AuthorID)
        -> );

[여기](http://sqlmvp.kr/220340062504)에서 다양한 데이터 타입을 살펴볼 수 있음.

## 레코드 삽입하기


**특정 칼럼만 삽입**

```
INSERT INTO authorstbl (AuthorName) 
    -> VALUES ('Agatha Christie'), ('Stephen King'), ('John Clancy');
```

**전체 칼럼 삽입**

    INSERT INTO bookstbl (BookName, AuthorID, BookPrice, BookIsAvailable) VALUES 
        -> ('And Then There were None', 1, 14.95, 1),
        -> ('The Man in the Brown Suit', 1, 23.99, 1),
        -> ('The Stand', 2, 35.99, 1),
        -> ('The Green Mile', 2, 23.99, 1),
        -> ('Jack Ryan - The Preque', 3, 24.99, 1);

## 레코드 확인하기

**모든 레코드**

    SELECT * FROM authorstbl;

**특정 레코드**

    SELECT * FROM authorstbl WHERE AuthorName='Agatha Christie';

**특정 칼럼**

    SELECT authorstbl.AuthorName FROM authorstbl;

### 레코드 수정하기

    UPDATE bookstbl SET BookPrice=14.99 WHERE AuthorID=2;

### 레코드 삭제하기

    DELETE FROM bookstbl WHERE BookID=4;

## Advanced Select Statement

    SELECT CONCAT(bookstbl.BookName, ' (', authorstbl.AuthorName, ')') AS Description,
        -> bookstbl.BookPrice FROM authorstbl 
        -> JOIN bookstbl ON authorstbl.AuthorID = bookstbl.AuthorID;

Concat은 ( )안의 항목들을 이어서 보여줌.

JOIN에 대한 자세한 설명은 [여기](https://postitforhooney.tistory.com/entry/DBMARIADB-SQL-%EC%98%88%EC%A0%9C%EB%A5%BC-%ED%86%B5%ED%95%9C-JOIN%EC%9D%98-%EC%A2%85%EB%A5%98-%ED%8C%8C%EC%95%85)를 참조.

## User 생성하기

Note: 유저를 생성하기 전, 원하는 데이터베이스에 접근해 있는지 확인할 것!

비밀번호를 통해 유저 확인

    CREATE USER bookstoredbadmin@localhost IDENTIFIED BY 'PASSWORD';

권한 부여

    GRANT ALL PRIVILEGES ON bookstoredb.* TO bookstoredbadmin@localhost;

새로고침

    FLUSH PRIVILEGES;


**참고 :** 

- 유튜브 강의 : [https://www.youtube.com/watch?v=RZa3VYVohXg](https://www.youtube.com/watch?v=RZa3VYVohXg)

- [Mac] 마리아DB!! mariaDB - 세팅하기(초보는 힘드렁)출처: [https://devuryu.tistory.com/41](https://devuryu.tistory.com/41)
