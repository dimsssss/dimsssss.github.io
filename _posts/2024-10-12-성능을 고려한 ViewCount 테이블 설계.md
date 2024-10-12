---
title: "성능을 고려한 viewCount 테이블 설계"
date: 2024-10-12T13:23:48+09:00
categories: 
    - 데이터베이스
tags:
    - mysql
    - lock
---

데이터베이스의 테이블을 설계할 때 테이블들의 관계가 ORM에서 설정한 관계와 다른 경우가 있다. 나는 유저 리스트에 개인 프로필 누적 조회 수를 보여주는 기능을 만들 때 경험을 하였다. 예를 들어 유저 리스트를 조회하는 기능에 유저 별로 프로필을 방문한 사람들의 누적 count를 반드시 포함해야 한다고 하면 적혀있는 그대로 다음처럼 설계할 수 있다.

| id  | name | viewCount | createdAt  |
| --- | ---- | --------- | ---------- |
| 1   | 유저1  | 10        | 2024-10-12 |

이렇게 설계하면 장점은 직관적이고 필요한 모든 데이터가 한 테이블에 있기 때문에 조회도 간단하다. 그런데 읽기와 수정 쿼리가 꾸준히 증가하면 병목 현상이 발생할 수 있다. 왜냐하면 `update` 문은 잠금을 사용하기 때문이다.

프로필 조회는 충분히 많이 일어날 수 있는 기능이기 때문에 쿼리가 자주 발생하게 되고 잠금 때문에 수정 쿼리가 늦어지게 된다. 이것을 해결하기 위해서 update하는 쿼리를 줄이기 위해서 조회를 할 때마다 바로 조회 수를 반영하지 않고 특정 주기로 viewcount를 업데이트하는 프로세스로 수정하였다. 이를 위해 view count를 관리하는 테이블을 하나 만들게 된다. user 테이블에 있는 view_count 컬럼 중복을 줄이기 위해서 최종 view_count에 해당하는 테이블을 만들고 view_log를 쌓는 테이블을 만든다

그러면 프로필을 조회할 때마다 view logs에 행이 쌓이고 주기적으로 count한 데이터(count 후에는 관련 데이터를 지운다)를 view count 테이블에 수정을 한다. 그러면 누적 조회수는 view count에서 찾으면 된다. 
![](https://i.imgur.com/8J2ZEOV.png)

이렇게 만들고 보니 굳이 테이블 3개가 필요한가? 그리고 유저와 viewcount를 한번에 조회를 해야하는데 그러러면 join을 사용해야한다. 그럴바에는 viewcount를 없애고 user에 컬럼으로 추가하는게 낫다고 생각을 했다

![](https://i.imgur.com/feAorMt.png)

최종적으로 나온 결과물은 유저와 조회수 테이블은 1:N으로 만들어졌다. 그러나 실제 ORM에서 User와 ViewCount 관계는 1:1이다.  유저 리스트를 조회하는 기능에서 유저는 viewCount를 반드시 포함해야한다고 했기 때문에. 그러다보니 user와 viewcount JPA의 연관 관계를 1:1로 만들고 조회를 하였는데 그때부터 테이블과 ORM 연관 관계 불일치로 에러와 잘못된 데이터(ORM은 정확하게 동작하지만 내 입장에서는 잘못된 데이터)를 조회하게 되었다

서브쿼리를 이용하거나 연관 관계를 역으로 설정해서 조회를 시도해봤지만 다 실패를 하였고 억지로 만들어봐도 코드가 불필요하게 복잡해지고 예상치 못한 동작을 할 확률이 높았기 때문에 연관 관계를 설정해서 해결하면 안된다는 생각이 들었다. 그래서 테이블에 다시 view_count 컬럼을 추가하고 연관 관계는 제거했다. 최종 테이블과 JPA Entity 설정은 다음으로 수정되었다

![](https://i.imgur.com/A95LCPH.png)


```java
package com.dimsss.toy.freelancer;  
  
import jakarta.persistence.*;  
import lombok.Getter;  
  
import java.time.LocalDateTime;  
  
@Getter  
@Entity  
@Table(name = "users")  
public class UserEntity {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Getter  
    private int id;  
    private String name;  
    private int viewCount;  
    private LocalDateTime createdAt;  
    private LocalDateTime updatedAt;  
}
```

잠금의 측면으로 봤을 때 유저의 프로필을 조회하면 `INSERT`로 view count가 쌓이게 되어 잠금과 관련된 지연 이슈는 없다. 또한 `UPDATE`하려는 행이 유일하다면 `Gap Lock`[^1], `Next-Key Lock`[^2] 이 없기 때문에 update할 때도 유리하다[^3] (그러나 transaction isolation level이 3단계 이상이면 지연이 발생할 수 있다)

결과적으로 `SELECT` , `UPDATE`가 발생하더라도 데이터베이스에서는 잠금에 대한 지연 없이 쿼리가 동작할 수 있다.

## 각주

[^1]: https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-gap-locks
[^2]: https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-next-key-locks
[^3]: Gap locking is not needed for statements that lock rows using a unique index to search for a unique row. (This does not include the case that the search condition includes only some columns of a multiple-column unique index; in that case, gap locking does occur.) For example, if the `id` column has a unique index, the following statement uses only an index-record lock for the row having `id` value 100 and it does not matter whether other sessions insert rows in the preceding gap:


```sql
SELECT * FROM child WHERE id = 100;
```

