[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# Express API Relationships

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

```js
//...

const restaurantSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  
  cuisine: {
    type: String,
    required: true
  }
}, {
    timestamps: true
})

module.exports = mongoose.model('Restaurant', restaurantSchema)

```

Set up a controller and add some routes for restaurants

```js
//...
// INDEX
// GET /restaurants
router.get('/', (req, res, next) => {
  Restaurant.find()
    .then(restaurants => res.json(restaurants))
    .catch(next)
})

// SHOW
// GET /restaurants/:id
router.get('/:id', (req, res, next) => {
  const id = req.params.id
  Restaurant.findById(id)
    .then(restaurant => res.json(restaurant))
    .catch(next)
})

// CREATE
// POST /restaurants/
router.post('/', (req, res, next) => {
  const restaurantData = req.body
  Restaurant.create(restaurantData)
    .then(restaurant => res.status(201).json(restaurant))
    .catch(next)
})

// UPDATE
// PATCH /restaurants/:id
router.patch('/:id', (req, res, next) => {
  const id = req.params.id
  const restaurantData = req.body
  Restaurant.findOneAndUpdate({_id: id }, restaurantData, {new: true})
    .then(restaurant => res.json(restaurant))
    .catch(next)
})

// DESTROY
// DELETE /restaurants/:id
router.delete('/:id', (req, res, next) => {
  const id = req.params.id
  Restaurant.findOneAndDelete({_id: id})
    .then(() => res.sendStatus(204))
    .catch(next)
})

module.exports = router
```

## Reviews - One to Many Subdocuments

Each restaurant should have its own set of reviews and a single review only
belongs to that one restaurant. We consider this relationship
one-to-many subdocuments.

![One to many diagram. One Restaurant has many Reviews](https://media.git.generalassemb.ly/user/16320/files/441ca000-f75c-11ea-94f7-30405bf9720c)

Restaurant will need `reviews`.
Reviews will need `title` and `body`.

Here we're **not** creating a model from our schema, we'll be using this schema inside our Restaurant model instead.

- Review Schema
- Review CRUD Routes

```js
//...

const reviewSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  body: {
    type: String,
    required: true
  }
}, {
    timestamps: true
})

module.exports = reviewSchema
```

Update the model for restaurants now...

```js
...
// import the schema for our reviews:
const reviewSchema = require('./review')

const mongoose = require('connection')

const restaurantSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  cuisine: {
    type: String,
    required: true
  },
  reviews: [reviewSchema]
}, {
    timestamps: true
})

module.exports = mongoose.model('Restaurant', restaurantSchema)

```
Add some review routes:

```js
const express = require('express')
const router = express.Router()

// require restaurant model
const Restaurant = require('./../models/restaurant')

// CREATE
// POST /reviews/
router.post('/', (req, res, next) => {
  // get the review data from the body of the request
  const reviewData = req.body
  // get the restaurant id from the body
  const restaurantId = reviewData.restaurantId
  // find the restaurant by its id
  Restaurant.findById(restaurantId)
    .then(restaurant => {
      // add review to restaurant
      restaurant.reviews.push(reviewData)
      // save restaurant
      return restaurant.save()
    })
    // send responsne back to client
    .then(restaurant => res.status(201).json({restaurant: restaurant}))
    .catch(next)
})

// DESTROY
// DELETE /reviews/:id
router.delete('/:id', (req, res, next) => {
  const id = req.params.id
  Restaurant.findOne({ 'reviews._id': id })
    .then(restaurant => {
      restaurant.reviews.id(id).remove()
      return restaurant.save()
    })
    .then(() => res.sendStatus(204))
    .catch(next)
})

// UPDATE
// PATCH /reviews/:id
router.patch('/:id', (req, res, next) => {
  const id = req.params.id
  const reviewData = req.body

  Restaurant.findOne({
    'reviews._id': id,
  })
    .then(restaurant => {
      const review = restaurant.reviews.id(id)
      review.set(reviewData)
      return restaurant.save()
    })
    .then(() => res.sendStatus(204))
    .catch(next)
})

module.exports = router
```



## Owner - One to Many Reference

A review can only have one authorr, but an author can have many reviews.
We consider this relationship a one-to-many reference.

![One to many diagram. One User (author) has many Reviews](https://media.git.generalassemb.ly/user/17300/files/e2d68300-2817-11eb-8e5a-d0bc3b9c64df)


Review will need `author` reference.
User will need `name`.

- User Schema and Model

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
