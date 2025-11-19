# Java Stream API Coding Questions with Solutions

## BASIC LEVEL (1-15)

### 1. Find unique/distinct numbers from an array
```java
List<Integer> numbers = List.of(1,2,3,2,4,5,4);
// Output: [1, 2, 3, 4, 5]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
System.out.println(unique); // [1, 2, 3, 4, 5]
```
</details>

---

### 2. Filter even numbers from a list
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
// Output: [2, 4, 6, 8, 10]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(evenNumbers); // [2, 4, 6, 8, 10]
```
</details>

---

### 3. Calculate sum of all numbers
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
// Output: 15
```

<details>
<summary>Show Answer</summary>

```java
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
System.out.println(sum); // 15

// Alternative
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);
```
</details>

---

### 4. Convert list of strings to uppercase
```java
List<String> words = List.of("java", "spring", "boot");
// Output: ["JAVA", "SPRING", "BOOT"]
```

<details>
<summary>Show Answer</summary>

```java
List<String> upperCase = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
System.out.println(upperCase); // [JAVA, SPRING, BOOT]
```
</details>

---

### 5. Find maximum number from a list
```java
List<Integer> numbers = List.of(10, 5, 20, 15, 30);
// Output: 30
```

<details>
<summary>Show Answer</summary>

```java
int max = numbers.stream()
    .mapToInt(Integer::intValue)
    .max()
    .orElse(0);
System.out.println(max); // 30

// Alternative using Optional
Optional<Integer> maxOpt = numbers.stream()
    .max(Integer::compareTo);
```
</details>

---

### 6. Find minimum number from a list
```java
List<Integer> numbers = List.of(10, 5, 20, 15, 30);
// Output: 5
```

<details>
<summary>Show Answer</summary>

```java
int min = numbers.stream()
    .mapToInt(Integer::intValue)
    .min()
    .orElse(0);
System.out.println(min); // 5

// Alternative
Optional<Integer> minOpt = numbers.stream()
    .min(Integer::compareTo);
```
</details>

---

### 7. Calculate average of numbers
```java
List<Integer> numbers = List.of(10, 20, 30, 40, 50);
// Output: 30.0
```

<details>
<summary>Show Answer</summary>

```java
double average = numbers.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0.0);
System.out.println(average); // 30.0
```
</details>

---

### 8. Count total number of elements
```java
List<String> words = List.of("apple", "banana", "orange");
// Output: 3
```

<details>
<summary>Show Answer</summary>

```java
long count = words.stream()
    .count();
System.out.println(count); // 3
```
</details>

---

### 9. Join strings with delimiter
```java
List<String> words = List.of("Java", "Spring", "Boot");
// Output: "Java, Spring, Boot"
```

<details>
<summary>Show Answer</summary>

```java
String joined = words.stream()
    .collect(Collectors.joining(", "));
System.out.println(joined); // "Java, Spring, Boot"

// With prefix and suffix
String withBrackets = words.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Output: "[Java, Spring, Boot]"
```
</details>

---

### 10. Square all numbers
```java
List<Integer> numbers = List.of(2, 3, 4, 5);
// Output: [4, 9, 16, 25]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> squared = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
System.out.println(squared); // [4, 9, 16, 25]
```
</details>

---

### 11. Filter strings starting with specific letter
```java
List<String> words = List.of("apple", "banana", "avocado", "orange");
// Filter words starting with 'a'
// Output: ["apple", "avocado"]
```

<details>
<summary>Show Answer</summary>

```java
List<String> filtered = words.stream()
    .filter(word -> word.startsWith("a"))
    .collect(Collectors.toList());
System.out.println(filtered); // [apple, avocado]

// Case insensitive
List<String> filteredIgnoreCase = words.stream()
    .filter(word -> word.toLowerCase().startsWith("a"))
    .collect(Collectors.toList());
```
</details>

---

### 12. Remove null values from list
```java
List<String> words = Arrays.asList("Java", null, "Spring", null, "Boot");
// Output: ["Java", "Spring", "Boot"]
```

<details>
<summary>Show Answer</summary>

```java
List<String> nonNull = words.stream()
    .filter(Objects::nonNull)
    .collect(Collectors.toList());
System.out.println(nonNull); // [Java, Spring, Boot]
```
</details>

---

### 13. Convert string array to integer array
```java
String[] strArray = {"1", "2", "3", "4", "5"};
// Output: [1, 2, 3, 4, 5]
```

<details>
<summary>Show Answer</summary>

```java
int[] intArray = Arrays.stream(strArray)
    .mapToInt(Integer::parseInt)
    .toArray();
System.out.println(Arrays.toString(intArray)); // [1, 2, 3, 4, 5]

// To List<Integer>
List<Integer> intList = Arrays.stream(strArray)
    .map(Integer::parseInt)
    .collect(Collectors.toList());
```
</details>

---

### 14. Sort list in ascending order
```java
List<Integer> numbers = List.of(5, 2, 8, 1, 9);
// Output: [1, 2, 5, 8, 9]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sorted); // [1, 2, 5, 8, 9]

// Using Comparator
List<Integer> sortedAsc = numbers.stream()
    .sorted(Comparator.naturalOrder())
    .collect(Collectors.toList());
```
</details>

---

### 15. Sort list in descending order
```java
List<Integer> numbers = List.of(5, 2, 8, 1, 9);
// Output: [9, 8, 5, 2, 1]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> sortedDesc = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
System.out.println(sortedDesc); // [9, 8, 5, 2, 1]

// Alternative
List<Integer> sorted = numbers.stream()
    .sorted((a, b) -> b - a)
    .collect(Collectors.toList());
```
</details>

---

## INTERMEDIATE LEVEL (16-35)

### 16. Find duplicate elements from list
```java
List<Integer> numbers = List.of(1,2,3,2,4,6,7,4);
// Output: [2, 4]
```

<details>
<summary>Show Answer</summary>

```java
Set<Integer> seen = new HashSet<>();
List<Integer> duplicates = numbers.stream()
    .filter(n -> !seen.add(n))
    .distinct()
    .collect(Collectors.toList());
System.out.println(duplicates); // [2, 4]

// Alternative using frequency
List<Integer> duplicates2 = numbers.stream()
    .filter(n -> Collections.frequency(numbers, n) > 1)
    .distinct()
    .collect(Collectors.toList());
```
</details>

---

### 17. Find first repeated character in string
```java
String input = "hello world";
// Output: 'l'
```

<details>
<summary>Show Answer</summary>

```java
Set<Character> seen = new HashSet<>();
Character result = input.chars()
    .mapToObj(c -> (char) c)
    .filter(c -> !seen.add(c))
    .findFirst()
    .orElse(null);
System.out.println(result); // l
```
</details>

---

### 18. Find first non-repeated character in string
```java
String input = "hello";
// Output: 'h'
```

<details>
<summary>Show Answer</summary>

```java
Character result = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), 
             LinkedHashMap::new, Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() == 1)
    .map(Map.Entry::getKey)
    .findFirst()
    .orElse(null);
System.out.println(result); // h
```
</details>

---

### 19. Count character frequency in string
```java
String input = "bcbca";
// Output: "b2c2a1"
```

<details>
<summary>Show Answer</summary>

```java
String result = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), 
             LinkedHashMap::new, Collectors.counting()))
    .entrySet().stream()
    .map(e -> e.getKey() + "" + e.getValue())
    .collect(Collectors.joining());
System.out.println(result); // b2c2a1
```
</details>

---

### 20. Find words starting with 'b'
```java
String input = "baby bird asked about blue ants";
// Output: "baby bird blue"
```

<details>
<summary>Show Answer</summary>

```java
String result = Arrays.stream(input.split(" "))
    .filter(word -> word.startsWith("b"))
    .collect(Collectors.joining(" "));
System.out.println(result); // baby bird blue
```
</details>

---

### 21. Move all zeros to right side
```java
List<Integer> numbers = List.of(1, 0, -3, 0, 5, -2, 0, 8, 0, -4);
// Output: [1, -3, 5, -2, 8, -4, 0, 0, 0, 0]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> result = Stream.concat(
    numbers.stream().filter(n -> n != 0),
    numbers.stream().filter(n -> n == 0)
).collect(Collectors.toList());
System.out.println(result); // [1, -3, 5, -2, 8, -4, 0, 0, 0, 0]
```
</details>

---

### 22. Merge two arrays
```java
int[] arr1 = {1, 2, 3};
int[] arr2 = {4, 5, 6};
// Output: [1, 2, 3, 4, 5, 6]
```

<details>
<summary>Show Answer</summary>

```java
int[] merged = IntStream.concat(
    Arrays.stream(arr1), 
    Arrays.stream(arr2)
).toArray();
System.out.println(Arrays.toString(merged)); // [1, 2, 3, 4, 5, 6]

// For List
List<Integer> mergedList = Stream.concat(
    Arrays.stream(arr1).boxed(),
    Arrays.stream(arr2).boxed()
).collect(Collectors.toList());
```
</details>

---

### 23. Find second highest number
```java
List<Integer> numbers = List.of(10, 20, 30, 40, 50);
// Output: 40
```

<details>
<summary>Show Answer</summary>

```java
Integer secondHighest = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElse(null);
System.out.println(secondHighest); // 40

// Alternative - distinct to handle duplicates
Integer secondHighest2 = numbers.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElse(null);
```
</details>

---

### 24. Find second lowest number
```java
List<Integer> numbers = List.of(10, 20, 30, 40, 50);
// Output: 20
```

<details>
<summary>Show Answer</summary>

```java
Integer secondLowest = numbers.stream()
    .sorted()
    .skip(1)
    .findFirst()
    .orElse(null);
System.out.println(secondLowest); // 20

// With distinct
Integer secondLowest2 = numbers.stream()
    .distinct()
    .sorted()
    .skip(1)
    .findFirst()
    .orElse(null);
```
</details>

---

### 25. Sum of all even numbers
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
// Output: 30
```

<details>
<summary>Show Answer</summary>

```java
int sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
System.out.println(sum); // 30

// Alternative using reduce
int sum2 = numbers.stream()
    .filter(n -> n % 2 == 0)
    .reduce(0, Integer::sum);
```
</details>

---

### 26. Sum of all odd numbers
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
// Output: 25
```

<details>
<summary>Show Answer</summary>

```java
int sum = numbers.stream()
    .filter(n -> n % 2 != 0)
    .mapToInt(Integer::intValue)
    .sum();
System.out.println(sum); // 25
```
</details>

---

### 27. Partition list into even and odd
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8);
// Output: {false=[1,3,5,7], true=[2,4,6,8]}
```

<details>
<summary>Show Answer</summary>

```java
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
System.out.println(partitioned); 
// {false=[1, 3, 5, 7], true=[2, 4, 6, 8]}

// Access even and odd separately
List<Integer> evenNumbers = partitioned.get(true);
List<Integer> oddNumbers = partitioned.get(false);
```
</details>

---

### 28. Count frequency of each element
```java
List<String> words = List.of("apple", "banana", "apple", "orange", "banana");
// Output: {apple=2, banana=2, orange=1}
```

<details>
<summary>Show Answer</summary>

```java
Map<String, Long> frequency = words.stream()
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));
System.out.println(frequency); 
// {orange=1, banana=2, apple=2}

// For ordered output
Map<String, Long> orderedFrequency = words.stream()
    .collect(Collectors.groupingBy(
        Function.identity(),
        LinkedHashMap::new,
        Collectors.counting()
    ));
```
</details>

---

### 29. Find numbers starting with specific digit
```java
List<Integer> numbers = List.of(12, 23, 34, 21, 45, 56);
// Find numbers starting with 2
// Output: [23, 21]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> result = numbers.stream()
    .filter(n -> String.valueOf(n).startsWith("2"))
    .collect(Collectors.toList());
System.out.println(result); // [23, 21]

// For any digit
int digit = 2;
List<Integer> result2 = numbers.stream()
    .filter(n -> String.valueOf(n).startsWith(String.valueOf(digit)))
    .collect(Collectors.toList());
```
</details>

---

### 30. Reverse each word in string
```java
String input = "Hello World Java";
// Output: "olleH dlroW avaJ"
```

<details>
<summary>Show Answer</summary>

```java
String result = Arrays.stream(input.split(" "))
    .map(word -> new StringBuilder(word).reverse().toString())
    .collect(Collectors.joining(" "));
System.out.println(result); // olleH dlroW avaJ
```
</details>

---

### 31. Find common elements between two lists
```java
List<Integer> list1 = List.of(1, 2, 3, 4, 5);
List<Integer> list2 = List.of(4, 5, 6, 7, 8);
// Output: [4, 5]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> common = list1.stream()
    .filter(list2::contains)
    .collect(Collectors.toList());
System.out.println(common); // [4, 5]

// More efficient for large lists
Set<Integer> set2 = new HashSet<>(list2);
List<Integer> common2 = list1.stream()
    .filter(set2::contains)
    .collect(Collectors.toList());
```
</details>

---

### 32. Square numbers and filter greater than 50
```java
List<Integer> numbers = List.of(2, 5, 7, 8, 10);
// Output: [64, 100]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> result = numbers.stream()
    .map(n -> n * n)
    .filter(n -> n > 50)
    .collect(Collectors.toList());
System.out.println(result); // [64, 100]
```
</details>

---

### 33. Skip first N elements and get next M elements (Pagination)
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
// Skip 3, take 3
// Output: [4, 5, 6]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> page = numbers.stream()
    .skip(3)
    .limit(3)
    .collect(Collectors.toList());
System.out.println(page); // [4, 5, 6]

// Generic pagination method
public static <T> List<T> getPage(List<T> list, int pageNumber, int pageSize) {
    return list.stream()
        .skip((long) (pageNumber - 1) * pageSize)
        .limit(pageSize)
        .collect(Collectors.toList());
}
```
</details>

---

### 34. Check if all elements are even (allMatch)
```java
List<Integer> numbers = List.of(2, 4, 6, 8, 10);
// Output: true
```

<details>
<summary>Show Answer</summary>

```java
boolean allEven = numbers.stream()
    .allMatch(n -> n % 2 == 0);
System.out.println(allEven); // true
```
</details>

---

### 35. Check if any element is even (anyMatch)
```java
List<Integer> numbers = List.of(1, 3, 5, 6, 7);
// Output: true
```

<details>
<summary>Show Answer</summary>

```java
boolean anyEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
System.out.println(anyEven); // true

// noneMatch - opposite of anyMatch
boolean noneEven = numbers.stream()
    .noneMatch(n -> n % 2 == 0);
```
</details>

---

## ADVANCED LEVEL (36-60)

### 36. Flatten list of lists (FlatMap)
```java
List<List<Integer>> listOfLists = List.of(
    List.of(1, 2, 3),
    List.of(4, 5),
    List.of(6, 7, 8, 9)
);
// Output: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> flattened = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
System.out.println(flattened); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// For array of arrays
Integer[][] arrayOfArrays = {{1,2,3}, {4,5}, {6,7,8,9}};
List<Integer> flattened2 = Arrays.stream(arrayOfArrays)
    .flatMap(Arrays::stream)
    .collect(Collectors.toList());
```
</details>

---

### 37. Group employees by department
```java
class Employee {
    private String name;
    private String department;
    private double salary;
    // constructors, getters, setters
}
List<Employee> employees = /* list */;
// Output: Map<String, List<Employee>>
```

<details>
<summary>Show Answer</summary>

```java
Map<String, List<Employee>> grouped = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Get count per department
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// Get average salary per department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```
</details>

---

### 38. Convert List to Map (id -> name)
```java
class Employee {
    private Integer id;
    private String name;
    // constructors, getters
}
List<Employee> employees = /* list with id and name */;
// Output: Map<Integer, String>
```

<details>
<summary>Show Answer</summary>

```java
Map<Integer, String> map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Employee::getName
    ));

// Handle duplicate keys
Map<Integer, String> map2 = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Employee::getName,
        (existing, replacement) -> existing
    ));

// Map to entire object
Map<Integer, Employee> empMap = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Function.identity()
    ));
```
</details>

---

### 39. Sort Map by value
```java
Map<String, Integer> map = Map.of("A", 5, "B", 2, "C", 8);
// Output: {B=2, A=5, C=8}
```

<details>
<summary>Show Answer</summary>

```java
Map<String, Integer> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));
System.out.println(sorted); // {B=2, A=5, C=8}

// Descending order
Map<String, Integer> sortedDesc = map.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));
```
</details>

---

### 40. Sort Map by key
```java
Map<String, Integer> map = Map.of("C", 5, "A", 2, "B", 8);
// Output: {A=2, B=8, C=5}
```

<details>
<summary>Show Answer</summary>

```java
Map<String, Integer> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByKey())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));
System.out.println(sorted); // {A=2, B=8, C=5}
```
</details>

---

### 41. Find top N salaries
```java
class Employee {
    private String name;
    private double salary;
    // constructors, getters
}
List<Employee> employees = /* list */;
// Find top 3 salaries
```

<details>
<summary>Show Answer</summary>

```java
List<Double> topSalaries = employees.stream()
    .map(Employee::getSalary)
    .sorted(Comparator.reverseOrder())
    .limit(3)
    .collect(Collectors.toList());

// Top 3 employees by salary
List<Employee> topEmployees = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .limit(3)
    .collect(Collectors.toList());

// Distinct top 3 salaries
List<Double> topDistinctSalaries = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .limit(3)
    .collect(Collectors.toList());
```
</details>

---

### 42. Find Nth highest salary
```java
List<Employee> employees = /* list */;
// Find 3rd highest salary
```

<details>
<summary>Show Answer</summary>

```java
int n = 3;
Double nthSalary = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(n - 1)
    .findFirst()
    .orElse(null);

// Get employee with nth highest salary
Employee nthEmployee = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .skip(n - 1)
    .findFirst()
    .orElse(null);
```
</details>

---

### 43. Find employees with salary greater than average
```java
List<Employee> employees = /* list */;
```

<details>
<summary>Show Answer</summary>

```java
double avgSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

List<Employee> aboveAverage = employees.stream()
    .filter(e -> e.getSalary() > avgSalary)
    .collect(Collectors.toList());
```
</details>

---

### 44. Group strings by length
```java
List<String> words = List.of("a", "bb", "ccc", "dd", "eee", "f");
// Output: {1=[a, f], 2=[bb, dd], 3=[ccc, eee]}
```

<details>
<summary>Show Answer</summary>

```java
Map<Integer, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length));
System.out.println(grouped); 
// {1=[a, f], 2=[bb, dd], 3=[ccc, eee]}

// Get count per length
Map<Integer, Long> countByLength = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.counting()
    ));
```
</details>

---

### 45. Find longest string in list
```java
List<String> words = List.of("Java", "Spring", "Microservices", "Boot");
// Output: "Microservices"
```

<details>
<summary>Show Answer</summary>

```java
String longest = words.stream()
    .max(Comparator.comparing(String::length))
    .orElse(null);
System.out.println(longest); // Microservices

// Alternative using reduce
String longest2 = words.stream()
    .reduce((w1, w2) -> w1.length() > w2.length() ? w1 : w2)
    .orElse(null);
```
</details>

---

### 46. Separate uppercase and lowercase characters
```java
String input = "HelloWorld";
// Output: Uppercase: "HW", Lowercase: "elloorld"
```

<details>
<summary>Show Answer</summary>

```java
Map<Boolean, String> partitioned = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.partitioningBy(
        Character::isUpperCase,
        Collectors.mapping(
            String::valueOf,
            Collectors.joining()
        )
    ));

String uppercase = partitioned.get(true);   // "HW"
String lowercase = partitioned.get(false);  // "elloorld"

// Alternative
String upper = input.chars()
    .filter(Character::isUpperCase)
    .mapToObj(c -> String.valueOf((char) c))
    .collect(Collectors.joining());

String lower = input.chars()
    .filter(Character::isLowerCase)
    .mapToObj(c -> String.valueOf((char) c))
    .collect(Collectors.joining());
```
</details>

---

### 47. Sum of digits of a number
```java
int number = 12345;
// Output: 15
```

<details>
<summary>Show Answer</summary>

```java
int sum = String.valueOf(number)
    .chars()
    .map(Character::getNumericValue)
    .sum();
System.out.println(sum); // 15

// Alternative
int sum2 = String.valueOf(number)
    .chars()
    .map(c -> c - '0')
    .sum();
```
</details>

---

### 48. Check if string is palindrome
```java
String input = "madam";
// Output: true
```

<details>
<summary>Show Answer</summary>

```java
String reversed = input.chars()
    .mapToObj(c -> (char) c)
    .reduce("", (s, c) -> c + s, (s1, s2) -> s2 + s1);
boolean isPalindrome = input.equals(reversed);

// Alternative - more efficient
boolean isPalindrome2 = IntStream.range(0, input.length() / 2)
    .allMatch(i -> input.charAt(i) == input.charAt(input.length() - 1 - i));

// Using StringBuilder (non-stream)
boolean isPalindrome3 = input.equals(
    new StringBuilder(input).reverse().toString()
);
```
</details>

---

### 49. Find intersection of multiple lists
```java
List<Integer> list1 = List.of(1, 2, 3, 4, 5);
List<Integer> list2 = List.of(3, 4, 5, 6, 7);
List<Integer> list3 = List.of(4, 5, 6, 7, 8);
// Output: [4, 5]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> intersection = list1.stream()
    .filter(list2::contains)
    .filter(list3::contains)
    .collect(Collectors.toList());
System.out.println(intersection); // [4, 5]

// For variable number of lists
List<List<Integer>> allLists = List.of(list1, list2, list3);
List<Integer> result = allLists.get(0).stream()
    .filter(element -> allLists.stream()
        .allMatch(list -> list.contains(element)))
    .collect(Collectors.toList());
```
</details>

---

### 50. Find union of two lists (without duplicates)
```java
List<Integer> list1 = List.of(1, 2, 3, 4);
List<Integer> list2 = List.of(3, 4, 5, 6);
// Output: [1, 2, 3, 4, 5, 6]
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> union = Stream.concat(list1.stream(), list2.stream())
    .distinct()
    .collect(Collectors.toList());
System.out.println(union); // [1, 2, 3, 4, 5, 6]

// Sorted union
List<Integer> sortedUnion = Stream.concat(list1.stream(), list2.stream())
    .distinct()
    .sorted()
    .collect(Collectors.toList());
```
</details>

---

### 51. Calculate factorial using streams
```java
int n = 5;
// Output: 120
```

<details>
<summary>Show Answer</summary>

```java
int factorial = IntStream.rangeClosed(1, n)
    .reduce(1, (a, b) -> a * b);
System.out.println(factorial); // 120

// Using long for larger numbers
long factorial2 = LongStream.rangeClosed(1, n)
    .reduce(1, (a, b) -> a * b);
```
</details>

---

### 52. Generate Fibonacci series
```java
int n = 10;
// Generate first 10 Fibonacci numbers
```

<details>
<summary>Show Answer</summary>

```java
List<Integer> fibonacci = Stream.iterate(new int[]{0, 1}, 
        f -> new int[]{f[1], f[0] + f[1]})
    .limit(n)
    .map(f -> f[0])
    .collect(Collectors.toList());
System.out.println(fibonacci); 
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Alternative - without first element
List<Integer> fibonacci2 = Stream.iterate(new int[]{0, 1}, 
        f -> new int[]{f[1], f[0] + f[1]})
    .limit(n)
    .map(f -> f[1])
    .collect(Collectors.toList());
```
</details>

---

### 53. Remove special characters from string
```java
String input = "Hello@World#123!";
// Output: "HelloWorld123"
```

<details>
<summary>Show Answer</summary>

```java
String result = input.chars()
    .filter(Character::isLetterOrDigit)
    .mapToObj(c -> String.valueOf((char) c))
    .collect(Collectors.joining());
System.out.println(result); // HelloWorld123

// Only letters
String onlyLetters = input.chars()
    .filter(Character::isLetter)
    .mapToObj(c -> String.valueOf((char) c))
    .collect(Collectors.joining());
```
</details>

---

### 54. Check if list contains duplicates
```java
List<Integer> numbers = List.of(1, 2, 3, 2, 4);
// Output: true
```

<details>
<summary>Show Answer</summary>

```java
boolean hasDuplicates = numbers.size() != 
    numbers.stream().distinct().count();
System.out.println(hasDuplicates); // true

// Alternative
Set<Integer> set = new HashSet<>();
boolean hasDuplicates2 = numbers.stream()
    .anyMatch(n -> !set.add(n));
```
</details>

---

### 55. Merge two maps
```java
Map<String, Integer> map1 = Map.of("A", 1, "B", 2);
Map<String, Integer> map2 = Map.of("B", 3, "C", 4);
// Merge with value addition for common keys
// Output: {A=1, B=5, C=4}
```

<details>
<summary>Show Answer</summary>

```java
Map<String, Integer> merged = Stream.concat(
        map1.entrySet().stream(),
        map2.entrySet().stream()
    )
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        Integer::sum
    ));
System.out.println(merged); // {A=1, B=5, C=4}

// Keep first value for duplicates
Map<String, Integer> merged2 = Stream.concat(
        map1.entrySet().stream(),
        map2.entrySet().stream()
    )
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (v1, v2) -> v1
    ));
```
</details>

---

### 56. Filter employees by multiple conditions
```java
List<Employee> employees = /* list */;
// Filter: age > 30 AND department = "IT" AND salary > 50000
```

<details>
<summary>Show Answer</summary>

```java
List<Employee> filtered = employees.stream()
    .filter(e -> e.getAge() > 30)
    .filter(e -> "IT".equals(e.getDepartment()))
    .filter(e -> e.getSalary() > 50000)
    .collect(Collectors.toList());

// Alternative - single filter
List<Employee> filtered2 = employees.stream()
    .filter(e -> e.getAge() > 30 && 
                 "IT".equals(e.getDepartment()) && 
                 e.getSalary() > 50000)
    .collect(Collectors.toList());

// Using Predicate
Predicate<Employee> ageFilter = e -> e.getAge() > 30;
Predicate<Employee> deptFilter = e -> "IT".equals(e.getDepartment());
Predicate<Employee> salaryFilter = e -> e.getSalary() > 50000;

List<Employee> filtered3 = employees.stream()
    .filter(ageFilter.and(deptFilter).and(salaryFilter))
    .collect(Collectors.toList());
```
</details>

---

### 57. Find employees whose name starts with vowel
```java
List<Employee> employees = /* list */;
// Output: employees with names starting with A,E,I,O,U
```

<details>
<summary>Show Answer</summary>

```java
Set<Character> vowels = Set.of('A', 'E', 'I', 'O', 'U', 
                                'a', 'e', 'i', 'o', 'u');

List<Employee> result = employees.stream()
    .filter(e -> vowels.contains(e.getName().charAt(0)))
    .collect(Collectors.toList());

// Alternative
List<Employee> result2 = employees.stream()
    .filter(e -> "AEIOUaeiou".indexOf(e.getName().charAt(0)) != -1)
    .collect(Collectors.toList());

// Case insensitive
List<Employee> result3 = employees.stream()
    .filter(e -> "AEIOU".contains(
        e.getName().substring(0, 1).toUpperCase()))
    .collect(Collectors.toList());
```
</details>

---

### 58. Calculate age from date of birth
```java
LocalDate dob = LocalDate.of(1990, 5, 15);
// Output: current age
```

<details>
<summary>Show Answer</summary>

```java
import java.time.LocalDate;
import java.time.Period;

// Single date
int age = Period.between(dob, LocalDate.now()).getYears();
System.out.println(age);

// For list of persons
class Person {
    private String name;
    private LocalDate dob;
    // getters, setters
}

List<Person> persons = /* list */;

// Add age to each person (assuming Person has setAge method)
persons.stream()
    .forEach(p -> p.setAge(
        Period.between(p.getDob(), LocalDate.now()).getYears()
    ));

// Filter persons above 30
List<Person> above30 = persons.stream()
    .filter(p -> Period.between(p.getDob(), LocalDate.now()).getYears() > 30)
    .collect(Collectors.toList());
```
</details>

---

### 59. Get distinct elements from list of objects by property
```java
class Employee {
    private String department;
    private String name;
    // constructors, getters
}
List<Employee> employees = /* list with duplicate departments */;
// Get distinct departments
```

<details>
<summary>Show Answer</summary>

```java
// Get distinct department names
List<String> distinctDepts = employees.stream()
    .map(Employee::getDepartment)
    .distinct()
    .collect(Collectors.toList());

// Get one employee per department (first occurrence)
Map<String, Employee> onePerDept = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Function.identity(),
        (e1, e2) -> e1
    ));

// Using custom comparator for distinct objects
List<Employee> distinctByDept = employees.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toMap(
            Employee::getDepartment,
            Function.identity(),
            (e1, e2) -> e1
        ),
        map -> new ArrayList<>(map.values())
    ));
```
</details>

---

### 60. Custom sorting with multiple fields
```java
class Employee {
    private String department;
    private double salary;
    private String name;
    // constructors, getters
}
List<Employee> employees = /* list */;
// Sort by department (asc) then by salary (desc)
```

<details>
<summary>Show Answer</summary>

```java
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Comparator.comparing(Employee::getSalary).reversed()))
    .collect(Collectors.toList());

// Sort by department (asc), then salary (desc), then name (asc)
List<Employee> sorted2 = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Comparator.comparing(Employee::getSalary).reversed())
                      .thenComparing(Employee::getName))
    .collect(Collectors.toList());

// Using multiple comparators
Comparator<Employee> byDept = Comparator.comparing(Employee::getDepartment);
Comparator<Employee> bySalaryDesc = Comparator.comparing(Employee::getSalary).reversed();
Comparator<Employee> byName = Comparator.comparing(Employee::getName);

List<Employee> sorted3 = employees.stream()
    .sorted(byDept.thenComparing(bySalaryDesc).thenComparing(byName))
    .collect(Collectors.toList());
```
</details>

---

## EXPERT LEVEL - Spring Boot Specific (61-70)

### 61. Filter Spring beans by annotation
```java
ApplicationContext context = /* spring context */;
// Get all beans annotated with @Service
```

<details>
<summary>Show Answer</summary>

```java
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;

Map<String, Object> serviceBeans = Arrays.stream(context.getBeanDefinitionNames())
    .filter(name -> context.findAnnotationOnBean(name, Service.class) != null)
    .collect(Collectors.toMap(
        Function.identity(),
        context::getBean
    ));

// Get only bean names
List<String> serviceBeanNames = Arrays.stream(context.getBeanDefinitionNames())
    .filter(name -> context.findAnnotationOnBean(name, Service.class) != null)
    .collect(Collectors.toList());

// Get beans of specific type
Map<String, MyService> myServiceBeans = context.getBeansOfType(MyService.class);
```
</details>

---

### 62. Convert list of DTOs to Entities
```java
class UserDTO {
    private String username;
    private String email;
    // getters, setters
}

class User {
    private String username;
    private String email;
    // getters, setters
}

List<UserDTO> userDTOs = /* list */;
// Convert to List<User>
```

<details>
<summary>Show Answer</summary>

```java
// Using ModelMapper
import org.modelmapper.ModelMapper;

ModelMapper modelMapper = new ModelMapper();

List<User> users = userDTOs.stream()
    .map(dto -> modelMapper.map(dto, User.class))
    .collect(Collectors.toList());

// Manual mapping
List<User> users2 = userDTOs.stream()
    .map(dto -> {
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        return user;
    })
    .collect(Collectors.toList());

// Using constructor
List<User> users3 = userDTOs.stream()
    .map(dto -> new User(dto.getUsername(), dto.getEmail()))
    .collect(Collectors.toList());

// Using MapStruct (interface)
@Mapper
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
    User toEntity(UserDTO dto);
    
    default List<User> toEntities(List<UserDTO> dtos) {
        return dtos.stream()
            .map(this::toEntity)
            .collect(Collectors.toList());
    }
}
```
</details>

---

### 63. Group API responses by status code
```java
class ApiResponse {
    private int statusCode;
    private String message;
    private Object data;
    // getters, setters
}
List<ApiResponse> responses = /* list */;
// Output: Map<Integer, List<ApiResponse>>
```

<details>
<summary>Show Answer</summary>

```java
Map<Integer, List<ApiResponse>> grouped = responses.stream()
    .collect(Collectors.groupingBy(ApiResponse::getStatusCode));

// Get count per status code
Map<Integer, Long> countByStatus = responses.stream()
    .collect(Collectors.groupingBy(
        ApiResponse::getStatusCode,
        Collectors.counting()
    ));

// Get only successful responses (2xx)
List<ApiResponse> successful = responses.stream()
    .filter(r -> r.getStatusCode() >= 200 && r.getStatusCode() < 300)
    .collect(Collectors.toList());

// Group by success/failure
Map<Boolean, List<ApiResponse>> partitioned = responses.stream()
    .collect(Collectors.partitioningBy(
        r -> r.getStatusCode() >= 200 && r.getStatusCode() < 300
    ));
```
</details>

---

### 64. Extract unique tags from list of articles
```java
class Article {
    private String title;
    private List<String> tags;
    // getters, setters
}
List<Article> articles = /* each article has List<String> tags */;
// Output: Set<String> of all unique tags
```

<details>
<summary>Show Answer</summary>

```java
Set<String> uniqueTags = articles.stream()
    .flatMap(article -> article.getTags().stream())
    .collect(Collectors.toSet());

// Sorted unique tags
List<String> sortedUniqueTags = articles.stream()
    .flatMap(article -> article.getTags().stream())
    .distinct()
    .sorted()
    .collect(Collectors.toList());

// Count frequency of each tag
Map<String, Long> tagFrequency = articles.stream()
    .flatMap(article -> article.getTags().stream())
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));

// Find most popular tag
String mostPopularTag = articles.stream()
    .flatMap(article -> article.getTags().stream())
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .map(Map.Entry::getKey)
    .orElse(null);
```
</details>

---

### 65. Calculate total price from cart items
```java
class CartItem {
    private String product;
    private int quantity;
    private double price;
    // getters, setters
}
List<CartItem> items = /* each has quantity and price */;
// Output: total amount
```

<details>
<summary>Show Answer</summary>

```java
double total = items.stream()
    .mapToDouble(item -> item.getQuantity() * item.getPrice())
    .sum();

// Using reduce
double total2 = items.stream()
    .map(item -> item.getQuantity() * item.getPrice())
    .reduce(0.0, Double::sum);

// Get subtotals per item
Map<String, Double> subtotals = items.stream()
    .collect(Collectors.toMap(
        CartItem::getProduct,
        item -> item.getQuantity() * item.getPrice()
    ));

// Add discount logic
double totalWithDiscount = items.stream()
    .mapToDouble(item -> {
        double subtotal = item.getQuantity() * item.getPrice();
        return item.getQuantity() >= 10 ? subtotal * 0.9 : subtotal;
    })
    .sum();
```
</details>

---

### 66. Group transactions by date
```java
class Transaction {
    private String id;
    private LocalDateTime timestamp;
    private double amount;
    // getters, setters
}
List<Transaction> transactions = /* list with LocalDateTime */;
// Output: Map<LocalDate, List<Transaction>>
```

<details>
<summary>Show Answer</summary>

```java
Map<LocalDate, List<Transaction>> groupedByDate = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.getTimestamp().toLocalDate()
    ));

// Sum amounts per day
Map<LocalDate, Double> totalPerDay = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.getTimestamp().toLocalDate(),
        Collectors.summingDouble(Transaction::getAmount)
    ));

// Count transactions per day
Map<LocalDate, Long> countPerDay = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.getTimestamp().toLocalDate(),
        Collectors.counting()
    ));

// Group by month
Map<String, List<Transaction>> groupedByMonth = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.getTimestamp().getMonth().toString()
    ));
```
</details>

---

### 67. Find products out of stock
```java
class Product {
    private String name;
    private int stock;
    private double price;
    // getters, setters
}
List<Product> products = /* list with stock quantity */;
// Output: products where stock == 0
```

<details>
<summary>Show Answer</summary>

```java
List<Product> outOfStock = products.stream()
    .filter(p -> p.getStock() == 0)
    .collect(Collectors.toList());

// Low stock (less than 10)
List<Product> lowStock = products.stream()
    .filter(p -> p.getStock() > 0 && p.getStock() < 10)
    .collect(Collectors.toList());

// Partition by stock availability
Map<Boolean, List<Product>> partitioned = products.stream()
    .collect(Collectors.partitioningBy(p -> p.getStock() > 0));

List<Product> inStock = partitioned.get(true);
List<Product> outOfStockList = partitioned.get(false);

// Get product names that are out of stock
List<String> outOfStockNames = products.stream()
    .filter(p -> p.getStock() == 0)
    .map(Product::getName)
    .collect(Collectors.toList());
```
</details>

---

### 68. Calculate student average marks and filter passed students
```java
class Student {
    private String name;
    private List<Integer> marks;
    // getters, setters
}
List<Student> students = /* list with List<Integer> marks */;
// Filter students with average > 40
```

<details>
<summary>Show Answer</summary>

```java
List<Student> passedStudents = students.stream()
    .filter(s -> s.getMarks().stream()
                   .mapToInt(Integer::intValue)
                   .average()
                   .orElse(0.0) > 40)
    .collect(Collectors.toList());

// Get student name with average
Map<String, Double> studentAverages = students.stream()
    .collect(Collectors.toMap(
        Student::getName,
        s -> s.getMarks().stream()
              .mapToInt(Integer::intValue)
              .average()
              .orElse(0.0)
    ));

// Get top 3 students by average
List<Student> top3 = students.stream()
    .sorted(Comparator.comparing(
        s -> s.getMarks().stream()
              .mapToInt(Integer::intValue)
              .average()
              .orElse(0.0),
        Comparator.reverseOrder()
    ))
    .limit(3)
    .collect(Collectors.toList());

// Partition passed/failed
Map<Boolean, List<Student>> results = students.stream()
    .collect(Collectors.partitioningBy(
        s -> s.getMarks().stream()
              .mapToInt(Integer::intValue)
              .average()
              .orElse(0.0) > 40
    ));
```
</details>

---

### 69. Combine two lists of objects by common field
```java
class Order {
    private String orderId;
    private Integer customerId;
    private double amount;
    // getters, setters
}

class Customer {
    private Integer id;
    private String name;
    // getters, setters
}

List<Order> orders = /* list with customerId */;
List<Customer> customers = /* list with id */;
// Join orders with customers
```

<details>
<summary>Show Answer</summary>

```java
class OrderWithCustomer {
    private Order order;
    private Customer customer;
    // constructor, getters, setters
}

// Create map for faster lookup
Map<Integer, Customer> customerMap = customers.stream()
    .collect(Collectors.toMap(Customer::getId, Function.identity()));

// Join orders with customers
List<OrderWithCustomer> joined = orders.stream()
    .map(order -> new OrderWithCustomer(
        order,
        customerMap.get(order.getCustomerId())
    ))
    .filter(owc -> owc.getCustomer() != null)
    .collect(Collectors.toList());

// Get order with customer name
class OrderDetails {
    private String orderId;
    private String customerName;
    private double amount;
    // constructor, getters
}

List<OrderDetails> orderDetails = orders.stream()
    .map(order -> {
        Customer customer = customerMap.get(order.getCustomerId());
        return new OrderDetails(
            order.getOrderId(),
            customer != null ? customer.getName() : "Unknown",
            order.getAmount()
        );
    })
    .collect(Collectors.toList());

// Group orders by customer
Map<String, List<Order>> ordersByCustomer = orders.stream()
    .filter(order -> customerMap.containsKey(order.getCustomerId()))
    .collect(Collectors.groupingBy(
        order -> customerMap.get(order.getCustomerId()).getName()
    ));
```
</details>

---

### 70. Build hierarchical structure from flat list
```java
class Employee {
    private Integer id;
    private String name;
    private Integer managerId;
    // getters, setters
}
List<Employee> employees = /* list with managerId */;
// Create manager -> List<Employee> reporting structure
```

<details>
<summary>Show Answer</summary>

```java
// Group employees by manager
Map<Integer, List<Employee>> reportingStructure = employees.stream()
    .filter(e -> e.getManagerId() != null)
    .collect(Collectors.groupingBy(Employee::getManagerId));

// Get all employees under a specific manager
Integer managerId = 1;
List<Employee> directReports = reportingStructure.getOrDefault(managerId, 
    Collections.emptyList());

// Find all managers (employees who have reports)
List<Employee> managers = employees.stream()
    .filter(e -> reportingStructure.containsKey(e.getId()))
    .collect(Collectors.toList());

// Create hierarchical structure
class EmployeeNode {
    private Employee employee;
    private List<EmployeeNode> subordinates;
    
    public EmployeeNode(Employee employee) {
        this.employee = employee;
        this.subordinates = new ArrayList<>();
    }
    // getters, setters
}

// Build tree (for root employees - no manager)
List<EmployeeNode> buildTree(List<Employee> allEmployees) {
    Map<Integer, List<Employee>> map = allEmployees.stream()
        .filter(e -> e.getManagerId() != null)
        .collect(Collectors.groupingBy(Employee::getManagerId));
    
    return allEmployees.stream()
        .filter(e -> e.getManagerId() == null)
        .map(e -> buildNode(e, map))
        .collect(Collectors.toList());
}

EmployeeNode buildNode(Employee emp, Map<Integer, List<Employee>> map) {
    EmployeeNode node = new EmployeeNode(emp);
    List<Employee> subordinates = map.getOrDefault(emp.getId(), 
        Collections.emptyList());
    node.setSubordinates(
        subordinates.stream()
            .map(e -> buildNode(e, map))
            .collect(Collectors.toList())
    );
    return node;
}
```
</details>

---

**Happy Coding! 🚀**
