---
layout: post
title: (C++)Foward Declaration 순방향 선언
date: 2022-04-18 02:45 +0900
modified: 2022-04-18 02:45:49 +09:00
description: 순방향 선언
tag:
  - C++
  - Forward Declaration
  - 순방향 선언
---
> 본 게시글은 다음 Article에서 가져온 내용입니다.   
> [What are forward declarations in C++](https://stackoverflow.com/questions/4757565/what-are-forward-declarations-in-c)


## 1. C++에서 순방향 선언이 필요한 이유
컴파일러는 구문 오류나 함수 인자의 수가 잘못 전달된 것을 확인합니다.
이는 함수를 사용하기 전 해당 함수 선언을 살펴본다는 것입니다.

이러한 이유는 컴파일러가 코드를 더 잘 검증할 수 있게 해주며, 명확한 오브젝트 파일을 만들 수 있도록 합니다.   
선언을 전달할 필요가 없는 경우 컴파일러는 함수에 대한 추측 가능한 모든 정보를 .obj 파일로 생성합니다.   

그 후, 만약 링커에서 실제 호출하려는 함수가 다른 obj 파일에 존재할 경우 링커는 DLL 또는 EXE를 생성하기 위해 이것과 결합하는 함수가 무엇인지 알아내기 위한 논리적 구조를 가지고 있습니다.   
그렇지 않으면 잘못된 함수를 불러올 것입니다.   

> 따라서, 상황을 명확하게 유지하며 컴파일러가 개발자의 예측 불가능한 추측을 하지 못하도록 함수를 사용하기 전, 모든 것을 선언해야 합니다.

## 2. 두 파일 사이의 사이클 끊기
- 순방향 선언은 두 파일 사이의 사이클을 끊는데 유용합니다.   

~~~C++
#include "Wheel.h"  // Include Wheel's definition so it can be used in Car.
#include <vector>

class Car
{
    std::vector<Wheel> wheels;
};
// car.h
~~~

~~~C++
class Car;     // forward declaration

class Wheel
{
    Car* car;
};
~~~

Wheel이 Car에 대한 포인터를 가지고 있기 때문에 Car 선언이 필요하지만,   

만약 Car.h를 포함한다면 컴파일러 오류가 발생합니다.   

Car.h가 포함되면 Wheel.h가 포함되도록 시도하고 Wheel.h를 포함하는 Car.h 가 포함되도록 시도하고 이것은 사이클 형태가 됩니다. 그래서 컴파일러가 대신 오류를 발생시킵니다.   
이러한 사이클 형태를 끊을 때 순방향 선언은 유용합니다.