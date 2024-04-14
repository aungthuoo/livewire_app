# Laravel Livewire CRUD
We will learn about Laravel 10 Livewire CRUD operations by developing a complete Laravel 10 Livewire CRUD application with simple and step by step guide.

we will use the 
- `Laravel 10`, 
- `Livewire 3.4` with the 
- `Bootstrap 5.3` version.

If you are building application on Laravel then you do not need to use any front end framework such as `React`, `Angular` or `Vue.js` because Livewire is here to do all stuff that are done by using `React`, `Angular` or `Vue.js`.


## What is Livewire?
Livewire is a `full-stack framework` that works on Laravel framework. It helps to build dynamic user interfaces easily, you can update user interfaces without leaving the comfort of Laravel. In simple words, Livewire perform all AJAX operations and avoid page refresh `without writing JavaScript code`.

## Application Flow:
We will create a simple `products` table in our Laravel Livewire application and perform CRUD operations on it. Product table will store product information such as product name and product description. We will add new product records, view all products records, update some products and also delete some products from database. This is our complete CRUD operations in Laravel using Livewire.

## 1. Install Laravel 10 App
```sh 
cd desktop\workspace
composer create-project --prefer-dist laravel/laravel:^10 livewire_crud

cd livewire_crud
```

## 2. Configure Database Credentials
We need to configure database credentials for Livewire CRUD app. Just update the `.env` file which is available on the application root directory.
Just open the `.env` file and enter your own database credentials details.

```sh
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_db_name
DB_USERNAME=your_db_username
DB_PASSWORD=your_db_password
```

## 3. Create and Update Product Model with Migration
Now, we need to create a product Model and Migration. Just run the below command to create model and migration of product.
```sh
php artisan make:model Product -m
```
In the above command `–m` flag is to create a migration of the product Model.

Now we need to update product migration file, simply go to the directory `livewire_crud\database\migrations` and there you will see a product migration file like below.

```sh 
// YYYY_MM_DD_TIMESTAMP_create_products_table.php
2024_04_14_034523_create_products_table.php
```

Simply open this migration file and paste the following code in it.
```php 
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

Now we will also update the product model file, simply go to the app\Models\Product.php and update the following code in Product.php file.
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
        'description'
    ];
}
```

## 4. Migrate Tables to Database
We have created and updated the product migration, now we need to migrate tables into our database.
Run the below command to migrate all tables into database.
```php 
php artisan migrate
```

## 5. Install Laravel Livewire in our Application
In this step, we will install Livewire in our Laravel application by running the below command.
```php 
composer require livewire/livewire
```

## 6. Create Livewire Layout
After installing Livewire, we will need to create a Livewire layout. Just run the below command in your terminal and it will create the Livewire layout in \resources\views\components\layouts\app.blade.php

```sh
php artisan livewire:layout
```

Now open this layout file `app.blade.php` and update the following code in it.
```html 
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? 'Livewire CRUD Application' }}</title>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-4bw+/aepP/YC94hEpVNVgiZdgIC5+VKNBQNGCHeKRQN+PtmoHDEXuppvnDJzQIu9" crossorigin="anonymous">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css">
    </head>
    <body>
        <div class="container">
            <h3 class="mt-3">Livewire CRUD Application</h3>
        
            {{ $slot }}

        </div>

        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.1/dist/js/bootstrap.bundle.min.js" integrity="sha384-HwwvtgBNo3bZJJLYd8oVXjrBZt8cqVSpeBNS5n7C8IVInixGAoxmnlMuBnhbgrkm" crossorigin="anonymous"></script>
    </body>
</html>
```

## 7. Create Livewire Product Component Class and Blade View Files
Now, we need to create a Livewire product component class and blade view files. Just run the below command in terminal.
```sh
php artisan make:livewire products
```

The above command will create two files, a Livewire component class and livewire blade view files on the following locations:

```sh
app/Livewire/Products.php
```
```sh 
resources/views/livewire/products.blade.php
```

Product component file will work like our controller, we will write all functions of our application in this file. Such as save(), render(), edit(), delete(), cancel(), and resetFields().

Now open the Livewire product component file `app/Livewire/Products.php` and paste the following code in it.
```php 
<?php

namespace App\Livewire;

use Livewire\Attributes\Locked;
use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Product;

class Products extends Component
{
    public $products;

    #[Locked]
    public $product_id;

    #[Validate('required')]
    public $name = '';

    #[Validate('required')]
    public $description = '';

    public $isEdit = false;

    public $title = 'Add New Product';

    public function resetFields()
    {
        $this->title = 'Add New Product';

        $this->reset('name', 'description');

        $this->isEdit = false;
    }

    public function save()
    {
        $this->validate();

        Product::updateOrCreate(['id' => $this->product_id], [
            'name' => $this->name,
            'description' => $this->description,
        ]);

        session()->flash('message', $this->product_id ? 'Product is updated.' : 'Product is added.');

        $this->resetFields();
    }

    public function edit($id)
    {
        $this->title = 'Edit Product';

        $product = Product::findOrFail($id);

        $this->product_id = $id;

        $this->name = $product->name;

        $this->description = $product->description;

        $this->isEdit = true;
    }

    public function cancel()
    {
        $this->resetFields();
    }

    public function delete($id)
    {
        Product::find($id)->delete();

        session()->flash('message', 'Product is deleted.');
    }

    public function render()
    {
        $this->products = Product::latest()->get();

        return view('livewire.products');
    }
}
```
Now open products blade view file `resources/views/livewire/products.blade.php` and update the following code in it.
```php 
<div class="row justify-content-center mt-3">
    <div class="col-md-12">

        @if (session()->has('message'))
            <div class="alert alert-success" role="alert">
                {{ session('message') }}
            </div>
        @endif

        @include('livewire.updateOrCreate')

        <div class="card">
            <div class="card-header">Product List</div>
            <div class="card-body">
                <table class="table table-striped table-bordered">
                    <thead>
                      <tr>
                        <th scope="col">S#</th>
                        <th scope="col">Name</th>
                        <th scope="col">Description</th>
                        <th scope="col">Action</th>
                      </tr>
                    </thead>
                    <tbody>
                        @forelse ($products as $product)
                            <tr wire:key="{{ $product->id }}">
                                <th scope="row">{{ $loop->iteration }}</th>
                                <td>{{ $product->name }}</td>
                                <td>{{ $product->description }}</td>
                                <td>
                                    <button wire:click="edit({{ $product->id }})" 
                                        class="btn btn-primary btn-sm">
                                        <i class="bi bi-pencil-square"></i> Edit
                                    </button>   

                                    <button wire:click="delete({{ $product->id }})" 
                                        wire:confirm="Are you sure you want to delete this product?"
                                        class="btn btn-danger btn-sm">
                                        <i class="bi bi-trash"></i> Delete
                                    </button>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="4">
                                    <span class="text-danger">
                                        <strong>No Product Found!</strong>
                                    </span>
                                </td>
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
        </div>
    </div>    
</div>
```

Create another blade view file in `/resources/views/livewire/` with named `updateOrCreate.blade.php` and paste the following code in it.

/resources/views/livewire/updateOrCreate.blade.php
```sh
<div class="row justify-content-center mt-3 mb-3">
    <div class="col-md-12">

        <div class="card">
            <div class="card-header">
                <div class="float-start">
                    {{ $title }}
                </div>
            </div>
            <div class="card-body">
                <form wire:submit="save">

                    <div class="mb-3 row">
                        <label for="name" class="col-md-4 col-form-label text-md-end text-start">Product Name</label>
                        <div class="col-md-6">
                          <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" wire:model="name">
                            @if ($errors->has('name'))
                                <span class="text-danger">{{ $errors->first('name') }}</span>
                            @endif
                        </div>
                    </div>

                    <div class="mb-3 row">
                        <label for="description" class="col-md-4 col-form-label text-md-end text-start">Product Description</label>
                        <div class="col-md-6">
                            <textarea class="form-control @error('description') is-invalid @enderror" id="description" wire:model="description"></textarea>
                            @if ($errors->has('description'))
                                <span class="text-danger">{{ $errors->first('description') }}</span>
                            @endif
                        </div>
                    </div>
                    
                    <div class="mb-3 row">
                        <button type="submit" class="col-md-3 offset-md-5 btn btn-success">
                             Save
                        </button>
                    </div>

                    @if($isEdit)
                        <div class="mb-3 row">
                            <button wire:click="cancel" 
                                class="col-md-3 offset-md-5 btn btn-danger">
                                Cancel
                            </button>
                        </div>
                    @endif

                    <div class="mb-3 row"> 
                        <span wire:loading class="col-md-3 offset-md-5 text-primary">Processing...</span>
                    </div>
                    
                </form>
            </div>
        </div>
    </div>    
</div>
```

## 8. Define Application Product Route
In this step, we need to define our product component route in `web.php`. just copy and paste the below code in your `routes/web.php` file.
```sh
<?php

use Illuminate\Support\Facades\Route;
use App\Livewire\Products;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/products', Products::class);
```

## 9. Run Laravel Development Server
In this final step, we will run the Laravel development server and test our application. We have finally completed our Laravel 10 Livewire CRUD application.

Just run the below command to start Laravel development server.
```sh
php artisan serve
```

After running application server now open the web browser and enter the below link to test our Laravel 10 Livewire CRUD application.
```sh
http://127.0.0.1:8000/products
```

Output: 
![Screenshot from 2024-04-14 10-23-06](https://github.com/aungthuoo/livewire_app/assets/6800568/8be154b0-bd7a-4cfd-bd03-33adf99c9acf)


Ref : [Laravel 10 Livewire CRUD Application Tutorial](https://www.allphptricks.com/laravel-10-livewire-crud-application-tutorial/?utm_source=pocket_reader)
