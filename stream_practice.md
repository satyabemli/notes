# Java Stream API Coding Questions with Complete Solutions

## Online practice for Java Streams — just copy the solution, paste it into this site: [https://www.codingshuttle.com/compilers/java/](https://www.codingshuttle.com/compilers/java/), then compile and run it to get the result.


## BASIC LEVEL (1-15)

### 1. Find unique/distinct numbers from an array
```java
List<Integer> numbers = List.of(1,2,3,2,4,5,4);
// Output: [1, 2, 3, 4, 5]
```

<details>
<summary>Show Answer</summary>

```java
import java.util.*;
import java.util.stream.*;

public class UniqueNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 2, 4, 5, 4);
        
        List<Integer> unique = numbers.stream()
            .distinct()
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Unique: " + unique);
    }
}
```
**Output:**
```
Original: [1, 2, 3, 2, 4, 5, 4]
Unique: [1, 2, 3, 4, 5]
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
import java.util.*;
import java.util.stream.*;

public class FilterEvenNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        List<Integer> evenNumbers = numbers.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Even Numbers: " + evenNumbers);
    }
}
```
**Output:**
```
Original: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Even Numbers: [2, 4, 6, 8, 10]
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
import java.util.*;
import java.util.stream.*;

public class SumOfNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);
        
        // Method 1: Using mapToInt and sum
        int sum = numbers.stream()
            .mapToInt(Integer::intValue)
            .sum();
        
        // Method 2: Using reduce
        int sum2 = numbers.stream()
            .reduce(0, Integer::sum);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Sum (Method 1): " + sum);
        System.out.println("Sum (Method 2): " + sum2);
    }
}
```
**Output:**
```
Numbers: [1, 2, 3, 4, 5]
Sum (Method 1): 15
Sum (Method 2): 15
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
import java.util.*;
import java.util.stream.*;

public class StringToUpperCase {
    public static void main(String[] args) {
        List<String> words = List.of("java", "spring", "boot");
        
        List<String> upperCase = words.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());
        
        System.out.println("Original: " + words);
        System.out.println("Uppercase: " + upperCase);
    }
}
```
**Output:**
```
Original: [java, spring, boot]
Uppercase: [JAVA, SPRING, BOOT]
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
import java.util.*;
import java.util.stream.*;

public class MaxNumber {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(10, 5, 20, 15, 30);
        
        // Method 1: Using max with mapToInt
        int max = numbers.stream()
            .mapToInt(Integer::intValue)
            .max()
            .orElse(0);
        
        // Method 2: Using max with comparator
        Optional<Integer> maxOpt = numbers.stream()
            .max(Integer::compareTo);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Maximum (Method 1): " + max);
        System.out.println("Maximum (Method 2): " + maxOpt.orElse(0));
    }
}
```
**Output:**
```
Numbers: [10, 5, 20, 15, 30]
Maximum (Method 1): 30
Maximum (Method 2): 30
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
import java.util.*;
import java.util.stream.*;

public class MinNumber {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(10, 5, 20, 15, 30);
        
        // Method 1: Using min with mapToInt
        int min = numbers.stream()
            .mapToInt(Integer::intValue)
            .min()
            .orElse(0);
        
        // Method 2: Using min with comparator
        Optional<Integer> minOpt = numbers.stream()
            .min(Integer::compareTo);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Minimum (Method 1): " + min);
        System.out.println("Minimum (Method 2): " + minOpt.orElse(0));
    }
}
```
**Output:**
```
Numbers: [10, 5, 20, 15, 30]
Minimum (Method 1): 5
Minimum (Method 2): 5
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
import java.util.*;
import java.util.stream.*;

public class AverageOfNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(10, 20, 30, 40, 50);
        
        double average = numbers.stream()
            .mapToInt(Integer::intValue)
            .average()
            .orElse(0.0);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Average: " + average);
    }
}
```
**Output:**
```
Numbers: [10, 20, 30, 40, 50]
Average: 30.0
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
import java.util.*;
import java.util.stream.*;

public class CountElements {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "orange");
        
        long count = words.stream()
            .count();
        
        System.out.println("Words: " + words);
        System.out.println("Count: " + count);
    }
}
```
**Output:**
```
Words: [apple, banana, orange]
Count: 3
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
import java.util.*;
import java.util.stream.*;

public class JoinStrings {
    public static void main(String[] args) {
        List<String> words = List.of("Java", "Spring", "Boot");
        
        // Simple join
        String joined = words.stream()
            .collect(Collectors.joining(", "));
        
        // With prefix and suffix
        String withBrackets = words.stream()
            .collect(Collectors.joining(", ", "[", "]"));
        
        System.out.println("Words: " + words);
        System.out.println("Joined: " + joined);
        System.out.println("With Brackets: " + withBrackets);
    }
}
```
**Output:**
```
Words: [Java, Spring, Boot]
Joined: Java, Spring, Boot
With Brackets: [Java, Spring, Boot]
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
import java.util.*;
import java.util.stream.*;

public class SquareNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(2, 3, 4, 5);
        
        List<Integer> squared = numbers.stream()
            .map(n -> n * n)
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Squared: " + squared);
    }
}
```
**Output:**
```
Original: [2, 3, 4, 5]
Squared: [4, 9, 16, 25]
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
import java.util.*;
import java.util.stream.*;

public class FilterByStartingLetter {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "avocado", "orange");
        
        // Case sensitive
        List<String> filtered = words.stream()
            .filter(word -> word.startsWith("a"))
            .collect(Collectors.toList());
        
        // Case insensitive
        List<String> filteredIgnoreCase = words.stream()
            .filter(word -> word.toLowerCase().startsWith("a"))
            .collect(Collectors.toList());
        
        System.out.println("Original: " + words);
        System.out.println("Starting with 'a': " + filtered);
        System.out.println("Starting with 'a' (ignore case): " + filteredIgnoreCase);
    }
}
```
**Output:**
```
Original: [apple, banana, avocado, orange]
Starting with 'a': [apple, avocado]
Starting with 'a' (ignore case): [apple, avocado]
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
import java.util.*;
import java.util.stream.*;

public class RemoveNullValues {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("Java", null, "Spring", null, "Boot");
        
        List<String> nonNull = words.stream()
            .filter(Objects::nonNull)
            .collect(Collectors.toList());
        
        System.out.println("Original: " + words);
        System.out.println("Non-null: " + nonNull);
    }
}
```
**Output:**
```
Original: [Java, null, Spring, null, Boot]
Non-null: [Java, Spring, Boot]
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
import java.util.*;
import java.util.stream.*;

public class StringToIntArray {
    public static void main(String[] args) {
        String[] strArray = {"1", "2", "3", "4", "5"};
        
        // To int array
        int[] intArray = Arrays.stream(strArray)
            .mapToInt(Integer::parseInt)
            .toArray();
        
        // To List<Integer>
        List<Integer> intList = Arrays.stream(strArray)
            .map(Integer::parseInt)
            .collect(Collectors.toList());
        
        System.out.println("String Array: " + Arrays.toString(strArray));
        System.out.println("Int Array: " + Arrays.toString(intArray));
        System.out.println("Int List: " + intList);
    }
}
```
**Output:**
```
String Array: [1, 2, 3, 4, 5]
Int Array: [1, 2, 3, 4, 5]
Int List: [1, 2, 3, 4, 5]
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
import java.util.*;
import java.util.stream.*;

public class SortAscending {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(5, 2, 8, 1, 9);
        
        // Method 1: Simple sorted
        List<Integer> sorted = numbers.stream()
            .sorted()
            .collect(Collectors.toList());
        
        // Method 2: Using Comparator
        List<Integer> sortedAsc = numbers.stream()
            .sorted(Comparator.naturalOrder())
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Sorted: " + sorted);
    }
}
```
**Output:**
```
Original: [5, 2, 8, 1, 9]
Sorted: [1, 2, 5, 8, 9]
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
import java.util.*;
import java.util.stream.*;

public class SortDescending {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(5, 2, 8, 1, 9);
        
        // Method 1: Using reverseOrder
        List<Integer> sortedDesc = numbers.stream()
            .sorted(Comparator.reverseOrder())
            .collect(Collectors.toList());
        
        // Method 2: Using lambda
        List<Integer> sorted = numbers.stream()
            .sorted((a, b) -> b - a)
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Sorted Descending: " + sortedDesc);
    }
}
```
**Output:**
```
Original: [5, 2, 8, 1, 9]
Sorted Descending: [9, 8, 5, 2, 1]
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
import java.util.*;
import java.util.stream.*;

public class FindDuplicates {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 2, 4, 6, 7, 4);
        
        // Method 1: Using Set
        Set<Integer> seen = new HashSet<>();
        List<Integer> duplicates = numbers.stream()
            .filter(n -> !seen.add(n))
            .distinct()
            .collect(Collectors.toList());
        
        // Method 2: Using Collections.frequency
        List<Integer> duplicates2 = numbers.stream()
            .filter(n -> Collections.frequency(numbers, n) > 1)
            .distinct()
            .collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Duplicates: " + duplicates);
    }
}
```
**Output:**
```
Original: [1, 2, 3, 2, 4, 6, 7, 4]
Duplicates: [2, 4]
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
import java.util.*;
import java.util.stream.*;

public class FirstRepeatedCharacter {
    public static void main(String[] args) {
        String input = "hello world";
        
        Set<Character> seen = new HashSet<>();
        Character result = input.chars()
            .mapToObj(c -> (char) c)
            .filter(c -> !seen.add(c))
            .findFirst()
            .orElse(null);
        
        System.out.println("Input: " + input);
        System.out.println("First Repeated Character: " + result);
    }
}
```
**Output:**
```
Input: hello world
First Repeated Character: l
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
import java.util.*;
import java.util.function.Function;
import java.util.stream.*;

public class FirstNonRepeatedCharacter {
    public static void main(String[] args) {
        String input = "hello";
        
        Character result = input.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(Function.identity(), 
                     LinkedHashMap::new, Collectors.counting()))
            .entrySet().stream()
            .filter(e -> e.getValue() == 1)
            .map(Map.Entry::getKey)
            .findFirst()
            .orElse(null);
        
        System.out.println("Input: " + input);
        System.out.println("First Non-Repeated Character: " + result);
    }
}
```
**Output:**
```
Input: hello
First Non-Repeated Character: h
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
import java.util.*;
import java.util.function.Function;
import java.util.stream.*;

public class CharacterFrequency {
    public static void main(String[] args) {
        String input = "bcbca";
        
        String result = input.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(Function.identity(), 
                     LinkedHashMap::new, Collectors.counting()))
            .entrySet().stream()
            .map(e -> e.getKey() + "" + e.getValue())
            .collect(Collectors.joining());
        
        System.out.println("Input: " + input);
        System.out.println("Character Frequency: " + result);
    }
}
```
**Output:**
```
Input: bcbca
Character Frequency: b2c2a1
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
import java.util.*;
import java.util.stream.*;

public class WordsStartingWithB {
    public static void main(String[] args) {
        String input = "baby bird asked about blue ants";
        
        String result = Arrays.stream(input.split(" "))
            .filter(word -> word.startsWith("b"))
            .collect(Collectors.joining(" "));
        
        System.out.println("Input: " + input);
        System.out.println("Words starting with 'b': " + result);
    }
}
```
**Output:**
```
Input: baby bird asked about blue ants
Words starting with 'b': baby bird blue
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
import java.util.*;
import java.util.stream.*;

public class MoveZerosToRight {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 0, -3, 0, 5, -2, 0, 8, 0, -4);
        
        List<Integer> result = Stream.concat(
            numbers.stream().filter(n -> n != 0),
            numbers.stream().filter(n -> n == 0)
        ).collect(Collectors.toList());
        
        System.out.println("Original: " + numbers);
        System.out.println("Zeros Moved: " + result);
    }
}
```
**Output:**
```
Original: [1, 0, -3, 0, 5, -2, 0, 8, 0, -4]
Zeros Moved: [1, -3, 5, -2, 8, -4, 0, 0, 0, 0]
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
import java.util.*;
import java.util.stream.*;

public class MergeTwoArrays {
    public static void main(String[] args) {
        int[] arr1 = {1, 2, 3};
        int[] arr2 = {4, 5, 6};
        
        // To int array
        int[] merged = IntStream.concat(
            Arrays.stream(arr1), 
            Arrays.stream(arr2)
        ).toArray();
        
        // To List<Integer>
        List<Integer> mergedList = Stream.concat(
            Arrays.stream(arr1).boxed(),
            Arrays.stream(arr2).boxed()
        ).collect(Collectors.toList());
        
        System.out.println("Array 1: " + Arrays.toString(arr1));
        System.out.println("Array 2: " + Arrays.toString(arr2));
        System.out.println("Merged Array: " + Arrays.toString(merged));
        System.out.println("Merged List: " + mergedList);
    }
}
```
**Output:**
```
Array 1: [1, 2, 3]
Array 2: [4, 5, 6]
Merged Array: [1, 2, 3, 4, 5, 6]
Merged List: [1, 2, 3, 4, 5, 6]
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
import java.util.*;
import java.util.stream.*;

public class SecondHighest {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(10, 20, 30, 40, 50);
        
        Integer secondHighest = numbers.stream()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst()
            .orElse(null);
        
        // With distinct to handle duplicates
        List<Integer> numbersWithDup = List.of(10, 50, 30, 50, 20);
        Integer secondHighest2 = numbersWithDup.stream()
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst()
            .orElse(null);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Second Highest: " + secondHighest);
        System.out.println("\nNumbers with duplicates: " + numbersWithDup);
        System.out.println("Second Highest (distinct): " + secondHighest2);
    }
}
```
**Output:**
```
Numbers: [10, 20, 30, 40, 50]
Second Highest: 40

Numbers with duplicates: [10, 50, 30, 50, 20]
Second Highest (distinct): 30
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
import java.util.*;
import java.util.stream.*;

public class SecondLowest {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(10, 20, 30, 40, 50);
        
        Integer secondLowest = numbers.stream()
            .sorted()
            .skip(1)
            .findFirst()
            .orElse(null);
        
        // With distinct
        List<Integer> numbersWithDup = List.of(10, 10, 30, 20, 50);
        Integer secondLowest2 = numbersWithDup.stream()
            .distinct()
            .sorted()
            .skip(1)
            .findFirst()
            .orElse(null);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Second Lowest: " + secondLowest);
        System.out.println("\nNumbers with duplicates: " + numbersWithDup);
        System.out.println("Second Lowest (distinct): " + secondLowest2);
    }
}
```
**Output:**
```
Numbers: [10, 20, 30, 40, 50]
Second Lowest: 20

Numbers with duplicates: [10, 10, 30, 20, 50]
Second Lowest (distinct): 20
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
import java.util.*;
import java.util.stream.*;

public class SumOfEvenNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Method 1: Using filter and sum
        int sum = numbers.stream()
            .filter(n -> n % 2 == 0)
            .mapToInt(Integer::intValue)
            .sum();
        
        // Method 2: Using reduce
        int sum2 = numbers.stream()
            .filter(n -> n % 2 == 0)
            .reduce(0, Integer::sum);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Sum of Even Numbers: " + sum);
    }
}
```
**Output:**
```
Numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Sum of Even Numbers: 30
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
import java.util.*;
import java.util.stream.*;

public class SumOfOddNumbers {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        int sum = numbers.stream()
            .filter(n -> n % 2 != 0)
            .mapToInt(Integer::intValue)
            .sum();
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Sum of Odd Numbers: " + sum);
    }
}
```
**Output:**
```
Numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Sum of Odd Numbers: 25
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
import java.util.*;
import java.util.stream.*;

public class PartitionEvenOdd {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8);
        
        Map<Boolean, List<Integer>> partitioned = numbers.stream()
            .collect(Collectors.partitioningBy(n -> n % 2 == 0));
        
        List<Integer> evenNumbers = partitioned.get(true);
        List<Integer> oddNumbers = partitioned.get(false);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Partitioned: " + partitioned);
        System.out.println("Even Numbers: " + evenNumbers);
        System.out.println("Odd Numbers: " + oddNumbers);
    }
}
```
**Output:**
```
Numbers: [1, 2, 3, 4, 5, 6, 7, 8]
Partitioned: {false=[1, 3, 5, 7], true=[2, 4, 6, 8]}
Even Numbers: [2, 4, 6, 8]
Odd Numbers: [1, 3, 5, 7]
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
import java.util.*;
import java.util.function.Function;
import java.util.stream.*;

public class ElementFrequency {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "apple", "orange", "banana");
        
        Map<String, Long> frequency = words.stream()
            .collect(Collectors.groupingBy(
                Function.identity(),
                Collectors.counting()
            ));
        
        // For ordered output
        Map<String, Long> orderedFrequency = words.stream()
            .collect(Collectors.groupingBy(
                Function.identity(),
                LinkedHashMap::new,
                Collectors.counting()
            ));
        
        System.out.println("Words: " + words);
        System.out.println("Frequency: " + frequency);
        System.out.println("Ordered Frequency: " + orderedFrequency);
    }
}
```
**Output:**
```
Words: [apple, banana, apple, orange, banana]
Frequency: {banana=2, orange=1, apple=2}
Ordered Frequency: {apple=2, banana=2, orange=1}
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
import java.util.*;
import java.util.stream.*;

public class NumbersStartingWithDigit {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(12, 23, 34, 21, 45, 56);
        
        // Find numbers starting with 2
        List<Integer> result = numbers.stream()
            .filter(n -> String.valueOf(n).startsWith("2"))
            .collect(Collectors.toList());
        
        // Generic method for any digit
        int digit = 2;
        List<Integer> result2 = numbers.stream()
            .filter(n -> String.valueOf(n).startsWith(String.valueOf(digit)))
            .collect(Collectors.toList());
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Starting with 2: " + result);
    }
}
```
**Output:**
```
Numbers: [12, 23, 34, 21, 45, 56]
Starting with 2: [23, 21]
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
import java.util.*;
import java.util.stream.*;

public class ReverseEachWord {
    public static void main(String[] args) {
        String input = "Hello World Java";
        
        String result = Arrays.stream(input.split(" "))
            .map(word -> new StringBuilder(word).reverse().toString())
            .collect(Collectors.joining(" "));
        
        System.out.println("Input: " + input);
        System.out.println("Reversed Each Word: " + result);
    }
}
```
**Output:**
```
Input: Hello World Java
Reversed Each Word: olleH dlroW avaJ
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
import java.util.*;
import java.util.stream.*;

public class CommonElements {
    public static void main(String[] args) {
        List<Integer> list1 = List.of(1, 2, 3, 4, 5);
        List<Integer> list2 = List.of(4, 5, 6, 7, 8);
        
        // Method 1: Simple contains
        List<Integer> common = list1.stream()
            .filter(list2::contains)
            .collect(Collectors.toList());
        
        // Method 2: More efficient for large lists
        Set<Integer> set2 = new HashSet<>(list2);
        List<Integer> common2 = list1.stream()
            .filter(set2::contains)
            .collect(Collectors.toList());
        
        System.out.println("List 1: " + list1);
        System.out.println("List 2: " + list2);
        System.out.println("Common Elements: " + common);
    }
}
```
**Output:**
```
List 1: [1, 2, 3, 4, 5]
List 2: [4, 5, 6, 7, 8]
Common Elements: [4, 5]
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
import java.util.*;
import java.util.stream.*;

public class SquareAndFilter {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(2, 5, 7, 8, 10);
        
        List<Integer> result = numbers.stream()
            .map(n -> n * n)
            .filter(n -> n > 50)
            .collect(Collectors.toList());
        
        System.out.println("Original Numbers: " + numbers);
        System.out.println("Squared and > 50: " + result);
    }
}
```
**Output:**
```
Original Numbers: [2, 5, 7, 8, 10]
Squared and > 50: [64, 100]
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
import java.util.*;
import java.util.stream.*;

public class Pagination {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Skip 3, take 3
        List<Integer> page = numbers.stream()
            .skip(3)
            .limit(3)
            .collect(Collectors.toList());
        
        System.out.println("All Numbers: " + numbers);
        System.out.println("Skip 3, Take 3: " + page);
        
        // Generic pagination
        int pageNumber = 2;
        int pageSize = 3;
        List<Integer> pageResult = getPage(numbers, pageNumber, pageSize);
        System.out.println("Page " + pageNumber + " (size " + pageSize + "): " + pageResult);
    }
    
    public static <T> List<T> getPage(List<T> list, int pageNumber, int pageSize) {
        return list.stream()
            .skip((long) (pageNumber - 1) * pageSize)
            .limit(pageSize)
            .collect(Collectors.toList());
    }
}
```
**Output:**
```
All Numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
Skip 3, Take 3: [4, 5, 6]
Page 2 (size 3): [4, 5, 6]
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
import java.util.*;
import java.util.stream.*;

public class AllMatch {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(2, 4, 6, 8, 10);
        List<Integer> mixedNumbers = List.of(2, 4, 5, 8, 10);
        
        boolean allEven = numbers.stream()
            .allMatch(n -> n % 2 == 0);
        
        boolean allEven2 = mixedNumbers.stream()
            .allMatch(n -> n % 2 == 0);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("All Even: " + allEven);
        System.out.println("\nMixed Numbers: " + mixedNumbers);
        System.out.println("All Even: " + allEven2);
    }
}
```
**Output:**
```
Numbers: [2, 4, 6, 8, 10]
All Even: true

Mixed Numbers: [2, 4, 5, 8, 10]
All Even: false
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
import java.util.*;
import java.util.stream.*;

public class AnyMatch {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 3, 5, 6, 7);
        List<Integer> oddNumbers = List.of(1, 3, 5, 7, 9);
        
        boolean anyEven = numbers.stream()
            .anyMatch(n -> n % 2 == 0);
        
        boolean anyEven2 = oddNumbers.stream()
            .anyMatch(n -> n % 2 == 0);
        
        // noneMatch - opposite of anyMatch
        boolean noneEven = oddNumbers.stream()
            .noneMatch(n -> n % 2 == 0);
        
        System.out.println("Numbers: " + numbers);
        System.out.println("Any Even: " + anyEven);
        System.out.println("\nOdd Numbers: " + oddNumbers);
        System.out.println("Any Even: " + anyEven2);
        System.out.println("None Even: " + noneEven);
    }
}
```
**Output:**
```
Numbers: [1, 3, 5, 6, 7]
Any Even: true

Odd Numbers: [1, 3, 5, 7, 9]
Any Even: false
None Even: true
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
import java.util.*;
import java.util.stream.*;

public class FlattenLists {
    public static void main(String[] args) {
        List<List<Integer>> listOfLists = List.of(
            List.of(1, 2, 3),
            List.of(4, 5),
            List.of(6, 7, 8, 9)
        );
        
        List<Integer> flattened = listOfLists.stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());
        
        // For array of arrays
        Integer[][] arrayOfArrays = {{1, 2, 3}, {4, 5}, {6, 7, 8, 9}};
        List<Integer> flattened2 = Arrays.stream(arrayOfArrays)
            .flatMap(Arrays::stream)
            .collect(Collectors.toList());
        
        System.out.println("List of Lists: " + listOfLists);
        System.out.println("Flattened: " + flattened);
    }
}
```
**Output:**
```
List of Lists: [[1, 2, 3], [4, 5], [6, 7, 8, 9]]
Flattened: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
</details>

---

### 37. Group employees by department
```java
// Output: Map<String, List<Employee>>
```

<details>
<summary>Show Answer</summary>

```java
import java.util.*;
import java.util.stream.*;

class Employee {
    private String name;
    private String department;
    private double salary;
    
    public Employee(String name, String department, double salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return name + "(" + department + ", $" + salary + ")";
    }
}

public class GroupEmployees {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("John", "IT", 60000),
            new Employee("Jane", "HR", 55000),
            new Employee("Jack", "IT", 65000),
            new Employee("Jill", "Finance", 70000),
            new Employee("James", "HR", 58000)
        );
        
        // Group by department
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
        
        System.out.println("Grouped by Department:");
        grouped.forEach((dept, empList) -> 
            System.out.println(dept + ": " + empList));
        
        System.out.println("\nCount by Department: " + countByDept);
        System.out.println("\nAverage Salary by Department: " + avgSalaryByDept);
    }
}
```
**Output:**
```
Grouped by Department:
HR: [Jane(HR, $55000.0), James(HR, $58000.0)]
Finance: [Jill(Finance, $70000.0)]
IT: [John(IT, $60000.0), Jack(IT, $65000.0)]

Count by Department: {HR=2, Finance=1, IT=2}

Average Salary by Department: {HR=56500.0, Finance=70000.0, IT=62500.0}
```
</details>

---

### 38. Convert List to Map (id -> name)
```java
// Output: Map<Integer, String>
```

<details>
<summary>Show Answer</summary>

```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.*;

class Employee {
    private Integer id;
    private String name;
    private double salary;
    
    public Employee(Integer id, String name, double salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
    }
    
    public Integer getId() { return id; }
    public String getName() { return name; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return "Employee{id=" + id + ", name='" + name + "', salary=" + salary + "}";
    }
}

public class ListToMap {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee(1, "John", 60000),
            new Employee(2, "Jane", 55000),
            new Employee(3, "Jack", 65000),
            new Employee(4, "Jill", 70000)
        );
        
        // Map id to name
        Map<Integer, String> map = employees.stream()
            .collect(Collectors.toMap(
                Employee::getId,
                Employee::getName
            ));
        
        // Map id to entire object
        Map<Integer, Employee> empMap = employees.stream()
            .collect(Collectors.toMap(
                Employee::getId,
                Function.identity()
            ));
        
        System.out.println("ID to Name Map: " + map);
        System.out.println("\nID to Employee Map:");
        empMap.forEach((id, emp) -> System.out.println(id + " -> " + emp));
    }
}
```
**Output:**
```
ID to Name Map: {1=John, 2=Jane, 3=Jack, 4=Jill}

ID to Employee Map:
1 -> Employee{id=1, name='John', salary=60000.0}
2 -> Employee{id=2, name='Jane', salary=55000.0}
3 -> Employee{id=3, name='Jack', salary=65000.0}
4 -> Employee{id=4, name='Jill', salary=70000.0}
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
import java.util.*;
import java.util.stream.*;

public class SortMapByValue {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("A", 5);
        map.put("B", 2);
        map.put("C", 8);
        map.put("D", 1);
        
        // Ascending order
        Map<String, Integer> sorted = map.entrySet().stream()
            .sorted(Map.Entry.comparingByValue())
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue,
                (e1, e2) -> e1,
                LinkedHashMap::new
            ));
        
        // Descending order
        Map<String, Integer> sortedDesc = map.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue,
                (e1, e2) -> e1,
                LinkedHashMap::new
            ));
        
        System.out.println("Original Map: " + map);
        System.out.println("Sorted by Value (Asc): " + sorted);
        System.out.println("Sorted by Value (Desc): " + sortedDesc);
    }
}
```
**Output:**
```
Original Map: {A=5, B=2, C=8, D=1}
Sorted by Value (Asc): {D=1, B=2, A=5, C=8}
Sorted by Value (Desc): {C=8, A=5, B=2, D=1}
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
import java.util.*;
import java.util.stream.*;

public class SortMapByKey {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("C", 5);
        map.put("A", 2);
        map.put("B", 8);
        
        Map<String, Integer> sorted = map.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue,
                (e1, e2) -> e1,
                LinkedHashMap::new
            ));
        
        System.out.println("Original Map: " + map);
        System.out.println("Sorted by Key: " + sorted);
    }
}
```
**Output:**
```
Original Map: {A=2, B=8, C=5}
Sorted by Key: {A=2, B=8, C=5}
```
</details>

---

### 41. Find top N salaries
```java
// Find top 3 salaries
```

<details>
<summary>Show Answer</summary>

```java
import java.util.*;
import java.util.stream.*;

class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return name + "($" + salary + ")";
    }
}

public class TopNSalaries {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("John", 60000),
            new Employee("Jane", 55000),
            new Employee("Jack", 80000),
            new Employee("Jill", 70000),
            new Employee("James", 65000)
        );
        
        // Top 3 salaries
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
        
        // Top 3 distinct salaries
        List<Double> topDistinctSalaries = employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .limit(3)
            .collect(Collectors.toList());
        
        System.out.println("Top 3 Salaries: " + topSalaries);
        System.out.println("Top 3 Employees: " + topEmployees);
        System.out.println("Top 3 Distinct Salaries: " + topDistinctSalaries);
    }
}
```
**Output:**
```
Top 3 Salaries: [80000.0, 70000.0, 65000.0]
Top 3 Employees: [Jack($80000.0), Jill($70000.0), James($65000.0)]
Top 3 Distinct Salaries: [80000.0, 70000.0, 65000.0]
```
</details>

---

### 42. Find Nth highest salary
```java
// Find 3rd highest salary
```

<details>
<summary>Show Answer</summary>

```java
import java.util.*;
import java.util.stream.*;

class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return name + "($" + salary + ")";
    }
}

public class NthHighestSalary {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("John", 60000),
            new Employee("Jane", 55000),
            new Employee("Jack", 80000),
            new Employee("Jill", 70000),
            new Employee("James", 65000)
        );
        
        int n = 3;
        
        // Nth highest salary
        Double nthSalary = employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(n - 1)
            .findFirst()
            .orElse(null);
        
        // Employee with nth highest salary
        Employee nthEmployee = employees.stream()
            .sorted(Comparator.comparing(Employee::getSalary).reversed())
            .skip(n - 1)
            .findFirst()
            .orElse(null);
        
        System.out.println(n + "rd Highest Salary: " + nthSalary);
        System.out.println("Employee with " + n + "rd Highest Salary: " + nthEmployee);
    }
}
```
**Output:**
```
3rd Highest Salary: 60000.0
Employee with 3rd Highest Salary: James($65000.0)
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
import java.util.*;
import java.util.stream.*;

class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public String getName() { return name; }
    public double getSalary() { return salary; }
    
    @Override
    public String toString() {
        return name + "($" + salary + ")";
    }
}

public class SalaryAboveAverage {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("John", 60000),
            new Employee("Jane", 55000),
            new Employee("Jack", 80000),
            new Employee("Jill", 70000),
            new Employee("James", 50000)
        );
        
        double avgSalary = employees.stream()
            .mapToDouble(Employee::getSalary)
            .average()
            .orElse(0.0);
        
        List<Employee> aboveAverage = employees.stream()
            .filter(e -> e.getSalary() > avgSalary)
            .collect(Collectors.toList());
        
        System.out.println("Average Salary: $" + avgSalary);
        System.out.println("Employees Above Average: " + aboveAverage);
    }
}
```
**Output:**
```
Average Salary: $63000.0
Employees Above Average: [Jack($80000.0), Jill($70000.0)]
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
import java.util.*;
import java.util.stream.*;

public class GroupByLength {
    public static void main(String[] args) {
        List<String> words = List.of("a", "bb", "ccc", "dd", "eee", "f");
        
        Map<Integer, List<String>> grouped = words.stream()
            .collect(Collectors.groupingBy(String::length));
        
        // Get count per length
        Map<Integer, Long> countByLength = words.stream()
            .collect(Collectors.groupingBy(
                String::length,
                Collectors.counting()
            ));
        
        System.out.println("Words: " + words);
        System.out.println("Grouped by Length: " + grouped);
        System.out.println("Count by Length: " + countByLength);
    }
}
```
**Output:**
```
Words: [a, bb, ccc, dd, eee, f]
Grouped by Length: {1=[a, f], 2=[bb, dd], 3=[ccc, eee]}
Count by Length: {1=2, 2=2, 3=2}
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
import java.util.*;
import java.util.stream.*;

public class LongestString {
    public static void main(String[] args) {
        List<String> words = List.of("Java", "Spring", "Microservices", "Boot");
        
        // Method 1: Using max
        String longest = words.stream()
            .max(Comparator.comparing(String::length))
            .orElse(null);
        
        // Method 2: Using reduce
        String longest2 = words.stream()
            .reduce((w1, w2) -> w1.length() > w2.length() ? w1 : w2)
            .orElse(null);
        
        System.out.println("Words: " + words);
        System.out.println("Longest Word: " + longest);
        System.out.println("Length: " + (longest != null ? longest.length() : 0));
    }
}
```
**Output:**
```
Words: [Java, Spring, Microservices, Boot]
Longest Word: Microservices
Length: 13
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
import java.util.*;
import java.util.stream.*;

public class SeparateUpperLower {
    public static void main(String[] args) {
        String input = "HelloWorld";
        
        // Method 1: Using partition
        Map<Boolean, String> partitioned = input.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.partitioningBy(
                Character::isUpperCase,
                Collectors.mapping(
                    String::valueOf,
                    Collectors.joining()
                )
            ));
        
        String uppercase = partitioned.get(true);
        String lowercase = partitioned.get(false);
        
        // Method 2: Separate filters
        String upper = input.chars()
            .filter(Character::isUpperCase)
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
        
        String lower = input.chars()
            .filter(Character::isLowerCase)
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
        
        System.out.println("Input: " + input);
        System.out.println("Uppercase: " + uppercase);
        System.out.println("Lowercase: " + lowercase);
    }
}
```
**Output:**
```
Input: HelloWorld
Uppercase: HW
Lowercase: elloorld
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
import java.util.*;
import java.util.stream.*;

public class SumOfDigits {
    public static void main(String[] args) {
        int number = 12345;
        
        // Method 1: Using Character.getNumericValue
        int sum = String.valueOf(number)
            .chars()
            .map(Character::getNumericValue)
            .sum();
        
        // Method 2: Using subtraction
        int sum2 = String.valueOf(number)
            .chars()
            .map(c -> c - '0')
            .sum();
        
        System.out.println("Number: " + number);
        System.out.println("Sum of Digits: " + sum);
    }
}
```
**Output:**
```
Number: 12345
Sum of Digits: 15
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
import java.util.*;
import java.util.stream.*;

public class CheckPalindrome {
    public static void main(String[] args) {
        String input1 = "madam";
        String input2 = "hello";
        
        // Method 1: Compare with reversed
        boolean isPalindrome1 = input1.equals(
            new StringBuilder(input1).reverse().toString()
        );
        
        // Method 2: Using IntStream
        boolean isPalindrome2 = IntStream.range(0, input1.length() / 2)
            .allMatch(i -> input1.charAt(i) == 
                      input1.charAt(input1.length() - 1 - i));
        
        boolean isPalindrome3 = IntStream.range(0, input2.length() / 2)
            .allMatch(i -> input2.charAt(i) == 
                      input2.charAt(input2.length() - 1 - i));
        
        System.out.println(input1 + " is Palindrome: " + isPalindrome1);
        System.out.println(input2 + " is Palindrome: " + isPalindrome3);
    }
}
```
**Output:**
```
madam is Palindrome: true
hello is Palindrome: false
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
import java.util.*;
import java.util.stream.*;

public class IntersectionOfLists {
    public static void main(String[] args) {
        List<Integer> list1 = List.of(1, 2, 3, 4, 5);
        List<Integer> list2 = List.of(3, 4, 5, 6, 7);
        List<Integer> list3 = List.of(4, 5, 6, 7, 8);
        
        // Method 1: Chain filters
        List<Integer> intersection = list1.stream()
            .filter(list2::contains)
            .filter(list3::contains)
            .collect(Collectors.toList());
        
        // Method 2: For variable number of lists
        List<List<Integer>> allLists = List.of(list1, list2, list3);
        List<Integer> result = allLists.get(0).stream()
            .filter(element -> allLists.stream()
                .allMatch(list -> list.contains(element)))
            .collect(Collectors.toList());
        
        System.out.println("List 1: " + list1);
        System.out.println("List 2: " + list2);
        System.out.println("List 3: " + list3);
        System.out.println("Intersection: " + intersection);
    }
}
```
**Output:**
```
List 1: [1, 2, 3, 4, 5]
List 2: [3, 4, 5, 6, 7]
List 3: [4, 5, 6, 7, 8]
Intersection: [4, 5]
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
import java.util.*;
import java.util.stream.*;

public class UnionOfLists {
    public static void main(String[] args) {
        List<Integer> list1 = List.of(1, 2, 3, 4);
        List<Integer> list2 = List.of(3, 4, 5, 6);
        
        // Unsorted union
        List<Integer> union = Stream.concat(list1.stream(), list2.stream())
            .distinct()
            .collect(Collectors.toList());
        
        // Sorted union
        List<Integer> sortedUnion = Stream.concat(list1.stream(), list2.stream())
            .distinct()
            .sorted()
            .collect(Collectors.toList());
        
        System.out.println("List 1: " + list1);
        System.out.println("List 2: " + list2);
        System.out.println("Union: " + union);
        System.out.println("Sorted Union: " + sortedUnion);
    }
}
```
**Output:**
```
List 1: [1, 2, 3, 4]
List 2: [3, 4, 5, 6]
Union: [1, 2, 3, 4, 5, 6]
Sorted Union: [1, 2, 3, 4, 5, 6]
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
import java.util.*;
import java.util.stream.*;

public class Factorial {
    public static void main(String[] args) {
        int n = 5;
        
        // Using int
        int factorial = IntStream.rangeClosed(1, n)
            .reduce(1, (a, b) -> a * b);
        
        // Using long for larger numbers
        long factorial2 = LongStream.rangeClosed(1, n)
            .reduce(1, (a, b) -> a * b);
        
        // For larger factorial
        int n2 = 10;
        long factorial3 = LongStream.rangeClosed(1, n2)
            .reduce(1, (a, b) -> a * b);
        
        System.out.println(n + "! = " + factorial);
        System.out.println(n2 + "! = " + factorial3);
    }
}
```
**Output:**
```
5! = 120
10! = 3628800
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
import java.util.*;
import java.util.stream.*;

public class FibonacciSeries {
    public static void main(String[] args) {
        int n = 10;
        
        // Generate first n Fibonacci numbers
        List<Integer> fibonacci = Stream.iterate(new int[]{0, 1}, 
                f -> new int[]{f[1], f[0] + f[1]})
            .limit(n)
            .map(f -> f[0])
            .collect(Collectors.toList());
        
        // Alternative - starting from 1
        List<Integer> fibonacci2 = Stream.iterate(new int[]{0, 1}, 
                f -> new int[]{f[1], f[0] + f[1]})
            .limit(n)
            .map(f -> f[1])
            .collect(Collectors.toList());
        
        System.out.println("First " + n + " Fibonacci numbers:");
        System.out.println(fibonacci);
    }
}
```
**Output:**
```
First 10 Fibonacci numbers:
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
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
import java.util.*;
import java.util.stream.*;

public class RemoveSpecialCharacters {
    public static void main(String[] args) {
        String input = "Hello@World#123!";
        
        // Keep letters and digits
        String result = input.chars()
            .filter(Character::isLetterOrDigit)
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
        
        // Only letters
        String onlyLetters = input.chars()
            .filter(Character::isLetter)
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
        
        // Only digits
        String onlyDigits = input.chars()
            .filter(Character::isDigit)
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());
        
        System.out.println("Input: " + input);
        System.out.println("Letters and Digits: " + result);
        System.out.println("Only Letters: " + onlyLetters);
        System.out.println("Only Digits: " + onlyDigits);
    }
}
```
**Output:**
```
Input: Hello@World#123!
Letters and Digits: HelloWorld123
Only Letters: HelloWorld
Only Digits: 123
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
import java.util.*;
import java.util.stream.*;

public class CheckDuplicates {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 2, 4);
        List<Integer> uniqueNumbers = List.of(1, 2, 3, 4, 5);
        
        // Method 1: Compare size with distinct count
        boolean hasDuplicates = numbers.size() != 
            numbers.stream().distinct().count();
        
        // Method 2: Using Set
        Set<Integer> set = new HashSet<>();
        boolean hasDuplicates2 = numbers.stream()
            .anyMatch(n -> !set.add(n));
        
        boolean hasUnique = uniqueNumbers.size() != 
            uniqueNumbers.stream().distinct().count();
        
        System.out.println("List 1: " + numbers);
        System.out.println("Has Duplicates: " + hasDuplicates);
        System.out.println("\nList 2: " + uniqueNumbers);
        System.out.println("Has Duplicates: " + hasUnique);
    }
}
```
**Output:**
```
List 1: [1, 2, 3, 2, 4]
Has Duplicates: true

List 2: [1, 2, 3, 4, 5]
Has Duplicates: false
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
import java.util.*;
import java.util.stream.*;

public class MergeMaps {
    public static void main(String[] args) {
        Map<String, Integer> map1 = new HashMap<>();
        map1.put("A", 1);
        map1.put("B", 2);
        
        Map<String, Integer> map2 = new HashMap<>();
        map2.put("B", 3);
        map2.put("C", 4);
        
        // Merge with value addition
        Map<String, Integer> merged = Stream.concat(
                map1.entrySet().stream(),
                map2.entrySet().stream()
            )
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue,
                Integer::sum
            ));
        
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
        
        System.out.println("Map 1: " + map1);
        System.out.println("Map 2: " + map2);
        System.out.println("Merged (sum): " + merged);
        System.out.println("Merged (first): " + merged2);
    }
}
```
**Output:**
```
Map 1: {A=1, B=2}
Map 2: {B=3, C=4}
Merged (sum): {A=1, B=5, C=4}
Merged (first): {A=1, B=2, C=4}
```
</details>

---
