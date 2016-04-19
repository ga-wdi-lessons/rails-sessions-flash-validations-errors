# Validations

Flash is especially helpful for correcting the user when they use your app in
the wrong way -- entering bogus data, for example.

## Framing (5 min)

In our Rails apps, we've been entering a lot of bogus data -- artists with blank
names, for instance. That's fine while we're in development, but we don't want
the users of our live app to be able to get away with that.

Q. Based on what we know, how could we prevent users from entering **blank
data** into a field?


> A. Javascript.

...but that's easily circumvented with Chrome's Inspector.

> A. We could put that in the controller, like so:

```rb
# app/controllers/todos_controller.rb
def create
  if params[:todo][:author] == ""
    flash[:alert] = "Author can't be blank!"
    redirect_to @todo
  else
    @todo = Todo.create(todo_params)
    redirect_to @todo
  end
end
  ```

...but putting it in the controller is going to lead to some pretty unwieldy
controller methods. Besides, we may want there to be several routes that create
or update a todo, and we'd need to copy and paste that bit of code for each
one. That's hardly DRY!

> A. We could put in database constraints, like `NOT NULL` and `UNIQUE`, by
going and playing around in SQL.

But...it's SQL.  Rails provides [some
helpers](http://edgeguides.rubyonrails.org/active_record_migrations.html#column-modifiers)
for managing database constraints.  Beyond those, it can be difficult.

Wouldn't it be nice if, before a Todo record was saved, we could validate
that the data entered into its fields were correct?

## Validations Intro

### Exercise: Exploring Validations (20 min)

Spend ~5-10 minutes skimming the
[Rails Guide on Validations](http://guides.rubyonrails.org/active_record_validations.html).

Then, try adding a validation for your Artist model. A suggestion would be that
an artist must have a name.

Use the `rails console` to play with your `Artist` model after adding
validations.

Some suggested things methods to try:

```
# assuming you have a validation for name must be present on artist:
bob = Artist.new(name: "Bob", nationality: "USA")
bob.errors
bob.errors
bob.save
bob.save!
bob.valid?

anon = Artist.new(nationality: "USA")
anon.errors
anon.save
anon.save!
anon.errors
anon.errors.full_messages
anon.valid?

result = Artist.create(nationality: "USA")
Artist.last # was it created?
puts result

Artist.create!(nationality: "USA")
Artist.last # was it created?
```


## How Validations Work

### Declare Validations on your Model(s)

Common validations:

* presence
* uniqueness
* numericality
* length

Example:

```ruby
class Todo < ActiveRecord::Base
  validates :body, presence: true
  validates :author, presence: true
  ## OR
  # validates :body, :author, presence: true

  validates :body, length: { maximum: 100 }
end
```

These will now *ensure* your models pass these validations before

### Validations are 'Triggered' during `save`, `create`, `valid?`, `update`

If you only write these:

```
my_todo = Todo.new(author: "bob")
my_todo.errors.full_messages
```

...you'll get a blank array. That's because `.new` doesn't actually make the
validations happen. The only methods that *do* are:

```
.valid?
.save
.save!
.create
.create!
.update
.update!
```

How is this useful? First, go back to our example earlier where we played with
validations in the console... what was returned from `save` if the object was
valid vs invalid?

> A. `save` returns the object (truthy) if it passes, and `false` if it fails.

How could we use this in our controllers?
We'll come back to this, but we can look at the result of the `save` method to
decide what to show the user.

### The Resulting Instance is Either Valid or Invalid (has `errors`)

Once you've run a method that triggers validations, then you can look at the
`errors` method to examine errors on the object.

Q. How might this be useful?

> A. We could display these to the user.

## Break (10 minutes)

## Putting it All Together

The discussion on validations has so far shown you about 18 different ways a
user can "break" your app.

Truth be told, however, we don't usually use `.create` or `.create!` in Rails
apps. Instead, we use `.new` and `.save`.

Q. What's the difference between `.create` and `.new / .save`?

> A. `.create` is the same thing as `.new` and `.save` run right after each other.

This gives us a way of making sure the user doesn't see a broken app:

```rb
# app/controllers/todos_controller.rb
def create
  @todo = Todo.new(todo_params)
  if @todo.save
    flash[:notice] = "Todo was successfully created."
    redirect_to @todo
  else
    render :new
  end
end
```

...and in the form view:

```erb
<!-- app/views/todos/_form.html.erb -->
<% @todo.errors.full_messages.each do |message| %>
  <p><%= message %></p>
<% end %>
```

This way, if the create was *successful*, then we show the user the todo they just
created, along with a messing informing them it worked.

If it *failed*, then we re-display the form along with any error messages.

## Exercise: Add Validations to Tunr

Make sure you update the controllers to display errors on create / update.

## Bonus Content

## Custom validations (10 min)

You can also easily create your own custom validations. For instance, this will
make Tunr reject any artist named Billy Ray Cyrus:

```rb
# app/models/artist.rb
class Artist < ActiveRecord::Base
  validate :break_billy_rays_achy_breaky_heart

  def break_billy_rays_achy_breaky_heart
    if self.name == "Billy Ray Cyrus"
      errors.add(:name, "cannot be Billy Ray Cyrus, because he doesn't qualify as an artist.")
    end
  end
end
```

Q. Test it out by using the form to create a new artist named "Billy Ray Cyrus".
What does `errors.add` do?

> A. Determines what the error message is.

Q. This validation won't be triggered if the user writes "billy ray cyrus" in
all-lowercase. How could you fix that?

> A. `if self.name.downcase == "billy ray cyrus"`

## Bangin' methods (10 min)

Let's try using `.create` instead of `.new`:

```
$ billy = Artist.create(name: "Billy Ray Cyrus")
```

### Rollback

You should see something like:

```
2.2.1 :024 > billy = Artist.create(name: "Billy Ray Cyrus")
   (0.2ms)  BEGIN
   (0.6ms)  ROLLBACK
 => #<Artist id: nil, name: "Billy Ray Cyrus", photo_url: nil, nationality: nil, created_at: nil, updated_at: nil>
```

That `ROLLBACK` indicates that ActiveRecord tried running a SQL command, but it
was unsuccessful, so it's undoing -- or "rolling-back" -- any changes that it
made.

### With a bang

Now try the same command, but put a bang `!` at the end of `.create`:

```
$ billy = Artist.create!(name: "Billy Ray Cyrus")
```

Q. What's the difference between `.create` and `.create!`?

> A. Adding in a bang makes the app throw a fatal error -- that is, "break" --
if a validation fails. Without the bang, it fails "silently".

You can add a bang to both `.create` and `.save`.


### Pro (debugging) tip

Add `<%= debug(@todo) %>` to "app/views/todos/new.html.erb" to see information
contained in `@todo`.  Try submitting invalid information for a new Artist.


## SimpleForm (10 min)

Ensure your Gemfile contains:
```
gem 'simple_form'
```

`bundle install`, and **restart the server**.

Change your `todos/_form.html.erb` form to use `simple_form_for`:

```erb
<!-- app/views/todos/_form.html.erb -->
<%= simple_form_for @todo do |f| %>
  <%= f.input :author %>
  <%= f.input :body %>
  <%= f.submit %>
<% end %>
```

Finally, make sure your Artist model has a validation in it.

Try creating a new Artist, making sure you fail that validation.

Pretty cool, huh?
