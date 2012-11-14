# RubyMotion

@justincampbell

# Dividata iOS

bit.ly/dividata-ios-beta

# iOS #1

* iPhone 3G/App Store
* First Mac
* Inexperienced
* Everything was NDA

# iOS #2

* Movitas
* Only Mac user in office
* PhoneGap/SIP

# iOS #3

* RubyMotion
* May 2012
* Prototype in 4 hours
* Finished/polished 1 month

# RubyMotion

* Laurent Sansonetti (@lrz)
* HipByte
* MacRuby
* $199

# What is RubyMotion?

* Ruby compiler for iOS
* Rake workflow
* 1.9 (ish)
* Core library
* Missing require/eval
* Named-parameters

# Named Parameters

```objc
[@"Ruby"
  compare:@"ruby"
  options:NSCaseInsensitiveSearch];
```

# Named Parameters

```objc
NSString
-compare:options:
```

# Named Parameters

```rb
"Ruby".compare("ruby",
  options: NSCaseInsensitiveSearch)
```

# Named Parameters

```rb
"Ruby".compare "ruby",
  options: NSCaseInsensitiveSearch
```

# Named Parameters

```rb
def application(application,
  didFinishLoadingWithOptions: options)
  # ...
end
```

# Types

```rb
String < NSMutableString
Array < NSMutableArray
Hash < NSMutableDictionary
Time < NSDate
...
```

# Selector Shorcuts

```rb
alias foo= setFoo:
alias foo? isFoo
alias []   objectForKey:
alias []=  setObject:forKey:
```

# Pointers

```rb
pointer = Pointer.new :object
```

# Concurrency

```rb
Dispatch::Queue.concurrent.async do
  Dispatch::Queue.main.sync do
    puts "Hi!"
  end
end
```

# Demo

New App

# Demo

Dividata + REPL

# Demo

TestFlight

#

Questions?

# Thanks!

@justincampbell
