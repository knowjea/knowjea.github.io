---
layout: post
title: "8. Obey the general contract when overriding equals (equals를 재정의할 때는 일반 규약을 따르라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java, effective java]
image: obey-the-rules.png
comments: true
---


자바에서 최상위 객체인 Object는 하위 객체들이 오버라이딩하여 사용하도록 설계된 메소드들이 있다. (equals, hashCode, toString, clone, finalize)
그리고 이 메소드들은 일반 규약이 존재하는데 이를 따르지 않으면 자바에서 제공하는 클래스와 함께 사용할 때 제대로 동작하지 않는다.

이번 장에서는 equals 메소드의 일반 규약을 작성한다.

 
 
## equals 메소드

equals 메소드는 객체와 다른 객체가 동일한 지 여부를 반환한다. equals를 오버라이딩 하지 않았을 경우 최상위객체인 Object의 메소드가 호출된다.
이 경우 오직 자기 자신하고만 같다. (메모리 주소가 동일)

{% highlight java %}
public class Object {

    public boolean equals(Object obj) {
        return (this == obj);
    }
}
{% endhighlight %}

```
하지만, 아래의 조건 가운데 하나라도 만족되면 그래도 된다.
```
 
 
 
## equals를 오버라이딩 하지 않아도 되는 경우

* **각각의 객체가 고유하다.** 클래스 특성상 객체가 고유할 수 밖에 없는 경우에는 오버라이딩 할 필요가 없다. 예를 들어 Thread 같은 클래스가 있다.
 
 
 
* **클래스에 논리적 동일성 검사 방법이 있건 없건 상관없다.** 클래스 특성상 equals 메소드가 있어봤자 사용할 일이 거의(절대?) 없을 때 오버라이딩 하지 않는다. 예를 들어 Random 클래스는 난수열을 생성하는 클래스인데 두 개의 Random 객체가 동일한 난수열을 만드는지 알아야하는 클라이언트는 없을거라고 생각하기 때문에 equals 메소드를 오버라이딩 하지않았다.
 
 
 
* **상위 클래스에서 재정의한 equals가 하위 클래스에서 사용하기에도 적당하다.** 예를들어 대부분의 Set, List, Map 클래스들은 각각 AbstractSet, AbstractList, AbstractMap의 equals를 사용한다. 
 
 
 
* **클래스가 private 또는 protect로 선언되었고, equals 메소드를 호출할 일이 없다.** 이 경우는 클라이언트가 실수로 호출 할 수 있으므로 경고를 알리도록 오버라이딩 한다.
{% highlight java %}
	@Override
	public boolean equals(Object obj) {
		throw new AssertionError();
	}

{% endhighlight %}

 
 
## equals의 일반 규약 (Object 클래스 명세)

* **It is reflexive: for any non-null reference value x, x.equals(x) should return true.**

반사성이란 모든 객체는 자기 자신과 같아야 한다. 이 규약을 의도적으로 깨트릴수는 있으나, 그럴 이유도 없고 지키지 않기도 힘들다.
아래 코드는 의도적으로 깨트렸다.

{% highlight java %}
public class ViolatingReflexiveTest {
	int i;

	public static void main(String[] args) {
		ViolatingReflexiveTest test = new ViolatingReflexiveTest();
		test.i = 1;

		System.out.println(test.equals(test));
	}

	@Override
	public boolean equals(Object obj) {
		return ((ViolatingReflexiveTest) obj).i < this.i;
	}
}
{% endhighlight %}



* **It is symmetric: for any non-null reference values x and y, x.equals(y) should return true if and only if y.equals(x) returns true.** 

대칭성이란 X가 Y가 같으면, Y도 X와 같아야 한다. 이 규약은 쉽게 깨질 수 있다. 예를 들어 동일한(비슷한) 의미를 가진 다른 클래스인 X와 Y가 존재한다고 하자. X는 Y와 의미가 비슷하기 때문에 자기 자신클래스 뿐만 아니라 Y 클래스와 호환되도도록 equals 메소드에서 Y 클래스를 입력받아서 처리하도록 설계했다. 하지만, Y는 X 클래스가 구현되기 전에 구현된 클래스고 자기자신인 Y만 입력받아서 equals 메소드를 처리하도록 하였다. 따라서 X.equals(Y)는 참일 수 있지만 Y.equlas(X)는 X가 자기자신 클래스가 아니기 때문에 거짓을 항상 반환할 것이다.


{% highlight java %}
public class XClass {
	public int age;

	@Override
	public boolean equals(Object obj) {
		if (obj instanceof XClass) {
			return age == ((XClass) obj).age;
		}

		if (obj instanceof YClass) {
			return age == ((YClass) obj).years;
		}
		return false;
	}

}

public class YClass {
	public int years;

	@Override
	public boolean equals(Object obj) {
		if (obj instanceof YClass) {
			return years == ((YClass) obj).years;
		}
		return false;
	}

	public static void main(String[] args) {
		XClass xClass = new XClass();
		YClass yClass = new YClass();

		xClass.age = 10;
		yClass.years = 10;

		System.out.println(xClass.equals(yClass)); // true
		System.out.println(yClass.equals(xClass)); // false

	}
}
{% endhighlight %}
 
 
* **It is transitive: for any non-null reference values x, y, and z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) should return true.**

추이성이란 
* **It is consistent: for any non-null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.**
 
 
 
* **For any non-null reference value x, x.equals(null) should return false.**

