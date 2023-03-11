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
| `float`          | 4 |     3.4E +/- 38 (7 digits) | |
| `double`         | 8 |   1.7E +/- 308 (15 digits) | |
| `bool`           | 1 |          `false` or `true` | |

### Cast to `int`

```cpp
atoi()
stoi()
```

## Input/Output

## STL

### Array

### Vector

### List

### Set

### Map

### Stack

### Queue

### Bitset

## Sort

### Merge Sort

### Quick Sort

## Permutation

## Partition

## Memoization
