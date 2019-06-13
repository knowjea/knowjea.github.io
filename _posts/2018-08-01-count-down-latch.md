---
layout: post
title: "JAVA CountDownLatch (다른 쓰레드를 기다리는 방법)"
author: "Know jea"
categories: 개발
tags: [java]
image: wating.jpg
comments: true
---


멀티쓰레드 프로그래밍에서 쓰레드들이 모든 작업을 마친 후에 특정한 작업을 해야하는 경우가 있다.
이를 위해 다른 쓰레드들에서 일련의 작업이 완료 될 때까지 대기하도록 Sync를 맞춰주는 기능을 자바는 java.util.concurrent.CountDownLatch(int count)을 제공한다.

대기하는 방식은 명세서에 나와 있다.

CountDownLatch를 인스턴스화할 때 주어진 카운트로 초기화된다. await 메서드를 실행하면 해당 쓰레드는 다른 쓰레드에서 countDown 메서드 호출로 인해 현재 카운트가 0이 될 때까지 대기한다. 그 후에는 대기중인 스레드가 해제된다.


## 예제

아래 예제는 Main 쓰레드는 **latch.await()** 메소드에 의해 대기상태로 진입한다. 이 후 Worker 쓰레드들이 자신의 작업을 마친 후, **latch.countDown()** 메소드를 통해 카운트를 줄여간다. 카운트가 0이 되면 Main 쓰레드를 대기상태에서 해제된다.

{% highlight java %}
public class CountDownLatchExample {
	static final int max = 3;

	public static void testCountDownLatch() throws Exception {
		final CountDownLatch latch = new CountDownLatch(max);

		for (long i = 1; i <= max; i++) {
			new Thread(new Worker(latch, i * 100)).start();
		}

		latch.await();

		System.out.println("########### CountDownLatch End ###########");
	}

	static class Worker implements Runnable {
		private CountDownLatch latch;
		private long n;

		public Worker(CountDownLatch latch, long n) {
			this.latch = latch;
			this.n = n;
		}

		@Override
		public void run() {
			try {
				int cnt = 0;
				for (int i = 0; i < n; i++) {
					cnt++;
				}

				System.out.println(cnt);
			} catch (Exception ex) {
				ex.printStackTrace();
			} finally {
				if (this.latch == null) {
					return;
				}

				latch.countDown();
			}
		}
	}

	public static void main(String[] args) throws Exception {
		System.out.println("start");
		testCountDownLatch();
		System.out.println("end");
	}

	/**
	 * start
	 * 200
	 * 100
	 * 300
	 * ########### CountDownLatch End ###########
	 * end
	 */
}
{% endhighlight %}
