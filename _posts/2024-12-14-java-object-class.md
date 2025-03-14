---
title: 스택(Stack)

categories:
  - Java

toc: true
toc_sticky: true

date: 2024-12-14
last_modified_at: 2024-12-14
---

이번에는 자바의 Object 클래스에 대해 학습한 내용을 정리했다. Object 클래스가 무엇이고, 왜 필요한지, 그리고 어떤 기능이 있는지를 이번 글을 통해 알아보자.

**목차**

1. java.lang 패키지
2. Object 클래스란
   1. 묵시적으로 상속받는 Object 클래스
   2. Object 클래스가 최상위 부모 클래스인 이유
3. Object 클래스와 다형성
4. Object 클래스와 배열
5. Object 클래스의 기능

# Object 클래스

Object 클래스는 모든 자바 객체의 부모 클래스를 의미하며, Java.lang 패키지에 속한다.

## 1. java.lang 패키지

- 자바 언어(Language: lang)를 이루는 가장 기본이 되는 클래스들을 보관하는 패키지
- java.lang 패키지는 모든 자바 애플리케이션에서 자동으로 import 되므로 import 구문을 사용하지 않아도 된다.

[코드 1. java.lang 패키지 import 생략]

```java
// import java.lang.System;

public class LangMain {

	public static void main(String[] args) {
		// java.lang 패키지는 모든 자바 애플리케이션에서 자동으로 import 되므로 import 구문을 생략해도 된다.
		System.out.println("hello java");
	}
}
```

[표 1. java.lang 패키지의 대표적인 클래스]

| 클래스                | 설명                                            |
| --------------------- | ----------------------------------------------- |
| Object                | 모든 자바 객체의 부모 클래스                    |
| String                | 문자열                                          |
| Integer, Long, Double | 래퍼 타입 (기본형 데이터 타입을 객체로 만든 것) |
| Class                 | 클래스 메타 정보                                |
| System                | 시스템과 관련된 기본 기능들을 제공              |

## 2. Object 클래스

- 자바에서 모든 클래스의 최상위 부모 클래스는 항상 Object 클래스이다.

### 2.1 묵시적으로 상속받는 Object 클래스

- 클래스에 상속받을 부모 클래스가 없으면 묵시적\*⑴으로 Object 클래스를 상속받는다.

  - 묵시적으로 상속받기 때문에 `extends Object`는 생략하는 것을 권장한다. (코드 2 참고)

  - 묵시적으로 상속받는 것은 곧 메모리에도 함께 생성된다는 것을 의미한다.

  - 명시적으로 상속받는 클래스가 있는 경우 Object 클래스를 직접 상속받지 않는다.

\*⑴ **개발에서의 묵시적(Implicit) vs. 명시적(Explicit)**

- 묵시적(Implicit): 개발자가 코드에 직접 기술하지 않아도 시스템 또는 컴파일러에 의해 자동으로 수행됨을 의미
- 명시적(Explicit): 개발자가 코드에 직접 기술해서 작동하는 것을 의미

[코드 2. 묵시적으로 상속받는 Object 클래스]

```java
// 부모 클래스가 없으면 묵시적으로 Object 클래스를 상속받는다.
public class Parent { // public class extends Object와 동일하다.

	public void parentMethod() {
		System.out.println("Parent.parentMethod");
	}
}
```

[코드 3. 명시적으로 상속받는 클래스]

```java
public class Child extends Parent {

	public void childMethod() {
		System.out.println("Child.childMethod");
	}
}
```

[코드 4. Object, Parent, Child 클래스의 관계]

```java
public class ObjectMain {

	public static void main(String[] args) {
		Child child = new Child();
		child.childMethod();
		child.parentMethod();

		// toString(): Object 클래스의 메서드로, 객체에 대한 정보를 제공한다.
		String string = child.toString();
		System.out.println(string);
	}
}
```

[그림 1. Object, Parent, Child 클래스의 관계]

<img src="https://github.com/user-attachments/assets/bc5070b5-6f4d-4625-95a2-fb9714dc9df3">

### 2.2 Object 클래스가 최상위 부모 클래스인 이유

- ① 공통 기능을 제공하기 위해

- ② 다형성의 기본 구현을 위해

#### ① 공통 기능을 제공하기 위해

- Object는 모든 객체에게 필요한 기본 기능을 제공한다. (예시: 객체의 정보 제공, 객체끼리의 비교 등)
- Object는 최상위 부모 클래스이므로 모든 객체는 Object가 제공하는 공통 기능을 편리하게 상속받을 수 있다.
- 따라서 프로그래밍이 단순화되고 일관성을 가질 수 있게 된다.

[표 2. Object가 제공하는 대표적인 공통 기능]

| 메서드     | 설명                                                     |
| ---------- | -------------------------------------------------------- |
| toString() | 객체의 정보를 제공한다. ('5. Object 클래스의 기능' 참고) |
| equals()   | 객체의 같음을 비교한다.                                  |
| getClass() | 객체의 클래스 정보를 제공한다.                           |

#### ② 다형성의 기본 구현을 위해

- Object 클래스는 다형성을 지원하는 기본적인 메커니즘을 제공한다.
- Object는 모든 클래스의 부모 클래스이므로 모든 객체를 참조할 수 있다. (담을 수 있다.)
  - 예시: `Object obj1 = new OtherClass1();`, `Object obj2 = new OtherClass2();` 모두 가능하다.
- 모든 자바 객체는 Object 타입으로 처리될 수 있으며, 따라서 다양한 타입의 객체를 통합적으로 처리할 수 있다.
  - 타입이 다른 객체들을 어딘가에 보관해야 한다면 Object 타입에 보관하면 된다.

## 3. Object 클래스와 다형성

- Object 클래스는 모든 클래스의 부모 클래스이다. 따라서 Object는 모든 객체를 대상으로 다형적 참조를 할 수 있다. (즉, 모든 객체를 담을 수 있다.)
- 하지만 Object 클래스는 자식 클래스에 대한 내용을 모르기 때문에 필요한 경우 각 객체에 맞는 다운캐스팅을 해야 한다.
  - 부모 클래스는 자식 클래스를 참조는 할 수 있지만 탐색은 할 수 없다. 따라서 자식 클래스의 기능을 사용하려면 다운캐스팅을 해야 한다.\*⑵
- 결과적으로 Object 클래스로 다형적 참조는 가능하지만 메서드 오버라이딩이 안 되므로 다형성을 활용하기엔 한계가 있다.

\*⑵ **Object 클래스와 다형성의 한계**

다형성을 제대로 활용하려면 다형적 참조 + 메서드 오버라이딩을 함께 사용해야 하는데, Object 클래스에 자식 클래스가 가진 커스텀 메서드를 추가할 수는 없는 문제가 있다. 따라서 메서드 오버라이딩을 활용할 수 없다. 결국 각 객체의 기능을 호출하기 위해서는 다운캐스팅을 해야 한다.

[코드 5. Object 클래스와 다형성]

```java
public class ObjectPolyExample1 {

	public static void main(String[] args) {
		Dog dog = new Dog();
		Car car = new Car();

		action(dog);
		action(car);
	}

	private static void action(Object obj) { // 모든 클래스를 담을 수 있는 Object 클래스
		// 아래 두 개의 코드는 컴파일 오류가 발생한다.
		// obj.sound(); // Object 클래스는 sound()가 없다.
		// obj.move(); // Object 클래스는 move()가 없다.

		// 객체에 맞는 다운캐스팅이 필요하다.
		// 참고로, 다운캐스팅을 좀 더 간단하게 하기 위해 아래 문법을 사용한다. (그렇지 않으면 ((Dog)obj).sound();로 호출해야 한다.)
		if (obj instanceof Dog dog) {
			dog.sound();
		} else if (obj instanceof Car car) {
			car.move();
		}
	}
}
```

참고로, Object 클래스는 모든 타입의 부모 클래스이고, 부모 클래스는 자식 클래스를 담을 수 있으므로 아래와 같이 객체를 선언할 수 있다.

[코드 6. Object 클래스의 자식 클래스 참조]

```java
Object dog = new Dog();
Object car = new Car();
```

## 4. Object 클래스와 배열

- Object 클래스는 모든 타입의 객체를 담을 수 있다. 따라서 Object[]를 통해 모든 타입의 객체를 담을 수 있는 배열을 만들 수 있다.

[코드 7. Object 클래스와 배열]

```java
public class ObjectPolyExample2 {

	public static void main(String[] args) {
		Dog dog = new Dog();
		Car car = new Car();
		Object object = new Object(); // Object 인스턴스도 만들 수 있다.

		Object[] objects = {dog, car, object};

		size(objects);
	}

	private static void size(Object[] objects) {
		System.out.println("전달된 객체의 수는: " + objects.length);
	}
}
```

[그림 2. Object 클래스와 배열]

<img src="https://github.com/user-attachments/assets/116bc4a2-16ab-4758-843c-97d0362fe7aa">

## 5. Object 클래스의 기능

### 5.1 toString()

- Object 클래스에 정의된 toString() 메서드는 모든 클래스에서 상속받아 사용할 수 있다.
- 기본적으로 패키지를 포함한 객체의 이름과 객체의 참조값(해시코드)를 16진수로 제공한다.
- 주로 메서드 오버라이딩을 통해 객체의 유용한 정보를 제공하기 위한 용도로도 많이 사용된다.
- 주로 디버깅과 로깅에 유용하게 사용된다.

#### toString() 메서드 기본 기능

[코드 8. Object 클래스의 toString() 메서드]

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

[코드 9. Object 클래스의 toString() 메서드 출력 값]

```java
public class ToStringMain1 {

	public static void main(String[] args) {
		Object object = new Object();
		String string = object.toString();

		// toString() 반환값 출력
		// Output: java.lang.Object@a09ee92
		System.out.println(string);

		// object 직접 출력
		// Output: java.lang.Object@a09ee92
		System.out.println(object);
	}
}
```

위 코드에서는 객체 출력과 toString()의 출력 값이 동일하다. 그 이유는 `System.out.println()` 메서드는 내부적으로 toString() 메서드를 호출하기 때문이다. 따라서 `System.out.println()`을 사용할 때는 toString() 메서드를 직접 호출할 필요 없이 객체를 바로 전달해도 동일한 결과를 얻을 수 있다.

#### toString() 메서드 오버라이딩

- Object 클래스의 toString() 메서드가 클래스 정보와 참조값을 제공하지만 이 정보만으로는 객체의 상태를 적절히 나타내지 못한다.
- 보통 toString() 메서드를 재정의(오버라이딩)\*⑶함으로써 보다 객체의 유용한 정보를 제공하는 것이 일반적이다.

\*⑶ **객체의 참조값 직접 출력**

만약 toString() 메서드를 재정의함으로써 toString() 메서드를 통해 객체의 참조값을 출력할 수 없게 된 경우 아래 [코드 10]을 사용하면 객체의 참조값을 출력할 수 있다.

[코드 10. 객체의 참조값 직접 출력]

```java
String refValue = Integer.toHexString(System.identityHashCode(object));
System.out.println("refValue = " + refValue);
```

[코드 11. Object 클래스의 toString() 오버라이딩]

```java
public class Dog {

	private String dogName;
	private int age;

	public Dog(String dogName, int age) {
		this.dogName = dogName;
		this.age = age;
	}

	@Override
	public String toString() {
		return "Dog{" +
				"dogName='" + dogName + '\'' +
				", age=" + age +
				'}';
	}
}
```

[그림 3. Object 클래스의 toString() 오버라이딩]

<img src="https://github.com/user-attachments/assets/3e2d3006-393a-4c9c-9213-ec2d2266a7ff">

위 그림은 [코드 11]에서 정의한 Dog 클래스의 객체를 생성한 후 toString() 메서드를 호출했을 때의 과정을 표현한 것이다.
