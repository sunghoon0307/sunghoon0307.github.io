---
title: "[JAVA8_함수형 프로그래밍] String, Comparator 그리고 filter"
author: sunghoon
date: 2022-06-23 17:00:00 +0900
categories: [Java8]
tags: [Java8, 함수형 프로그래밍, 람다, stream]
pin: false
comments: true
--- 

## ▶︎ 스트링 이터레이션

- chars() 메서드는 forEach() 내부 이터레이터를 사용하여 이터레이션하는 스트림을 리턴한다.

```java
final String str = "w00t";

str.chars()
    .forEach(ch -> System.out.println(ch));

// 메서드 레퍼런스로 변환
str.chars()
    .forEach(System.out::println);
```

> 119  
> 48  
> 48  
> 116  

→ 문자가 출력되는 대신 숫자가 출력됐다. 그 이유는 chars() 메서드가 Characters의 스트림 대신 문자를 표현하는 intStream을 리턴했기 때문

```java
// chars() 메서드내부
public default IntStream chars() {
  class CharIterator implements PrimitiveIterator.OfInt {
      int cur = 0;

      public boolean hasNext() {
          return cur < length();
      }

      public int nextInt() {
			...
			...
```

- 컨비니언스 메소드를 사용한 int를 문자로 출력

```java
str.chars()
    .forEach(IterateString::printChar);

private static void printChar(int aChar) {
    System.out.println((char)(aChar));
}
```

> w  
0  
0  
t  
> 

- 처음부터 int가 아닌 문자로 처리하고 싶다면 chars()를 호출하고 나서 바로 문자로 변환하면 된다.

```java
str.chars()
    .mapToObj(ch -> Character.valueOf((char) ch))
    .forEach(System.out::println);
```

- 한번 스트림을 얻으면 스트림에서 제공하는 메서드를 이용하여 스트링에 있는 문자들을 처리할 수 있다. (map(), filter(), reduce() 등)

```java
// 숫자 필터링하기
str.chars()
  .filter(ch -> Character.isDigit(ch))
  .forEach(ch -> printChar(ch));
```

> 0  
0  
> 

```java
// 숫자 필터링하기의 메서드 레퍼런스 형식
str.chars()
    .filter(Character::isDigit)
    .forEach(IterateString::printChar);
```

→ 메서드 레퍼런스는 일반적인 파라미터 라우팅을 제거하도록 해준다.

## ▶︎ Comparator 인터페이스 구현

- Comparator을 사용한 정렬
    
    → name, age 혹은 여러 필드의 조합으로 엘리먼트를 비교하고 싶을때 Comparator 인터페이스로 가능 
    

```java
// person.java
public class Person {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public int ageDifference(final Person other) {
        return age - other.age;
    }

    public String toString() {
        return String.format("%s - %d", name, age);
    }
}

// 작업할 사람들의 이름과 나이를 생성
final List<Person> people = Arrays.asList(
                new Person("John", 20),
                new Person("Sara", 21),
                new Person("Jane", 21),
                new Person("Greg", 35));
```

- age 오름차순 정렬

```java
List<Person> ascendingAge =
        people.stream()
                .sorted((person1, person2) -> person1.ageDifference(person2))
                .collect(Collectors.toList());

printPeople("Sorted in ascending order by age : ", ascendingAge);
```

> Sorted in ascending order by age :   
John - 20  
Sara - 21  
Jane - 21  
Greg - 35  
> 

→ 리스트에 대한 sort()메서드는 void메서드이기 때문에 리스트 자체가 변경될수 있음.
    스트림에서 sorted()메서드는 이미 갖고 있는 컬렉션을 변경하는 대신 정렬된 컬렉션을 리턴함

```java
// 메서드 레퍼런스로 변환
List<Person> ascendingAge2 =
      people.stream()
              .sorted(Person::ageDifference)
              .collect(Collectors.toList());
```

*`Stream*<T> sorted(*Comparator*<? *super* T> comparator);`

→ sorted 메서드는 파라미터로 Comparator를 갖는다. Comparator가 함수형 인터페이스이기 때문에 람다 표현식을 인수로 쉽게 넘길 수 있다.

`collect()`

→ 이터레이션의 타깃 멤버를 원하는 타입의 포맷으로 변환하는 리듀서(reducer)

- age 내림차순 정렬

```java
// age내림차순
// 파라미터의 순서만 바꾸면되지만 중복코드가 된다
List<Person> descendingAge =
        people.stream()
                .sorted((person1, person2) -> person2.ageDifference(person1))
                .collect(Collectors.toList());

printPeople("Sorted in descending order by age : ", descendingAge);

System.out.println("=========================");

// reversed()메소드로 중복제거
Comparator<Person> compareAscending = (person1, person2) -> person1.ageDifference(person2);
Comparator<Person> compareDescending = compareAscending.reversed();

printPeople("Sorted in ascending order by age : " ,
        people.stream()
            .sorted(compareAscending)
            .collect(Collectors.toList()));

printPeople("Sorted in descending order by age : " ,
        people.stream()
                .sorted(compareDescending)
                .collect(Collectors.toList()));
```

→ 위 소스는 람다 표현식이 아닌 메서드 레퍼런스로는 쉽게 변경할 수 없다. 왜냐하면 파라미터 순서가 메서드 레퍼런스를 사용하기 위한 파라미터 라우팅 규칙을 따르지 않기 때문!

> Sorted in descending order by age :  
Greg - 35  
Sara - 21  
Jane - 21  
John - 20  
Sorted in ascending order by age :  
John - 20  
Sara - 21  
Jane - 21  
Greg - 35  
> 

- name 알파벳의 오름차순 정렬

```java
printPeople("Sorted in ascending order by name : " ,
    people.stream()
            .sorted((person1, person2) -> person1.getName().compareTo(person2.getName()))
            .collect(Collectors.toList()));
```

> Sorted in ascending order by name :  
Greg - 35  
Jane - 21  
John - 20  
Sara - 21  
> 

- 가장 젊은 사람, 가장 늙은 사람 출력

```java
// 가장 젊은 사람
people.stream()
        .min(Person::ageDifference)
        .ifPresent(youngest -> System.out.println("Youngest : " + youngest));

// 가장 나이가 많은 사람
people.stream()
        .max(Person::ageDifference)
        .ifPresent(eldest -> System.out.println("eldest : " + eldest));
```

→ min, max 메서드는 Optional을 리턴한다. 왜냐하면 리스트가 비어 있을 수 있고 따라서 가장 어린사람, 나이많은 사람이 없을 수 있기 때문에

## ▶︎ 여러가지 비교연산

- comparing() 메서드

```java
// 이름으로 사람 정렬하기 Function 이용
final Function<Person, String> byName = person -> person.getName();
printPeople("이름순 정렬하기 : ",
people.stream()
        .sorted(Comparator.comparing(byName))
        .collect(Collectors.toList()));
```

> 이름순 정렬하기 :   
Greg - 35  
Jane - 21  
John - 20  
Sara - 21  
> 

- 복합정렬

```java
// 나이, 이름으로 복합 정렬 (나이 -> 이름)
final Function<Person, Integer> byAge = person -> person.getAge();
final Function<Person, String> byTheirName = person -> person.getName();
printPeople("복합 정렬하기 : ",
        people.stream()
                .sorted(Comparator.comparing(byAge).thenComparing(byTheirName))
                .collect(Collectors.toList()));
```

> 복합 정렬하기 :  
John - 20  
Jane - 21  
Sara - 21  
Greg - 35  
> 

## ▶︎ collect 메서드와 Collectors 클래스 사용하기

- 원본 리스트에서 20살 이상의 사람들만 출력하기

1. 가변성과 forEach()를 사용한 버전

```java
List<Person> olderThan20 = new ArrayList<>();

people.stream()
        .filter(person -> person.getAge() > 20)
        .forEach(person -> olderThan20.add(person));
System.out.println("olderThan20 = " + olderThan20);
```

→ 타깃 컬렉션 (olderThan20)에 엘리먼트를 추가하는 오퍼레이션이 너무 로우레벨이다. 이터레이션을 동시에 실행하려면, 즉시 스레드 세이프티 문제에 대해 고려해야함.

2. collect() 메서드 사용한 버전

```java
List<Person> olderThan20 =
    people.stream()
        .filter(person -> person.getAge() > 20)
        .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
```

-첫 번째 파라미터 : 결과 컨테이너 만들기 (팩토리(factory)나 서플라이어(supplier))

-두 번째 파라미터 : 하나의 엘리먼트를 결과 컨테이너에 추가하는 방법

-세 번째 파라미터 : 하나의 결과 컨테이너를 다른 것과 합치는 방법

- collect() 메서드를 사용함으로써 코드에서 명시적 변경이 발생하지 않기 때문에 이터레이션의 실행을 병렬화하기가 쉽다. 변경에 대한 부분은 라이브러리에서 제어하기 때문에 라이브러리를 사용한 조율이 간단하고 스레드 세이프티를 보장해준다.
- collect() 메서드는 다른 서브 리스트 간의 병렬 덧셈을 수행하여 그 병렬 덧셈의 결과를 스레드 세이프티하게 좀 더 큰 규모의 리스트로 합칠 수 있다.

3. Collectors 유틸리티 클래스

```java
List<Person> orderThan20_3 =
    people.stream()
            .filter(person -> person.getAge() > 20)
            .collect(toList());
```

→ 추가적으로 세트를 모으는 toSet(), key-value 컬렉션을 모으는 toMap()도 있음. mapping(), colectingAndThen(), minBy(), maxBy(), groupingBy()과 같은 메서드를 사용하여 여러 오퍼레이션을 합쳐서 사용할 수도 있다.

- 나이로 groupingBy()

```java
//Collectors 유틸리티 클래스 응용 - groupingBy()
Map<Integer, List<Person>> peopleByAge =
        people.stream()
                .collect(groupingBy(Person::getAge));
System.out.println("Grouping by age : " + peopleByAge);
```

> Grouping by age : {35=[Greg - 35], 20=[John - 20], 21=[Sara - 21, Jane - 21]}
> 

- 이름과 나이 순서대로 정렬

```java
Map<Integer, List<String>> nameOfPeopleByAge =
        people.stream()
                .collect(groupingBy(Person::getAge, mapping(Person::getName, toList())));
System.out.println("People grouped by age : " + nameOfPeopleByAge);
```

> People grouped by age : {35=[Greg], 20=[John], 21=[Sara, Jane]}  
>   

```java
// groupingBy 메서드 내부
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                      Collector<? super T, A, D> downstream) {
    return groupingBy(classifier, HashMap::new, downstream);
}
```

→ 첫 번째 인자는 그룹을 만드는 기준(나이)이고, 두 번째는 mapping() 함수 호출의 결과인 Collector

- 첫 번째 문자로 이름을 그룹핑하고 각 그룹의 나이가 많은 사람을 찾기

```java
Comparator<Person> byAge = Comparator.comparing(Person::getAge);
Map<Character, Optional<Person>> oldestPersonOfEachLetter =
        people.stream()
                .collect(groupingBy(person -> person.getName().charAt(0), reducing(BinaryOperator.maxBy(byAge))));

System.out.println("Oldest person of each letter : ");
System.out.println(oldestPersonOfEachLetter);
```

> Oldest person of each letter :  
{S=Optional[Sara - 21], G=Optional[Greg - 35], J=Optional[Jane - 21]}  
>

  


[출처]  
![책](/assets/img/java8/java8.jpeg)  

[자바 8 람다의 힘(벤컷 수브라마니암 지음, 루비페이퍼, 2014)] 



