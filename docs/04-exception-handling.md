# 4. Exception Handling - Complete Guide

## 📊 Interview Frequency: 95% | Expected Questions: 8

---

## Exception Hierarchy

```
                     Throwable
                    /         \
                   /           \
              Error            Exception
            (JVM)         (Application)
             /           /              \
            /           /                \
    OutOfMemory   Checked         Unchecked (Runtime)
    StackOverflow  ┌─────────────┐     ├─ NullPointerException
                  /              \    ├─ ArithmeticException
           IOException    RuntimeException
           SQLException   ClassCastException
           FileNotFound   IndexOutOfBounds
                          IllegalArgument
```

### Key Differences:

| Type | Checked | Unchecked (Runtime) | Error |
|------|---------|-------------|-------|
| Inherits from | Exception | RuntimeException | Error |
| Must handle? | YES (compile time) | NO (optional) | NO |
| When caught? | Compile time | Runtime | System level |
| Examples | IOException, SQLException | NullPointerException, ArithmeticException | OutOfMemoryError, StackOverflowError |
| Cause | External (file not found) | Programming error | JVM issue |

---

## 1️⃣ Try-Catch-Finally

### Basic Syntax:

```java
public class ExceptionHandlingExample {
    public static void main(String[] args) {
        try {
            // Code that might throw exception
            int[] numbers = {1, 2, 3};
            System.out.println(numbers[5]);  // ArrayIndexOutOfBoundsException
        } catch (ArrayIndexOutOfBoundsException e) {
            // Handle specific exception
            System.out.println("Array index out of bounds!");
            e.printStackTrace();  // Print stack trace for debugging
        } finally {
            // Always executes, regardless of exception
            System.out.println("Cleanup code");
        }
    }
}
```

### Multiple Catch Blocks:

```java
public void processFile(String filename) {
    try {
        // Risky operation
        FileReader reader = new FileReader(filename);
        int data = reader.read();
        int result = data / 0;  // Potential division by zero
    } catch (FileNotFoundException e) {
        // Specific handling for file not found
        System.out.println("File not found: " + e.getMessage());
        // Recover or log
    } catch (IOException e) {
        // Handle IO errors
        System.out.println("IO error: " + e.getMessage());
    } catch (ArithmeticException e) {
        // Handle arithmetic errors
        System.out.println("Math error: " + e.getMessage());
    } catch (Exception e) {
        // Catch-all for any other exceptions
        System.out.println("Unexpected error: " + e.getMessage());
    } finally {
        // Cleanup (close resources)
        System.out.println("Processing complete");
    }
}
```

### Multi-Catch (Java 7+):

```java
try {
    // risky code
} catch (FileNotFoundException | IOException | ParseException e) {
    // Handle multiple exceptions with same logic
    System.out.println("Error: " + e.getMessage());
    logger.error(e);
}
```

---

## 2️⃣ Try-With-Resources (Java 7+)

### Automatically Closes Resources:

```java
// ❌ Old way - must close manually
public void readFileOld(String filename) throws IOException {
    FileReader reader = null;
    try {
        reader = new FileReader(filename);
        char[] data = new char[1024];
        reader.read(data);
    } finally {
        if (reader != null) {
            reader.close();  // Must remember to close
        }
    }
}

// ✅ New way - auto-closes
public void readFileNew(String filename) throws IOException {
    try (FileReader reader = new FileReader(filename)) {
        // reader is auto-closed after try block
        char[] data = new char[1024];
        reader.read(data);
    }
    // reader.close() called automatically
}

// ✅ Multiple resources
public void copyFile(String source, String dest) throws IOException {
    try (FileReader reader = new FileReader(source);
         FileWriter writer = new FileWriter(dest)) {
        // Both auto-closed after block
        int character;
        while ((character = reader.read()) != -1) {
            writer.write(character);
        }
    }
}
```

---

## 3️⃣ Custom Exceptions

### Creating Custom Exception:

```java
// Custom checked exception
public class InsufficientFundsException extends Exception {
    private double amount;
    
    public InsufficientFundsException(String message, double amount) {
        super(message);
        this.amount = amount;
    }
    
    public double getAmount() {
        return amount;
    }
}

// Custom unchecked exception
public class InvalidAgeException extends RuntimeException {
    public InvalidAgeException(String message) {
        super(message);
    }
}

// Using custom exceptions
public class BankAccount {
    private double balance = 1000;
    
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Insufficient balance. Available: " + balance,
                amount
            );
        }
        balance -= amount;
    }
}

public class User {
    private int age;
    
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new InvalidAgeException("Age must be between 0 and 150");
        }
        this.age = age;
    }
}

// Handling custom exceptions
public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        
        try {
            account.withdraw(2000);
        } catch (InsufficientFundsException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Requested: " + e.getAmount());
        }
        
        User user = new User();
        try {
            user.setAge(200);
        } catch (InvalidAgeException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

## 4️⃣ Throws vs Throw

### throws - Declare Exception:

```java
// Method declares it throws checked exception
public void readFile(String filename) throws FileNotFoundException, IOException {
    FileReader reader = new FileReader(filename);
    reader.read();
    reader.close();
    // Caller must handle or propagate exception
}

// Caller must catch
try {
    readFile("data.txt");
} catch (FileNotFoundException e) {
    System.out.println("File not found");
} catch (IOException e) {
    System.out.println("IO error");
}
```

### throw - Throw Exception:

```java
public void validateAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("Must be 18 or older");
    }
}

// Usage
try {
    validateAge(15);
} catch (IllegalArgumentException e) {
    System.out.println("Error: " + e.getMessage());
}
```

---

## 5️⃣ Exception Chaining

### Wrapping Exceptions:

```java
public class DataProcessor {
    public void processData(String filename) throws ProcessingException {
        try {
            FileReader reader = new FileReader(filename);
            int data = reader.read();
            parseData(data);
            reader.close();
        } catch (FileNotFoundException e) {
            // Wrap low-level exception with high-level context
            throw new ProcessingException("Failed to process file: " + filename, e);
        } catch (IOException e) {
            throw new ProcessingException("IO error during processing", e);
        }
    }
}

// Custom exception with cause
public class ProcessingException extends Exception {
    public ProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Usage
try {
    processor.processData("data.txt");
} catch (ProcessingException e) {
    System.out.println(e.getMessage());
    System.out.println("Caused by: " + e.getCause());
    e.printStackTrace();
}
```

---

## 6️⃣ Best Practices

### ❌ Don't Do This:

```java
// 1. Empty catch - swallows exception silently
try {
    risky();
} catch (Exception e) {
    // ❌ Bad: exception ignored
}

// 2. Catch generic Exception
try {
    risky();
} catch (Exception e) {
    // ❌ Bad: catches everything, hard to debug
}

// 3. Catch Throwable
try {
    risky();
} catch (Throwable t) {
    // ❌ Bad: even catches Errors
}

// 4. Ignore checked exceptions
List<String> lines = Files.readAllLines(path);  // ❌ Must handle IOException
```

### ✅ Do This Instead:

```java
// 1. Catch specific exceptions
try {
    risky();
} catch (FileNotFoundException e) {
    logger.error("File not found", e);
    // Handle appropriately
} catch (IOException e) {
    logger.error("IO error", e);
    // Handle appropriately
}

// 2. Log with context
try {
    operation();
} catch (OperationException e) {
    logger.error("Operation failed for user: " + userId, e);
}

// 3. Use try-with-resources
try (FileReader reader = new FileReader("file.txt")) {
    // Work with reader
} catch (IOException e) {
    logger.error("Error reading file", e);
}

// 4. Re-throw with context
tryCatch {
    processData();
} catch (DataException e) {
    logger.error("Failed to process data for dataset: " + datasetId, e);
    throw new ApplicationException("Data processing failed", e);
}
```

---

## Real-World Example: API Request Handler

```java
public class ApiService {
    private static final Logger logger = LoggerFactory.getLogger(ApiService.class);
    
    public Response callExternalApi(String url) throws ApiException {
        try {
            // Make HTTP request
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder(URI.create(url)).build();
            HttpResponse<String> response = client.send(request, 
                HttpResponse.BodyHandlers.ofString());
            
            if (response.statusCode() != 200) {
                throw new ApiException(
                    "API returned status: " + response.statusCode()
                );
            }
            
            return parseResponse(response.body());
        } catch (HttpConnectException e) {
            logger.error("Failed to connect to API: " + url, e);
            throw new ApiException("Connection failed", e);
        } catch (HttpTimeoutException e) {
            logger.error("API request timeout for: " + url, e);
            throw new ApiException("Request timeout", e);
        } catch (IOException e) {
            logger.error("IO error calling API: " + url, e);
            throw new ApiException("IO error", e);
        } catch (ParseException e) {
            logger.error("Failed to parse API response", e);
            throw new ApiException("Response parsing failed", e);
        }
    }
    
    private Response parseResponse(String json) throws ParseException {
        if (json == null || json.isEmpty()) {
            throw new ParseException("Empty response");
        }
        // Parse JSON
        return new Response(json);
    }
}

// Custom API exception
public class ApiException extends Exception {
    public ApiException(String message) {
        super(message);
    }
    
    public ApiException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        ApiService service = new ApiService();
        
        try {
            Response response = service.callExternalApi("https://api.example.com/data");
            System.out.println("Success: " + response.getData());
        } catch (ApiException e) {
            System.err.println("API call failed: " + e.getMessage());
            // Implement retry logic, fallback, etc.
        }
    }
}
```

---

## Interview Tips:

✅ **Mention checked vs unchecked exceptions**

✅ **Show try-with-resources for resource cleanup**

✅ **Demonstrate custom exceptions**

✅ **Explain exception chaining for context**

✅ **Emphasize logging over silent failures**

---

📖 **Next Topic:** [Multithreading](./05-multithreading.md)
