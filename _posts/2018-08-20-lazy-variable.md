---
layout: post
title:  "lazy var에 대해 알아보자!"
date:   2018-08-20 22:18:00 +0900
categories: lazy variable
---

### lazy var에 대해서 알아보자

> 코드 베이스로 UI를 개발할 때, 왜 서브 뷰 프로퍼티들에 lazy 키워드를 사용해야할까 의문점이 생겨서 정리해보려고 한다.

#### 1. 우선 lazy var는...

Lazy var는 접근될 때 클로저가 실행됨으로써 초기값이 설정된다. 즉, 어떤 클래스(or 구조체) 내부에 lazy var 프로퍼티가 있다면, 클래스(or 구조체) 인스턴스가 생성되는 시점에 프로퍼티 초기값을 설정해주지 않아도 된다. 계산 비용이 어느정도 있는 연산을 할 때 lazy var를 유용하게 쓴다고 한다.

#### 2. 그렇다면 코드로 UI를 만들 때, 서브 뷰 프로퍼티들을 lazy로 선언하는 것이 더 좋을까?

#### 3. lazy var를 사용할 때, self에 대해서 caturing을 해줘야 하는가

추가로, 클로저를 통해 초기화 되는 lazy 변수에서는 @noescape가 자동적으로 적용되기 때문에 self에 대한 순환 참조가 생기지 않는다. 변수 타입이 클로저인 경우와 분리해야한다. 변수 타입이 클로저인 경우에는 self에 대해서 순환 참조가 발생하므로 값을 캡처해야 한다.

```swift
// 클로저를 통해 초기화 되는 지연 변수
class DownLoadAppCell: UICollectionViewCell {
    lazy var labelStackView: UIStackView = {
        let stackView = UIStackView(arrangedSubviews: [appNameLabel, appSimpleCommentLabel])
        stackView.axis = .vertical
        stackView.distribution = .fill
        stackView.arrangedSubviews.forEach { $0.autoLayout }
        stackView.backgroundColor = UIColor.green
        return stackView
    }()
   ...
}
```

```swift
class HTMLElement {
    // 변수 타입이 클로저인 경우
    let name: String
    let text: String?
    
    lazy var asHTML: Void -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }
...    
}
```
#### 참고자료

https://www.bobthedeveloper.io/blog/swift-lazy-initialization-with-closures

https://outofbedlam.github.io/swift/2016/03/04/Lazy/

https://stackoverflow.com/questions/47367454/swift-lazy-var-vs-let-when-creating-views-programmatically-saving-memory



