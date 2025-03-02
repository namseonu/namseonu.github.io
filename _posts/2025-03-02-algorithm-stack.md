---
title: 스택(Stack)

categories:
  - Algorithm

toc: true
toc_sticky: true

date: 2025-03-02
last_modified_at: 2025-03-02
---

# 스택(Stack)

## 1. 스택이란?

- 단방향 입출력 구조
- 한쪽 끝에서만 원소를 1개씩 넣거나 뺄 수 있는 자료구조
- FILO(First in Last Out, 선입후출) 혹은 LIFO(Last in First Out, 후입선출) 구조
- Restricted Structure 중 하나 (스택, 큐, 덱)
- 원칙적으로 원소의 추가/제거/제일 상단의 원소 확인이라는 기능만 제공한다.
- 재귀 함수의 동작 흐름과 같은 구조를 가진다.

## 2. 스택의 성질

| 구분                                     | 시간 복잡도          |
| ---------------------------------------- | -------------------- |
| 원소 추가                                | O(1)                 |
| 원소 제거                                | O(1)                 |
| 제일 상단의 원소 확인                    | O(1)                 |
| 제일 상단이 아닌 나머지 원소의 확인/변경 | 원칙적으로 불가능\*⑴ |

\*⑴ 배열을 이용하여 스택을 구현하는 경우 제일 상단이 아닌 나머지 원소의 확인/변경도 가능하도록 만들 수는 있다. 하지만 보통 스택의 응용 사례는 모든 원소의 추가/제거/제일 상단의 원소 확인 기능만을 필요로 한다.

## 3. 스택의 구현

### 3.1 배열을 이용하여 스택 구현하기

배열을 이용하여 스택을 구현하려면 원소를 담는 배열과 인덱스를 저장하는 변수로 구현할 수 있다.

```java
public class ArrayStack {
	private final static int MAX = 1000005;
	private static int[] data = new int[MAX]; // 원소를 담는 배열
	private static int position = 0; // 인덱스를 저장하는 변수로, 다음 원소가 추가될 떄 삽입해야 하는 위치를 가리킨다.

	public static void main(String[] args) {
		push(5);
		push(4);
		push(3);
		System.out.println(top()); // 3

		pop();
		pop();
		System.out.println(top()); // 5

		push(10);
		push(12);
		System.out.println(top()); // 12

		pop();
		System.out.println(top()); // 10
	}

	private static void push(int x) {
		data[position++] = x;
	}

	private static void pop() {
		if (position < 1) {
			System.out.println("Stack is empty.");
			return;
		}

		position--;
	}

	private static int top() {
		if (position < 1) {
			System.out.println("Stack is empty.");
			return -1;
		}

		return data[position - 1];
	}
}

```

### 3.2 연결 리스트를 이용하여 스택 구현하기

> 참고로 아래 코드는 내가 임의로 작성한 코드이므로 맞는지는 확인이 필요하다.

```java
import java.util.LinkedList;

public class LinkedListStack {
	private static LinkedList<Integer> linkedList = new LinkedList<>();
	private static int position = 0;

	public static void main(String[] args) {
		push(5);
		push(4);
		push(3);
		System.out.println(top()); // 3

		pop();
		pop();
		System.out.println(top()); // 5

		push(10);
		push(12);
		System.out.println(top()); // 12

		pop();
		System.out.println(top()); // 10
	}

	private static void push(int x) {
		linkedList.addLast(x);
		position++;
	}

	private static void pop() {
		if (linkedList.isEmpty()) {
			System.out.println("Stack is empty.");
			return;
		}

		linkedList.remove(--position);
	}

	private static int top() {
		if (linkedList.isEmpty()) {
			System.out.println("Stack is empty.");
			return -1;
		}

		return linkedList.peekLast();
	}
}
```

## 4. 스택의 응용

- 깊이 우선 탐색(DFS)
- Flood Fill
- 수식의 괄호 쌍
- 전위/중위/후위 표기법 (코딩 테스트 출제 가능성 ↓)
