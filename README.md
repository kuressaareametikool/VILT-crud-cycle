# Laravel Request Lifecycle - VILT Stack Study Guide

## Overview
This guide explains how data flows through a Laravel application using the VILT stack (Vue, Inertia, Laravel, Tailwind). We'll follow a request from the browser all the way to the database and back, with detailed explanations at each step.

---

## The Complete Data Flow

```
Browser Request → Routes → Middleware → Controller → Model → Database
                                                        ↓
Browser ← Inertia Response ← Controller ← Query Results
                    ↓
              Vue Component
```

---

## 1. Migrations: Building Your Database Structure

### What Are Migrations?

**Migrations** are like version control for your database. They allow you to:
- Define table structures in PHP code (not SQL)
- Share database schemas with your team
- Track changes over time
- Easily rollback changes if needed

Think of migrations as blueprints for your database. Instead of manually creating tables in phpMyAdmin or MySQL Workbench, you write PHP code that Laravel executes.

### Creating a Migration

```bash
php artisan make:migration create_posts_table
```

**What happens:**
1. Laravel generates a file in `database/migrations/` 
2. The filename includes a timestamp: `2024_01_01_120000_create_posts_table.php`
3. The timestamp ensures migrations run in order
4. Laravel opens the file and adds two methods: `up()` and `down()`

### Anatomy of a Migration

```php
// database/migrations/2024_01_01_create_posts_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     * This method executes when you run: php artisan migrate
     */
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            // Primary key - auto-incrementing ID
            $table->id(); // Creates: id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
            
            // String column - max 255 characters by default
            $table->string('title'); // Creates: title VARCHAR(255) NOT NULL
            
            // Text column - for longer content
            $table->text('content'); // Creates: content TEXT NOT NULL
            
            // String with custom length
            $table->string('author', 100); // Creates: author VARCHAR(100) NOT NULL
            
            // Boolean with default value
            $table->boolean('published')->default(false); 
            // Creates: published TINYINT(1) NOT NULL DEFAULT 0
            
            // Automatically adds created_at and updated_at timestamp columns
            $table->timestamps(); 
            // Creates: created_at TIMESTAMP NULL, updated_at TIMESTAMP NULL
        });
    }

    /**
     * Reverse the migrations.
     * This method executes when you run: php artisan migrate:rollback
     */
    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### Common Column Types

```php
// Numeric types
$table->integer('views');              // INT
$table->bigInteger('large_number');    // BIGINT
$table->decimal('price', 8, 2);        // DECIMAL(8,2) - 8 digits, 2 after decimal

// String types
$table->string('name');                // VARCHAR(255)
$table->string('code', 10);            // VARCHAR(10)
$table->text('description');           // TEXT

// Date and time
$table->date('birth_date');            // DATE
$table->dateTime('published_at');      // DATETIME
$table->timestamp('created_at');       // TIMESTAMP

// Special types
$table->boolean('is_active');          // TINYINT(1)
$table->json('metadata');              // JSON

// Foreign keys (relationships)
$table->foreignId('user_id')           // Creates BIGINT UNSIGNED
      ->constrained()                  // Adds foreign key to 'users' table
      ->onDelete('cascade');           // Delete posts when user is deleted
```

### Column Modifiers

```php
// Make column nullable (allows NULL values)
$table->string('subtitle')->nullable();

// Set default value
$table->integer('views')->default(0);

// Make column unique (no duplicates)
$table->string('email')->unique();

// Combine modifiers
$table->string('nickname')->nullable()->unique()->default('anonymous');
```

### Running Migrations

```bash
# Run all pending migrations
php artisan migrate

# Rollback the last batch
php artisan migrate:rollback

# See migration status
php artisan migrate:status
```

**What happens when you run `php artisan migrate`:**
1. Laravel checks the `migrations` table in your database
2. It compares which migrations have already run
3. It executes only the new migrations in order
4. Each migration's filename is recorded in the `migrations` table
5. Your database structure is updated

---

## 2. Routes: The Entry Point

### What Are Routes?

**Routes** are the mapping between URLs and your application code. When a user visits a URL, Laravel checks the route files to determine what code should run.

Think of routes as the reception desk of a building - they direct visitors to the right office (controller method).

### How Route Matching Works

When a request comes in (e.g., `GET /posts`):
1. Laravel loads `routes/web.php`
2. It goes through each route definition from top to bottom
3. It checks if the HTTP method (GET, POST, etc.) matches
4. It checks if the URL pattern matches
5. When it finds a match, it stops and executes that route
6. If no match is found, it returns a 404 error

### Basic Route Definitions

```php
// routes/web.php
use App\Http\Controllers\PostController;

// GET request to /posts → calls index method
Route::get('/posts', [PostController::class, 'index'])->name('posts.index');

// POST request to /posts → calls store method
Route::post('/posts', [PostController::class, 'store'])->name('posts.store');

// GET request to /posts/5 → calls show method with id=5
Route::get('/posts/{post}', [PostController::class, 'show'])->name('posts.show');

// PUT request to /posts/5 → calls update method with id=5
Route::put('/posts/{post}', [PostController::class, 'update'])->name('posts.update');

// DELETE request to /posts/5 → calls destroy method with id=5
Route::delete('/posts/{post}', [PostController::class, 'destroy'])->name('posts.destroy');
```

### Understanding Route Parameters

```php
// Simple parameter
Route::get('/posts/{id}', function ($id) {
    return "Post ID: " . $id;
});
// /posts/1 → "Post ID: 1"

// Multiple parameters
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    return "Post: $postId, Comment: $commentId";
});

// Optional parameters
Route::get('/users/{name?}', function ($name = 'Guest') {
    return "Hello, " . $name;
});

// Parameter constraints
Route::get('/posts/{id}', function ($id) {
    return "Post: " . $id;
})->where('id', '[0-9]+'); // Only matches if id is numeric
```

### Route Model Binding (Automatic Model Loading)

**Instead of manually finding:**
```php
Route::get('/posts/{id}', function ($id) {
    $post = Post::findOrFail($id); // Manually find the post
    return view('post', ['post' => $post]);
});
```

**Laravel does it automatically:**
```php
Route::get('/posts/{post}', function (Post $post) {
    // Laravel automatically finds the post by ID!
    return view('post', ['post' => $post]);
});
```

**How it works:**
1. User visits `/posts/5`
2. Laravel sees the `{post}` parameter
3. It sees the `Post $post` type hint
4. It automatically runs `Post::findOrFail(5)`
5. If found, it injects the Post model
6. If not found, it returns a 404 error

### Resource Routes (RESTful Shorthand)

Instead of defining 7 routes manually:

```php
Route::resource('posts', PostController::class);
```

**This automatically creates:**
| HTTP Method | URL | Controller Method | Purpose |
|------------|-----|------------------|---------|
| GET | /posts | index | List all posts |
| GET | /posts/create | create | Show create form |
| POST | /posts | store | Save new post |
| GET | /posts/{post} | show | Display single post |
| GET | /posts/{post}/edit | edit | Show edit form |
| PUT/PATCH | /posts/{post} | update | Update post |
| DELETE | /posts/{post} | destroy | Delete post |

### Named Routes

Named routes let you reference routes by name instead of URL:

```php
// Define named route
Route::get('/posts', [PostController::class, 'index'])->name('posts.index');

// In controller - redirect by name
return redirect()->route('posts.index');

// With parameters
return redirect()->route('posts.show', ['post' => 5]); // Goes to /posts/5
```

**Why use named routes?**
- If you change `/posts` to `/articles`, you only update the route definition
- All redirects and links still work because they use the name, not the URL

---

## 3. Middleware: The Gatekeeper

### What is Middleware?

**Middleware** sits between the route and controller. It inspects/modifies requests before they reach your controller.

**Flow with middleware:**
```
Browser Request → Route → Middleware #1 → Middleware #2 → Controller
                                                             ↓
Browser ← Middleware #2 ← Middleware #1 ← Response ← Controller
```

**Common uses:**
- Check if user is authenticated
- Verify CSRF tokens
- Log requests
- Set headers

### Built-in Middleware Example

```php
// Only authenticated users can access these routes
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::resource('posts', PostController::class);
});

// If user is not authenticated:
// 1. Middleware intercepts the request
// 2. Redirects to /login
// 3. Controller never executes
```

### Custom Middleware

```bash
php artisan make:middleware CheckPostOwnership
```

```php
// app/Http/Middleware/CheckPostOwnership.php
public function handle(Request $request, Closure $next)
{
    $post = $request->route('post');
    
    // Check if current user owns the post
    if ($post->user_id !== auth()->id()) {
        abort(403, 'Unauthorized'); // Stop here, return 403 error
    }
    
    return $next($request); // Continue to controller
}
```

---

## 4. Controllers: The Traffic Director

### What Are Controllers?

**Controllers** contain the business logic of your application. They:
- Receive incoming requests
- Process data
- Interact with models/database
- Return responses (views, redirects, JSON)

Think of controllers as the brain that makes decisions about what to do with each request.

### Creating a Controller

```bash
# Create controller with resource methods
php artisan make:controller PostController --resource

# Create controller, model, and migration all at once
php artisan make:model Post -mcr
```

### Controller Structure and Methods

```php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Inertia;

class PostController extends Controller
{
    /**
     * INDEX - Display all posts
     * URL: GET /posts
     */
    public function index()
    {
        // Step 1: Query the database
        // latest() orders by created_at DESC
        // paginate(10) returns 10 posts per page + pagination data
        $posts = Post::latest()->paginate(10);
        
        // Step 2: Return data to Inertia/Vue
        // Inertia::render() does:
        // - Takes component name: 'Posts/Index'
        // - Maps to: resources/js/Pages/Posts/Index.vue
        // - Serializes $posts to JSON
        // - Sends to Vue as props
        return Inertia::render('Posts/Index', [
            'posts' => $posts
        ]);
    }

    /**
     * CREATE - Show form to create new post
     * URL: GET /posts/create
     */
    public function create()
    {
        // Just render the form component
        // No data needed - form starts empty
        return Inertia::render('Posts/Create');
    }

    /**
     * STORE - Save new post to database
     * URL: POST /posts
     */
    public function store(Request $request)
    {
        // Step 1: Validate incoming data
        // If validation fails, user is redirected back with error messages
        $validated = $request->validate([
            'title' => 'required|max:255',      // Must exist, max 255 chars
            'content' => 'required|min:10',     // Must exist, min 10 chars
            'author' => 'required|max:100',     // Must exist, max 100 chars
            'published' => 'boolean',           // Must be true/false
        ]);

        // Step 2: Create new post in database
        // Laravel automatically sets created_at and updated_at
        Post::create($validated);
        
        // Step 3: Redirect to index page
        // with() adds a flash message to the session
        return redirect()
            ->route('posts.index')
            ->with('success', 'Post created successfully!');
    }

    /**
     * SHOW - Display single post
     * URL: GET /posts/{post}
     */
    public function show(Post $post)
    {
        // Route model binding already loaded $post
        // Just pass it to the Vue component
        return Inertia::render('Posts/Show', [
            'post' => $post
        ]);
    }

    /**
     * EDIT - Show form to edit existing post
     * URL: GET /posts/{post}/edit
     */
    public function edit(Post $post)
    {
        // Pass existing post data to the form
        // Vue will pre-fill the form fields
        return Inertia::render('Posts/Edit', [
            'post' => $post
        ]);
    }

    /**
     * UPDATE - Save changes to existing post
     * URL: PUT /posts/{post}
     */
    public function update(Request $request, Post $post)
    {
        // Step 1: Validate the updated data
        $validated = $request->validate([
            'title' => 'required|max:255',
            'content' => 'required|min:10',
            'author' => 'required|max:100',
            'published' => 'boolean',
        ]);

        // Step 2: Update the post in database
        // updated_at is automatically set
        $post->update($validated);

        // Step 3: Redirect back to index
        return redirect()
            ->route('posts.index')
            ->with('success', 'Post updated successfully!');
    }

    /**
     * DESTROY - Delete post from database
     * URL: DELETE /posts/{post}
     */
    public function destroy(Post $post)
    {
        // Delete the post from database
        $post->delete();

        // Redirect back to index
        return redirect()
            ->route('posts.index')
            ->with('success', 'Post deleted successfully!');
    }
}
```

### Request Object - Accessing Form Data

```php
public function store(Request $request)
{
    // Get specific input
    $title = $request->input('title');
    
    // Get input with default value if missing
    $published = $request->input('published', false);
    
    // Get all input as array
    $all = $request->all();
    
    // Get only specific fields
    $data = $request->only(['title', 'content']);
    
    // Get all except specific fields
    $data = $request->except(['_token']);
    
    // Check if input exists
    if ($request->has('title')) {
        // Field exists
    }
    
    // Get uploaded file
    $file = $request->file('image');
}
```

### Validation Rules

```php
$request->validate([
    // Required field
    'title' => 'required',
    
    // Multiple rules separated by |
    'email' => 'required|email|unique:users',
    
    // Rules as array
    'password' => ['required', 'min:8', 'confirmed'],
    
    // Numeric constraints
    'age' => 'integer|min:18|max:100',
    'price' => 'numeric|between:0,1000',
    
    // String constraints
    'username' => 'string|min:3|max:20|alpha_dash',
    
    // Date validation
    'birth_date' => 'date|before:today',
    
    // File validation
    'avatar' => 'file|image|max:2048', // Max 2MB
    'document' => 'file|mimes:pdf,doc,docx',
    
    // Array validation
    'tags' => 'array|min:1|max:5',
    'tags.*' => 'string|max:20',
]);
```

### Custom Validation Messages

```php
$request->validate([
    'title' => 'required|max:255',
    'content' => 'required',
], [
    'title.required' => 'Please enter a post title',
    'title.max' => 'The title is too long (max 255 characters)',
    'content.required' => 'Post content cannot be empty',
]);
```

---

## 5. Models: Interacting with the Database

### What Are Models?

**Models** represent your database tables in object-oriented code. Instead of writing SQL queries, you use model methods.

**Benefits:**
- Clean, readable code
- Database agnostic (works with MySQL, PostgreSQL, SQLite, etc.)
- Built-in security (prevents SQL injection)
- Easy relationships between tables

### Creating a Model

```bash
# Basic model
php artisan make:model Post

# Model with migration
php artisan make:model Post -m

# Model with migration, controller, and resource routes
php artisan make:model Post -mcr
```

### Basic Model Structure

```php
// app/Models/Post.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * The table associated with the model.
     * Laravel auto-guesses 'posts' from 'Post' (plural, lowercase)
     */
    protected $table = 'posts';

    /**
     * Mass assignable attributes
     * These fields CAN be set via Post::create() or $post->fill()
     * This protects against mass-assignment vulnerabilities
     */
    protected $fillable = [
        'title',
        'content',
        'author',
        'published',
    ];

    /**
     * Attributes that should be cast to native types
     * When you access these attributes, Laravel converts them automatically
     */
    protected $casts = [
        'published' => 'boolean',      // '1' becomes true, '0' becomes false
        'created_at' => 'datetime',    // String becomes Carbon instance
        'metadata' => 'array',         // JSON becomes PHP array
    ];

    /**
     * Indicates if the model should be timestamped
     * If true, Laravel manages created_at and updated_at automatically
     */
    public $timestamps = true;
}
```

### Eloquent Query Examples (CRUD Operations)

```php
// CREATE - Adding new records
$post = Post::create([
    'title' => 'My First Post',
    'content' => 'This is the content',
    'author' => 'John Doe',
]);

// READ - Retrieving records
$posts = Post::all();                              // Get all posts
$post = Post::find(5);                             // Find by ID (returns null if not found)
$post = Post::findOrFail(5);                       // Find by ID (throws 404 if not found)
$posts = Post::where('published', true)->get();    // Get with conditions
$posts = Post::orderBy('created_at', 'desc')->get(); // Order results
$posts = Post::latest()->get();                    // Shorthand for orderBy created_at desc
$posts = Post::paginate(10);                       // Pagination (10 per page)

// UPDATE - Modifying records
$post = Post::find(5);
$post->update([
    'title' => 'Updated Title',
    'content' => 'Updated content',
]);

// DELETE - Removing records
$post = Post::find(5);
$post->delete();

// Or delete directly by ID
Post::destroy(5);
Post::destroy([1, 2, 3]); // Delete multiple
```

### Advanced Queries

```php
// Where clauses
Post::where('views', '>', 100)->get();
Post::where('title', 'like', '%Laravel%')->get();
Post::whereIn('id', [1, 2, 3])->get();
Post::whereNull('deleted_at')->get();
Post::whereBetween('views', [100, 500])->get();

// OR conditions
Post::where('author', 'John')
    ->orWhere('author', 'Jane')
    ->get();

// Aggregates
$count = Post::count();
$average = Post::avg('views');
$sum = Post::sum('views');
$max = Post::max('views');
```

### Accessors and Mutators (Getters and Setters)

```php
class Post extends Model
{
    /**
     * Accessor - Modifies data when you READ it
     * Automatically called when you access $post->title
     */
    protected function title(): Attribute
    {
        return Attribute::make(
            get: fn ($value) => ucwords($value), // Capitalize first letter
        );
    }

    /**
     * Mutator - Modifies data when you WRITE it
     * Automatically called when you set $post->title = 'something'
     */
    protected function title(): Attribute
    {
        return Attribute::make(
            set: fn ($value) => strtolower($value), // Convert to lowercase
        );
    }

    /**
     * Virtual attribute (doesn't exist in database)
     */
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}",
        );
    }
}
```

---

## 6. Inertia: The Bridge Between Laravel and Vue

### What is Inertia?

**Inertia.js** is a protocol that lets you build single-page applications (SPAs) without building an API. It acts as glue between Laravel and Vue.

**Traditional approach:**
```
Frontend (Vue) → API Request → Laravel API → JSON Response → Frontend renders
```

**Inertia approach:**
```
Frontend (Vue) → Page Request → Laravel Controller → Inertia → Frontend renders
```

### How Inertia Works

**First Visit (Server-Side):**
1. User visits `/posts`
2. Laravel route matches
3. Controller returns `Inertia::render('Posts/Index', $data)`
4. Inertia creates HTML with Vue app + data embedded
5. Browser receives full HTML page
6. Vue hydrates and takes over

**Subsequent Visits (Client-Side):**
1. User clicks link to `/posts/5`
2. Inertia intercepts the click
3. Makes XHR request to `/posts/5` with special header
4. Laravel recognizes it's an Inertia request
5. Returns JSON instead of HTML
6. Vue swaps out the component with new data
7. URL updates without page reload

**Result:** Feels like a SPA, but you never build an API!

### Controller Response with Inertia

```php
// In your controller
public function index()
{
    $posts = Post::latest()->paginate(10);
    
    // Inertia::render does three things:
    // 1. Component name: 'Posts/Index'
    //    Maps to: resources/js/Pages/Posts/Index.vue
    // 2. Data array: ['posts' => $posts]
    //    Gets serialized to JSON and passed as props
    // 3. Returns appropriate response:
    //    - First visit: Full HTML page
    //    - Inertia request: JSON only
    return Inertia::render('Posts/Index', [
        'posts' => $posts,
        'filters' => [
            'search' => request('search'),
            'status' => request('status'),
        ]
    ]);
}
```

### Sharing Data Globally

Some data needs to be available on every page (like auth user, flash messages):

```php
// app/Http/Middleware/HandleInertiaRequests.php
public function share(Request $request): array
{
    return array_merge(parent::share($request), [
        // Available on every page as $page.props.auth
        'auth' => [
            'user' => $request->user() ? [
                'id' => $request->user()->id,
                'name' => $request->user()->name,
                'email' => $request->user()->email,
            ] : null
        ],
        
        // Flash messages (available once after redirect)
        'flash' => [
            'success' => $request->session()->get('success'),
            'error' => $request->session()->get('error'),
        ],
    ]);
}
```

---

## 7. Vue Components: The User Interface

### What Are Vue Components?

**Vue components** are reusable UI pieces written in `.vue` files. They combine HTML (template), JavaScript (logic), and CSS (styling) in one file.

### Component Structure

```vue
<!-- resources/js/Pages/Posts/Index.vue -->

<!-- TEMPLATE: The HTML structure -->
<template>
    <div>
        <!-- Your HTML here -->
    </div>
</template>

<!-- SCRIPT: The JavaScript logic -->
<script setup>
// Your JavaScript here
</script>

<!-- STYLE: The CSS (optional) -->
<style scoped>
/* Your CSS here */
/* 'scoped' means styles only apply to this component */
</style>
```

### Receiving Data from Laravel

```vue
<script setup>
// defineProps declares what data this component expects
// Laravel passes this data via Inertia::render()
const props = defineProps({
    posts: Object,      // The posts pagination object
    filters: Object,    // Optional filters
});

// Access props in script
console.log(props.posts.data); // Array of posts
</script>

<template>
    <div>
        <!-- Access props directly in template -->
        <h1>Total Posts: {{ posts.total }}</h1>
    </div>
</template>
```

### Essential Vue Directives

```vue
<template>
    <!-- v-if: Conditional rendering (element removed from DOM) -->
    <div v-if="posts.length > 0">Posts exist</div>
    <div v-else>No posts</div>

    <!-- v-show: Conditional display (element hidden with CSS) -->
    <div v-show="isVisible">Hidden/shown but stays in DOM</div>

    <!-- v-for: Loop through arrays -->
    <div v-for="post in posts.data" :key="post.id">
        {{ post.title }}
    </div>

    <!-- v-model: Two-way data binding -->
    <input v-model="searchQuery" />

    <!-- v-bind (shorthand :): Bind attributes -->
    <img :src="post.image" :alt="post.title" />
    <a :href="`/posts/${post.id}`">View</a>

    <!-- v-on (shorthand @): Event listeners -->
    <button @click="handleClick">Click me</button>
    <form @submit.prevent="submitForm"></form>
    
    <!-- Dynamic classes -->
    <div :class="{ active: isActive, 'text-bold': isBold }"></div>
</template>
```

### Form Handling with Inertia

```vue
<script setup>
import { useForm } from '@inertiajs/vue3';

// useForm creates a reactive form object
// It automatically handles:
// - Form state
// - Validation errors
// - Loading states
const form = useForm({
    title: '',
    content: '',
    author: '',
});

// Submit handler
const submit = () => {
    // form.post() sends POST request to /posts
    form.post('/posts');
};
</script>

<template>
    <form @submit.prevent="submit">
        <div>
            <input v-model="form.title" type="text" />
            <!-- Validation error -->
            <span v-if="form.errors.title" class="text-red-500">
                {{ form.errors.title }}
            </span>
        </div>

        <button 
            type="submit"
            :disabled="form.processing"
        >
            {{ form.processing ? 'Creating...' : 'Create Post' }}
        </button>
    </form>
</template>
```

### Inertia Links

```vue
<script setup>
import { Link } from '@inertiajs/vue3';
</script>

<template>
    <!-- Inertia Link - no page reload! -->
    <Link href="/posts/create" class="btn">
        Create Post
    </Link>
    
    <!-- Dynamic href -->
    <Link :href="`/posts/${post.id}`">
        View Post
    </Link>
    
    <!-- With method (DELETE, PUT, etc.) -->
    <Link 
        :href="`/posts/${post.id}`" 
        method="delete"
        as="button"
    >
        Delete
    </Link>
</template>
```

---

## 8. Complete Request Lifecycle - Step by Step

Let's trace every step when a user creates a new post.

### Scenario: User Creates a New Post

#### **Step 1: Navigate to Create Page**

```
User Action: Clicks "Create Post" button
Browser: Inertia intercepts → GET /posts/create
```

**Route matches:**
```php
Route::get('/posts/create', [PostController::class, 'create']);
```

**Controller executes:**
```php
public function create()
{
    return Inertia::render('Posts/Create');
}
```

**Inertia:**
- Returns HTML with Vue app (first visit)
- Or JSON component data (subsequent visits)

**Vue:**
- Mounts `Posts/Create.vue` component
- Empty form renders

---

#### **Step 2: User Fills and Submits Form**

```
User Action: 
- Types: Title="Learning Laravel"
- Types: Author="Jane Doe"
- Types: Content="This is my first post..."
- Clicks: "Create Post"
```

**Vue component:**
```vue
<script setup>
const form = useForm({
    title: 'Learning Laravel',
    content: 'This is my first post...',
    author: 'Jane Doe',
});

const submit = () => {
    form.post('/posts'); // Sends POST request
};
</script>
```

**Inertia (Client):**
- Serializes form data to JSON
- Sends POST request to `/posts`

---

#### **Step 3: Laravel Receives Request**

**Route matches:**
```php
Route::post('/posts', [PostController::class, 'store']);
```

**Middleware runs:**
- CSRF verification ✓
- Authentication check (if required) ✓

**Controller processes:**
```php
public function store(Request $request)
{
    // STEP 1: Validate
    $validated = $request->validate([
        'title' => 'required|max:255',
        'content' => 'required|min:10',
        'author' => 'required|max:100',
    ]);
    // Checks:
    // - title exists? ✓
    // - title < 255 chars? ✓
    // - content exists? ✓
    // - content > 10 chars? ✓
    // - author exists? ✓
    
    // STEP 2: Create record
    Post::create($validated);
    
    // STEP 3: Redirect with success message
    return redirect()
        ->route('posts.index')
        ->with('success', 'Post created successfully!');
}
```

---

#### **Step 4: Database Interaction**

**Eloquent processes:**
```php
Post::create($validated);
```

**What happens:**
1. Takes validated data
2. Adds timestamps automatically:
```php
[
    'title' => 'Learning Laravel',
    'content' => 'This is my first post...',
    'author' => 'Jane Doe',
    'created_at' => '2024-10-22 14:30:00',
    'updated_at' => '2024-10-22 14:30:00',
]
```

3. Converts to SQL:
```sql
INSERT INTO posts (
    title, 
    content, 
    author, 
    created_at, 
    updated_at
) VALUES (
    'Learning Laravel',
    'This is my first post...',
    'Jane Doe',
    '2024-10-22 14:30:00',
    '2024-10-22 14:30:00'
);
```

4. Database executes query
5. Returns new record with auto-increment ID: `15`

---

#### **Step 5: Redirect Response**

**Controller returns:**
```php
return redirect()
    ->route('posts.index')
    ->with('success', 'Post created successfully!');
```

**What happens:**
1. Flash message stored in session
2. Redirect response created:
```
Status: 303 See Other
Location: /posts
```

**Inertia (Server):**
- Recognizes redirect
- Tells client to navigate to `/posts`

**Browser:**
- Receives redirect
- Inertia makes new GET request to `/posts`

---

#### **Step 6: Display Updated List**

**New request:**
```
GET /posts
```

**Route matches:**
```php
Route::get('/posts', [PostController::class, 'index']);
```

**Controller executes:**
```php
public function index()
{
    $posts = Post::latest()->paginate(10);
    
    return Inertia::render('Posts/Index', [
        'posts' => $posts
    ]);
}
```

**Database query:**
```sql
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 0
```

**Returns:** 10 most recent posts, including our new post (id: 15)

**Inertia response:**
```json
{
    "component": "Posts/Index",
    "props": {
        "posts": {
            "data": [
                {
                    "id": 15,
                    "title": "Learning Laravel",
                    "author": "Jane Doe",
                    "content": "This is my first post...",
                    "created_at": "2024-10-22 14:30:00"
                },
                ...9 more posts
            ],
            "total": 42,
            "current_page": 1
        },
        "flash": {
            "success": "Post created successfully!"
        }
    }
}
```

**Vue component updates:**
- Displays success message
- Renders posts list
- New post appears at the top

**Browser:**
- User sees their new post! ✓

---

## 9. Data Flow Summary Diagram

```
┌─────────────────────────────────────────────────────────┐
│                      USER INTERACTION                    │
│  Clicks "Create Post" → Fills form → Clicks "Submit"    │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    VUE COMPONENT                         │
│  form.post('/posts', data)                              │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    INERTIA (CLIENT)                      │
│  Intercepts → Serializes → Sends XHR POST request       │
└──────────────────────┬──────────────────────────────────┘
                       │ POST /posts + JSON data
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   LARAVEL ROUTES                         │
│  Matches: POST /posts → PostController@store            │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    MIDDLEWARE                            │
│  CSRF check ✓ → Auth check ✓ → Continue                │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    CONTROLLER                            │
│  1. Validate data                                        │
│  2. Post::create($validated)                            │
│  3. redirect()->route('posts.index')                    │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                      MODEL                               │
│  Prepare INSERT query → Execute                          │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    DATABASE                              │
│  INSERT INTO posts ... → Return new ID                   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   CONTROLLER                             │
│  Redirect with flash message                             │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                 INERTIA (SERVER)                         │
│  Send redirect response                                  │
└──────────────────────┬──────────────────────────────────┘
                       │ GET /posts (new request)
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   CONTROLLER                             │
│  Query: Post::latest()->paginate(10)                    │
│  Return: Inertia::render('Posts/Index', $posts)         │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                    DATABASE                              │
│  SELECT * FROM posts ORDER BY created_at DESC           │
│  Returns: 10 posts including new one                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                 INERTIA (SERVER)                         │
│  Serialize posts + flash message to JSON                │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                  VUE COMPONENT                           │
│  Receive props → Update UI → Display posts              │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                     BROWSER                              │
│  User sees new post in the list! ✓                      │
└─────────────────────────────────────────────────────────┘
```

---

## 10. Common Patterns and Best Practices

### Search and Filter Pattern

**Controller:**
```php
public function index(Request $request)
{
    $query = Post::query();
    
    // Apply search if provided
    if ($request->has('search')) {
        $query->where('title', 'like', '%' . $request->search . '%');
    }
    
    // Apply status filter
    if ($request->status === 'published') {
        $query->where('published', true);
    }
    
    $posts = $query->paginate(10)->withQueryString();
    
    return Inertia::render('Posts/Index', [
        'posts' => $posts,
        'filters' => $request->only('search', 'status'),
    ]);
}
```

**Vue:**
```vue
<script setup>
import { ref, watch } from 'vue';
import { router } from '@inertiajs/vue3';

const props = defineProps({
    filters: Object,
});

const search = ref(props.filters.search || '');

// Watch for changes and update URL
watch(search, (value) => {
    router.get('/posts', { search: value }, {
        preserveState: true,  // Keep component state
        replace: true,        // Don't add to history
    });
});
</script>

<template>
    <input v-model="search" placeholder="Search..." />
</template>
```

### Flash Messages Pattern

**Layout Component:**
```vue
<!-- resources/js/Layouts/AppLayout.vue -->
<script setup>
import { computed } from 'vue';
import { usePage } from '@inertiajs/vue3';

const page = usePage();
const flash = computed(() => page.props.flash);
</script>

<template>
    <div>
        <!-- Success message -->
        <div v-if="flash.success" class="alert alert-success">
            {{ flash.success }}
        </div>
        
        <!-- Error message -->
        <div v-if="flash.error" class="alert alert-error">
            {{ flash.error }}
        </div>
        
        <!-- Page content -->
        <slot />
    </div>
</template>
```

### Loading States Pattern

```vue
<script setup>
import { router } from '@inertiajs/vue3';
import { ref } from 'vue';

const deleting = ref(false);

const deletePost = (postId) => {
    if (confirm('Delete this post?')) {
        deleting.value = true;
        
        router.delete(`/posts/${postId}`, {
            onFinish: () => {
                deleting.value = false;
            }
        });
    }
};
</script>

<template>
    <button 
        @click="deletePost(post.id)"
        :disabled="deleting"
    >
        {{ deleting ? 'Deleting...' : 'Delete' }}
    </button>
</template>
```

### File Upload Pattern

**Migration:**
```php
$table->string('image')->nullable();
```

**Controller:**
```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|max:255',
        'image' => 'nullable|image|max:2048', // Max 2MB
    ]);
    
    if ($request->hasFile('image')) {
        $path = $request->file('image')->store('images', 'public');
        $validated['image'] = $path;
    }
    
    Post::create($validated);
    
    return redirect()->route('posts.index');
}
```

**Vue:**
```vue
<script setup>
import { useForm } from '@inertiajs/vue3';

const form = useForm({
    title: '',
    image: null,
});

const handleFileChange = (event) => {
    form.image = event.target.files[0];
};

const submit = () => {
    form.post('/posts');
};
</script>

<template>
    <form @submit.prevent="submit">
        <input v-model="form.title" type="text" />
        <input type="file" @change="handleFileChange" accept="image/*" />
        <button type="submit">Submit</button>
    </form>
</template>
```

---

## 11. Key Concepts Summary

### 1. Migrations
- **Purpose:** Define database structure in code
- **When:** Run once when creating/modifying tables
- **Command:** `php artisan migrate`

### 2. Routes
- **Purpose:** Map URLs to controller methods
- **Location:** `routes/web.php`
- **Types:** GET, POST, PUT, DELETE
- **Bonus:** Use resource routes for automatic CRUD routes

### 3. Controllers
- **Purpose:** Handle business logic
- **Responsibilities:** 
  - Validate data
  - Interact with models
  - Return responses
- **Methods:** index, create, store, show, edit, update, destroy

### 4. Models
- **Purpose:** Interact with database tables
- **Benefits:** No SQL needed, clean syntax
- **Key Properties:**
  - `$fillable` - mass assignable fields
  - `$casts` - automatic type conversion
  - `$timestamps` - auto manage created_at/updated_at

### 5. Inertia
- **Purpose:** Bridge between Laravel and Vue
- **Benefit:** SPA without building API
- **How:** 
  - First visit: Full HTML page
  - Subsequent: JSON only (fast!)

### 6. Vue Components
- **Purpose:** Build user interface
- **Structure:** Template + Script + Style
- **Key Features:**
  - Reactive data
  - Two-way binding (v-model)
  - Event handling (@click)
  - Conditional rendering (v-if)

---

## 12. Common Commands Reference

```bash
# Migrations
php artisan make:migration create_posts_table
php artisan migrate
php artisan migrate:rollback
php artisan migrate:fresh

# Models
php artisan make:model Post
php artisan make:model Post -m          # with migration
php artisan make:model Post -mcr        # with migration, controller, resource

# Controllers
php artisan make:controller PostController
php artisan make:controller PostController --resource

# View all routes
php artisan route:list

# Clear caches
php artisan cache:clear
php artisan config:clear
php artisan route:clear

# Run development server
php artisan serve

# Run Vite (for Vue/assets)
npm run dev
```

---

## 13. Practice Exercise

Build a simple blog system with the following features:

### Database Table
```
posts:
- id
- title (string, required)
- content (text, required)
- author (string, required)
- published (boolean, default false)
- created_at
- updated_at
```

### Routes Needed
- GET /posts - List all posts
- GET /posts/create - Show create form
- POST /posts - Save new post
- GET /posts/{post} - Show single post
- GET /posts/{post}/edit - Show edit form
- PUT /posts/{post} - Update post
- DELETE /posts/{post} - Delete post

### Vue Components Needed
1. **Posts/Index.vue** - Display list of posts with:
   - Post title, author, excerpt
   - Links to view, edit, delete
   - "Create Post" button

2. **Posts/Create.vue** - Form with:
   - Title input
   - Author input
   - Content textarea
   - Published checkbox
   - Submit button

3. **Posts/Show.vue** - Display:
   - Full post details
   - Back to list button
   - Edit button

4. **Posts/Edit.vue** - Form pre-filled with post data

### Steps to Complete
1. Create migration for posts table
2. Run migration
3. Create Post model with $fillable
4. Create PostController with resource methods
5. Add routes (use resource route)
6. Create Vue components
7. Test each CRUD operation

---

## 14. Debugging Tips

### Check if route exists
```bash
php artisan route:list
```

### Check database connection
```bash
php artisan tinker
>>> DB::connection()->getPdo();
```

### Test database queries
```bash
php artisan tinker
>>> Post::all();
>>> Post::create(['title' => 'Test', ...]);
```

### View what data is sent to Vue
```php
// In controller
dd($posts); // Shows data and stops execution
```

### Check validation errors in Vue
```vue
<template>
    <div>
        <pre>{{ form.errors }}</pre>
    </div>
</template>
```

### Enable query logging
```php
// In controller
DB::enableQueryLog();
$posts = Post::all();
dd(DB::getQueryLog()); // See SQL queries
```

---

## 15. Additional Resources

- **Laravel Documentation:** https://laravel.com/docs
- **Inertia.js Documentation:** https://inertiajs.com
- **Vue 3 Documentation:** https://vuejs.org
- **Tailwind CSS Documentation:** https://tailwindcss.com

---

## Final Notes

The VILT stack workflow always follows this pattern:

1. **Define structure** (Migration)
2. **Create entry point** (Route)
3. **Handle logic** (Controller)
4. **Talk to database** (Model)
5. **Bridge to frontend** (Inertia)
6. **Display to user** (Vue)

Master this flow, and you can build any feature! Each piece has a specific job, and they all work together seamlessly.
