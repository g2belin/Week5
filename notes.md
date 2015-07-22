# Security Tips

Twenty years ago hacking was mostly something people did for fun / learning. As we rely more and more on computers (think: online banking, cars, nuclear power plants) hacking is becoming a serious issue. Identity theft, espionage, even tech warfare is a reality.

Security is something you need to consider at all stages of development. It is not something that you can just add in at the end of a project.

We used the pinterest clone application to demo some common security vulnerabilities. https://github.com/lighthouse-labs/sinatra-pinterest 

Resources:

- Rails security guide: http://guides.rubyonrails.org/security.html

## SQL Injection

How do we protect against SQL Injection attacks?

```ruby
User.find_by(username: params[:username]) # AR applies Regex
User.where("username = ?", params[:username]) # AR applies Regex
User.where("username = #{params[:username]}") # WRONG! Input not sanitized
```

## Mass assignment

### In Sinatra

If you are using Sinatra and have something like this in your `actions.rb`

```ruby
post '/signup' do
  if User.create(params) # <--- Problem. User can now pass in values to the user model that we may not want them to. ex: admin: true
    redirect '/'
  else
    erb :signup
  end
end
```

To fix this you want to explicitly list which parameters you are expecting from the signup form:

```ruby
post '/signup' do
  if User.create(username: params[:username], password: params[:password])
    redirect '/'
  else
    erb :signup
  end
end
```

### In Rails

Rails protects against mass assigment by forcing us to whitelist form params.

If you try to use the params directly, Rails will raise an exception.

```ruby
# app/controllers/users_controller.rb
def create
  User.create(params)
  # raises ActiveModel::ForbiddenAttributesError
end
```

To whitelist the params you have to:

```ruby
# app/controllers/users_controller.rb
def create
  User.create(params.require(:user).permit(:username, :password))
end

```

You can read more about strong params in the Rails Guide, or here: http://blog.trackets.com/2013/08/17/strong-parameters-by-example.html

## Denial of Service

Make a lot of requests that are legitimate to take down a server temporarily.

Typically done using BotNets / Zombie Machines (regular people's computers that are infected)

Worm: Self-propagating virus that is transmitted over computer networks.

The first person in the US to be criminally charged for writing a worm is one of the founders of YCombinator, Robert Morris.

## Buffer Overflow Attack

Not specific to the web. (ssh, ftp, web server (nginx, apache))

Most networked programs in unix operating systems are written in C, because C is fast.

In C, computer programmers have to allocate memory manually when assigning variables. This means a programmer needs to know ahead of time what the maximum value of a variable will be. 

Buffer overflow attacks take advantage of situations where a programmer did not sanitize a variable input to limit it's length. The value of the variable then 'overflows' the allocated memory (buffer) and allows the attacker to run any code they want on the affected computer.

Link: http://www.metasploit.com/  - site that lists known security vulnerabilities

## XSS 

Whenever you display user generated content on your site, you have to make sure you are not allowing people to inject html / javascript into your site. Example: If a user types in `<script type="text/javascript">alert('hello');</script>` into a comment box, what will happen?

### In Sinatra

You have to use the `escape_html` method.

```
<p><%= escape_html(comment.body) %></p> 
```

### In Rails

Content is escaped by default. You can unescape content if you need to actually render user generated content as html (example: if you are building a blogging platform where users can write their own html for articles).

```
<article><%= raw @article.content %></article>
```


