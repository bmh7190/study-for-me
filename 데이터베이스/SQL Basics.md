---

---
---
### Domain Tyoes in SQL
- char(n). Fixed length character string, with user-specified length n.
- varchar(n). Variable length character strings, with user-specified maximum length n. 이거보다 짧게 입력되면 그것만큼 줄어든다.

- int. Integer (a finite subset of the integers that is machine-dependent).
- smallint. Small integer (a machine-dependent subset of the integer domain type). numeric(p,d). Fixed point number, with user-specified precision of p digits, with d digits to the right of decimal point. 
- real, double precision. Floating point and double-precision floating point numbers, with machine-dependent precision.
- float(n). Floating point number, with user-specified precision of at least n digits.

---
### Create Table Constuct

create table 로 realation을 만들 수 있음
```SQL
create table r (A1 D1, A2 D2, ..., An Dn,
(integrity-constraint1),
...,
(integrity-constraintk))
```



---
### Integrity Constraints in Create Table
- primary key 
- foregin key  - references r
- not null

Example: Declare ID as the primary key for instructor
Declare dept_name as the foreign key referencing department

```SQL
create table instructor (
	ID char(5),
	name varchar(20) not null,
	dept_name varchar(20),
	salary numeric(8,2),
	primary key (ID),
	foreign key (dept_name) references department
)

```

primary key declaration on an attribute automatically ensures not null


foreign key 할 때 입력 값이 모두 있어야 함

---
### Drop and Alter Table Constructs

drop table : relation 삭제 할 때 사용 데이터도 삭제
alter table 
alter table r add A  D
A의 이름(attribute)를 가진 domain (D) 를 table r에 추가! 
새로운 칼럼을 추가하면 relation은 모든 그 값은 null 로 배정된다. 
alter table r drop A
A ( attribute)를 relation r 에서 삭제 !
fk로 이뤄질 수도 있고 , 여러 문제를 발생시킬 수 있어서 많은 데이터베이스에서 지원하지 않는다.

---
### Basic Query Structure

```SQL
select A1, A2, ..., An
from r1, r2, ..., rm
where P
```

algebra에서는 기본적으로 중복을 제거 하고 출력!
하지만 SQL에서는 허용

만약 중복을 제거하고 싶다면?
```SQL
select distinct dept_name
from instructor
```

만약 복제된걸 모두 보고 싶다면?
```SQL
select all dept_name
from instructor
```

---
### Natural Join

주어진 SQL 쿼리:

```sql
SELECT name, course_id
FROM instructor NATURAL JOIN teaches;
```

이 쿼리는 **`instructor`** 테이블과 **`teaches`** 테이블을 **자연 조인(NATURAL JOIN)** 하여, `name`과 `course_id`를 선택하는 SQL입니다.

---

$$R=πname,courseid(instructor⋈teaches)R = \pi_{name, course_id} (\text{instructor} \bowtie \text{teaches})$$

여기서:

- ⋈ : 자연 조인(NATURAL JOIN)
- π_name,courseid : `name`과 `course_id` 속성만 선택하는 **π (projection, 투영 연산)**

---

