---

---

### Domain Types in SQL
---

| 데이터 타입             | 설명                                         |
| ------------------ | ------------------------------------------ |
| `CHAR(n)`          | 고정 길이 문자열 (n보다 짧으면 공백 추가)                  |
| `VARCHAR(n)`       | 가변 길이 문자열 (짧으면 짧게 저장)                      |
| `INT`              | 일반 정수 (기계 의존 크기)                           |
| `SMALLINT`         | 작은 정수 (메모리 절약)                             |
| `NUMERIC(p,d)`     | 고정 소수점 숫자 (`p` = 전체 자릿수, `d` = 소수점 이하 자릿수) |
| `REAL`             | 부동 소수점 숫자 (낮은 정밀도)                         |
| `DOUBLE PRECISION` | 부동 소수점 숫자 (더 높은 정밀도)                       |
| `FLOAT(n)`         | 최소 `n`자리 유효 숫자의 부동 소수점                     |

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

