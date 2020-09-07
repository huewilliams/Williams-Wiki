# 5. 리포지터리 조회 구현 \(JPA\)

DDD Start! 에서는 JPA를 이용하여 구현한다. 하지만 나는 JPA를 한번 더 매핑한 Spring Data JPA를 사용하기 때문에 책에서는 리포지터리 조회 기법들만 추출하여 Spring Data JPA기반으로 구현하는 방법을 찾아서 정리했다.

## 검색을 위한 스펙 <a id="243b7dce-bd1a-4c16-ac6b-7922bd55a031"></a>

### 기존의 문제점 <a id="f8e69e99-a171-4974-ad5f-51617970160c"></a>

기존에는 하나의 메서드에서 하나의 검색 조건만 처리하기 때문에 `title`과 `tag`를 동시에 검색하는 기능을 구현하려면 또 하나의 메서드를 작성해야 한다.

```text
@Entity
public class Post {
    @Id @GeneratedValue
    private Long id;
    private String title;
    private String tag;
    private int likes;
...
}

public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findAllByTitle(String title);
    List<Post> findAllByTag(String tag);
    List<Post> findAllByLikesGreaterThan(int likes);
}
```

### Specification <a id="d128b84d-104f-4380-92f8-0f97d5eed611"></a>

**Specification**을 적용하기 위해서는 Repository에 `JpaSpecificationExecutor<T>` 인터페이스를 추가로 상속받아야 한다.

```text
public interface PostRepository extends JpaRepository<Post, Long>, JpaSpecificationExecutor<Post> {
    ...
}
```

**title로 Post를 검색하는 Specification**

`root.get("title")`을 통해 Post 인스턴스의 title 필드가 검색조건으로 입력받은 매개변수 title과 일치하는지 확인하고 해당하는 Post 객체를 반환한다.

```text
public class PostSpecs {
    public static Specification<Post> withTitle(String title) {
        return (Specification<Post>) ((root, query, builder) -> 
                builder.equal(root.get("title"), title)
        );
    }
}
```

해당 람다식은 **Specification**의 `toPredicate()` 메소드를 구현한 것이다.

```text
@Nullable
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
```

이렇게 만들어진 Specification은 Repository의 매개변수에 적용할 수 있다.

```text
@RestController
public class PostController {

   ...

    @GetMapping("/post/list")
    public List<Post> getPostList(@RequestParam(required = false) String title,
                                  @RequestParam(required = false) String tag,
                                  @RequestParam(required = false) Integer likes) {
        if (title != null) {
            return postRepository.findAll(PostSpecs.withTitle(title));
        } else if (...) {
            ...
        }
    }

}
```

### Spec 조합 <a id="7baf2fb9-6142-455a-aed3-7a21ccdfe41a"></a>

다음과 같이 AND/OR로 Spec을 조합하여 사용할 수 있다.

```text
UserSpecification spec1 = new UserSpecification(new SearchCriteria(
"firstName", ":", "Admin"));
UserSpecification spec2 = new UserSpecification(new SearchCriteria("lastName",
 ":", "Fox"));

List<User> results = repository.findAll(Specification.where(spec1).and(spec2));
```

## 정렬 구현 <a id="56468a6e-db1b-40dd-9b56-dfbe67ca8114"></a>

일반적으로 JPA에서 정렬 기능을 구현하기 위해 아래와 같이 메서드명에 `OrderBy`를 붙여 사용한다.

```text
public Page<T> findAllByNameOrderByCreateDateDesc();
```

이와 같이 만드는 경우 조회 시 다수의 정렬 기능\(이름순, 최신순, 추천순..\)을 필요로 할 경우 위와 같은 메서드를 정렬 방식 개수대로 만들어야 하는 단점이 있다.

### Sort 클래스 <a id="2b3830e0-32af-4c84-8a3b-623b33ce5603"></a>

Controller에 아래와 같이 인자값으로 등록하면 정렬을 쿼리 값으로 지정할 수 있다.

```text
@Controller
public List<User> list(Sort sort) {
	List<User> users = userRepository.findAll(sort);
	return users;
}
```

파라미터 형식은 아래와 같이 넘길 수 있다.

* 이름으로 정렬 : /path?sort=name, asc
* 역순으로 정렬 : /path?sort=name, desc
* 이름으로 정렬 + ID로 정렬 : /path?sort=name,id
* 이름으로 정렬 + ID 역순으로 정렬 : /path?sort=name,asc&sort=id,desc

파라미터에서 전달받은 정렬 조건외에 추가적으로 정렬 조건을 추가하고 싶은 경우 아래와 같이 `and` 메소드를 이용하여 추가적으로 정렬 조건을 넣을 수 있다.

```text
@Controller
public List<User> list(Sort sort) {
	sort = sort.and(new Sort(Sort.Direction.DESC, "count"));
	List<User> users = userRepository.findAll(sort);
	return users;
}
```

보통 목록에서 정렬은 paging과 같이 하는 경우가 많은데 아래와 같이 Pageable을 인자값으로 받으면 자동적으로 정렬값이 추가된다.

```text
@Controller
public List<User> list(Pageable pageable) {
	List<User> users = userRepository.findAll(pageable);
	return users;
}
```

## 페이징과 개수 구하기 구현 <a id="7ec21147-5e5b-46ea-9bbc-e4d89be57abf"></a>

Query 메소드의 입력변수로 아래와 같이 **Pageable** 변수를 추가하면 **Page** 타입을 반환형으로 사용할 수 있다. Pageable 객체를 통해 페이징과 정렬을 위한 파라미터를 전달한다.

Pageable 입력 변수는 아래와 같이 Controller에서부터 전달받아야 한다.

```text
@RestController
@RequestMapping("/member")
public class MemberController {
	
	...

	@RequestMapping("")
	Page<Member> getMembers(Pageable pageable) {
		return memberService.getList(pageable)
	}
}
```

위와 같이 작성된 **Pagealbe**에서는 다음과 같은 파라미터를 자동으로 수집한다.

#### Pageable 파라미터

| query parameter 명 | 설명 |
| :--- | :--- |
| [page](https://www.notion.so/page-e43e9eddb68a4f9aafb6498624bb6122) | 몇 번째 페이지인지 전달 |
| [size](https://www.notion.so/size-6c074f8ac37a4508a8fa48170566cfee) | 한 페이지에 몇 개의 항목을 보여줄 것인지 전달 |
| [sort](https://www.notion.so/sort-3dc7fc38016a41ec83bf32776bbeebb2) | 정렬 정보를 전달, 정렬 정보는 필드이름, 정렬방향의 포맷으로 전달한다. 여러 필드를 순차적으로 정렬도 가능하다. \(sort=createdAt,desc&sort=userId,asc\) |

아래는 위 Controller를 통해 HTTP요청으로 페이징과 정렬된 데이터를 전달받는 URI 샘플이다.

**GET /users?page=1&size=10&sort=createdAt,desc&sort=userId,asc**

## 조회 전용 기능 구현 <a id="e556a364-5ab8-4e36-8b59-46f52b39f1be"></a>

리포지터리는 여러 애그리거트를 조합하여 데이터를 제공하거나 각종 통계 데이터를 제공하는 것은 적합하지 않다. 이러한 기능은 **조회 전용 쿼리**로 처리해야 한다. `JPA`와 `하이버네이트`를 사용하면 **동적 인스턴스 생성**, **`@Subselect`확장 기능, 네이티브 쿼리**를 이용하여 조회 전용 기능을 구현할 수 있다.

### 동적 인스턴스 생성 <a id="9b80fc76-20c3-49c0-9092-9ff392cdc0c3"></a>

**JPQL에서 동적 인스턴스를 사용한 코드**

코드에서 JPQL의 select 절에는 new 키워드가 있다. new 키워드 뒤에 생성할 인스턴스의 완전한 클래스 이름을 지정하고 괄호 안에 생성자에 인자로 전달할 값을 지정한다. 이 코드는 OrderView 생성자에 각각 Order, Member, Product를 전달하고 생성자는 전달받은 객체로 부터 필요한 값을 추출한다. 이런 방법 이외에 모델의 개별 프로퍼티를 생성자에 전달할 수도 있다.

```text
@Repository
public class JpaOrderViewDao implements OrderViewDao {
	@PersistenceContext
	private EntityManager em;

	@Override
	public List<OrderView> selectByOrderer(String ordererId) {
		String selectQuery = 
					"select new com.shop.order.dto.OrderView(o,m,p)" +
					"form Order o join o.orderLines ol, Member m, Product p" + 
					"where o.orderer.memberId.id = :ordererId" +
					"and o.orderer.memberId = m.id" +
					"and index(ol) = 0" +
					"and ol.productId = p.id" +
					"order by o.number.number desc";
		TypedQuery<OrderView> query = em.createQuery(selectQuery, OrderView.class);
		query.setParameter("ordererId", ordererId);
		return query.getResultList();
	}
}

public class OrderView {
	private String number;
	private long totalAmounts;
	private String productName;

	public OrderView(Order order, Member member, Product product) {
		this.number = order.getNumber().getNumber();
		this.totalAmounts = order.getTotalAmounts().getValue();
		this.productName = product.getBane();
	}
}
```

동적 인스턴스의 장점은 JPQL을 그대로 사용하므로 객체 기준으로 쿼리를 작성하면서도 지연/즉시 로딩과 같은 고민 없이 원하는 모습으로 데이터를 조회할 수 있다는 점이다.

### 하이버네이트의 @Subselect 사용 <a id="550a5d7f-a8c9-435c-ac82-9af1966ea7cc"></a>

하이버네이트는 JPA의 확장 기능으로 쿼리 결과를 `@Entity`로 매핑할 수 있는 `@Subselect`를 제공한다.

`@Immutable` , `@Subselect`, `@Synchronize`는 하이버네이트 전용 어노테이션으로 이 태그를 사용하면 테이블이 아닌 조회\(select\) 쿼리 결과를 `@Entity`로 매핑할 수 있다.

**@Immutable**

뷰를 수정할 수 없듯이 `@Subselect`로 조회한 `@Entity`도 수정할 수 없다. 이를 보장하기 위해 `@Immutable`을 사용한다.

**@Synchronize**

`@Synchonize`는 해당 엔티티와 관련된 테이블 목록을 명시한다. 하이버네이트는 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 플러시를 먼저한다. 따라서 OrderSummary를 로딩하는 시점에는 변경 내역이 반영된다.

**@Subselect**

`@Subselect`는 값으로 지정한 쿼리를 from 절의 서브 쿼리로 사용한다. 서브 쿼리를 사용하고 싶지 않다면 네이티브 SQL을 사용하거나 MyBatis와 같은 별도 매퍼를 사용해서 조회 기능을 구현해야 한다.

```text
// @Subselect를 적용한 @Entity는 일반 @Entity와 동일한 방법으로 조회할 수 있다.
// find, JPQL, Criteria, Spec 등
@Entity
@Immutable
@Subselect("select o.order_number as number, " +
					"o.orderer_id, o.orderer_name, o.total_amounts, " + 
					"o.receiver_name, o.state, o.order_date" +
					"p.product_id, p.name as product_name" + 
					"from purchase_order o inner join order_line ol" +
					"   on o.order_number = ol.order_number" +
					"   cross join product p" +
					"where ol.line_idx = 0 and ol.product_id = p.product_id"
)
@Synchornize({"purchase_order", "order_line", "product"})
public class OrderSummary {
	@Id
	private String number;
	private String ordererId;
	private String ordererName;
	private int totalAmounts;
	private String receiverName;
	private String state;
	@Temporal(TemporalType.TIMESTAMP)
	@Column(name = "orderDate")
	private Date orderDate;
	private String productId;
	private String productName;
	
	protected OrderSummary() {}
}
```

## Reference <a id="41c91519-0334-4126-aae3-259bbba6bfd4"></a>

* [https://velog.io/@hellozin/JPA-Specification으로-쿼리-조건-처리하기](https://velog.io/@hellozin/JPA-Specification%EC%9C%BC%EB%A1%9C-%EC%BF%BC%EB%A6%AC-%EC%A1%B0%EA%B1%B4-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0)
* [https://www.baeldung.com/rest-api-search-language-spring-data-specifications](https://www.baeldung.com/rest-api-search-language-spring-data-specifications)
* [https://jistol.github.io/java/2017/02/11/jpa-sort/?fbclid=IwAR17szwmoU3nNUFQX8uo3\_lNtEFCtTujDR5L6WLXXTie2ooAtcYtKebN5gI](https://jistol.github.io/java/2017/02/11/jpa-sort/?fbclid=IwAR17szwmoU3nNUFQX8uo3_lNtEFCtTujDR5L6WLXXTie2ooAtcYtKebN5gI)
* [https://jobc.tistory.com/120](https://jobc.tistory.com/120)

