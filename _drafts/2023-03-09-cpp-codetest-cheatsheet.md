---
date: 2023-03-09 12:00:00 +0900
title: 코테용 C++ Cheat Sheet
excerpt: 프로그래밍하다 이런거 좀 그만 헛갈리자
categories: dev
---

## Datatype

웬만하면 `int` 부터 쓸 수 있는지 확인. 기껏 작은 Type 쓰더라도 연산 시 `int`로
형변환 되기 때문.
[(링크)](https://learn.microsoft.com/en-us/cpp/cpp/data-type-ranges?view=msvc-170)

| Type Name        | Bytes |                    Min |                        Max |
| ----------------:|:-:| --------------------------:| --------------------------:|
| `char`           | 1 |                       -128 |                        127 |
| `unsigned char`  | 1 |                          0 |                        255 |
| `short`          | 2 |                    –32,768 |                     32,767 |
| `unsigned short` | 2 |                          0 |                     65,535 |
| `int`, `long`    | 4 |             –2,147,483,648 |              2,147,483,647 |
| `unsigned int`   | 4 |                          0 |              4,294,967,295 |
| `long long`      | 8 | –9,223,372,036,854,775,808 |  9,223,372,036,854,775,807 |
| `unsigned long`  | 8 |                          0 | 18,446,744,073,709,551,615 |
| `float`          | 4 |     3.4E +/- 38 (7 digits) |                            |
| `double`         | 8 |   1.7E +/- 308 (15 digits) |                            |
| `bool`           | 1 |          `false` or `true` |                            |

## Input/Output

C++에서는 `cin`과 `cout`을 사용한다. 특정 입출력 형식을 지정하고 싶다면 C의
`scanf`와 `printf`를 사용한다.

```cpp
int i;
char c;
char *cs;

#include <iostream>
// basic IO
cin >> i >> c;
cout << "#" << i << " " << cs << endl;

// print format
cout << fixed;
cout.precision(10);
cout << d << endl;

#include <stdio.h>
// basic IO
scanf("%d %c", i, c);
printf("#%d %s\n", i, cs);

// print format
printf("%.10lf\n", d);  // precision
printf("%05d\n", i);    // zero padding
```

입출력이 빈번한 경우에는 속도 증가를 위해 다음과 같은 코드를 출력 전 사용한다.

```cpp
/*
 * C 표준 stream과 C++ 표준 stream의 동기화를 끊는다.
 * 그러므로 C와 C++ 입출력 형식을 혼용하면 안 된다.
 */
ios_base::sync_with_stdio(false);

/*
 * cin을 cout으로부터 untie 한다.
 */
cin.tie(NULL);
```

### `int` Type Casting

정수를 각 자리수 별로 다루어야 한다면 `char *`나 `string`을 활용하자. 처음부터
자료형을 문자열로 선언하거나 `int`와 형 변환을 하면 된다.

```cpp
#include <string>

int num;
string str;
char *cs;
char c;

// int to string
str = std::to_string(num);
// string to int
num = stoi(str);

// int to char*
cs = to_string(num).c_str();
// char* to int
num = atoi(cs);

// int to char
c = num + '0';
// char to int
num = c - '0';
```

## STL

직관적으로 Primitive Type을 활용하는게 더 빠를 수도 있다. 하지만 복잡한
allocation과 handling이 필요할 것 같으면 안전하게 STL을 활용하자.

### String

```cpp
for (it=dict.begin(); it!=dict.end(); it++) {
  if (suffix.compare(*it) < 0) {
    dict.insert(it, suffix);
    break;
  }
}
```

### Array

다른 언어의 Array처럼 사용하면 된다. C++만의 차이점만 익혀두자.

```cpp
void foo(int arr1[10], int *arr2) {
  // 함수 매개변수 선언 방법: 1) fixed-sized array 2) pointer
}

// init 1. fixed-sized
int arr1[10] = {0, };

// init 2. variable-sized
int *arr2;
// ...
arr2 = new int[10];
// ...
delete[] arr2;
```

### Vector

가변 길이의 배열이면서 끝 부분의 데이터 변경이 빈번할 때 `vector`를 사용하자.

```cpp
#include <vector>

// init
vecotr<int> v;
vecotr<int> v(4);           // size 4, fill with 0
vecotr<int> v(4, 1);        // size 4, fill with 1
vecotr<int> v = {1, 2, 3};

// member function
v.at(idx);
v[idx];

v.push_back(e);
v.pop_back();

v.insert(idx, e);
v.erase(gteq, lt);

v.clear();
v.empty();
```

### List

`list`는 `node`로 구현되어 있어 있다. 배열 중간의 변경이 빈번하면 유리하지만
탐색은 불리하다.

```cpp
#include <list>

// init
list<int> l;
list<int> l(4);           // size 4, fill with 0
list<int> l(4, 1);        // size 4, fill with 1
list<int> l = {1, 2, 3};

// member function
// l.at(idx);             // not exists
// l[idx];                // not exists

l.push_back(e);
l.push_front(e);
l.pop_back();
l.pop_front();

l.insert(iter, e);    // insert before the position
l.erase(iter);        // return next iter

l.remove(e);          // remove all elements equal to e
l.remove_if(predict);
```

여기서 `iter`와 `predict`는 아래와 같이 쓰면 된다.

```cpp
// iterator
list<int>::iterator iter = l.begin();
iter++;

// predict
bool predict (const int& e) { return e == E; }
```

### Set

값이 중복되지 않는 배열이 필요할 때 `set`을 사용하면 된다.

```cpp

```
s.top() 전에는 !s.empty() 확인

### Map

### Stack

### Queue

### Bitset

## Sort

### Merge Sort

### Quick Sort

## Techniques

### Logic to Array

### Use Delta

### Memoization

## Number of Cases

### Permutation

## Partition
