# 타임리프 thymeleaf



### 타임리프 특징
* 서버 사이드 HTML 렌더링(SSR)
	* 타임리프는 백엔드 서버에서 HTML을 동적을 렌더링 하는 용도로 사용된다.
* 네츄럴 템플릿
	* 타임리프는 순수 HTML을 최대한 유지하는 특징이 있다.
	* 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 탬플릿이라 한다.
* 스프링 통합지원

### 타임리프의 선언
`<html xmlns:th=“http://www.thymeleaf.org”>`

### Escape
* HTML 문서는 `<`,`>` 같은 특수문자를 기반으로 정의된다. 따라서 뷰 템플릿으로 HTML 화면을 생성할 때는 출력하는 데이터에 이러한 특수 문자가 있는 것을 주의해서 사용해야 한다. 

**HTML 엔티티**
웹브라우저는 `<` 를 HTML 태그의 시작으로 인식한다. 따라서 `<` 를 태그의 시작이 아니라 문자로 표현할 수 있는 방법이 필요한데, 이것을 HTML 엔티티라 한다. 이렇게 HTML에서 사용하는 특수문자를 HTML 엔티티로 변경하는 것을 Escape라고 한다.

### Unescape
이스케이프 기능을 사용하지 않으려면?
* `th:text` -> `th:utext`
* `[[…]]` -> `[(…)]`


## 변수 - SpringEL
**변수 표현식: `${…}`

**Object**
* `user.username` : user의 username 을 프로퍼티 접근 -> `user.getUsername()`
* `user.[‘username’]` : 위와 같음 ->  `user.getUsername()`
* `user.getUsername()` : user의 `getUsername()` 을 직접 호출

**List**
* `user[0].username` : List에서 첫번째 회원을 찾고 username 프로퍼티 접근 -> `list.get(0).getUsername()`
* `user[0].['username']` : 위와 같음
* `user[0].getUSeranme()` : List에서 첫번째 회원을 찾고 메서드 직접 호출

**Map**
* `userMap[‘userA’].username` : Map에서 userA를 찾고 username 프로퍼티 접근 -> `map.get(“userA”).getUsername()`
* `userMap['userA]['username']`  : 위와 같음
* `userMap[‘userA’].getUsername()` : Map에서 userA를 찾고 메서드 직접 호출


### 지역 변수 선언 
* `th:with` 를 사용하면 지역 변수를 선언해서 사용할 수 있다. 지역 변수는 선언한 테그 안에서만 사용할 수 있다.

## 기본 객체들
* `${#requeset}`
* `${#response}`
* `${#session}`
* `${#servletContext}`
* `${#locale}`

* `#request` 는 `HttpservletRequest` 에 객체가 그대로 제공되기 때문에 데이터를 조회하려면 request.getParameter(“data”) 처럼 불편하게 접근해야 한다.

## URL 링크
**쿼리 파라미터**
* `@{/hello(param1=${param1}, param2=${param2})}`
	* -> `/hello?param1=data1&param2=data2`
	* `()` 안에 있는 부분은 쿼리파라미터로 처리된다.
	**경로 변수(Path Variable)**
* `@{/helllo/{param1}/{param2}(param1=${param1}, param2=${parma2})}`
	* -> `/hello/data1/data2`
	* URL 경로상에 변수가 있으면 `()` 부분은 경로 변수로 처리된다.

**경로 변수 + 쿼리 파라미터**
* `@{/hello/{param1}(param1=${param1}, param2=${param2})}`
	* -> `/hello/data1/param2=data2`
	* 경로 변수와 쿼리 파라미터를 함께 사용할 수 있다.

상대경로, 절대경로, 프로토콜 기준을 표현할 수 있다.
* `/hello` : 절대 경로
* `hello` : 상대 경로

## 리터럴(Literals)

타임리프에서 문자 리터럴은 항상 `'` (작은 따옴표)로 감싸야한다.
`<span th:text=“’hello’”>`

그런데 문자를 항상 `'` 로 감싸는 것은 너무 귀찮은 일이다. 공백 없이 쭉 이어진다면 하나의 의미있는 토큰으로 인지해서 다음과 같이 작은 따옴표를 생략할 수 있다.
룰: `A-Z`,`a-z`,`0-9`,`[]`,`.`,`-`,`_`

`<span th:text=“hello”>`

**오류**
`<span th:text=“hello world!”></span>`
-> 문자 리터럴은 원칙상 `'` 로 감싸야 한다. 중간에 공백이 있어서 하나의 의미있는 토큰으로 인식되지 않는다.

**수정**
`<span th:text=“'hello world!'”></span>`
-> 이렇게 `'` 로 감싸면 정상 동작한다.

## 연산
* 비교연산: HTML 엔티티를 사용해야 하는 부분을 주의하자.
	 * > (gt), < (lt), >= (ge), <= (le), ! (not), == (eq), != (neq, ne)
* 조건식: 자바의 조건식과 유사하다.
* Elvis 연산자: 조건식의 편의 버전
* No-Operation: `_` 인 경우 마치 타임리프가 실행되지 않는 것 처럼 동작한다. 이것을 잘 사용하면 HTML 의 내용 그대로 활용할 수 있다. 마지막 예를 보면 데이터가 없습니다. 부분이 그대로 출력된다.

### 속성 값 설정 
**타임리프 태그 속성(Attribute)**
* 타임리프는 주로 HTML 태그에 `th:*` 속성을 지정하는 방식으로 동작한다. `th:*` 로 속성을 적용하면 기존 속성을 대체한다. 기존 속성이 없으면 새로 만든다

**속성 설정**
`th:*` 속성을 지정하면 타임리프는 기존 속성을 `th:*` 로 지정한 속승으로 대체한다. 기존 속성이 없다면 새로 만든다.
`<input type=“text” name="mock" th:name=“userA” />` 
-> 타임리프 렌더링 후 `<input type=“text” name=“userA” />` 

**속성 추가**
`th:attrappend` : 속성 값의 뒤에 값을 추가한다.
`th:attrprepend` : 속성 값의 앞에 값을 추가한다.
`th:classappend` : class 속성에 자연스럽게 추가한다.

**checked 처리**
HTML에서는 `<input type=“checkbox” name=“active” checked=“false” />` -> 이 경우에도 checked 속성이 있기 대문에 checked 처리가 되어버린다.

HTML 에서는 `checked` 속성은 `checked` 속성의 값과 상관없이 `checked` 라는 속성만 있어도 체크가 된다. 이런 부분이 `true`, `false` 값을 주로 사용하는 개발자 입장에서 불편하다.

타임리프의 `th:checked` 라는 값이 `false` 인 경우 `checked` 속성 자체를 제거한다.
`<input type=“checkbox” name=“active” th:checked=“false” />`
-> 타임리프 렌더링 후: `<input type=“checkbox” name=“active” />`