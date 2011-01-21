---
layout: post
title: Can't Extend Has One with a Block
---

{{ page.title }}
================

<p class="meta">21 January 2011</p>

I was trying to have my has_one association return something other than nil when one wasn't found.  Oh boy, I ended on quite the wild goose chase when I found that this failed:

    has_one :thing do
      def find_target
        super || DummyThing.instance
      end
    end
  
It was somewhat enlightening.  When I went to inspect thing, I didn't get my DummyThing when I expected it, I got "nil."  And when I tried to call find_target, I also got nil.  I started just calling random methods, they all returned nil, not method not found for NilClass, nut nil.
Turns out that AssociationProxy pretty much removes all instance methods from association proxy and then delegates method missing to the target. 

    class AssociationProxy #:nodoc:
      alias_method :proxy_respond_to?, :respond_to?
      alias_method :proxy_extend, :extend
      delegate :to_param, :to => :proxy_target
      instance_methods.each { |m| undef_method m unless m.to_s =~ /^(?:nil\?|send|object_id|to_a)$|^__|^respond_to_missing|proxy_/ }

 But if you don't have a target, method missing always returns nil.  Still doesn't explain why it doesn't find my find_target and therefore not return nil.  Well, a bit of undocumented magic, as far as I could see, the block syntax doesn't work on has_one and belongs_to, you have to use ":extend => AssociationExtension"