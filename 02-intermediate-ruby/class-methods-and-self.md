# Class Methods & Self

## Learning Goals

- Review what we've learned about classes so far:
  - Constructor (`initialize`)
  - Attributes (stored in instance variables)
  - Instance Methods
- Discover new functionality within classes:
  - _Class Methods_
  - `self`
  - Class variables
- Examine why we'd use one type of method over another

## Introduction

The examples we will use in class today will assume we are writing software for a music player that plays audio tracks based off files on your computer, and keeps track of how many times each audio track has been played.

## Ruby Classes Thus Far

### Constructors (`initialize`), Instance Variables, and Instance Methods

The `initialize` method within a class is special. When the `new` method is called on a _class_ a new _instance_ is created and the `initialize` method is promptly invoked. This method is used to create the default state of an _object instance_.

Within the `initialize` method, we do any setup that should happen at the beginning of creating any instance of a class:

  1. make any necessary data manipulations
  1. set any given attributes in _instance variables_.

By default, instance variables are only visible inside their object. Within the class definition, we refer to and access the values of instance variables using the `@variable_name` syntax.

Outside of the class definition, code that wants to read or write those instance variables must use one of the _helper methods_ `attr_reader`, `attr_writer` or `attr_accessor` (which are shortcuts for _getter_ and _setter_ methods).

Here is a class that might represent a song in a music player program:

```ruby
class Song
  attr_reader :title, :artist, :filename, :play_count

  def initialize(title, artist, filename)
    @title = title
    @artist = artist
    @filename = filename
    @play_count = 0
  end
end
```

```ruby
song = Song.new("Respect", "Aretha Franklin", "songs/respect.mp3")
song.title
# => "Respect"
```

### Instance Methods

Instance methods are defined with `def a_snake_case_name`. Instance methods are invoked on _instances_ of a class. **These are exactly the type of Ruby method we've seen thus far**.

Here is an expanded version of our `Song` class, now with a couple of instance methods.

```ruby
class Song
  attr_reader :title, :artist, :filename, :play_count

  def initialize(title, artist, filename)
    @title = title
    @artist = artist
    @filename = filename
    @play_count = 0
  end

  def summary
    return "#{@title}, by #{@artist}"
  end

  def play
    @play_count += 1
    # ... more code that loads the song data from the file and sends it to the speakers ...
  end
end
```

```ruby
song = Song.new("Respect", "Aretha Franklin", "songs/respect.mp3")
puts song.summary
# => Respect, by Aretha Franklin
song.play
# 🎶 R-E-S-P-E-C-T 🎶
song.play_count
# => 1
```

#### Assess

With this code above, identify which parts of the following code are:

1. instance variables
1. instance methods
1. creating an instance of a class
1. invoking/calling an instance method on an instance of a class

<details>
  <summary>
  Answers
  </summary>

  1. `@title`, `@artist`, `@filename`, `@play_count`

  1. `summary`, `play`. `attr_reader` technically defines more instance methods, too! So, additionally: `title`, `artist`, `filename`, `play_count`

  1. `Song.new("Respect", "Aretha Franklin", "songs/respect.mp3")`

  1. `song.summary`, `song.play`, `song.play_count`
</details>

### Review Summary

When we review each of these pieces of code that we've written there is one major thing in common: we are always creating an **instance** of our class prior to _calling_ any of its instance methods.

This is because all of the methods we use are always relying on specific data that is related to each individual instance of the object we are using. For the `Song`'s `summary` method, we were showing the data about a **specific** `Song`. If we had 500 different instances of `Song`, it would probably be true that each instance of `Song` had a different result for the `summary` method, even though they are all `Song` instances with the same implementation. The instance method `summary` used an instance variable within it.

There are times in which we'll make methods that are related to the concept of `Song`, but are not related to a specific instance of a `Song`.

  - What if we have things that related to **all** `Song`s?
  - What if we want `Song`-specific code used to set up a bunch of `Song`s, and not just one `Song`?

## Class Methods and `self`

Instance methods are methods that are called on an instance of a class. **_Class methods_ are methods that _are called on a class_**.

### Why?

Class methods provide functionality to a class itself. We may soon need to implement features that belong to the class, but don't belong to a single instance of the class. Let's look at the syntax

### Syntax to Use Class Methods

Class methods are called directly on the _class_ and not on an _instance_ of the class. Assuming that there is a class named `ClassName` and a class method named `class_method`...

```ruby
puts ClassName.class_method
# 'This is a CLASS METHOD'
```

Compare this to our syntax of calling an instance method.

```ruby
instance = ClassName.new
puts instance.instance_method
# 'This is an INSTANCE METHOD'
```

We've already seen many different class methods!

`new` is probably the class method we're most familiar with. If you haven't noticed before, take the time to notice that we always call `.new` off of the name of the class we're instantiating. That's because `new` is a class method! `new` is a built-in Ruby method defined on all objects, which builds an instance of that class, calls its `initialize` method, and returns that instance. It is a class method because it doesn't make sense to operate off of an instance of a class... initializing an instance of a class is the functionality of a class as a whole!

Another class method we might have seen before is `Random.rand` to generate a random number. `Random` is a class that doesn't particularly necessitate an _instance_ of a Thing that is Random, but it _is_ a class that has a feature available to the concept of Random.

### Syntax to Define Class Methods

Let's see how to define a class method within a class definition:

```ruby
class ClassName
  def self.class_method
    return "This is a CLASS METHOD"
  end

  def instance_method
    return "This is an INSTANCE METHOD"
  end
end
```

**Class methods** are _defined_ like **instance methods**, but they start with `self.`

**Review:** What's the syntax to call the class method? What's the syntax to call the instance method?

#### Usage of Class Methods Within the Class

Class methods are called internally with the `self.class` identifier or the name of the class identifier.

```ruby
class ClassName
  def self.class_method
    return "This is a CLASS METHOD"
  end

  def instance_method
    return "This is an INSTANCE METHOD... and I'm calling the class method with syntax 1: #{self.class.class_method}, and syntax 2: #{ClassName.class_method}"
  end
end
```

## Adding Class Methods to `Song`

Now that we see the syntax for how we'd use a **class method** versus an **instance method**, let's see why we'd want to use one over the other.

Let's think back to the `Song` class we created earlier. We'll start with tracking the total number of plays across all songs. For this, we'll use a strategy to allow the `Song` class to manage this tracking. We'll create a _class variable_, `@@total_plays`, as well as a method to read its value, `Song.total_plays`.

```ruby
class Song
  attr_reader :title, :artist, :filename, :play_count

  def initialize(title, artist, filename)
    @title = title
    @artist = artist
    @filename = filename
    @play_count = 0
  end

  def summary
    return "#{@title}, by #{@artist}"
  end

  def play
    @play_count += 1
    # ... load the song data from the file and send it to the speakers ...
  end

  # Define a class method 
  def self.definition
    return "A song is a short poem or other set of words set to music or meant to be sung."
  end

end
```

```ruby
puts Song.definition
```
## Class Variables

While we are discussing class methods, now seems like a good time to also introduce class variables. Class methods and class variables are about as related to each other as instance methods and instance variables are related to each other... There is no strict relationship, but their concepts in how they relate to object-oriented programming are similar.

- Class variables begin with `@@`
- Class variables are typically defined with an initial value at the top of the class
- Class variables are available to the entire class (in any method)
- Class variables will raise an error if they are read before they're created
- Class variables can cause problems later, so **avoid using them**
- Class variables are sometimes used for application configuration

## Adding Class Variables to `Song`

```ruby
class Song
  attr_reader :title, :artist, :filename, :play_count

  # Initialize our total play count
  # This will be set to 0 when the program is loaded
  @@total_plays = 0

  def initialize(title, artist, filename)
    @title = title
    @artist = artist
    @filename = filename
    @play_count = 0
  end

  def summary
    return "#{@title}, by #{@artist}"
  end

  def play
    @play_count += 1
    @@total_plays += 1
    # ... load the song data from the file and send it to the speakers ...
  end

  # Define a class method to access the class variable
  def self.total_plays
    return @@total_plays
  end
end
```

```ruby
# No instances required!
puts "total: #{Song.total_plays} plays"

respect = Song.new("Respect", "Aretha Franklin", "songs/respect.mp3")
moonlight = Song.new("What a Little Moonlight Can Do", "Billie Holiday", "songs/moonlight.mp3")

3.times do
  respect.play
end

5.times do
  moonlight.play
end

# No instances required!
puts "total: #{Song.total_plays} plays"

# Respect: 3 plays
# What a Little Moonlight Can Do: 5 plays
# total: 8 plays
```

All that `total_plays` does is return the value of `@@total_plays`. If `@@total_plays` were an instance method, we would use the `attr_reader` helper method to accomplish the same thing. Unfortunately there's no equivalent to `attr_reader` for class variables, so we have to do the work ourselves.

#### Questions
- Consider the class variable `@@total_plays`
  - Where is `@@total_plays` initialized?
  - How would our program change if we did not initialize this variable?
  - How would our program change if we initialized this variable in the `initialize` method?
- Why is `total_plays` a class method? How would our program change if it was an instance method?

#### Assess

1. What's the syntax to define a class method? Where are class methods defined?
1. What's the syntax to invoke a class method?
1. What's the syntax to define a class variable? Where are class variables defined?

<details>
  <summary>
  Answers
  </summary>
  
  1. `def self.class_method`. Class methods are defined within a class.
  1. with a dot, off of the class name (with the proper capitalization, because this must match the name of the class)
  1. `@@variable_name`. Class variables are defined within a class.
</details>

## Activity: `Song.most_played`

In the previous example, we used a class method to access a class variable. Another common use of a class method is to work with a collection of instances of that class.

For example, what if we wanted a method that, given an array of `Song`s, picks the one with the most plays? Since the argument is a collection of `Song`s, it doesn't make sense to require the method to be called on one particular instance. In this case, a class method is a good choice.

Work with a partner to implement `Song.most_played`. As you write the method, think about how you would test it - what interesting scenarios can you imagine?

Once you've come up with an version you're happy with, [you can see ours here](source/song.rb).

## Avoiding class variables using composition.

Review the jukebox class below.

* How does it keep track of total plays?
* How does it determine the most played song?

Create jukebox.rb and main.rb files. Copy and paste the code. 

* What do you expect runing main.rb to produce?
* Run `ruby main.rb` in the terminal. 

```ruby
#jukebox.rbclass Jukebox
  attr_reader :songs, :total_plays

  def initialize
    @total_plays = 0
    @songs = []
  end

  def add_song(song)
    @songs << song
  end

  def play(song)
    song.play
    @total_plays += 1
  end

  def calculate_most_played
    most_played = @songs.max_by do |song|
      song.play_count
    end
    return most_played
  end
  
end
```

```ruby
#main.rb
require_relative 'jukebox'
require_relative 'song'

def make_jukebox
  ada_jukebox = Jukebox.new

  ada_jukebox.add_song(Song.new("Respect", "Aretha Franklin", "songs/respect.mp3"))
  ada_jukebox.add_song(Song.new("What a Little Moonlight Can Do", "Billie Holiday", "songs/moonlight.mp3"))
  ada_jukebox.add_song(Song.new("Adore", "Savages", "songs/adore.mp3"))

  return ada_jukebox
end

def main
  ada_jukebox = make_jukebox

  s1 = ada_jukebox.songs[0]
  s2 = ada_jukebox.songs[1]
  s3 = ada_jukebox.songs[2]

  3.times do
    ada_jukebox.play(s1)
  end

  5.times do
    ada_jukebox.play(s2)
  end

  2.times do
    ada_jukebox.play(s3)
  end

  puts ada_jukebox.total_plays
  puts ada_jukebox.calculate_most_played.title
end

main
```

We could go deeper into class variables. However, in general, we will discourage the use of class variables because of their usually unintended side-effects and encourage other types of design.

## Writing About Methods in Documentation

When Rubyists write English text about a class's instance methods in documentation, we typically write their names in the following format:

```
ClassName#method_name
```

The octothorp indicates that the method is an instance method. In the case of our `Song` class, we have `Song#play` and `Song#summary`.

When writing about a class method, we use a dot instead of a pound sign: `ClassName.class_method`. Note that this matches the way the method is called.

## Conclusion

Class methods provide functionality to a class itself.

We should be able to contrast class methods against instance methods, and class variables against instance variables.

We should be able to combine our regular Ruby logic to this new concept of object-oriented programming to do complex functionality, such as what our `Song.most_played` activity accomplished.

## Additional Resources
- [Some Additional Examples](https://www.jimmycuadra.com/posts/self-in-ruby/)
- [Few More Examples](https://hackhands.com/three-golden-rules-understand-self-ruby/)
- [Even more uses of `self` than you'll need right now](http://blog.honeybadger.io/ruby-self-cheat-sheet/)
