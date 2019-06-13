---
layout: post
title: "[JAVA] 난수(Random Number) 깊게 알아보기"
author: "Know jea"
categories: 개발
tags: [java]
comments: true
---

Effective Java의 규칙 47을 읽다가 Random 클래스에 대해서 좀 더 알아봐야 겠다고 생각해서
이 포스팅을 작성한다.

## 난수

난수란 무엇일까? 위키피디아에서는 아래와 같이 정의한다.

```
난수(亂數)란 정의된 범위 내에서 무작위로 추출된 수를 일컫는다. 난수는 누구라도 그 다음에 나올 값을 확신할 수 없어야 한다. 특히 난수의 추출에 대해 민감할 수 있는 복권 추첨에서는 난수를 발생하기 위하여 컴퓨터를 사용하지 않고 도구를 사용한다.
```

그런데 프로그래밍을 할 때 난수를 찾아보면 난수라고 하지 않고 의사(擬似)난수, 즉 Pseudorandom number라고 한다. Pseudo는 거짓, 가짜, 유사의 뜻을 가진다.
그런데 왜 Pseudo를 붙일까?

수학자들은 난수를 생성해내기 위해 여러가지 알고리즘을 만들었다. 알고리즘이 처음 난수를 만들어내려면 특정 값의 입력으로부터 시작한다.
알고리즘은 입력된 특정 값을 이용하여 계산해 난수를 생성하며, 생성된 난수를 가지고 다시 특정 값을 만들어내어 알고리즘을 반복하여 난수를 생성한다.
하지만 알고리즘은 특정 값이 동일하다면 난수의 값도 같다.

위와 같이 알고리즘을 사용한 난수 생성은 진정한 의미에서의 난수는 아니지만 그 결과값이 충분히 추측되기 어렵다면 어느정도 난수로서의 의미를 가지기 때문에
프로그래밍을 할 때는 이러한 유사, 가짜같은 난수를 사용하므로 Pseudo를 붙여 부른다.


## JAVA의 난수 알고리즘

난수를 생성하는 알고리즘은 많지만 JAVA의 난수를 생성하는 Random 클래스에서는 선형 합동법을 이용한 [선형 합동 생성기(Linear congruential generator, LCG)](https://en.wikipedia.org/wiki/Linear_congruential_generator)를 사용한다.

선형 합동법 생성기의 장점은 공식이 간단하여 빠르고 적은 메모리를 사용한다는 점이다.
단점은 이 포스팅을 쓰기 시작한 이유이다. 좀 



{% highlight java %}
public class PositivePointWithPublicField {
	// Can not limit value
	public double x;
	public double y;

	public PositivePointWithPublicField(double x, double y) {
		this.x = x;
		this.y = y;
	}
}
{% endhighlight %}

위 클래스는 x 와 y의 값이 모두 양의 값인 점을 표현하는 클래스이다. 하지만 x 와 y는 public이므로 클라이언트는 직접 조작할 수 있어 값이 음의 값이 될 수 있다.
또한 필드를 사용하는 순간에 어떤 동작이 실행되도록 만들 수도 없다. 이런 클래스는 아래와 같이 변경해주어야 한다.

{% highlight java %}
public class PositivePointWithPrivateField {
	private double x;
	private double y;

	public PositivePointWithPrivateField(double x, double y) {
		this.x = x;
		this.y = y;
	}

	public double getX() {
		// Any action can be performed
		return x;
	}

	public void setX(double x) {
		// Can limit value
		if (x <= 0) {
			throw new IllegalArgumentException();
		}
		this.x = x;
	}

	public double getY() {
		// Any action can be performed
		return y;
	}

	public void setY(double y) {
		// Can limit value
		if (y <= 0) {
			throw new IllegalArgumentException();
		}

		this.y = y;
	}
}
{% endhighlight %}

setter 메서드를 이용하여 필드의 값을 강제할 수 있으며 getter 메서드를 이용하여 필드를 사용하는 순간에 특정 동작을 수행하도록 할 수 있다.

package-private 클래스나 private 중첩 클래스는 public으로 선언해도 상관없다고 한다. 클래스가 정확히 어떤 클래스인지 기술만 한다면 말이다.
책 내용을 정확히 이해하지 못했지만, package-private 클래스나 private 중첩 클래스는 클라이언트에게 공개하는 API가 아니라 API를 구현하기 위한 클래스이므로 API 개발자가 잘만 쓰면 상관없다는 의미인 것 같다.

자바 클래스에서는 이 주제의 규칙을 지키지 않는 클래스들이 있다. 대표적으로 Point와 Dimension 클래스이다. 이런 클래스는 참고하지 않는 것이 좋다.

{% highlight java %}
public class Point extends Point2D implements java.io.Serializable {
    public int x;
    public int y;
     
    // ...
}

public class Dimension extends Dimension2D implements java.io.Serializable {
    public int width;
    public int height;
    
    // ...
}
{% endhighlight %}


[link to Rule15](https://knowjea.github.io/%EA%B0%9C%EB%B0%9C/2018/09/29/rule15.html).