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
- **오른값에 해당하는 표현식과 왼값에 해당하는 표현식이 구분된다는 점에 근거한다.**
    - **표현식** : 평가되면 값이나 객체가 되는 코드 조각
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
const int cx = x;
const int& rx = x;

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

**경우 2: ParamType이 보편 참조임**
---
https://ddukddaksudal.tistory.com/155
https://boycoding.tistory.com/207#google_vignette
https://boycoding.tistory.com/category
p.13
