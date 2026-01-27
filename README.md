# modern_Cpp_study_note
**modern Cpp 학습 및 Github 사용법을 익히기 위한 공부 기록장**

Book name : Effective Modern C++
---
- **Item 1~6** (연역 + auto)
- **Item 7, 11, 12, 14, 15, 17** (현대 C++ 핵심 설계 감각)
- **Item 18~21** (스마트 포인터 실전)
- **Item 23~25** (move/forward 기초)
- **Item 41~42** (함수 인자 pass-by-value 판단, emplacement)
---

## 소개
**타입추론(type inference)** : auto 키워드를 사용하면 초깃값의 형식에 맟춰 선언하는 인스턴스(변수)의 형식이 '자동'으로 결정된다.
**이동 의미론(move semantics)**
**오른값에 해당하는 표현식과 왼값에 해당하는 표현식이 구분된다는 점에 근거한다.**
    **표현식** : 평가되면 값이나 객체가 되는 코드 조각
        - 리터럴까지 포함해서 거의 다 표현식이다.
- 복사하지 말고, 자원의 '소유권'을 통째로 넘겨서 비용을 없애는 규칙

```
class Widget {
public:
    Widget(Widget&& rhs)
}
```
**매개변수의 타입이 rvalue 참조라도, 매개변수라는 ‘변수명’ 자체는 항상 lvalue다.**

## 1장 형식 연역
**항목 1: 템플릿 형식 연역 규칙을 숙지하라**
    **참조 형식(reference type)** : 어떤 객체에 붙은 또 하나의 이름
    **포인터(pointer)** : 주소를 담는 변수
    **lvalue** : 객체를 가리키는 표현식
    **rvalue** : 객체가 아닐 수도 있는 값(임시 결과)을 나타내는 표현식
    **바인딩** : 어떤 표현식(값/객체)을 어떤 변수/참조/매개변수에 연결해서 쓰기로 결정하는 것

```
템플릿 의사코드
template<typename T>
void f(ParamType param);

f(expr);    // expr로부터 T와 ParamType을 연역
```

**경우 1: ParamType이 포인터 또는 참조 형식이지만 보편 참조는 아님**
```
// 예시 코드
template<typename T>
void f(T& param);

int x = 1;
const int cx = x; //값
const int& rx = x; //참조

f(x);       /* T == int, param == int& */
f(cx);      /* T == const int, param == const int& */
f(rx);      /* T == const int, param == const int& */
```

```
// 예시코드
template<typename T>
void f(const T& param);

int x = 1;
const int cx = x;
const int& rx = x;

f(x);       /* T == int, param == const int& */
f(cx);      /* T == int, param == const int& */
f(rx);      /* T == int, param == const int& */
```

```
// 예시코드
template<typename T>
void f(T* param);

int x = 27;
const int *px = &x;

f(&x);      /* T == int, param == int* */
f(px);      /* T == const int, param == const int* */
```
---

**경우 2: ParamType이 보편 참조임**
**&&** : rvalue 참조를 나타내는 문법
**보편 참조** : 넘겨진 값이 rvalue면 rvalue참조, lvalue면 lvalue참조이다.
**연역 결과** : 주어진 전제로부터 컴파일러가 논리적으로 도출한 최종 결론

**일단은 이렇게만 알아두자.**
- 매개변수에 &&가 보인다고 해서 "무조건 rvalue 참조다"라고 생각하면 안 된다.
- &&는 기본적으로 rvalue 참조 문법이 맞지만, 템플릿에서 T&& 처럼 연역 대상으로 쓰이면 보편 참조가 되어 왼값인지 오른값인지는 "호출 시점"에 결정된다.

**T는 왼값 인수가 들어와서 int&로 연역되고, param은 T&&이기 때문에 int& &&가 되는데, 참조 붕괴 규칙에 의해 int&로 정규화된다.**
```
template<typename T>
void f(T&& param);

int x = 27;
const int cx = x;
const int& rx = x;

f(x); // T == int& AND param == int&
f(cx); // T == const int& AND param == const int& 
f(rx); // T == const int& AND param == const int&
f(27); // T == int AND param == int&&
```
**일단 지금은, 보편 참조 매개변수에 관한 형식 연역 규칙들이 왼값 참조나 오른값 참조 매개변수들에 대한 규칙들과는 다르다는 점만 기억하기 바란다.**

---

**경우 3: ParamType이 포인터도 아니고 참조도 아님**
---
https://ddukddaksudal.tistory.com/155
https://boycoding.tistory.com/207#google_vignette
https://boycoding.tistory.com/category
p.14
