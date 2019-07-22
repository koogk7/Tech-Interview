# Stream API



### Stream API 란

Java 8에서 새롭게 추가된 Api로 **함수형 인터페이스(람다식)을 적용하여 컬렉션과 같은 저장요소를 반복적으로 처리**할 수 있는 기능이다.  java에서 완전한 Funtional Programming은 아닐지라도 비슷하게 흉내라도 낼 수 있게 도와준다. Stream Api는 다음과 같은 특성을 가진다.

- Stream은 **Immutable** 하다. 다시 말해 원본의 데이터를 변경하지 않는다.
- Stream의 연산은 **layz** 하다. 즉 필요 할 때만 연산함으로 효율적인 처리가 가능하다.
- Stream은 **재사용이 불가능**하다.

Stream API를 사용함으로서 얻는 이점은 **코드의 가독성이 높아지고,  실수의 여지를 줄여준다**. 아래의 예제를 살펴보자.

단순히 주어진 배열중 3~9사이의 숫자 중 두배했을 때 10이 넘는 첫번째 인자를 찾는 코드이다

```java
final List<Integer> NUMBERS = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
Integer result = null;
for (final Integer number : NUMBERS) {
  if (number > 3 && number < 9) {
    final Integer newNumber = number * 2;
    if (newNumber > 10) {
      result = newNumber;
      break;
    }
  }
}
System.out.println("Imperative Result: " + result);
```

코드가 나타내는 로직이 한눈에 들어오지 않고, if문과 여러 변수들의 혼용으로 인해 실수할 여지가 많다. 또한 추가적인 요구사항이 들어올 경우 수정해야 될 곳이 명확하게 보이지 않는다. 같은 결과를 출력하는 Stream Api를 적용한 아래 코드를 살펴보자.

```java
final List<Integer> NUMBERS = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
System.out.println("Functional Result: " +
                   NUMBERS.stream()
                   .filter(number -> number > 3)
                   .filter(number -> number < 9)
                   .map(number -> number * 2)
                   .filter(number -> number > 10)
                   .findFirst()
                  );

```

필요한 조건이나 연산들만 파라미터로 넣어줌으로서 코드가 훨씬 더 간결해짐을 확인 할 수 있다. 또한 layz 방식으로 연산이 수행되기에 기존에 명령형 코드와 효율성 차이에서 거의 차이가 없다.  Stream Api의 위력을 확인했으니 이제 보다 자세하게 알아보자



### Stream API 구조

Strema API는 **Stream 생성, 중개연산, 최종연산** 세 단계로 구분된다.  

- Java의 모든 Collection은 내부에 **stream()**이라는 메소드가 존재, 이 함수로 stream을 생성 할 수 있다. 또는 **Stream.of()** 메소드를 통해서도 생성 할 수 있다.

- 중개연산은 map, filter와 같이 stream을 return 하는 함수들을 말한다. stream을 리턴함으로, **중개연산은 메소드 체이닝**이 가능하다.  
- 최종연산은 중개연산과는 달리 특정 자료형을 리턴하기에 stream api 연산을 이어서 할 수 없다.



### Collect Method

중개연산등에 의해 **가공이 끝난 stream을 collection으로** 바꾸어주는 최종연산 method이다. 파라미터로 넘겨주는 값에 따라서 어떤 형태의 collection을 return 할지 정 할 수 있다. 

`joining(구분자, prefix, postfix)  `  : stream의 원소들을 String으로 concat 한다.

```java
Stream.of(1, 3, 3, 5, 5)
      .filter(i -> i > 2)
      .map(i -> i * 2)
      .map(i -> "#" + i)
      .collect(joining(", ", "[", "]")) // // [#6, #6, #10, #10]
  
```

`toList()`  : stream의 원소들을 List에 담아서 return 한다.

`toSet()` : stream의 원소들을 set에 담아서 return 한다. (**distinct()** 라는 중개연산자를 사용하면, toList를 통해서 순서가 보장된 set을 구현 할 수 도 있다.)



### Reduce Method

 stream의 원소들을 하나씩 줄여나간다고 해서 reduce라는 이름이 붙었다.  최종연산 메소드로, **redeuce(초기값, BiFuction)** 형태로 사용된며, 원소들을 하나의 결과로 합친다. **BiFuction은 같은 타입의 인자를 두개를 가지는 Fuction 인터페이스다**.  그중 첫번째 파라미터는 이전 원소값이다. 맨 첫번째 원소는 이전 값이 없는데, 이 때문에 첫번째 인자인 초기값이 필요하다. 이 초기값은 항등값으로 설정 해주는 것이 좋다.

```java
final List<Product> products =
        Arrays.asList(
            new Product(1L, "A", new BigDecimal("100.50")),
            new Product(2L, "B", new BigDecimal("23.00")),
            new Product(3L, "C", new BigDecimal("31.45")),
            new Product(4L, "D", new BigDecimal("80.20")),
            new Product(5L, "E", new BigDecimal("7.50"))
        );

System.out.println("Total Price: " +
                   products.stream()
                   .map(product -> product.getPrice())
                   .reduce(BigDecimal.ZERO, (price1, price2) -> price1.add(price2))
                  );
```



### Parallel Programming (추후 다룰 예정)

- ORM에서 사용할 때는 객체 내부의 객체는 반드시 Eger 패치를 사용!



이외에 Stream API에서 제공하는 메소드들은 [공식문서](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)를 확인하자

