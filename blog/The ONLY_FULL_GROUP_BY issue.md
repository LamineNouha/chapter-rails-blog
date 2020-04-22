# The `ONLY_FULL_GROUP_BY` issue

## C'est quoi

[Documentation et example](https://www.percona.com/blog/2019/05/13/solve-query-failures-regarding-only_full_group_by-sql-mode/)

Une query comme

```ruby
Membership.group(:account_id)
```

est ambigu. Oui sa retourne 1 membership par account mais lequel? Pas facile de savoir, par id peut-être. Mais si je veux le plus récent, 

```ruby
Membership.group(:account_id).order(created_at: :desc)
```

ne fonctionnera pas puisque le `order()` s'applique après le `group()`. Donc je vais avoir un membership par account_id
en ordre de `created_at`.

## Un autre example

Le membership le plus récent pour les accounts qui en ont plus de 2

```ruby
Membership.group(:account_id).having('count(account_id) > 2').map(&:id)
# [
#     [0] 480,
#     [1] 482,
#     [2] 509,
#     [3] 473
# ]
```

Si je tente de mettre un order pour que sa retourne le plus récent

```ruby
Membership.group(:account_id).having('count(account_id) > 2').order(:created_at).map(&:id)
# [
#     [0] 473,
#     [1] 480,
#     [2] 482,
#     [3] 509
# ]
```

```ruby
Membership.order(:created_at).group(:account_id).having('count(account_id) > 2').map(&:id)
# [
#     [0] 473,
#     [1] 480,
#     [2] 482,
#     [3] 509
# ]
```

C'est toujours les mêmes.


## Solutions

C'est un problème commun. MySQL a même une page de [documentation](https://dev.mysql.com/doc/refman/5.6/en/example-maximum-column-group-row.html)
 qui explique comme faire ce genre de query. Conslusion sa prend une sub query.


```ruby
group_query = Membership
                .select('account_id', 'max(created_at) as created_at')
                .group(:account_id)
                .having('count(account_id) > 2')
                .to_sql
joins = <<-SQL
  INNER JOIN (#{group_query}) as S
  ON memberships.account_id = S.account_id
  AND memberships.created_at = S.created_At
SQL
Membership.joins(joins).map(&:id)
# [
#     [0] 545,
#     [1] 505,
#     [2] 509,
#     [3] 553
# ]
```

Si on group par `id`, c'est plus facile.

```ruby
# Ref: scope :solo, -> { where(private: true).where.not(account_id: nil).joins(:conversers).group(:id).having('count(mess__conversers.id) = 1') }
group_query = Messaging::Conversation.select(:id).where(private: true).where.not(account_id: nil).joins(:conversers).group(:id).having('count(mess__conversers.id) = 1')
Messaging::Conversation.where(id: group_query)
```

Sinon en Ruby, moins performant parce qu'il faut loader plus de ActiveRecord

```ruby
Membership.all
  .group_by(&:account_id)
  .values
  .select { |memberships| memberships.size > 2 }
  .map { |memberships| memberships.max_by(&:created_at) }
  .map(&:id)
# [
#     [0] 505,
#     [1] 545,
#     [2] 509,
#     [3] 553
# ]
```


