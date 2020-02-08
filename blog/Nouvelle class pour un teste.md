---
layout: default
title: Nouvelle class pour un teste
parent: Blog
nav_order: 1
---

# Nouvelle class pour un teste

Quand on veut tester un module on peux soit texter la class qui l'inclue ou (meillieur pratique) créé une nouvelle class qui l'include et juste testé la/les méthode.

## Ce qu'on voit parfois

On voit souvent des testes qui commence par:

```ruby
describe MyConcern do
  class Foo
    def hello
      'world'
    end
  end
end
```

Le problème est que une fois déclarer la classe sera disponible dans tout les autres testes.

2 problèmes:

* On ne veut pas polluer le namespace inutilement
* Si on overwrite/open une class sa sera effectif pour toute la suite des testes

**Example**

```ruby
describe 'My specs' do
  context 'define class' do
    class Foo
      def hello
        'world'
      end
    end
    it { expect(Foo.new.hello).to eq('world') } # 🟢
  end

  context 'not defined class' do
    it { expect(defined? Foo).to be_falsey } # 🔴
  end
end
```

Bien que `Foo` soit créé dans un autre context, il est définie dans un autre.

## Comment définir une class

Avec `Class.new`, sa retourne un objet class (tout est une objet en Ruby) qui définie la méthode `#new` pour créé une instance de cette class.

```ruby
describe 'My specs' do
  context 'define class' do
    let(:foo) do
      Class.new do
        def hello
          'world'
        end
      end
    end
    it { expect(foo.new.hello).to eq('world') } # 🟢
  end
  
  context 'not defined class' do
   # `foo` is not accessible
  end
end
```

## Plus

**include**

```ruby
let(:foo) do
  Class.new do
    include MyModule
    
    def hello
      'world'
    end
  end
end
```

**inherit**

```ruby
let(:foo) do
  Class.new(MySuperClass) do    
    def hello
      'world'
    end
  end
end
```

**struct**

```ruby
let(:foo) do
  Struct.new(:bar)
end
```

## Références

https://makandracards.com/makandra/47189-rspec-how-to-define-classes-for-specs
