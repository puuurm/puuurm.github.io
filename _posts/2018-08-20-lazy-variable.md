---
layout: post
title:  "lazy var에 대해 알아보자!"
date:   2018-08-20 22:18:00 +0900
categories: lazy variable
---

### lazy var에 대해서 알아보자

> 코드 베이스로 UI를 개발할 때, UICollection/TableViewCell을 상속받은 커스텀 셀 클래스 내부에 클로저를 통해서 초기화 하는 서브 뷰 프로퍼티들을 선언했다. 이 때 서브 뷰 프로퍼티들에 lazy 키워드를 사용하는 것이 좋을까 의문점이 생겨서 정리해 보려고 한다. 

#### 1. 우선 lazy var는...

Lazy var는 접근될 때 클로저가 실행됨으로써 초기값이 설정된다. 즉, 어떤 클래스(or 구조체) 내부에 lazy var 프로퍼티가 있다면, 클래스(or 구조체) 인스턴스가 생성되는 시점에 프로퍼티 초기값을 설정해주지 않아도 된다. 최초의 접근에만 클로저가 실행되어 초기화 된다.

#### 2. lazy var는 언제 쓰는게 좋을까?

공식 문서를 찾아보면 아래와 같이 나와있다.

```
Lazy properties are useful when the initial value for a property is dependent on 
outside factors whose values are not known until after an instance’s initialization 
is complete. Lazy properties are also useful when the initial value for a property 
requires complex or computationally expensive setup that should not be performed unless 
or until it is needed.
```

Lazy var는 인스턴스 초기화가 완료된 이후 외부 요인에 의해 초기값이 결정되는 경우, 복잡하거나 계산 비용이 많이 드는 설정을 필요로 하는 경우에 사용하면 좋다. 지금 딱 떠오르는 건 테이블 생성이나 파일 읽기 등이 있다.

#### 3. 그렇다면 코드로 UI를 만들 때, 서브 뷰 프로퍼티들을 lazy로 선언하는 것이 더 좋을까?

서브 뷰 프로퍼티를 클로저로 초기화하는 상수로 만들것인가 lazy 변수로 만들것인가? 디버깅 결과, 둘의 차이는 인스턴스 생성 전에 클로저가 실행되는지, 프로퍼티에 접근할 때 클로저가 실행돼 초기화 되는지 여부이다. 

만약 서브뷰가 항상 보여져야 한다면 lazy를 사용했을 때 이점이 적을 것이고, 특정 상황에서만 보여져야 한다면 lazy를 사용했을 때 이점이 있을 수 있다. 뷰 컨트롤러를 생성할 때, 해당 서브뷰를 만들지 않고 생성이 가능하므로 메모리와 성능 측면에서 좋을 것 이다.

#### 4. lazy var를 사용할 때, self에 대해서 caturing을 해줘야 하는가

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
// 애플 공식문서에 나오는 코드의 일부분이다.
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
      deinit {
        print("\(name) is being deinitialized")
    }
}
```
순환 참조가 정말 안생기는지 궁금해서 실험 해봤다.

```swift
class Apple {
    private var count: Int = 10
    lazy var counting: String = {
        return "the number of apple is \(self.count)"
    }()
    deinit {
        print("delete apple")
    }
}

var apple: Apple? = Apple()
apple?.counting
apple = nil // delete apple
```

```swift
class Lemon {
    private var count: Int = 5
    lazy var counting: () -> String = {
        return "the number of lemon is \(self.count)"
    }
    deinit {
        print("delete lemon")
    }
}

var lemon: Lemon? = Lemon()
lemon?.counting() 
lemon = nil // 소멸자가 호출되지 않는다.
```



#### 참고자료

[애플공식문서-Properties](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID254)

[https://www.bobthedeveloper.io/blog/swift-lazy-initialization-with-closures](https://www.bobthedeveloper.io/blog/swift-lazy-initialization-with-closures)

[https://outofbedlam.github.io/swift/2016/03/04/Lazy/](https://outofbedlam.github.io/swift/2016/03/04/Lazy/)

[https://stackoverflow.com/questions/47367454/swift-lazy-var-vs-let-when-creating-views-programmatically-saving-memory](https://stackoverflow.com/questions/47367454/swift-lazy-var-vs-let-when-creating-views-programmatically-saving-memory)

[http://seorenn.blogspot.com/2015/02/swift-lazy-stored-properties.html](http://seorenn.blogspot.com/2015/02/swift-lazy-stored-properties.html)

[https://medium.com/@abhimuralidharan/lazy-var-in-ios-swift-96c75cb8a13a](https://medium.com/@abhimuralidharan/lazy-var-in-ios-swift-96c75cb8a13a)

[https://medium.com/@hossamghareeb/lazy-properties-in-ios-39cda14ec14f](https://medium.com/@hossamghareeb/lazy-properties-in-ios-39cda14ec14f)

