# Common Rails Methods
>## **Finders**
---
### **`Model.first`**
* Gets the first model record.
### **`Model.last`**
* Gets the last model record. 
### **`Model.find(int)`**
* Finds a model row based on the id in the models table.
* If we want to check if it exists, we can do this too. Will return <span style="color: #007D53">**nil**</span>.
  * We can only use find if we **KNOW** what the parameter are exactly, can't do **`LIKE`**.
### **`Model.find_by(hash)`**
* Similar to ::find, but we can pass in a hash with corresponding table column values.
    <pre><code>User.find_by(username: 'the_wizard', fav_language: 'ruby')</code></pre>
---
>## **SQL Query Methods in Rails**
---
### **`Model.where(condition)`**
* <span style="color: yellow">WHERE</span> in SQL.
  * Can pass in a condition like this: ex. <span style="color: #88AA00">"price > 10"</span>
  * Can pass in a string into condition: ex. User.where(<span style="color: #88AA00">"name = ?", "Gloria"</span>)
    * SQL ~> SELECT * FROM users WHERE name = 'Gloria';
### **`Model.select(column)`**
* <span style="color: yellow">SELECT</span> in SQL, where parameters are separated by commas.
  * Columns can be strings or symbols.
### **`Model.group(column)`**
* <span style="color: yellow">GROUP BY</span> in SQL.
  * Columns can be a string or symbol.
### **`Model.having(condition)`**
* <span style="color: yellow">HAVING</span> in SQL.
  * Must be a condition with an aggregate or a selected column.
### **`Model.order('column ASC/DESC')`**
* <span style="color: yellow">ORDER</span> BY in SQL.
  * Best to enter a string: ex. <span style="color: #88AA00">"name ASC"</span>
### **`Model.limit(int)`**
* <span style="color: yellow">LIMIT</span> in SQL.
### **`Model.offset(int)`**
* <span style="color: yellow">OFFSET</span> in SQL.
---
>## **Aggregates**
---
### `count()` | `average()` | `maximum()` | `minimum()` | `sum()`
* All these return integer values, and cannot chain rails SQL method on them, so they must be added to the end of a rails SQL query.
* We don't have to pass in arguments into each, but may need to get specific values for which we want a calculation.
  * Product.sum(<span style="color: #88AA00">"price"</span>) => returns the total sum of all products in the database.
* Can be chained onto the #group method.
  * User.group(<span style="color: #88AA00">"city"</span>).count => { 'New York' => 32, 'San Francisco' => 44 }
---
>## **Conditions**
---
### ~> SQL condition statements like <span style="color: orange">**where**</span> and <span style="color: orange">**having**</span> can have conditions passed in.
* Can use <span style="color: orange">**where.not**</span> for the opposite of the condition being passed in.
  * User.where.not(<span style="color: #88AA00">"age < 13"</span>) => returns all users with an age greater than 12.
  * Product.group(<span style="color: #88AA00">"color"</span>).having(<span style="color: #88AA00">"price = 10"</span>) => returns all products grouped by color and are $10.
* Inputs can be passed in as well, but should be sanitized.
  * User.where.not(<span style="color: #88AA00">"age < ?"</span>, user_input)
* Conditions passed in MUST be a <span style="color: #88AA00">**string**</span>, and not a `{hash}` or `:symbol` comparison.
  * <s/>where(:name = 'Bob')
  * having(age: > 12)</s>
---
## **Example Queries**
---
<pre><code>User
  .where(cohorts_taught: 3..5)
  .select(:username, :fav_language)</code></pre>
* Finds all users that have taught between 3 to 5 cohorts, inclusive. Shows their username and favorite language.  
---
<pre><code>User
  .select(:fav_language)
  .group(:fav_language)</code></pre>
* Finds all favorite languages of our users.
---
<pre><code>fav_languages = ["Ruby","JavaScript"]
User
  .where(fav_language: fav_languages)
  .order("username ASC")</code></pre>
* Finds all users who have a favorite language in this list and order by username ascending.

---
>## **Querying with Associations**
---
### **joins**
* `Model.joins() | Model.left_outer_joins()`
  * Both take in association names as parameters
  * Returns ActiveRecord::Relation.
  * Any right joins can be done by flipping the parameter and Model.
  * `users = User.joins(:post)`  
    * SELECT `users.*` FROM `users` JOIN `posts` ON `users.id` = `posts.user_id`
    * SELECT defaults to the Model's table.
* `Model.joins(:foreign_key).where(table: { column: value })`
  * The foreign_key is from the association.
  * The symbol table: is the name of the table.
  * The symbol column: is the name of the column in that particular table.
### **Pluck**
* Pluck returns an Array of attribute values type-casted to match the plucked column names, if they can be deduced.
* Plucking an SQL fragment returns String values by default.
* Triggers an immediate query.
  * `Person.where(age: 21).limit(5).pluck(:id)`  
    SELECT `people.id` FROM `people` WHERE `people.age = 21`  
    => [2, 3] <~ these are the IDs of the people whose age = 21.
  * `Person.pluck(:id, :name)`  
    SELECT `people.id people.name` FROM `people`  
    => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]
---
## Example Queries
---
<pre><code>Song
  .joins(:author)
  .where(users: { username: "the_wizard" })</code></pre>  
* Finds all songs for a particular user <span style="color: #88AA00">"the_wizard"</span>
---
<pre><code>Song
  .joins(:likers)
  .where(users: { fav_language: "JavaScript" })</code></pre>  
* Finds all songs liked by people whose favorite language is JavaScript
---
<pre><code>Song
  .joins(:likers)
  .where("users.fav_language = ?", "JavaScript")
  .distinct</code></pre>  
* Gets only the unique values from the previous entry.
---
<pre><code>Song
  .left_outer_joins(likes)
  .where(likes: { id: nil })</code></pre>  
* Finds all songs with no likes.
  * `likes: { liker_id: nil }` would not be best practice.
---
<pre><code>Song
  .joins(:likes)
  .select(:id, :body, "COUNT(*) AS num_likes")
  .group(:id)</code></pre>  
* Find how many likes each song has.
* If we run the query, it won't return num_likes because the **`Song`** class does not natively have access to values in the **`likes`** table. The values will be there, but not visible in the returned values.
---
<pre><code>Song
  .joins(:likes)
  .group(:id)
  .having("COUNT(*) >= ?", 3)
  .select(:body, "COUNT(*)")</code></pre>  
* Finds songs with at least 3 likes. Shows the body and number of likes. May use pluck.
<pre><code>Song
  .joins(:likes)
  .group(:id)
  .having("COUNT(*) >= ?", 3)
  .pluck(:body, "COUNT(*)")</pre></code>  
* Pluck returns an array:
  * `[[:body1, count_value],[:body2, count_value2],[:body3, count_value3]]`

---
>## **Joins for N+1 Queries**
---
<pre><code>posts = user1
  .posts
  .select("posts.*, COUNT(*) AS comments_count")
  .joins(:comments)
  .group("posts.id")

posts.map do |post|
  [post.title, post.comments_count]
end</code></pre>
#### For some N+1 quesries, includes may not be the best option because it prefetches data and can eat up a lot of memory.

---  
## **Example N+1 Queries and fixes**  
---  
### **<span style="color:red">N+1</span> example:**  
    def self.see_song_authors_n_plus_one
      songs = Songs.all
      songs.each do |song|
        puts song.author.song
      end
    end

### **<span style="color:lime">Optimized</span> with `joins`:**

    def self.see_song_num_likes_optimized
      song_with_likes = Song
        .select("songs.*, COUNT(*) AS num_likes")
        .joins(:likes)
        .group(:id)
      
      songs_with_likes.each do |song|
        puts song.num_likes
      end
    end  

Prefetches the query using rails SQL methods. 
* *Remember, num_likes doesn't visibly appear in that query but it does exist.*  

~> When we use **`includes`**, we get the full entity of the other table and it's accessible in our code.

~> When we use **`joins`**, we get the full QUERY specified in the code, and only have access to that data, and not the full tables.

---