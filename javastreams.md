# 50 Java Stream Scenario-Based Programs with Explanations

Below is a beautified version of the provided content. Each scenario is presented with:
- **Title**: A clear, bolded description of the problem.
- **Code Snippet**: The Java Stream-based solution, formatted as a code block for readability.
- **Explanation**: A concise breakdown of the approach, key Stream operations, and real-world applications.

---

### 1. Find the k most frequent elements in a list
```java
int k = 3;
List<Integer> topK = list.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
    .entrySet().stream()
    .sorted(Map.Entry.comparingByValue().reversed())
    .limit(k)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find the k most frequent elements in a list. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 2. Implement a custom collector to get summary statistics of string lengths
```java
Collector<String, ?, IntSummaryStatistics> summaryCollector = Collectors.summarizingInt(String::length);
IntSummaryStatistics stats = list.stream().collect(summaryCollector);
```
**Explanation**: This problem demonstrates how to use Java Streams to implement a custom collector to get summary statistics of string lengths. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 3. Find all distinct permutations of characters in each string
```java
List<String> permutations = list.stream()
    .flatMap(s -> permute(s).stream())
    .distinct()
    .collect(Collectors.toList());
// Assume permute is a custom method returning all permutations of a string
```
**Explanation**: This problem demonstrates how to use Java Streams to find all distinct permutations of characters in each string. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 4. Batch process a large stream in fixed-size chunks (e.g., 100 items each)
```java
AtomicInteger counter = new AtomicInteger();
Map<Integer, List<Object>> batches = list.stream()
    .collect(Collectors.groupingBy(i -> counter.getAndIncrement() / 100));
```
**Explanation**: This problem demonstrates how to use Java Streams to batch process a large stream in fixed-size chunks (e.g., 100 items each). The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 5. Create a time-windowed stream from timestamped events
```java
Map<Long, List<Event>> windows = events.stream()
    .collect(Collectors.groupingBy(e -> e.getTimestamp() / 60000)); // Group by minute
```
**Explanation**: This problem demonstrates how to use Java Streams to create a time-windowed stream from timestamped events. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 6. Find the longest increasing subsequence using Stream (complex logic)
```java
List<Integer> lis = new ArrayList<>();
for (Integer num : list) {
    int i = Collections.binarySearch(lis, num);
    if (i < 0) i = -(i + 1);
    if (i == lis.size()) lis.add(num);
    else lis.set(i, num);
}
```
**Explanation**: This problem demonstrates how to use Java Streams to find the longest increasing subsequence using stream (complex logic). The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 7. Group by multiple fields (e.g., department and designation)
```java
Map<List<String>, List<Employee>> grouped = employees.stream()
    .collect(Collectors.groupingBy(e -> Arrays.asList(e.getDepartment(), e.getDesignation())));
```
**Explanation**: This problem demonstrates how to use Java Streams to group by multiple fields (e.g., department and designation). The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 8. Flatten a hierarchical tree structure into a list using recursion and stream
```java
List<Node> flatten(Node node) {
    return Stream.concat(Stream.of(node), 
        node.getChildren().stream().flatMap(child -> flatten(child).stream()))
        .collect(Collectors.toList());
}
```
**Explanation**: This problem demonstrates how to use Java Streams to flatten a hierarchical tree structure into a list using recursion and stream. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 9. Parallel stream processing with thread-safe accumulation
```java
ConcurrentMap<Object, Long> result = list.parallelStream()
    .collect(Collectors.groupingByConcurrent(Function.identity(), Collectors.counting()));
```
**Explanation**: This problem demonstrates how to use Java Streams for parallel stream processing with thread-safe accumulation. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 10. Custom stream-based DSL for filtering dynamic criteria
```java
Predicate<Employee> isManager = e -> e.getRole().equals("Manager");
Predicate<Employee> earnsAbove50k = e -> e.getSalary() > 50000;
List<Employee> filtered = employees.stream()
    .filter(isManager.and(earnsAbove50k))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams for custom stream-based DSL for filtering dynamic criteria. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 11. Find common elements from two lists using streams
```java
List<Integer> common = list1.stream()
    .filter(list2::contains)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find common elements from two lists using streams. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 12. Convert a list of strings to a map with string lengths as values
```java
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(Function.identity(), String::length));
```
**Explanation**: This problem demonstrates how to use Java Streams to convert a list of strings to a map with string lengths as values. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 13. Calculate the product of all integers in a list
```java
int product = list.stream()
    .reduce(1, (a, b) -> a * b);
```
**Explanation**: This problem demonstrates how to use Java Streams to calculate the product of all integers in a list. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 14. Partition students based on pass/fail using score > 40
```java
Map<Boolean, List<Student>> result = students.stream()
    .collect(Collectors.partitioningBy(s -> s.getScore() > 40));
```
**Explanation**: This problem demonstrates how to use Java Streams to partition students based on pass/fail using score > 40. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 15. Sort a list of employees by department then salary descending
```java
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
        .thenComparing(Employee::getSalary, Comparator.reverseOrder()))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to sort a list of employees by department then salary descending. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 16. Get all manager names reporting to a particular department
```java
List<String> managers = employees.stream()
    .filter(e -> e.getDepartment().equals("Sales") && e.getRole().equals("Manager"))
    .map(Employee::getName)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to get all manager names reporting to a particular department. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 17. Group books by author and then by genre
```java
Map<String, Map<String, List<Book>>> grouped = books.stream()
    .collect(Collectors.groupingBy(Book::getAuthor, 
        Collectors.groupingBy(Book::getGenre)));
```
**Explanation**: This problem demonstrates how to use Java Streams to group books by author and then by genre. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 18. Count files by file extension
```java
Map<String, Long> countByExt = files.stream()
    .collect(Collectors.groupingBy(f -> getExtension(f.getName()), Collectors.counting()));
```
**Explanation**: This problem demonstrates how to use Java Streams to count files by file extension. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 19. Find customers with more than 3 purchases
```java
List<Customer> loyal = purchases.stream()
    .collect(Collectors.groupingBy(Purchase::getCustomer, Collectors.counting()))
    .entrySet().stream()
    .filter(e -> e.getValue() > 3)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find customers with more than 3 purchases. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 20. Convert a list of dates to strings in dd-MM-yyyy format
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
List<String> formatted = dates.stream()
    .map(d -> d.format(formatter))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to convert a list of dates to strings in dd-MM-yyyy format. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 21. Filter out non-prime numbers using streams
```java
List<Integer> primes = list.stream()
    .filter(n -> IntStream.rangeClosed(2, (int)Math.sqrt(n))
        .allMatch(i -> n % i != 0))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to filter out non-prime numbers using streams. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 22. Find the longest string in a list
```java
String longest = list.stream()
    .max(Comparator.comparingInt(String::length))
    .orElse("");
```
**Explanation**: This problem demonstrates how to use Java Streams to find the longest string in a list. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 23. Calculate moving average with window size 3
```java
List<Double> movingAvg = IntStream.range(0, list.size() - 2)
    .mapToDouble(i -> (list.get(i) + list.get(i+1) + list.get(i+2)) / 3.0)
    .boxed().collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to calculate moving average with window size 3. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 24. Find all palindromes in a list
```java
List<String> palindromes = list.stream()
    .filter(s -> s.equalsIgnoreCase(new StringBuilder(s).reverse().toString()))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find all palindromes in a list. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 25. Group employees by city and then by team size
```java
Map<String, Map<Integer, List<Employee>>> grouped = employees.stream()
    .collect(Collectors.groupingBy(Employee::getCity, 
        Collectors.groupingBy(e -> e.getTeam().size())));
```
**Explanation**: This problem demonstrates how to use Java Streams to group employees by city and then by team size. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 26. Normalize and deduplicate emails
```java
List<String> cleanEmails = emails.stream()
    .map(String::toLowerCase)
    .map(e -> e.replaceAll("\\s", ""))
    .distinct()
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to normalize and deduplicate emails. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 27. Get all students whose names start and end with vowels
```java
List<String> result = students.stream()
    .map(Student::getName)
    .filter(n -> n.matches("(?i)^[aeiou].*[aeiou]$"))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to get all students whose names start and end with vowels. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 28. Sum the values of a nested list structure
```java
int sum = nestedList.stream()
    .flatMap(Collection::stream)
    .mapToInt(Integer::intValue)
    .sum();
```
**Explanation**: This problem demonstrates how to use Java Streams to sum the values of a nested list structure. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 29. Get the earliest and latest transaction dates
```java
Optional<LocalDate> min = transactions.stream()
    .map(Transaction::getDate)
    .min(LocalDate::compareTo);
Optional<LocalDate> max = transactions.stream()
    .map(Transaction::getDate)
    .max(LocalDate::compareTo);
```
**Explanation**: This problem demonstrates how to use Java Streams to get the earliest and latest transaction dates. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 30. Convert list of objects to comma-separated values for a field
```java
String csv = objects.stream()
    .map(Object::toString)
    .collect(Collectors.joining(", "));
```
**Explanation**: This problem demonstrates how to use Java Streams to convert list of objects to comma-separated values for a field. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 31. Generate stream of all substrings of a string
```java
List<String> substrings = IntStream.range(0, str.length())
    .boxed()
    .flatMap(i -> IntStream.range(i + 1, str.length() + 1)
        .mapToObj(j -> str.substring(i, j)))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to generate stream of all substrings of a string. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 32. Create a frequency map of digit counts in a list of numbers
```java
Map<Integer, Long> digitCount = list.stream()
    .flatMap(n -> String.valueOf(n).chars().mapToObj(c -> c - '0'))
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```
**Explanation**: This problem demonstrates how to use Java Streams to create a frequency map of digit counts in a list of numbers. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 33. Partition strings by length: short (<5), medium (5-10), long (>10)
```java
Map<String, List<String>> partitioned = list.stream()
    .collect(Collectors.groupingBy(s -> s.length() < 5 ? "short" 
        : s.length() <= 10 ? "medium" : "long"));
```
**Explanation**: This problem demonstrates how to use Java Streams to partition strings by length: short (<5), medium (5-10), long (>10). The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 34. Find repeating characters in a string
```java
Map<Character, Long> freq = str.chars()
    .mapToObj(c -> (char)c)
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
List<Character> repeating = freq.entrySet().stream()
    .filter(e -> e.getValue() > 1)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find repeating characters in a string. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 35. Zip two lists into a Map using index
```java
Map<Object, Object> zipped = IntStream.range(0, list1.size())
    .boxed()
    .collect(Collectors.toMap(list1::get, list2::get));
```
**Explanation**: This problem demonstrates how to use Java Streams to zip two lists into a map using index. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 36. Find the first recurring element in a list
```java
Set<Object> seen = new HashSet<>();
Optional<Object> firstRecurring = list.stream()
    .filter(n -> !seen.add(n))
    .findFirst();
```
**Explanation**: This problem demonstrates how to use Java Streams to find the first recurring element in a list. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 37. Extract domain names from a list of emails
```java
List<String> domains = emails.stream()
    .map(email -> email.substring(email.indexOf("@") + 1))
    .distinct()
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to extract domain names from a list of emails. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 38. Find longest common prefix of a list of strings
```java
String prefix = list.stream()
    .reduce((a, b) -> {
        int i = 0;
        while (i < a.length() && i < b.length() && a.charAt(i) == b.charAt(i)) i++;
        return a.substring(0, i);
    }).orElse("");
```
**Explanation**: This problem demonstrates how to use Java Streams to find longest common prefix of a list of strings. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 39. Calculate the variance of a list of integers
```java
double avg = list.stream().mapToDouble(i -> i).average().orElse(0);
double variance = list.stream()
    .mapToDouble(i -> Math.pow(i - avg, 2))
    .average().orElse(0);
```
**Explanation**: This problem demonstrates how to use Java Streams to calculate the variance of a list of integers. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 40. Find the top 3 longest words in a paragraph
```java
List<String> top3Words = Arrays.stream(paragraph.split("\\s+"))
    .sorted(Comparator.comparingInt(String::length).reversed())
    .limit(3)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to find the top 3 longest words in a paragraph. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 41. Detect circular references in a list of parent-child relationships
```java
Set<String> visited = new HashSet<>();
boolean hasCycle = relations.stream()
    .anyMatch(r -> !visited.add(r.getParent() + "->" + r.getChild()));
```
**Explanation**: This problem demonstrates how to use Java Streams to detect circular references in a list of parent-child relationships. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 42. Calculate median of a list using streams
```java
List<Integer> sorted = list.stream().sorted().collect(Collectors.toList());
double median = list.size() % 2 == 0 
    ? (sorted.get(list.size()/2 - 1) + sorted.get(list.size()/2)) / 2.0 
    : sorted.get(list.size()/2);
```
**Explanation**: This problem demonstrates how to use Java Streams to calculate median of a list using streams. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 43. Create a map of initials to concatenated names
```java
Map<Character, String> map = names.stream()
    .collect(Collectors.groupingBy(n -> n.charAt(0), 
        Collectors.mapping(Function.identity(), Collectors.joining(", "))));
```
**Explanation**: This problem demonstrates how to use Java Streams to create a map of initials to concatenated names. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 44. Find the second most frequent element
```java
String secondMost = list.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
    .entrySet().stream()
    .sorted(Map.Entry.comparingByValue().reversed())
    .skip(1).findFirst().map(Map.Entry::getKey).orElse(null);
```
**Explanation**: This problem demonstrates how to use Java Streams to find the second most frequent element. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 45. Identify anagrams in a list of strings
```java
Map<String, List<String>> anagrams = list.stream()
    .collect(Collectors.groupingBy(s -> s.chars().sorted()
        .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append).toString()));
```
**Explanation**: This problem demonstrates how to use Java Streams to identify anagrams in a list of strings. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 46. Create a suffix tree-like structure from list of strings
```java
Map<String, List<String>> suffixMap = list.stream()
    .flatMap(word -> IntStream.range(0, word.length())
        .mapToObj(i -> new AbstractMap.SimpleEntry<>(word.substring(i), word)))
    .collect(Collectors.groupingBy(Map.Entry::getKey, 
        Collectors.mapping(Map.Entry::getValue, Collectors.toList())));
```
**Explanation**: This problem demonstrates how to use Java Streams to create a suffix tree-like structure from list of strings. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 47. Get all possible pairs of two lists using flatMap
```java
List<String> pairs = list1.stream()
    .flatMap(a -> list2.stream().map(b -> a + "-" + b))
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to get all possible pairs of two lists using flatMap. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 48. Group employees by age range buckets (20-29, 30-39...)
```java
Map<String, List<Employee>> grouped = employees.stream()
    .collect(Collectors.groupingBy(e -> { 
        int age = e.getAge(); 
        return (age / 10) * 10 + "-" + ((age / 10) * 10 + 9); 
    }));
```
**Explanation**: This problem demonstrates how to use Java Streams to group employees by age range buckets (20-29, 30-39...). The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 49. Extract hashtags from a list of tweets
```java
List<String> hashtags = tweets.stream()
    .flatMap(tweet -> Arrays.stream(tweet.split(" ")))
    .filter(word -> word.startsWith("#"))
    .distinct()
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to extract hashtags from a list of tweets. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.

### 50. Remove consecutive duplicates from a list using streams
```java
List<Object> result = IntStream.range(0, list.size())
    .filter(i -> i == 0 || !list.get(i).equals(list.get(i - 1)))
    .mapToObj(list::get)
    .collect(Collectors.toList());
```
**Explanation**: This problem demonstrates how to use Java Streams to remove consecutive duplicates from a list using streams. The key Stream operations involved include mapping, filtering, grouping, reducing, or flatMapping based on the use case. This is useful in real-world applications like analytics, processing, or building backend logic for services.
