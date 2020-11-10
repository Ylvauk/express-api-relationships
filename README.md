[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# Express as an API

## Objectives

By the end of this, developers should be able to:

- Use Express to CRUD on a Mongoose subdocument
- Use Express to CRUD on a Mongoose reference

## Overview

Including one to many and many to many relationships in an Express API using Mongoose.

## Restaurants - Single Resource

A restaurant will be our first resource. Restaurant will need `name` and `cuisine`.

- Restaurant Schema and Model
- Restaurant CRUD Routes

## Reviews - One to Many Subdocuments

Each restaurant should have its own set of reviews and a single review only
belongs to that one restaurant. We consider this relationship
one-to-many subdocuments.

![One to many diagram. One Restaurant has many Reviews](https://media.git.generalassemb.ly/user/16320/files/441ca000-f75c-11ea-94f7-30405bf9720c)

Restaurant will need `reviews`.
Reviews will need `title` and `body`.

- Review Schema
- Review CRUD Routes

## Owner - One to Many Reference

A restaurant can only have one owner, but an owner can have many restaurants.
We consider this relationship a one-to-many reference.

![One to many diagram. One User (owner) has many Restaurants](https://media.git.generalassemb.ly/user/16320/files/323afd00-f75c-11ea-919e-863c8266e780)


Restaurant will need `owner` reference.
Owner will need `name`.

- Owner Schema and Model
- Owner CRUD Routes

## Bonus: Customers - Many to Many Reference

A customer should be able to visit many restaurants and a restaurant may have
many customers. We consider this relationship many-to-many reference.

Restaurant will need `customers` references.
Customer will need `name` and `restaurants` references.

- Customer Schema and Model
- Customer CRUD Routes

## Additional Resources

- [Subdocuments vs References Tradeoffs](https://stackoverflow.com/questions/21302279/embedded-document-vs-reference-in-mongoose-design-model)
- [Mongoose Subdocuments Docs](http://mongoosejs.com/docs/subdocs.html)
- [Mongoose Populate Docs](http://mongoosejs.com/docs/populate.html)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
