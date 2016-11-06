**Understanding Rails Associations**

The amount of configuration that Rails saves is wonderful for a seasoned dev.  For a non-seasoned dev, it can be an impediment to understanding what your application is actually doing, or it was for me in 2014.  One regular source of pain came from not really understanding how `has_*` or `belongs_to` relationships worked.

**Dummy App**
The most associationally complex app I've built was designed to compare different methodologies for measuring affect.  For the purposes of this post, it'll be worth understanding that some of the users tested out the day reconstruction method (read a paper about the DRM by Dylan Smith, the social scientist I built the app for, <a href="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&ved=0ahUKEwjHmP_05pTQAhWJ54MKHdoaDzwQFgggMAE&url=http%3A%2F%2Fstat.gov.pl%2Fdownload%2Fgfx%2Fportalinformacyjny%2Fen%2Fdefaultlistaplikow%2F3450%2F7%2F1%2F7a_smith_s429-440_1_8_2016.pdf&usg=AFQjCNFdXtJHcTAZVzjmZE7-sK_wJDFzRw&sig2=LG4U7LfqBCupkr9YjYLohw&bvm=bv.137904068,d.eWE&cad=rja">here</a>).  For our purposes, it's worth knowing that each of the users in this condition users had many DRMs (they filled out two over the course of the study).  The DRM is a complex instrument, and it had various parts.  Each DRM has a DrmMorning, DrmAfternoon, and a DrmEvening.

**`belongs_to`**

I'll just assume that anyone writing a `belongs_to` association has written a migration before and understands a db table.  Every table has a primary key, typically some self-incrementing non-null integer.  When you declare that a model class belongs to another model class you are telling rails, 'my table includes another key, one that references the primary key of a different table.  Whenever you join these tables for a query join them on this foreign key.'  That's it!  All belongs_to does is tell rails "hey--this is how to join these tables."

Since tables that belong to other tables have reference to those tables, any instantiation that doesn't reference that other table will populate the foreign key column with `NULL` values--something to keep in mind.

For the dummy app described above this meant that the `drm_mornings`, `drm_afernoons`, and `drm_evenings` tables each held a reference to to the DRM.  Similarly the `drms` table held a reference to the `users` table.  This should make sense by now.  We had users in different conditions, and only some of them filled out a DRM.  The `users` table wouldn't be well served by including a `Drm_id` column--2/3s of them would be `NULL`.  The `drms` table, though, had to have a `user_id` column.  There couldn't be a DRM that wasn't filled out by a user -- and even if there were, it wouldn't be very useful for the purposes of the study if we didn't know who filled it out.

Now that we know what belongs to means, let's see what it does.

my app had the simple line
`belongs_to :user`
Stripping away <em>some</em> of the Rails work, we could write this out longhand as:
````
class DRM
belongs_to(:user,
  {class_name: "User",
  primary_key: id,
  foreign_key: user_id})
````

If you follow conventions, Rails knows that if the DRM belongs to a User, the class name is going to be "User" and the foreign_key is going to be "user_id".  So, you don't have to write this out.

The belongs_to macro is a method that defines a method named `user`.  That's why you can write `Drm.user`.  It's just a method definition.

You could do even more work and write it out as:
````
  def user
    Drm.joins(:users)
    on, where, blah blah, blah, other railsified SQL
  end
````
**`has_many` or `has_one`**
Now that we've gotten the hard work out of the way, the rest should be easy.  the tables underlying models that have other objects don't hold references to the tables of those objects, and expect those tables to hold references to to themselves.  God, that ws a mouthful.  Let's just look at code.

Just as the `users` table didn't hold a reference to the  `drms` table, the `drms` table didn't hold a reference to the `drm_mornings`, `drm_afternoons` or `drm_evenings` table.  But, each `drm` table had many of those other tables (up to eight each).

in the app, this was simply
````
has_many :drm_mornings
has_many :drm_afternoons
has_many :drm_evenings
````
This code just tells rails -- "Hey!  When you want to join these tables, look in the `drm_mornings` (or afternoons, or evenings) table for the foreign key".  That's it!
````
class DRM
has_many(:drm_mornings,
  {class_name: "DrmMorning",
  primary_key: id,
  foreign_key: drm_id})
````
Again, this just defines a method in the `Drm` class.

**`has_many_through`**

Now that we know what` has_many` and `belongs_to` macros actually do (define methods that tell rails how to perform sql queries), it's a lot easier to understand `has_many_through`, something that gave me no small amount of grief.

In our dummy app the User had many DrmMornings through Drms.
````
class User
  has_many :drm_morning, :through => :drm
````
This macro, again, defines a method in the user class to tell rails how to join the `users` table to the `drm_mornings` table.  The long-hand is:
````
class User
  has_many(:drm_mornings,
    through: :drm,
    source: :drm_mornings)
````
The first `drm_mornings` here is the new method defined on the User class, while the second is a reference to the same method name in the Drm class.  The fact that these methods are the same is what allows rails to condense this longhand.
