.. _ft-index:

=====================
Fractal Tree Indexing
=====================

Fractal Tree indexing is the technology behind TokuDB and TokuMX, protected by multiple patents. It enhances the traditional B-tree data structure used in other database engines, and optimizes performance for modern hardware and data sets.

Background
----------

Although the B-tree data structure was invented for large blocks of data, performance today is limited by I/O bandwidth. This happens because the size of a production database generally exceeds available main memory. As a consequence, most leaves are stored on disk, not in RAM. And inserting into a leaf that is not in main memory requires a disk I/O operation.

It would be too expensive to increase RAM on par with growing storage needs for big data. So it is important to optimize disk I/O. This is where Fractal Tree indexing comes into play.

Buffers
-------

Like a B-tree, a Fractal Tree index is a tree data structure. However, besides pivots, each internal node in a Fractal Tree index also contains buffers. These buffers are used to temporarily store insertions. When a buffer is full, it is flushed to children nodes. This ensures that an I/O operation performs a lot of useful work, instead of just a small write per I/O.

Reading data is not affected by buffers. Querying a database with the Fractal Tree index data structure involves the same algorithmic complexity as in the case of a B-tree. And there is no data loss, because queries follow the path from root to leaf passing through all messages (insertions, deletions, and updates) in buffers. So a query will know the current state of data even if changes have not yet been propagated to corresponding leaves.

Buffers also get serialized to disk, so messages in internal nodes will not be lost in case of a crash or power outage. If a write happened after a checkpoint and before a crash, then recovery will replay the operation from the log.

http://www.tokutek.com/2013/07/tokumx-fractal-treer-indexes-what-are-they/

http://www.tokutek.com/wp-content/uploads/2012/11/Fractal-Tree-Technology-and-the-Art-of-Indexing.pdf

http://cdn.oreillystatic.com/en/assets/1/event/36/How%20TokuDB%20Fractal%20Tree%20Databases%20Work%20Presentation.pdf

http://www.slideshare.net/mongodb/20121024-mongodbboston

https://en.wikipedia.org/wiki/Fractal_tree_index
https://en.wikipedia.org/wiki/B-tree