---
layout: default
title: La magie des blocks
parent: Blog
nav_order: 2
---

# La magie des blocks
Se déclare avec `do / end` ou `{ }`

## yield VS call

Exécuter le block en question

```ruby
def print_once
  yield
end

def print_once2(&block)
  block.call
end

print_once { puts "Block is being run" }
print_once2 { puts "Block is being run" }
```

## Les arguments

```ruby
class Array
  def map(times = 0)
    mapping = []
    index = 0

    while index < self.size
      mapping << (yield(self[index]) * times)
      index += 1
    end
    mapping
  end
end

[1, 2, 3].map(2) { |n| n.to_s } 
# => ['11', '22', '33']
```

## next

Le `return` des blocks

```ruby
[1,2,3,4].map do |n|
  next n if n.odd?
  n - 1
end
# => [1,1,3,3]
```

Example dans un serializer
```ruby
attribute do |object|
  next object.name if object.respond_to?(:name)
  
  "#{object.first_name} #{object.last_name}"
end
```

## redo

Je vois pas a quoi sa peut servire 🤷‍♂️

```ruby
[1,2,3].map do |n|
  n += 1
  redo if n <= 2
  n
end
# => [3, 3, 4]
```

## block_given? + super

Sa permet d'exécuter du code au "milieu" d'une méthode, le plus souvent 
on modifier l'input (super au début) ou on modifie l'output (super à la fin)

```ruby
class Printer
  def print(text)
    text = yield text if block_given?
    puts 'Printing... ' + text
  end
end

class UpperPrinter < Printer
  def print(text)
    super { |text| text.upcase }
  end
end

Printer.new.print('abcdefgh') # => Printing... abcdefgh
UpperPrinter.new.print('abcdefgh') # => Printing... ABCDEFGH
```

## block avec `&`

```ruby
[1,2,3].map(&:to_s) # => ["1", "2", "3"]
[1,2,3].map { |n| n.to_s }

[1,2,3].max_by(2, &:to_s) # => [3, 2]
[1,2,3].max_by(2) { |n| n.to_s } # => [3, 2]
```

**Magie [`Symbol#to_proc`](https://devdocs.io/ruby~2.4/symbol#method-i-to_proc)**

```ruby
def to_proc
  Proc.new { |obj, *args| obj.send(self, *args) }
end
```

```ruby
[1,2,3].max_by(2, &(Proc.new { |number| number.send(:to_s) }))
```

## Utiliser un proc

Pourquoi parler de proc... On peut mettre un proc dans une variable et donc passer cette variable comme block.

```ruby
to_s_proc = proc { |n| n.to_s }
# ou
to_s_proc = proc(&:to_s)
# ou
to_s_proc = Proc.new { |n| n.to_s }
# ou
to_s_proc = Proc.new(&:to_s)

[1,2,3].map(&to_s_proc) # => ['1', '2', '3']
```
Pour info `p = proc { |n| puts n }` est préférable https://rubystyle.guide/#proc

