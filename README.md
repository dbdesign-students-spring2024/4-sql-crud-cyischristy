# SQL CRUD assignment

## Part One：Restaurant Finder

### 1. Restaurants Table Created

+ Create the **restaurants** table which includes eight columns: **restaurant_id** which is also the primary key that list all the restaurants; **restaurant_name** which states the name of each restaurants; **category** which includes different kinds of food gerne the restaurant might belongs to, such as Chinese, Korean, and Thai; **price_tier** which has three levels cheap, medium, and expensive; **neighbourhood** which includes three places uptown, midtown, and downtown of New York City; **opening_hour** which indicates the restaurant's open time, **average rating** for the restaurant with the range from zero to five; and whether it is **good_for_kids** with boolean value True or False.

+ Below is the sql codes that create the restaurants table

```sql
sqlite> CREATE TABLE restaurants(
(x1...> restaurant_id INT PRIMARY KEY NOT NULL,
(x1...> restaurant_name TXT NOT NULL,
(x1...> price_tier TXT NOT NULL,
(x1...> neighbourhood TXT NOT NULL,
(x1...> opening_hour TXT NOT NULL,
(x1...> average_rating INT NOT NULL,
(x1...> good_for_kids BOOLEAN NOT NULL);
```

### 2. Restaurants Mock CSV Data Created

+ [restaurants.csv file we created using website Mockaroo](data/restaurants.csv)

### 3. Imput CSV file into Restaurants Table

+ Below is the sql codes that import the csv file **restaurants.csv** we have inside the computer into sqlite system and use the datas inside the csv file to fill the restaurants table we created in the sql system

```sql
sqlite> .mode csv
sqlite> .import C:\Users\cyisc\Downloads\restaurants.csv restaurants
```

### 4. Reviews Table Created

+ Create the **reviews** table which includes three columns: **review_id** which is also the primary key that list all the reviews; **restaurant_id** which is also a foreign key that referencing the restaurant_id in the restaurants table, establishing a relationship between the two tables; **comment** to input restaurant comments. This design assumes a one-to-many relationship, where one restaurant can have multiple reviews.
  
+ Below is the sql codes that create the reviews table

```sql
sqlite> CREATE TABLE reviews (
(x1...> review_id INTEGER PRIMARY KEY,
(x1...> restaurant_id INTEGER,
(x1...> comment TEXT,
(x1...> FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
(x1...> );
```

### 5. SQL Queries to Perform Specific Tasks for restaurant

- [x] 1. Find all cheap restaurants in a particular neighborhood (Uptown)

```sql
sqlite> SELECT * FROM restaurants WHERE price_tier = 'Cheap' AND neighbourhood = 'Uptown';
```

- [x] 2. Find all restaurants in a particular genre (Chinese) with 3 stars or more, ordered by the number of stars in descending order.

```sql
sqlite> SELECT * FROM restaurants WHERE category = 'Chinese' AND average_rating >= 3 ORDER BY average_rating DESC;
```

- [x] 3. Find all restaurants that are open now (We suppose every restaurants opens eight hours after their openning hour).

```sql
sqlite> SELECT * FROM restaurants WHERE strftime('%H:%M', 'now', 'localtime') BETWEEN time(opening_hour) AND time(opening_hour, '+8 hours');
```

- [x] 4. Leave a review for a restaurant (Leave comment "Taste Good" for the restaurant which id is 1).

```sql
sqlite> INSERT INTO reviews (restaurant_id, comment) VALUES (1, 'Taste Good');
```

- [x] 5. Delete all restaurants that are not good for kids.

```sql
sqlite> DELETE FROM restaurants WHERE good_for_kids = false;
```

- [x] 6. Find the number of restaurants in each NYC neighborhood（Uptown,Midtown,Downtown).

```sql
sqlite> SELECT neighbourhood, COUNT(*) AS num_restaurants
   ...> FROM restaurants
   ...> GROUP BY neighbourhood;
```

## Part Two：Social Media App

### 1. Users Mock CSV Data Created

+ [users.csv file we created using website Mockaroo](data/users.csv)

### 2. Users Table Created and Imput CSV file into Users Table

+ In the below code, we first input users.csv file into sql system and create users table, then we create a new table named new_users which includes all the correct version columns we need to modify users table, we then import the data from the users table into new_users table. In the last step, we delete the old users table and rename new_users table as users table.

```sql
sqlite> .mode csv
sqlite> .import C:\Users\cyisc\Downloads\users.csv users

sqlite> CREATE TABLE new_users(
(x1...> user_id INTEGER PRIMARY KEY NOT NULL,
(x1...> email TEXT NOT NULL UNIQUE,
(x1...> password TEXT NOT NULL,
(x1...> handle TEXT NOT NULL UNIQUE);

sqlite> INSERT INTO new_users (user_id, email, password, handle)
...> SELECT user_id, email, password, handle FROM users;

sqlite> DROP TABLE users;
sqlite> ALTER TABLE new_users RENAME TO users;
```

+ In this way, we finnaly get the **users** table which includes four columns: **user_id** which is also the primary key that list all the users; **email** which records the registered email of each users; **password** includes all users' registered password; **handle** is the users' registered username.

### 3. Post Mock CSV Data Created

+ [post.csv file we created using website Mockaroo](data/post.csv)

### 4. Post Table Created and Imput CSV file into Post Table

+ In the below code, we first input post.csv file into sql system and create post table, then we create a new table named new_post which includes all the correct version columns we need to modify post table, we then import the data from the post table into new_post table. In the last step, we delete the old post table and rename new_post table as post table.

```sql
sqlite> .mode csv
sqlite> .import C:\Users\cyisc\Downloads\post.csv post

sqlite> CREATE TABLE new_post (
(x1...>   post_id INTEGER PRIMARY KEY NOT NULL,
(x1...>   sender_id INTEGER NOT NULL REFERENCES users(user_id),
(x1...>   receiver_id INTEGER REFERENCES users(user_id),
(x1...>   content TEXT NOT NULL,
(x1...>   post_type TEXT NOT NULL,
(x1...>   visibility TEXT NOT NULL,
(x1...>   post_date TEXT NOT NULL,
(x1...>   post_time TEXT NOT NULL);

sqlite> INSERT INTO new_post (post_id, sender_id, receiver_id, content, post_type, visibility, post_date, post_time)
...> SELECT post_id, sender_id, receiver_id, content, post_type, visibility, post_date, post_time FROM post;

sqlite> DROP TABLE post;
sqlite> ALTER TABLE new_post RENAME TO post;
```

+ For all the rows in the table which post_type is "Story", we change their receiver_id coulumn's value to NULL.
  
```sql
sqlite> UPDATE post
...> SET receiver_id = NULL
...> WHERE post_type = 'Story';
```

+ For all the rows in the table which post_date and post_time combination is 24 hours before the current time, we need to hide the post, so we change their visibility column's value to "false".

```sql
sqlite> UPDATE new_post
...> SET visibility = 'false'
...> WHERE ROUND((JULIANDAY('now') - JULIANDAY(post_date || ' ' || post_time)) * 24) >= 24;
```

+ In this way, we finnaly create the **post** table which includes eight columns: **post_id** which is also the primary key that list all the posts; **sender_id** which records the user who send out the post, referencing to users table; **receiver_id** which records the user who receive the post, referencing to users table, for message, receiver_id will be a exact user_id, but for story, receiver_id will be null; **content** includes all the posts users input; **post_type** can be "message" or "story"; **visibility** changes from "true" to "false" after 24 hours the post input; **post_date** and **post_time** list the exact date and time users input the specific post.

### 5. SQL Queries to Perform Specific Tasks for app

- [x] 1. Register a new User(email: cyischristy@gmail.com password: christy handle: cyischristy)

```sql
sqlite> INSERT INTO users (user_id, email, password, handle)
...> VALUES (1001, 'cyischristy@gmail.com', 'christy', 'cyischristy');
```

- [x] 2. Create a new Message "Hello Christy." sent by a user_id 1 to user_id 1001 at 2024-02-27 14:30 (current time)

```sql
sqlite> INSERT INTO post (post_id, sender_id, receiver_id, content, post_type, visibility, post_date, post_time)
...> VALUES (1001, 1, 1001, 'Hello Christy.', 'Message', 'true', '2024-02-27', '14:30');
```

- [x] 3. Create a new Story "Hi everyone, this is Christy!" send by user_id 1001 at 2024-02-27 14:30 (current time).

```sql
sqlite> INSERT INTO post (post_id, sender_id, receiver_id, content, post_type, visibility, post_date, post_time)
...> VALUES (1002, 1001, NULL, 'Hi everyone, this is Christy!', 'Story', 'true', '2024-02-27', '14:35');
```

- [x] 4. Show the 10 most recent visible Messages and Stories, in order of recency.

```sql
sqlite> SELECT *
...> FROM post
...> WHERE visibility = 'true'
...> ORDER BY post_date || ' ' || post_time DESC
...> LIMIT 10;
```

- [x] 5. Show the 10 most recent visible Messages sent by user whose user_id is 1 to user whose user_id is 2, in order of recency.

```sql
sqlite> SELECT *
...> FROM post
...> WHERE visibility = 'true'
...>   AND post_type = 'Message'
...>   AND sender_id = 1
...>   AND receiver_id = 2
...> ORDER BY post_date || ' ' || post_time DESC
...> LIMIT 10;
```

- [x] 6. Make all Stories that are more than 24 hours old invisible.

```sql
sqlite> UPDATE post
...> SET visibility = 'false'
...> WHERE post_type = 'Story'
...>   AND visibility = 'true'
...>   AND ROUND((JULIANDAY('now') - JULIANDAY(post_date || ' ' || post_time)) * 24) > 24;
```

- [x] 7. Show all invisible Messages and Stories, in order of recency.
  
```sql
sqlite> SELECT *
...> FROM post
...> WHERE visibility = 'false'
...> ORDER BY post_date || ' ' || post_time DESC;
```

- [x] 8. Show the number of posts by each User.

```sql
sqlite> SELECT sender_id, COUNT(*) AS post_count
...> FROM post
...> GROUP BY sender_id
...> ORDER BY post_count DESC;
```

- [x] 9. Show the post text and email address of all posts and the User who made them within the last 24 hours.

```sql
sqlite> SELECT post.content, users.email
...> FROM post
...> JOIN users ON post.sender_id = users.user_id
...> WHERE ROUND((JULIANDAY('now') - JULIANDAY(post.post_date || ' ' || post.post_time)) * 24) <= 24
...> ORDER BY post.post_date || ' ' || post.post_time DESC;
```

- [x] 10. Show the email addresses of all Users who have not posted anything yet.

```sql
sqlite> SELECT email
...> FROM users
...> WHERE NOT EXISTS (
(x1...> SELECT 1
(x1...> FROM post
(x1...> WHERE post.sender_id = users.user_id;
```
