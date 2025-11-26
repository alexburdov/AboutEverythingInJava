#### Проекции в Spring Data

DTO-объект можно формировать не только в контроллере или сервисе, но и возвращать сразу из базы. Тогда они называются проекциями. Роль  проекции может играть как класс, так и интерфейс.
Исходный DTO
```java
@Entity
@Data
@NoArgsConstructor
public class Post {
    @Id
    @GeneratedValue(generator = "sequence")
    private Long id;

    private String title;

    private String text;

    @ManyToOne
    private User user;
}
```

```java
@Data
@Entity
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(generator = "sequence")
    private Long id;

    private String email;

    private String nickname;

    private String password;

    private String role="ROLE_USER";

    private boolean locked=false;

}
```

#### Interface-based проекции
**Закрытая проекция** — в ней нет никаких полей кроме тех, что есть в сущностях Post.

```java
public interface PostView {
    long getId();
    String getTitle();
    UserView getUser();
}
```

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    List<PostView> findByTitle(String title);
}
```

Можно задать и вычисляемое поле, такая проекцию будет считаться **открытой**:
```java
public interface UserView {

    String getNickname();

    @Value("#{target.email + ' ' + target.password}")
    String getInfo();
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {

    UserView findByNickname(String nickname);
    
}
```



#### Class-based проекции

Ограничение ее в том, что вложенный объект в ней прописывать смысла нет — он не будет извлечен. Сделаем такую проекцию UserDto:

```java
public class UserDto {
    private Long id;
    private String nickname;

    public UserDto(long id, String nickname) {
        this.id = id;
        this.nickname = nickname;
    }
    //getters
}
```

```java
public interface UserRepository extends JpaRepository<User, Long> {

    UserView findByNickname(String nickname);

    List<UserDto> findByEmail(String email);

}
```

#### Динамические проекции
Возвращаемый тип можно сделать общим (Generic) и возвращать что угодно: хоть Interface-based проекцию, хоть Class-based проекцию, хоть просто сущность. Сделаем это.
Добавим метод findByNickname(). Для простоты пусть он возвращает не список, а просто объект (хотя можно и список):
```java
public interface UserRepository extends JpaRepository<User, Long> {

    UserView findByNickname(String nickname);

    List<UserDto> findByEmail(String email);

    <T> T findByNickname(String nickname, Class<T> type);
}
```
