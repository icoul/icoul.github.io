---
title: "JPA에서 순환참조 무한루프를 해결하는 방법"
date: 2020-05-17 18:00:00 -0400
categories: JPA, 순환참조 
---

JPA에서의 순환참조
-----------------------------
JPA는 ORM이기 때문에 RDB를 관리하는데 있어서 양방향 참조를 필요로 한다.  
이러한 참조는 필수적인 것은 아니지만 Entity는 데이터의 구조를 보여주는 역할도 하기 때문에 명시적으로 표시해주는 것이 좋은데 이 과정에서 순환참조의 무한루프에 빠지는 경우가 있다.  

```java
public class Module {
  @Id
  private long id;

  @OneToMany(mappedBy="module", fetch=FetchType.LAZY)
  private List Users;
}

public class User {
  @Id
  private long id;

  @ManyToOne(fetch=FetchType.LAZY)
  @JoinColumn(name="module_id")
  @LazyToOne(value = LazyToOneOption.NO_PROXY)
  private Module module;
}
```

위와 같이 Entity를 구성하여 Module 클래스를 호출하였을 경우 LazyFetch를 적용하였다 하더라도 모든 Entity를 순회하게 되고 이 과정에서 User 클래스를 참조하게 된다.  
그러면 User 클래스는 다시 Module 클래스를 참조하게 되어 무한루프에 빠지게 되는 것이다.  
이를 해결하기 위한 방법으로는 크게 2가지가 있다.  

@JsonIgnore
-------------------------------
JsonIgnore는 해당 어노테이션이 달린 객체를 무시하도록 하는 기능을 한다.  
따라서 이 어노테이션을 자식의 module 변수에 붙이면 해당 클래스를 조회할 때 해당 변수는 무시되어 순환참조의 무한루프를 해결할 수 있다.  
문제는 Module 클래스에 의한 순환참조로 User 클래스를 호출한 것이 아닌 User 클래스를 직접 호출했을때도 JsonIgnore는 기능하여 해당 변수를 무시한다는 점이다.  

@JsonManagedReference, @JsonBackReference
-------------------------------
JsonIgnore의 경우 본래 순환참조를 막기 위한 기능이 아니다. 단지, 활용에 따라 가능하기 때문에 쓰였을 뿐.
하지만 @JsonManagedReference, @JsonBackReference는 본래부터 이 순환참조의 문제를 해결하기 위한 용도로 만들어진 기능이다.  
부모 클래스인 Module 클래스에 @JsonManagedReference를 추가하고  
자식 클래스인 User 클래스에 @JsonBackReference를 추가하면  
부모 클래스를 통해 순환참조 된 자식 클래스가 다시 부모 클래스를 참조하는 것을 막을 수 있다.  
