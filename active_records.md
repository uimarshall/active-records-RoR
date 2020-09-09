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
