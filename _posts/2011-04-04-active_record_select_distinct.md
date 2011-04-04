---
layout: post
title: Creating a Scope with Unique Records 
---

{{ page.title }}
================

<p class="meta">04 April 2011</p>

After a whole bunch of googling, I didn't find anything helpful regarding the "accepted" or "recommended" way to create a scope that would bring us back to a unique set of records after joining one to many.

I wanted something that when given a scope, #count and #all and any chained scope would return an expected result.



Trying to just group by id has problems with count:

    scope :distinct, group(arel_table['id'])
    
Running count on grouped scope will give you a hash of counts of which I haven't been able to make any use.



In the end I landed on this:

    scope :distinct, 
      select("DISTINCT(#{quoted_table_name}.id)").select("#{quoted_table_name}.*")
    
It works well for chaining, but count will need :distinct => true.   I'd like to figure out how to fold that in to the scope definition.  And I haven't yet tried eager loading (#includes), not sure how selects and eager loading play together, skeptical whether it would work.