---
title: 큐(Queue)

categories:
  - Algorithm

toc: true
toc_sticky: true

date: 2025-03-02
last_modified_at: 2025-03-02
---

# 큐(Queue)

<br>

## 1. 큐의 정의

- FIFO(First in First Out, 선입선출)

<br>

▼ 표 1. 큐에서 사용하는 용어

| 명칭          | 설명                 |
| ------------- | -------------------- |
| 앞/Front/Head | 데이터가 제거되는 곳 |
| 뒤/Rear/Tail  | 데이터가 추가되는 곳 |

<br>

## 2. 큐의 성질

<br>

▼ 표 2. 큐의 시간 복잡도

| 구분                                      | 시간 복잡도          |
| ----------------------------------------- | -------------------- |
| 원소 추가                                 | O(1)                 |
| 원소 제거                                 | O(1)                 |
| 제일 앞/뒤의 원소 확인                    | O(1)                 |
| 제일 앞/뒤가 아닌 나머지 원소의 확인/변경 | 원칙적으로 불가능\*⑴ |

\*⑴ 배열을 이용하여 큐를 구현하는 경우 제일 상단이 아닌 나머지 원소의 확인/변경도 가능하도록 만들 수는 있다. 하지만 보통 큐의 응용 사례는 모든 원소의 추가/제거/제일 앞/뒤의 원소 확인 기능만을 필요로 한다.

<br>

## 3. 큐의 구현

### 3.1 (선형) 큐

- 배열을 이용하여 큐를 구현하려면 원소를 담는 배열과 인덱스를 저장하는 변수(`head`, `tail`)로 구현할 수 있다.
- 데이터 추가/제거를 반복하여 배열의 끝까지 `head`가 이동하게 되는 경우 더 이상 배열에 원소를 추가할 수 없는 문제가 생긴다.
- 선형 배열로 큐를 구현할 경우 데이터 제거 시 배열의 앞쪽 메모리가 그대로 낭비되며, 재사용할 수 없다.

<br>

▼ 코드 1. 배열을 이용한 (선형) 큐 구현

```java
public class ArrayQueue {

	private static int MAX = 1000005;
	private static int[] data = new int[MAX];
	private static int head = 0;
	private static int tail = 0;

	public static void main(String[] args) {
		push(10);
		push(20);
		push(30);

		System.out.println(front()); // 10
		System.out.println(back()); // 30

		pop();
		pop();

		push(15);
		push(25);

		System.out.println(front()); // 30
		System.out.println(back()); // 25
	}

	private static void push(int x) {
		if (tail == MAX) {
			System.out.println("push >>> Queue is full.");
			return;
		}

		data[tail++] = x;
	}

	private static void pop() {
		if (head == tail) {
			System.out.println("pop >>> Queue is empty.");
			return;
		}

		head++;
	}

	private static int front() {
		if (head == tail) {
			System.out.println("front >>> Queue is empty.");
			return -1;
		}

		return data[head];
	}

	private static int back() {
		if (head == tail) {
			System.out.println("back >>> Queue is empty.");
			return -1;
		}

		return data[tail - 1];
	}
}
```

<br>

▼ 코드 2. java.util.Queue를 이용한 (선형) 큐 구현

```java
import java.util.LinkedList;
import java.util.Queue;

public class JavaUtilQueue {

	// java.util.Queue는 인터페이스이므로 구현체로 LinkedList를 사용해야 한다.
	private static Queue<Integer> data = new LinkedList<>();

	public static void main(String[] args) {
		push(10);
		push(20);
		push(30);

		System.out.println(front()); // 10
		System.out.println(back()); // Do not support.

		pop();
		pop();

		push(15);
		push(25);

		System.out.println(front()); // 30
		System.out.println(back()); // 25
	}

	private static void push(int x) {
		data.offer(x);
	}

	private static void pop() {
		data.poll();
	}

	private static int front() {
		return data.peek();
	}

	private static int back() {
		System.out.println("back >>> Do not support.");
		return -1;
	}
}
```

<br>

### 3.2 원형 큐(Circular Queue)

큐를 배열로 구현할 때는 불필요한 메모리 낭비를 막기 위해 (개념적으로) 원형 배열로 구현을 하는 것이 좋다.

이렇게 원형의 배열을 가정하고 구현한 큐를 **원형 큐(Circular Queue)**라고 한다. (↔︎ 선형 배열)

<br>

<hr>

**▣ 원형 큐(Circular Queue)**

- 일반 배열로 구현하되 원형의 배열을 가정하고 구현된 큐를 말한다.
- 배열의 마지막에 도달하면 배열의 처음으로 다시 돌아가 순환 구조를 만든다.
  - `head`, `tail`이 각각 MAX 값이 될 때 0으로 초기화하여 구현한다.
- 큐를 배열로 구현할 때 데이터를 제거함으로써 사용되지 않는 배열 앞쪽의 메모리 낭비 문제를 해결할 수 있다.
- 원소의 개수가 배열의 크기보다 커지지 않는 한 불필요한 메모리 낭비없이 배열로 큐를 구현할 수 있다.
- 💡 Tip. 실무에서는 큐를 배열로 구현해야 할 때 원형 큐를 이용하는 것이 좋다.
- 💡 Tip. 코딩 테스트에서는 배열의 크기를 데이터 입력 최대 개수 이상으로 설정하면 원형 큐로 구현하지 않아도 된다.

<hr>

<br>

▼ 코드 3. 배열을 이용한 원형 큐 구현

```java
public class ArrayCircularQueue {

	private final static int MAX = 2;
	private static int[] data = new int[MAX];
	private static int head = 0;
	private static int tail = 0;

	public static void main(String[] args) {
		push(10);
		push(20);
		push(30);

		System.out.println(front()); // 10
		System.out.println(back()); // 20

		pop();
		pop();

		push(15);
		push(25);

		System.out.println(front()); // 15
		System.out.println(back()); // 25
	}

	private static void push(int x) {
		int queueSize = tail - head;
		if ((tail % MAX) == 0 && queueSize == MAX) {
			System.out.println("push >>> Queue is full.");
			return;
		}

		data[tail % MAX] = x;
		tail++;
	}

	private static void pop() {
		if (head == tail) {
			System.out.println("pop >>> Queue is empty.");
			return;
		}

		head++;
	}

	private static int front() {
		if (head == tail) {
			System.out.println("front >>> Queue is empty.");
			return -1;
		}

		int firstDataIdx = head >= MAX ? head % MAX : head;
		return data[firstDataIdx];
	}

	private static int back() {
		if (head == tail) {
			System.out.println("back >>> Queue is empty.");
			return -1;
		}

		int lastDataIdx = tail > MAX ? tail / MAX - 1 : tail - 1;
		return data[lastDataIdx];
	}
}
```

참고로, java.util.Queue는 `LinkedList` 구현체를 사용하였으므로 별도의 원형 큐 구현 코드를 작성하지 않았다.

<br>

## 4. 큐의 응용

- 너비 우선 탐색(BFS)
- Flood Fill
- 💡 Tip. BFS와 Flood Fill 모두 코딩 테스트에 자주 출제되므로 큐를 사용하는 방법은 잘 익혀두자.
