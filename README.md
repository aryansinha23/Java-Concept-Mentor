# Java Concept & Code Mentor

An offline, terminal-based Java tutor chatbot for the OOPs with Java assignment. It uses a small object-oriented rule engine and curated topic knowledge (Track A); it does not call a cloud model or need an API key. For a recognized topic, it always gives the assignment's three stages: conceptual grounding, algorithmic blueprint, and a complete Java example with two test scenarios.

## Architecture

- `ChatbotApp` owns the interactive terminal session and exit/help behavior.
- `ChatbotEngine` selects a topic using normalized keyword matching and a small edit-distance fallback, then formats the three-phase response.
- `TopicCatalog` contains the offline topic data: definitions, relevant APIs, production context, steps, edge cases, complexity, and complete code examples.
- `Topic` is an immutable data model.

The catalog includes 25 topics, including OOP, collections, generics, exceptions, I/O, streams, lambdas, arrays, recursion, threads, synchronization, and atomics. This is intentionally a curated teaching chatbot, not an unrestricted generative AI: an unrecognized question receives suggested topics instead of fabricated code.

## Prerequisites

- JDK 17 or later
- Apache Maven 3.8 or later

Check your installation:

```bash
java -version
mvn -version
```

## Build and run

From the project root:

```bash
mvn clean package
java -jar target/java-concept-code-mentor-1.0.0.jar
```

## Complete copyable sample dialogues

### Dialogue 1: Create a thread with `Runnable`

```text
You: How do I create a custom thread in Java using Runnable?
Matched topic: Threads with Runnable

[PHASE 1: CONCEPTUAL BREAKDOWN]
Core concept and runtime: Runnable separates task logic from a Thread. Calling start() asks the JVM to schedule a new thread; calling run() directly is an ordinary method call on the current thread. Java thread stacks are distinct, while heap objects may be shared.
Syntax and APIs: Use java.lang.Runnable (a functional interface), new Thread(task), start(), and join(). For managed production work, prefer ExecutorService or other java.util.concurrent APIs over creating unbounded raw threads.
Production context: Runnable is appropriate for expressing a task independent of its execution mechanism. Use bounded executors in servers; avoid one thread per request and never assume thread scheduling order.

[PHASE 2: ALGORITHMIC BLUEPRINT]
1. Put task logic in Runnable.run or a lambda.
2. Create a Thread with the task.
3. Call start, not run.
4. Join when the caller must wait for completion.
Edge cases and design: join throws InterruptedException; restore the interrupt flag or propagate it. Shared mutable state needs synchronization/atomics. Thread.start may be called only once.
Time complexity: O(1) for creating one thread plus the task's work; scheduling overhead is platform-dependent.
Space complexity: O(1) task state here, plus a native/Java stack per thread.

[PHASE 3: COMPLETE JAVA IMPLEMENTATION & TEST HARNESS]
Save as Example.java, then run: javac Example.java && java Example
```

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.*;
import java.util.stream.*;

public class Example {
    static String solve(String input) throws Exception {
        int repetitions = Integer.parseInt(input);
        AtomicInteger completed = new AtomicInteger();
        Runnable task = completed::incrementAndGet;
        Thread worker = new Thread(() -> {
            for (int i = 0; i < repetitions; i++) task.run();
        }, "worker-thread");
        worker.start();
        worker.join();
        return Integer.toString(completed.get());
    }

    private static void check(String scenario, String input, String expected) throws Exception {
        String actual = solve(input);
        if (!Objects.equals(expected, actual)) {
            throw new AssertionError(scenario + " expected <" + expected + "> but got <" + actual + ">");
        }
        System.out.println(scenario + " PASS: " + actual);
    }

    public static void main(String[] args) throws Exception {
        check("Scenario 1: ordinary input", "3", "3");
        check("Scenario 2: alternate or edge input", "1", "1");
    }
}
```

```text
You: exit
Goodbye. Keep practicing Java!
```

### Dialogue 2: Sum even numbers with streams

```text
You: How do I sum even numbers using streams?
Matched topic: Streams

[PHASE 1: CONCEPTUAL BREAKDOWN]
Core concept and runtime: A Stream is a one-use pipeline over data: a source, intermediate operations, and a terminal operation. Intermediate operations are generally lazy; a terminal operation triggers traversal.
Syntax and APIs: Use stream(), filter, map, sorted, distinct, and terminal operations such as collect, count, or reduce from java.util.stream.
Production context: Streams can make data transformations expressive and support parallel processing when appropriate. Avoid side effects in pipelines and do not assume parallel streams are automatically faster.

[PHASE 2: ALGORITHMIC BLUEPRINT]
1. Create a stream from a source.
2. Apply transformations such as filter/map.
3. Finish with a terminal operation.
4. Verify empty and malformed input behavior.
Edge cases and design: A stream cannot be reused after a terminal operation. Parsing can throw NumberFormatException; parallel execution can change encounter order unless explicitly preserved.
Time complexity: O(n) for filtering/summing n values.
Space complexity: O(1) auxiliary space for a sequential sum; collectors may require O(n).

[PHASE 3: COMPLETE JAVA IMPLEMENTATION & TEST HARNESS]
Save as Example.java, then run: javac Example.java && java Example
```

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.*;
import java.util.stream.*;

public class Example {
    static String solve(String input) {
        int sum = Arrays.stream(input.split(","))
                .filter(token -> !token.isBlank())
                .mapToInt(Integer::parseInt)
                .filter(value -> value % 2 == 0)
                .sum();
        return Integer.toString(sum);
    }

    private static void check(String scenario, String input, String expected) throws Exception {
        String actual = solve(input);
        if (!Objects.equals(expected, actual)) {
            throw new AssertionError(scenario + " expected <" + expected + "> but got <" + actual + ">");
        }
        System.out.println(scenario + " PASS: " + actual);
    }

    public static void main(String[] args) throws Exception {
        check("Scenario 1: ordinary input", "1,2,3,4", "6");
        check("Scenario 2: alternate or edge input", "6,8", "14");
    }
}
```

```text
You: exit
Goodbye. Keep practicing Java!
```

Try `Runnable`, `streams`, `ArrayList`, `HashMap`, `generics`, `exception handling`, or `recursion`. Matching is case-insensitive. An unknown question produces a list of supported topic examples.

## Tests and smoke test

Run the automated project tests, including compilation and execution of each catalog example:

```bash
mvn test
```

The tests verify all 25 example programs and their two scenarios, the three-phase response, and the unknown-topic fallback.

Run the app with a few questions and verify all stages are printed:

```bash
printf 'Runnable\nstreams\nnot a Java topic\nexit\n' | java -jar target/java-concept-code-mentor-1.0.0.jar
```

Every generated Phase 3 example is a standalone `Example.java` source file with a `main` method and two descriptive test scenarios. Copy a code block into `Example.java` and run `javac Example.java && java Example`.

## GitHub submission checklist

1. Create a new GitHub repository and push this project (do not commit `target/` or API secrets).
2. Confirm the **Java CI** workflow passes under the repository's **Actions** tab after the first push.
3. Include your own screenshots or a short terminal transcript if your instructor requests evidence.
4. Explain that this is the offline Track A approach and that new topics must be added to `TopicCatalog`.

The included GitHub Actions workflow runs Maven verification on pushes and pull requests. The project contains no API key, account credential, or external service dependency.
