# 🛣️ Laravel Routing & Middleware

```markdown
# 🛣️ Laravel Routing & Middleware

## 📑 Table of Contents

- [Introduction](#introduction)
- [What is Routing?](#what-is-routing)
- [Route Files Overview](#route-files-overview)
- [Basic Routing](#basic-routing)
- [HTTP Methods](#http-methods)
- [Route Parameters](#route-parameters)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
- [Route Prefixes](#route-prefixes)
- [Controller Routes](#controller-routes)
- [Resource Routes](#resource-routes)
- [Route Model Binding](#route-model-binding)
- [Fallback Routes](#fallback-routes)
- [Rate Limiting](#rate-limiting)
- [What is Middleware?](#what-is-middleware)
- [Types of Middleware](#types-of-middleware)
- [Creating Custom Middleware](#creating-custom-middleware)
- [Registering Middleware](#registering-middleware)
- [Middleware Parameters](#middleware-parameters)
- [Middleware Groups](#middleware-groups)
- [Middleware Execution Order](#middleware-execution-order)
- [Best Practices](#best-practices)
- [Common Interview Questions](#common-interview-questions)

---

## Introduction

**Routing** is the mechanism by which Laravel maps incoming HTTP requests (URLs) to specific code that handles them. **Middleware** acts as a filtering layer for these HTTP requests entering your application.

Together, they form the **request lifecycle backbone** of every Laravel application.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REQUEST LIFECYCLE OVERVIEW                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Browser ──► public/index.php ──► Kernel ──► Middleware ──► Route   │
│                                                              │      │
│                                                              ▼      │
│  Browser ◄── Response ◄── Middleware ◄── Controller ◄── Route Match │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## What is Routing?

Routing defines **how your application responds** to a client request to a particular endpoint (URI) with a specific HTTP method (GET, POST, etc.).

```
┌──────────────────────────────────────────────────────────────┐
│                     HOW ROUTING WORKS                       │
├──────────────────────────────────────────────────────────────┤
│                                                            │
│   User Request: GET /products/5                            │
│         │                                                  │
│         ▼                                                  │
│   ┌──────────────────────────────────────┐               │
│   │  Laravel checks routes/web.php         │               │
│   │  Route::get('/products/{id}', ...)     │               │
│   └──────────────────────────────────────┘               │
│         │                                                  │
│         ▼  Match found! id = 5                            │
│   ┌──────────────────────────────────────┐               │
│   │  Execute the route handler             │               │
│   │  (Closure or Controller method)        │               │
│   └──────────────────────────────────────┘               │
│         │                                                  │
│         ▼                                                  │
│   Return Response to User                                  │
│                                                            │
└──────────────────────────────────────────────────────────────┘
```

---

## Route Files Overview

Laravel organizes routes into different files based on their purpose:

| File | Purpose | Middleware Applied | URL Prefix |
|------|---------|-------------------|------------|
| `routes/web.php` | Web interface routes | `web` (sessions, CSRF) | none |
| `routes/api.php` | API/stateless routes | `api` (throttle) | `/api` |
| `routes/console.php` | Artisan console commands | none | none |
| `routes/channels.php` | Broadcasting channels | none | none |

```
project-root/
└── routes/
    ├── web.php        ← Browser-based routes (HTML responses)
    ├── api.php        ← API routes (JSON responses)
    ├── console.php    ← Custom Artisan commands
    └── channels.php   ← WebSocket broadcast channels
```

> **📝 Note:** In Laravel 12, route files are registered in `bootstrap/app.php` using the new streamlined configuration.

```php
// bootstrap/app.php

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

---

## Basic Routing

The most basic route accepts a **URI** and a **Closure**.

### 💻 Code Example

```php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Simplest route returning a string
Route::get('/', function () {
    return 'Hello, Laravel 12!';
});

// Route returning a view
Route::get('/home', function () {
    return view('home');
});

// Route returning JSON
Route::get('/data', function () {
    return [
        'framework' => 'Laravel',
        'version' => '12.x',
        'language' => 'PHP',
    ];
});

// Route returning HTML
Route::get('/welcome', function () {
    return '<h1>Welcome to Laravel</h1><p>This is a basic route.</p>';
});
```

### 📤 Output

**Visit:** `http://localhost:8000/`
```
Hello, Laravel 12!
```

**Visit:** `http://localhost:8000/data`
```json
{
    "framework": "Laravel",
    "version": "12.x",
    "language": "PHP"
}
```

---

## HTTP Methods

Laravel supports all standard HTTP verbs for RESTful routing.

```
┌────────────────────────────────────────────────────────────────┐
│                     HTTP METHODS & USAGE                      │
├──────────┬──────────────────────┬──────────────────────────────┤
│  Method  │      Purpose         │         Example Use          │
├──────────┼──────────────────────┼──────────────────────────────┤
│  GET     │  Retrieve data       │  View a page, list items     │
│  POST    │  Create new data     │  Submit a form, create user  │
│  PUT     │  Update entire data  │  Full update of a record     │
│  PATCH   │  Update partial data │  Partial update of a record  │
│  DELETE  │  Remove data         │  Delete a record             │
│  OPTIONS │  Get allowed methods │  CORS preflight requests     │
└──────────┴──────────────────────┴──────────────────────────────┘
```

### 💻 Code Example

```php
// routes/web.php

use Illuminate\Support\Facades\Route;

// GET - Retrieve data
Route::get('/users', function () {
    return 'List all users';
});

// POST - Create data
Route::post('/users', function () {
    return 'Create a new user';
});

// PUT - Full update
Route::put('/users/{id}', function ($id) {
    return "Update entire user {$id}";
});

// PATCH - Partial update
Route::patch('/users/{id}', function ($id) {
    return "Partially update user {$id}";
});

// DELETE - Remove data
Route::delete('/users/{id}', function ($id) {
    return "Delete user {$id}";
});

// Match multiple methods
Route::match(['get', 'post'], '/contact', function () {
    return 'Handles GET and POST requests';
});

// Respond to ALL HTTP methods
Route::any('/api/webhook', function () {
    return 'Handles any HTTP method';
});

// Redirect routes
Route::redirect('/old-page', '/new-page');
Route::redirect('/temp', '/permanent', 301);

// View route (shortcut)
Route::view('/about', 'pages.about', ['title' => 'About Us']);
```

### 🔄 HTTP Method Spoofing

HTML forms only support GET and POST. To use PUT/PATCH/DELETE, use the `@method` directive:

```blade
{{-- resources/views/users/edit.blade.php --}}

<form action="/users/1" method="POST">
    @csrf
    @method('PUT')  {{-- Spoofs PUT method --}}

    <input type="text" name="name" value="John Doe">
    <button type="submit">Update User</button>
</form>

{{-- For DELETE --}}
<form action="/users/1" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">Delete User</button>
</form>
```

---

## Route Parameters

### Required Parameters

```php
// routes/web.php

// Single parameter
Route::get('/user/{id}', function ($id) {
    return "User ID: {$id}";
});

// Multiple parameters
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    return "Post: {$postId}, Comment: {$commentId}";
});
```

### Optional Parameters

```php
// Optional parameter with default value
Route::get('/user/{name?}', function ($name = 'Guest') {
    return "Hello, {$name}!";
});

// Multiple optional parameters
Route::get('/products/{category?}/{id?}', function ($category = 'all', $id = null) {
    if ($id) {
        return "Category: {$category}, Product ID: {$id}";
    }
    return "Showing all products in: {$category}";
});
```

### Regular Expression Constraints

```php
// routes/web.php

// Only numbers allowed
Route::get('/user/{id}', function ($id) {
    return "User ID: {$id}";
})->where('id', '[0-9]+');

// Only alphabetic characters
Route::get('/category/{name}', function ($name) {
    return "Category: {$name}";
})->where('name', '[A-Za-z]+');

// Multiple constraints
Route::get('/posts/{id}/{slug}', function ($id, $slug) {
    return "Post {$id}: {$slug}";
})->where(['id' => '[0-9]+', 'slug' => '[a-z\-]+']);

// Helper methods for common patterns
Route::get('/user/{id}', function ($id) {
    return "User: {$id}";
})->whereNumber('id');

Route::get('/user/{name}', function ($name) {
    return "Name: {$name}";
})->whereAlpha('name');

Route::get('/user/{name}', function ($name) {
    return "Name: {$name}";
})->whereAlphaNumeric('name');

Route::get('/category/{type}', function ($type) {
    return "Type: {$type}";
})->whereIn('type', ['electronics', 'books', 'clothing']);

// UUID constraint
Route::get('/user/{id}', function ($id) {
    return "UUID: {$id}";
})->whereUuid('id');
```

### 📤 Output

**Visit:** `http://localhost:8000/user/42`
```
User ID: 42
```

**Visit:** `http://localhost:8000/user/abc` (with `->whereNumber('id')`)
```
404 | Not Found
```

---

## Named Routes

Named routes allow convenient generation of URLs and redirects.

### 💻 Code Example

```php
// routes/web.php

// Define named routes
Route::get('/user/profile', function () {
    return 'User Profile';
})->name('profile');

Route::get('/posts/{id}', function ($id) {
    return "Post {$id}";
})->name('posts.show');

Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
```

### Using Named Routes

```php
// Generate URLs using route() helper

// Simple named route
$url = route('profile');  
// Output: http://localhost:8000/user/profile

// Named route with parameter
$url = route('posts.show', ['id' => 5]);  
// Output: http://localhost:8000/posts/5

// Named route with multiple parameters
$url = route('posts.show', ['id' => 5, 'extra' => 'value']);  
// Output: http://localhost:8000/posts/5?extra=value

// Redirect to named route
return redirect()->route('dashboard');

// Redirect with parameters
return redirect()->route('posts.show', ['id' => 10]);
```

### In Blade Templates

```blade
{{-- Generate URL in views --}}
<a href="{{ route('profile') }}">My Profile</a>

<a href="{{ route('posts.show', ['id' => $post->id]) }}">
    Read More
</a>

{{-- Check current route --}}
@if(request()->routeIs('dashboard'))
    <span>You are on the dashboard</span>
@endif

{{-- Active link example --}}
<a href="{{ route('dashboard') }}" 
   class="{{ request()->routeIs('dashboard') ? 'active' : '' }}">
    Dashboard
</a>
```

---

## Route Groups

Route groups allow you to share route attributes across multiple routes.

```
┌──────────────────────────────────────────────────────────────┐
│                     ROUTE GROUP ATTRIBUTES                  │
├──────────────────────────────────────────────────────────────┤
│                                                            │
│   ┌────────────────────────────────────────┐             │
│   │  Route::group([...], function () {       │             │
│   │      ┌────────────────────────────┐     │             │
│   │      │  Route 1 (shares attrs)    │     │             │
│   │      │  Route 2 (shares attrs)    │     │             │
│   │      │  Route 3 (shares attrs)    │     │             │
│   │      └────────────────────────────┘     │             │
│   │  });                                     │             │
│   └────────────────────────────────────────┘             │
│                                                            │
│   Shared: middleware, prefix, namespace, name, domain      │
│                                                            │
└──────────────────────────────────────────────────────────────┘
```

### 💻 Code Example

```php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Middleware group
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', function () {
        return 'Dashboard - Auth Required';
    });

    Route::get('/profile', function () {
        return 'Profile - Auth Required';
    });

    Route::get('/settings', function () {
        return 'Settings - Auth Required';
    });
});

// Prefix group
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        return 'Admin Users';  // URL: /admin/users
    });

    Route::get('/posts', function () {
        return 'Admin Posts';  // URL: /admin/posts
    });
});

// Name prefix group
Route::name('admin.')->group(function () {
    Route::get('/admin/dashboard', function () {
        return 'Admin Dashboard';
    })->name('dashboard');  // Route name: admin.dashboard
});

// Controller group (shared controller)
Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
    Route::delete('/orders/{id}', 'destroy');
});

// Combining multiple attributes
Route::middleware(['auth', 'verified'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminController::class, 'index'])
            ->name('dashboard');
        // URL: /admin/dashboard
        // Name: admin.dashboard
        // Middleware: auth, verified
    });

// Domain routing
Route::domain('{account}.example.com')->group(function () {
    Route::get('/', function ($account) {
        return "Account: {$account}";
    });
});
```

---

## Route Prefixes

```php
// routes/web.php

// API versioning example
Route::prefix('api/v1')->group(function () {
    Route::get('/users', function () {
        return 'API v1 Users';  // URL: /api/v1/users
    });

    Route::get('/posts', function () {
        return 'API v1 Posts';  // URL: /api/v1/posts
    });
});

Route::prefix('api/v2')->group(function () {
    Route::get('/users', function () {
        return 'API v2 Users';  // URL: /api/v2/users
    });
});

// Nested prefixes
Route::prefix('admin')->group(function () {
    Route::prefix('users')->group(function () {
        Route::get('/active', function () {
            return 'Active Users';  // URL: /admin/users/active
        });

        Route::get('/banned', function () {
            return 'Banned Users';  // URL: /admin/users/banned
        });
    });
});
```

---

## Controller Routes

```php
// routes/web.php

use App\Http\Controllers\UserController;
use App\Http\Controllers\PostController;

// Single action route
Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{id}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);

// Single Action Controller (Invokable)
Route::get('/report', ReportController::class);

// Controller with all methods grouped
Route::controller(PostController::class)->group(function () {
    Route::get('/posts', 'index');
    Route::get('/posts/{id}', 'show');
    Route::post('/posts', 'store');
    Route::put('/posts/{id}', 'update');
    Route::delete('/posts/{id}', 'destroy');
});
```

### Single Action Controller

```php
// app/Http/Controllers/ReportController.php

<?php

namespace App\Http\Controllers;

class ReportController extends Controller
{
    /**
     * Handle the incoming request.
     * Invokable controller - only one method __invoke()
     */
    public function __invoke()
    {
        return view('reports.index', [
            'title' => 'Monthly Report',
            'data' => ['sales' => 50000, 'users' => 1200],
        ]);
    }
}
```

---

## Resource Routes

Resource routes generate all CRUD routes with a single line.

```
┌─────────────────────────────────────────────────────────────────────────┐
│              RESOURCE ROUTES (Route::resource)                          │
├──────────┬───────────────────────┬─────────────┬─────────────────────────┤
│  Verb    │         URI           │   Action    │     Route Name         │
├──────────┼───────────────────────┼─────────────┼─────────────────────────┤
│  GET     │  /posts               │  index      │  posts.index           │
│  GET     │  /posts/create        │  create     │  posts.create          │
│  POST    │  /posts               │  store      │  posts.store           │
│  GET     │  /posts/{post}        │  show       │  posts.show            │
│  GET     │  /posts/{post}/edit   │  edit       │  posts.edit            │
│  PUT/PATCH│ /posts/{post}        │  update     │  posts.update          │
│  DELETE  │  /posts/{post}        │  destroy    │  posts.destroy         │
└──────────┴───────────────────────┴─────────────┴─────────────────────────┘
```

### 💻 Code Example

```php
// routes/web.php

use App\Http\Controllers\PostController;

// Single resource route generates all 7 CRUD routes
Route::resource('posts', PostController::class);

// Multiple resources
Route::resources([
    'posts' => PostController::class,
    'users' => UserController::class,
    'comments' => CommentController::class,
]);

// API resource (excludes create & edit forms)
Route::apiResource('posts', PostController::class);
// Generates: index, store, show, update, destroy (5 routes)

// Partial resource routes
Route::resource('posts', PostController::class)->only([
    'index', 'show'
]);

Route::resource('posts', PostController::class)->except([
    'destroy'
]);

// Nested resources
Route::resource('posts.comments', CommentController::class);
// URL: /posts/{post}/comments/{comment}

// Naming resource routes
Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build',
    'index' => 'photos.list',
]);
```

### Resource Controller Template

```php
// app/Http/Controllers/PostController.php

<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    // GET /posts
    public function index()
    {
        $posts = Post::all();
        return view('posts.index', compact('posts'));
    }

    // GET /posts/create
    public function create()
    {
        return view('posts.create');
    }

    // POST /posts
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'body' => 'required',
        ]);

        Post::create($validated);
        return redirect()->route('posts.index');
    }

    // GET /posts/{post}
    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    // GET /posts/{post}/edit
    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }

    // PUT/PATCH /posts/{post}
    public function update(Request $request, Post $post)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'body' => 'required',
        ]);

        $post->update($validated);
        return redirect()->route('posts.show', $post);
    }

    // DELETE /posts/{post}
    public function destroy(Post $post)
    {
        $post->delete();
        return redirect()->route('posts.index');
    }
}
```

### Viewing All Routes

```bash
# List all registered routes
php artisan route:list

# Filter by name
php artisan route:list --name=posts

# Filter by method
php artisan route:list --method=GET

# Show only specific path
php artisan route:list --path=api
```

**Output:**
```
GET|HEAD   /posts ............... posts.index › PostController@index
POST       /posts ............... posts.store › PostController@store
GET|HEAD   /posts/create ........ posts.create › PostController@create
GET|HEAD   /posts/{post} ........ posts.show › PostController@show
PUT|PATCH  /posts/{post} ........ posts.update › PostController@update
DELETE     /posts/{post} ........ posts.destroy › PostController@destroy
GET|HEAD   /posts/{post}/edit ... posts.edit › PostController@edit
```

---

## Route Model Binding

Laravel can automatically inject model instances directly into your routes.

### Implicit Binding

```php
// routes/web.php

use App\Models\Post;

// Laravel automatically fetches Post by ID
Route::get('/posts/{post}', function (Post $post) {
    return $post;  // Auto-fetched from database
});

// In controller
Route::get('/posts/{post}', [PostController::class, 'show']);
```

```php
// Controller method
public function show(Post $post)
{
    // $post is already fetched! No need for Post::find()
    return view('posts.show', compact('post'));
}
```

### Custom Key Binding

```php
// Use 'slug' instead of 'id'
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
// URL: /posts/my-first-post (uses slug column)
```

```php
// In Model - set default route key
// app/Models/Post.php

class Post extends Model
{
    public function getRouteKeyName()
    {
        return 'slug';  // Use slug for all route bindings
    }
}
```

### Explicit Binding

```php
// bootstrap/app.php or RouteServiceProvider

use App\Models\Post;
use Illuminate\Support\Facades\Route;

Route::bind('post', function ($value) {
    return Post::where('slug', $value)->firstOrFail();
});
```

### 📤 Output Comparison

```
┌────────────────────────────────────────────────────────────────┐
│                  WITHOUT vs WITH Model Binding                │
├────────────────────────────────────────────────────────────────┤
│                                                              │
│  WITHOUT (Manual):                                           │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Route::get('/posts/{id}', function ($id) {         │       │
│  │     $post = Post::findOrFail($id);  ← Manual fetch │       │
│  │     return $post;                                  │       │
│  │ });                                                │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  WITH (Automatic):                                           │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Route::get('/posts/{post}', function (Post $post) {│       │
│  │     return $post;  ← Auto-fetched & 404 handled   │       │
│  │ });                                                │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
└────────────────────────────────────────────────────────────────┘
```

---

## Fallback Routes

Define a route to handle unmatched requests.

```php
// routes/web.php

// Must be defined LAST in your routes file
Route::fallback(function () {
    return response()->view('errors.404', [], 404);
});

// Or simple message
Route::fallback(function () {
    return 'Page not found. Please check the URL.';
});
```

---

## Rate Limiting

Protect routes from abuse by limiting request frequency.

### 💻 Code Example

```php
// routes/web.php

// Built-in throttle middleware
Route::middleware('throttle:60,1')->group(function () {
    // 60 requests per 1 minute
    Route::get('/api/data', function () {
        return 'Rate limited data';
    });
});

// Custom rate limiter definition
// bootstrap/app.php or AppServiceProvider

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Http\Request;

RateLimiter::for('api', function (Request $request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});

RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->isPremium()
        ? Limit::none()
        : Limit::perMinute(10);
});

// Using named rate limiter
Route::middleware('throttle:api')->group(function () {
    Route::get('/api/users', [UserController::class, 'index']);
});
```

### 📤 Output (When Rate Limit Exceeded)

```json
{
    "message": "Too Many Attempts.",
    "exception": "Illuminate\\Http\\Exceptions\\ThrottleRequestsException"
}
```

**Response Headers:**
```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
```

---

## What is Middleware?

Middleware provides a **filtering mechanism** for HTTP requests entering your application. Think of it as a series of "layers" that requests pass through.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MIDDLEWARE FLOW (ONION MODEL)                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│      Request ──►                                    ◄── Response         │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────┐      │
│   │  Middleware 1 (e.g., Authenticate)                          │      │
│   │  ┌───────────────────────────────────────────────────────┐  │      │
│   │  │  Middleware 2 (e.g., CheckRole)                       │  │      │
│   │  │  ┌─────────────────────────────────────────────────┐  │  │      │
│   │  │  │  Middleware 3 (e.g., LogRequest)               │  │  │      │
│   │  │  │  ┌───────────────────────────────────────────┐  │  │  │      │
│   │  │  │  │                                           │  │  │  │      │
│   │  │  │  │         CONTROLLER / ROUTE HANDLER        │  │  │  │      │
│   │  │  │  │                                           │  │  │  │      │
│   │  │  │  └───────────────────────────────────────────┘  │  │  │      │
│   │  │  └─────────────────────────────────────────────────┘  │  │      │
│   │  └───────────────────────────────────────────────────────┘  │      │
│   └─────────────────────────────────────────────────────────────┘      │
│                                                                         │
│   Request flows IN (top to bottom), Response flows OUT (bottom to top)   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Real-World Analogy

```
┌──────────────────────────────────────────────────────────┐
│            AIRPORT SECURITY ANALOGY                     │
├──────────────────────────────────────────────────────────┤
│                                                        │
│   Passenger (Request)                                   │
│        │                                                │
│        ▼                                                │
│   🎫 Check Ticket    ──► Middleware 1 (Authenticate)   │
│        │                                                │
│        ▼                                                │
│   🪪 Verify ID       ──► Middleware 2 (Verify)         │
│        │                                                │
│        ▼                                                │
│   🔍 Security Scan   ──► Middleware 3 (CheckRole)      │
│        │                                                │
│        ▼                                                │
│   ✈️ Board Plane     ──► Controller (Destination)      │
│                                                        │
└──────────────────────────────────────────────────────────┘
```

---

## Types of Middleware

```
┌────────────────────────────────────────────────────────────────┐
│                     MIDDLEWARE TYPES                          │
├──────────────────┬─────────────────────────────────────────────┤
│      Type        │              Description                    │
├──────────────────┼─────────────────────────────────────────────┤
│  Global          │  Runs on EVERY HTTP request                 │
│  Route/Aliased   │  Assigned to specific routes                │
│  Group           │  Bundled middleware (web, api)              │
│  Terminable      │  Runs AFTER response sent to browser        │
└──────────────────┴─────────────────────────────────────────────┘
```

### Built-in Middleware in Laravel

| Middleware | Alias | Purpose |
|-----------|-------|---------|
| `Authenticate` | `auth` | Verify user is logged in |
| `EnsureEmailIsVerified` | `verified` | Check email verification |
| `Authorize` | `can` | Check authorization/permissions |
| `RedirectIfAuthenticated` | `guest` | Redirect logged-in users |
| `ThrottleRequests` | `throttle` | Rate limiting |
| `ValidateSignature` | `signed` | Validate signed URLs |
| `SubstituteBindings` | `bindings` | Route model binding |

---

## Creating Custom Middleware

### Step 1: Generate Middleware

```bash
php artisan make:middleware CheckAge
```

This creates `app/Http/Middleware/CheckAge.php`

### Step 2: Implement Logic

```php
// app/Http/Middleware/CheckAge.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckAge
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        // BEFORE middleware - logic before request reaches controller
        if ($request->age < 18) {
            return redirect('/home')
                ->with('error', 'You must be 18 or older.');
        }

        // Pass request to next middleware/controller
        return $next($request);
    }
}
```

### Before vs After Middleware

```php
// BEFORE Middleware - acts before request handling
class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action BEFORE request handled
        Log::info('Request received: ' . $request->path());

        return $next($request);
    }
}

// AFTER Middleware - acts after request handling
class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Perform action AFTER request handled
        $response->headers->set('X-Custom-Header', 'Value');

        return $response;
    }
}
```

### Practical Examples

#### Example 1: Log Request Middleware

```php
// app/Http/Middleware/LogRequests.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogRequests
{
    public function handle(Request $request, Closure $next): Response
    {
        Log::info('Incoming Request', [
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'timestamp' => now(),
        ]);

        return $next($request);
    }
}
```

#### Example 2: Check Admin Role

```php
// app/Http/Middleware/EnsureUserIsAdmin.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!$request->user() || !$request->user()->is_admin) {
            abort(403, 'Unauthorized. Admin access required.');
        }

        return $next($request);
    }
}
```

#### Example 3: Maintenance Mode

```php
// app/Http/Middleware/CheckMaintenance.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckMaintenance
{
    public function handle(Request $request, Closure $next): Response
    {
        if (config('app.maintenance_mode')) {
            // Allow admins through
            if (!$request->user()?->is_admin) {
                return response()->view('maintenance', [], 503);
            }
        }

        return $next($request);
    }
}
```

#### Example 4: Set Locale (Language)

```php
// app/Http/Middleware/SetLocale.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\App;
use Symfony\Component\HttpFoundation\Response;

class SetLocale
{
    public function handle(Request $request, Closure $next): Response
    {
        $locale = $request->header('Accept-Language', 'en');

        if (in_array($locale, ['en', 'es', 'fr', 'de'])) {
            App::setLocale($locale);
        }

        return $next($request);
    }
}
```

---

## Registering Middleware

In **Laravel 12**, middleware is registered in `bootstrap/app.php`.

### Global Middleware

```php
// bootstrap/app.php

use App\Http\Middleware\LogRequests;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Runs on EVERY request
        $middleware->append(LogRequests::class);

        // Or prepend to run first
        $middleware->prepend(LogRequests::class);
    })
    ->create();
```

### Route Middleware (Aliases)

```php
// bootstrap/app.php

use App\Http\Middleware\EnsureUserIsAdmin;
use App\Http\Middleware\CheckAge;

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Register aliases
        $middleware->alias([
            'admin' => EnsureUserIsAdmin::class,
            'age' => CheckAge::class,
        ]);
    })
    ->create();
```

### Applying Middleware to Routes

```php
// routes/web.php

// Single middleware
Route::get('/admin', function () {
    return 'Admin Panel';
})->middleware('admin');

// Multiple middleware
Route::get('/dashboard', function () {
    return 'Dashboard';
})->middleware(['auth', 'verified']);

// Using class directly
Route::get('/profile', function () {
    return 'Profile';
})->middleware(EnsureUserIsAdmin::class);

// Middleware on groups
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/users', [AdminController::class, 'users']);
    Route::get('/admin/settings', [AdminController::class, 'settings']);
});

// Exclude middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/public', function () {
        return 'Public';
    })->withoutMiddleware(['auth']);
});
```

### Middleware in Controllers

```php
// app/Http/Controllers/AdminController.php

<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class AdminController extends Controller implements HasMiddleware
{
    // Laravel 12 way of defining middleware in controllers
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('admin', only: ['destroy']),
            new Middleware('verified', except: ['index']),
        ];
    }

    public function index() { /* ... */ }
    public function destroy() { /* ... */ }
}
```

---

## Middleware Parameters

Pass additional parameters to middleware.

### 💻 Code Example

```php
// app/Http/Middleware/CheckRole.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CheckRole
{
    /**
     * Handle with role parameter
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (!$request->user() || !$request->user()->hasRole($role)) {
            abort(403, "Requires {$role} role.");
        }

        return $next($request);
    }
}
```

### Passing Parameters

```php
// routes/web.php

// Single parameter
Route::get('/admin', function () {
    return 'Admin Area';
})->middleware('role:admin');

// Multiple parameters
Route::get('/editor', function () {
    return 'Editor Area';
})->middleware('role:editor,author');
```

```php
// Middleware accepting multiple roles
public function handle(Request $request, Closure $next, string ...$roles): Response
{
    foreach ($roles as $role) {
        if ($request->user()->hasRole($role)) {
            return $next($request);
        }
    }

    abort(403, 'Insufficient permissions.');
}
```

---

## Middleware Groups

Group multiple middleware under a single key.

### Default Groups (Laravel 12)

```
┌────────────────────────────────────────────────────────────────┐
│                    DEFAULT MIDDLEWARE GROUPS                  │
├──────────────┬─────────────────────────────────────────────────┤
│   Group      │              Included Middleware                │
├──────────────┼─────────────────────────────────────────────────┤
│   web        │  • EncryptCookies                               │
│              │  • AddQueuedCookiesToResponse                   │
│              │  • StartSession                                 │
│              │  • ShareErrorsFromSession                       │
│              │  • VerifyCsrfToken                              │
│              │  • SubstituteBindings                           │
├──────────────┼─────────────────────────────────────────────────┤
│   api        │  • SubstituteBindings                           │
│              │  • ThrottleRequests (throttle:api)              │
└──────────────┴─────────────────────────────────────────────────┘
```

### Creating Custom Groups

```php
// bootstrap/app.php

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Define custom middleware group
        $middleware->group('admin', [
            'auth',
            'verified',
            \App\Http\Middleware\EnsureUserIsAdmin::class,
            \App\Http\Middleware\LogAdminActivity::class,
        ]);

        // Append to existing group
        $middleware->web(append: [
            \App\Http\Middleware\SetLocale::class,
        ]);

        // Prepend to api group
        $middleware->api(prepend: [
            \App\Http\Middleware\LogRequests::class,
        ]);
    })
    ->create();
```

### Using Custom Groups

```php
// routes/web.php

Route::middleware('admin')->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'index']);
    Route::get('/admin/users', [AdminController::class, 'users']);
});
```

---

## Middleware Execution Order

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  MIDDLEWARE EXECUTION ORDER                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   REQUEST PHASE (Top → Bottom)                                          │
│   ════════════════════════════                                          │
│                                                                         │
│   1. Global Middleware                                                   │
│        ↓                                                                 │
│   2. Group Middleware (web/api)                                          │
│        ↓                                                                 │
│   3. Route Middleware                                                    │
│        ↓                                                                 │
│   ┌─────────────────────────┐                                          │
│   │   CONTROLLER ACTION     │  ◄── Your code runs here                 │
│   └─────────────────────────┘                                          │
│        ↑                                                                 │
│   RESPONSE PHASE (Bottom → Top)                                         │
│   ═════════════════════════════                                         │
│                                                                         │
│   3. Route Middleware (after)                                           │
│        ↑                                                                 │
│   2. Group Middleware (after)                                           │
│        ↑                                                                 │
│   1. Global Middleware (after)                                          │
│        ↑                                                                 │
│   4. Terminable Middleware (after response sent)                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Forcing Priority

```php
// bootstrap/app.php

return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        // Set priority order
        $middleware->priority([
            \Illuminate\Cookie\Middleware\EncryptCookies::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \App\Http\Middleware\Authenticate::class,
            \App\Http\Middleware\EnsureUserIsAdmin::class,
        ]);
    })
    ->create();
```

### Terminable Middleware

```php
// app/Http/Middleware/TerminableMiddleware.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminableMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Runs AFTER the response is sent to the browser.
     * Perfect for logging, cleanup, analytics.
     */
    public function terminate(Request $request, Response $response): void
    {
        // This runs after user receives response
        \Log::info('Request completed', [
            'url' => $request->fullUrl(),
            'status' => $response->getStatusCode(),
            'duration' => microtime(true) - LARAVEL_START,
        ]);
    }
}
```

---

## Best Practices

### ✅ Do's

| Practice | Description |
|----------|-------------|
| Use named routes | Always name routes for maintainability |
| Group related routes | Use groups for shared attributes |
| Use Route Model Binding | Let Laravel auto-fetch models |
| Keep middleware focused | One responsibility per middleware |
| Use resource routes | For standard CRUD operations |
| Apply rate limiting | Protect APIs from abuse |
| Order middleware correctly | Auth before role checks |

### ❌ Don'ts

| Practice | Description |
|----------|-------------|
| Don't put logic in routes | Keep routes thin |
| Don't skip CSRF protection | Always use `@csrf` |
| Don't hardcode URLs | Use `route()` helper |
| Don't create fat middleware | Split complex logic |
| Don't forget fallback | Handle 404s gracefully |

### Code Examples

```php
// ❌ Bad: Logic in route
Route::get('/users', function () {
    $users = DB::table('users')->where('active', 1)->get();
    foreach ($users as $user) {
        // complex processing
    }
    return view('users', compact('users'));
});

// ✅ Good: Delegate to controller
Route::get('/users', [UserController::class, 'index'])->name('users.index');

// ❌ Bad: Hardcoded URL
return redirect('/dashboard');

// ✅ Good: Named route
return redirect()->route('dashboard');

// ❌ Bad: Manual model fetch
Route::get('/posts/{id}', function ($id) {
    $post = Post::findOrFail($id);
    return $post;
});

// ✅ Good: Route model binding
Route::get('/posts/{post}', function (Post $post) {
    return $post;
});
```

---

## Common Interview Questions

### Q1: What is the difference between `web.php` and `api.php`?

**Answer:**
| Feature | web.php | api.php |
|---------|---------|---------|
| Middleware | `web` group (sessions, CSRF) | `api` group (throttle) |
| State | Stateful (sessions) | Stateless |
| Prefix | none | `/api` |
| Response | HTML/Views | JSON |
| CSRF | Required | Not required |

### Q2: What is Middleware and why is it used?

**Answer:** Middleware is a filtering mechanism that processes HTTP requests before they reach the controller. It's used for:
- Authentication
- Authorization
- Logging
- CORS handling
- Rate limiting
- Request/Response modification

### Q3: What is Route Model Binding?

**Answer:** A feature that automatically injects model instances into routes based on the route parameter. Instead of manually fetching with `Post::find($id)`, Laravel automatically resolves and injects the model, handling 404 errors automatically.

### Q4: Explain Before vs After Middleware.

**Answer:**
- **Before Middleware:** Executes logic BEFORE the request reaches the controller (e.g., authentication checks)
- **After Middleware:** Executes logic AFTER the controller processes the request (e.g., adding response headers)

### Q5: What is the difference between `Route::resource` and `Route::apiResource`?

**Answer:**
- `Route::resource` generates 7 routes (including `create` and `edit` for showing forms)
- `Route::apiResource` generates 5 routes (excludes `create` and `edit` since APIs don't need HTML forms)

### Q6: How does middleware execution order work?

**Answer:** Middleware follows an "onion" model:
1. **Request:** Global → Group → Route middleware → Controller
2. **Response:** Controller → Route → Group → Global middleware
3. **Terminable:** Runs after response is sent

### Q7: What is the purpose of `$next($request)` in middleware?

**Answer:** `$next($request)` passes the request to the next middleware in the stack (or the controller). Code before it runs on the way IN, code after it runs on the way OUT.

---

## Quick Reference: Artisan Commands

```bash
# Create middleware
php artisan make:middleware CheckAge

# List all routes
php artisan route:list

# List routes with middleware
php artisan route:list -v

# Filter routes
php artisan route:list --name=user
php artisan route:list --method=POST
php artisan route:list --path=admin

# Clear route cache
php artisan route:clear

# Cache routes (production)
php artisan route:cache
```

---

## Quick Reference: Route Methods Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                     ROUTE METHODS CHEAT SHEET                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Route::get($uri, $callback)        ── GET requests             │
│  Route::post($uri, $callback)       ── POST requests            │
│  Route::put($uri, $callback)        ── PUT requests             │
│  Route::patch($uri, $callback)      ── PATCH requests           │
│  Route::delete($uri, $callback)     ── DELETE requests          │
│  Route::any($uri, $callback)        ── All methods              │
│  Route::match([...], $uri, $cb)     ── Specific methods         │
│  Route::resource($name, $ctrl)      ── CRUD routes              │
│  Route::apiResource($name, $ctrl)   ── API CRUD routes          │
│  Route::view($uri, $view)           ── Direct view              │
│  Route::redirect($from, $to)        ── Redirect                 │
│  Route::fallback($callback)         ── 404 handler              │
│                                                                 │
│  ->name('route.name')               ── Name the route          │
│  ->middleware(['auth'])             ── Apply middleware         │
│  ->prefix('admin')                  ── URL prefix               │
│  ->where('id', '[0-9]+')            ── Constraint               │
│  ->group(function () {...})         ── Group routes             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 Navigation

- [← Previous: MVC Architecture & Request Flows](./01-MVC-Architecture-Request-Flows/README.md)
- [Next: Eloquent ORM & Relationships →](./03-Eloquent-ORM-Relationships/README.md)

---

> **📝 Note**: This documentation is part of the **Laravel 12 Documentation Repository** - a comprehensive guide covering all Laravel functionalities with practical examples.
```

---

This is your **complete single-file documentation** for **Routing & Middleware**. It includes:

✅ Visual ASCII diagrams (onion model, request flow, airport analogy)  
✅ All routing types (basic, parameters, named, groups, resource)  
✅ Complete middleware coverage (creation, registration, parameters, groups)  
✅ Laravel 12 specific syntax (`bootstrap/app.php`)  
✅ Code examples with outputs  
✅ Comparison tables  
✅ Best practices & interview questions  
✅ Artisan command references  
