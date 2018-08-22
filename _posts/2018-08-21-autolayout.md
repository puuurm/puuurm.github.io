---
layout: post
title:  "오토 레이아웃, 정확히 알고 쓰자!"
date:   2018-08-21 16:27:00 +0900
categories: autolayout
---

### 오토 레이아웃 파헤치기

> 오토레이아웃을 사용하면서 궁금했지만 넘어갔던 부분들을 짚고 넘어가려 한다.

#### 1. 오토 레이아웃이 뭔가요?

```
Auto Layout dynamically calculates the size and position of all the 
views in your view hierarchy, based on constraints placed on those views. 
```

뷰의 ```제약조건```을 기반으로, 뷰 계층구조에 있는 모든 ```뷰의 크기와 위치를 동적으로 계산```하는 것이 오토 레이아웃이다. 더 자세한건 다음 주제에서...

#### 2. 제약 조건

제약조건이 무엇인지 알아보기 위해 아래 식을 예로 들어보자.

```swift
// 1
RedView.leading = 1.0 x BlueView.trailing + 8.0
// 2
View.height = 2.0 * View.width + 0.0
// 3
View.height = 40.0
```

먼저 뷰나 레이아웃 가이드를 item이라 하고 leading, trailing, height, width 등은 attribute라고 한다. 제약조건은 두 개의 아이템 사이의 관계를 정의 할 수도 있고(1), 하나의 아이템의 두 개의 다른 attribute 사이의 관계를 정의할 수도 있으며(2), 아이템의 height이나 width에 상수 값을 설정(3)해 줄 수도 있다.

오토 레이아웃을 통해 동일한 레이아웃을 구현해내는 방식은 다양하다. 이 때 중요한 것은 일관성이다. (Here, being consistent is better than being right. You will experience fewer problems in the long run if you choose an approach and always stick with it.)

#### 3. 오토레이아웃을 왜 쓰는거죠

#### 4. 오토레이아웃 테스트하는 방법 

#### 5. 레이아웃 관련 인터페이스





