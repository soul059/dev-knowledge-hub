# JSP Lifecycle

The lifecycle of a JavaServer Page (JSP) is the process that begins with its creation and ends with its destruction. It is similar to the lifecycle of a servlet, but with an additional step for compiling the JSP into a servlet. The JSP engine automatically manages this entire process.

The key phases of the JSP lifecycle are:

### 1. Translation

In this initial phase, the JSP container translates the `.jsp` file into a Java servlet source file (`.java`). The container parses the JSP file, and its elements are converted into Java servlet code. This translation occurs the first time the JSP is requested or when the JSP file has been modified since the last request.

### 2. Compilation

The generated Java servlet source file is then compiled into a Java bytecode file (`.class`). This step converts the servlet code into bytecode that can be executed by the Java Virtual Machine (JVM).

### 3. Class Loading

The compiled servlet class is loaded into memory by the classloader.

### 4. Instantiation

The JSP engine creates an instance of the loaded servlet class. This single instance will typically handle multiple requests.

### 5. Initialization

After instantiation, the container calls the `jspInit()` method only once. This method is used to perform any one-time setup, such as initializing database connections or opening files. You can override this method for custom initialization logic.

```java
public void jspInit() {
    // Initialization code
}
```

### 6. Request Processing

For every client request, the JSP engine invokes the `_jspService()` method. This method is responsible for generating the response for each request and handles all HTTP methods like GET and POST. The `_jspService()` method receives `HttpServletRequest` and `HttpServletResponse` objects as parameters. This method cannot be overridden by the developer.

```java
void _jspService(HttpServletRequest request, HttpServletResponse response) {
    // Request handling code
}
```

### 7. Destruction

When the JSP is about to be removed from service by the container, the `jspDestroy()` method is called. This method is invoked only once and is the last phase of the lifecycle. It is used for cleanup activities, such as releasing resources that were allocated in the `jspInit()` method.

```java
public void jspDestroy() {
    // Cleanup code
}
```
