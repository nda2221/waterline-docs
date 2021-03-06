# Many-to-Many Associations

A many-to-many association states that a model can be associated with many other models and vice-versa.
Because both models can have many related models a new join table will need to be created to keep track
of these relations.

Waterline will look at your models and if it finds that two models both have collection attributes that
point to each other, it will automatically build up a join table for you.

Because you may want a model to have multiple many-to-many associations on another model a `via` key
is needed on the `collection` attribute. This states which `model` attribute on the one side of the
association is used to populate the records.

You will also need to add a `dominant` property on one side of the association. This allows Waterline
to know which side it can write the join table to in the case of different connections.

Using the `User` and `Pet` example, let's look at how to build a schema where a `User` may have many
`Pet` records and a `Pet` may have multiple owners.

```javascript
// A user may have many pets
var User = Waterline.Collection.extend({

  identity: 'user',
  connection: 'local-postgresql',

  attributes: {
    firstName: 'string',
    lastName: 'string',

    // Add a reference to Pet
    pets: {
      collection: 'pet',
      via: 'owners',
      dominant: true
    }
  }
});

// A pet may have many owners
var Pet = Waterline.Collection.extend({

  identity: 'pet',
  connection: 'local-postgresql',

  attributes: {
    breed: 'string',
    type: 'string',
    name: 'string',

    // Add a reference to User
    owners: {
      collection: 'user',
      via: 'pets'
    }
  }
});
```

Now that the `User` and `Pet` models have been created and the join table has been setup
automatically, we can start associating records and querying the join table. To do this, let's add a
`User` and `Pet` and then associate them together.

There are two ways of creating associations when a many-to-many association is used. You can associate
two existing records together or you can associate a new record to the existing record. To show how
this is done we will introduce the special methods attached to a `collection` attribute: `add` and `remove`.

Both these methods are sync methods that will queue up a set of operations to be run when an instance
is saved. If a primary key is used for the value on an `add`, a new record in the join table will be
created linking the current model to the record specified in the primary key. However if an object
is used as the value in an `add`, a new model will be created and then the primary key of that model
will be used in the new join table record. You can also use an array of previous values.

## When Both Records Exist

```javascript
// Given a User with ID 2 and a Pet with ID 20

User.findOne(2).exec(function(err, user) {
  if(err) // handle error

  // Queue up a record to be inserted into the join table
  user.pets.add(20);

  // Save the user, creating the new associations in the join table
  user.save(function(err) {});
});
```

## With A New Record

```javascript
User.findOne(2).exec(function(err, user) {
  if(err) // handle error

  // Queue up a new pet to be added and a record to be created in the join table
  user.pets.add({ breed: 'labrador', type: 'dog', name: 'fido' });

  // Save the user, creating the new pet and associations in the join table
  user.save(function(err) {});
});
```

## With An Array of New Record

```javascript
// Given a User with ID 2 and a Pet with ID 20, 24, 31

User.findOne(2).exec(function(err, user) {
  if(err) // handle error

  // Queue up a record to be inserted into the join table
  user.pets.add([ 20, 24, 31 ]);

  // Save the user, creating the new pet and associations in the join table
  user.save(function(err) {});
});
```

Removing associations is just as easy using the `remove` method. It works the same as the `add`
method except it only accepts primary keys as a value. The two methods can be used together as well.

```javascript
User.findOne(2).exec(function(err, user) {
  if(err) // handle error

  // Queue up a new pet to be added and a record to be created in the join table
  user.pets.add({ breed: 'labrador', type: 'dog', name: 'fido' });

  // Queue up a join table record to remove
  user.pets.remove(22);

  // Save the user, creating the new pet and syncing the associations in the join table
  user.save(function(err) {});
});
```
