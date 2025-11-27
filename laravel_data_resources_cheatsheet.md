# Laravel Data Resources Cheatsheet

## Quick Start: Generate Everything at Once

The simplest way to create a complete data resource is to use a single artisan command:

```bash
php artisan make:model Post -mfs
```

**What does `-mfs` mean?**
- `-m` = creates a Migration
- `-f` = creates a Factory
- `-s` = creates a Seeder

This generates four files at once:
1. `app/Models/Post.php` (Model)
2. `database/migrations/YYYY_MM_DD_HHMMSS_create_posts_table.php` (Migration)
3. `database/factories/PostFactory.php` (Factory)
4. `database/seeders/PostSeeder.php` (Seeder)

**Adapting This Pattern to Other Models**

In this cheatsheet, we use **Post** as the example, but this pattern works for ANY model. Here's how to adapt it:

- Model name: `Post` → Replace with your model name (e.g., `UserProfile`, `Product`, `Article`)
- Table name: `posts` → Automatically becomes plural lowercase (e.g., `user_profiles`, `products`, `articles`)
- Factory class: `PostFactory` → Automatically matches your model (e.g., `UserProfileFactory`)
- Seeder class: `PostSeeder` → Automatically matches your model (e.g., `UserProfileSeeder`)
- Migration name: `create_posts_table` → Automatically matches your table (e.g., `create_user_profiles_table`)

**Examples:**

| Command | Creates | Model | Migration | Factory | Seeder |
|---------|---------|-------|-------|---------|--------|
| `make:model Post -mfs` | All 4 files | `Post.php` | `YYYY_MM_DD_HHMMSS_create_posts_table.php` | `PostFactory.php` | `PostSeeder.php` |
| `make:model UserProfile -mfs` | All 4 files | `UserProfile.php` | `YYYY_MM_DD_HHMMSS_create_user_profiles_table.php` | `UserProfileFactory.php` | `UserProfileSeeder.php` |
| `make:model Product -mfs` | All 4 files | `Product.php` | `YYYY_MM_DD_HHMMSS_products_table.php` | `ProductFactory.php` | `ProductSeeder.php` |

**Laravel is smart:** It automatically handles the naming. You only write the model name in PascalCase (like `UserProfile`), and Laravel figures out the rest!

---

## Step-by-Step: Creating a Post Resource

### Step 1: Generate All Files

```bash
php artisan make:model Post -mfs
```

### Step 2: Define the Migration (Database Schema)

**File:** `database/migrations/YYYY_MM_DD_HHMMSS_create_posts_table.php`

This file defines the database table structure:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('content');
            $table->string('slug')->unique();
            $table->integer('views')->default(0);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

**Key column types:**
- `$table->id()` - Auto-incrementing primary key
- `$table->string('title')` - VARCHAR(255)
- `$table->text('content')` - Longer text field
- `$table->integer('views')` - Whole number
- `$table->timestamps()` - Adds `created_at` and `updated_at`
- `$table->unique()` - Makes column unique

### Step 3: Define the Model (Application Logic)

**File:** `app/Models/Post.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    protected $fillable = ['title', 'content', 'slug', 'views'];
}
```

**Alternative using `$guarded`:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    protected $guarded = ['id'];
}
```

**`$fillable` vs `$guarded` - Which one to use?**

- **`$fillable`** (whitelist approach) - Explicitly list columns that CAN be mass-assigned. Everything else is protected. **Recommended for security.**
  - Use: `protected $fillable = ['title', 'content', 'slug', 'views'];`
  - Only these 4 columns can be mass-assigned; `id`, `created_at`, `updated_at` are automatically protected

- **`$guarded`** (blacklist approach) - Explicitly list columns that CANNOT be mass-assigned. Everything else can be assigned.
  - Use: `protected $guarded = ['id'];`
  - Only `id` is protected; all other columns can be mass-assigned
  - Use `$guarded = []` to allow everything (not recommended)

**Important:** You must use one or the other. Without either, you'll get a `MassAssignmentException`. Most developers prefer `$fillable` because it's more explicit and secure—you decide what's allowed rather than what's blocked.

### Step 4: Create Fake Data in the Factory

**File:** `database/factories/PostFactory.php`

The factory generates realistic fake data:

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class PostFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title' => $this->faker->sentence(),
            'content' => $this->faker->paragraphs(3, true),
            'slug' => $this->faker->unique()->slug(),
            'views' => $this->faker->numberBetween(0, 10000),
        ];
    }
}
```

**Why is the factory definition structured this way?**

The `definition()` method returns an associative array where:
- **Keys** = column names in your database table (must match exactly!)
- **Values** = fake data generated by Faker

When you call `Post::factory(50)->create()`, Laravel does this:

1. Loops 50 times
2. Calls `definition()` to get a fresh array of fake data
3. Creates a new Post with that data
4. Saves it to the database

**Important:** Notice we only define columns that are not auto-generated. We DON'T include:
- `id` - Database auto-increments this
- `created_at` and `updated_at` - Laravel sets these automatically via `timestamps()`

If we tried to include them, Laravel would either ignore them or throw an error. The factory only provides data for columns that need custom values.

**Faker methods explained:**

- `$this->faker->sentence()` - Random sentence (good for titles)
- `$this->faker->paragraphs(3, true)` - 3 paragraphs as one string (true = join with newlines)
- `$this->faker->unique()->slug()` - Unique URL-friendly string (perfect for slugs)
- `$this->faker->numberBetween(0, 10000)` - Random integer between 0 and 10000
- `$this->faker->email()` - Fake email address
- `$this->faker->name()` - Fake person's name
- `$this->faker->phoneNumber()` - Fake phone number
- `$this->faker->address()` - Fake street address
- `$this->faker->word()` - Single random word
- `$this->faker->boolean()` - True or false

### Step 5: Define the Seeder (Populate Database)

**File:** `database/seeders/PostSeeder.php`

```php
<?php

namespace Database\Seeders;

use App\Models\Post;
use Illuminate\Database\Seeder;

class PostSeeder extends Seeder
{
    public function run(): void
    {
        // Create 50 posts with fake data
        Post::factory(50)->create();
    }
}
```

### Step 6: Register Seeder in DatabaseSeeder

**File:** `database/seeders/DatabaseSeeder.php`

Add your seeder to the main seeder:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            PostSeeder::class,
        ]);
    }
}
```

### Step 7: Run Migrations and Seeders

```bash
# Run all migrations and seeders
php artisan migrate:fresh --seed

# OR separately:
php artisan migrate          # Creates tables
php artisan db:seed          # Populates with fake data
```

---

## One-to-Many Relationship Example

Let's create a Post → Comment relationship (one post has many comments).

### Step 1: Create Comment Resource

```bash
php artisan make:model Comment -mfs
```

### Step 2: Update Comment Migration

**File:** `database/migrations/YYYY_MM_DD_HHMMSS_create_comments_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('post_id')->constrained()->onDelete('cascade');
            $table->string('author');
            $table->text('body');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
};
```

**Key parts:**
- `$table->foreignId('post_id')` - Foreign key to posts table
- `->constrained()` - Links to `posts.id` automatically
- `->onDelete('cascade')` - Delete comments when post is deleted

### Step 3: Define Models with Relationships

**File:** `app/Models/Post.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    use HasFactory;

    protected $fillable = ['title', 'content', 'slug', 'views'];

    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

**File:** `app/Models/Comment.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    use HasFactory;

    protected $fillable = ['post_id', 'author', 'body'];

    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

### Step 4: Create Comment Factory

**File:** `database/factories/CommentFactory.php`

```php
<?php

namespace Database\Factories;

use App\Models\Post;
use Illuminate\Database\Eloquent\Factories\Factory;

class CommentFactory extends Factory
{
    public function definition(): array
    {
        return [
            'post_id' => Post::factory(),
            'author' => $this->faker->name(),
            'body' => $this->faker->paragraph(),
        ];
    }
}
```

### Step 5: Create Comment Seeder

**File:** `database/seeders/CommentSeeder.php`

```php
<?php

namespace Database\Seeders;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Database\Seeder;

class CommentSeeder extends Seeder
{
    public function run(): void
    {
        // Get all posts and create comments for each
        Post::all()->each(function (Post $post) {
            Comment::factory(5)->create(['post_id' => $post->id]);
        });
    }
}
```

**Better approach using state:**

```php
<?php

namespace Database\Seeders;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Database\Seeder;

class CommentSeeder extends Seeder
{
    public function run(): void
    {
        // Create 5 comments for each of the 50 posts (250 comments total)
        Post::all()->each(function (Post $post) {
            Comment::factory(5)->for($post)->create();
        });
    }
}
```

### Step 6: Register in DatabaseSeeder

**File:** `database/seeders/DatabaseSeeder.php`

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }
}
```

### Step 7: Run Everything

```bash
php artisan migrate:fresh --seed
```

### Step 8: Use in Code

```php
// Get a post with all its comments
$post = Post::with('comments')->find(1);

// Loop through comments
foreach ($post->comments as $comment) {
    echo $comment->author . ': ' . $comment->body;
}

// Get a comment's parent post
$comment = Comment::find(1);
echo $comment->post->title;

// Count comments on a post
$post->comments()->count();
```

---

## Practice Exercise: UserProfile → Setting Relationship

Now that you understand the pattern with Post → Comment, try building this yourself! Here's what you need to create:

- **UserProfile model** (one user profile)
- **Setting model** (many settings per profile)
- **Relationship:** One UserProfile `hasMany` Settings

### What You Should Create:

1. Generate both models:
```bash
php artisan make:model UserProfile -mfs
php artisan make:model Setting -mfs
```

2. Update the **UserProfile migration** to include columns like `username`, `email`, `bio`, `avatar_url`

3. Update the **Setting migration** to include:
   - `user_profile_id` (foreign key - this is the key!)
   - `setting_name` (e.g., "theme", "notifications_enabled")
   - `setting_value` (e.g., "dark", "true")

4. Add the relationship in **UserProfile model**:
```php
public function settings(): HasMany
{
    return $this->hasMany(Setting::class);
}
```

5. Add the inverse relationship in **Setting model**:
```php
public function userProfile(): BelongsTo
{
    return $this->belongsTo(UserProfile::class);
}
```

6. Update factories to generate fake data for both models

7. Create seeders that:
   - Create 20 UserProfiles
   - For each profile, create 3-5 Settings

8. Register both seeders in `DatabaseSeeder` and run `php artisan migrate:fresh --seed`

**Key differences from Post → Comment:**
- Model names are different (UserProfile instead of Post, Setting instead of Comment)
- Table names are different (`user_profiles`, `settings`)
- Factory and Seeder names follow the model names
- Everything else is the same!

**If you get stuck:** Compare your code to the Post → Comment example. The only differences should be:
- Class names
- Table names
- Column names
- The actual fake data being generated

The structure is identical!

```bash
# Create model with all files
php artisan make:model Post -mfs

# Create only migration
php artisan make:migration create_posts_table

# Create only factory
php artisan make:factory PostFactory

# Create only seeder
php artisan make:seeder PostSeeder

# Run migrations
php artisan migrate

# Run migrations with seeders
php artisan migrate:fresh --seed

# Rollback last migration
php artisan migrate:rollback

# Run all seeders
php artisan db:seed

# Run specific seeder
php artisan db:seed --class=PostSeeder

# Refresh and seed (rollback + migrate + seed)
php artisan migrate:refresh --seed
```

---

## Common Mistakes to Avoid

1. **Forgetting `$fillable` in Model** - You'll get `MassAssignmentException`
2. **Not registering seeder in DatabaseSeeder** - Your seeder won't run
3. **Forgetting `constrained()` in foreign key** - You'll get database errors
4. **Not using `factory()` in factories** - Relationships won't work properly
5. **Running seeders before migrations** - Tables don't exist yet
6. **Not using `migrate:fresh --seed`** - Old data causes conflicts
