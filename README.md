# 🚀 Laravel 13 Complete Setup & API Guide

Technical Documentation | Platform: Linux | Date: July 2026

---

## 📋 Table of Contents

- [0. Environment Preparation (From Zero)](#0-environment-preparation-from-zero)
- [1. Installing Laravel 13](#1-installing-laravel-13)
- [2. Database Configuration (SQLite)](#2-database-configuration-sqlite)
- [3. Creating Model & Migration](#3-creating-model--migration)
- [4. Creating API Controller](#4-creating-api-controller)
- [5. Creating API Resource](#5-creating-api-resource)
- [6. Routing Setup](#6-routing-setup)
- [7. CORS Configuration](#7-cors-configuration)
- [8. Testing API Endpoint](#8-testing-api-endpoint)
- [9. Seeding Data](#9-seeding-data)
- [10. Common Errors & Solutions](#10-common-errors--solutions)
- [11. Quick Command Summary](#11-quick-command-summary)
- [12. Postman Collection](#12-postman-collection)

---

## 0. Environment Preparation (From Zero)

### 0.1 Install PHP 8.2+

```bash
# Add PHP repository
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.2 with required extensions
sudo apt install php8.2 php8.2-cli php8.2-common \
php8.2-mysql php8.2-zip php8.2-gd php8.2-mbstring \
php8.2-curl php8.2-xml php8.2-bcmath php8.2-sqlite3 -y

# Check PHP version
php --version
```

#### Required Extensions for Laravel

| Extension | Function |
|-----------|----------|
| `php8.2-mysql` | MySQL connection |
| `php8.2-sqlite3` | SQLite connection |
| `php8.2-mbstring` | Multibyte string manipulation |
| `php8.2-xml` | XML parsing |
| `php8.2-curl` | HTTP requests (API) |
| `php8.2-zip` | Zip file extraction |
| `php8.2-gd` | Image manipulation |
| `php8.2-bcmath` | High precision calculations |

---

### 0.2 Install Composer (Global)

```bash
# Download Composer installer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Run installer
php composer-setup.php

# Move to global
sudo mv composer.phar /usr/local/bin/composer

# Remove installer
php -r "unlink('composer-setup.php');"

# Check version
composer --version
```

**Alternative (using apt):**

```bash
sudo apt install composer
```

---

### 0.3 Install Node.js & NPM (Optional for Frontend)

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Check versions
node --version
npm --version
```

---

### 0.4 Install Git (Optional for Version Control)

```bash
sudo apt install git
git --version
```

---

### 0.5 Install SQLite (For Lightweight Database)

```bash
sudo apt install sqlite3
sqlite3 --version
```

---

## 1. Installing Laravel 13

### 1.1 Create New Project

```bash
composer create-project laravel/laravel laravel-13-api
```

This process will:

- Download latest Laravel 13
- Install all dependencies
- Generate `.env` file
- Setup folder structure

---

### 1.2 Enter Project Directory

```bash
cd laravel-13-api
```

---

### 1.3 Start Development Server

```bash
php artisan serve
```

**Output:**

```
INFO Server running on [http://127.0.0.1:8000].
```

Open `http://127.0.0.1:8000` in browser. You should see Laravel welcome page.

---

## 2. Database Configuration (SQLite)

### 2.1 Create SQLite Database File

```bash
touch database/database.sqlite
```

### 2.2 Edit .env File

```bash
nano .env
```

Change database section to:

```
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
```

Comment out these lines:

```
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

### 2.3 Clear Config Cache

```bash
php artisan config:clear
```

### 2.4 Test Database Connection

```bash
php artisan migrate
```

If successful:

```
Migration table created successfully.
```

---

## 3. Creating Model & Migration

### 3.1 Create Model with Migration

```bash
php artisan make:model Product -m
```

### 3.2 Edit Migration File

**📍 Location:** `database/migrations/..._create_products_table.php`

```php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->integer('price');
        $table->text('description')->nullable();
        $table->boolean('is_active')->default(true);
        $table->timestamps();
    });
}
```

### 3.3 Edit Model File

**📍 Location:** `app/Models/Product.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'price',
        'description',
        'is_active',
    ];
}
```

### 3.4 Run Migration

```bash
php artisan migrate
```

---

## 4. Creating API Controller

### 4.1 Create Controller

```bash
php artisan make:controller ProductController --api
```

**📍 Location:** `app/Http/Controllers/ProductController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use App\Http\Resources\ProductResource;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index()
    {
        $products = Product::paginate(10);
        return ProductResource::collection($products);
    }

    public function store(Request $request)
    {
        try {
            $validated = $request->validate([
                'name' => 'required|string|max:255',
                'price' => 'required|integer|min:0',
                'description' => 'nullable|string',
                'is_active' => 'boolean'
            ]);

            $product = Product::create($validated);
            return new ProductResource($product);
        } catch (\Exception $e) {
            return response()->json([
                'error' => 'Failed to create product',
                'message' => $e->getMessage()
            ], 500);
        }
    }

    public function show($id)
    {
        try {
            $product = Product::findOrFail($id);
            return new ProductResource($product);
        } catch (\Exception $e) {
            return response()->json([
                'error' => 'Product not found',
                'message' => $e->getMessage()
            ], 404);
        }
    }

    public function update(Request $request, $id)
    {
        try {
            $product = Product::findOrFail($id);

            $validated = $request->validate([
                'name' => 'sometimes|string|max:255',
                'price' => 'sometimes|integer|min:0',
                'description' => 'nullable|string',
                'is_active' => 'boolean'
            ]);

            $product->update($validated);
            return new ProductResource($product);
        } catch (\Exception $e) {
            return response()->json([
                'error' => 'Failed to update product',
                'message' => $e->getMessage()
            ], 500);
        }
    }

    public function destroy($id)
    {
        try {
            $product = Product::findOrFail($id);
            $product->delete();
            return response()->json([
                'message' => 'Product deleted successfully'
            ]);
        } catch (\Exception $e) {
            return response()->json([
                'error' => 'Failed to delete product',
                'message' => $e->getMessage()
            ], 500);
        }
    }
}
```

---

## 5. Creating API Resource

### 5.1 Create Resource

```bash
php artisan make:resource ProductResource
```

**📍 Location:** `app/Http/Resources/ProductResource.php`

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'price' => $this->price,
            'description' => $this->description,
            'is_active' => $this->is_active,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

---

## 6. Routing Setup

**📍 Location:** `routes/api.php`

```php
<?php

use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::apiResource('products', ProductController::class);
```

#### Generated Routes

| Method | URL | Controller Method |
|--------|-----|------------------|
| GET | `/api/products` | `index()` |
| POST | `/api/products` | `store()` |
| GET | `/api/products/{id}` | `show()` |
| PUT/PATCH | `/api/products/{id}` | `update()` |
| DELETE | `/api/products/{id}` | `destroy()` |

---

## 7. CORS Configuration

**📍 Location:** `config/cors.php`

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => ['*'], // For development only
'allowed_origins_patterns' => [],
'allowed_headers' => ['*'],
'exposed_headers' => [],
'max_age' => 0,
'supports_credentials' => false,
```

#### For Production:

```php
'allowed_origins' => [
    'https://yourdomain.com',
    'http://localhost:5173',
],
```

---

## 8. Testing API Endpoint

### 8.1 Start Server

```bash
php artisan serve
```

### 8.2 Test with Browser

Open: `http://127.0.0.1:8000/api/products`

### 8.3 Test with cURL

```bash
# GET all products (paginated)
curl http://127.0.0.1:8000/api/products

# GET single product
curl http://127.0.0.1:8000/api/products/1

# POST new product
curl -X POST http://127.0.0.1:8000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name": "Gaming Laptop", "price": 15000000, "description": "High performance gaming laptop"}'

# PUT update product
curl -X PUT http://127.0.0.1:8000/api/products/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "Updated Laptop", "price": 17000000}'

# DELETE product
curl -X DELETE http://127.0.0.1:8000/api/products/1
```

---

## 9. Seeding Data

### 9.1 Create Seeder

```bash
php artisan make:seeder ProductSeeder
```

**📍 Location:** `database/seeders/ProductSeeder.php`

```php
<?php

namespace Database\Seeders;

use App\Models\Product;
use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        Product::create([
            'name' => 'Gaming Laptop',
            'price' => 15000000,
            'description' => 'High performance gaming laptop with RTX 4060',
            'is_active' => true,
        ]);

        Product::create([
            'name' => 'Mechanical Keyboard',
            'price' => 1500000,
            'description' => 'RGB mechanical keyboard with Cherry MX switches',
            'is_active' => true,
        ]);

        Product::create([
            'name' => 'Wireless Mouse',
            'price' => 500000,
            'description' => 'Ergonomic wireless mouse with 16000 DPI',
            'is_active' => false,
        ]);
    }
}
```

### 9.2 Run Seeder

```bash
php artisan db:seed --class=ProductSeeder
```

### 9.3 Verify Data

```bash
php artisan tinker
```

```php
App\Models\Product::all();
```

---

## 10. Common Errors & Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `SQLSTATE[HY000] [14] unable to open database file` | SQLite permission issue | `sudo chmod 777 database/database.sqlite` |
| `Class "Product" not found` | Wrong namespace | Add `use App\Models\Product;` |
| `Call to undefined method` | Missing method in controller | Check method name matches route |
| `404 Not Found` | Wrong URL | Check route prefix (`/api/products`) |
| `500 Internal Server Error` | Code syntax error | Check logs: `storage/logs/laravel.log` |
| `CORS header missing` | CORS not configured | Edit `config/cors.php` |
| `Validation failed` | Invalid input data | Check validation rules and data format |

---

## 11. Quick Command Summary

| Command | Description |
|---------|-------------|
| `composer create-project laravel/laravel project-name` | Create new Laravel project |
| `php artisan serve` | Start development server |
| `php artisan make:model Product -m` | Create Model + Migration |
| `php artisan make:controller ProductController --api` | Create API Controller |
| `php artisan make:resource ProductResource` | Create API Resource |
| `php artisan migrate` | Run migrations |
| `php artisan migrate:fresh` | Reset and run migrations |
| `php artisan db:seed --class=ProductSeeder` | Run seeder |
| `php artisan tinker` | Enter interactive shell |
| `php artisan config:clear` | Clear config cache |
| `php artisan route:list` | List all routes |
| `php artisan optimize:clear` | Clear all cache |
| `touch database/database.sqlite` | Create SQLite database |
| `curl -X GET http://127.0.0.1:8000/api/products` | Test API endpoint |

---

## 12. Postman Collection

### Export Postman Collection

Create file `Laravel-13-API.postman_collection.json`:

```json
{
  "info": {
    "name": "Laravel 13 API",
    "description": "Complete API endpoints for Product CRUD",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "GET All Products",
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "http://127.0.0.1:8000/api/products",
          "protocol": "http",
          "host": ["127", "0", "0", "1"],
          "port": "8000",
          "path": ["api", "products"]
        }
      }
    },
    {
      "name": "GET Single Product",
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "http://127.0.0.1:8000/api/products/1",
          "protocol": "http",
          "host": ["127", "0", "0", "1"],
          "port": "8000",
          "path": ["api", "products", "1"]
        }
      }
    },
    {
      "name": "POST Create Product",
      "request": {
        "method": "POST",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n    \"name\": \"New Product\",\n    \"price\": 100000,\n    \"description\": \"Product description\",\n    \"is_active\": true\n}"
        },
        "url": {
          "raw": "http://127.0.0.1:8000/api/products",
          "protocol": "http",
          "host": ["127", "0", "0", "1"],
          "port": "8000",
          "path": ["api", "products"]
        }
      }
    },
    {
      "name": "PUT Update Product",
      "request": {
        "method": "PUT",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n    \"name\": \"Updated Product\",\n    \"price\": 150000\n}"
        },
        "url": {
          "raw": "http://127.0.0.1:8000/api/products/1",
          "protocol": "http",
          "host": ["127", "0", "0", "1"],
          "port": "8000",
          "path": ["api", "products", "1"]
        }
      }
    },
    {
      "name": "DELETE Product",
      "request": {
        "method": "DELETE",
        "header": [],
        "url": {
          "raw": "http://127.0.0.1:8000/api/products/1",
          "protocol": "http",
          "host": ["127", "0", "0", "1"],
          "port": "8000",
          "path": ["api", "products", "1"]
        }
      }
    }
  ]
}
```

---

## ✅ Final Status Summary

| Component | Status |
|-----------|--------|
| PHP 8.2 + Extensions | ✅ |
| Composer | ✅ |
| Laravel 13 Project | ✅ |
| SQLite Database | ✅ |
| Product Model | ✅ |
| Product Migration | ✅ |
| Product Controller | ✅ |
| Product Resource | ✅ |
| API Routes | ✅ |
| CORS | ✅ |
| Seeder | ✅ |
| API Testing | ✅ |
| Postman Collection | ✅ |
| Documentation | ✅ |

---

## 📂 Repository Structure

```
laravel-13-api/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── ProductController.php
│   │   └── Resources/
│   │       └── ProductResource.php
│   └── Models/
│       └── Product.php
├── config/
│   └── cors.php
├── database/
│   ├── database.sqlite
│   ├── migrations/
│   │   └── ..._create_products_table.php
│   └── seeders/
│       └── ProductSeeder.php
├── routes/
│   └── api.php
├── .env
└── README.md
```

---

## 👨‍💻 Author

Faris

---

## 📌 Status

Complete / Ready for Portfolio

---

## 📥 How to Clone & Run This Project

```bash
# Clone project
git clone https://github.com/username/laravel-13-api.git

# Enter directory
cd laravel-13-api

# Install dependencies
composer install

# Copy .env file
cp .env.example .env

# Generate key
php artisan key:generate

# Setup SQLite
touch database/database.sqlite

# Edit .env file for SQLite
# Then run migrations
php artisan migrate

# Run seeder
php artisan db:seed

# Start server
php artisan serve
```

---

**Last Updated:** July 2026
