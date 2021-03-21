---
layout: post
title: How to structure data in Firestore?
---

He was reading [Best practices for Cloud Firestore](https://firebase.google.com/docs/firestore/best-practices) when he realized there is no one fits all solution for structuring data. Therefore he shouldn't have expected to find instructions on how to store basic and additional user information in the documentation.

His proposed data model looked like this:

```
members (collection)
-- memberID (document)
---- name (string)
---- ...
---- member_info (sub-collection) 
------ infoID (document)
-------- name (string)
-------- favorite_book (string)
-------- favourite_song (string)
-------- ...
```

He spent more time thinking about business needs and defining the queries he might need in the future. One of the queries he hadn't thought about was to get the most favorite book on his platform. He could have used [collection group queries](https://firebase.google.com/docs/firestore/query-data/queries#collection-group-query) for this purpose but excessive usage of this feature would have lead to reaching the [composite indexes limit](https://firebase.google.com/docs/firestore/quotas#indexes) let alone [maximum depth of subcollections](https://firebase.google.com/docs/firestore/quotas#collections_documents_and_fields). 

He revised his proposal and drafted a new one:

```
members (collection)
-- memberID (document)
---- name (string)
---- ...
member_info (collection) 
-- infoID (document)
---- name (string)
---- favorite_book (string)
---- favourite_song (string)
---- ...
```

This was much better except few common mistakes:

- Using overloaded definitions like `info` or `data` leads to confusion when adding new collections.
- Breaking naming convention. Both `favorite` and `favourite` are correct spellings but he must stick to either American or British English to avoid client-side mix-up.
- Duplicated data. By duplicating the `name` field in both `members` and `member_info` he is calling for unavoidable inconsistencies in the future.

He ended up building this the data model:

```
members (collection)
-- memberID (document)
---- name (string)
---- ...
member_favorites (collection) 
-- memberID (document)
---- favorite_book (string)
---- favorite_song (string)
---- ...
```

This is by no means the ultimate guide for structuring data. As described above, one should think of the use cases and fit the data model to them. It is also important to consider how much flexibility one needs in the long term.
