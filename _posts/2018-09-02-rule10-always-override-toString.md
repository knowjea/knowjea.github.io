---
layout: post
title: "10. Always override toString (toSring은 항상 재정의하라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java, effective java]
comments: true
---


자바에서 최상위 객체인 Object는 하위 객체들이 오버라이딩하여 사용하도록 설계된 메서드들이 있다. (equals, hashCode, toString, clone, finalize)
그리고 이 메서드들은 일반 규약이 존재하는데 이를 따르지 않으면 자바에서 제공하는 클래스와 함께 사용할 때 제대로 동작하지 않는다.

이번 장에서는 toString 메서드의 일반 규약을 작성한다.

## toString 메서드, 일반 규약

toString 메서드는 오브젝트를 사람이 이해하기 쉬운 문자열로 표현한다. 명세서에는 다음과 같이 규약이 표현되어있다.

**toString의 결과는 간결하지만 유익한 표현이어야하며 사람이 쉽게 읽을 수 있어야합니다.**

하지만 디폴트로 제공하는 Object의 toString은 위 규약을 만족하진 않는다. Object의 toString 메서드는 아래와 같다.

{% highlight java %}
public class Object {
	// ...
	
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
}
{% endhighlight %}

클래스 이름 다음에 @ 기호와 1진수로 표현된 해시 코드가 붙은 문자열을 반환한다. 간결해 보일지는 몰라도 사람이 쉽게 읽고 이해하기는 힘들 것 같다.
따라서 명세서에는 다음과 같은 글이 이어져있다.

**모든 서브 클래스가 이 메소드를 오버라이드 하는 것을 추천합니다.**



 
[link to Rule11](https://knowjea.github.io/%EA%B0%9C%EB%B0%9C/2018/08/28/rule9-always-override-hashcode-overriding-equals.html).