# ACTIVE RECORDS

“anything you can do in SQL, you can do in Active Record”. They mostly use the same terminology as well. Active Record just extends that functionality by giving you a suite of versatile methods (and concepts like Relations) to make it much more user-friendly along the way.

Active records thrive on the concept of **Relations**.

## NOTE

ActiveRecord queries usually return Relations because you’ll run into them often when coding and debugging. The knowledge should make you comfortable chaining query methods together to construct elaborate queries.

If you end up working with a Relation when you really want it to act like an Array, you can sometimes run #to_a on it to force it to evaluate the query.

## Relations and Lazy Evaluation

Using `User.find(1)` will return an unambiguous object – **it’s going to find the user with ID = 1 and give it to you as a Ruby object**. But this behavior is actually unusual. Most queries don’t actually return a Ruby object, they just fake it. For example:

`User.where(id: 1)` will return `[#<User id: 1, email: "foo@bar.com">]`

But try running `User.where(id: 1).class` and you’ll see that it isn’t an `Array`, it’s actually an instance of `ActiveRecord::Relation`. Relations are actually just really good at looking like arrays but they’ve got more going on.

## Learning Outcomes

- What is an ActiveRecord::Relation?

It is the M in MVC architecture in rails.

- What does Lazy Evaluation mean?

Active Record queries return relations(which looks like an array) to be lazy instead of returning an object.
Relations only get executed when it becomes absolutely necessary to know what’s inside them.

- How do you make a relation evaluate into an array?

ActiveRecord queries usually return Relations.
If you end up working with a Relation when you really want it to act like an Array, you can sometimes run `#to_a` on it to force it to evaluate the query.

- How do you check whether a database already contains a record?

The simplest new concept is how to check whether an object actually exists yet or not, which you may want to do before running a method which depends on the object actually having been saved already.

#exists? will return true/false. #any? will be true if any records match the specified criteria and #many? will be true if multiple records match the specified criteria. You can run each of these either on a model directly, a Relation, an association, or a scope

# From the Guide:

# via a model

Post.any?
Post.many?

# via a relation

Post.where(published: true).any?
Post.where(published: true).many?

# via an association

Post.first.categories.any?
Post.first.categories.many?

- Why is #find_by useful and how is it used?

`#find_by` is a really neat method that basically lets you build your own finder method. It’s an alternative to using #where (to which you’d have to add another method like #take or `#first` to pull the result out of the returned array). If you want to find by a user’s email, write `User.find_by(email: 'foo@bar.com')`.

`#select` should be pretty obvious to a SQL ninja like you – it lets you choose which columns to select from the table(s), just like in SQL. To select just the ID column for all users, it’s as simple as `User.select(:id)`. You can also use aliases like in SQL but should use quotes instead of symbols, e.g. `@users = User.select("users.id AS user_id")` will create a new attribute called user_id, e.g. allowing you to access `@users.first.user_id`.

- What’s the difference between what’s returned using a #where query and a #find query?

The key thing to note is that `#find` returns the actual record while #where returns an `ActiveRecord::Relation` which basically acts like an array. So if you’re using `#where` to find a single record, you still need to remember to go into that “array” and grab the first record, e.g. `User.where(email: "foo@bar.com")[0] or User.where(email: "foo@bar.com").first`.

- How do you join tables together in Rails?
- When can you use symbols / hashes and when do you need to use explicit strings for query parameters?
- What are Scopes and why are they useful?
- What needs to happen for a class method to act like a scope?

## Chaining Queries

Relations aren’t just built for speed… they’re also built for flexibility. Let’s say you want to grab the first 5 posts listed in ascending order `(Post.limit(5).order(created_at: :desc))`. Because `#limit` returns a Relation, `#order` takes that relation and adds its own criteria to it.

## Why Care?

You should care that ActiveRecord queries usually return Relations because you’ll run into them often when coding and debugging. The knowledge should make you comfortable chaining query methods together to construct elaborate queries.

If you end up working with a Relation when you really want it to act like an Array, you can sometimes run #to_a on it to force it to evaluate the query.

Methods implemented in ActiveRecord::FinderMethods do NOT return ActiveRecord::Relation objects. The #find, #find_by, #first and #last methods return a single record (a model instance). #take returns an array of model instances. Unlike the methods that return Relation objects, when called, these will run SQL queries immediately.

## Aggregations

Just like with SQL, you often want to group fields together (or “roll up” the values under one header). For example, grouping blog posts written on a certain date. This is most useful when you also apply mathematical operations to them like `#count` or `#max`. An example (a bit more complex because it involves joining two tables) is if I want to get a count of all the blog posts categorized by each tag. I might write something like:

Post.joins(:tags).group("tags.name").count

# => {"tag1" => 4, "tag2" => 2, "tag3" => 5}

## N + 1 Queries

The N + 1 query problem is the classic case of this – you grab all the records for your users (User.all) then loop through each user and call an association it has, like the city the user lives in (user.city). For this example we’re assuming an association exists between User and City, where User belongs_to a City. This might look like:

`User.all.each do |user| puts user.city end`
This is going to result in one query to get all the users, then another query for each user to find its city through the association… so N additional queries, where N is the total number of users. Hence “N+1” problems.
If the best way to make an application run faster is to reduce database calls, we’ve just messed up badly by causing a potentially huge number of them.

Rails is well aware of your distress and has provided a simple solution – “eager loading”. When you first grab the list of all users, you can tell Rails to also grab the cities at the same time (with just one additional query) and store them in memory until you’d like to call upon them. Then user.city gets treated the same way as user.name… it doesn’t run another query. The trick is the `#includes` method.

`#includes` basically takes the name of one or more associations that you’d like to load at the same time as your original object and brings them into memory. You can chain it onto other methods like #where or #order clauses.

Almost as useful is the `#pluck` method, which is covered in the Rails Guide. #pluck lets you skip several steps in the process of pulling up a bunch of records, storing them in memory, then grabbing a specific column and placing it into an array. #pluck just gives you the resulting array right away:

`User.pluck(:name)`

# => ["Foo", "Bar", "Baz", "Jimmy-Bob"]

## Scopes

Scopes are underappreciated, awesome and very simple. A scope is basically a custom chain of ActiveRecord methods that you can slap onto an existing Relation by calling its name like a normal method. It’s easiest to see in an example.

Let’s say you let your user choose to filter your blog posts only for those marked “important”:

# app/models/post.rb

```ruby
scope :important, -> { where(is_important: true) }
```

# app/controllers/posts_controller.rb

```ruby
def index
    if params[:important] == true
    @posts = Post.important.all
    else
    @posts = Post.all
    end
end
```

You might be thinking, Why use a scope when you can write a class method to do the same thing? You can, as long as your class method returns a Relation (which can take some additional thought for edge cases). In fact, using a class method is often best if your logic chains are quite complicated. The example above could be solved using the following class method as well:

# app/models/post.rb

```ruby
def self.important
  self.where(is_important: true)
end
```
