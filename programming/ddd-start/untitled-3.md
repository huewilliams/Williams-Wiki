# 4. 리포지터리와 모델 구현 \(JPA\)

## JPA를 이용한 리포지터리 구현 <a id="73a56b84-7a17-4d7a-acd8-afe224f33217"></a>

### 모듈 위치 <a id="f561e716-f689-4511-bc66-4432414a0d83"></a>

리포지터리 인터페이스는 애그리거트와 같이 도메인 영역에 속하고,

리포지터리를 구현한 클래스는 인프라스트럭쳐 영역에 속한다.

domain \( Model → &lt;&lt;interface&gt;&gt;ModelRepository \) ← infrastructure\( JpaModelRepository \)

### 리포지터리 기본 기능 구현 <a id="7d334a04-4b53-4083-b57c-6180e29578c3"></a>

리포지터리의 기본 기능은 다음 두 가지이다.

* 아이디로 애그리거트 조회하기
* 애그리거트 저장하기

인터페이스는 애그리거트 루트를 기준으로 작성한다.

**이 두 메서드를 위한 리포지터리 인터페이스**

```text
public interface OrderRepository {
	public Order findById(OrderNo no);
	// 아이디에 해당하는 애그리거트가 존재하면 Order를 리턴하고, 존재하지 않으면 null 리턴
	// Optional로 null 대신 처리 가능

	public void save(Order order);
	// 전달받은 애그리거트를 저장
	// JPA의 EntityManager를 이용하여 기능 구현

	public void delete(Order order);
	// 애트리거트 삭제
}
```

**JPA와 스프링을 이용한 리포지터리 구현**

```text
@Repository
public class JpaOrderRepository implements OrderRepository {
	@PersistenceContext
	private EntityManager entityManager;

	@Override
	public Order findById(OrderNo id) {
		return entityManager.find(Order.class, id);
	}

	@Override
	public void save(Order order) {
		entityManager.persist(order);
	}

	@Override
	public void remove(Order order) {
		entityManager.remmove(order);
	}
}
```

애그리거트를 수정한 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다. JPA를 사용하면 트랜잭션 범위에서 변경한 데이터를 자동으로 DB에 반영하기 때문이다.

## 매핑 구현 <a id="d568e316-4cea-4c15-b28a-4a44b6ab01d0"></a>

### 엔티티와 밸류 기본 매핑 구현 <a id="fadfcbc7-66fb-4076-b170-319f06d5798d"></a>

* 애그리거트 루트는 엔티티이므로 `@Entity`로 매핑 설정한다.
* 한 테이블에 엔티티와 밸류 데이터가 같이 있다면,
  * 밸류는 `@Embeddable`로 매핑 설정한다.
  * 밸류 타입 프로퍼티는 `@Embedded`로 매핑 설정한다.

**예시**

주문 애그리거트의 루트 엔티티인 Order는 JPA의 `@Entity`로 매핑한다. `@Embedded`를 이용해서 밸류 타입 프로퍼티를 설정한다.

```text
@Entity
@Table(name = "purchase_order")
public class Order {
	@Embedded
	private Orderer orderer;
	
	@Embedded
	private ShippingInfo shippingInfo;
}
```

Order에 속하는 Orderer는 밸류이므로 `@Embeddable`로 매핑한다. Orderer의 memeberId 프로퍼티와 매핑되는 칼럼 이름은 'order\_id'이므로 MemberId에 설정된 'member\_id'와 이름이 다르다. 따라서 `@AttributeOverrides`로 매핑할 칼럼 이름을 변경한다.

```text
@Embeddable
public class Orderer {
	
	@Embedded
	// MemberId에 정의된 칼럼 이름을 변경하기 위해 사용
	@AttributeOverrides(
		@AttributeOverride(name="id", column=@Column(name="order_id"))
	)
	private MemberId memberId;
	
	@Column(name="orderer_name")
	private String name;
}

@Embeddable
public class MemberId implements Serializable {
	@Column(name = "member_id")
	private Stirng id;
}
```

### 기본 생성자 <a id="b5b8095d-c059-40f6-8d08-f5ed5fc08740"></a>

엔티티와 밸류의 생성자는 객체를 생성할 때 필요한 것을 전달받는다. 불변 타입으로 구현하므로 기본 생성자는 필요없다. 하지만 JPA의 `@Entity`와 `@Embeddedable`로 클래스를 매핑하려면 **기본 생성자**를 제공해야 한다. 하이버네이트와 같은 JPA 프로바이더는 DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본 생성자를 사용해서 객체를 생성한다.

### 필드 접근 방식 사용 <a id="4acf6f47-669a-496b-b7eb-801af39c546f"></a>

JPA는 `필드`와 `메서드` 두 가지 방식으로 매핑을 처리할 수 있다.

메서드 방식을 사용하려면 프로퍼티를 위한 get/set 메서드를 구현해야 한다. 하지만 프로퍼티를 위한 공개 get/set 메서드를 추가하면 도메인의 의도가 사라지고 캡슐화를 깨며 밸류 타입을 불변으로 구현하는데 방해가 된다.

엔티티를 객체가 제공할 기능을 중심으로 구현하도록 유도하려면 JPA 매필 처리를 프로퍼티 방식이 아닌 **필드** 방식으로 선택해서 불필요한 get/set 메서드를 구현하지 말아야 한다. \(`@Access(AccessType.FIELD)`를 통해 명시적으로 접근 방식을 지정하지 않으면 `@Id`나 `@EmbeddedId`가 필드위에 있으면 필드 접근 방식, get 메서드에 위치하면 메서드 접근 방식을 선택한다.\)

### AttributeConverter를 이용한 밸류 매핑 처리 <a id="464d25e2-b3f6-4685-9019-3c559ae8bc38"></a>

두 개 이상의 프로퍼티를 가진 밸류 타입을 한 개 칼럼에 매핑해야 할 경우 **AttributeConverter**를 사용해서 변환을 처리할 수 있다.

```text
// X : 밸류 타입, Y : DB 타입
public interface AttributeConverter<X,Y> {
	// 밸류 타입 -> DB 컬럼 값
	public Y converToDatabaseColumn (X attribute);
	// DB 컬럼 값 -> 밸류 타입
	public X converToEntityAttribute (Y dbData);
}
```

Money 밸류 타입을 위한 AttributeConverter

```text
// AttributeConverter 인터페이스를 구현한 클래스는 @Converter을 적용한다.
// autoApply = true : 모든 Money 타입의 프로퍼티에 자동으로 해당 Converter를 적용한다.
@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Interger> {
	@Override
	public Integer convertToDatabaseColumn(Money money) {
		return money.getValue();
	}

	@Override
	public Money convertToEntityAttribute(Integer value) {
		return new Money(value);
	}
}
```

`@Converter`의 autoApply 속성이 false인 경우 프로퍼티 값을 변환할 때 사용할 컨버터를 직접 지정할 수 있다.

```text
public class Order {
	@Convert(converter = MoneyConverter.class)
	private Money totalAmounts;
}
```

### 밸류 컬렉션 : 별도 테이블 매핑 <a id="90cab2a7-5bdc-4ef6-9c41-ca8e7cb0b82d"></a>

밸류 타입의 컬렉션은 별도 테이블에 보관한다. 밸류 컬렉션을 별도 테이블로 매핑할 때는 `@ElementCollection`과 `@CollectionTable`을 함께 사용한다.

```text
@Entity
public class Order {
	@ElementCollection
	@CollectionTable(name = "order_line", // 밸류를 저장할 테이블 지정
									// 외부 키로 사용되는 컬럼을 지정
									joinColumns = @JoinColumn(name = "order_number")
	@OrderColumn(name = "line_idx") //리스트의 인덱스 값 지정
	private List<OrderLine> orderLines;
}
```

### 밸류 컬렉션 : 한 개 컬럼 매핑 <a id="8a7f0953-d243-4f01-9b55-8c9c402ae8f5"></a>

밸류 컬렉션을 별도 테이블이 아닌 한 개 칼럼에 저장해야 할 때가 있다.\(이메일 주소 목록을 콤마로 구분해서 저장해야 할 때\). 이 때는 밸류 컬렉션을 표현하는 새로운 **밸류 타입**을 추가하여 **AttributeConverter**를 사용하면 매핑할 수 있다.

```text
@Converter
public class EmailSetConverter implements AttributeConverter<EmailSet, String> {
	@Override
	public String convertToDatabaseColumn(EmailSet attribute) {
		if (attribute == null) return null;	
		return attribute.getEmails().stream()
							.map(Email::toString())
							.collect(Collectors.joining(","));

	@Override
	public EmailSet convertToEntityAttribute(String dbData) {
		if (dbData == null) return null;
		String[] emails = dbData.split(",");
		Set<Email> emailSet = Arrays.stream(emails)
							.map(value -> new Email(value))
							.collect(toSet());
		return new EmailSet(emailSet);
	}
}

// EmailSet타입의 프로퍼티가 Converter로 EmailSetConverter 사용 지정
@Convert(converter = EmailSetConverter.class)
private EmailSet emailSet;
```

### 밸류를 이용한 아이디 매핑 <a id="9b8c5572-e36c-4316-9f0c-75282cbee058"></a>

엔티티의 식별자로 식별자라는 의미를 부각시키기 위해 기본 타입이 아닌 식별자 자체를 별도 밸류 타입으로 만들 수 있다. 밸류 타입을 식별자로 매핑하면 `@Id` 대신 `@EmbeddedId` 를 사용한다.

```text
@Entity
public class Order {
	@EmbeddedId
	private OrderNo number;
}

@Embeddable
public class OrderNo implements Serializable {
	@Column(name = "order_number")
	private String number;
}
```

### 밸류 : 별도 테이블 매핑 <a id="6d422e41-60c2-403a-a458-26d93ab78087"></a>

ArticleContent는 밸류이므로 `@Embeddable`로 매핑한다. ArticleContent와 매핑되는 테이블은 Article과 매핑되는 테이블과 다르므로 밸류를 매핑한 테이블을 지정하기 위해 `@SecondaryTable`과 `@AttributeOverride`를 사용한다.

```text
@Entity
@SecondaryTable(
	name = "article_content", // 밸류를 저장할 테이블 지정
	// 밸류 테이블에서 엔티티 테이블로 조인할 때 사용할 칼럼 지정
	pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {

	@AttributeOverrides({
		@AttributeOverride(name = "content",
					column = @Column(table = "article_content"),
		@AttributeOverride(name = "contentType",
					column = @Column(table = "article_content")
	})
	private ArticleContent content;
}

// @SecondaryTable로 매핑된 article_content 테이블을 조인하여 조회
Article article = entityManager.find(Article.class, 1L);
```

### 밸류 컬렉션 : @Entity로 매핑 <a id="692be90e-b827-4e93-929c-74a033ef5996"></a>

개념적으로 밸류인데 구현 기술의 한계나 팀 표준으로 `@Entity`를 사용해야 할 때도 있다. JPA는 `@Embeddable` 타입의 클래스 상속 매핑을 지원하지 않는다. 따라서 상속 구조를 갖는 밸류 타입을 사용하려면 `@Entity`를 이용한 상속 매핑으로 처리해야 한다. 밸류 타입을 `@Entity`로 매핑하므로 **식별자** 매핑을 위한 필드도 추가해야 한다. 또한, 구현 클래스를 구분하기 위한 **타입 식별\(discriminator\)** 칼럼을 추가해야 한다.

⚠ 밸류를 `@Entity`로 매핑했으므로 상태 변경 메서드를 제공하지 않는다.

```text
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
// 타입 식별 칼럼 추가
@DiscriminatorColumn(name = "image_type")
public abstract class Image {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "image_id)
	private Long id; // 식별자 매핑을 위한 필드

	@Column(name = "image_path)
	private String path;

	@Temporal(TemporalType.TIMESTAMP)
	private Date upload_time;
}

// Image를 상속받은 클래스는 @Entity와 @Disciminator를 사용해서 매핑을 설정한다.
@Entity
@DiscriminatorValue("II")
public class InternalImage extends Image {
}

@Entity
@DiscriminatorValue("EI")
public class ExternalImage extends Image {
}
```

Image가 `@Entity`이므로 목록을 담고 있는 Product는 `@OneTo-Many`를 사용해서 매핑을 처리한다. Image는 밸류이므로 라이프사이클을 갖지 않고 Product에 완전히 의존한다.

```text
@Entity
public class Product {
	@EmbeddedId
	private ProductId productId;
	
	@Convert(converter = MoneyConverter.class)
	private Money price;
	
	// cascade 속성으로 Product를 저장할 때 함께 저장하고, 삭제할 때 함께 삭제하도록 설정
	@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE}, 
	// orphanRemoval = true : Image 객체를 제거하면 DB에서 함께 삭제시킴
					orphanRemoval = true)
	@JoinColumn(name = "product_id")
	@OrderColumn(name = "list_idx")
	private List<Image> images = new ArrayList<>();
}
```

## 애그리거트 로딩 전략 <a id="9ccdead7-e18d-4d16-aef8-99e178c44d65"></a>

조회 시점에서 애그리거트를 완전한 상태가 되도록 하려면 애그리거트 루트에서 연관 매핑의 조회 방식을 `즉시 로딩`\(FetchType.EAGER\)으로 설정하면 된다. 즉시 로딩으로 설정하면 EntityManager.find\(\)메서드로 애그리거트 루트를 구할 때 연관된 구성요소를 DB에서 함꼐 읽어온다.

```text
// @Entity 컬렉션에 대한 즉시 로딩 설정
@OneToMany(cascade= {CascadeType.PERSIST, CascadeType.REMOVE},
				orphanReomval = true, fetch = FetchType.EAGER)
@JoinColumn(name = "product_id")
@OrderColumn(name = "list_idx")
private List<Image> images = new ArrayList<>();

// @Embeddable 컬렉션에 대한 즉시 로딩 설정
@ElementCollection(fetch = FetchType.EAGER)
@CollectionTable(name = "order_line",
							joinColumns = @JoinColumn(name = "order_number"))
@OrderColumn(name = "line_idx")
private List<OrdereLine> orderLines;
```

애그리거트는 개념적으로 하나여야 한다. 하지만, 루트 엔티티를 로딩하는 시점에 애그리거트에 속한 객체를 모두 로딩해야 하는 것은 아니다. 애그리거트 상태 변경 기능을 실행하기 위해 조회 시점에 즉시 로딩을 이용해서 애그리거트를 완전한 상태로 로딩할 필요는 없다. JPA는 트랜잭션 범위 내에서 지연 로딩을 허용하므로 실제로 상태를 변경하는 시점에 필요한 구성요소만 로딩해도 문제되지 않는다.

```text
@Entity
public class Product {
	@ElementCollection(fetch = FetchType.LAZY)
	@CollectionTable(name = "product_option",
								joinColumns = @JoinColumn(name = "product_id")
	@OrderColumn(name = "list_idx")
	private List<Option> options = new ArrayList<>();

	public void removeOption(int optIdx) {
		// 실제 컬렉션에 접근할 때 로딩
		this.options.remove(optIdx);
	}
}
```

## 영속성 전파 <a id="c5cb024e-b617-4164-a48d-946c394efe82"></a>

애그리거트가 완전한 상태여야 한다는 것은 애그리거트 루트를 조회할 때뿐만 아니라 저장하고 삭제할 때도 하나로 처리해야 함을 의미한다.

`@Embeddable` 매핑 타입의 경우 함께 저장되고 삭제되므로 cascade 속성을 추가 설정하지 않아도 된다. 반면 애그리거트에 속한 `@Entity` 타입에 대한 매핑은 cascade 속성을 사용해서 저장과 삭제 시에 함께 처리되도록 설정해야 한다.

## 식별자 생성 기능 <a id="b8d09cbf-a2f1-4fbf-bae9-1629dab9fc65"></a>

식별자는 크게 세 가지 방식 중 하나로 생성한다.

* 사용자가 직접 생성
* 도메인 로직으로 생성
* DB를 이용한 일련번호 사용

`식별자 규칙`이 있는 경우 엔티티를 생성할 때 이미 생성한 식별자를 전달하므로 엔티티가 식별자 생성 기능을 제공하는 것보다 별도 서비스로 분리해야 한다. 식별자 생성 규칙은 모메인 서비스에 위치시키거나 리포지터리에서 구현한다.

식별자 생성으로 `DB의 자동 증가 칼럼`을 사용할 경우 JPA의 식별자 매핑에서 `@GeneratedValue`를 사용한다. 자동 증가 칼럼은 DB insert 쿼리를 실행해야 식별자가 생서외므로 도메인 객체를 리포지터리에 저장할 때 식별자가 생성된다.

