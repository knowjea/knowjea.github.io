---
layout: post
title: "Avoid finalize (종료자 사용을 피하라)"
author: "Gyeongjae Gwon"
categories: 개발
tags: [java]
image: garbage-collection.jpg
comments: true
---

## finalize 메소드

특정 객체에 대한 참조가 더 이상 없다고 판단할 때 가비지 컬렉션이 객체의 finalize를 호출한다.
finalize 메소드는 자바의 최상위 클래스인 Object 클래스에 포함되어 있다. 
가끔 개발자는 특정 객체가 소멸될 시점에 어떠한 자원 정리를 위해 해당 메소드를 오버라이딩하여 자신만의 코드를 작성한다.


{% highlight java %}
public class Rule7 {

	@Override
	protected void finalize() throws Throwable {
		// do something
		super.finalize();
	}
}
{% endhighlight %}

<div class="post_caption">"하지만 finalize는 예측 불가능하며, 대체로 위험하고, 일반적으로 불필요하다."</div>


## 실행을 보장하지 않음

finalize 메소드는 호출되더라도 즉시 실행되라는 보장이 없으며, 반드시 실행된다는 보장도 없다.
따라서 finalize 메소드 안에서 유한한 자원에 메모리 해제, 중요한 상태정보 갱신을 하면 안 된다.

{% highlight java %}
public class FileReadUtil {
	private static FileInputStream input;

	static {
		try {
			input = new FileInputStream("c:/out.txt");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
	}

	public static int readByte() throws IOException {
		return input.read();
	}

	@Override
	protected void finalize() throws Throwable {
		try {
			// Do not do this.
			if (input != null) {
				input.close();
			}
		} finally {
			super.finalize();
		}
	}

}
{% endhighlight %}

그렇다면 명시적으로 반환해야 하는 자원을 삭제하는 방법은 무엇일까...?
파일입출력을 해봤다면 익숙한 구문인 try-catch-finally를 사용하면 된다.

{% highlight java %}
public class TryCatchFinally {
	public static void main(String[] args) throws IOException {
		FileInputStream fileInputStream = new FileInputStream("c:/out.txt");
		try {
			// do somgting
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			fileInputStream.close();
		}
	}
}
{% endhighlight %}


## 예외를 무시

finalize 메소드 안에서 예외가 발생한다고 하더라도, 해당 예외는 무시되며 스택 트레이스도 표시되지 않는다.
또한 확인은 하지 못했지만, 해당 finalize 메소드도 중단된다.
(예제에서 gc가 동작하여 finalize가 호출되도록 하였다. 하지만 system.gc는 finalize 호출을 보장하지는 않지만, 일반적으로는 잘 동작한다.)

{% highlight java %}
public class ExceptionInFinalizeTest {

	public static void main(String[] args) throws Throwable {
		ExceptionInFinalizeTest exceptionInFinalizeTest = new ExceptionInFinalizeTest();
		exceptionInFinalizeTest = null;

		// System.gc does not guarantee finalize, but generally works fine.
		System.gc();
	}

	@Override
	protected void finalize() throws Throwable {
		System.out.println("The finalize method start");

		// Exceptions are ignored.
		System.out.println(2 / 0);

		super.finalize();

		System.out.println("The finalize method end");
	}
}

{% endhighlight %}


## 성능 저하

객체에 finalize 메소드를 작성하면, 프로그램 성능이 심각하게 저하된다고 한다. (약 430배)

그러나, 실제로 확인 할 수 는 없었다. 잘 이해되지 않는 부분이다.
 
Object 클래스에 존재하므로 어떠한 클래스든 finalize 메소드를 가지고 있는 것이 아닌가?

특정한 객체에 finalize 메소드를 오버라이딩 했다고 해서 성능이 왜 저하되는지 이해가 되지 않는다. 

(도와주세요!)



## finalize 그러면 어디에 사용할까?

위에서 finalize 메소드 안에서 유한한 자원에 메모리 해제를 하면 안 된다고 하였다. 즉, 다른 방법 (try-catch-finally)으로 메모리 해제를 해야 한다.
하지만 API 개발자는 항상 클라이언트가 API를 올바르게 사용하지 않을 수도 있다는 것을 고려해야 한다.

예를 들어, FileOutputSteam 클래스를 이용하여 파일(자원)을 사용하였다면 반드시 close로 메모리를 해제해야 한다.
클라이언트가 close 메소드 사용을 잊을 것을 대비해 언제 호출될지는 모르지만 finalize에 close 메소드를 명시적으로 호출한다.

**동일한 패턴 클래스들 - FileInputStream, FileOutputSteam, Timer, Connection**

명시적 종료이므로 클라이언트를 위해 로그를 남기는 것이 좋으나 위 API 개발자는 아쉽게도 하지 않았다.

{% highlight java %}
public class FileInputStream extends InputStream{
	// ...
	
	 /**
     * Ensures that the <code>close</code> method of this file input stream is
     * called when there are no more references to it.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FileInputStream#close()
     */
    protected void finalize() throws IOException {
        if ((fd != null) &&  (fd != FileDescriptor.in)) {
            /* if fd is shared, the references in FileDescriptor
             * will ensure that finalizer is only called when
             * safe to do so. All references using the fd have
             * become unreachable. We can call close()
             */
            close();
        }
    }
}
	
{% endhighlight %}


다른 사용방법으로는 네이티브 피어(native peer) 객체를 제거할 때 사용한다. 네이티브란 자바 외의 C나 C++ 등 다른 언어로 작성된 프로그램을 나타낸다.
그렇다면 네이티브 피어란 무엇인가? 네이티브 피어란 자바에서 다른 언어로 작성된 프로그램을 다루는 메소드를 가진 클래스이다. 하지만 이런 클래스는 실제로는 C나 C++로 작성된 프로그램을 다루기 때문에
gc는 해당 프로그램을 직접 건드릴 수 없다. 따라서 네이티브 피어 객체의 finalize 메소드에 해당 프로그램 자원을 종료하는 코드를 추가해서 처리 할 수 있다.

하지만 일반적으로 이러한 API는 명시적인 종료 메소드가 존재한다. 예를 들어 네이티브 피어인 JFrame 클래스는 상위클래스인 Window 클래스의 dispose 메소드를 가지고 있다.

{% highlight java %}
public class JFrame  extends Frame implements WindowConstants,
                                              Accessible,
                                              RootPaneContainer,
                              TransferHandler.HasGetTransferHandler{ 
	// ... 
}

public class Frame extends Window implements MenuContainer {
	// ...
}

public class Window extends Container implements Accessible {
	// ...

	    public void dispose() {
        doDispose();
    }
}
	
{% endhighlight %}
 
 
## 사용할 때는 반드시 super.finalize()

finalize를 사용한다면 반드시 부모의 finalize를 호출해야 한다. 어떠한 경우라도 호출되도록 하기 위해 finally에 작성한다.
그렇지 않으면, 부모 클래스는 절대 종료되지 않는다.

{% highlight java %}
	@Override
	protected void finalize() throws Throwable {
		try {
			// do something
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			super.finalize();
		}
	}
{% endhighlight %}

만약 API 개발자가 특정한 클래스가 finalize에서 반드시 유한한 자원의 메모리를 해제해야 하는데
하위 클래스에서 super.finalize를 잊으면 문제가 생긴다. 이러한 문제를 막기 위해 Finalizer Guardian 패턴이 존재한다.

Finalizer Guardian 패턴이란 해당 클래스에 private 필드인 익명클래스를 작성하여 익명클래스에게 자원의 종료를 위임하는 방식이다.
자바에서는 객체가 종료되어야 할 때 해당 객체가 참조하는 다른 객체가 먼저 종료되어야 한다. 따라서 객체에서 정의한 익명클래스가 종료되어야 하며
이때 익명클래스에 정의한 finalize가 호출된다. 그러므로 익명클래스의 finalize 메소드 안에 상위 클래스가 필요한 종료 작업을 위임하면 된다.

{% highlight java %}
public class ParentFinalizerGuardianTest {

	public static void main(final String[] args) throws Exception {
		doIt();
		System.gc();
	}

	private final Object guardian = new Object() {
		@Override
		protected void finalize() throws Throwable {
			System.out.println("Finalize of class Parent in guardian");
			doFinalize();
		}
	};

	private void doFinalize() {
		System.out.println("Do Something");
	}

	public static void doIt() {
		ChildFinalizerGuardianTest c = new ChildFinalizerGuardianTest();
		System.out.println(c);
	}

	@Override
	protected void finalize() throws Throwable {
		System.out.println("Finalize of class Parent in finalize");
		super.finalize();
	}
}

public class ChildFinalizerGuardianTest extends ParentFinalizerGuardianTest {
	// Child class does not call super.finalize()
	@Override
	protected void finalize() {
		System.out.println("Finalize of class Child");
	}
}


{% endhighlight %}

위 코드에서 하위클래스는 super.finalize를 호출하지 않아 상위클래스의 finalize는 호출되지 않는다.
하지만 익명클래스를 사용해서 Finalizer Guardian을 작성하였으므로, 해당 finalize에서 필요한 자원 해제를 처리한다.




[link to Rule9](https://knowjea.github.io/facts/rule8-obey-the-general-overriding-equals.html).
