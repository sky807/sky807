# 래퍼 클래스(Wrapper Class) 란?

자바의 자료형은 크게 기본타입 (primitive type)과 참조타입 (reference type)으로 나누어진다.
대표적으로 기본타입은 bite, char, int, long, float, double, boolean 등이 있고 
참조타입은 class, interface 등이 있다.

프로그래밍을 하다 보면 기본 타입의 데이터를 객체로 표현해야 하는 경우가 종종 있다.
그럴때 기본타입을 객체로 다루기 위해서 사용하는 것이 **래퍼 클래스(Wrapper Class)**라고 한다.

#### 래퍼클래스의 종류

| 기본타입 (primitive type) | 래퍼 클래스 (wrapper class) |
| --------------------- | ---------------------- |
| byte                  | Byte                   |
| char                  | Character              |
| int                   | Integer                |
| float                 | Float                  |
| double                | Double                 |
| boolean               | Boolean                |
| long                  | Long                   |
| short                 | Short                  |

래퍼 클래스는 java.lang 패키지에 포함되어있는데 
char, int 타입만 각각 Character, Integer로 래퍼 클래스를 가지고있고 나머지는 기본타입의 첫 글자를 대문자로 바꾼 이름을 가지고있다.

기본 타입의 값을 포장 객체로 만드는 과정을 박싱(Boxing)이라고 하고
반대로 포장객체에서 기본타입의 값을 얻어내는 과정을 언박싱(UnBoxing)이라고 한다.

```
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = new Integer(17); // 박싱
        int n = num.intValue(); //언박싱
        System.out.println(n);
    }
}
```

JDK 1.5 부터는 박싱과 언박싱이 필요한 상황에 자바 컴파일러가 자동으로 처리해준다.
자동화된 박싱과 언박싱을 오토 박싱 (AutoBoxing) 과  오토언박싱 (AutoUnBoxing) 이라고 부른다.

```
public class Wrapper_Auto_Ex {
    public static void main(String[] args)  {
        Integer num = 17; // 오토박싱
        int n = num; // 오토 언박싱
        System.out.println(n);
    }
}
```

#### 값 비교 

아래의 코드를 보면 
래퍼 객체는 내부의 값을 비교하기 위해 == 연산자를 사용할 수 없다. 
이 연산자는 내부의 값을 비교하는 것이 아니라 래퍼 객체의 참조 주소를 비교하기 때문이다. 
비교 대상인 래퍼는 객체이므로 서로의 참조주소는 달라서 num == num2 는 false가 된다. 
하지만 내부의 값을 비교할때 equals를 사용하면 내부의 값을 비교할 수 있다. 

래퍼 클래스와 기본 자료형과의 비교는 ==, equals 연산 모두 가능하다. 
그 이유는 컴파일러가 자동으로 오토박싱 언박싱을 해주기 때문이다.

```
public class Wrapper_Ex {
    public static void main(String[] args)  {
        Integer num = new Integer(10); //래퍼 클래스1
        Integer num2 = new Integer(10); //래퍼 클래스2
        int i = 10; //기본타입
		 
        System.out.println("래퍼클래스 == 기본타입 : "+(num == i)); //true
        System.out.println("래퍼클래스.equals(기본타입) : "+num.equals(i)); //true
        System.out.println("래퍼클래스 == 래퍼클래스 : "+(num == num2)); //false
        System.out.println("래퍼클래스.equals(래퍼클래스) : "+num.equals(num2)); //true
    }
}
```



그리고 
래퍼 클래스는 제네릭 사용 시 필수로 들어간다. 
여기서 제네릭이란? <>를 생각하면 된다. 

```
public static void main(String[] args) {
  ArrayList<Integer> list = new ArrayList<>(); //정상
  ArrayList<int> list = new ArrayList<>(); //오류
}
```

제네릭에서 래퍼 클래스를 사용하면 정상적으로 작동하지만 
기본타입으로 사용할 결우는 오류가 뜬다. 

> 즉 제네릭 안에서는 객체 자료형을 쓸때 무조건 Wrapper Class를 사용해야한다.

그리고 또 한가지 차이점은 래퍼클래스는 null을 허용한다. 
하지만 기본타입은 null을 넣을 수 없다. 
