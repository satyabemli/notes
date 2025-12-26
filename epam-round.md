# Deep Preparation Guide: 60-min Technical Interview

## PART 1: PROBLEM SOLVING (30 mins)

---

## 📊 TIME & SPACE COMPLEXITY - MUST KNOW

```
O(1)       - Constant (Array access, HashMap get/put)
O(log n)   - Logarithmic (Binary Search)
O(n)       - Linear (Single loop, Linear Search)
O(n log n) - Linearithmic (Merge Sort, Quick Sort avg)
O(n²)      - Quadratic (Nested loops, Bubble Sort)
O(2^n)     - Exponential (Recursive Fibonacci)
```

### Quick Complexity Cheat Sheet:
```
HashMap: get/put O(1), worst O(n)
ArrayList: add O(1), get O(1), remove O(n)
LinkedList: add O(1), get O(n), remove O(1) if node known
TreeMap: get/put O(log n)
```

---

## 🔢 ARRAYS

### Pattern 1: Two Pointer Technique

```java
// Problem: Two Sum - Find two numbers that add to target
// Brute Force: O(n²)
public int[] twoSumBrute(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[]{i, j};
            }
        }
    }
    return new int[]{};
}

// Optimized: O(n) using HashMap
public int[] twoSumOptimized(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

### Pattern 2: Sliding Window

```java
// Problem: Maximum sum subarray of size k
// Brute Force: O(n*k)
public int maxSumBrute(int[] arr, int k) {
    int maxSum = Integer.MIN_VALUE;
    for (int i = 0; i <= arr.length - k; i++) {
        int sum = 0;
        for (int j = i; j < i + k; j++) {
            sum += arr[j];
        }
        maxSum = Math.max(maxSum, sum);
    }
    return maxSum;
}

// Optimized: O(n) Sliding Window
public int maxSumOptimized(int[] arr, int k) {
    int windowSum = 0;
    
    // First window
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    
    int maxSum = windowSum;
    
    // Slide the window
    for (int i = k; i < arr.length; i++) {
        windowSum = windowSum + arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### Pattern 3: Kadane's Algorithm (Maximum Subarray)

```java
public int maxSubArray(int[] nums) {
    int maxSoFar = nums[0];
    int maxEndingHere = nums[0];
    
    for (int i = 1; i < nums.length; i++) {
        maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
```

### Common Array Problems:

```java
// 1. Find duplicate in array
public int findDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) return num;
    }
    return -1;
}

// 2. Move zeros to end
public void moveZeroes(int[] nums) {
    int insertPos = 0;
    for (int num : nums) {
        if (num != 0) {
            nums[insertPos++] = num;
        }
    }
    while (insertPos < nums.length) {
        nums[insertPos++] = 0;
    }
}

// 3. Rotate array by k positions
public void rotate(int[] nums, int k) {
    k = k % nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}

private void reverse(int[] nums, int start, int end) {
    while (start < end) {
        int temp = nums[start];
        nums[start] = nums[end];
        nums[end] = temp;
        start++;
        end--;
    }
}

// 4. Find missing number (1 to n)
public int missingNumber(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) {
        actualSum += num;
    }
    return expectedSum - actualSum;
}

// 5. Remove duplicates from sorted array (in-place)
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}
```

---

## 🔗 LINKED LIST

### Basic Structure:

```java
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}
```

### Essential Operations:

```java
// 1. Reverse a Linked List - O(n)
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    
    while (current != null) {
        ListNode nextTemp = current.next;
        current.next = prev;
        prev = current;
        current = nextTemp;
    }
    return prev;
}

// 2. Detect Cycle (Floyd's Algorithm) - O(n)
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) return false;
    
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}

// 3. Find Middle Element - O(n)
public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

// 4. Merge Two Sorted Lists - O(n+m)
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    current.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}

// 5. Remove Nth Node from End
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy;
    ListNode second = dummy;
    
    // Move first n+1 steps ahead
    for (int i = 0; i <= n; i++) {
        first = first.next;
    }
    
    // Move both until first reaches end
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    
    second.next = second.next.next;
    return dummy.next;
}

// 6. Check Palindrome
public boolean isPalindrome(ListNode head) {
    // Find middle
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    // Reverse second half
    ListNode secondHalf = reverseList(slow);
    
    // Compare
    ListNode firstHalf = head;
    while (secondHalf != null) {
        if (firstHalf.val != secondHalf.val) return false;
        firstHalf = firstHalf.next;
        secondHalf = secondHalf.next;
    }
    return true;
}
```

---

## 📚 STACK & QUEUE

### Stack Implementation & Problems:

```java
// Using Deque (preferred)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);      // Add to top
stack.pop();        // Remove from top
stack.peek();       // View top
stack.isEmpty();    // Check empty

// 1. Valid Parentheses
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', '}', '{', ']', '[');
    
    for (char c : s.toCharArray()) {
        if (map.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != map.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    return stack.isEmpty();
}

// 2. Min Stack - O(1) for all operations
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    
    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }
    
    public void pop() {
        if (stack.pop().equals(minStack.peek())) {
            minStack.pop();
        }
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}

// 3. Next Greater Element
public int[] nextGreaterElement(int[] nums) {
    int[] result = new int[nums.length];
    Deque<Integer> stack = new ArrayDeque<>();
    
    for (int i = nums.length - 1; i >= 0; i--) {
        while (!stack.isEmpty() && stack.peek() <= nums[i]) {
            stack.pop();
        }
        result[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(nums[i]);
    }
    return result;
}

// 4. Evaluate Reverse Polish Notation
public int evalRPN(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();
    
    for (String token : tokens) {
        if ("+-*/".contains(token)) {
            int b = stack.pop();
            int a = stack.pop();
            switch (token) {
                case "+": stack.push(a + b); break;
                case "-": stack.push(a - b); break;
                case "*": stack.push(a * b); break;
                case "/": stack.push(a / b); break;
            }
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    return stack.pop();
}
```

### Queue Implementation & Problems:

```java
// Using LinkedList or ArrayDeque
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);     // Add to end
queue.poll();       // Remove from front
queue.peek();       // View front

// 1. Implement Queue using Stacks
class MyQueue {
    private Deque<Integer> input = new ArrayDeque<>();
    private Deque<Integer> output = new ArrayDeque<>();
    
    public void push(int x) {
        input.push(x);
    }
    
    public int pop() {
        peek();
        return output.pop();
    }
    
    public int peek() {
        if (output.isEmpty()) {
            while (!input.isEmpty()) {
                output.push(input.pop());
            }
        }
        return output.peek();
    }
    
    public boolean empty() {
        return input.isEmpty() && output.isEmpty();
    }
}

// 2. Implement Stack using Queues
class MyStack {
    private Queue<Integer> queue = new LinkedList<>();
    
    public void push(int x) {
        queue.offer(x);
        int size = queue.size();
        while (size > 1) {
            queue.offer(queue.poll());
            size--;
        }
    }
    
    public int pop() {
        return queue.poll();
    }
    
    public int top() {
        return queue.peek();
    }
    
    public boolean empty() {
        return queue.isEmpty();
    }
}
```

---

## 🔤 STRING MANIPULATION

```java
// 1. Reverse String
public String reverseString(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}

// 2. Check Palindrome
public boolean isPalindrome(String s) {
    s = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}

// 3. Check Anagram - O(n)
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    
    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}

// 4. First Non-Repeating Character - O(n)
public int firstUniqChar(String s) {
    int[] count = new int[26];
    
    for (char c : s.toCharArray()) {
        count[c - 'a']++;
    }
    
    for (int i = 0; i < s.length(); i++) {
        if (count[s.charAt(i) - 'a'] == 1) {
            return i;
        }
    }
    return -1;
}

// 5. Longest Substring Without Repeating Characters - O(n)
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0;
    int left = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c)) {
            left = Math.max(left, map.get(c) + 1);
        }
        map.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}

// 6. Group Anagrams
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}

// 7. String Compression
public String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int count = 1;
    
    for (int i = 1; i <= s.length(); i++) {
        if (i < s.length() && s.charAt(i) == s.charAt(i - 1)) {
            count++;
        } else {
            sb.append(s.charAt(i - 1));
            if (count > 1) sb.append(count);
            count = 1;
        }
    }
    return sb.length() < s.length() ? sb.toString() : s;
}
```

---

## 🔍 BINARY SEARCH

```java
// Basic Binary Search - O(log n)
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoid overflow
        
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}

// Find First Occurrence
public int findFirst(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            right = mid - 1;  // Keep searching left
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return result;
}

// Find Last Occurrence
public int findLast(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            result = mid;
            left = mid + 1;  // Keep searching right
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return result;
}

// Search in Rotated Sorted Array
public int searchRotated(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) return mid;
        
        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        // Right half is sorted
        else {
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}

// Find Peak Element
public int findPeakElement(int[] nums) {
    int left = 0, right = nums.length - 1;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[mid + 1]) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}

// Square Root using Binary Search
public int mySqrt(int x) {
    if (x < 2) return x;
    
    long left = 1, right = x / 2;
    
    while (left <= right) {
        long mid = left + (right - left) / 2;
        long square = mid * mid;
        
        if (square == x) return (int) mid;
        else if (square < x) left = mid + 1;
        else right = mid - 1;
    }
    return (int) right;
}
```

---

## 📊 SORTING ALGORITHMS

```java
// 1. Bubble Sort - O(n²)
public void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;  // Optimization
    }
}

// 2. Selection Sort - O(n²)
public void selectionSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        int temp = arr[minIdx];
        arr[minIdx] = arr[i];
        arr[i] = temp;
    }
}

// 3. Insertion Sort - O(n²), best O(n)
public void insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

// 4. Merge Sort - O(n log n)
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] leftArr = Arrays.copyOfRange(arr, left, mid + 1);
    int[] rightArr = Arrays.copyOfRange(arr, mid + 1, right + 1);
    
    int i = 0, j = 0, k = left;
    while (i < leftArr.length && j < rightArr.length) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];
        } else {
            arr[k++] = rightArr[j++];
        }
    }
    while (i < leftArr.length) arr[k++] = leftArr[i++];
    while (j < rightArr.length) arr[k++] = rightArr[j++];
}

// 5. Quick Sort - O(n log n) avg, O(n²) worst
public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

private int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
}
```

### Sorting Comparison:
| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No |

---

# PART 2: CORE JAVA (30 mins)

---

## 🔷 JAVA FUNDAMENTALS

### OOP Concepts

```java
// 1. ENCAPSULATION - Hiding internal state
public class Employee {
    private String name;
    private double salary;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public double getSalary() { return salary; }
    public void setSalary(double salary) {
        if (salary > 0) this.salary = salary;
    }
}

// 2. INHERITANCE - IS-A relationship
public class Animal {
    protected String name;
    public void eat() { System.out.println("Eating..."); }
}

public class Dog extends Animal {
    @Override
    public void eat() { System.out.println("Dog eating..."); }
    public void bark() { System.out.println("Barking..."); }
}

// 3. POLYMORPHISM - Many forms
// Compile-time (Overloading)
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

// Runtime (Overriding)
Animal animal = new Dog();  // Upcasting
animal.eat();  // Calls Dog's eat() - Runtime polymorphism

// 4. ABSTRACTION
public abstract class Shape {
    abstract double area();
    abstract double perimeter();
    
    public void display() {  // Concrete method
        System.out.println("Area: " + area());
    }
}

public interface Drawable {
    void draw();  // implicitly public abstract
    default void print() { System.out.println("Printing..."); }
    static void info() { System.out.println("Drawable interface"); }
}
```

### Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract + Concrete | Abstract + Default + Static (Java 8+) |
| Variables | Any type | public static final only |
| Constructor | Yes | No |
| Inheritance | Single | Multiple |
| Access Modifiers | Any | public only |

### String, StringBuilder, StringBuffer

```java
// String - Immutable
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

s1 == s2;           // true (String pool)
s1 == s3;           // false (different objects)
s1.equals(s3);      // true (content comparison)

// StringBuilder - Mutable, NOT thread-safe, faster
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
sb.insert(5, ",");
sb.reverse();
sb.delete(0, 2);

// StringBuffer - Mutable, Thread-safe, slower
StringBuffer sbf = new StringBuffer("Hello");
// Same methods as StringBuilder
```

### Exception Handling

```java
// Hierarchy: Throwable -> Exception, Error
//           Exception -> RuntimeException (Unchecked), Other Exceptions (Checked)

public class ExceptionDemo {
    
    // Checked Exception - Must be declared or caught
    public void readFile() throws IOException {
        FileReader fr = new FileReader("file.txt");
    }
    
    // Unchecked Exception - RuntimeException
    public void divide(int a, int b) {
        if (b == 0) throw new ArithmeticException("Cannot divide by zero");
        System.out.println(a / b);
    }
    
    // Try-with-resources (Java 7+)
    public void readWithResources() {
        try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
            String line = br.readLine();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    // Custom Exception
    public class CustomException extends Exception {
        public CustomException(String message) {
            super(message);
        }
    }
}

// finally vs return
public int testFinally() {
    try {
        return 1;
    } finally {
        System.out.println("Finally executes");  // This runs!
        // return 2;  // If uncommented, returns 2 (not recommended)
    }
}
```

### Collections Framework

```java
// Collection Hierarchy
// Collection -> List, Set, Queue
// Map (separate hierarchy)

// LIST - Ordered, allows duplicates
List<String> arrayList = new ArrayList<>();    // O(1) access, O(n) insert
List<String> linkedList = new LinkedList<>();  // O(n) access, O(1) insert

// SET - No duplicates
Set<String> hashSet = new HashSet<>();         // O(1), no order
Set<String> linkedHashSet = new LinkedHashSet<>();  // O(1), insertion order
Set<String> treeSet = new TreeSet<>();         // O(log n), sorted

// MAP - Key-Value pairs
Map<String, Integer> hashMap = new HashMap<>();        // O(1), no order
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();  // O(1), insertion order
Map<String, Integer> treeMap = new TreeMap<>();        // O(log n), sorted

// QUEUE
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);  // Add
queue.poll();    // Remove and return
queue.peek();    // View front

// DEQUE - Double-ended queue
Deque<Integer> deque = new ArrayDeque<>();
deque.addFirst(1);
deque.addLast(2);
deque.removeFirst();
deque.removeLast();

// PriorityQueue - Heap
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

### HashMap Internals

```java
// How HashMap works:
// 1. hashCode() of key -> bucket index
// 2. If collision -> LinkedList (Java 7) or Red-Black Tree (Java 8, when > 8 nodes)
// 3. equals() to find exact match

// Custom Key in HashMap
public class Person {
    private String name;
    private int age;
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
}

// Why both hashCode and equals?
// hashCode -> finds bucket
// equals -> finds exact object in bucket
```

### Multithreading Basics

```java
// Creating Threads
// 1. Extending Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

// 2. Implementing Runnable (preferred)
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable running");
    }
}

// 3. Lambda (Java 8)
Thread t = new Thread(() -> System.out.println("Lambda thread"));

// Thread States: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED

// Synchronization
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    // Or using synchronized block
    public void incrementBlock() {
        synchronized(this) {
            count++;
        }
    }
}

// wait(), notify(), notifyAll()
public class ProducerConsumer {
    private List<Integer> buffer = new ArrayList<>();
    private int capacity = 5;
    
    public synchronized void produce(int value) throws InterruptedException {
        while (buffer.size() == capacity) {
            wait();
        }
        buffer.add(value);
        notify();
    }
    
    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait();
        }
        int value = buffer.remove(0);
        notify();
        return value;
    }
}
```

### Comparator vs Comparable

```java
// Comparable - Natural ordering (in the class itself)
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    
    @Override
    public int compareTo(Employee other) {
        return this.salary - other.salary;  // Ascending by salary
    }
}

// Comparator - Custom ordering (external)
Comparator<Employee> byName = (e1, e2) -> e1.getName().compareTo(e2.getName());
Comparator<Employee> bySalary = Comparator.comparingInt(Employee::getSalary);
Comparator<Employee> bySalaryDesc = Comparator.comparingInt(Employee::getSalary).reversed();

// Chained Comparators
Comparator<Employee> byNameThenSalary = Comparator
    .comparing(Employee::getName)
    .thenComparingInt(Employee::getSalary);

Collections.sort(employees, byName);
employees.sort(bySalary);
```

---

## ☕ JAVA 8 FEATURES

### 1. Functional Interfaces

```java
// Interface with exactly ONE abstract method
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Can have default and static methods
    default void print() { System.out.println("Calculator"); }
    static void info() { System.out.println("Info"); }
}

// Built-in Functional Interfaces
// 1. Predicate<T> - T -> boolean
Predicate<Integer> isEven = n -> n % 2 == 0;
isEven.test(4);  // true

Predicate<String> isEmpty = String::isEmpty;
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> combined = isEven.and(isPositive);

// 2. Function<T, R> - T -> R
Function<String, Integer> length = s -> s.length();
Function<String, Integer> length2 = String::length;
length.apply("Hello");  // 5

Function<Integer, Integer> square = n -> n * n;
Function<Integer, Integer> addOne = n -> n + 1;
Function<Integer, Integer> squareThenAddOne = square.andThen(addOne);

// 3. Consumer<T> - T -> void
Consumer<String> printer = s -> System.out.println(s);
Consumer<String> printer2 = System.out::println;
printer.accept("Hello");

// 4. Supplier<T> - () -> T
Supplier<Double> random = () -> Math.random();
Supplier<List<String>> listSupplier = ArrayList::new;
random.get();

// 5. BiFunction<T, U, R> - (T, U) -> R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
add.apply(2, 3);  // 5

// 6. BiPredicate<T, U> - (T, U) -> boolean
BiPredicate<String, Integer> checkLength = (s, len) -> s.length() == len;

// 7. UnaryOperator<T> - T -> T (specialization of Function)
UnaryOperator<Integer> doubleIt = n -> n * 2;

// 8. BinaryOperator<T> - (T, T) -> T (specialization of BiFunction)
BinaryOperator<Integer> sum = (a, b) -> a + b;
```

### 2. Lambda Expressions

```java
// Syntax: (parameters) -> expression
//         (parameters) -> { statements; }

// Examples
Runnable r = () -> System.out.println("Hello");

Comparator<String> comp = (s1, s2) -> s1.compareTo(s2);

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println(name));

// Effectively Final - Variables used in lambda must be final or effectively final
int multiplier = 2;  // effectively final
Function<Integer, Integer> multiply = n -> n * multiplier;
// multiplier = 3;  // This would cause compilation error
```

### 3. Method References

```java
// 4 Types of Method References

// 1. Static method reference - Class::staticMethod
Function<String, Integer> parser = Integer::parseInt;
// Equivalent: s -> Integer.parseInt(s)

// 2. Instance method of particular object - object::instanceMethod
String prefix = "Hello ";
Function<String, String> concat = prefix::concat;
// Equivalent: s -> prefix.concat(s)

// 3. Instance method of arbitrary object - Class::instanceMethod
Function<String, Integer> length = String::length;
// Equivalent: s -> s.length()

BiPredicate<String, String> startsWith = String::startsWith;
// Equivalent: (s1, s2) -> s1.startsWith(s2)

// 4. Constructor reference - Class::new
Supplier<ArrayList<String>> listCreator = ArrayList::new;
// Equivalent: () -> new ArrayList<>()

Function<Integer, int[]> arrayCreator = int[]::new;
// Equivalent: size -> new int[size]
```

### 4. Stream API

```java
// Creating Streams
Stream<String> stream1 = list.stream();
Stream<String> stream2 = Stream.of("a", "b", "c");
Stream<Integer> stream3 = Stream.iterate(0, n -> n + 1);
Stream<Double> stream4 = Stream.generate(Math::random);
IntStream stream5 = IntStream.range(1, 10);  // 1 to 9
IntStream stream6 = IntStream.rangeClosed(1, 10);  // 1 to 10

// Intermediate Operations (return Stream)
// filter, map, flatMap, distinct, sorted, peek, limit, skip

// Terminal Operations (return result)
// forEach, collect, reduce, count, findFirst, findAny, anyMatch, allMatch, noneMatch

List<Employee> employees = Arrays.asList(
    new Employee("Alice", 50000),
    new Employee("Bob", 60000),
    new Employee("Charlie", 55000)
);

// FILTER - Predicate
List<Employee> highEarners = employees.stream()
    .filter(e -> e.getSalary() > 55000)
    .collect(Collectors.toList());

// MAP - Transform
List<String> names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// FLATMAP - Flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4, 5)
);
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [1, 2, 3, 4, 5]

// SORTED
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary))
    .collect(Collectors.toList());

// REDUCE
int sum = IntStream.of(1, 2, 3, 4, 5).reduce(0, (a, b) -> a + b);
Optional<Integer> max = Stream.of(1, 2, 3).reduce(Integer::max);

// COLLECT - Collectors
// toList, toSet, toMap, joining, groupingBy, partitioningBy, counting, 
// summarizingInt, averagingInt, maxBy, minBy

// toList
List<String> list = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// toSet
Set<String> set = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toSet());

// toMap
Map<String, Integer> map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getName,
        Employee::getSalary
    ));

// joining
String joined = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// groupingBy
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// groupingBy with downstream collector
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// partitioningBy - split into two groups (true/false)
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 55000));

// Statistics
IntSummaryStatistics stats = employees.stream()
    .mapToInt(Employee::getSalary)
    .summaryStatistics();
stats.getMax();
stats.getMin();
stats.getAverage();
stats.getSum();
stats.getCount();

// MATCH operations
boolean anyMatch = employees.stream().anyMatch(e -> e.getSalary() > 55000);
boolean allMatch = employees.stream().allMatch(e -> e.getSalary() > 40000);
boolean noneMatch = employees.stream().noneMatch(e -> e.getSalary() < 0);

// FIND operations
Optional<Employee> first = employees.stream()
    .filter(e -> e.getSalary() > 55000)
    .findFirst();

Optional<Employee> any = employees.stream()
    .filter(e -> e.getSalary() > 55000)
    .findAny();  // Better for parallel streams

// Parallel Streams
employees.parallelStream()
    .filter(e -> e.getSalary() > 50000)
    .forEach(System.out::println);
```

### 5. Optional

```java
// Creating Optional
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("Hello");  // Throws NPE if null
Optional<String> nullable = Optional.ofNullable(null);  // Safe with null

// Checking and Getting
if (optional.isPresent()) {
    String value = optional.get();
}

// Better approaches
optional.ifPresent(System.out::println);

String value = optional.orElse("default");
String value2 = optional.orElseGet(() -> "computed default");
String value3 = optional.orElseThrow(() -> new RuntimeException("Not found"));

// Transforming
Optional<Integer> length = optional.map(String::length);
Optional<String> upper = optional.filter(s -> s.length() > 3).map(String::toUpperCase);

// flatMap for nested Optional
Optional<Optional<String>> nested = Optional.of(Optional.of("Hello"));
Optional<String> flat = nested.flatMap(Function.identity());
```

### 6. Default and Static Methods in Interfaces

```java
public interface Vehicle {
    // Abstract method
    void start();
    
    // Default method - can be overridden
    default void stop() {
        System.out.println("Vehicle stopping");
    }
    
    // Static method - cannot be overridden
    static void honk() {
        System.out.println("Honk!");
    }
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting");
    }
    
    @Override
    public void stop() {
        Vehicle.super.stop();  // Call default method
        System.out.println("Car stopped");
    }
}

// Calling static method
Vehicle.honk();  // Not Car.honk()
```

---

## 📊 HANDS-ON JAVA 8 PROBLEMS

```java
// Problem 1: Find second highest salary
public Optional<Employee> secondHighestSalary(List<Employee> employees) {
    return employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .skip(1)
        .findFirst();
}

// Problem 2: Group employees by department and find max salary in each
public Map<String, Optional<Employee>> maxSalaryByDept(List<Employee> employees) {
    return employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.maxBy(Comparator.comparing(Employee::getSalary))
        ));
}

// Problem 3: Find duplicate elements in list
public Set<Integer> findDuplicates(List<Integer> numbers) {
    Set<Integer> seen = new HashSet<>();
    return numbers.stream()
        .filter(n -> !seen.add(n))
        .collect(Collectors.toSet());
}

// Problem 4: Flatten and remove duplicates
public List<Integer> flattenUnique(List<List<Integer>> nested) {
    return nested.stream()
        .flatMap(List::stream)
        .distinct()
        .sorted()
        .collect(Collectors.toList());
}

// Problem 5: Count occurrences of each character
public Map<Character, Long> charFrequency(String str) {
    return str.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        ));
}

// Problem 6: Find longest string
public Optional<String> longestString(List<String> strings) {
    return strings.stream()
        .max(Comparator.comparingInt(String::length));
}

// Problem 7: Partition numbers into even and odd
public Map<Boolean, List<Integer>> partitionEvenOdd(List<Integer> numbers) {
    return numbers.stream()
        .collect(Collectors.partitioningBy(n -> n % 2 == 0));
}

// Problem 8: Convert list to comma-separated string
public String toCommaSeparated(List<String> strings) {
    return strings.stream()
        .collect(Collectors.joining(", "));
}

// Problem 9: Find average salary of employees older than 30
public double avgSalaryAbove30(List<Employee> employees) {
    return employees.stream()
        .filter(e -> e.getAge() > 30)
        .mapToDouble(Employee::getSalary)
        .average()
        .orElse(0.0);
}

// Problem 10: Sort and collect to LinkedHashMap (maintain order)
public Map<String, Integer> sortByValueDesc(Map<String, Integer> map) {
    return map.entrySet().stream()
        .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
        .collect(Collectors.toMap(
            Map.Entry::getKey,
            Map.Entry::getValue,
            (e1, e2) -> e1,
            LinkedHashMap::new
        ));
}
```

---

## 🗄️ SQL

### Basic Queries

```sql
-- SELECT, WHERE, ORDER BY
SELECT name, salary FROM employees WHERE salary > 50000 ORDER BY salary DESC;

-- DISTINCT
SELECT DISTINCT department FROM employees;

-- LIKE
SELECT * FROM employees WHERE name LIKE 'A%';  -- Starts with A
SELECT * FROM employees WHERE name LIKE '%son';  -- Ends with son
SELECT * FROM employees WHERE name LIKE '%a%';  -- Contains a

-- IN, BETWEEN
SELECT * FROM employees WHERE department IN ('IT', 'HR', 'Sales');
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;

-- NULL handling
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

### Aggregate Functions & GROUP BY

```sql
-- COUNT, SUM, AVG, MIN, MAX
SELECT COUNT(*) FROM employees;
SELECT department, COUNT(*) as emp_count FROM employees GROUP BY department;
SELECT department, AVG(salary) as avg_salary FROM employees GROUP BY department;

-- HAVING (filter groups)
SELECT department, AVG(salary) as avg_salary 
FROM employees 
GROUP BY department 
HAVING AVG(salary) > 50000;
```

### JOINS

```sql
-- INNER JOIN - Only matching rows
SELECT e.name, d.department_name 
FROM employees e 
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN - All left + matching right
SELECT e.name, d.department_name 
FROM employees e 
LEFT JOIN departments d ON e.department_id = d.id;

-- RIGHT JOIN - All right + matching left
SELECT e.name, d.department_name 
FROM employees e 
RIGHT JOIN departments d ON e.department_id = d.id;

-- FULL OUTER JOIN - All from both
SELECT e.name, d.department_name 
FROM employees e 
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- SELF JOIN
SELECT e1.name as employee, e2.name as manager 
FROM employees e1 
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

### Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM
SELECT dept_avg.department, dept_avg.avg_salary
FROM (
    SELECT department, AVG(salary) as avg_salary 
    FROM employees 
    GROUP BY department
) dept_avg
WHERE dept_avg.avg_salary > 50000;

-- EXISTS
SELECT * FROM departments d 
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.id);
```

### Common Interview Questions

```sql
-- 1. Second highest salary
SELECT MAX(salary) FROM employees 
WHERE salary < (SELECT MAX(salary) FROM employees);

-- OR using LIMIT (MySQL)
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;

-- 2. Nth highest salary
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET N-1;

-- 3. Employees with no manager
SELECT * FROM employees WHERE manager_id IS NULL;

-- 4. Duplicate records
SELECT name, COUNT(*) FROM employees GROUP BY name HAVING COUNT(*) > 1;

-- 5. Delete duplicates (keep one)
DELETE e1 FROM employees e1
INNER JOIN employees e2 
WHERE e1.id > e2.id AND e1.name = e2.name;

-- 6. Department with highest average salary
SELECT department FROM employees 
GROUP BY department 
ORDER BY AVG(salary) DESC 
LIMIT 1;

-- 7. Employees earning more than their manager
SELECT e.name FROM employees e 
JOIN employees m ON e.manager_id = m.id 
WHERE e.salary > m.salary;

-- 8. CASE WHEN
SELECT name, salary,
    CASE 
        WHEN salary > 70000 THEN 'High'
        WHEN salary > 50000 THEN 'Medium'
        ELSE 'Low'
    END as salary_grade
FROM employees;
```

---

## 🧪 JUNIT

### JUnit 5 Basics

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeAll
    static void setupAll() {
        System.out.println("Runs once before all tests");
    }
    
    @BeforeEach
    void setup() {
        calculator = new Calculator();
    }
    
    @AfterEach
    void tearDown() {
        System.out.println("Runs after each test");
    }
    
    @AfterAll
    static void tearDownAll() {
        System.out.println("Runs once after all tests");
    }
    
    @Test
    void testAdd() {
        assertEquals(5, calculator.add(2, 3));
    }
    
    @Test
    @DisplayName("Test Division by Zero")
    void testDivideByZero() {
        assertThrows(ArithmeticException.class, 
            () -> calculator.divide(10, 0));
    }
    
    @Test
    void testMultipleAssertions() {
        assertAll(
            () -> assertEquals(4, calculator.add(2, 2)),
            () -> assertEquals(0, calculator.add(0, 0)),
            () -> assertEquals(-1, calculator.add(2, -3))
        );
    }
    
    @Test
    @Disabled("Not implemented yet")
    void testSubtract() {
        // Test disabled
    }
    
    @ParameterizedTest
    @ValueSource(ints = {2, 4, 6, 8})
    void testIsEven(int number) {
        assertTrue(calculator.isEven(number));
    }
    
    @ParameterizedTest
    @CsvSource({"1, 2, 3", "5, 5, 10", "0, 0, 0"})
    void testAddWithCsv(int a, int b, int expected) {
        assertEquals(expected, calculator.add(a, b));
    }
}
```

### Common Assertions

```java
// Basic assertions
assertEquals(expected, actual);
assertNotEquals(unexpected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(object);
assertNotNull(object);
assertSame(obj1, obj2);  // Same reference
assertNotSame(obj1, obj2);

// Arrays
assertArrayEquals(expectedArray, actualArray);

// Exception
assertThrows(ExceptionType.class, () -> methodThatThrows());
assertDoesNotThrow(() -> methodThatDoesNotThrow());

// Timeout
assertTimeout(Duration.ofSeconds(1), () -> longRunningMethod());

// Grouped assertions
assertAll(
    () -> assertEquals(1, 1),
    () -> assertEquals(2, 2)
);
```

### Mocking with Mockito

```java
import org.mockito.Mock;
import org.mockito.Mockito;
import static org.mockito.Mockito.*;

public class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @BeforeEach
    void setup() {
        MockitoAnnotations.openMocks(this);
    }
    
    @Test
    void testFindUser() {
        // Arrange
        User expectedUser = new User(1, "John");
        when(userRepository.findById(1)).thenReturn(Optional.of(expectedUser));
        
        // Act
        User actualUser = userService.findById(1);
        
        // Assert
        assertEquals(expectedUser, actualUser);
        verify(userRepository, times(1)).findById(1);
    }
    
    @Test
    void testSaveUser() {
        User user = new User(1, "John");
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        userService.save(user);
        
        verify(userRepository).save(user);
    }
}
```

---

## 📝 QUICK REVISION CHECKLIST

### Problem Solving
- [ ] Two Pointer technique
- [ ] Sliding Window
- [ ] Binary Search variations
- [ ] LinkedList operations (reverse, cycle, merge)
- [ ] Stack problems (valid parentheses, next greater)
- [ ] String manipulation

### Java Fundamentals
- [ ] OOP concepts (SOLID)
- [ ] Collections (HashMap internals, when to use what)
- [ ] Exception handling (checked vs unchecked)
- [ ] String vs StringBuilder vs StringBuffer
- [ ] equals() and hashCode() contract

### Java 8
- [ ] Functional Interfaces (Predicate, Function, Consumer, Supplier)
- [ ] Lambda expressions
- [ ] Method references (4 types)
- [ ] Stream API (intermediate vs terminal operations)
- [ ] Collectors (groupingBy, partitioningBy, toMap)
- [ ] Optional

### SQL
- [ ] JOINs (INNER, LEFT, RIGHT, FULL)
- [ ] GROUP BY with HAVING
- [ ] Subqueries
- [ ] Common problems (nth highest, duplicates)

### JUnit
- [ ] Annotations (@Test, @BeforeEach, @AfterEach)
- [ ] Assertions (assertEquals, assertThrows)
- [ ] Parameterized tests

---

**Good luck with your interview! 🚀**
