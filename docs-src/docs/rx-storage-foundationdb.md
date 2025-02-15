---
title: RxDB on FoundationDB - Performance at Scale
slug: rx-storage-foundationdb.html
description: Combine FoundationDB's reliability with RxDB's indexing and schema validation. Build scalable apps with faster queries and real-time data.
---

# RxDB Database on top of FoundationDB

[FoundationDB](https://www.foundationdb.org/) is a distributed key-value store designed to handle large volumes of structured data across clusters of computers while maintaining high levels of performance, scalability, and fault tolerance. While FoundationDB itself only can store and query key-value pairs, it lacks more advanced features like complex queries, encryption and replication.

With the FoundationDB based [RxStorage](./rx-storage.md) of [RxDB](https://rxdb.info/) you can combine the benefits of FoundationDB while having a fully featured, high performance NoSQL database.

## Features of RxDB+FoundationDB

Using RxDB on top of FoundationDB, gives you many benefits compare to using the plain FoundationDB API:

- **Indexes**: In RxDB with a FoundationDB storage layer, indexes are used to optimize query performance, allowing for fast and efficient data retrieval even in large datasets. You can define single and compound indexes with the [RxDB schema](./rx-schema.md).
- **Schema Based Data Model**: Utilizing a [jsonschema](./rx-schema.md) based data model, the system offers a highly structured and versatile approach to organizing and [validating data](./schema-validation.md), ensuring consistency and clarity in database interactions.
- **Complex Queries**: The system supports complex [NoSQL queries](./rx-query.md), allowing for advanced data manipulation and retrieval, tailored to specific needs and intricate data relationships. For example you can do `$regex` or `$or` queries which is hardy possible with the plain key-value access of FoundationDB.
- **Observable Queries & Documents**: RxDB's observable queries and documents feature ensures real-time updates and synchronization, providing dynamic and responsive data interactions in applications.
- **Compression**: RxDB employs data [compression techniques](./key-compression.md) to reduce storage requirements and enhance transmission efficiency, making it more cost-effective and faster, especially for large volumes of data. You can compress the [NoSQL document](./key-compression.md) data, but also the [binary attachments](./rx-attachment.md#attachment-compression) data.
- **Attachments**: RxDB supports the storage and management of [attachments](./rx-attachment.md) which allowing for the seamless inclusion of binary data like images or documents alongside structured data within the database.


## Installation

- Install the [FoundationDB client cli](https://apple.github.io/foundationdb/getting-started-linux.html) which is used to communicate with the FoundationDB cluster.
- Install the [FoundationDB node bindings npm module](https://www.npmjs.com/package/foundationdb) via `npm install foundationdb --save`. If the latest version does not work for you, you should use the same version as stated in the `storage-foundationdb` job of the RxDB CI `main.yml`.


## Usage

```typescript
import {
    createRxDatabase
} from 'rxdb';
import {
    getRxStorageFoundationDB
} from 'rxdb/plugins/storage-foundationdb';

const db = await createRxDatabase({
    name: 'exampledb',
    storage: getRxStorageFoundationDB({
        /**
         * Version of the API of the FoundationDB cluster..
         * FoundationDB is backwards compatible across a wide range of versions,
         * so you have to specify the api version.
         * If in doubt, set it to 620.
         */
        apiVersion: 620,
        /**
         * Path to the FoundationDB cluster file.
         * (optional)
         * If in doubt, leave this empty to use the default location.
         */
        clusterFile: '/path/to/fdb.cluster',
        /**
         * Amount of documents to be fetched in batch requests.
         * You can change this to improve performance depending on
         * your database access patterns.
         * (optional)
         * [default=50]
         */
        batchSize: 50
    })
});
```

## Multi Instance

Because FoundationDB does not offer a [changestream](https://forums.foundationdb.org/t/streaming-data-out-of-foundationdb/683/2), it is not possible to use the same cluster from more than one Node.js process at the same time. For example you cannot spin up multiple servers with RxDB databases that all use the same cluster. There might be workarounds to create something like a FoundationDB changestream and you can make a Pull Request if you need that feature.
