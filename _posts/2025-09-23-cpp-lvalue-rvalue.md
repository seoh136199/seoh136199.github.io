---
title: "[C++] lvalue와 rvalue"
hidden: true

categories:
  - Computer Science
tags:
  - [C++]

toc: true
toc_sticky: true

date: 2025-09-25
last_modified_at: 2024-09-25
---

이 글은 현재 작성 중입니다.

# • 값 범주의 종류 •

## 용어 설명

```planetext
                    +--------------+
                    |  expression  |
                    +--------------+
                            |
                  +---------+---------+
                  ↓                   ↓
            +-----------+       +-----------+
            |  glvalue  |       |  rvalue   |
            +-----------+       +-----------+
                  |                   |
        +---------+------+     +------+---------+
        ↓                ↓     ↓                ↓
  +-----------+       +-----------+       +-----------+
  |  lvalue   |       |   xvalue  |       |  prvalue  |
  +-----------+       +-----------+       +-----------+
```

  <br>

아래는 [MS 공식 문서](https://learn.microsoft.com/ko-kr/cpp/cpp/lvalues-and-rvalues-visual-cpp?view=msvc-170)에 작성된 값 범주에 대한 C++17에서의 정의이다.

- **glvalue**: 평가에서 개체, 비트 필드 또는 함수의 ID를 결정하는 식
- **prvalue**: 계산에서 개체 또는 비트 필드를 초기화하거나 표시되는 컨텍스트에 지정된 대로 연산자의 피연산자 값을 계산하는 식
- **xvalue**: 리소스를 재사용할 수 있는 개체 또는 비트 필드를 나타내는 glvalue
- **lvalue**: xvalue가 아닌 glvalue
- **rvalue**: prvalue 또는 xvalue
  <br>

난해하다. 우리가 일반적으로 자주 사용하는 용어인 lvalue와 rvalue에 대한 직접적인 정의는 없고, glvalue, prvalue, xvalue를 이용해 정의되어 있음을 볼 수 있다.
조금 더 풀어서 설명해보자면 아래와 같다.

- **glvalue**: **주소값을 갖는** 식. 일반적으로 생각하는 lvalue와 유사하나, xvalue를 포함하는 개념이다. 일반적인 변수, 배열 원소, 함수 이름, 문자열 리터럴 등이 이에 속한다.
- **prvalue**: **주소값을 갖지 않고**, 값 자체를 주는 식. 일반적으로 생각하는 rvalue와 유사하나, xvalue를 포함하지 않는 개념이다. 기본 타입의 리터럴 등이 이에 속한다.
- **xvalue**: glvalue 중 **곧 소멸하여 자원을 재사용**할 수 있는 식. `std::move()`의 결과 등이 이에 속한다.
  <br><br>

## 주의사항

값 범주에 대해 공부할 때 주의해야 할 점은, 이는 **표현식**에 대한 개념이라는 것이다.

- 문맥과 함께 작성된 표현식은 항상 '타입'과 '값 범주'를 함께 갖는다. 즉, 어떤 표현식의 값 범주를 판단할 때 타입과 혼동하면 안 된다.
- 함수 이름이 glvalue에 속한다고 작성했으나, 이는 함수 이름이 함수 그 자체를 가리키는 glvalue라는 의미이며, 함수의 호출식과는 구분해야 한다. 함수의 호출식은 그 함수의 반환값에 따라 값 범주가 결정된다.
  <br>

설명만으로는 감이 잘 오지 않으므로, 예시 코드를 보며 자세히 확인해보겠다.
<br><br>

---

# • lvalue의 예시 •

## 이름 붙은 변수, 대입식

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 10, b = 20;
    cout << &a << '\n'; //0000007784EFFB34
    (a = b) = 99;
    cout << a << '\n'; //99
    cout << &a << ' ' << &(a = 1) << '\n'; //0000007784EFFB34 0000007784EFFB34
}
```

- `a`는 lvalue로, 주소 연산자(`&`)를 사용해 주소값을 얻을 수 있다.
- 대입식 `(a = b)`은 다시 `a`를 가리키는 lvalue이다. 그 자체에 값을 할당할 수 있다.
- 대입식 `(a = b)`은 lvalue이므로 `a`와 마찬가지로 주소 연산자(`&`)를 사용해 주소값을 얻을 수 있다. 또한 그 주소값은 `a`와 동일하다.
  <br><br>

## 배열 원소, 포인터 역참조

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[3] = { 1,2,3 };
    int* p = arr;
    cout << &arr[0] << ' ' << &arr[1] << '\n'; //000000464A4FF9D8 000000464A4FF9DC
    cout << &(*p) << '\n'; //000000464A4FF9D8
}
```

- 배열 `arr`의 각 원소(`arr[0]`, `arr[1]` 등)는 lvalue로, 주소 연산자(`&`)를 사용해 주소값을 얻을 수 있다.
- 포인터 역참조 `*p` 또한 lvalue로, 다시 주소 연산자(`&`)를 붙여 주소값을 얻을 수 있다.

<br><br>

## 함수 이름, 함수가 lvalue 참조를 반환

```cpp
#include <iostream>
using namespace std;

int& first(int* a) { return a[0]; }

int main() {
    auto *pf = &first;

    int a[2] = { 7,8 };
    first(a) = 42;
    pf(a) = 100;

    cout << a[0] << '\n'; //100
    cout << &first(a) << ' ' << &a[0] << '\n'; //000000DA922FF688 000000DA922FF688
}
```

- 함수 이름 `first`는 함수 타입의 lvalue로, 주소 연산자(`&`)를 사용해 주소값을 얻을 수 있다.
- 함수 `first()`의 반환 타입이 lvalue 참조인 `int&`이므로, 함수 호출식 `first(a)`은 lvalue이며 그 자체에 값을 할당할 수 있다.
- 함수 포인터 `pf`를 통해 호출해도 동일하게 lvalue 참조를 반환하므로 `pf(a) = 100;` 또한 유효하다.
- 함수 `first()`의 호출식이 lvalue이므로 주소 연산자(`&`)를 사용할 수 있다. `&first(a)`는 `a[0]`의 주소와 동일하다.
  <br><br>

## 중제목중제목

```cpp
코드코드
```

- 설명설명
  <br><br>

- ***

# • rvalue •

```cpp

```

---

# • xvalue •

```cpp

```

---

<!--
- 값 범주의 종류
	- 도식
  - lvalue
  - rvalue
  - xvalue


https://chatgpt.com/g/g-p-68d2459d1894819197da9915df59f534/c/68d49cc0-0e3c-8320-a6ab-36ea69c4a38d

https://chatgpt.com/g/g-p-68d2459d1894819197da9915df59f534/c/68d6246f-f2a4-832a-8999-ba089918c3c1


- 생성과 대입
  - 이동 생성
  - 복사 생성
  - 이동 대입
  - 복사 대입
	- trivial type

- i++, ++i
  - i++ (어셈블리 포함)
  - ++i (어셈블리 포함)
  - i++는 임시 공간이 생긴다? 레지스터 vs 메모리

https://blog.naver.com/luku756/221808884092
-->
