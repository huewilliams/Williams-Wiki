# 1. 도메인 모델 시작

## 도메인 <a id="87b9e29c-7baf-45aa-b666-1391a8491229"></a>

도메인\(domain\) : 소프트웨어로 해결하고자하는 `문제 영역`

* 도메인은 여러 `하위 도메인`으로 구성된다.

  온라인 서점 도메인은 회원, 주문, 결제, 배송 등의 하위 도메인으로 구성된다.

* 소프트웨어가 도메인의 모든 기능을 제공하지는 않는다.

  결제 도메인을 직접 구현하기보다 결제 대행 업체\(PG사\)를 이용해서

## 도메인 모델 <a id="3b1dde82-3cf9-411e-925b-4ffc19942633"></a>

도메인 모델 : 특정 도메인을 `개념적`으로 표현한 것.

* 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는 데 도움이 된다.
* 모델의 각 요소는 특정 도메인을 한정할 때 비로소 의미가 완전해지기 때문에, 각 하위 모델마다 `별도의 모델`을 만들어야 한다.

## 도메인 모델 패턴 <a id="525f955a-bb88-40ab-b470-8bf816b7d4fc"></a>

### 아키텍쳐 구성

| 계층\(Layer\) | 설명 |
| :--- | :--- |
| [사용자인터페이스\(UI\) 또는 표현\(Presentation\)](https://www.notion.so/UI-Presentation-2f0a30604e1045af82aecfbd9cc71116) | 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. |
| [응용\(Application\)](https://www.notion.so/Application-a0fc60f2b8944bc5be41cd646032f9b7) | 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다. |
| [도메인](https://www.notion.so/43e14c423126477797d792d61cb819cc) | 시스템이 제공할 도메인 규칙을 구현한다. |
| [인프라스트럭쳐\(Infrastructure\)](https://www.notion.so/Infrastructure-d601f7a675cf473f970be4d50cb83cab) | DB나 메세징 시스템 같은 외부 시스템과의 연동을 처리한다. |

도메인 계층은 도메인의 `핵심 규칙`을 구현한다. 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하므로 규칙이 바뀌거나 확장해야 할 때 다른 코드에 영향을 덜 주고 변경내역을 모델에 반영할 수 있게 된다.

## 도메인 모델 도출 <a id="76a68853-39d5-43f2-a2fe-45337b85d644"></a>

도메인을 모델링할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다. 이 과정은 `요구사항`에서 출발한다.

요구사항에서 기능과 규칙, 데이터를 추출하여 코드에 반영하며 도메인 모델을 구성해간다.

## 엔티티와 밸류 <a id="764ac445-c13f-4de4-b976-cdfc393a2ac1"></a>

도출한 모델은 크게 `엔티티(Entity)`와 `밸류(Value)`로 구분할 수 있다.

**엔티티**

엔티티의 가장 큰 특징은 `식별자`를 갖는다는 것이다. 식별자는 엔티티 객체마다 고유해서 각 엔티티는 서로 다른 식별자를 갖는다. 엔티티를 생성하고 엔티티의 속성을 바꾸고 엔티티를 삭제할 때까지 식별자는 유지된다.

엔티티의 식별자는 바뀌지 않고 고유하므로 두 엔티티 객체의 식별자가 같다면 두 엔티티는 같다고 판단할 수 있다.

**밸류 타입**

밸류 타입은 `개념적으로 완전한 하나`를 표현할 때 사용할 수 있다.

밸류 타입이 꼭 두 개 이상의 데이터를 가져야 하는 것은 아니다. 의미를 명확하게 표현하기 위해 밸류 타입을 사용하는 경우도 있다.

## 도메인 모델에 set 메서드 넣지 않기 <a id="17f38a10-6496-413b-8b78-964d1975d0f8"></a>

도메인 모델에 set 메서드는 도메인의 핵심 개념이나 의도를 코드에서 사라지게 한다.

또한 도메인 객체를 생성할 때 완전한 상태가 아닐수도 있게 된다. 도메인 객체가 불완전한 상태로 사용되는 것을 막으려면 `생성자`를 통해 필요한 데이터를 모두 받아야 한다.

## 도메인 용어 <a id="42beb8d2-2ee2-4de5-a9a8-6b2043f55cea"></a>

코드를 작성할 때 도메인에서 사용하는 `용어`는 매우 중요하다. 도메인에서 사용하는 용어를 코드에 반영하지 않으면 그 코드는 개발자에게 코드의 의미를 해석해야 하는 부담을 준다.

도메인 용어에 알맞는 영어 단어를 찾는 것은 쉽지 않은 일이지만 시간을 들여 찾는 노력을 해야 한다.

