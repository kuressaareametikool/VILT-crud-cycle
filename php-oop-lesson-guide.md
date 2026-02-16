# PHP Object-Oriented Programming Fundamentals for Laravel

## Course Overview

This comprehensive guide covers essential Object-Oriented Programming (OOP) concepts in PHP that form the foundation for Laravel development. You will learn core OOP principles through practical examples and hands-on exercises.

## What You'll Learn

- Define and instantiate classes with properties and methods
- Understand and apply visibility modifiers (public, protected, private)
- Implement constructors for object initialization
- Utilize static methods and properties effectively
- Organize code using namespaces
- Create and use traits for code reusability
- Build a practical mini-project applying all learned concepts

## Prerequisites

- Basic PHP syntax knowledge (variables, functions, arrays)
- PHP 8.0 or higher installed
- A code editor (VS Code, PHPStorm, etc.)
- Basic understanding of web development concepts

## Table of Contents

1. [Classes in PHP](#1-classes-in-php)
2. [Properties and Methods](#2-properties-and-methods)
3. [Visibility Modifiers](#3-visibility-modifiers)
4. [Constructors](#4-constructors)
5. [Static Methods and Properties](#5-static-methods-and-properties)
6. [Namespaces](#6-namespaces)
7. [Traits](#7-traits)
8. [Mini Project: Task Management System](#8-mini-project-task-management-system)

---

## 1. Classes in PHP

### What is a Class?

A class is a blueprint or template for creating objects. It defines the structure and behavior that objects of that type will have. Think of a class as a cookie cutter, and objects as the cookies made from it.

### Basic Class Syntax

```php
<?php

class Car
{
    // Class body goes here
}
```

### Creating Objects (Instantiation)

Objects are instances of a class. You create them using the `new` keyword.

```php
<?php

class Car
{
    // Empty class for now
}

// Creating an instance (object) of the Car class
$myCar = new Car();
$yourCar = new Car();

var_dump($myCar);
var_dump($yourCar);
```

### Key Concepts

- **Class**: The blueprint (e.g., Car)
- **Object**: An instance of a class (e.g., $myCar)
- **Instantiation**: The process of creating an object from a class
- Each object is independent and has its own identity

### Exercise 1.1: Create Your First Class

Create a `Book` class and instantiate three different book objects.

```php
<?php

// Your code here
class Book
{
    
}

$book1 = new Book();
$book2 = new Book();
$book3 = new Book();
```

**Expected Output:** Three separate Book objects created successfully.

---

## 2. Properties and Methods

### Properties (Attributes)

Properties are variables that belong to a class. They represent the state or characteristics of an object.

```php
<?php

class Car
{
    public $brand;
    public $color;
    public $year;
}

$myCar = new Car();
$myCar->brand = "Toyota";
$myCar->color = "Red";
$myCar->year = 2023;

echo $myCar->brand; // Output: Toyota
```

### Methods (Functions)

Methods are functions that belong to a class. They define the behavior or actions an object can perform.

```php
<?php

class Car
{
    public $brand;
    public $color;
    
    public function start()
    {
        return "The car is starting...";
    }
    
    public function getDescription()
    {
        return "This is a {$this->color} {$this->brand}.";
    }
}

$myCar = new Car();
$myCar->brand = "Honda";
$myCar->color = "Blue";

echo $myCar->start(); // Output: The car is starting...
echo $myCar->getDescription(); // Output: This is a Blue Honda.
```

### The $this Keyword

The `$this` keyword refers to the current object instance. It allows you to access properties and methods within the class.

```php
<?php

class BankAccount
{
    public $balance = 0;
    
    public function deposit($amount)
    {
        $this->balance += $amount;
        return "Deposited: $" . $amount;
    }
    
    public function withdraw($amount)
    {
        if ($amount <= $this->balance) {
            $this->balance -= $amount;
            return "Withdrew: $" . $amount;
        }
        return "Insufficient funds";
    }
    
    public function getBalance()
    {
        return $this->balance;
    }
}

$account = new BankAccount();
$account->deposit(100);
$account->deposit(50);
echo $account->getBalance(); // Output: 150
$account->withdraw(30);
echo $account->getBalance(); // Output: 120
```

### Exercise 2.1: Build a Student Class

Create a `Student` class with the following:

- Properties: name, age, grade
- Methods: 
  - introduce() - returns a string introduction
  - study($subject) - returns a string about studying that subject

```php
<?php

// Your code here
class Student
{
    
}
```

**Expected Usage:**

```php
$student = new Student();
$student->name = "Alice";
$student->age = 16;
$student->grade = 10;

echo $student->introduce(); // Hi, I'm Alice, 16 years old, and I'm in grade 10.
echo $student->study("Mathematics"); // Alice is studying Mathematics.
```

---

## 3. Visibility Modifiers

Visibility modifiers control access to properties and methods from outside the class. PHP has three visibility levels: public, protected, and private.

### Visibility Types Overview

| Modifier | Accessible in Class | Accessible in Subclass | Accessible Outside Class |
|----------|---------------------|------------------------|--------------------------|
| public | Yes | Yes | Yes |
| protected | Yes | Yes | No |
| private | Yes | No | No |

### Public Visibility

Public members are accessible from anywhere: inside the class, in subclasses, and outside the class.

```php
<?php

class User
{
    public $username = "john_doe";
    
    public function getUsername()
    {
        return $this->username;
    }
}

$user = new User();
echo $user->username; // Accessible directly: john_doe
echo $user->getUsername(); // Also accessible: john_doe
```

### Protected Visibility

Protected members are accessible only within the class and its subclasses (inherited classes). They cannot be accessed from outside.

```php
<?php

class Vehicle
{
    protected $engineStatus = "off";
    
    public function startEngine()
    {
        $this->engineStatus = "on";
        return "Engine started";
    }
    
    public function getEngineStatus()
    {
        return $this->engineStatus;
    }
}

$vehicle = new Vehicle();
// echo $vehicle->engineStatus; // ERROR: Cannot access protected property
echo $vehicle->getEngineStatus(); // Works: returns "off"
$vehicle->startEngine();
echo $vehicle->getEngineStatus(); // Works: returns "on"
```

### Private Visibility

Private members are accessible only within the class itself, not even in subclasses.

```php
<?php

class BankAccount
{
    private $accountNumber;
    private $balance = 0;
    
    public function setAccountNumber($number)
    {
        $this->accountNumber = $number;
    }
    
    public function deposit($amount)
    {
        if ($amount > 0) {
            $this->balance += $amount;
            return true;
        }
        return false;
    }
    
    public function getBalance()
    {
        return $this->balance;
    }
    
    private function logTransaction($type, $amount)
    {
        // Internal method not accessible outside
        echo "Transaction: {$type} - ${$amount}\n";
    }
}

$account = new BankAccount();
$account->setAccountNumber("ACC123456");
// $account->balance = 1000000; // ERROR: Cannot access private property
$account->deposit(500); // Correct way
echo $account->getBalance(); // Output: 500
// $account->logTransaction("deposit", 500); // ERROR: Cannot access private method
```

### Why Use Visibility Modifiers?

1. **Encapsulation**: Hide internal implementation details
2. **Data Protection**: Prevent unauthorized or invalid access to sensitive data
3. **Maintainability**: Change internal implementation without breaking external code
4. **Security**: Control how data is modified and accessed

### Best Practices

- Make properties `private` or `protected` by default
- Provide `public` methods (getters/setters) for controlled access
- Use `public` only when direct access is intentionally needed
- This is called "encapsulation" and is a core OOP principle

### Exercise 3.1: Build a Secure Product Class

Create a `Product` class with:

- Private properties: name, price, stock
- Public methods:
  - setName($name), setPrice($price), setStock($stock)
  - getName(), getPrice(), getStock()
  - addStock($quantity) - adds to existing stock
  - purchase($quantity) - reduces stock only if enough items are available

```php
<?php

// Your code here
class Product
{
    
}
```

**Expected Usage:**

```php
$product = new Product();
$product->setName("Laptop");
$product->setPrice(999.99);
$product->setStock(10);

echo $product->getName(); // Laptop
$product->addStock(5);
echo $product->getStock(); // 15
$product->purchase(3);
echo $product->getStock(); // 12
```

---

## 4. Constructors

### What is a Constructor?

A constructor is a special method that is automatically called when an object is created. It is used to initialize object properties and perform setup tasks.

### Basic Constructor Syntax

The constructor method is named `__construct()` (with two underscores).

```php
<?php

class User
{
    public $name;
    public $email;
    
    public function __construct($name, $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
}

$user = new User("John Doe", "john@example.com");
echo $user->name; // Output: John Doe
echo $user->email; // Output: john@example.com
```

### Constructor with Default Values

You can provide default values for constructor parameters.

```php
<?php

class Product
{
    public $name;
    public $price;
    public $inStock;
    
    public function __construct($name, $price, $inStock = true)
    {
        $this->name = $name;
        $this->price = $price;
        $this->inStock = $inStock;
    }
}

$product1 = new Product("Laptop", 999.99);
$product2 = new Product("Phone", 599.99, false);

var_dump($product1->inStock); // true
var_dump($product2->inStock); // false
```

### PHP 8 Constructor Property Promotion

PHP 8 introduced a shorthand syntax for declaring and initializing properties directly in the constructor.

```php
<?php

// Traditional way (PHP 7 and earlier)
class UserOld
{
    public $name;
    public $email;
    
    public function __construct($name, $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
}

// PHP 8 way (Constructor Property Promotion)
class UserNew
{
    public function __construct(
        public string $name,
        public string $email,
        private int $age = 18
    ) {}
}

$user = new UserNew("Jane", "jane@example.com", 25);
echo $user->name; // Output: Jane
echo $user->email; // Output: jane@example.com
// echo $user->age; // ERROR: Cannot access private property
```

### Validation in Constructors

Constructors are perfect places to validate data before creating an object.

```php
<?php

class BankAccount
{
    private $accountNumber;
    private $balance;
    
    public function __construct($accountNumber, $initialBalance)
    {
        if ($initialBalance < 0) {
            throw new Exception("Initial balance cannot be negative");
        }
        
        if (empty($accountNumber)) {
            throw new Exception("Account number is required");
        }
        
        $this->accountNumber = $accountNumber;
        $this->balance = $initialBalance;
    }
    
    public function getBalance()
    {
        return $this->balance;
    }
    
    public function getAccountNumber()
    {
        return $this->accountNumber;
    }
}

$account = new BankAccount("ACC001", 1000);
echo $account->getBalance(); // Output: 1000

// This will throw an exception:
// $invalidAccount = new BankAccount("ACC002", -500);
```

### Exercise 4.1: Create a Book Class with Constructor

Create a `Book` class with:

- Private properties: title, author, year, price
- Constructor that initializes all properties
- Validation:
  - year should be between 1000 and current year
  - price should be positive
- Method: getInfo() - returns formatted book information

```php
<?php

// Your code here
class Book
{
    
}
```

**Expected Usage:**

```php
$book = new Book("Clean Code", "Robert C. Martin", 2008, 39.99);
echo $book->getInfo(); // Clean Code by Robert C. Martin (2008) - $39.99

// This should throw an exception:
// $invalidBook = new Book("Old Book", "Author", 500, 10);
```

---

## 5. Static Methods and Properties

### What are Static Members?

Static methods and properties belong to the class itself rather than to instances of the class. They can be accessed without creating an object.

### Static Properties

Static properties are shared across all instances of a class.

```php
<?php

class Counter
{
    public static $count = 0;
    public $instanceId;
    
    public function __construct()
    {
        self::$count++;
        $this->instanceId = self::$count;
    }
    
    public static function getCount()
    {
        return self::$count;
    }
}

$obj1 = new Counter();
$obj2 = new Counter();
$obj3 = new Counter();

echo Counter::$count; // Output: 3
echo Counter::getCount(); // Output: 3
echo $obj1->instanceId; // Output: 1
echo $obj2->instanceId; // Output: 2
```

### Static Methods

Static methods can be called without creating an instance of the class.

```php
<?php

class MathHelper
{
    public static function add($a, $b)
    {
        return $a + $b;
    }
    
    public static function multiply($a, $b)
    {
        return $a * $b;
    }
    
    public static function calculateCircleArea($radius)
    {
        return pi() * $radius * $radius;
    }
}

// No need to instantiate the class
echo MathHelper::add(5, 3); // Output: 8
echo MathHelper::multiply(4, 7); // Output: 28
echo MathHelper::calculateCircleArea(5); // Output: 78.539816339745
```

### Accessing Static Members

- Use `self::` to access static members within the same class
- Use `ClassName::` to access static members outside the class
- Use `::` operator (scope resolution operator)

```php
<?php

class Config
{
    public static $appName = "My Application";
    private static $apiKey = "secret_key_123";
    
    public static function getAppName()
    {
        return self::$appName;
    }
    
    public static function getApiKey()
    {
        return self::$apiKey;
    }
    
    public static function setAppName($name)
    {
        self::$appName = $name;
    }
}

echo Config::$appName; // Direct access: My Application
echo Config::getAppName(); // Via method: My Application
echo Config::getApiKey(); // Access private property via method

Config::setAppName("New App Name");
echo Config::getAppName(); // Output: New App Name

// echo Config::$apiKey; // ERROR: Cannot access private property
```

### Practical Example: Singleton Pattern

The singleton pattern ensures only one instance of a class exists.

```php
<?php

class Database
{
    private static $instance = null;
    private static $connectionCount = 0;
    private $connection;
    
    // Private constructor prevents direct instantiation
    private function __construct()
    {
        self::$connectionCount++;
        $this->connection = "Connected to database";
    }
    
    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    public static function getConnectionCount()
    {
        return self::$connectionCount;
    }
    
    public function query($sql)
    {
        return "Executing: " . $sql;
    }
}

$db1 = Database::getInstance();
$db2 = Database::getInstance();
$db3 = Database::getInstance();

echo Database::getConnectionCount(); // Output: 1 (only one instance created)
echo $db1->query("SELECT * FROM users"); // Executing: SELECT * FROM users

// This won't work (private constructor):
// $db = new Database();
```

### When to Use Static Members

**Use static when:**
- The method doesn't depend on instance state (utility functions)
- You need to share data across all instances
- Implementing design patterns like Singleton or Factory

**Avoid static when:**
- The method needs access to instance properties
- You want testability and dependency injection (important in Laravel)
- The behavior varies between instances

### Exercise 5.1: Build a Validator Class

Create a `Validator` class with static methods:

- validateEmail($email) - returns true if valid email format
- validateAge($age) - returns true if age is between 0 and 120
- validatePassword($password) - returns true if password is at least 8 characters
- Keep track of total validations performed using a static property

```php
<?php

// Your code here
class Validator
{
    
}
```

**Expected Usage:**

```php
echo Validator::validateEmail("test@example.com"); // true
echo Validator::validateEmail("invalid-email"); // false
echo Validator::validateAge(25); // true
echo Validator::validateAge(150); // false
echo Validator::validatePassword("secret123"); // true
echo Validator::validatePassword("short"); // false
echo Validator::getTotalValidations(); // 6
```

---

## 6. Namespaces

### What are Namespaces?

Namespaces are a way to organize and group related classes, interfaces, functions, and constants. They prevent naming conflicts and make code more maintainable.

Think of namespaces like folders on your computer - they organize files and prevent two files from having the same name in different directories.

### Why Use Namespaces?

1. **Avoid naming conflicts**: Two classes can have the same name if they're in different namespaces
2. **Organization**: Group related code together
3. **Readability**: Clear code structure
4. **Laravel requirement**: Laravel heavily uses namespaces

### Basic Namespace Syntax

```php
<?php

namespace App\Models;

class User
{
    public $name;
    
    public function __construct($name)
    {
        $this->name = $name;
    }
}
```

### Using Namespaced Classes

```php
<?php

// File: User.php
namespace App\Models;

class User
{
    public $name;
}

// File: index.php
// Option 1: Use fully qualified name
$user = new \App\Models\User();

// Option 2: Import with 'use' statement
use App\Models\User;

$user = new User();
$user->name = "John";
```

### Multiple Classes in Same Namespace

```php
<?php

// File: Models/User.php
namespace App\Models;

class User
{
    public $name;
    public $email;
}

// File: Models/Product.php
namespace App\Models;

class Product
{
    public $name;
    public $price;
}

// File: index.php
use App\Models\User;
use App\Models\Product;

$user = new User();
$product = new Product();
```

### Aliasing Namespaces

When two classes have the same name, use aliases to differentiate them.

```php
<?php

namespace App\Models;

class User
{
    public $type = "App User";
}

namespace Admin\Models;

class User
{
    public $type = "Admin User";
}

// Using both in the same file
use App\Models\User as AppUser;
use Admin\Models\User as AdminUser;

$appUser = new AppUser();
$adminUser = new AdminUser();

echo $appUser->type; // Output: App User
echo $adminUser->type; // Output: Admin User
```

### Nested Namespaces

Namespaces can be nested to create deeper organizational structures.

```php
<?php

namespace App\Services\Payment;

class PayPalService
{
    public function processPayment($amount)
    {
        return "Processing ${$amount} via PayPal";
    }
}

namespace App\Services\Email;

class EmailService
{
    public function send($to, $subject)
    {
        return "Sending email to {$to}: {$subject}";
    }
}

// Using them
use App\Services\Payment\PayPalService;
use App\Services\Email\EmailService;

$payment = new PayPalService();
$email = new EmailService();

echo $payment->processPayment(100);
echo $email->send("user@example.com", "Welcome");
```

### Practical Example: Laravel-Style Structure

```php
<?php

// File: app/Models/User.php
namespace App\Models;

class User
{
    private $name;
    private $email;
    
    public function __construct($name, $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
    
    public function getName()
    {
        return $this->name;
    }
}

// File: app/Controllers/UserController.php
namespace App\Controllers;

use App\Models\User;

class UserController
{
    public function createUser($name, $email)
    {
        $user = new User($name, $email);
        return "User created: " . $user->getName();
    }
}

// File: public/index.php
use App\Controllers\UserController;

$controller = new UserController();
echo $controller->createUser("Alice", "alice@example.com");
```

### Global Namespace

Classes without a namespace belong to the global namespace. You can access them using a leading backslash.

```php
<?php

namespace App\Models;

// This DateTime is in the global namespace
$date = new \DateTime();

// This would look for App\Models\DateTime (which doesn't exist)
// $date = new DateTime(); // ERROR

echo $date->format('Y-m-d');
```

### Exercise 6.1: Create a Namespaced Project Structure

Create the following namespaced classes:

1. `App\Models\Article` with properties: title, content, author
2. `App\Services\ArticleService` with method: publish($article)
3. Use these classes together in a main file

```php
<?php

// File 1: Models/Article.php
namespace App\Models;

class Article
{
    // Your code here
}

// File 2: Services/ArticleService.php
namespace App\Services;

// Your code here

// File 3: index.php
// Your code here - import and use both classes
```

**Expected Output:**

```php
// Create an article and publish it
// Output: Publishing article: "Understanding PHP OOP" by John Doe
```

---

## 7. Traits

### What are Traits?

Traits are a mechanism for code reuse in PHP. They allow you to reuse methods across multiple classes without using inheritance. PHP only supports single inheritance, but traits let you compose behavior from multiple sources.

### Why Use Traits?

1. **Code reuse**: Share methods across unrelated classes
2. **Avoid inheritance issues**: Don't need to create complex inheritance hierarchies
3. **Horizontal reuse**: Add functionality to classes without changing their parent class
4. **Laravel uses them**: Many Laravel features use traits (e.g., `HasFactory`, `Notifiable`)

### Basic Trait Syntax

```php
<?php

trait Loggable
{
    public function log($message)
    {
        echo "[LOG] " . date('Y-m-d H:i:s') . " - {$message}\n";
    }
}

class User
{
    use Loggable;
    
    public $name;
    
    public function __construct($name)
    {
        $this->name = $name;
        $this->log("User {$name} created");
    }
}

class Product
{
    use Loggable;
    
    public $name;
    
    public function __construct($name)
    {
        $this->name = $name;
        $this->log("Product {$name} created");
    }
}

$user = new User("Alice");
// Output: [LOG] 2024-02-16 10:30:00 - User Alice created

$product = new Product("Laptop");
// Output: [LOG] 2024-02-16 10:30:01 - Product Laptop created
```

### Using Multiple Traits

A class can use multiple traits by listing them separated by commas.

```php
<?php

trait Timestampable
{
    public $createdAt;
    public $updatedAt;
    
    public function setTimestamps()
    {
        $this->createdAt = date('Y-m-d H:i:s');
        $this->updatedAt = date('Y-m-d H:i:s');
    }
    
    public function updateTimestamp()
    {
        $this->updatedAt = date('Y-m-d H:i:s');
    }
}

trait Sluggable
{
    public function generateSlug($text)
    {
        $slug = strtolower($text);
        $slug = preg_replace('/[^a-z0-9]+/', '-', $slug);
        $slug = trim($slug, '-');
        return $slug;
    }
}

class Article
{
    use Timestampable, Sluggable;
    
    public $title;
    public $slug;
    
    public function __construct($title)
    {
        $this->title = $title;
        $this->slug = $this->generateSlug($title);
        $this->setTimestamps();
    }
}

$article = new Article("Understanding PHP Traits");
echo $article->slug; // Output: understanding-php-traits
echo $article->createdAt; // Output: 2024-02-16 10:30:00
```

### Trait with Properties

Traits can contain properties, but be careful about conflicts.

```php
<?php

trait HasStatus
{
    public $status = 'active';
    
    public function activate()
    {
        $this->status = 'active';
    }
    
    public function deactivate()
    {
        $this->status = 'inactive';
    }
    
    public function isActive()
    {
        return $this->status === 'active';
    }
}

class User
{
    use HasStatus;
    
    public $name;
    
    public function __construct($name)
    {
        $this->name = $name;
    }
}

$user = new User("Bob");
echo $user->isActive(); // Output: 1 (true)
$user->deactivate();
echo $user->isActive(); // Output: (false)
echo $user->status; // Output: inactive
```

### Resolving Trait Conflicts

When using multiple traits with methods of the same name, you must resolve conflicts.

```php
<?php

trait Logger
{
    public function log($message)
    {
        echo "[Logger] {$message}\n";
    }
}

trait Debugger
{
    public function log($message)
    {
        echo "[Debugger] {$message}\n";
    }
}

class Application
{
    use Logger, Debugger {
        Logger::log insteadof Debugger;
        Debugger::log as debugLog;
    }
}

$app = new Application();
$app->log("This is a message"); // Output: [Logger] This is a message
$app->debugLog("This is a debug message"); // Output: [Debugger] This is a debug message
```

### Traits with Abstract Methods

Traits can require the using class to implement certain methods.

```php
<?php

trait Validatable
{
    abstract protected function getRules();
    
    public function validate($data)
    {
        $rules = $this->getRules();
        $errors = [];
        
        foreach ($rules as $field => $rule) {
            if ($rule === 'required' && empty($data[$field])) {
                $errors[] = "{$field} is required";
            }
        }
        
        return empty($errors) ? true : $errors;
    }
}

class User
{
    use Validatable;
    
    protected function getRules()
    {
        return [
            'name' => 'required',
            'email' => 'required'
        ];
    }
}

$user = new User();
$result = $user->validate(['name' => 'John']);
print_r($result); // Output: Array ( [0] => email is required )
```

### Practical Example: Laravel-Style Traits

```php
<?php

namespace App\Traits;

trait HasUuid
{
    public function generateUuid()
    {
        return sprintf(
            '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0xffff)
        );
    }
}

trait Searchable
{
    public function search($items, $query)
    {
        return array_filter($items, function($item) use ($query) {
            return stripos($item, $query) !== false;
        });
    }
}

namespace App\Models;

use App\Traits\HasUuid;
use App\Traits\Searchable;

class User
{
    use HasUuid, Searchable;
    
    public $id;
    public $name;
    
    public function __construct($name)
    {
        $this->id = $this->generateUuid();
        $this->name = $name;
    }
}

$user = new User("Alice");
echo $user->id; // Output: a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d (example)

$users = ["Alice Johnson", "Bob Smith", "Charlie Brown"];
$results = $user->search($users, "bob");
print_r($results); // Output: Array ( [1] => Bob Smith )
```

### Exercise 7.1: Create Reusable Traits

Create the following traits and use them in classes:

1. `Timestampable` trait with:
   - Properties: createdAt, updatedAt
   - Methods: touch() - updates the updatedAt timestamp

2. `Sortable` trait with:
   - Method: sortBy($array, $key) - sorts an array by a specific key

3. Create a `Post` class that uses both traits

```php
<?php

// Your code here
trait Timestampable
{
    
}

trait Sortable
{
    
}

class Post
{
    
}
```

**Expected Usage:**

```php
$post = new Post("My First Post");
$post->touch();
echo $post->updatedAt; // Current timestamp

$posts = [
    ['title' => 'Post C', 'views' => 100],
    ['title' => 'Post A', 'views' => 500],
    ['title' => 'Post B', 'views' => 250]
];

$sorted = $post->sortBy($posts, 'views');
print_r($sorted); // Sorted by views
```

---

## 8. Mini Project: Task Management System

Now it's time to put everything together. You'll build a simple task management system that demonstrates all the OOP concepts covered in this lesson.

### Project Overview

Build a task management system with the following features:
- Create tasks with title, description, and status
- Assign tasks to users
- Mark tasks as complete
- List all tasks and filter by status
- Track creation and update timestamps

### Project Structure

```
task-manager/
├── src/
│   ├── Models/
│   │   ├── Task.php
│   │   └── User.php
│   ├── Traits/
│   │   ├── Timestampable.php
│   │   └── HasStatus.php
│   ├── Services/
│   │   └── TaskManager.php
│   └── Helpers/
│       └── Validator.php
├── public/
│   └── index.php
└── README.md
```

### Step 1: Create the Traits

**File: src/Traits/Timestampable.php**

```php
<?php

namespace App\Traits;

trait Timestampable
{
    private $createdAt;
    private $updatedAt;
    
    public function initializeTimestamps()
    {
        $this->createdAt = date('Y-m-d H:i:s');
        $this->updatedAt = date('Y-m-d H:i:s');
    }
    
    public function touch()
    {
        $this->updatedAt = date('Y-m-d H:i:s');
    }
    
    public function getCreatedAt()
    {
        return $this->createdAt;
    }
    
    public function getUpdatedAt()
    {
        return $this->updatedAt;
    }
}
```

**File: src/Traits/HasStatus.php**

```php
<?php

namespace App\Traits;

trait HasStatus
{
    private $status = 'pending';
    
    public function setStatus($status)
    {
        $allowedStatuses = ['pending', 'in_progress', 'completed'];
        
        if (in_array($status, $allowedStatuses)) {
            $this->status = $status;
            if (method_exists($this, 'touch')) {
                $this->touch();
            }
            return true;
        }
        
        return false;
    }
    
    public function getStatus()
    {
        return $this->status;
    }
    
    public function markAsCompleted()
    {
        return $this->setStatus('completed');
    }
    
    public function isCompleted()
    {
        return $this->status === 'completed';
    }
}
```

### Step 2: Create the Models

**File: src/Models/User.php**

```php
<?php

namespace App\Models;

class User
{
    private $id;
    private $name;
    private $email;
    private static $userCount = 0;
    
    public function __construct($name, $email)
    {
        self::$userCount++;
        $this->id = self::$userCount;
        $this->name = $name;
        $this->email = $email;
    }
    
    public function getId()
    {
        return $this->id;
    }
    
    public function getName()
    {
        return $this->name;
    }
    
    public function getEmail()
    {
        return $this->email;
    }
    
    public static function getTotalUsers()
    {
        return self::$userCount;
    }
}
```

**File: src/Models/Task.php**

```php
<?php

namespace App\Models;

use App\Traits\Timestampable;
use App\Traits\HasStatus;

class Task
{
    use Timestampable, HasStatus;
    
    private $id;
    private $title;
    private $description;
    private $assignedUser;
    private static $taskCount = 0;
    
    public function __construct($title, $description)
    {
        self::$taskCount++;
        $this->id = self::$taskCount;
        $this->title = $title;
        $this->description = $description;
        $this->initializeTimestamps();
    }
    
    public function getId()
    {
        return $this->id;
    }
    
    public function getTitle()
    {
        return $this->title;
    }
    
    public function getDescription()
    {
        return $this->description;
    }
    
    public function assignTo(User $user)
    {
        $this->assignedUser = $user;
        $this->touch();
    }
    
    public function getAssignedUser()
    {
        return $this->assignedUser;
    }
    
    public function getInfo()
    {
        $assignedTo = $this->assignedUser 
            ? $this->assignedUser->getName() 
            : "Unassigned";
            
        return sprintf(
            "[Task #%d] %s - Status: %s - Assigned to: %s",
            $this->id,
            $this->title,
            $this->status,
            $assignedTo
        );
    }
    
    public static function getTotalTasks()
    {
        return self::$taskCount;
    }
}
```

### Step 3: Create the Helper Classes

**File: src/Helpers/Validator.php**

```php
<?php

namespace App\Helpers;

class Validator
{
    private static $validationCount = 0;
    
    public static function validateEmail($email)
    {
        self::$validationCount++;
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
    
    public static function validateNotEmpty($value)
    {
        self::$validationCount++;
        return !empty(trim($value));
    }
    
    public static function getValidationCount()
    {
        return self::$validationCount;
    }
}
```

### Step 4: Create the Service Layer

**File: src/Services/TaskManager.php**

```php
<?php

namespace App\Services;

use App\Models\Task;
use App\Models\User;

class TaskManager
{
    private $tasks = [];
    private $users = [];
    
    public function createTask($title, $description)
    {
        $task = new Task($title, $description);
        $this->tasks[] = $task;
        return $task;
    }
    
    public function createUser($name, $email)
    {
        $user = new User($name, $email);
        $this->users[] = $user;
        return $user;
    }
    
    public function assignTask($taskId, $userId)
    {
        $task = $this->findTaskById($taskId);
        $user = $this->findUserById($userId);
        
        if ($task && $user) {
            $task->assignTo($user);
            return true;
        }
        
        return false;
    }
    
    public function completeTask($taskId)
    {
        $task = $this->findTaskById($taskId);
        
        if ($task) {
            return $task->markAsCompleted();
        }
        
        return false;
    }
    
    public function getAllTasks()
    {
        return $this->tasks;
    }
    
    public function getTasksByStatus($status)
    {
        return array_filter($this->tasks, function($task) use ($status) {
            return $task->getStatus() === $status;
        });
    }
    
    public function getAllUsers()
    {
        return $this->users;
    }
    
    private function findTaskById($id)
    {
        foreach ($this->tasks as $task) {
            if ($task->getId() === $id) {
                return $task;
            }
        }
        return null;
    }
    
    private function findUserById($id)
    {
        foreach ($this->users as $user) {
            if ($user->getId() === $id) {
                return $user;
            }
        }
        return null;
    }
    
    public function displayAllTasks()
    {
        echo "\n=== All Tasks ===\n";
        foreach ($this->tasks as $task) {
            echo $task->getInfo() . "\n";
        }
        echo "Total Tasks: " . Task::getTotalTasks() . "\n";
    }
}
```

### Step 5: Create the Main Application

**File: public/index.php**

```php
<?php

// In a real application, you would use Composer's autoloader
// For this example, we'll manually include files

spl_autoload_register(function ($class) {
    $prefix = 'App\\';
    $baseDir = __DIR__ . '/../src/';
    
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        return;
    }
    
    $relativeClass = substr($class, $len);
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
    
    if (file_exists($file)) {
        require $file;
    }
});

use App\Services\TaskManager;
use App\Helpers\Validator;

echo "=== Task Management System ===\n\n";

// Initialize the task manager
$taskManager = new TaskManager();

// Create users
echo "Creating users...\n";
$user1 = $taskManager->createUser("Alice Johnson", "alice@example.com");
$user2 = $taskManager->createUser("Bob Smith", "bob@example.com");

if (Validator::validateEmail($user1->getEmail())) {
    echo "User 1 email is valid\n";
}

if (Validator::validateEmail($user2->getEmail())) {
    echo "User 2 email is valid\n";
}

echo "Total users created: " . count($taskManager->getAllUsers()) . "\n\n";

// Create tasks
echo "Creating tasks...\n";
$task1 = $taskManager->createTask(
    "Implement login system",
    "Create a secure login system with password hashing"
);

$task2 = $taskManager->createTask(
    "Design database schema",
    "Design the database schema for the application"
);

$task3 = $taskManager->createTask(
    "Write documentation",
    "Write comprehensive documentation for the API"
);

echo "Total tasks created: " . count($taskManager->getAllTasks()) . "\n\n";

// Assign tasks to users
echo "Assigning tasks...\n";
$taskManager->assignTask(1, 1); // Assign task 1 to user 1
$taskManager->assignTask(2, 2); // Assign task 2 to user 2
$taskManager->assignTask(3, 1); // Assign task 3 to user 1

// Update task statuses
echo "Updating task statuses...\n";
$task1->setStatus('in_progress');
$taskManager->completeTask(2); // Complete task 2

// Display all tasks
$taskManager->displayAllTasks();

// Display tasks by status
echo "\n=== Pending Tasks ===\n";
$pendingTasks = $taskManager->getTasksByStatus('pending');
foreach ($pendingTasks as $task) {
    echo $task->getInfo() . "\n";
}

echo "\n=== Completed Tasks ===\n";
$completedTasks = $taskManager->getTasksByStatus('completed');
foreach ($completedTasks as $task) {
    echo $task->getInfo() . "\n";
}

// Show statistics
echo "\n=== Statistics ===\n";
echo "Total validations performed: " . Validator::getValidationCount() . "\n";
echo "Total tasks in system: " . $taskManager->getAllTasks()[0]::getTotalTasks() . "\n";
echo "Total users in system: " . $taskManager->getAllUsers()[0]::getTotalUsers() . "\n";
```

### Step 6: Run the Project

To run this project:

1. Create the folder structure as shown above
2. Create all the files with the provided code
3. Run from the command line:

```bash
php public/index.php
```

### Expected Output

```
=== Task Management System ===

Creating users...
User 1 email is valid
User 2 email is valid
Total users created: 2

Creating tasks...
Total tasks created: 3

Assigning tasks...
Updating task statuses...

=== All Tasks ===
[Task #1] Implement login system - Status: in_progress - Assigned to: Alice Johnson
[Task #2] Design database schema - Status: completed - Assigned to: Bob Smith
[Task #3] Write documentation - Status: pending - Assigned to: Alice Johnson
Total Tasks: 3

=== Pending Tasks ===
[Task #3] Write documentation - Status: pending - Assigned to: Alice Johnson

=== Completed Tasks ===
[Task #2] Design database schema - Status: completed - Assigned to: Bob Smith

=== Statistics ===
Total validations performed: 2
Total tasks in system: 3
Total users in system: 2
```

### Project Exercises

Now that you have a working task management system, try extending it:

1. Add a priority system (high, medium, low) to tasks
2. Implement a due date feature with validation
3. Create a method to reassign tasks from one user to another
4. Add a method to delete tasks
5. Implement task filtering by assigned user
6. Add a description character limit validation
7. Create a method to display tasks sorted by creation date
8. Add a task category system

### What You've Learned

In this mini project, you've applied:

- **Classes**: Task, User, TaskManager, Validator
- **Properties and Methods**: Private properties with public getters/setters
- **Visibility**: Public, private, and protected modifiers
- **Constructors**: Initializing objects with validation
- **Static Members**: Counting instances, utility methods
- **Namespaces**: Organizing code into logical groups
- **Traits**: Reusing Timestampable and HasStatus across classes
- **Best Practices**: Encapsulation, separation of concerns, single responsibility

### Next Steps

To prepare for Laravel, you should also learn about:

1. **Inheritance**: Extending classes and method overriding
2. **Interfaces**: Defining contracts for classes
3. **Abstract Classes**: Creating base classes that cannot be instantiated
4. **Type Hinting**: Declaring parameter and return types
5. **Exceptions**: Proper error handling
6. **Composer**: PHP dependency management (used by Laravel)
7. **PSR Standards**: PHP coding standards (PSR-4 autoloading, PSR-12 style)

---

## Additional Resources

### Official Documentation
- [PHP Official Documentation - Classes and Objects](https://www.php.net/manual/en/language.oop5.php)
- [Laravel Documentation](https://laravel.com/docs)

### Recommended Reading
- PHP The Right Way (phptherightway.com)
- Laravel Best Practices
- SOLID Principles in PHP

### Practice Platforms
- LeetCode (PHP section)
- HackerRank (PHP track)
- Exercism (PHP exercises)

---

## Summary

Congratulations on completing this comprehensive OOP fundamentals lesson! You've learned:

1. **Classes and Objects**: Creating blueprints and instances
2. **Properties and Methods**: Defining state and behavior
3. **Visibility Modifiers**: Controlling access with public, protected, and private
4. **Constructors**: Initializing objects properly
5. **Static Members**: Class-level properties and methods
6. **Namespaces**: Organizing and preventing naming conflicts
7. **Traits**: Reusing code across multiple classes
8. **Real-World Application**: Building a complete task management system

These concepts are fundamental to Laravel development. Laravel extensively uses OOP principles, namespaces, traits, and follows modern PHP best practices. With this foundation, you're well-prepared to dive into Laravel and understand how its elegant architecture works under the hood.

Keep practicing, and happy coding!
