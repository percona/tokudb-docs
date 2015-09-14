.. _ft-index:

=====================
Fractal Tree Indexing
=====================

Fractal Tree indexing is the technology behind TokuDB and TokuMX, protected by multiple patents. It enhances the traditional B-tree data structure used in other database engines, and optimizes performance for modern hardware and data sets.

Background
----------

Although the B-tree data structure was invented for large blocks of data, performance today is limited by I/O bandwidth. This happens because the size of a production database generally exceeds available main memory. As a consequence, most leaves in a tree are stored on disk, not in RAM. And inserting into a leaf that is not in main memory requires a disk I/O operation.

It would be too expensive to increase RAM on par with growing storage needs for big data. So it is important to optimize disk I/O. This is where Fractal Tree indexing comes into play.

Buffers
-------

Like a B-tree, a Fractal Tree index is a tree data structure. However, besides pivots, each internal node in a Fractal Tree index also contains buffers. These buffers are used to temporarily store insertions. When a buffer is full, it is flushed to children nodes. This ensures that an I/O operation performs a lot of useful work when messages reach leaves on disk, instead of just a small write per I/O.

Reading data is not affected by buffers in terms of data consistency. Querying a database with the Fractal Tree index data structure involves the same algorithmic complexity as in the case of a B-tree. And there is no data loss, because queries follow the path from root to leaf passing through all messages (insertions, deletions, and updates) in buffers. So a query will know the current state of data even if changes have not yet been propagated to corresponding leaves.

Each message is stamped with a unique message sequence number (MSN) when it is stored in a non-leaf node message buffer. The MSN is unique within each fractal tree.  The MSN maintains the order of messages and ensures that messages are only applied once to leaf nodes when the leaf node is updated by messages buffered by its parent nodes.

Buffers also get serialized to disk, so messages in internal nodes will not be lost in case of a crash or power outage. If a write happened after a checkpoint and before a crash, then recovery will replay the operation from the log.