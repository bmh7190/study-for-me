---

---
---
attribute

A

domian 이란?
attribute에 들어갈 수 있는 모든 값의 집합 
각 도메인은 atomic해야 한다. -> 분할 되지 않아야한다.
null 은 모든 domain에 들어갈 수 있다.
하지만 null은 여러가지 문제를 발생시킸을 수 있으므로 없는게 좋음!



relation r은 모든 D 의 카테시안 곱 중 서브셋이다.


---
### Schema and Instance
Database schema - logical design of the database
Database instance - A snapshot of the data in the database  at a given instant in time

relation schema?


---
### Keys
superkey 
tuple 간 구분만 할 수 있다면 super key가 될 수 있다.
유일성만 만족하면 됨.

candidate key
superkey 중 최소성을 만족하면 candidate key가 되는 듯
즉, 유일성도 만족하면서 최소성을 만족해야 한다. 

primary key
candidate key 중 하나로 선택된 키( 단 하나만 선택 됨)
null 값이 없다.

foreign key
referenced relation의 pk가 fk 가 될 수 있음/ 되어야 함
제약 조건이 생긴다. 
예를 들어 fk로 지정한 대상이 없다면? 쿼리가 수행되지 않겠죠?


---
### Relation Query Languages
