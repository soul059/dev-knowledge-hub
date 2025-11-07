# A Guide to Design Patterns in Java

Design patterns are proven, reusable solutions to commonly occurring problems within a given context in software design. They are not finished designs that can be transformed directly into code; rather, they are templates for how to solve a problem that can be used in many different situations.

---

## Why Use Design Patterns?

-   **Common Platform**: They provide a standard terminology and understanding for developers.
-   **Proven Solutions**: They are well-tested solutions to common problems, which helps prevent subtle issues.
-   **Code Reusability**: They promote reusability that leads to more robust and maintainable code.
-   **Improved Readability**: Using a standard pattern makes the code's intent clearer to other developers.

Design patterns are typically categorized into three main groups:

1.  **Creational Patterns**: Deal with object creation mechanisms.
2.  **Structural Patterns**: Deal with object composition and relationships.
3.  **Behavioral Patterns**: Deal with communication between objects.

---

## 1. Creational Patterns

These patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.

### a) Singleton Pattern

**Intent**: Ensures that a class has only one instance and provides a global point of access to it.

**When to use**: When exactly one object is needed to coordinate actions across the system, such as for a logger, a database connection pool, or a configuration manager.

**Example (Thread-Safe):**
```java
public class DatabaseConnection {
    // The single instance, volatile to ensure visibility across threads.
    private static volatile DatabaseConnection instance;

    private DatabaseConnection() {
        // Private constructor to prevent instantiation from outside.
    }

    // Public method to provide access to the single instance.
    public static DatabaseConnection getInstance() {
        // Double-checked locking for thread safety and performance.
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }

    public void connect() {
        System.out.println("Connected to the database.");
    }
}
```

### b) Factory Method Pattern

**Intent**: Defines an interface for creating an object, but lets subclasses decide which class to instantiate.

**When to use**: When a class cannot anticipate the class of objects it must create, or when a class wants its subclasses to specify the objects it creates.

**Example:**
```java
// The Product interface
interface Document {
    void open();
}

// Concrete Products
class WordDocument implements Document {
    public void open() { System.out.println("Opening Word document."); }
}

class PdfDocument implements Document {
    public void open() { System.out.println("Opening PDF document."); }
}

// The Creator (Factory)
abstract class DocumentFactory {
    // The factory method
    public abstract Document createDocument();

    public void openDocument() {
        Document doc = createDocument();
        doc.open();
    }
}

// Concrete Creators
class WordDocumentFactory extends DocumentFactory {
    public Document createDocument() { return new WordDocument(); }
}

class PdfDocumentFactory extends DocumentFactory {
    public Document createDocument() { return new PdfDocument(); }
}
```

### c) Builder Pattern

**Intent**: Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

**When to use**: To avoid "telescoping constructors" (constructors with many parameters) and to create complex, immutable objects step-by-step.

**Example:**
```java
// The complex object to be built
class Computer {
    private final String cpu;
    private final String ram;
    private final String storage;
    // Optional parameters
    private final String graphicsCard;
    private final String os;

    private Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.storage = builder.storage;
        this.graphicsCard = builder.graphicsCard;
        this.os = builder.os;
    }

    // The static nested Builder class
    public static class Builder {
        private final String cpu; // required
        private final String ram;   // required
        private String storage = "256GB SSD"; // default value
        private String graphicsCard; // optional
        private String os;           // optional

        public Builder(String cpu, String ram) {
            this.cpu = cpu;
            this.ram = ram;
        }

        public Builder storage(String storage) {
            this.storage = storage;
            return this;
        }

        public Builder graphicsCard(String graphicsCard) {
            this.graphicsCard = graphicsCard;
            return this;
        }

        public Builder os(String os) {
            this.os = os;
            return this;
        }

        public Computer build() {
            return new Computer(this);
        }
    }
}

// Usage:
// Computer gamingPC = new Computer.Builder("Intel i9", "32GB")
//     .storage("2TB NVMe SSD")
//     .graphicsCard("NVIDIA RTX 4090")
//     .os("Windows 11")
//     .build();
```

### d) Abstract Factory Pattern

**Intent**: Provides an interface for creating families of related or dependent objects without specifying their concrete classes.

**When to use**: When your system needs to be independent of how its products are created, composed, and represented, and when you need to provide a library of products that reveals only their interfaces, not their implementations.

**Example:**
```java
// Abstract Products
interface Button {
    void render();
}

interface Checkbox {
    void render();
}

// Concrete Products for Windows
class WindowsButton implements Button {
    public void render() { System.out.println("Rendering Windows Button"); }
}

class WindowsCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Windows Checkbox"); }
}

// Concrete Products for Mac
class MacButton implements Button {
    public void render() { System.out.println("Rendering Mac Button"); }
}

class MacCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Mac Checkbox"); }
}

// Abstract Factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factories
class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Client code
class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void render() {
        button.render();
        checkbox.render();
    }
}

// Usage:
// GUIFactory factory = new WindowsFactory(); // or new MacFactory()
// Application app = new Application(factory);
// app.render();
```

### e) Prototype Pattern

**Intent**: Creates new objects by cloning an existing object, known as the prototype.

**When to use**: When the cost of creating a new object is expensive or complex, and you want to avoid the overhead of initialization.

**Example:**
```java
// The Prototype interface
interface Shape extends Cloneable {
    Shape clone();
    void draw();
}

// Concrete Prototypes
class Circle implements Shape {
    private int radius;
    private String color;

    public Circle(int radius, String color) {
        this.radius = radius;
        this.color = color;
    }

    @Override
    public Shape clone() {
        return new Circle(this.radius, this.color);
    }

    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " circle with radius " + radius);
    }

    public void setRadius(int radius) { this.radius = radius; }
    public void setColor(String color) { this.color = color; }
}

class Rectangle implements Shape {
    private int width;
    private int height;
    private String color;

    public Rectangle(int width, int height, String color) {
        this.width = width;
        this.height = height;
        this.color = color;
    }

    @Override
    public Shape clone() {
        return new Rectangle(this.width, this.height, this.color);
    }

    @Override
    public void draw() {
        System.out.println("Drawing a " + color + " rectangle " + width + "x" + height);
    }

    public void setWidth(int width) { this.width = width; }
    public void setHeight(int height) { this.height = height; }
    public void setColor(String color) { this.color = color; }
}

// Usage:
// Shape originalCircle = new Circle(10, "red");
// Shape clonedCircle = originalCircle.clone();
// clonedCircle.draw(); // Drawing a red circle with radius 10
```

---

## 2. Structural Patterns

These patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

### a) Adapter Pattern

**Intent**: Allows objects with incompatible interfaces to collaborate.

**When to use**: When you want to use an existing class, but its interface does not match the one you need.

**Example:**
```java
// The target interface that the client expects
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// The adaptee - an existing class with an incompatible interface
class AdvancedMediaPlayer {
    public void playVlc(String fileName) { System.out.println("Playing vlc file: " + fileName); }
    public void playMp4(String fileName) { System.out.println("Playing mp4 file: " + fileName); }
}

// The Adapter class
class MediaAdapter implements MediaPlayer {
    AdvancedMediaPlayer advancedPlayer = new AdvancedMediaPlayer();

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}

// The client code only works with the target interface
class AudioPlayer implements MediaPlayer {
    MediaAdapter mediaAdapter;

    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        } else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter();
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media type: " + audioType);
        }
    }
}
```

### b) Decorator Pattern

**Intent**: Attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**When to use**: When you want to add responsibilities to individual objects dynamically and transparently, without affecting other objects.

**Example:**
```java
// The Component interface
interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete Component
class SimpleCoffee implements Coffee {
    public double getCost() { return 2.0; }
    public String getDescription() { return "Simple coffee"; }
}

// The base Decorator class
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee decoratedCoffee;

    public CoffeeDecorator(Coffee coffee) { this.decoratedCoffee = coffee; }
    public double getCost() { return decoratedCoffee.getCost(); }
    public String getDescription() { return decoratedCoffee.getDescription(); }
}

// Concrete Decorators
class WithMilk extends CoffeeDecorator {
    public WithMilk(Coffee coffee) { super(coffee); }
    public double getCost() { return super.getCost() + 0.5; }
    public String getDescription() { return super.getDescription() + ", with milk"; }
}

class WithSugar extends CoffeeDecorator {
    public WithSugar(Coffee coffee) { super(coffee); }
    public double getCost() { return super.getCost() + 0.2; }
    public String getDescription() { return super.getDescription() + ", with sugar"; }
}

// Usage:
// Coffee myCoffee = new SimpleCoffee();
// myCoffee = new WithMilk(myCoffee);
// myCoffee = new WithSugar(myCoffee);
// System.out.println(myCoffee.getCost()); // 2.7
// System.out.println(myCoffee.getDescription()); // Simple coffee, with milk, with sugar
```

### c) Facade Pattern

**Intent**: Provides a unified, simplified interface to a complex subsystem, making the subsystem easier to use.

**When to use**: When you want to provide a simple interface to a complex system, or when you want to layer your subsystems.

**Example:**
```java
// Complex subsystem classes
class DVDPlayer {
    public void on() { System.out.println("DVD Player is ON"); }
    public void play(String movie) { System.out.println("Playing movie: " + movie); }
    public void off() { System.out.println("DVD Player is OFF"); }
}

class Projector {
    public void on() { System.out.println("Projector is ON"); }
    public void setInput(String input) { System.out.println("Projector input set to: " + input); }
    public void off() { System.out.println("Projector is OFF"); }
}

class SoundSystem {
    public void on() { System.out.println("Sound System is ON"); }
    public void setVolume(int level) { System.out.println("Volume set to: " + level); }
    public void off() { System.out.println("Sound System is OFF"); }
}

class Lights {
    public void dim(int level) { System.out.println("Lights dimmed to: " + level + "%"); }
    public void on() { System.out.println("Lights are ON"); }
}

// The Facade class
class HomeTheaterFacade {
    private DVDPlayer dvdPlayer;
    private Projector projector;
    private SoundSystem soundSystem;
    private Lights lights;

    public HomeTheaterFacade(DVDPlayer dvd, Projector proj, SoundSystem sound, Lights lights) {
        this.dvdPlayer = dvd;
        this.projector = proj;
        this.soundSystem = sound;
        this.lights = lights;
    }

    // Simplified methods
    public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...");
        lights.dim(10);
        projector.on();
        projector.setInput("DVD");
        soundSystem.on();
        soundSystem.setVolume(5);
        dvdPlayer.on();
        dvdPlayer.play(movie);
    }

    public void endMovie() {
        System.out.println("Shutting down the theater...");
        dvdPlayer.off();
        soundSystem.off();
        projector.off();
        lights.on();
    }
}

// Usage:
// HomeTheaterFacade homeTheater = new HomeTheaterFacade(
//     new DVDPlayer(), new Projector(), new SoundSystem(), new Lights()
// );
// homeTheater.watchMovie("Inception");
// homeTheater.endMovie();
```

---

## 3. Behavioral Patterns

These patterns are concerned with algorithms and the assignment of responsibilities between objects.

### a) Observer Pattern

**Intent**: Defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (the observers) are notified and updated automatically.

**When to use**: When a change to one object requires changing others, and you don't know how many objects need to be changed.

**Example:**
```java
import java.util.ArrayList;
import java.util.List;

// The Observer interface
interface Observer {
    void update(String news);
}

// The Subject interface
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// Concrete Subject
class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }

    public void registerObserver(Observer o) { observers.add(o); }
    public void removeObserver(Observer o) { observers.remove(o); }
    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(news);
        }
    }
}

// Concrete Observer
class NewsChannel implements Observer {
    private String channelName;
    public NewsChannel(String name) { this.channelName = name; }
    public void update(String news) {
        System.out.println(channelName + " received news: " + news);
    }
}
```

### b) Strategy Pattern

**Intent**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**When to use**: When you have multiple variants of an algorithm and you want to switch between them at runtime.

**Example:**
```java
// The Strategy interface
interface PaymentStrategy {
    void pay(int amount);
}

// Concrete Strategies
class CreditCardStrategy implements PaymentStrategy {
    public void pay(int amount) { System.out.println("Paid " + amount + " with Credit Card."); }
}

class PayPalStrategy implements PaymentStrategy {
    public void pay(int amount) { System.out.println("Paid " + amount + " with PayPal."); }
}

// The Context class
class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

// Usage:
// ShoppingCart cart = new ShoppingCart();
// cart.setPaymentStrategy(new PayPalStrategy());
// cart.checkout(100); // Paid 100 with PayPal.
```

### c) Iterator Pattern

**Intent**: Provides a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

**When to use**: When you need to traverse a collection without exposing its internal structure, or when you want to support multiple traversals of aggregate objects.

**Example:**
```java
import java.util.ArrayList;
import java.util.List;

// The Iterator interface
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// The Aggregate interface
interface Container<T> {
    Iterator<T> createIterator();
}

// Concrete implementation
class Book {
    private String title;
    public Book(String title) { this.title = title; }
    public String getTitle() { return title; }
}

class BookCollection implements Container<Book> {
    private List<Book> books = new ArrayList<>();

    public void addBook(Book book) {
        books.add(book);
    }

    @Override
    public Iterator<Book> createIterator() {
        return new BookIterator();
    }

    // Concrete Iterator
    private class BookIterator implements Iterator<Book> {
        private int currentIndex = 0;

        @Override
        public boolean hasNext() {
            return currentIndex < books.size();
        }

        @Override
        public Book next() {
            if (this.hasNext()) {
                return books.get(currentIndex++);
            }
            return null;
        }
    }
}

// Usage:
// BookCollection library = new BookCollection();
// library.addBook(new Book("Design Patterns"));
// library.addBook(new Book("Clean Code"));
// library.addBook(new Book("Effective Java"));
//
// Iterator<Book> iterator = library.createIterator();
// while (iterator.hasNext()) {
//     Book book = iterator.next();
//     System.out.println("Book: " + book.getTitle());
// }
```
