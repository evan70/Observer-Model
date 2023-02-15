
# Laravel Model Observer  - `Observer` 

Observer is an event listening class that will listen and can do action of any changes of that regarding Model. 
That means, when we need to do something after a Model changes or after a 
model event occurs. Then, we can use Observer to listen to that change and 
can do any database operation easily with that model. It's super useful for 
big projects, where you need to do many stuffs after a model is being created.



### Laravel Model Observer - `Step - 1` 
Observer can be created very easily with an Artisan command.
```php
// Demo to Create a UserObserver for User Model

php artisan make:observer UserObserver --model=User
```
### How is an Observer class look like 
```
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * Handle the User "created" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }

    /**
     * Handle the User "updated" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function updated(User $user)
    {
        //
    }

    /**
     * Handle the User "deleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function deleted(User $user)
    {
        //
    }

    /**
     * Handle the User "restored" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function restored(User $user)
    {
        //
    }

    /**
     * Handle the User "force deleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function forceDeleted(User $user)
    {
        //
    }
}

```

### Laravel Model Observer - `Step - 2` 
We need to register this observer in `EventServiceProvider` class inside the `boot()` function of `app/Providers/EventServiceProvider.php`
```php
<?php

namespace App\Providers;

use Illuminate\Auth\Events\Registered;
use Illuminate\Auth\Listeners\SendEmailVerificationNotification;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Event;

use App\Models\User;
use App\Observers\UserObserver;

class EventServiceProvider extends ServiceProvider
{
    /**
     * The event to listener mappings for the application.
     *
     * @var array<class-string, array<int, class-string>>
     */
    protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,
        ],
    ];

    /**
     * Register any events for your application.
     *
     * @return void
     */
    public function boot()
    {
        User::observe(UserObserver::class);
    }

    /**
     * Determine if events and listeners should be automatically discovered.
     *
     * @return bool
     */
    public function shouldDiscoverEvents()
    {
        return false;
    }
}

```

# How to use Observer in a real-life project `project-1`
We need to add a Mr text before user's name when registered. Like after saved to database, we will update user name to something else.

It's very simple to do that. Here is our `created()` function inside `UserObserver`

```
public function created(User $user)
{
    $user->name = "Mr. " . $user->name;
    $user->save();
}
```
### Open tinker terminal -
```
php artisan tinker
```
Now, save a new user :
```
$user = new User();
$user->name = "Akash";
$user->email = "akash@gmail.com";
$user->password = Hash::make('123456');
$user->save();
```
Now check the database. Name is not Akash. Name is - modified to Mr. Akash


# How to use Observer in a real-life project `project-2`
Assume, we're using Facebook. Where every time User updated his profile picture, a new post creates. And we can see that on post feed wall. 
So, we'll implement these things here.

- We'll create a user, and it would create a Post also with user ID and data
- We'll update a user, and it would create another post with updated information
 

First, Change Our UserObserver to this, so that we could add new Post after `created()` and `updated()` event.

```
public function created(User $user)
{
    $user->name = "Mr. ".$user->name;
    $user->save();

    $title = "This post is created by ".$user->name;
    $description = "Post creator name is ".$user->name." and is email is ".$user->email." and post is created at ".$user->created_at;

    Post::create([
        'title' => $title,
        'description' => $description,
        'category_id' => 1,
        'user_id' => $user->id,
    ]);

}

public function updated(User $user)
{
    $title = 'User ' . $user->name . ' Updated successfully';
    $description = 'User ' . $user->name . ' Updated. Updated time - ' . $user->updated_at->diffForHumans();

    Post::create([
        'title'       => $title,
        'description' => $description,
        'user_id'     => $user->id,
        'category_id' => 1,
    ]);
}
```
Before running Artisan command insert, let's check our Post model class to accept fillable property.
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'description',
        'category_id',
        'user_id'
    ];
}
```
Great, all set up is done. Now just do some artisan command to check if Our desired posts added or not -
```
php artisan tinker;

// Create New User
$user = new User();
$user->name = "nohasha";
$user->email = "userhash@gmail.com";
$user->password = Hash::make('123456');
$user->save();

// Update a User
$user = User::orderBy('id', 'desc')->first();
$user->name = "Nohasha Fobia";
$user->save();
```
Great, now check the database, or do the artisan command to check the last posts - 
```
php artisan tinker
```
Then write this command in tinker console : 
```
Post::orderBy('id', 'desc')->limit(2)->get();
```
# These are by default event. There's more we can get in Laravel observer. Let's grab them all.

- creating
- created
- saving
- saved
- deleting
- deleted
- updating
- updated
- restoring
- restored
- forceDeleted
- retreived

# Documented by :  `evan70 - FSskDopz`.
