### Vector DB란..?


view level
명시적으로 안 들어갈 수도 있음!

#### Schema
- DB 가 파일 시스템 위에 있다.
	- 기존에는 파일 시스템 - Application이라면  파일 시스템 - DB - Application 
- 데이터베이스 틀

---
#### Keys of Relations
key: a subset of the attributes associated with a tuple
- same values X
- underlining the key attributes

---
### SQL
user 나 application이 DB 사이에서 정보를 저장하고, 관리하는 언어
- non-procedural language ( 비 절차 언어 ) -> 선언만 해준다.
- Data Definition Language(DDL) -> declare database schema
- Data Manipulation Language(DML) -> query and modify the database / query language

##### select
list the attributes desired in the result of a query
- SQL names are case insensitive
- upper-case 

##### from
lists the relations involved in the query

**cartesian product ?** 
generates every possible pair, with all attributes from both relations.
- not very useful
- useful combined with where-claues condition
##### where
specifies conditions that the result must satisfy


