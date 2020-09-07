# 9. 도메인 모델과 BOUNDED CONTEXT

## 도메인 모델과 경계 <a id="5643b380-4b8a-4a2a-a9a4-6bb3be99df08"></a>

한 도메인은 여러 하위 도메인으로 구분되기 때문에 **한 개의 모델**로 여러 하위 도메인을 모두 표현하려고 시도하면 모든 하위 도메인에 맞지 않는 모델을 만들게 된다.

하위 도메인마다 같은 용어라도 의미가 다르고 같은 대상이라도 지칭하는 용어가 다를 수 있기 때문에 롱바른 도메인 모델을 개발하려면 **하위 도메인마다 모델을 만들어야 한다. 각 모델은 명시적으로 구분되는 경계를 가져서 섞이지 않도록 해야 한다.**

모델은 특정한 컨텍스트\(문맥\)하에서 완전한 의미를 갖는다. 이렇게 구분되는 경계를 갖는 컨텍스트를 DDD에서는 `BOUNDED CONTEXT`라고 부른다.

## BOUNDED CONTEXT <a id="d72638d5-1f28-4131-9327-ca1c164283fc"></a>

`BOUNDED CONTEXT`는 모델의 경계를 결정하며 한 개의 **BONDED CONTEXT**는 논리적으로 한 개의 모델을 갖는다. **BOUNDED CONTEXT**는 실제로 사용자에게 기능을 제공하는 물리적 시스템으로 도메인 모델은 이 **BOUNDED CONTEXT** 안에서 도메인을 구현한다.

이상적으로 하위 도메인과 **BOUNDED CONTEXT**가 일대일 관계를 가지면 좋지만 **BOUNDED CONTEXT**가 기업의 팀 조직 구조에 따라 결정되기도 한다. 같은 도메인의 로직을 따로 구현하는 팀이 있을 경우 한 도메인에 두 **BOUNDED CONTEXT**가 존재한다. 아직 용어를 며오학하게 하지 못해 두 하위 도메인을 한 **BOUNDED CONTEXT**에서 구현하기도 한다. 규모가 작은 기업은 전체 시스템을 한 개의 팀에서 구현\(여러 하위 도메인을 한 개의 **BOUNDED CONTEXT**에서 구현\)하기도 한다.

여러 하위 도메인을 하나의 **BOUNDED CONTEXT**에서 개발할 때 주의할 점은 하위 도메인의 모델이 뒤섞이지 않도록 하는 것이다. 한 개의 **BOUNDED CONTEXT**에서 여러 하위 도메인을 포함하더라도 하위 도메인마다 구분되는 패키지를 갖도록 구현해야 하위 도메인을 위한 모델이 서로 뒤섞이지 않아서 하위 도메인마다 **BOUNDED CONTEXT**를 갖는 효과를 낼 수 있다.

**BOUNDED CONTEXT**는 도메인 모델을 구분하는 경계가 되기 때문에 **BOUNDED CONTEXT**는 구현하는 하위 도메인에 알맞는 모델을 포함한다.

## BOUNDED CONTEXT의 구현 <a id="abb9bf4b-76a1-4fa2-95cc-160a3d5be054"></a>

**BOUNDED CONTEXT**는 도메인 모델 뿐만 아니라 도메인 기능을 사용자에게 제공하는 데 필요한 표현 영역, 응용 서비스, 인프라 영역 등을 모두 포함한다.

모든 **BOUNDED CONTEXT**가 반드시 도메인 주도로 개발할 필요는 없다. `CQRS` 패턴에서는 상태 변경과 관련된 기능은 **도메인 모델 기반**으로 구현하고 조회 기능은 **서비스-DAO**를 이용해서 구현할 수 있다.

각 **BOUNDED CONTEXT**는 서로 다른 구현 기술을 사용할 수 있다. 또한 **BOUNDED CONTEXT**가 반드시 사용자에게 보여지는 UI를 가져야 하는 것은 아니다.\(REST API\)

## BOUNDED CONTEXT 간 통합 <a id="346265a0-3b8b-4b17-9c86-d2cd05ac0113"></a>

두 팀이 관련된 **BOUNDED CONTEXT**를 개발하면 자연스럽게 두 **BOUNDED CONTEXT** 간 통합이 발생한다.

**예시**

쇼핑몰 서비스에서 카탈로그 하위 도메인에 추천 시스템을 도입할 경우 카탈로그 하위 도메인에는 기존 카탈로그를 위한 **BOUNDED CONTEXT**와 추천 기능을 위한 **BOUNDED CONTEXT**가 생긴다. 카탈로그 시스템은 추천 시스템으로부터 추천 데이터를 받아오지만, 추천의 도메인 모델을 사용하기보단 카탈로그 도메인 모델을 이용하여 추천 상품을 표현해야 한다. 즉, 카탈로그 모델을 기반으로 하는 `도메인 서비스`를 이용해서 상품 추천 기능을 표현해야 한다.

```text
// 상품 추천 기능을 표현하는 도메인 서비스
public interface ProductRecommendationService {
	public List<Product> getRecommendationsOf(ProductId id);
}
```

도메인 서비스를 구현한 클래스는 **인프라스트럭쳐 영역**에 위치한다. 이 클래스는 외부 시스템과의 `연동`을 처리하고 외부 시스템의 모델과 현재 모델간의 `변환`을 책임진다.

```text
// RecSystemClient는 외부 추천 시스템이 제공하는 REST API를 이용해서 특정 상품을 위한 추천 
// 목록을 반환한다.
public class RecSystemClient implements ProductRecommendationService {
	private ProductRepository productRepository;
	
	@Override
	public List<Product> getRecommendationsOf(ProductId id) {
		List<RecommendationItem> items = getRecItems(id.getValue());
		return toProducts(items);
	}

	// RecommendationItem은 추천 시스템의 모델, 외부 시스템에서 추천 목록 가져옴
	private List<RecommendationItem> getRecItems(String itemId) {
		// externalRecClient : 외부 추천 시스템 클라이언트
		return externalRecClient.getRecs(itemId);
	}
	
	// 추천 시스템의 모델을 카탈로그 시스템의 Product 모델로 변환
	private List<Product> toProducts(List<RecommendationItem> items) {
		return items.stream()
						.map(item -> ProductId::new)
					  .map(prodId -> productRepository.findById(prodId))
					  .collect(toList());
	}

}
```

REST API를 호출하는 것은 두 **BOUNDED CONTEXT**를 직접 통합하는 방법이다. 직접 통합하는 대신 간접적으로 통합하는 방법도 있다. 대표적인 간접 통합 방식이 `메시지 큐`를 사용하는 것이다.

**예시**

추천 시스템은 사용자의 조회 상품 이력이나 구매 이력과 같은 사용자 활동 이력을 필요로 하는데 이 내역을 전달할 때 메시지 큐를 사용할 수 있다.

1. 카탈로그 **BOUNDED CONTEXT**는 추천 시스템이 필요로 하는 사용자 활동 이력을 메시지 큐에 추가한다.
2. 추천 **BOUNDED CONTEXT**는 큐에서 이력 메시지를 읽어와 추천을 계산하는 데 사용한다. 이는 두 **BOUNDED CONTEXT**가 사용할 메시지의 데이터 구조를 맞춰야 함을 의미한다.

두 **BOUNDED CONTEXT**를 개발하는 팀은 메시징 큐에 담을 데이터의 구조를 협의 하게 되는데 그 큐를 `누가 제공`하는냐에 따라 데이터 구조가 결정된다.

**예시**

1. 카탈로그 시스템에서 큐를 제공한다면 큐에 담기는 애용은 카탈로그 도메인을 따른다.
2. 다른 **BOUNDED CONTEXT**는 이 큐로부터 필요한 메시지를 수신하는 방식을 사용한다.
3. 이 방식은 한쪽에서 메시지를 출판하고 다른 쪽에서 메시지를 구독하는 `출판/구독` 모델을 따른다.

### 마이크로서비스와 BOUNDED CONTEXT <a id="acda7043-45ed-4de3-8a44-1aa5bf856f3a"></a>

`마이크로서비스`는 애플리케이션을 작은 서비스로 나누어 개발하는 아키텍쳐 스타일이다. 개별 서비스를 독립된 프로세스로 실행하고 각 서비스가 REST API나 메시징을 이용해서 통신하는 구조를 갖는다.

각 **BOUNDED CONTEXT**는 모델의 경계를 형성하는데, **BOUNDED CONTEXT**를 마이크로서비스로 구현하면 자연스럽게 컨텍스트 별로 모델이 분리된다. 별도 프로세스로 개발한 **BOUNDED CONTEXT**는독립적으로 배포하고 모니터링하고 확장하게 되는데 이 역시 마이크로서비스의 특징이다.

## BOUNDED CONTEXT 간 관계 <a id="62272a7f-6adc-4f9d-bf75-53502cb87a0c"></a>

**BOUNDED CONTEXT**는 어떤 식으로든 연결되므로 두 **BOUNDED CONTEXT**는 다양한 방식으로 관계를 맺는다.

### 고객/공급자 관계 <a id="6b7cac23-fdcd-4215-b3b6-bfd5f1f92a7d"></a>

한쪽에서 API를 제공하고 다른 한쪽에서 그 API를 호출하는 관계이다. REST API가 대표적이다.

이 관계에서 API를 사용하는 **BOUNDED CONTEXT**는 API를 제공하는 **BOUNDED CONTEXT**에 의존하게 된다. 상류 컴포넌트는 일종의 `서비스 공급자` 역할을 하며, 하류 컴포넌트는 그 서비스를 사용하는 `고객` 역할을 한다. 고객과 공급자 관계에 있는 두 팀은 상호 협력이 필수적이다.

### 공고 호스트 서비스\(OPEN HOST SERVICE\) <a id="d5f92d87-c3bf-4985-969b-f529d56f0b1c"></a>

상류 팀의 고객인 하류 팀이 다수 존재하면 상류 팀은 여러 하류 팀의 요구사항을 수용할 수 있는 API를 만들고 이를 서비스 형태로 공개해서 서비스의 일관성을 유지할 수 있다. 이런 서비스를 `공개 호스트 서비스`라고 한다.

상류 컴포넌트 서비스는 상류 **BOUNDED CONTEXT**의 도메인 모델을 따른다. 따라서, 하류 컴포넌트는 상류 서비스의 모델이 자신의 도메인 모델에 영향을 주지 않도록 보호해 주는 완충 지대를 만들어야 한다. 즉, 내 모델이 깨지는 것을 막아 주는 `안티코럽션 계층`\(Anticorruption Layer\)이 된다. 이 계층에서 두 **BOUNDED CONTEXT** 간의 모델 변환을 처리해 주기 때문에 다른 **BOUNDED CONTEXT**의 모델에 영향을 받지 않고 내 도메인 모델을 유지할 수 있다.

안티코럽션 계층은 인프라 계층에 속한다.

### 공유 커널\(SHARED KERNEL\) <a id="5e1e90f5-5e1f-4c7a-974f-668e62e0c0f8"></a>

두 **BOUNDED CONTEXT**가 같은 모델을 공유하는 경우도 있다. 이렇게 두 팀이 공유하는 모델을 `공유 커널`이라고 부른다.

공유 커널의 장접은 중복을 줄여준다. 하지만 두 팀이 한 모델을 공유하기 때문에 한 팀에서 임의로 모델을 변경해서는 안 되며 두 팀이 밀접한 관계를 유지해야 한다.

### 독립 방식\(SEPERATE WAY\) <a id="69cd7d5c-7ba7-4bb6-a69f-76289ca861ed"></a>

독립 방식은 서로 통합하지 않는 방식이다. 두 **BOUNDED CONTEXT** 간에 통합을 하지 않으므로 서로 독립적으로 모델을 발전시킨다.

독립 방식에서 두 **BOUNDED CONTEXT** 간의 통합은 수동으로 이루어지며, 두 **BOUNDED CONTEXT**를 통합해 주는 별도의 시스템을 만들어야 할 수도 있다.

## 컨텍스트 맵 <a id="df33fe26-f2fe-41da-8e2b-34584df74809"></a>

`컨텍스트 맵`은 **BOUNDED CONTEXT** 간의 관계를 표시한 것이다. 컨텍스트 맵은 시스템의 전체 구조를 보여준다. 이는 하위 도메인과 일치하지 않는 **BOUNDED CONTEXT**를 찾아 도메인에 맞게 **BOUNDED CONTEXT**를 조절하고 사업의 핵심 도메인을 조직 역량을 어떤 **BOUNDED CONTEXT**에 집중할지 파악하는 데 도움을 준다.

