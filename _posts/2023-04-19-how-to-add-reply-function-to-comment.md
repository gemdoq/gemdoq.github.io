---
layout: single
title: "댓글에 댓글을 달 수 있는 방법"
date: 2023-04-19 08:58:41 +0900
categories: java
tags: [Comment, Reply, Soft Delete]
typora-root-url: ../
---

## 댓글에 댓글을 달 수 있는 방법
> - 기획 고민
> - 코드

<br>

## 기획 고민

### 댓글(comment) 기능 추가 하기

프로젝트 기획을 할 때 댓글에 대댓글을 달 수 있게끔 구상

문제는 댓글에 대댓글이 있을 경우, 게시판에 달아놓은 댓글을 `부모 댓글`이라 하고, 부모 댓글에 속한 대댓글을 `자식 댓글`이라고 한다면, 만약 `부모 댓글`이 삭제되었을 때, 부모 댓글과 자식 댓글의 처리를 어떻게 해야 하는가

부모 댓글과 함께 그에 속한 자식 댓글도 같이 노출되지 않게끔 삭제하는 방법이 있음

하지만 그렇게 하면 자식 댓글에 담긴 `필요성이 남아있는 유용한 정보`도 의도치 않게 삭제될 수 있음

그래서 정보 게시의 자유를 보장하기 위해 부모 댓글이 삭제되어도 삭제된 부모 댓글은 "이 댓글은 삭제되었습니다."라는 메시지를 보여주게끔 전환되고 자식 댓글들은 그대로 남아있게끔 구현하기로 결정

또한 특정 부모 댓글을 삭제한다고 하더라도, 특정 부모 댓글의 하위 댓글까지 다 삭제하지 않는 이상 그대로 남아서 노출됨

![commentreply](/images/2023-04-19-how-to-add-reply-function-to-comment/commentreply.png){: width="560"}

예를 들어 댓글 4(삭제)와 댓글 7(삭제)는 각각 댓글 5와 6의 부모 댓글, 댓글 8의 부모 댓글이기 때문에 삭제를 했더라도 그대로 남아있게 됨

만약 댓글 4의 자식 댓글인 댓글 5와 6을 모두 삭제하면 그때 댓글 4가 완전히 삭제됨 

### 부모 댓글을 삭제하면 내용만 변경 후 그대로 유지

다음은 댓글 저장 시 필요한 내용 

1. 댓글id : PK
2. 부모 댓글id : FK
3. 게시글id
4. 작성자id
5. 댓글 내용
6. 삭제 여부

댓글 삭제 시 댓글ID만 있으면 삭제 가능

삭제 방식은 데이터 레코드의 완전 삭제가 아닌, Soft-Delete 방식

#### Soft delete

일반적으로 소프트웨어 개발에서 데이터의 삭제는 물리적인 삭제가 아니라 논리적인 삭제 의미

Soft delete는 데이터베이스 레코드를 삭제하지 않고 삭제된 것처럼 처리하는 기술

##### 장점

1. 삭제된 데이터를 복원 가능
2. 데이터 무결성 유지
3. 데이터베이스의 성능 향상 가능

##### 단점

1. 삭제된 데이터가 여전히 데이터베이스에 남아있어서, 데이터베이스가 커질 수 있음
2. 삭제된 데이터를 복원하려면 복원 프로세스를 수행해야 함

<br>

## 코드

### entity.comment.Comment 엔티티

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Slf4j
public class Comment extends EntityDate {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    @Lob
    private String content;

    @Column(nullable = false)
    private boolean deleted; // 1

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    @OnDelete(action = OnDeleteAction.CASCADE)
    private Member member; // 2

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    @OnDelete(action = OnDeleteAction.CASCADE)
    private Post post; // 3

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    @OnDelete(action = OnDeleteAction.CASCADE)
    private Comment parent; // 4

    @OneToMany(mappedBy = "parent")
    private List<Comment> children = new ArrayList<>(); // 5

    public Comment(String content, Member member, Post post, Comment parent) {
        this.content = content;
        this.member = member;
        this.post = post;
        this.parent = parent;
        this.deleted = false;
    }

    public Optional<Comment> findDeletableComment() { // 6
        return hasChildren() ? Optional.empty() : Optional.of(findDeletableCommentByParent());
    }

    public void delete() { // 7
        this.deleted = true;
    }

    private Comment findDeletableCommentByParent() { // 8
        return isDeletableParent() ? getParent().findDeletableCommentByParent() : this;
    }

    private boolean hasChildren() { // 9
        return getChildren().size() != 0;
    }

    private boolean isDeletableParent() { // 10
        return getParent() != null && getParent().isDeleted() && getParent().getChildren().size() == 1;
    }
}
```

(1) 삭제는 했더라도 아직 하위 댓글이 있어서 실제로는 남겨둬야한다면, deleted는 true가 됨

(2~4) @OnDelete(action=onDeleteAction.CASCADE)를 설정해놨기 때문에, Member나 Category, 상위 Comment가 제거된다면, 연쇄적으로 현재 댓글도 제거

(5) 각 댓글의 하위 댓글들을 참조할 수 있도록, @OneToMany 매핑

(6) 현재 댓글을 기준으로, 자식의 유무를 통해 실제로 삭제가 가능한 댓글인지 확인

만약 이 메소드의 결과로 찾아낸 댓글이 없다면, 하위 댓글이 아직 있다는 것을 의미하므로 삭제해서는 안됨

실제 데이터를 제거하는 것이 아니라, 현재 댓글은 delete() 메소드로 삭제 마킹

현재 댓글에 하위 댓글이 있으면, 즉시 제거하면 안되는 댓글이므로 비어있는 Optional이 반환될 것이고, 현재 댓글에 하위 댓글이 없으면, 실제로 제거해도 되는 댓글을 찾기 위해 상위 댓글로 거슬러 올라가며 검사할 것

(7) 실제로 삭제하는 것이 아니라 삭제 마킹만 해야되는 상황이라면, deleted를 true로 설정

(8) 상위 댓글로 거슬러 올라가면서, 실제로 제거해도 되는 댓글을 찾음

상위 댓글이 실제로 제거해도 되는 댓글이라면, 다시 상위 댓글로 거슬러올라가면서, 삭제 가능한 지점을 찾음

(9) 하위 댓글이 있는지 판별

(10) 현재 댓글의 상위 댓글이 제거해도 되는 것인지 판별

부모가 있고, 이미 삭제 처리를 받았었고, 자식의 개수가 1이라면, 제거

자식의 개수가 1이라는 것은, 지금 삭제 요청을 받은 현재의 댓글 외에, 다른 하위 댓글들은 없는 상황을 의미

여기서 주의할 점은, parent나 children을 참조할 때, getter 메소드를 이용

jpa에서는 엔티티를 조회할 때, 작성한 엔티티를 상속받은 프록시를 만들어서 사용

만약 this.parent나 this.children으로 참조한다면, 해당 값들은 실제로 데이터베이스에 저장되어있음에도 불구하고, fetch 전략을 LAZY로 설정해둔 까닭에 실제 데이터들을 불러올 수 없음

LAZY 전략으로 설정되어서 아직 조회되지 않은 데이터들을 적절한 시기에 불러올 수 있도록, getChildren()과 getParent()와 같은 형태로 작성

<br>
