## Laravel Sail MongoDB Setup

**Laravel 11, PHP 8.3**

[You Can Review All The Changes Here](https://github.com/egekibar/laravel-sail-mongodb/commit/e0aea713776d245a598bc16a814527dcff8982ca)

### Step 1 - Laravel Setup
Create a Laravel project. **If you already have a Laravel project, skip this step.**
```shell
composer global require laravel/installer
laravel new project-name
cd project-name
```

### Step 2 - MongoDB Integration
Install and configure Laravel Sail:
```shell
composer install
composer require mongodb/laravel-mongodb --ignore-platform-reqs
php artisan sail:install # choose MySQL during installation
```

#### Update the `.env` file
```dotenv
DB_CONNECTION=mongodb
DB_HOST=mongodb # docker service name
DB_PORT=27017
DB_DATABASE=laravel
DB_USERNAME=sail
DB_PASSWORD=password

SESSION_DRIVER=file # use file or redis for session driver
```

#### Update `docker-compose.yml` file
```yaml
# services:
#    laravel.test:
#        ...
        depends_on:
            - mongodb # replace mysql with mongodb
```

```yaml
# services:
#    ...
    mongodb:
        image: 'mongo:latest'
        ports:
            - '${DB_PORT}:${DB_PORT}'
        volumes:
            - 'sail-mongodb:/data/db'
        environment:
            MONGO_INITDB_ROOT_USERNAME: '${DB_USERNAME}'
            MONGO_INITDB_ROOT_PASSWORD: '${DB_PASSWORD}'
        networks:
            - sail
```

```yaml
# ...
# volumes:
    sail-mongodb: # rename sail-mysql to sail-mongodb
        driver: local
```

#### Publish Sail
```shell
./vendor/bin/sail up -d
./vendor/bin/sail artisan sail:publish
./vendor/bin/sail stop
```

#### In `docker/8.3/Docker` file, add the following:
```dockerfile
#    ...
#    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN yes '' | pecl install -f mongodb

# Uncomment if necessary
# RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.3
# ...
```

#### In `docker/8.3/php.ini` file, add the following:
```dotenv
# ...
# pcov.directory = .
extension=mongodb.so
```

#### Start Sail
```shell
./vendor/bin/sail up -d --build
./vendor/bin/sail composer require mongodb/laravel-mongodb
```

#### Verify MongoDB Extension
```shell
./vendor/bin/sail php -m | grep mongodb
```

#### Update `config/database.php` file:
```php
# 'connections' => [

    'mongodb' => [
        'driver' => 'mongodb',
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', 27017),
        'username' => env('DB_USERNAME'),
        'password' => env('DB_PASSWORD'),
        'database' => env('DB_DATABASE'),
    ],
```

#### Run Migration Command
```shell
./vendor/bin/sail artisan migrate
```

### Step 3 - Model Configuration

Update `app/Models/User` to use MongoDB's Authenticatable class:
```php
# use Illuminate\Foundation\Auth\User as Authenticatable;
use MongoDB\Laravel\Eloquent\Model as Authenticatable;
```

Read more about Eloquent Model Class [here](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/model-class/).

### Step 4 - Testing

Start an interactive shell:
```shell
./vendor/bin/sail tinker
```

#### Create a User
```shell                  
> User::create([ "name" => "Taylor Otwell", "email" => "taylor@laravel.com", "password" => "12345678" ])

= App\Models\User {#5874
    name: "Taylor Otwell",
    email: "taylor@laravel.com",
    #password: "$2y$12$wtfLnviwEio3IPEwz0xeoekHktGrxyn7rrDRUzt3kO4iVFo0gS2YS",
    updated_at: MongoDB\BSON\UTCDateTime {#6093
      +"milliseconds": "1714745297192",
    },
    created_at: MongoDB\BSON\UTCDateTime {#6093},
    _id: MongoDB\BSON\ObjectId {#6097
      +"oid": "6634efd13dce5a120b0042f2",
    },
  }
```

#### Get all Users
```shell
> User::all()

= Illuminate\Database\Eloquent\Collection {#5153
    all: [
      App\Models\User {#5152
        _id:

 MongoDB\BSON\ObjectId {#6148
          +"oid": "6634efd13dce5a120b0042f2",
        },
        name: "Taylor Otwell",
        email: "taylor@laravel.com",
        #password: "$2y$12$wtfLnviwEio3IPEwz0xeoekHktGrxyn7rrDRUzt3kO4iVFo0gS2YS",
        updated_at: MongoDB\BSON\UTCDateTime {#5944
          +"milliseconds": "1714745297192",
        },
        created_at: MongoDB\BSON\UTCDateTime {#5466
          +"milliseconds": "1714745297192",
        },
      },
    ],
  }
```
