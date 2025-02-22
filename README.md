nosqlite.py
===========

``nosqlite.py`` is a pure python library for Python 3
that aims to provide a schemaless wrapper for interacting with sqlite databases.
Much of the behavior follows how the API for [pymongo](http://api.mongodb.org/python/current)
works, so those familiar with that library should have a similar experience. Example::

```python
import nosqlite

with nosqlite.Connection(':memory:') as conn:
    foo_collection = conn['foo_collection']
    foo_collection.insert({'foo': 'bar', 'baz': 'qux'})
    foo_collection.find({'foo': 'bar'})
```

Indexes
-------
- Index creation is supported for single key, compound key, and even nested key.

  ```python
  foo_collection.insert({'a':1, 'b':2, 'c':10})
  foo_collection.insert({'a':1, 'b':3, 'c':20})
  foo_collection.insert({'x':{'y':100}, 'z':200})
  foo_collection.create_index('a')        # single key
  foo_collection.create_index(['a','b'])  # compound key
  foo_collection.create_index('x.y')      # nested key
  ```

- Index is actually being used in `find` operation.
  e.g. `foo_collection.find({'a':1, 'b':3})` the library will put them in `where`
  clause as a subquery to search `_id` on index with `a=1` and `b=3` which result
  is a subset of `_id` on the collection, so searching on the collection will use
  its primary key instead of a whole table scan.  
  - *however, the other operations like `$gt` or `$lt` still use table scan*

- Unique index is supported and force contraint on `insert`, `save` (`update`).
  - You cannot create a unique index on collection with non-unique keys.
  - You cannot update a document to break uniqueness on a collection with unique index.
  - Any attempt above will raise an `IntegrityError` and the document will not be
    inserted or updated.

- Reindex with only a subset of documents is supported.

- Find with hint index is supported e.g. you don't have index on `('a', 'c')` but
  already have index on `'a'`.

  ```python
  foo_collection.find({'a':1, 'c':20})  # use table scan
  foo_collection.find({'a':1, 'c':20},
      hint='[foo_collection{a}]'        # use index to narrow down rows with a=1
  )
  ```

Sorting
-------
- Result documents from `find()` can be sorted by any key and also nested key
  ```python
  foo_collection.find({'a':1}, sort={'c':nosqlite.DESCENDING})
  foo_collection.find({'a':1}, sort={'b':nosqlite.ASCENDING,
                                     'c':nosqlite.DESCENDING})
  foo_collection.find(sort={'x.y':nosqlite.DESCENDING})
  ```

TODOs
-----
- Support using index on other operations.
- Support for embedded documents and queries on those (i.e. {'foo.bar': 5})


Contribution and License
------------------------
This project was originally developed by Shaun Duncan <shaun[dot]duncan[at]gmail[dot]com>
and is now maintained by Chaiwat Suttipongsakul <cwt[at]bashell[dot]com>.
It is licensed under the MIT license, which means you are free to use, modify,
and distribute the code for your own purposes.

Contributions to the project are highly encouraged and greatly appreciated.
If you find a bug, have an enhancement in mind, or want to suggest a new feature,
please feel free to contribute.
