# Strong Params

## More than parameter that are just strong.

### Why we use them:

Strong parameters allow you to take greater control over the sanitizing user input coming from ubiquitous forms.  On a high-level, every request wraps params in `ActionController::Parameters` which allows you to whitelist specific keys.

Calling them directly will give you an error.

```ruby
params = ActionController::Parameters.new(username: "john")
User.new(params)
# => ActiveModel::ForbiddenAttributesError
```

To get this to work, we need to whitelist the attributes we want.

```ruby
params = ActionController::Parameters.new(username: "john")
User.new(params.permit(:username))
```

If you don't whitelist all of the attributes, it still works but you will get a warning:

```ruby
params = ActionController::Parameters.new(username: "john", password: "secret")
params.permit(:username)
# Unpermitted parameters: password
# => { "username" => "john" }
```

`permit` will not mind if the permitted attribute is missing

```ruby
params = ActionController::Parameters.new(username: "john", password: "secret")
params.permit(:username, :password, :foobar)
# => { "username" => "john", "password" => "secret" }
```

This can be nice, but there are times we do want to require a specific parameter to be present. Here, `require` is used instead of `permit`

```ruby
params.require(:something)
# ActionController::ParameterMissing: param not found: something
```

The one thing that is different about `require` and `permit` is that `require` returns the actual value of the parameter, where `permit` returns the actual hash.

```ruby
params = ActionController::Parameters.new(username: "john")
params.permit(:username)
# => { "username" => "john" }
params.require(:username)
# => "john"
```

This is useful when we have nester params, as we do in forms:

```ruby
params = ActionController::Parameters.new(user: { username: "john" })
params.require(:user)
# => { "username" => "john" }
params.require(:user).permit(:username)
# => { "username" => "john" }

params.require(:user).permitted?
# => false
params.require(:user).permit(:username).permitted?
# => true
```

If you use the first of the permitted calls above, you'll get `ActiveModel::ForbiddenAttributesError`

Now, we can convert our attr_accessible to strong_parameters:

```ruby
# model
class User < ActiveRecord::Base
  attr_accessible :username
end

# controller
def create
  User.create!(params[:user])
end
```

Transforming the code using `strong_parameters`

```ruby
# model
class User < ActiveRecord::Base
end

# controller
def create
  User.create!(params.require(:user).permit(:username))
end
```

If you are using any form of nested attributes, or just simply sending along some nested form data, we need to tell strong_parameters about it. By default permit only allows scalar values.

We have to tell strong_paramter that the key is in an array.

```ruby
params = ActionController::Parameters.new(usernames: ["john", "kate"])
params.permit(usernames: [])
# => { "usernames" => ["john", "kate"] }
```
