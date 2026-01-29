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
**포인터** : 주소값을 저장하는 변수
**참조** : 주소를 내부적으로 사용하지만, 변수로 저장하지 않는 별명
    - 또 다른 이름(alias)로 보장되는 건 참조뿐이다.

ParamType이 포인터도 아니고 참조도 아니라면, 인수가 함수에 값으로 전달(pass-by-value)되는 상황인 것이다.

param이 새로운 객체라는 사실 때문에, expr에서 T가 연역되는 과정에서 다음과 같은 규칙들이 적용된다.
1. 이전처럼, 만일 expr의 형식이 참조이면, 참조 부분은 무시한다.
2. expr의 참조성을 무시한 후, 만일 expr이 const이면 그 const 역시 무시한다.

```
template<typename T>
void f(T param);

int x = 27;
const int cx = x;
const int& rx = x; // 수정만 불가능한 x 참조 느낌.

f(x);   // T && param == int
f(cx);  // T && param == int
f(rx);  // T && param == int
```
cx와 rx가 수정될 수 없다는 점은 param의 수정 가능 여부와는 전혀 무관하다. expr을 수정할 수 없다고 해서, 그 복사본까지 수정할 수 없는 것은 아니다.

**expr이 const 객체를 가리키는 포인터이고 param에 값으로 전달되는 경우**
```
template<typename T>
void f(T param);

// const char를 가리키는 포인터 자체도 cosnt
const char* const ptr = 
"Fun with pointers";

f(ptr); // call-by-value 형태로 동작함.
// 함수 내부에서만 주소가 바뀌고 원본 포인터와는 관계가 없다.
```
**const는 자기 왼쪽을 수식하고, 왼쪽에 없으면 오른쪽을 수식함.**
    - **수식한다** : 어떤 대상에 제약(의미)을 덧붙인다

**배열 인수**
- 배열과 포인터를 맞바꿔 쓸 수 있는 것처럼 보이는 환상의 주된 원인은, 많은 문맥에서 배열이 배열의 첫 원소를 가리키는 포인터로 붕괴한다(decay)는 점이다.
```
// 배열 형식의 함수 매개변수라는 것은 없다.
void myFunc(int param[]);
void myFunc(int* param);    //위와 동일한 함수
```

- 함수의 매개변수를 진짜 배열로 선언 할 수는 없지만, 배열에 대한 참조로 선언할 수는 있다.
```
template<typename T>
void f(T& param);

f(name);    //T == const char[13], expr == const char (&)[13]
```

- 배열에 대한 참조를 선언하는 능력을 이용하면 배열에 담긴 원소들의 개수를 연역하는 템플릿을 만들 수 있다.
```
// 배열 매개변수에 이름을 붙이지 않은 것은, 이 템플릿에 필요한 것은 배열에 담긴 원소의 개수뿐이기 때문이다.
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
    return N;
}

int keyVals[] = {1, 3, 7, 9, 11, 22, 35};

int mappedVals[arraySize(keyVals)];

std::array<int, arraySize(keyVals)> mappedVals;
```
- constexpr로 선언하면 함수 호출의 결과를 컴파일 도중에 사용할 수 있게 된다.

# 시작할 때 이거 지우고 위 내용 부터 학습 시작 할 것.

---
https://ddukddaksudal.tistory.com/155
https://boycoding.tistory.com/207#google_vignette
https://boycoding.tistory.com/category
p.15
