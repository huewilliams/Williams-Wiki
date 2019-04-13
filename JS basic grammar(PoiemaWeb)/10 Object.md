10 Object(객체)
=========
## 객체 생성
* Built-in 함수란?  
*  Object()(Built-in 함수)로 객체 생성
* 생성자 함수
## 프로퍼티
* 표현식을 프로퍼티로 사용 (대괄호)
* 프로퍼티 값 읽기 (대괄호, 마침표)
*  객체의 프로퍼티 삭제 (delete 키워드)
### 순회
- for in (객체 순회, 배열 가능하지만 순서가 없어 비추천)
- for of(배열 순회, 배열의 순서대로 index 와 value 를 가져옴)

### 변수 할당 시 전달 되는 값
- pass-by-reference : mutable(변경 가능), 런타임 시 메모리 확보, 힙 영역에 저장, 객체는 참조 방식으로 전달됨
- pass-by-value : immutable(변경 불가능), 런타임 변수 할당 시점에 메모리 고정 확보, 스택 영역에 저장, 원시 타입은 값으로 전달됨
### 객체의 분류
- Object { Built-in Object (Standard , BOM, DOM),
Host Object }