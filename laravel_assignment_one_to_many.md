# Assignment: E-Commerce Product & Review System

## Objective

Create a Laravel data resource structure that models an e-commerce system where Products have many Reviews. Implement the complete workflow: generate models, create migrations, define relationships, build factories, write seeders, and populate your database with fake data.

---

## Requirements

### Part 1: Generate Models

Generate two models with all necessary files (migration, factory, seeder):
- `Product`
- `Review`

### Part 2: Product Migration

Create a `products` table with:
- `id`
- `name`
- `description`
- `price` (use decimal type)
- `sku` (unique)
- `stock_quantity`
- `created_at` and `updated_at`

### Part 3: Review Migration

Create a `reviews` table with:
- `id`
- `product_id` (foreign key with cascade delete)
- `customer_name`
- `rating` (integer)
- `comment`
- `created_at` and `updated_at`

### Part 4: Models with Relationships

**Product model:**
- Define `$fillable` with appropriate columns
- Create `reviews()` relationship method

**Review model:**
- Define `$fillable` with appropriate columns
- Create `product()` relationship method

### Part 5: Factories

**ProductFactory:**
- Generate fake data for all product columns

**ReviewFactory:**
- Generate fake data for all review columns

### Part 6: Seeders

**ProductSeeder:**
- Create 15 products

**ReviewSeeder:**
- Create 5-8 reviews for each product

**DatabaseSeeder:**
- Register both seeders

### Part 7: Test

Run migrations and seeders:
```bash
php artisan migrate:fresh --seed
```

Verify the data was created correctly and relationships work.

---

## Submission Checklist

- [ ] Both models generated with migrations, factories, and seeders
- [ ] Migrations created with correct columns and types
- [ ] Foreign key relationship properly configured
- [ ] Both models define relationships
- [ ] Factories generate appropriate fake data
- [ ] Seeders create data with correct relationships
- [ ] DatabaseSeeder calls both seeders
- [ ] Database seeded successfully without errors