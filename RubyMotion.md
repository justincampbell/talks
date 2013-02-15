# RubyMotion

@justincampbell / mashion.net

# iOS #1

* iPhone 3G/App Store
* First Mac
* Inexperienced
* Everything was NDA

# iOS #2

* Consultant-built
* Only Mac user in office
* PhoneGap/SIP

# iOS #3

* RubyMotion
* May 2012
* Prototype in 4 hours
* Finished/polished 1 month

# RubyMotion

* Laurent Sansonetti (@lrz)
* MacRuby @ Apple
* HipByte
* $199

# What is RubyMotion?

* Ruby compiler for iOS
* Rake workflow
* 1.9 (ish)
* Core library
* Missing require/eval/binding
* Adds named-parameters

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
class String
  def compare(other, options: options)
    ...
  end
end

"Ruby".compare("ruby",
  options: NSCaseInsensitiveSearch)
```

# Named Parameters

```objc
- (BOOL)application:
    (UIApplication *)application
  didFinishLaunchingWithOptions:
    (NSDictionary *)launchOptions
{
  return true;
}
```

# Named Parameters

```rb
def application(application,
  didFinishLoadingWithOptions: options)

  true
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
  @result = do_work

  Dispatch::Queue.main.sync do
    handle @result
  end
end
```

# New App

```rb
motion create my-app
rake config
rake
```

# Testflight

```sh
rake testflight notes="New stuff"
```

# Why?

* Ruby! ~3X more concise than Obj-C
* Rake! We ❤ the command line
* Vim/Emacs/Sublime/etc...
* Amazing community

# Twitter Search

justincampbell/rubymotion-demo

# Cocos2D

justincampbell/rubymotion-cocos2d

# Dividata

appstore.com/dividata

# Questions?

# Thanks!

@justincampbell
