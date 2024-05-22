# Server Side Tips N Tricks

<details>
<summary>
1- A social app, handle API for add/remove post from favorites.
</summary>

**What I did**:

- Implemented two distinct endpoints: one for adding posts to favorites and another for removing them.
- Additionally, when removing a post from favorites, if the post wasn't already in favorites, I returned an error message.

**Code Review**:

- Instead of returning error messages when the post isn't found, it's better to simply return without any message, especially to account for scenarios where users may click multiple times.
- For a more simpler and minimal API, consider combining these operations into a single endpoint for toggling between adding and removing posts from favorites.
  \_ _Credits: Saif_

</details>

---

<details>
<summary>
2- In our app, we aim to show various types of posts like all user posts, a single post, most viewed posts, and posts similar to a specific one.
</summary>

**What I did**: Made individual endpoints for each type of post.

**Code Review**: It's more efficient to create just a couple of endpoints—one for fetching all posts and another for fetching a single post. Then, we can use filters to adjust what we get back based on what the user needs. This not only simplifies the API but also gives us more freedom to adapt to future changes easily.

\_ _Credits: Saif_

</details>

---

<details>
<summary>
3- Imagine an app where clients can subscribe to different bundles. We want to keep track of how much of each bundle a client has left and how many times they've used it.
</summary>

**What I did**: Added 'remaining' and 'consumed/used' columns to the pivot table connecting bundles and clients, and if we need the total amount, we add them up.

**Code review:** It might be clearer to use a 'total' column instead of 'used'. Here's why: With the first approach, when a client uses a bundle, we increase the 'remaining' and decrease the 'used'. However, with the other approach, when a client uses a bundle, we only adjust the 'remaining', and when they subscribe to a bundle, we increase the 'total'. This second approach offers less functionality. Then, if we need to find out how many times a bundle has been used, we can calculate it by subtracting the 'remaining' from the 'total'.

\_ _Credits: Saif_

</details>

---

<details>
<summary>
4- Continuing from the previous Subscribe to a bundle (without payment).
</summary>

**What I did**: I added a method named `subscribeToBundle` in the Bundle model. It accepts a bundle's ID as a parameter and handles different cases: if the bundle doesn't exist, it returns an error message response; if the client already has the bundle, it returns a response indicating that it was extended; and if there's no existing relation, it returns a response indicating successful subscription.

**Code Review**: It's not recommended to handle responses directly from the model; that's the controller's job. Additionally, it's crucial to keep in mind that this method might be used in various contexts like web or API, where the response format could differ. Hence, it's better to handle responses in the controller to maintain flexibility across different implementations.

\_ _Credits: Saif_

</details>

---

<details>
<summary>
5- While working on an old project, I encountered errors after cloning the app and running migrations.
</summary>

**Error 1.0**: A late migration had a foreign key referencing a prior migration, causing an error.
**Solution 1.0**: I renamed the file to ensure it comes before the referenced migration.
**Error 1.1**: It led to a production issue because a new migration file was created, attempting to create a table that already existed in the database, causing schema conflicts.
**Solution 1.1**: I added a check to ensure the table doesn't exist before running the logic.

**Error**: Another error occurred in a different migration, and I wasn't sure why.
**Solution**: It was due to an old package updating its migration. I deleted the file, reran the package terminal to generate the new migration file, and added a check at the top to prevent problems with existing tables on the server.

```php
public function up(): void
{
    if (!Schema::hasTable('tableName')) {
      //...
    }
}
```

\_ _Credits: Saif_

</details>

---

<details>
<summary>
6- Working on a live app, we needed to update the 'name' column to be of JSON type instead of VARCHAR/String.
</summary>

**What I did**: Initially attempted to change the type directly, but encountered an error due to existing string values in the column.
So, I found a solution to create a new column called new_name, looped through each name and added it to new_name column in JSON format, then deleted named column and renamed new_name to be name, I also learned how to add new_name column directly after name column.

```php
Schema::table('users', function (Blueprint $table) {
    Schema::table('users', function (Blueprint $table) {
        $table->json('new_name')->after('name')->nullable();
    });

    // During code review, Saif opted for raw SQL queries over ORM
    // for more efficient execution.
    DB::table('users')->update(
      ['new_name' => DB::raw('JSON_OBJECT("en", name, "ar", name)')]
    );

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('name');
        $table->renameColumn('new_name', 'name');
    });
});
```

</details>

---

<details>
<summary>
7- Sometimes your code needs space for better readability.
</summary>

**Before**:

```php
Schema::create('users', function (Blueprint $table) {
  $table->id();
  $table->string('name');
  $table->string('email')->unique();
  $table->timestamp('email_verified_at')->nullable();
  $table->string('password');
  $table->string('phone_e164')->nullable()->unique();
  $table->timestamp('phone_verified_at')->nullable();
  $table->foreignId('country_id')->nullable()->constrained()->nullOnDelete();
  $table->foreignId('city_id')->nullable()->constrained()->nullOnDelete();
  $table->foreignId('area_id')->nullable()->constrained()->nullOnDelete();
  $table->rememberToken();
  $table->timestamps();
});

protected $fillable = [
  'name',
  'email',
  'email_verified_at',
  'password',
  'phone_e164',
  'phone_verified_at',
  'country_id',
  'city_id',
  'area_id',
];
```

**After**:

```php
Schema::create('users', function (Blueprint $table) {
  $table->id();
  $table->string('name');
  $table->string('email')->unique();
  $table->timestamp('email_verified_at')->nullable();

  $table->string('password');

  $table->string('phone_e164')->nullable()->unique();
  $table->timestamp('phone_verified_at')->nullable();

  $table->foreignId('country_id')->nullable()->constrained()->nullOnDelete();
  $table->foreignId('city_id')->nullable()->constrained()->nullOnDelete();
  $table->foreignId('area_id')->nullable()->constrained()->nullOnDelete();

  $table->rememberToken();
  $table->timestamps();
});

protected $fillable = [
  'name',
  'email',
  'email_verified_at',

  'password',

  'phone_e164',
  'phone_verified_at',

  'country_id',
  'city_id',
  'area_id',
];
```

\_ _Credits: Saif_

</details>

---

<details>
<summary>
8- Working on an app where users can make posts, and posts expire after a day of admin approval, and feeds show active posts.
</summary>

**What I did**:

- Added a `status` column to the posts table, handled as an enum where the default is 'pending'.
- Added an `expires_at` column that updates when the admin approves a post.
- Added another enum value to `status` called 'expired'.
- Implemented a cron job that runs every minute to check for 'active' posts that have passed the current time. If found, I changed their status to 'expired'.
- On the feeds page, I display posts where the `status` is `active'.

**Code Review**:

- It's a good approach, but instead of running a cron job, consider returning posts with 'active' status AND `expires_at` is greater than the current time.
- But how to handle user manually expiring a post? - We could set `expires_at` to null.

\_ _Credits: Saif_

</details>

---
