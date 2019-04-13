# Immutability 객체와 변경불가성
## Rest 파라미터(...) 
정해지지 않은 수의 파라미터(인자)를 배열로 나타냄

## 불변 데이터 패턴 (Immutable data pattern)
객체의 방어적 복사(defensive copy)
### Object.assign(target, ...sources)
 객체를 복사하여 새로운 객체를 생성한다. 새로운 객체를 생성하므로 기존 객체와 어드레스를 공유하지 않는다.

### 객체의 const 
객체를 const로 지정하면 객체의 재할당은 불가능하지만 객체의 내용(프로퍼티, 메서드)는 변경할 수 있다.

불변객체화를 통한 객체 변경 방지
### Object.freeze() 
 객체를 불변 객체로 만든다. 하지만 객체 안의 객체(Nested Object)는 변경 가능하다. 내부 객체까지 변경 불가능하게 만들려면 Deep freeze 해야 한다.  
### Immutable.js (Facebook 제공 라이브러리) 
List, Stack, Map, OrderedMap, Set, OrderedSet, Record와 같은 영구 불변 (Permit Immutable) 데이터 구조를 제공한다.