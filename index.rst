.. include:: <s5defs.txt>

============================
ZODB Happenings
============================

| Jim Fulton
| jim@jimfulton.info
|
| Plone Digital Experience Conference
| October 19, 2016
|
| http://jimfulton.info/talks/zodb-plone-conf-2016/

I'm working 100% on ZODB
=============================

ZeroDB made this possible by contracting with me to make enhancements
to ZODB and to work on their ZODB-based application.

ZeroDB decided to focus on their Hadoop-based product for now.

I plan to offer ZODB support, consulting development, and training.
If you're interested, let me know.

ZeroDB
======

Two products:

- Database that stores data encrypted at rest.

- Big-data analysis with Hadoop, where data at rest are encrypted.

I worked on the first product. :)

ZEO on Ayncio
=============

ZEO 4 used asyncore, the original Python Asynchronous I/O framework.

Asyncore has lots of issues and has been deprecated.

Rewrote most of ZEO to use asyncio instead.

Opportunity to clean up and especially simplify many things.

Performance improvements (in most cases). http://j1m.me/zp1

ZODB's API is synchronous
====================================

ZODB is an object-oriented database that seeks to provide
**transparent** persistence.  ZODB tries to provide the illusion that
your entire database is in memory.

ZEO has always used an asynchronous networking library, but that's an
implementation detail, that might change in the future.

JavaScript?
===========

IMO, the sanest way to run ZODB in a browser:

- Run ZODB (a client or a local storage) in a web-worker

- Provide an asynchronous API to your UI code.

Assumes that ZODB has been ported to JS. :)

(I wouldn't use a Python->JS compiler.)

An interesting post on JS databases and asynchronous APIs:
http://bit.ly/2dWxRr5

Prefetch
========

Especially on startup, it can take a long time to get objects loaded.

(You can often mitigate this using a **ZEO client cache**.)

The new ZODB prefetch API::

  conn.prefetch(oid1, oid2, ...)

  conn.prefetch(ghost1, ghost2, ...)

Prefetching starts loads asynchronously, so that objects are far more
likely to be available locally when you need them.

ZEO SSL
=======

- Encryption (of course)

- Certificate authentication (mainly for authenticating services)

Could provide the basis for finer-grained authentication:

- Per user certificates

- Per-user user names and passwords over SSL

ZeroDB does both of these.

Client-side conflict resolution
===============================

ZeroDB stored data encrypted, so the server couldn't perform conflict
resolution.

So I added a server option to let clients do conflict resolution.

Client-side conflict resolution
===============================

Pros

- Works with encrypted data.

- No longer need custom classes on the server for conflict resolution.

- Reduces server load.

- Opens the door to non-Python servers.

Client-side conflict resolution
===============================

Cons

- Increases round trips for conflicting transactions.

- Doesn't support conflict-resolution in undo.

Fun opportunity
===============

With conflict resolution on the client, you could potentially do a
much better job of resolving conflicts:

- Can work with objects, not just state.

- Can work with all of the objects in the transaction at once, so, for
  example, one might be able to resolve things like BTree bucket
  splits.

I'd like to **move** responsibility for **conflict resolution** up in the stack
**to ZODB**.

Object-level locks
==================

Currently ZEO locks the database for writes during the *second phase*
of the commit process.

After voting, ZEO has to wait for clients to make final decisions
while that lock is held.

It should be possible to allow multiple non-conflicting transactions
to be in-flight at once. NEO does this.

Object-level-lock attempt for ZEO
=================================

- Got it working.

- It didn't provide a performance win, except when clients were
  connected over slow links.

A big issue is that it's easy to get ZEO servers to be CPU-bound,
especially when running benchmarks. :)

- Partly, the extra computations for managing object-level locks
  offset the benefits, but,

- I think the main issue was that the server was too loaded to respond
  quickly even when locks wouldn't block it.

Interesting ZeroDB experiments
==============================

https://github.com/zerodb/zerodb

- Multi-tenant databases.

  - database split into multiple virtual databases, one per
    user.

  - Separate root objects.

  - Separate invalidations

  - Access control prevents users from seeing each other's records
    (in addition to encryption).

  - Super user can CRUD users/sub-databases

- Certificate and password-based authentication.

RelStorage, NEO, ZEO unification
================================

- ZODB uses a simpler, cleaner implementation of Multi-Version
  concurrency control based on what NEO does.

- This model fits RelStorage better as well.

- RelStorage is no-longer a special case.

  - Other ZODB storages are now *adapted* to provide an API similar to
    RelStorage's and the adapter employs the NEO-inspired.

(Oh, BTW, RelStorage has a maintainer again!)

Inconsistency between ZEO clients
=================================

Scenario:

- Object added on one client in response to a web request.

- Next web request hits a different client and the object isn't there.

This was a potential problem for RelStorage in the past depending on the
poll-interval setting.

Why inconsistent data?
======================

- Each client: has a **consistent** view of the database as of a **point in
  time**,

- But different clients see the database as of different times
  (because networks aren't instantaneous. :))

Forced synchronization
======================

- ZEO has a new ``server-sync`` option to force a server round-trip
  before each transaction.

- NEO always works this way.

- RelStorage now polls at the start of each transaction.

Significant cost, but worth it for many applications.

Post ZeroDB
===========

- Documentation!

- File-storage 2 and byteserver

- transaction-manager.run

- ???

Documentation
=============

- at http://zodb.org

- More extensive than ever before

  But more would be good.

  - Packing/operational considerations

  - Multi-databases

  - ...

- Executable tests thanks to Manuel.

- You can help! Request missing docs.

File-storage2
==============

- Learns lessons from FileStorage

  - FileStorage worked out much better than I ever imagined.

  - Remove unneeded features (versions and back-pointers)

- Multiple files.

- Wildly less disruptive packing (because packing doesn't have to deal
  with current data).

- Indexes mixed between memory and memory-mapped files.

- Requires external garbage collector.

Byteserver
==========

Alternate ZEO server implementation.

- Rust

  - Very fast (faster than Go for the most part)

  - Stack-based memory management.

  - No GIL :)

- Includes a file-storage 2 implementation.

- New API between server and storage, for speed rather than plugability.

- Object-level locks.

- Nearly as easy to set up and operate as ZEO.

Byteserver performance
======================

Initial tests promising.

Need more testing and maybe optimization.

Byteserver work remaining
=========================

- Performance

- Tests (have smoke tests now)

- Many features, including: packing, replication, full storage API, ...

ZODB Ideas
==========

- More speed

- More documentation

- OO conflict resolution

- Ability to subscribe to object updates

- Integration with external indexes, like Elastic Search

- Persistent pandas data frames

- Docker images

- ZEO Authorization, maybe modeled on Unix FS.

- Persistent classes?

- Other languages? (JS, Ruby, Scala?)

- ...

Questions/comments?
===================

| jim@jimfulton.info
| http://jimfulton.info/talks/zodb-plone-conf-2016/

.. image:: Survey-App-Reminder-Slide.png
