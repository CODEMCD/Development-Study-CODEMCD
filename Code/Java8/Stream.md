# Java8 in Action
## flatMap
### 문자열 리스트에서 고유 문자 로 이루어진 리스트 반환한다.

```java
List<String> words = Arrays.asList("Hello", "World");
List<String> uniqueCharacters = words.stream()
        .map( word -> word.split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(toList());
```

### 숫자 리스트가 주어졌을 때 각 숫자의 제곱근으로 이루어진 리스트를 반환하시오. 예를 들어 [1, 2, 3, 4, 5]가 주어지면 [1, 4, 9, 16, 25]를 반환한다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> squares = numbers.stream()
        .map(num -> num * num)
        .collect(toList());
```

### 두 개의 숫자 리스트가 있을 때 모든 숫자 쌍의 리스트를 반환하시오. 예를 들어 두 개의 리스트[1, 2, 3]과 [3, 4]가 주어지면 [(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]를 반환해야 한다.

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> pairs = numbers1.stream()
        .flatMap(i -> numbers2.stream()
                    .map(j -> new int[]{i, j})
        )
        .collect(toList());
```

### 위 예제에서 합이 3으로 나누어 떨어지는 쌍만 반환하려면 어떻게 해야 할까? 예를 들어 (2, 4), (3, 3)을 반환해야 한다.

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> pairs = numbers1.stream()
        .flatMap(i -> numbers2.stream()
                .filter(j -> (i + j) % 3 == 0)
                .map(j -> new int[]{i, j})
        )
        .collect(toList());
```

## 실전 연습

### ```Trader``` 클래스

```java
public  class Trader{

    private String name;
    private String city;

    public Trader(String n, String c){
        this.name = n;
        this.city = c;
    }

    public String getName(){
        return this.name;
    }

    public String getCity(){
        return this.city;
    }

    public void setCity(String newCity){
        this.city = newCity;
    }

    public String toString(){
        return "Trader:"+this.name + " in " + this.city;
    }
}
```

### ```Transaction``` 클래스

```java
public class Transaction{

    private Trader trader;
    private int year;
    private int value;

    public Transaction(Trader trader, int year, int value)
    {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Trader getTrader(){
        return this.trader;
    }

    public int getYear(){
        return this.year;
    }

    public int getValue(){
        return this.value;
    }

    public String toString(){
        return "{" + this.trader + ", " +
                "year: "+this.year+", " +
                "value:" + this.value +"}";
    }
}
```

### 데이터

```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario","Milan");
Trader alan = new Trader("Alan","Cambridge");
Trader brian = new Trader("Brian","Cambridge");

List<Transaction> transactions = Arrays.asList(
       new Transaction(brian, 2011, 300),
       new Transaction(raoul, 2012, 1000),
       new Transaction(raoul, 2011, 400),
       new Transaction(mario, 2012, 710),
       new Transaction(mario, 2012, 700),
       new Transaction(alan, 2012, 950)
);
```

### 풀이

1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.

```java
List<Transaction> q1 = transactions.stream()
                .filter(t -> t.getYear() == 2011)
                .sorted((Transaction t1, Transaction t2) -> {
                    if (t1.getValue() > t2.getValue()) return 1;
                    else return -1;
                })
                .collect(toList())
                ;
```

```java
List<Transaction> q1 = transactions.stream()
             .filter(transaction -> transaction.getYear() == 2011)
             .sorted(comparing(Transaction::getValue))
             .collect(toList())
             ;
```

2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.

```java
List<String> q2 = transactions.stream()
                .map(transaction -> transaction.getTrader().getCity())
                .distinct()
                .collect(toList())
                ;
```

3. 케임브리지(Cambridge)에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.

```java
List<String> q3 = transactions.stream()
                .filter(transaction -> "Cambridge".equals(transaction.getTrader().getCity()))
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted(String::compareTo)
                .collect(toList())
                ;
```

```java
List<Trader> q3 = transactions.stream()
                .map(Transaction::getTrader)
                .filter(trader -> "Cambridge".equals(trader.getCity()))
                .distinct()
                .sorted(comparing(Trader::getName))
                .collect(toList())
                ;
```

4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.

```java
List<String> q4 = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted(String::compareTo)
                .collect(toList())
                ;
```

```java
String q4 = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted()
                .reduce("", (n1, n2) -> n1 + n2)
                ;
```

```java
String q4 = transactions.stream()
                .map(transaction -> transaction.getTrader().getName())
                .distinct()
                .sorted()
                .collect(joining())
                ;
```

5. 밀라노(Milan)에 거래자가 있는가?

```java
Boolean q5 = transactions.stream()
               .anyMatch(transaction -> "Milan".equals(transaction.getTrader().getCity()))
               ;
```

6. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값을 출려하시오.

```java
List<Integer> q6 = transactions.stream()
                .filter(transaction -> "Cambridge".equals(transaction.getTrader().getCity()))
                .map(Transaction::getValue)
                .collect(toList())
                ;
```

```java
transactions.stream()
                .filter(transaction -> "Cambridge".equals(transaction.getTrader().getCity()))
                .map(Transaction::getValue)
                .forEach(System.out::println)
                ;
```

7. 전체 트랜잭션 중 최댓값은 얼마인가?

```java
int q7 = transactions.stream()
                .map(Transaction::getValue)
                .reduce(0, (n1, n2) -> n1 > n2 ? n1 : n2);
```

```java
Optional<Integer> q7 = transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::max)
                ;
```

8. 전체 트랜잭션 중 최솟값은 얼마인가?

```java
Optional<Integer> q8 = transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::min)
                ;
```

```java
Optional<Integer> q8 = transactions.stream()
                .min(comparing(Transaction::getValue))
                ;
```
