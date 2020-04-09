---
layout: default
title: Nouveautés Rails 5
parent: Blog
nav_order: 3
---

# Nouveautés Rails 5 ([Release notes](https://guides.rubyonrails.org/5_0_release_notes.html))

* `or`
* `left_outer_join`

### [in_batched](https://devdocs.io/rails~5.0/activerecord/batches#method-i-in_batches)

Retourne un ActiveRecord::Relation au lieu d'un array

```ruby
# Rails 4

Account.not.where(deactivated_at: nil).find_in_batches do |accounts|
  # accounts is a Enumerator
  accounts.each { |account| account.update(optin_scoville: true) }
  accounts.each(&:destroy!)
end

# Rails 5

Account.not.where(deactivated_at: nil).in_batches do |accounts|
  # accounts is a ActiveRecord::Batches::BatchEnumerator
  accounts.update_all(optin_scoville: true)
  accounts.destroy_all
end
```

### Better support enum

Pourquoi se changement, parce qu'on travaille avec des true/false en Ruby pas des 0/1. Alors même logique pour les enums.

```ruby
>> ContactMethod.kinds
{
           "work" => 1,
         "mobile" => 2,
          "pager" => 3,
           "home" => 4,
      "codepager" => 5,
            "fax" => 6,
       "pharmacy" => 7,
          "other" => 8,
     "speed_dial" => 9,
   "private_line" => 10
}
```

```ruby
# Rails 4

>> ContactMethod.limit(1).pluck(:kind)
[
    [0] 1
]
>> ContactMethod.limit(1).where(kind: ContactMethod.kinds[:work]).size

# Rails 5

>> ContactMethod.limit(1).pluck(:kind)
[
    [0] "work"
]
>> ContactMethod.limit(1).where(kind: :work).size
```

### Cache_key for ActiveRecord::Relation

Utilise un count et le max updated_at pour créer la cache_key.

Facile a utiliser avec [ActionView#cache](https://api.rubyonrails.org/classes/ActionView/Helpers/CacheHelper.html#method-i-cache)

```ruby
def index
  @memberships = Membership.where(group_id: 1)
end

# index.html.erb
<% cache(@memberships) %>
```

Il y a place a utiliser sont imagination avec `Rails.cache`

```ruby
relation = Membership.where(group_id: 1)

Rails.cache.fetch(relation.cache_key) do
  relation
end
```

### Drop table if exists

[Commit](https://github.com/rails/rails/pull/18597/files#diff-1ed2907b3b8f148c2533558a77673ffaR5)

```ruby
drop_table(:posts, if_exists: true)
```

### New attributes API

A other time




