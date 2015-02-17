# Find From Publication

Find from publication works around a limitation in Meteor core- there's no way to determine which records, client side, have resulted from a given subscription.

This package works around this issue by tracking the subscription's published document in a special, hidden _metadata_ collection.

### API

#### Server Side
To publish a tracked set of documents, simply call:

```
FindFromPublication.publish(name, publisherFn)
```

This behaves just as `Meteor.publish`, in that you can call `added/removed/changed/ready` or simply return (a set of) cursor(s).

#### Client Side
To get the documents published by a given named publication in a given collection, simply call:

```
Collection.findFromPublication(name, query, options);
```

#### Example

```
var Posts = new Mongo.Collection('posts');

if (Meteor.isServer) {
  FindFromPublication.publish('allPosts', function() {
    return Posts.find();
  });
} 

if (Meteor.isClient) {
  Meteor.subscribe('allPosts');
  
  // in a helper, etc
  var postsCursor = Posts.findFromPublication('allPosts');
}
```


#### Sorting

By default, the documents client-side will be sorted by the order they were added.

### How it works

When you call `added`, we simply add a record to the `subscriptionMetadata` client-only collection. It has the form:

```
{
  _id: 'a-unique-id-for-this-record',
  collectionName: 'posts',
  documentId: 'the-real-document-id',
  publicationName: 'allPosts',
  rank: 7 // a globally increasing rank over all publications.
}
```