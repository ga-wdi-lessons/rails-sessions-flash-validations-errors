# Flashes / Flash Messages

## Framing (5min, 0:05)

### Messages to the User

It would be nice if we could find a *nice* way let the user know that their
artist was successfully created.  

Q. Based on what we've learned so far, how could you show the user a message
that their Artist was successfully created?

> A. The easiest way would be to just create an instance variable `@message`.

```rb
# app/controllsers/todos_controller.rb
def create
  @todo = Todo.create!(todo_params)
  @message = "Todo was created."
  redirect_to @todo
end
```

Then, in `app/views/todos/show.html.erb`:

```erb
<!-- app/views/todos/show.html.erb -->
Message: <%= @message %>
```

Q. Try it! Why doesn't this work?

> A. Context. We redirect to a different page. A redirect is a separate GET
request. Instance variables exist only on *one* request. They don't persist
across requests.

Q. We've seen two main ways of storing user data so it's available from page to
page, and request to request. What are they?

> A. We could store alerts in the database, but that's hugely inefficient. A
better idea would be to use temporary **session variables**, which you just
learned about.

Rails has a built-in way of storing messages in sessions, called `flash`.

### Flash Lifecycle (5 min, 0:10)

Flashes are brief. `flash` is a hash that's created on one request,
available through the next, then destroyed. They exist only for a single
request cycle. It was created to "flash" a message to the user and then go
away.

Replace the `@message` instance variable with `flash[:notice]` in your Artist
controller to see it in action:

```rb
# app/controllers/todos_controller.rb
def create
  @todo = Todo.create!(todo_params)
  flash[:notice] = "Todo was created."
  redirect_to @todo
end
```

Then, in `views/todos/show.html.erb`, replace the `@message` line with this:

```erb
<!-- views/todos/show.html.erb -->
<% flash.each do |type, message| %>
  <p><%= type %>: <%= message %></p>
<% end %>
```

## [Flash convention](http://gaspull.geeksaresexytech.netdna-cdn.com/wp-content/uploads/2015/07/P1060110.jpg)

We used `flash[:alert]`, but as with any hash, we can use `flash[:notice]`,
`flash[:wombat]`, `flash[:flashy_hashy]`, `flash[:error]`, whatever.

It's convention to stick to two main ones: `:alert` and `:notice`.

Q. Why should you stick to this convention?

> Having one or two flash types makes it really easy to style your Flashes with
CSS.  Also, rails provides special helpers for alert and notice.  In the [controllers](http://guides.rubyonrails.org/action_controller_overview.html#the-flash) and [views](http://api.rubyonrails.org/classes/ActionDispatch/Flash.html).

Consider:

```erb
<!-- views/todos/show.html.erb -->
<% flash.each do |type, message| %>
  <p class="flash <%= type %>"><%= message %></p>
<% end %>
```

And this css:

```css
/* app/assets/stylesheets/application.css */

.flash {
  border: 1px solid black;
  padding: 1em;
  background: #eee;
}

.alert  { color: red;  }
.notice { color: blue; }
```


Q. Presumably, we'd want to show flash messages on any page. What problem do we
run into now?

> A. We only show flash messages on the Todo show page.

Q. Where would be the best place to put the flash messages so we don't have to
repeat anything?

> A. In `views/layouts/application.html.erb`

```erb
<!-- app/views/layouts/application.html.erb -->
<h1>Todos</h1>
<nav>
  <a href="/todos">All Todos</a>
  <a href="/incomplete">Incomplete</a>
</nav>
<% flash.each do |type, message| %>
  <p class="flash <%= type %>"><%= message %></p>
<% end %>
```

## Shorthand Flash (5 min)

You can DRY up your code a bit by putting the flash message right in the
`redirect_to`:

```rb
# app/controllers/todos_controller.rb
def create
  @todo = Todo.create!(todo_params)
  redirect_to @todo, notice: "Todo was successfully saved."
end
```

Note that this only works with flashes named `notice:` or `alert:`. It does not
work with `render`.

## You Do: Add Flash Messages to Tunr

Let the user know that Artists and Songs were successfully created and updated.

Mini-Bonus:
* Include the artist's name in the flash messages for artists
* Include the songs's title in the flash messages for songs
