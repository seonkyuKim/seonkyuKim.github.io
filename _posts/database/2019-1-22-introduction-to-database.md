---
title: "Introduction to database"
categories:
- database
---

# Database

이거 하나로 정리 끝

[https://medium.com/omarelgabrys-blog/database-introduction-part-1-4844fada1fb0](https://medium.com/omarelgabrys-blog/database-introduction-part-1-4844fada1fb0)

- 테이블을 만들 때는 독립적으로 그와 관련된, **변하지 않는**정보들만 저장한다. 예를 들어 고객 테이블에는 변하지 않는 고객들의 이름과 이메일 등만 저장을 한다.
- 물건을 주문 할 때는 따라서 변하지 않는 주문 테이블을 따로 만드는 것이다.

# 데이터베이스 만들기

## 1. 자료조사

## 2. Table, relationships 규정하기

objects를 먼저 규정해야 한다. 이는 고객, 강의, 주문, 프로젝트 등 어떠한 형태도 가능하다. 실제로는 존재하지 않는 카테고리 등도 가능하다.

- 중복된 것을 없애야 한다.
- 몇몇 object는 합치고 분리해야 한다.
- object의 attribute를 정의해야한다.

> object들은 데이터를 담고 있는 것이지, 관계를 포함하고 있지 않기에 객체지향 언어를 사용할 때는 주의해야 한다.

이후 relation을 규정한다.

- 1-M : one to many
- M-N : many to many
- 1-1 : one to one

## 3. Columns와 Data Type 규정하기

Datatype은 언어에 따라 달라진다.

- length : name, address와 같은 column은 데이터 타입 뿐 아니라 fixed length를 정해준다.
- Null : 입력을 하지 않아도 될 경우 null 을 이용한다.
- Default Value
- Other options : 이메일과 같은 column일 경우 형식이 맞는지 확인한다.

# Primary Keys

어떤 column이 primary key가 될지 정의한다. 주로 정수형을 쓴다. auto-increment를 안 사용하는 것이 좋을 때도 있다.

## Composite primary key

두 개의 column을 합쳐 하나의 primary key로 사용할 수 있다. 강의 수강 학생과 같은 다대다 관계에서 그 예를 볼 수 있다.

# Logical Design

## Weak Relation

**Partial Key**

일종의 primary key. 하지만 상위 요소가 없으면 있을 수 없기 때문에 partial key라 불림

Partial Key 와 Foreign key를 이용해 Primary key 처럼 사용함.

## 1 : 1

1. 한 쪽이 다른 쪽을 참조하는 방법
2. 두 테이블을 합친다.
3. 두 개의 Primary key를 갖고 있는 third table을 만든다.

## 1 : M

ER diagram 을 그린 후 relation을 토대로 table을 만들 때

**Many에서 1 Side를 참조하는 방향으로 만든다**

## Recursive

recursive는 1:M일 수도, M:M일 수도, 아니면 다른 형태일 수 있다. 그동안 하던 것과 똑같은 방법으로 하면 된다. 예를 들어 Employee와 Supervision과 같이 1:M의 관계가 있을 경우, Supervision이 1 side, employee가 M side다. 이때, employee table의 column에 supervision란을 추가하여 employee 를 참조하면 된다. 없을 경우 null값을 준다.

즉, 이전과 같은 방법으로 진행하되 같은 테이블에 column을 추가하여 같은 테이블을 참조하는 방식이다.

## Multi-valued attribute

예를 들어 department가 여러 개의 location을 가질 수 있다. 이는 1:M 관계로 department와 location 두 개의 column이 있는 테이블을 새로 만들어준다. 이 두 개의 column을 primary key로 사용한다.

# Normalization

## First Normal Form

First normal form requires that every column should have one, and only one value; there is no multi-valued attribute, and there shouldn’t be repeating groups of data.

## Second Normal Form

The second normal form requires that all of the non-primary columns have to be dependent on the entire (composite) primary key.

회원이 이메일 외에도 다른 attribute를 가질 경우, 그 값들은 third table로 만들지 않는 이상 중복된 값을 가진다. 그래서 third table을 만들어 non-primary column들이 오직 primary key에만 의존적이게 한다.

만약 한 회원이 여러 개의 이메일을 가질 경우, 위에 적힌 1:M을 사용할 수도 있지만, 확장성을 고려하여 새로운 third table을 만든다. 두 개의 column이 email과 ID이고 이 두개의 column을 primary key로 이용한다.

## Third Normal Form

한 column이 primary key가 아닌 다른 값에 의존적일 때, 그 값들을 새로운 table로 만들어준다. 이는 다른 이가 하나의 값을 변경했을 경우, 그 relation이 무너지는 것을 방지하기 위함이다. 예를 들어 course이름, 강의실 번호, 수용 인원이 있을 경우, 강의실 번호와 수용 인원은 상호관계가 있기 때문에 강의실 table을 따로 설정해주는 것이다.

참고 : [https://3months.tistory.com/193](https://3months.tistory.com/193)

## Denormalization

한 회원이 이메일을 여러 개 갖고 있을 경우, 만약 이메일의 개수를 2개로 제한한다면, email1 column과 email2 column을 설정하는 것으로 표를 만들 수 있다. 이럴 경우 더 쉽고 수월하게 작업할 수 있다.

# SQL(Structured Query Language)

The Structured Query Language (SQL) — pronounced S-Q-L — is used for creating, querying, updating and manipulating data in relational databases

# DML(Data Manipulation Language)

The data manipulation language is used to create, read, update, and delete data from a table.

[CRUD](https://www.notion.so/41756fc2e0c745b1a84a041cae904d14)

다양한 명령어들 : [https://medium.com/omarelgabrys-blog/database-structured-query-language-part-8-230a1808ec96](https://medium.com/omarelgabrys-blog/database-structured-query-language-part-8-230a1808ec96)

# Indexing

Use index to find the tuple that you want. You can use primary key with the reference to the object in binary tree. Or you can use other columns that you use often. But be careful because when you modify your datas, indexes could be messed up.

(part 9) still writing

# Transaction

(part 9) still writing

[비지니스 룰](https://www.notion.so/07c094190f9c4659a30a44e6d715080c)

# Single Table Inheritance

두 개의 테이블이 같은 데이터 폼을 갖고 있지만, 서로 다른 기능을 갖고 있을 때

[https://www.driftingruby.com/episodes/single-table-inheritance](https://www.driftingruby.com/episodes/single-table-inheritance)

ex) emergency contact - 전화번호가 있어야 함, friends contact - 생일이 있어야 함
