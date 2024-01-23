---
layout: post
title:  "범주(Category)"
date:   2023-10-24 01:21:38 +0900
categories: 
    - 함수형 프로그래밍
tags:
    - 프로그래밍
    - 함수
---

## 정의
범주란 다음과 같은 데이터로 구성된다. 먼저 대상(object)가 있다. 대상이란 어떤 모임에 속한 원소이다. 이 모임에 속하는 임의의 원소 a, b를 정의역과 공역으로 삼는 사상이 존재한다.
예를 들어 f : a -> b는 a에서 b로 가는 사상 f이다.

이 데이터들은 다음을 만족해야한다
- 사상 합성의 보존: 다음을 만족하는 임의의 대상 a, b, c, d가 존재할 때
$$ a \overset{f}{\rightarrow} b \overset{g}{\rightarrow} c \overset{h}{\rightarrow} d $$
$$ h\ \circ (g \ \circ \ f)\ =\ (h \ \circ \ g)\ \circ \ f $$
- 항등 사상의 보존: 임의의 대상 a, b가 존재할 때 다음을 만족해야한다(아래 수식은 아직 이해를 못했다)
$$ id_b \ \circ \ f \ = \ f \ \circ \ id_a $$

## 참고한 자료
https://ko.wikipedia.org/wiki/%EB%B2%94%EC%A3%BC_(%EC%88%98%ED%95%99)
https://ko.wikipedia.org/wiki/%EB%8F%99%ED%98%95_%EC%82%AC%EC%83%81
https://wikidocs.net/7056
