### 4.2.1 Spring Data keyword usage

#### Keyword usage in Spring Data JPA and generated JPQL
| Keyworld           | Example                                                      | Generated JPQL                                                     |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------------ |
| Is, Equals         | findByUsername<br>findByUsernameIs<br>findByUsernameEquals   | . . . where e.username = ?1                                        |
| And                | findByUsernameAndRegistrationDate                            | . . . where e.username = ?1<br>and e.registrationDate = ?2         |
| Or                 | findByUsernameOrRegistrationDate                             | . . . where e.username = ?1 or e.registrationDate = ?2             |
| LessThan           | findByRegistrationDateLessThan                               | . . . where<br>e.registrationDate < ?1                             |
| LessThanEqual      | findByRegistrationDateLessThanEqual                          | . . . where<br>e.registrationDate <= ?1                            |
| GreaterThan        | findByRegistrationDateGreaterThan                            | . . . where<br>e.registrationDate > ?1                             |
| GreaterThanEqual   | findByRegistrationDateGreaterThanEqual                       | . . . where<br>e.registrationDate >= ?1                            |
| Between            | findByRegistrationDateBetween                                | . . . where<br>e.registrationDatebetween<br>?1 and ?2              |
| OrderBy            | findByRegistrationDateOrderByUsernameDesc                    | . . . where<br>e.registrationDate = ?1<br>order by e.username desc |
| Like               | findByUsernameLike                                           | . . . where e.username like<br>?                                   |
| NotLike            | findByUsernameNotLike                                        | . . . where e.username not<br>like ?1                              |
| Before             | findByRegistrationDateBefore                                 | . . . where<br>e.registrationDate < ?1                             |
| After              | findByRegistrationDateAfter                                  | . . . where<br>e.registrationDate > ?1                             |
| Null, IsNull       | findByRegistrationDate(Is)Null                               | . . . where<br>e.registrationDate is null                          |
| NotNull, IsNotNull | findByRegistrationDate(Is)NotNull                            | . . . where<br>e.registrationDate is not<br>null                   |
| Not                | findByUsernameNot                                            | . . . where e.username <> ?1                                       |
| In                 | findByRegistrationDateIn<br>(Collection<LocalDate> dates)    | . . . where<br>e.registrationDate in ?1                            |
| NotIn              | findByRegistrationDateNotIn<br>(Collection<LocalDate> dates) | . . . where<br>e.registrationDate not in<br>?                      |
| True               | findByActiveTrue                                             | . . . where e.active = true                                        |
| False              | findByActiveFalse                                            | . . . where e.active = false                                       |
| StartingWith       | findByUsernameStartingWith                                   | . . . where e.username like<br>?1%                                 |
| EndingWith         | findByUsernameEndingWith                                     | . . . where e.username like<br>%?1                                 |
| Containing         | findByUsernameContaining                                     | . . . where e.username like<br>%?1%                                |
| IgnoreCase         | findByUsernameIgnoreCase                                     | . . . where<br>UPPER(e.username) =<br>UPPER(?1)                    |

#### Keyword usage in Spring Data JDBC and resulting conditions

| Keyworld           | Example                                                                                           | Condition                                         |
| ------------------ | ------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Is, Equals         | findByUsername(String name)<br>findByUsernameIs(String name)<br>findByUsernameEquals(String name) | username = name                                   |
| And                | findByUsernameAndRegistrationDate<br>(String name, LocalDate date)                                | username = name and<br>registrationDate = date    |
| Or                 | findByUsernameOrRegistrationDate<br>(String name, LocalDate date)                                 | username = name or<br>registrationDate = name     |
| LessThan           | findByRegistrationDateLessThan<br>(LocalDate date)                                                | registrationDate < date                           |
| LessThanEqual      | findByRegistrationDateLessThanEqual<br>(LocalDate date)                                           | registrationDate <= date                          |
| GreaterThan        | findByRegistrationDateLessThanEqual<br>(LocalDate date)                                           | registrationDate <= date                          |
| GreaterThanEqual   | findByRegistrationDateGreaterThan<br>(LocalDate date)                                             | registrationDate > date                           |
| Between            | findByRegistrationDateBetween<br>(LocalDate from, LocalDate to)                                   | registrationDate between<br>from and to           |
| OrderBy            | findByRegistrationDateOrderByUsernameDesc<br>(LocalDate date)                                     | registrationDate = date<br>order by username desc |
| Like               | findByUsernameLike(String name)                                                                   | username like name                                |
| NotLike            | findByUsernameNotLike(String name)                                                                | username not like name                            |
| Before             | findByRegistrationDateBefore<br>(LocalDate date)                                                  | registrationDate < date                           |
| After              | findByRegistrationDateAfter<br>(LocalDate date)                                                   | registrationDate > date                           |
| Null, IsNull       | findByRegistrationDate(Is)Null()                                                                  | registrationDate is null                          |
| NotNull, IsNotNull | findByRegistrationDate(Is)NotNull()                                                               | registrationDate is not null                      |
| Not                | findByUsernameNot(String name)                                                                    | username <> name                                  |
| In                 | findByRegistrationDateIn<br>(Collection<LocalDate> dates)                                         | registrationDate in (date1,<br>. . . dateN)       |
| NotIn              | findByRegistrationDateNotIn<br>(Collection<LocalDate> dates)                                      | registrationDate not in<br>(date1, . . . dateN)   |
| True, IsTrue       | findByActiveTrue()                                                                                | active is true                                    |
| False, IsFalse     | findByActiveFalse()                                                                               | active is false                                   |
| StartingWith       | findByUsernameStartingWith(String name)                                                           | username like name%                               |
| EndingWith         | findByUsernameEndingWith(String name)                                                             | username like %name                               |
| Containing         | findByUsernameContaining(String name)                                                             | username like %name%                              |
| IgnoreCase         | findByUsernameIgnoreCase(String name)                                                             | UPPER(username) =<br>UPPER(name)                  |

#### Keyword usage in Spring Data MongoDB and resulting conditions

| Keyworld           | Example                                                                                           | Condition                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Is, Equals         | findByUsername(String name)<br>findByUsernameIs(String name)<br>findByUsernameEquals(String name) | {"username":"name"}                                                                                  |
| And                | findByUsernameAndRegistrationDate<br>(String name, LocalDate date)                                | {"username":"username",<br>"email":"email"}                                                          |
| Or                 | findByUsernameOrRegistrationDate<br>(String name, LocalDate date)                                 | { "$or" : [{ "username" :<br>"username"}, { "email" :<br>"email"}]}                                  |
| LessThan           | findByRegistrationDateLessThan<br>(LocalDate date)                                                | { "registrationDate" : {<br>"$lt" : { "$date" : "date"}}}                                            |
| LessThanEqual      | findByRegistrationDateLessThanEqual<br>(LocalDate date)                                           | { "registrationDate" : {<br>"$lte" : { "$date" :<br>"date"}}}                                        |
| GreaterThan        | findByRegistrationDateLessThanEqual<br>(LocalDate date)                                           | { "registrationDate" : {<br>"$gt" : { "$date" : "date"}}}                                            |
| GreaterThanEqual   | findByRegistrationDateGreaterThan<br>(LocalDate date)                                             | { "registrationDate" : {<br>"$gte" : { "$date" :<br>"date"}}}                                        |
| Between            | findByRegistrationDateBetween<br>(LocalDate from, LocalDate to)                                   | "registrationDate" : { "$gte" :<br>{ "$date" : "date"}, "$lte" : {<br>"$date" : "date"}}}            |
| OrderBy            | findByRegistrationDateOrderByUsernameDesc<br>(LocalDate date)                                     | "registrationDate" : { "$date"<br>: "date"}}                                                         |
| Like               | findByUsernameLike(String name)                                                                   | { "username" : {<br>"$regularExpression" : {<br>"pattern" : "name", "options" :<br>""}}}             |
| NotLike            | findByUsernameNotLike(String name)                                                                | { "username" : { "$not" : {<br>"$regularExpression" : {<br>"pattern" : "name", "options" :<br>""}}}} |
| Before             | findByRegistrationDateBefore<br>(LocalDate date)                                                  | { "registrationDate" : { "$lt" :<br>{ "$date" : "date"}}}                                            |
| After              | findByRegistrationDateAfter<br>(LocalDate date)                                                   | { "registrationDate" : { "$gt" :<br>{ "$date" : "date"}}}                                            |
| Null, IsNull       | findByRegistrationDate(Is)Null()                                                                  | { "registrationDate" : null}                                                                         |
| NotNull, IsNotNull | findByRegistrationDate(Is)NotNull()                                                               | { "registrationDate" : { "$ne" :<br>null}}                                                           |
| Not                | findByUsernameNot(String name)                                                                    | { "username" : { "$ne" : "name"}}                                                                    |
| In                 | findByRegistrationDateIn<br>(Collection<LocalDate> dates)                                         | "registrationDate" : { "$in" :<br>[{ "$date" : "date1"}, . {<br>"$date" : "daten"}]}}                |
| NotIn              | findByRegistrationDateNotIn<br>(Collection<LocalDate> dates)                                      | "registrationDate" : { "$nin" :<br>[{ "$date" : "date1"}, . . . {<br>"$date" : "daten"}]}}           |
| True, IsTrue       | findByActiveTrue()                                                                                | { "active" : true}                                                                                   |
| False, IsFalse     | findByActiveFalse()                                                                               | { "active" : false}                                                                                  |
| StartingWith       | findByUsernameStartingWith(String name)                                                           | { "username" : {<br>"$regularExpression" : {<br>"pattern" : "^name", "options"<br>: ""}}}            |
| EndingWith         | findByUsernameEndingWith(String name)                                                             | { "username" : {<br>"$regularExpression" : {<br>"pattern" : "name$", "options"<br>: ""}}}            |
| Containing         | findByUsernameContaining(String name)                                                             | { "pattern" : ".\*name.\*",<br>"options" : ""}}}                                                     |
| IgnoreCase         | findByUsernameIgnoreCase(String name)                                                             | { "username" : {<br>"$regularExpression" : {<br>"pattern":"^name$","options"<br>: "i"}}}             |


