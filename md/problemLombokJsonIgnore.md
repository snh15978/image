# __Lombok 사용 시 @JsonIgnore 동작안하는 문제 해결__


`Lombok 1.18.10`을 사용하여 공부를 하는 중 `@JsonIgnore`가 동작안하는 이슈가 발생했었고, 이를 해결한 방법을 찾아봤습니다.

## __1. Problem__
---

유저 정보를 전달하는 API에서 `password` 필드가 포함되어 있어, 이를 제외하기 위해 `@JsonIgnore` 애노테이션을 사용했습니다.

```java
@Entity
@Getter @Setter
public class 

    ...

    @JsonIgnore
    private String password;

    ...
```

하지만 `@JsonIgnore` 애노테이션이 동작하지 않아 Json 결과에 계속 출력되는 문제가 발생했었습니다.

## __2. Solution__
---

이를 해결하기 위해 열심히 검색한 결과 `@JsonProperty` 애노테이션으로 해결하는 방법을 찾았습니다.

`@JsonProperty`애노테이션을 사용하면서 `access = JsonProperty.Access.WRITE_ONLY`값을 설정하면 해당 필드는 오직 쓰려는 경우(deserialize)에만 접근이 허용됩니다.

```java
@Entity
@Getter @Setter
public class 

    ...

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String password;

    ...
```

이렇게 수정하면 사용자를 생성하기 위한 요청 본문을 처리할 때는 사용되고, 응답결과를 생성할 때는 해당 필드는 제외되서 응답 본문에 표시되지 않게 됩니다.

이 문제를 해결하기 위해 `import`를 바꾸거나, 특정 필드만 `get` 메소드를 생성해서 `@JsonIgnore`를 사용하는 방법도 있었습니다. 이렇게하면 참고하는 라이브러리가 꼬이고 코드가 복잡해 질 수 있다고 생각하였습니다. 그래서 위의 방법을 선택해서 적용하였습니다.

또 다른 좋은 방법이 있다면 댓글 부탁드립니다.

## __3. Reference__
---

- https://stackoverflow.com/questions/24466464/jsonignore-with-getter-annotation
- https://src-bin.com/ko/q/1755420
- https://wickso.me/java/lombok-jsonignore-issue/