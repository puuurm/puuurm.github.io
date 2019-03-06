---
layout: post
title: 뷰 프로그래밍
date: 2019-03-04 21:42:00 +0900
categories: View Programming
---

nib file이나 순수 코드를 통해 커스텀 뷰를 만들 수 있다. 이러한 커스텀 뷰를 자유자재로 프로그래밍 하기 위해서는 객체의 라이프 사이클과 레이아웃을 제대로 이해해야 한다. 이번 포스트에서는 스토리보드로 커스텀 뷰를 만들 경우 알아야 할 모든 것을 파헤쳐보자!

### 1. Nib 파일

> 스토리보드로 그래픽하게 뷰 작업을 하면 이는 리소스 파일인 nib 파일에 저장된다. 뷰 하나 하나가 객체이며 객체들의 계층구조들을 객체 그래프라 한다. 객체 그래프는 nib 파일에 저장하면 아카이브 되고, 메모리에 적재하면 언아카이브 된다. Nib 파일을 제대로 사용하기 위해 nib 객체 라이프 사이클과 메모리 관리를 위해 주의해야 할 점을 알아보자.

####nib 오브젝트 라이프 사이클

1. nib 파일의 내용과 참조된 리소스 파일까지 메모리에 적재한다. 전체 nib 객체 그래프에 대한 로우 데이터가 메모리에 적재된다.
2. nib 객체 그래프 데이터를 언아카이브하고, 객체를 초기화한다.
   - 기본적으로 객체는 ```initWithCoder:```  (swift에서는 동일 메소드가 ```init?(coder aDecoder: NSCoder)``` 이다.)메시지를 받는다.
   - iOS의 경우 NSCoding 프로토콜을 채택한 오브젝트는 ```initWithCoder:``` 메소드를 사용하여 초기화된다. 물론 ```UIView```나 ```UIViewController```를 상속한 모든 오브젝트 또한 포함한다.
3. nib 파일에 있는 객체들 사이의 모든 커넥션(actions, outlets, bindings)을 재설정한다.
   - @IBOutlet: 각각의 outlet 객체를 다시 연결하기 위해 ```setValue:forKey``` 메소드를 사용한다. 커넥션이 제대로 이루어져있지 않으면 빌드할 때 이 함수에서 에러가 났다고 알려준다.
   - @IBAction:  `setValue:forKey:` 
4. nib 파일에 있는 객체에게 `awakeFromNib` 메시지를 보낸다. 
5. 타임에 nib 파일이 화면에 보인다.

#### nib 오브젝트의 라이프타임 관리

```NSBundle``` 또는 ```NSNib``` 클래스를 통해 nib 파일의 로딩을 요청할 때마다 파일에 있는 객체들을 새롭게 복사하여 리턴한다. 즉 이전에 nib 파일 로드했을 때 생성한 객체를 재사용하지 않는다. 이런 특성으로 인해 새 객체 그래프가 필요할때 까지만 메모리에 유지되고, 사용을 마쳤을 때는 해제해야 한다. 

IBOutlet 연결할 때 strong으로 할지 weak으로 할 지 고민된 적이 있을텐데, 강한 참조는 file owner와 같은 top-level 객체에 사용하는 것이 좋다. 부모 객체가 자식 객체를 가지고 있을 때, 순환 참조의 위험을 최소화하기 위해 자식 객체는 강한 참조를 사용하지 않는 것을 권장한다.

### 2. 레이아웃 관리는 어떻게?

> 레이아웃 관리는 스토리보드나 프로그래밍 기반 커스텀 뷰나 동일하다.

#### The Runtime Interaction Model for Views





---

https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html#//apple_ref/doc/uid/10000051i-CH4-SW8

https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW1