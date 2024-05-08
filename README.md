# Server Side Tips N Tricks

<details>
<summary>
A social app, handle API for add/remove post from favorites.
</summary>

**What I did**:

- Implemented two distinct endpoints: one for adding posts to favorites and another for removing them.
- Additionally, when removing a post from favorites, if the post wasn't already in favorites, I returned an error message.

**Code Review**:

- Instead of returning error messages when the post isn't found, it's better to simply return without any message, especially to account for scenarios where users may click multiple times.
- For a more simpler and minimal API, consider combining these operations into a single endpoint for toggling between adding and removing posts from favorites.
- _Credits: Saif_

</details>

---

<details>
<summary>
In our app, we aim to show various types of posts like all user posts, a single post, most viewed posts, and posts similar to a specific one.
</summary>

**What I did**: Made individual endpoints for each type of post.

**Code Review**: It's more efficient to create just a couple of endpoints—one for fetching all posts and another for fetching a single post. Then, we can use filters to adjust what we get back based on what the user needs. This not only simplifies the API but also gives us more freedom to adapt to future changes easily.

- _Credits: Saif_

</details>

---

<details>
<summary>
Imagine an app where clients can subscribe to different bundles. We want to keep track of how much of each bundle a client has left and how many times they've used it.
</summary>

**What I did**: Added 'remaining' and 'consumed/used' columns to the pivot table connecting bundles and clients, and if we need the total amount, we add them up.

**Code review:** It might be clearer to use a 'total' column instead of 'used'. Here's why: With the first approach, when a client uses a bundle, we increase the 'remaining' and decrease the 'used'. However, with the other approach, when a client uses a bundle, we only adjust the 'remaining', and when they subscribe to a bundle, we increase the 'total'. This second approach offers less functionality. Then, if we need to find out how many times a bundle has been used, we can calculate it by subtracting the 'remaining' from the 'total'.

- _Credits: Saif_

</details>

---

<details>
<summary>
Continuing from the previous Subscribe to a bundle (without payment).
</summary>

**What I did**: I added a method named `subscribeToBundle` in the Bundle model. It accepts a bundle's ID as a parameter and handles different cases: if the bundle doesn't exist, it returns an error message response; if the client already has the bundle, it returns a response indicating that it was extended; and if there's no existing relation, it returns a response indicating successful subscription.

**Code Review**: It's not recommended to handle responses directly from the model; that's the controller's job. Additionally, it's crucial to keep in mind that this method might be used in various contexts like web or API, where the response format could differ. Hence, it's better to handle responses in the controller to maintain flexibility across different implementations.

- _Credits: Saif_

</details>

---

<details>
<summary>
While working on an old project, I encountered errors after cloning the app and running migrations.
</summary>

**Error 1.0**: A late migration had a foreign key referencing a prior migration, causing an error.
**Solution 1.0**: I renamed the file to ensure it comes before the referenced migration.
**Error 1.1**: I added a check to ensure the table doesn't exist before running the logic.
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

- _Credits: Saif_

</details>

---

<details>
<summary>
Working on a live app, we needed to update the 'name' column to be of JSON type instead of VARCHAR/String.
</summary>

**Error**: Initially attempted to change the type directly, but encountered an error due to existing string values in the column.

**Solution**: Created a new column called new_name, looped through each name and added it to new_name column in JSON format, then deleted named column and renamed new_name to be name, I also learned how to add new_name column directly after name column.

```php
$table->json('new_name')->after('name');
```

- _Credits: Saif_

</details>
