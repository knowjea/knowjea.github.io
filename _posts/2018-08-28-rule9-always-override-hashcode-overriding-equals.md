---
layout: post
title: "9. Always override hashCode when you override equals (equals를 재정의할 때는 반드시 hashCode도 재정의하라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java, effective java]
comments: true
---


자바에서 최상위 객체인 Object는 하위 객체들이 오버라이딩하여 사용하도록 설계된 메서드들이 있다. (equals, hashCode, toString, clone, finalize)
그리고 이 메서드들은 일반 규약이 존재하는데 이를 따르지 않으면 자바에서 제공하는 클래스와 함께 사용할 때 제대로 동작하지 않는다.

이번 장에서는 equals 메서드의 일반 규약을 작성한다.

 
 
## equals 메서드

equals 메서드는 객체와 다른 객체가 동일한 지 여부를 반환한다. equals를 오버라이딩 하지 않았을 경우 최상위객체인 Object의 메서드가 호출된다.
이 경우 오직 자기 자신하고만 같다. (메모리 주소가 동일)

{% highlight java %}
public class Object {

    public boolean equals(Object obj) {
        return (this == obj);
    }
}
{% endhighlight %}

```
하지만, 아래의 조건 가운데 하나라도 만족하면 그래도 된다.
```
 
 
 
## equals를 오버라이딩 하지 않아도 되는 경우

* **각각의 객체가 고유하다.** 클래스 특성상 객체가 고유할 수밖에 없는 경우에는 오버라이딩 할 필요가 없다. 예를 들어 Thread 같은 클래스가 있다.
 
 
 
* **클래스에 논리적 동일성 검사 방법이 있건 없건 상관없다.** 클래스 특성상 equals 메서드가 있어봤자 사용할 일이 거의(절대?) 없을 때 오버라이딩 하지 않는다. 예를 들어 Random 클래스는 난수열을 생성하는 클래스인데 두 개의 Random 객체가 동일한 난수열을 만드는지 알아야 하는 클라이언트는 없을 거라고 생각하기 때문에 equals 메서드를 오버라이딩 하지 않았다.
 
 
 
* **상위 클래스에서 재정의한 equals가 하위 클래스에서 사용하기에도 적당하다.** 예를들어 대부분의 Set, List, Map 클래스들은 각각 AbstractSet, AbstractList, AbstractMap의 equals를 사용한다. 
 
 
 
* **클래스가 private 또는 protect로 선언되었고, equals 메서드를 호출할 일이 없다.** 이 경우는 클라이언트가 실수로 호출 할 수 있으므로 경고를 알리도록 오버라이딩 한다.
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

대칭성이란 X가 Y가 같으면, Y도 X와 같아야 한다. 이 규약은 쉽게 깨질 수 있다. 예를 들어 동일한(비슷한) 의미를 가진 다른 클래스인 X와 Y가 존재한다고 하자. X는 Y와 의미가 비슷하기 때문에 자기 자신 클래스 뿐만 아니라 Y 클래스와 호환되도록 equals 메서드에서 Y 클래스를 입력받아서 처리하도록 설계했다. 하지만, Y는 X 클래스가 구현되기 전에 구현된 클래스고 자기 자신인 Y만 입력받아서 equals 메서드를 처리하도록 하였다. 따라서 X.equals(Y)는 참일 수 있지만 Y.equals(X)는 X가 자기 자신 클래스가 아니기 때문에 거짓을 항상 반환할 것이다.


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

추이성이란 수학에서 많이 봤던 "a=b이고 b=c이면 a=c이다."과 동일한 의미이다. 먼저 이 예제를 보이기 위해 java.awt.Point 클래스를 상속하고, 색상을 추가로 가지는 ColorPoint를 구현한다.

ColorPoint의 equals 메서드는 자신과 동일한 객체만 검사하며 부모 클래스인 Point의 equals 메서드와 색상을 비교하여 객체의 동일 여부를 판단하도록 구현하였다.

하지만 이는 대칭성(symmetric)을 위반한다. Point를 ColorPoint와 비교하면 좌표 값(x,y)를 비교하지만, ColorPoint는 자신과 동일한 객체만 검사하므로 부모인 Point가 검사대상이 될 경우 false다. 

{% highlight java %}
public class ColorPoint extends Point {
	private final Color color;

	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	public ColorPoint(Point point, Color color) {
		super(point);
		this.color = color;
	}

	@Override
	public boolean equals(Object obj) {
		if (!(obj instanceof ColorPoint)) {
			return false;
		}
		return super.equals(obj) && ((ColorPoint) obj).color == color;
	}

	public static void main(String[] args) {
		Point point = new Point(1, 2);
		ColorPoint colorPoint = new ColorPoint(point, Color.RED);

		// Symmetry violation
		System.out.println(point.equals(colorPoint)); // true
		System.out.println(colorPoint.equals(point)); // false
	}

}
{% endhighlight %}

그렇다면 위 equals 메서드가 대칭성을 위반하지 않도록 하려면 어떻게 해야 할까? 일단 부모인 Point가 검사 대상이 되어도 false를 반환하지 말아야 하며 Point가 검사 대상일 경우에는 색상은 비교 안 하면 된다.

아래와 같이 equals 메서드를 변경하면 되는 것이다. **하지만, 이것은 추이성을 위반한다.**
{% highlight java %}
public class ColorPoint extends Point {
	// ...
	
	@Override
	public boolean equals(Object obj) {
		if (!(obj instanceof Point)) {
			return false;
		}

		if (!(obj instanceof ColorPoint)) {
			return obj.equals(this);
		}

		return super.equals(obj) && ((ColorPoint) obj).color == color;
	}
	
	public static void main(String[] args) {
		Point point = new Point(1, 2);
		ColorPoint colorPoint = new ColorPoint(point, Color.RED);

		// Symmetry
		System.out.println(point.equals(colorPoint)); // true
		System.out.println(colorPoint.equals(point)); // true

		ColorPoint blueColorPoint = new ColorPoint(point, Color.BLUE);

		// Transitivity violation
		System.out.println(colorPoint.equals(point)); // true
		System.out.println(point.equals(blueColorPoint)); // true
		System.out.println(colorPoint.equals(blueColorPoint)); // false
	}
{% endhighlight %}
 
**사실 부모를 상속하는 자식 클래스가 자신의 변수를 추가하면 추이성 규약을 어기지 않을 수 없다. 즉 어쩔 수 없다는 것이다.**

어쩔 수 없으므로 아예 다른 방법으로 equals 메서드를 구현해서 해결할 수 있다는 소문이 있다고 한다. equals 메서드를 구현할 때 instanceof 대신 getClass 메서드를 사용하는 방법이다.
하지만 이는 **리스코프 대체 원칙(Liskov substitution principle)을 위반한다.**  [리스코프 대체 원칙 참고](https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99).

리스코프 대체 원칙은 간단하게 말하면 자식의 인스턴스를 부모 객체에 대입하여도 부모 메서드의 결과는 동일하다는 의미이다.

위 코드에서 예를 들면 ColorPoint 인스턴스를 Point 객체에 대입하여도 equals 메서드의 결과는 동일하다는 의미이다.
하지만 소문처럼 instanceof를 getClass로 대체하면 Point의 equals 메서드에 자식 ColorPoint를 넣게 되면 false가 된다.
이는 리스코프 대체 원칙을 위반한다.

{% highlight java %}
    public boolean equals(Object obj) {
        if (obj == null || obj.getClass() != getClass()) {
            Point pt = (Point)obj;
            return (x == pt.x) && (y == pt.y);
        }
        return false;
    }
{% endhighlight %}

결국 부모를 상속하는 자식 클래스가 자신의 변수를 추가하면 추이성 규약을 어기지 않는 방법은 없다. 방법이 없지만, 피할 수는 있다.
[16. 계승하는 대신 구성하라]() 규칙을 사용하는 것이다. 즉 Point를 상속하지 말고 하나의 필드로 만들어서 사용하는 방법이다.

{% highlight java %}
public class CorrectColorPoint {
	private final Point point;
	private final Color color;

	public CorrectColorPoint(int x, int y, Color color) {
		this.point = new Point(x, y);
		this.color = color;
	}

	@Override
	public boolean equals(Object obj) {
		if (!(obj instanceof CorrectColorPoint)) {
			return false;
		}
		CorrectColorPoint cp = (CorrectColorPoint) obj;
		return cp.point.equals(point) && cp.color.equals(color);
	}
}
{% endhighlight %}

* **It is consistent: for any non-null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified.**
 
일관성이란 일단 같다고 판정된 객체들은 이후에 변화가 없으면 계속 같아야 한다는 것이다. 즉 신뢰성이 보장되지 않는 자원들을 비교하면 안된다.

예를 들어 java.net.URL의 equals 메서드는 URL에 대응되는 IP 주소를 비교하여 동일 여부를 판단하였다. 하지만 IP주소는 네트워크상에서 언제든 변경될 수 있으므로 일관성을 보장하지 않는다. 

아래 코드는 URL의 equals에 사용하는 메서드의 일부이다.
 
{% highlight java %}
    protected boolean hostsEqual(URL u1, URL u2) {
        InetAddress a1 = getHostAddress(u1);
        InetAddress a2 = getHostAddress(u2);
        // if we have internet address for both, compare them
        if (a1 != null && a2 != null) {
            return a1.equals(a2);
        // else, if both have host names, compare them
        } else if (u1.getHost() != null && u2.getHost() != null)
            return u1.getHost().equalsIgnoreCase(u2.getHost());
         else
            return u1.getHost() == null && u2.getHost() == null;
    }
{% endhighlight %}
 
* **For any non-null reference value x, x.equals(null) should return false.**

obj.equals(null)는 항상 false를 반환해야 한다. instanceof에 null을 체크할 경우, 항상 false를 반환한다.
따라서 equals 메서드에서 instanceof를 항상 사용하여 자동으로 이 규약을 지키도록 하자.


## equals 메서드 구현 순서

설명한 내용을 바탕으로 equals 메서드 구현 방법은 다음과 같다.

1. == 연산자를 사용하여 인자가 자기 자신인지 제일 먼저 검사한다. 성능을 위함이다.
2. instanceof 연산자를 사용하여 인자의 자료형이 정확한지 검사한다. 이 부분에서 5번 규약인 null을 처리할 수 있다.
3. 인자의 자료형을 캐스팅한다.
4. 동일함을 검사하는 필드를 각각 비교한다.
	1. float와 double은 각각 Float.compare와 Double.compare를 사용하여 비교한다.
	2. 필드의 비교 순서는 다를 가능성이 가장 높거나 비교 비용이 낮은 필드부터 비교하는 게 좋다.
5. 마지막으로 equals의 일반 규약을 만족하는지 검사한다.

예제는 다음과 같다.

{% highlight java %}
	@Override
	public boolean equals(Object obj) {
		// 1.
		if (obj == this) {
			return true;
		}
		// 2.
		if (!(obj instanceof CorrectColorPoint)) {
			return false;
		}
		// 3.
		CorrectColorPoint cp = (CorrectColorPoint) obj;

		// 4.
		return cp.point.equals(point) && cp.color.equals(color);
	}
{% endhighlight %}

