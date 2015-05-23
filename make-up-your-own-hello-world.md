#[fit] Make up your own
#[fit] "Hello, World!"
## @justincampbell

^
My name is Justin.
I'm a self-taught developer,
and Ruby is the first language I've used professionally.
It's the first language where I've had the moral obligation to produce code my teammates could understand,
and it's the first language where I've had to understand code that someone else wrote.

---

# Learn *(verb)*

* gain or acquire knowledge of or skill in (something) by study, experience, or being taught
  * commit to memory
  * become aware of (something) by information or from observation

^
But, I also consider Ruby the first language I've learned.
When I was younger I toyed around with BASIC, and it was part of one of our classes in school.
I've played with JavaScript since I was in high school,
and I wrote websites on the side throughout the 2000's in PHP.
But I never truly learned those languages,
I just cargo-culted code I found elsewhere until the thing I was trying worked.

---

![](http://cl.ly/image/0C3w3l100P3G/ruby.png)

^
So,
Ruby is really what I consider the first language that I've learned by committing myself to.
And because of that,
I have (I think) a good understanding of how Ruby works,
and also object-oriented design in general.

---

![](http://cl.ly/image/0I1z1n2z2v1R/languages.png)

^
I decided I wanted to learn new languages because I kept hearing and reading people talk about them.
What they said seemed so different from what I was used to.
Why is everyone talking about functional programming?
How could Erlang and Elixir possibly be more robust by letting things fail?
How could Haskell be pure with no side effects and do anything useful?
Why would anyone ever use Go or Rust or even C? Ruby seems fine for that stuff.
People keep saying Ruby is slow, is that really true? It doesn't feel *too* slow.
Every day on Reddit, Twitter, and Lobste.rs there are hundreds of posts about languages and frameworks I've never used, and that's what got me interested:
I wanted to know what else was out there; I wanted to learn.

---

#[fit] How?

^
So I installed some languages.
I think the first one I played with was Clojure.
I made a dice rolling API that returned JSON arrays of random numbers.

---

![fit](http://i01.i.aliimg.com/wsphoto/v1/1156738381_2/toy.jpg)

^
But, toying with a language randomly didn't seem effective, at least for me.
It was very reminiscent of installing Linux/BSD when I was younger.

---

![fit](http://www.absolutelinux.org/installing/images/lilo_sel_linux_partition.png)

^
I would download an ISO, install it, and then get bored and forget about it.
I never had a use for Linux at the time, and I never had a goal of what I wanted to learn.
I just compulsively installed it on a whim

---

#[fit] I *need* a project

^
I realized that I needed a project
This may seems obvious to some people but it was a big deal for me:
I needed a goal.
What was I trying to accomplish?
So I thought about what came to mind when I thought of programming.

---

#[fit] HTTP

^
Most projects I work on are on the internet.
They're either a web site, or an API for some client.
And behind those websites and public APIs,
there are other services being used over HTTP.
So I think HTTP support is important to me when evaluating a language.

---

#[fit] Writes

^
But, if I just have an HTTP endpoint that responds with data
and I can't write to it,
it's essentially just a static website.
So it would be nice if I could write some data to the server.

---

#[fit] State

^
And if I'm writing to the server,
it has to store the information I gave it somewhere.
One thing I don't want to do is use an external database.
Yes, I would probably use a database such as Postgres, Redis, et cetera in a real app,
but if I'm trying to learn many languages,
maintaining a SQL database would be a lot of work.
Redis would be easier, but still is an external dependency.
So I want to keep the state of the world in memory somewhere,
and that also forces me to think about concurrency.

---

#[fit] Testing

^
And last but not least,
testing is very important to me

---

#[fit] TDD

^
And for me that means TDD.
I'm most comfortable writing code when I have tests that cover it.
Mostly because I'm lazy.
If I have to start up an application
and manually make requests to it to make sure it functions correctly,
that sounds like a lot of work.

---

# Goals

* HTTP
* Writes
* Shared State
* TDD

^
So these are the things I wanted out of a project

---

* Billboard
* Blog
* Chat
* Counter
* Dice
* Stats
* Storage
* Twitter
* URL Shortener

![right](http://i.imgur.com/njhQX.gif)

^
So I wrote down different things that came to mind that
would satisfy those goals and tried to pick one

---

#[fit] URL
#[fit] Shortener

^
The project I came up with is a URL shortener.
It's an HTTP API that will return a "token" for a URL,
and then if you request that path, it will redirect you to the URL.
The tokens start at 1 and increment to keep things simple.
but if I were really building a URL shortener,
I might encode or deterministically randomize the tokens in some way.

---

# Shorten

```
POST / "url=http://justincampbell.me" -------> ,--------,
                                               | Server |
201 Created <--------------------------------- '--------'
/1
```

---

# Expand

```
GET /1 --------------------------------------> ,--------,
                                               | Server |
301 Redirect <-------------------------------- '--------'
Location: http://justincampbell.me
```

---

# Missing

```
GET /54321 ----------------------------------> ,--------,
                                               | Server |
404 Not Found <------------------------------- '--------'
```

^
So I'd like to touch on a few code samples of this in a few languages,
and the things that I learned from them.

---

#[fit] Ruby

^
To implement this in Ruby, here's a summary of the code I came up with.

---

```ruby
# Ruby
post '/' do
  token = shortener.shorten(params[:url])

  [201, "/#{token}"]
end
```

^
First, using Sinatra,
to shorten a URL we grab the URL from the params and ask a shortener object to shorten it.
This method returns the token,
and we tell the user what the token or path is, with a 201 created

---

```ruby
# Ruby
post '/' do
  token = shortener.shorten(params[:url])

  [201, "/#{token}"]
end

get '/:token' do |token|
  full_url = shortener.expand(token)

  halt 404 unless full_url

  redirect full_url
end
```

^
Next, to use the token, the user makes a request for the token.
We lookup the token with the `expand` method,
which returns the full URL or nil.
If we couldn't find that token, we 404.
Otherwise, we redirect the user to the full URL.

---

```ruby
# Ruby
class Shortener
  def shorten(url)
    token = id_generator.next.to_s
    urls[token] = url
    token
  end

  def expand(token)
    urls[token]
  end
end
```

^
Inside of the Shortener object, we have 2 methods to shorten and expand.
To shorten, we generate a new token and store it in our hash of URLs,
and to expand, we look up the token in the hash

---

```ruby
# Ruby
class Shortener
  def id_generator
    @id_generator ||= (1..Float::INFINITY).enum_for
  end

  def urls
    @urls ||= Hash.new
  end
end
```

^
The id_generator is an enumerator that yields sequential numbers,
and the urls method is just a memoized hash

---

```ruby
# Ruby
def shortener
  Shortener.instance
end

class Shortener
  def self.instance
    @instance ||= new
  end
end
```

^
And to use the Shortener class from Sinatra,
we use the singleton pattern with a class method called `instance`
that memoizes an instance of a Shortener,
so that every time we ask for the instance we get the same one back

---

```ruby
class Shortener
  def self.instance
    @instance ||= new
  end

  def id_generator
    @id_generator ||= (1..Float::INFINITY).enum_for
  end

  def urls
    @urls ||= Hash.new
  end
end
```

^
Is this code thread-safe?
If I run this code in JRuby or Rubinius, will I get missing URLs or token collisions,
because two things are trying to update these objects at the same time?
Or if I use Puma or Unicorn?
Should I have wrapped that data in a mutex?
TODO funny pic

---

![](https://cdn0.gamesports.net/storage/16000/16987.jpg)

^
I have absolutely no idea,
but I know enough to be skeptical.

---

# Clojure

![original](http://i.ytimg.com/vi/dGVqrGmwOAw/hqdefault.jpg)

^
Let's look at Clojure

---

```clojure
; Clojure
(def token
  (atom 0))
```

^
Clojure has a these things called atoms.
You create an atom and give it an initial state,
so for this we start creating tokens at 0.

---

```clojure
; Clojure
(def token
  (atom 0))

(defn next-token []
  (swap! token inc))
```

^
Then, you can use the `swap!` function to send a function into the atom to update it's value.
Here, we're using the `inc` function, which increments a number.
You can think of an atom as wrapping a value,
and it has a queue of operations to perform on that value,
and it guarantees that they happen one at a time.

---

```clojure
; Clojure
(def token
  (atom 0))

(defn next-token []
  (swap! token inc))

(next-token) ; => 1
```

^
So if we then call the `next-token` function,
it tells `token` to increment by one and return itself,
and we get 1 back.

---

```clojure
; Clojure
(def token
  (atom 0))

(defn next-token []
  (swap! token inc))

(next-token) ; => 1
(next-token) ; => 2
```

^
And then 2 and so on.
I love how little code there is to guarantee safe concurrent access to the token.
It's one function to wrap the value,
and the `swap!` function is in place of whatever you would normally write there.
So about 4-6 characters if you count parentheses.
Is this code thread-safe? Yes.

---

![original](https://pbs.twimg.com/media/B-seUr9UIAA0y1X.jpg)

^
Let's look at Haskell.
Haskell was really intimidating when I first looked at the syntax.

---

```ruby
# Ruby
def shorten(url)
  token = id_generator.next.to_s
  urls[token] = url
  token
end
```

^
Here's out shorten method in Ruby again.
The input to our function is a URL.
We do some stuff in the background,
including incrementing the token and storing the URL.
And then we return the token.

---

```ruby
# Ruby
def shorten(url)
  token = id_generator.next.to_s
  urls[token] = url
  token
end
```

```haskell
{- Haskell -}
shorten :: Url -> Token
```

^
Haskell is a statically typed language,
and this is what a type signature looks like.
These are optional;
the compiler will infer most types for you,
but it's idiomatic to add them.
Anywhere you see double-colon, you can say "is a",
and anywhere you see a right-arrow, it's a function between two types.

---

```ruby
# Ruby
def shorten(url)
  token = id_generator.next.to_s
  urls[token] = url
  token
end
```

```haskell
{- Haskell -}
shorten :: Url -> Token
{- shorten is a function from a Url to a Token -}
```

^
So you can read this as "Shorten is a function from a Url to a Token".

---

```ruby
# Ruby
def shorten(url)
  token = id_generator.next.to_s
  urls[token] = url
  token
end
```

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> Token
```

^
And Url and Token are aliases for String,
but I find it helpful to be explicit about the meaning of the string.

---

```ruby
# Ruby
def shorten(url)
  token = id_generator.next.to_s
  urls[token] = url
  token
end
```

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> Token
shorten url = token where
    {- increment token, store url -}
```

^
And this is what an implementation of the `shorten` function looks like, kind of...
If we try to copy the inputs and outputs of the Ruby implementation,
we run into a wall.
Haskell is a "pure" language,
which means it doesn't have side-effects,
such as changing a variable in some other part of the code.
And, there are no objects to hold instance variables, and no idea of `self`.

---

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> Token
```

^
In order to shorten a URL in a pure manner,

---

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> UrlStore -> Token
```

^
we have to also pass in the state of the world:
in this case I'm calling it `UrlShore`.
And then the output of our function,
if it updates the world,

---

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> UrlStore -> UrlStore
```

^
will also have to include the new state of the world,

---

```haskell
{- Haskell -}
type Token = String
type Url = String

shorten :: Url -> UrlStore -> UrlStore
lastToken :: UrlStore -> Token
```

^
and then I can use another function to get the last token from the `UrlStore`.

---

```haskell
{- Haskell -}
type Token = String
type Url = String

data UrlStore = UrlStore { id :: Int,
                           urls :: Map Token Url }

shorten :: Url -> UrlStore -> UrlStore
```

^
We make a new data type called "UrlStore",
and it has an id, and a map (or hash) of tokens to urls.

---

```haskell
{- Haskell -}
type Token = String
type Url = String

data UrlStore = UrlStore { id :: Int,
                           urls :: Map Token Url }

shorten :: Url -> UrlStore -> UrlStore
shorten url world = newUrlStore where
        nextId = succ $ id world
        token = show nextId
        newUrls = insert token url $ urls world
        newUrlStore = UrlStore nextId newUrls
```

^
And then we can write our function,
that only operates on the data that was passed into it,
and only returns a new `UrlStore`,
and doesn't affect anything outside of itself.

---

```haskell
{- Haskell -}
type Token = String
type Url = String

data UrlStore = UrlStore { id :: Int,
                           urls :: Map Token Url }

shorten :: Url -> UrlStore -> UrlStore
```

^
And if we get rid of the implementation,

---

```haskell
{- Haskell -}
type Token = String
type Url = String

data UrlStore = UrlStore { id :: Int,
                           urls :: Map Token Url }

shorten :: Url -> UrlStore -> UrlStore
expand :: Token -> UrlStore -> Maybe Url
```

^
and add the type signature for an `expand` function,
without even seeing the code for the functions,
you can tell what they do.
Not only can you tell,
but the Haskell compiler *guarantees* it will do exactly what these types say.

---

#[fit] Nim
![fit right](http://upload.wikimedia.org/wikipedia/en/b/b0/Green_Day_-_Nimrod_cover.jpg)

^
Let's look at Nim, which used to be called Nimrod.

---

```nimrod
# Nim
type Token = string
type Url = string

var id = 0
var urls = newTable[Token, Url]()

proc shorten*(url: Url): Token =
  id += 1
  let token = intToStr(id)
  urls[token] = url
  return token

proc expand*(token: Token): Url =
  if hasKey(urls, token):
    return urls[token]
```

^
Nim is a language that compiles to C code,
and is also statically typed like Haskell.
Here's some similar code in Nim,
but it uses global variables.

---

```haskell
{- Haskell -}
expand :: Token -> UrlStore -> Maybe Url
```

```nimrod
# Nim
proc expand*(token: Token): Url =
  # ...
```

^
But there's a difference in these return types
Haskell returns a Maybe Url, whereas Nim returns a Url.

---

```nimrod
# Nim
proc expand*(token: Token): Url =
  # ...
```

---

```nimrod
# Nim
proc expand*(token: Token): Url =
  # ...

shorten("google.com") # => "1"
```

^
So if we try this out

---

```nimrod
# Nim
proc expand*(token: Token): Url =
  # ...

shorten("google.com") # => "1"

expand("1") # => "google.com"
```

---

```nimrod
# Nim
proc expand*(token: Token): Url =
  # ...

shorten("google.com") # => "1"

expand("1") # => "google.com"
expand("54321") # => nil
```

^
Turns out, it says it returns a Url,
but it could just be nil.

---

![](http://ghcorps.org/wp-content/uploads//2013/01/Sad-Cat.jpg)

---

#[fit] nil

![](http://ghcorps.org/wp-content/uploads//2013/01/Sad-Cat.jpg)

^
nil sucks

---

```
NoMethodError: undefined method `name' for nil:NilClass
```

^
And I realized that every time I get a `NoMethodError` in Ruby,

---

```
NoMethodError: undefined method `name' for nil:NilClass
```

#[fit] Type Error

^
that's a type error

---

![left](https://pbs.twimg.com/media/B-5R78EU8AAF6uX.jpg)
#[fit] Go

^
Let's look at Go

---

```go
// Go
url, err := shortener.expand(token);

if err != nil {
    http.NotFound(response, request);
}

http.Redirect(response, request, url, 301);
```

^
A common convention people hate or fight in Go is error checking.
When you're using a function that could fail,
the function typically returns two values:
The result, in this case the number,
and an error.
And then you check to see if the error is nil or present.
Most people find this pretty ugly at first.

---

```go
// Go
url, err := shortener.expand(token);

// if err != nil {
//     http.NotFound(response, request);
// }

http.Redirect(response, request, url, 301);
```

^
But it's there for a reason.
If we didn't check for the error,
and explicitly handle error cases,

---

```go
// Go
url, err := shortener.expand(token);

// if err != nil {
//     http.NotFound(response, request);
// }

http.Redirect(response, request, url, 301); // ???
```

^
we don't know for sure what the result would be.

---

```ruby
# Ruby
get '/:token' do |token|
  url = shortener.expand(token)

  halt 404 unless url

  redirect url
end
```

^
In Ruby, we typically use nil to signify a failure state,
or quite literally the absense of a useful value.
But nil doesn't really carry any useful information,
other than it's not the other thing you wanted.

---

```ruby
# Ruby
get '/:token' do |token|
  begin
    url = shortener.expand(token)
    redirect url
  rescue # ???
    halt 404
  end
end
```

^
Sometimes we raise errors and rescue them,
but it's not really easy to tell if a method will return an error,
and which errors it might return.

---

#[fit] Elixir/Erlang
![](http://www.britishtelephones.com/ericsson/ericsson/pabx3ex.jpg)

^
Another language where the convention is to return 2 values,
the result and an error state,
is Erlang and Elixir

---

```elixir
# Elixir
{:ok, url} = Shortener.Core.expand(conn.params[:token])
```

^
Elixir uses pattern matching to ensure that the returned result matches what we expect.
Here, we expect to get an `:ok` symbol back, along with the token.

---

```elixir
# Elixir
{:ok, url} = Shortener.Core.expand(conn.params[:token])

# ** (MatchError) no match of right hand side value:
#   {:error, "Token not found"}
```

^
If things didn't go as planned,
maybe the shortener would return an error and message.
The pattern match would fail,
alerting us to the problem.

---

```elixir
# Elixir
case Shortener.Core.expand(conn.params[:token]) do
  {:ok, url} -> redirect conn, to: url
  _ -> conn.send 404, "Not found"
end
```

^
Instead we can use pattern matching to handle the happy path,
and 404 if anything else happens.

---

```ruby
# Ruby
case url = shortener.expand(token)
when String
  redirect url
when NilClass
  halt 404
end
```

^
We don't typically do this in Ruby, but it would work.

---

```ruby
# Ruby
case result = shortener.expand(token)
when result.success?
  redirect result.value
else
  halt 404
end
```

^
We also don't typically wrap things
in a Maybe or Option or Result class.
One reason might be the lack of pattern matching.

---

```ruby
# Ruby
def halve(number)
  number / 2.0
end

halve 5 # => 2.5
halve "5" # strings don't respond to `/`
```

^
But Matz has said the Ruby 3 *could* have type inference.
It's not a stretch to think Ruby could someday be smart enough to know
that this code doesn't work

---

```ruby
# Ruby
def halve(number -> Num) -> Num
  number / 2.0
end
```

^
I think people assume adding time checking to Ruby would be limiting.
But I wonder, if adding type inference to Ruby would allow us to evolve
and create new conventions, and borrow more from other languages.

---

![fit](http://upload.wikimedia.org/wikipedia/commons/c/cf/Hott_book_cover.png)

^
In learning all of these languages,
static typing hasn't been the biggest differentiator to me directly.
Instead, it's really been how each handles errors,
which can be related to static typing.

---

![fit](http://www.safetybanners.org/_images/Safety-is-everyones-responsibility.jpg)

^
It's really about confidence that my code works.
And in Ruby, I *am* confident, because I know many of the edge cases,
and it's very easy to test things.
But as I said in one of the first examples,
even *I* don't know what that code will do without trying it out in different environments.
As an aside, one of my next projects is a tool
that will try to find race conditions in these different implementations,
as well as benchmarking them.

---

![fit](http://i1.kym-cdn.com/photos/images/facebook/000/234/765/b7e.jpg)

^
Stepping back for a minute:
The code I've written is probably not idiomatic for each language.

---

![](http://i0.kym-cdn.com/photos/images/facebook/000/234/739/fa5.jpg)

^
But I've really tried hard to embrace the conventions of the languages when I learn them.
It's very easy to have a knee-jerk reaction when we see something we're not used to.
Like, why does Go not have versioned dependencies?

---

#[fit] Embrace
#[fit] Conventions
![right](http://i.imgur.com/GWY10z7.gif)

^
Embrace the conventions of the language you're working in.
Lot's of smart people are probably doing that stuff for a reason.

---

![](http://i.imgur.com/ZG2TY3w.gif)
![fit](http://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Vimlogo.svg/1022px-Vimlogo.svg.png)

^
Vim helps a lot with this for me.
There are so many plugins available for all of these languages,
that Vim becomes the best IDE for all of them.

---

![fit right](http://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Vimlogo.svg/1022px-Vimlogo.svg.png)

```vim
" Clojure
Plug 'guns/vim-clojure-static'
Plug 'guns/vim-sexp'
Plug 'tpope/vim-classpath'
Plug 'tpope/vim-fireplace'
Plug 'tpope/vim-leiningen'
Plug 'tpope/vim-sexp-mappings-for-regular-people'

" CoffeeScript
Plug 'kchmck/vim-coffee-script'

" Elixir
Plug 'elixir-lang/vim-elixir'

" Go
Plug 'fatih/vim-go'

" Haskell
Plug 'dag/vim2hs'
Plug 'eagletmt/ghcmod-vim'
Plug 'shougo/vimproc.vim'

" JavaScript
Plug 'pangloss/vim-javascript'

" Nim
Plug 'zah/nimrod.vim'
```

^
Saving a file checks for syntax errors,
does type checking,

---

![fit right](http://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Vimlogo.svg/1022px-Vimlogo.svg.png)

```vim
" OCaml
Plug 'ocamlpro/ocp-indent'
Plug 'the-lambda-church/merlin'

" PureScript
Plug 'raichoo/purescript-vim'

" Ruby
Plug 'ecomba/vim-ruby-refactoring'
Plug 'hwartig/vim-seeing-is-believing'
Plug 'jgdavey/vim-blockle'
Plug 'tpope/vim-rails'
Plug 'tpope/vim-rake'
Plug 'vim-ruby/vim-ruby'

" Rust
Plug 'wting/rust.vim'

" Scala
Plug 'derekwyatt/vim-scala'

" Shell
Plug 'markcornick/vim-bats'
Plug 'rosstimson/bats.vim'

" Thrift
Plug 'solarnz/thrift.vim'
```

^
automatically imports packages you're using,
and sometimes even suggests better code.

---

# `make`

```
```

^
Another tool I've really grown to love is `make`.

---

# `make`

```
default: deps test

deps:
  which bundle || gem install bundler
  bundle check || bundle install

test:
  bundle exec rspec spec/
```

^
Make runs commands and their dependencies.
In Ruby, we typically use Rake for this.
But when you're learning multiple languages,
or returning to a language you haven't used in some time,
or sharing a service you wrote at work with a coworker,
it's really hard to remember which commands to run.
Makefiles are executable documentation.

---

#[fit] Why?

^
So why do this?
Why spend our valuable free time,
or our downtime at work,
learning new languages?

---

#[fit] New Perspective
![original](http://i.imgur.com/8Y70IZi.gif)

^
To get a new perspective.
I truly believe learning some of these languages
has changed the way I think about programming.

---

> "If you do what you've always done, you'll get what you've always gotten"
> - Henry Ford

^
To progress our knowledge and challenge what we know,
it helps to move outside of our comfort zone.

---

#[fit] Language
#[fit] Matters

^
Language Matters
I see a lot of parallels with spoken languages.
Learning more languages allows you to communicate with more of the world,
and programming languages allow you to contribute to more code and projects.

---

## The principle of *linguistic relativity* holds that the structure of a language affects the ways in which its respective speakers conceptualize their world, i.e. their world view, or otherwise influences their cognitive processes.
### http://en.wikipedia.org/wiki/Linguistic_relativity

^
But also, learning other spoken languages can change the way your brain thinks,
known as linguistic relativity and linguistic determinism.
For me, learning new programming languages has also changed the way I think not just in that language,
but in every other language I'm working in

---

#[fit] Make you
#[fit] a better
#[fit] *x* developer

^
I believe it made me a better Rubyist
I didn't have an explicitly goal when I started to bring knowledge of other languages back to Ruby,
but it makes sense that learning more about programming makes you a better programmer.
But I didn't expect functional languages to teach me anything about object-oriented programming,
or statically typed languages to teach me anything about a dynamically typed language such as Ruby.

---

#[fit] @ work
![](http://www.adweek.com/files/uploads/iStock-Unfinished-Business-6.jpg)

^
Why would companies care about this?
and would companies want their employees to do this?

---

#[fit] Hiring Filters:

> Write your app in Rails, and enter the hiring fracas. Or, use Haskell or Clojure and hire better people with less competition.
-- Ben Orenstein

#### twitter.com/r00k/status/563821546016636928

^
If you have some variety in the technologies you use in your organization,
and you give people the power to move between or introduce new technologies,
you become much more attractive for people to work there,

---

# Bored People Quit
## - Michael Lopp

#### randsinrepose.com/archives/bored-people-quit

^
and also to stay there.
Michael Lopp of Rands in Repose
has this popular article called "Bored People Quit",
and the gist is the most important factor of keeping employees happy
after their basic needs of money and safety,
is keeping them interested in work,
and fighting off boredom.

---

#[fit] Thoughts + Ideas
### for
#[fit] Ruby:

^
So before I finish,
I have some random ideas for Ruby
that I didn't touch on earlier,
and in no particular order

---

# RSpec is hard

^
Testing becomes interesting when you're learning new languages.
How do you test when you don't know what you're doing?
I love RSpec
but it's complex for newcomers.

---

```ruby
# Ruby
describe User do
  subject(:user) {
    described_class.new(first_name, last_name)
  }

  let(:first_name) { "Justin" }
  let(:last_name) { "Campbell" }

  it "concatenates a full name" do
    expect(user.full_name).to start_with(first_name)
    expect(user.full_name).to end_with(last_name)
  end
end
```

^
This is a contrived example, but,
Subject/let, describe/context,
and all of the expectation and stub/mock syntaxes
are a complex language on their own.
Over DRYing specs can make them harder to read.
Also, stubbing and mocking requires a lot of understanding of how Ruby works.

---

```ruby
# Ruby
describe User do
  def test_full_name
    user = User.new("Justin", "Campbell")
    assert_equal "Justin Campbell", user.full_name
  end
end
```

^
So I found that doing assertion style tests
in an imperative coding style
with local variables and even copy-pasting code
made them easier to write and comprehend
when working in an unfamiliar language.

---

#[fit] Outside-in?
#[fit] Inside-out?

^
Another thing with testing:
I always test outside-in with Ruby,
and I use stubs and mocks heavily to isolate an objects dependencies.
When I'm writing Ruby, I'm very comfortable with this.
I know (I think) how Ruby works, and I understand what RSpec is doing.
I understand what happens at the language level when I stub or mock something, how to set them up,
and at what point mocks are verified.

---

#[fit] Inside-out?
#[fit] Outside-in?

^
However, when I'm writing tests in a language I installed 15 minutes ago, I don't really know any of that.
If I understand something, I can start building it with tests.
There's very little that I understand when learning a new language,
but I do know how functions work, or objects and methods, or whatever the language has.
So I think inside-out is okay if you don't know what you're doing.

---

# `$ sbt test`

```
[info] Set current project ...
[info] Run completed in 397 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 2 s
```

^
In Scala, there's a tool called SBT for Scala Build Tool

---

# `$ sbt ~test`

```
[info] Set current project ...
[info] Run completed in 397 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 2 s
1. Waiting for source changes... (press enter to interrupt)
```

^
One cool feature is,
if you prepend a tilde to the command,
it watches your files and reruns the command,
without having to reload the JVM

---

# `$ rake ~test`

```
```

^
What if Rake had a similar syntax,
and by default it just re-ran the command on any file change,

---

# `$ rake ~test`

```rb
task :test do |t|
  t.watch /^.+_test\.rb$/
end
```

^
But maybe we could control this with a task option

---

# `$ lein test`

```
Retrieving lein-exec/lein-exec/0.3.1/lein-exec-0.3.1.pom from clojars
Retrieving lein-exec/lein-exec/0.3.1/lein-exec-0.3.1.jar from clojars
Retrieving org/clojure/clojure/1.6.0/clojure-1.6.0.pom from central
Retrieving org/sonatype/oss/oss-parent/7/oss-parent-7.pom from central
Retrieving org/clojure/tools.nrepl/0.2.3/tools.nrepl-0.2.3.pom from central
[...]

lein test ...

Ran 1 tests containing 1 assertions.
0 failures, 0 errors.
```

^
In many languages, including all of the JVM languages that use Maven,
dependencies are installed automatically when you run any command
if they're missing

---

```
$ bundle exec rspec
Could not find rspec-5.0.0 in any of the sources
Run `bundle install` to install missing gems.
```

^
So why does Bundler give us this error?
The chances I'm going to type `bundle install` after I see this
are near 100%.

---

```
$ bundle exec rspec
The following gems are missing
 * rspec (3.0.0)
Installing rspec 3.0.0
Bundle complete!
...............

Finished in 0.16548 seconds
15 examples, 0 failures
```

^
Bundler is the thing that installs dependencies,
so maybe if we type bundle exec, it could, you know,
install dependencies instead of yelling at us.

---

```ruby
# Ruby
class Animal
  def initialize(type)
    @type = type
  end
end

Animal.new(:cat) # => #<Animal @type=:cat>
```

^
In Ruby, we make objects with the `new` class method.

---

```ruby
# Ruby
class Animal
  def initialize(type)
    @type = type
  end
end

Animal(:cat) # NoMethodError
```

^
But a lot of other languages use the class and parens as the syntax
for making objects or records.
So why not Ruby?

---

```ruby
# Ruby
require 'newk/everything'

class Animal
  def initialize(type)
    @type = type
  end
end

Animal(:cat) # => #<Animal @type=:cat>
```

^
I made a gem called newk to do this, and it worked

---

![inline](http://cl.ly/image/352b2L2l1z3G/Screen%20Shot%202015-03-10%20at%203.56.04%20PM.png)

^
and I put it on Reddit
But people didn't really like it

---

# "Learn a language a year"

^
It's been said that you should learn a new language every year

---

#[fit] Too long
# +
#[fit] Too short

^
But a year is way too long to dip your toes in and see if you like it,
and a year is way too short to dedicate yourself
and really absorb the conventions and ideals of the community around a language.

---

#### Start Small

^
Start with a few hours of a day,
see if you like it.
Then, as you spend a week with it,
you'll produce something substantial,
and you'll need to address meta problems such as
"how do I deploy/ship this?" and "how do I upgrade dependencies?".
After a month, you'll have experience interacting with the community,
and the honeymoon will be over.


---

#[fit] Pick
# a
#[fit] Project

---

#[fit] URL Shortener

---

#[fit] Conway's
#[fit] Game of Life

#### en.wikipedia.org/wiki/Conway's\_Game\_of\_Life

^
Conway's Game of Life is a coding exercise,
typically done at code retreats.
If you've never been to a code retreat by the way,
I highly recommend it.

---

#[fit] Which
#[fit] Language
#[fit] First?

^
So, which language should you learn right now?
I could stand here and tell you to decide for yourself which languages to learn,
but that would be disingenious of me. I have strong opinions.

---

#[fit] Haskell

^
You should learn Haskell.
You may never ever use it for a project.
I haven't yet.
But it's taught me the most about type systems.

---

#[fit] Clojure

^
You should learn Clojure,
or any Lisp.
It's taught me the most about how to think about my data,
and the syntax is incredibly simple once you get past the parentheses.

---

#[fit] Go

^
And you should learn Go,
it eschews conventions of modern languages
to keep the compiler and language simple,
and this can seem like a step backwards,
but it's a great language for getting stuff done.

---

#[fit] Ruby

^
And if you already know those languages,
you should learn Ruby!

---

#[fit] Fun + Community

^
Other than those,
I think the two reasons I would pick a language are fun and community

---

#[fit] Fun!
## Compiler + Readability

^
Fun is
how much do I enjoy working with the language?
It's easy to equate fun with dynamic typing,
but I find that these type systems help me think about the problem instead of the code, and I find that fun
how easy is it to read? what if someone else wrote it?

---

#[fit] Community!
## Adoption + Friendliness

^
And the community:
Is anyone using this for what I'm using it for?
How hard is it to find help?
When I go find help, am I welcomed or ridiculed for not getting it?
Is there a vocal minority of people
that make you embarrassed to be a part of that community?
In a way, this matters more than any other aspect of a language,
because it affects everyone who uses it.

---

#[fit] Resources:

---

![fit](https://imagery.pragprog.com/products/195/btlang.jpg)
![fit](https://imagery.pragprog.com/products/410/7lang.jpg)

^
The 7 languages in 7 weeks books are great.
You'll dip your toes in languages in a way
reminiscent of learning on your own.

---

#[fit] learn*x*in*y*minutes.com

^
Learn X in Y minutes is a website that just has code samples
in almost every language you can think of.
It's great for learning the basic syntax of a language.

---

#[fit] 🌱
#[fit] github.com/kata-seeds

^
Kata Seeds is a GitHub organization,
a collection of repositories that already has
a single passing test.
So you can clone it, `make`, and you should be able to
start playing with a language with TDD.
This is great for code retreats too.

---

#[fit] Share
# your
#[fit] Excitement
![left](http://i.imgur.com/NWq5WIq.gif)

^
I hope that everyone here will try to learn a new language soon,
and when you do, share your excitement and passion with someone else

---

#[fit] Have
#[fit] Fun
![right](http://giant.gfycat.com/SleepyThisGreendarnerdragonfly.gif)

^
Have fun

---

![](http://media0.giphy.com/media/8Ry7iAVwKBQpG/giphy.gif)

Dude, sucking at something is the first step towards being sorta good at something
- Jake, Adventure Time


^
And remember that sucking at something is the first step to being sorta good at something

---

#[fit] Thanks!
# @justincampbell
