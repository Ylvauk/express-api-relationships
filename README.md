[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# Express API Relationships

## Overview

Including one to many and many to many relationships in an Express API using Mongoose.

## Objectives

By the end of this, developers should be able to:

- Use Express to CRUD on a Mongoose subdocument
- Use Express to CRUD on a Mongoose reference

## Getting Started

Fork and clone this [starter code](../../../express-restaurants-starter). Follow the directions in the repo to download dependencies, add your MongoDB Atlas connection string, and start your development server.

If you get stuck, you can reference the solution branch [here](../../../express-restaurants-starter/tree/solution).

## Restaurants - Single Resource

A restaurant will be our first resource. Restaurant will need `name` and `cuisine`.

- Restaurant Schema and Model
- Restaurant CRUD Routes

```js
//models/restaurant.js

const restaurantSchema = new mongoose.Schema(
	{
		name: {
			type: String,
			required: true,
		},

		cuisine: {
			type: String,
			required: true,
		},
	},
	{
		timestamps: true,
	}
);

module.exports = mongoose.model('Restaurant', restaurantSchema);
```

Set up a controller and add some routes for restaurants in `controllers/restaurants.js`.

```js
//...
// INDEX
// GET /restaurants
router.get('/', (req, res, next) => {
	Restaurant.find()
		.then((restaurants) => res.json(restaurants))
		.catch(next);
});

// SHOW
// GET /restaurants/:id
router.get('/:id', (req, res, next) => {
	const id = req.params.id;
	Restaurant.findById(id)
		.then((restaurant) => res.json(restaurant))
		.catch(next);
});

// CREATE
// POST /restaurants/
router.post('/', (req, res, next) => {
	const restaurantData = req.body;
	Restaurant.create(restaurantData)
		.then((restaurant) => res.status(201).json(restaurant))
		.catch(next);
});

// UPDATE
// PATCH /restaurants/:id
router.patch('/:id', (req, res, next) => {
	const id = req.params.id;
	const restaurantData = req.body;
	Restaurant.findOneAndUpdate({ _id: id }, restaurantData, { new: true })
		.then((restaurant) => res.json(restaurant))
		.catch(next);
});

// DESTROY
// DELETE /restaurants/:id
router.delete('/:id', (req, res, next) => {
	const id = req.params.id;
	Restaurant.findOneAndDelete({ _id: id })
		.then(() => res.sendStatus(204))
		.catch(next);
});

module.exports = router;
```

## Reviews - One to Many Subdocuments

Each restaurant should have its own set of reviews and a single review only
belongs to that one restaurant. We consider this relationship
one-to-many subdocuments.

![One to many diagram. One Restaurant has many Reviews](https://media.git.generalassemb.ly/user/16320/files/441ca000-f75c-11ea-94f7-30405bf9720c)

Restaurant will need `reviews`.
Reviews will need `title` and `body`.

Here we're **not** creating a new MODEL from our schema, we'll be using this schema inside our Restaurant model instead.

- Review Schema
- Review CRUD Routes

```js
//models/review.js
const mongoose = require('../db/connection');

const reviewSchema = new mongoose.Schema(
	{
		title: {
			type: String,
			required: true,
		},
		body: {
			type: String,
			required: true,
		},
	},
	{
		timestamps: true,
	}
);

module.exports = reviewSchema;
```

Update the model for restaurants now...

```js
// models/restaurant.js
...
// import the schema for our reviews:
const reviewSchema = require('./review')


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

Add some review routes in a new file called `reviews.js` in your `controllers` folder:

```js
// controllers/reviews.js
const express = require('express');
const router = express.Router();

// require restaurant model
const Restaurant = require('./../models/restaurant');

// CREATE
// POST /reviews/
router.post('/', (req, res, next) => {
	// get the review data from the body of the request
	const reviewData = req.body;
	// get the restaurant id from the body
	const restaurantId = reviewData.restaurantId;
	// find the restaurant by its id
	Restaurant.findById(restaurantId)
		.then((restaurant) => {
			// add review to restaurant
			restaurant.reviews.push(reviewData);
			// save restaurant
			return restaurant.save();
		})
		// send responsne back to client
		.then((restaurant) => res.status(201).json({ restaurant: restaurant }))
		.catch(next);
});

// DESTROY
// DELETE /reviews/:id
router.delete('/:id', (req, res, next) => {
	const id = req.params.id;
	Restaurant.findOne({ 'reviews._id': id })
		.then((restaurant) => {
			restaurant.reviews.id(id).remove();
			return restaurant.save();
		})
		.then(() => res.sendStatus(204))
		.catch(next);
});

// UPDATE
// PATCH /reviews/:id
router.patch('/:id', (req, res, next) => {
	const id = req.params.id;
	const reviewData = req.body;

	Restaurant.findOne({
		'reviews._id': id,
	})
		.then((restaurant) => {
			const review = restaurant.reviews.id(id);
			review.set(reviewData);
			return restaurant.save();
		})
		.then(() => res.sendStatus(204))
		.catch(next);
});

module.exports = router;
```

Let's add the routes for reviews to our `index.js`. Beneath your restaurants controller:

```js
// Review routes
const reviewsController = require('./controllers/reviews.js');
app.use('/reviews', reviewsController);
```

## Owner - One to Many Reference

A review can only have one author, but an author can have many reviews.
We consider this relationship a one-to-many reference.

![One to many diagram. One User (author) has many Reviews](https://media.git.generalassemb.ly/user/17300/files/e2d68300-2817-11eb-8e5a-d0bc3b9c64df)

Review will need `author` reference.
User will need `name`.

- User Schema and Model

Create a new file called `user.js` in your `models` folder. Add the following code:

```js
// models/user.js
const mongoose = require('../db/connection');

const UserSchema = new mongoose.Schema({
	name: String,
});

const User = mongoose.model('User', UserSchema);

module.exports = User;
```

Add a file called `users.js` to your `controllers` folder with the following:

```js
const express = require('express');
const router = express.Router();
const User = require('../models/user');

router.get('/', (req, res, next) => {
	User.find({})
		.then((users) => res.json(users))
		.catch(next);
});
router.post('/', (req, res, next) => {
	User.create(req.body)
		.then((user) => res.status(201).json(user))
		.catch(next);
});

module.exports = router;
```

Then add the Users routes to your `index.js`:

```js
// User routes
const usersController = require('./controllers/users.js');
app.use('/users', usersController);
```

Let's update the `Review` schema in `models/review.js` to include a REFERENCE to the User model:

```js
const mongoose = require('../db/connection');
const reviewSchema = new mongoose.Schema(
	{
		title: {
			type: String,
			required: true,
		},
		body: {
			type: String,
			required: true,
		},
		reviewer: {
			type: mongoose.Schema.Types.ObjectId,
			ref: 'User',
		},
	},
	{
		timestamps: true,
	}
);
module.exports = reviewSchema;
```

## Testing our API

Be sure your server is running with `npm run dev`. Let's pull up our Postman application and create a new collection of requests for our Restaurant Express API.

### Create a restaurant

Create a `POST` request to create a new restaurant with the following sample data to `http://localhost:8080/restaurants/`:

```json
{
	"name": "Taco Cabana",
	"cuisine": "Tex-Mex"
}
```

Send the request from Postman. You should get back a response that looks something like this:

```json
{
	"_id": "60e9e0eaaa4dea11486f1ae4",
	"name": "Taco Cabana",
	"cuisine": "Tex-Mex",
	"reviews": [],
	"createdAt": "2021-07-10T18:03:22.797Z",
	"updatedAt": "2021-07-10T18:03:22.797Z",
	"__v": 0
}
```

Notice that it comes pre-populated with an empty array for Reviews, our subdocument.

### Create a user

Now let's try creating a User. Our User just needs a name:

```json
{
	"name": "esinator"
}
```

You should get back a response that looks something like this:

```json
{
	"_id": "60e9e185aa4dea11486f1ae6",
	"name": "esinator",
	"__v": 0
}
```

### Create a review

Let's try posting a Review to our Database. Our Review contains a Reference to the User object. It also should have a property called `restaurantId` on it as it's going to be a subdocument embedded in the Restaurant model.

Grab the `_id` from the newly created Restaurant and User from YOUR database (don't just copy and paste the ID from below!), and paste it into the `reviewer` and `restaurantId` fields on the JSON body of your request.

```json
{
	"title": "Do yourself a favor and grab some tacos from Taco C",
	"body": "The tortillas are so soft and addictive. I hate how much I love Taco C.",
	"reviewer": "60e9e185aa4dea11486f1ae6",
	"restaurantId": "60e9e0eaaa4dea11486f1ae4"
}
```

Notice that we get back our Restaurant data, with the review embedded into the `reviews` field as a subdocument:

```json
{
	"restaurant": {
		"_id": "60e9e0eaaa4dea11486f1ae4",
		"name": "Taco Cabana",
		"cuisine": "Tex-Mex",
		"reviews": [
			{
				"_id": "60e9e378fa53001187b576d8",
				"title": "Do yourself a favor and grab some tacos from Taco C",
				"body": "The tortillas are so soft and addictive. I hate how much I love Taco C.",
				"reviewer": "60e9e185aa4dea11486f1ae6",
				"createdAt": "2021-07-10T18:14:16.782Z",
				"updatedAt": "2021-07-10T18:14:16.782Z"
			}
		],
		"createdAt": "2021-07-10T18:03:22.797Z",
		"updatedAt": "2021-07-10T18:14:16.782Z",
		"__v": 1
	}
}
```

Try posting another Review!

```json
{
	"title": "Taco Tuesday is the highlight of my week",
	"body": "Give me all your tacos. I’d have to say, those tacos are on fleek. Side of rice and beans, please.",
	"reviewer": "60e9e185aa4dea11486f1ae6",
	"restaurantId": "60e9e0eaaa4dea11486f1ae4"
}
```

You'll see that it gets added as yet another Review in your Restaurant's `reviews` field:

```json
{
	"restaurant": {
		"_id": "60e9e0eaaa4dea11486f1ae4",
		"name": "Taco Cabana",
		"cuisine": "Tex-Mex",
		"reviews": [
			{
				"_id": "60e9e378fa53001187b576d8",
				"title": "Do yourself a favor and grab some tacos from Taco C",
				"body": "The tortillas are so soft and addictive. I hate how much I love Taco C.",
				"reviewer": "60e9e185aa4dea11486f1ae6",
				"createdAt": "2021-07-10T18:14:16.782Z",
				"updatedAt": "2021-07-10T18:14:16.782Z"
			},
			{
				"_id": "60e9e404fa53001187b576db",
				"title": "Taco Tuesday is the highlight of my week",
				"body": "Give me all your tacos. I’d have to say, those tacos are on fleek. Side of rice and beans, please.",
				"reviewer": "60e9e185aa4dea11486f1ae6",
				"createdAt": "2021-07-10T18:16:36.975Z",
				"updatedAt": "2021-07-10T18:16:36.975Z"
			}
		],
		"createdAt": "2021-07-10T18:03:22.797Z",
		"updatedAt": "2021-07-10T18:16:36.975Z",
		"__v": 2
	}
}
```

Congrats on creating an API that expresses different, more complex relationships among different data models!

![Yay Awkwafina](https://media.giphy.com/media/26u4cqiYI30juCOGY/giphy.gif)

## Hungry for More? Customers - Many to Many Reference

So far, we've examined how to implement many-to-one relationships, whether it be through referencing another model or through subdocuments. However, sometimes we need to express a many-to-many relationship, where one model may have many related documents from another model, and vice versa.

For example, a customer (user) should be able to visit many restaurants and a restaurant may have
many customers. We consider this relationship a many-to-many reference.

We are going to implement the following data relationship:

-**A Restaurant has many Customers. A Customer has many Restaurants.**

Restaurant >--< Customer (many-to-many)

Restaurant will need `customers` references.
Customer (User) will need `restaurants` references.

- Customer Schema and Model
- Customer CRUD Routes

### Referencing Customers in the Restaurant Model

We're going to need to update the Restaurant Model with a `customers` property (`models/restaurant.js`) to hold the ObjectIds of the user documents:

```js
const restaurantSchema = new mongoose.Schema(
	{
		// Your Restaurant schema code goes here
		name: {
			type: String,
			required: true,
		},
		cuisine: {
			type: String,
			required: true,
		},
		reviews: [reviewSchema],
		//add this line!
		customers: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
	},
	{ timestamps: true }
);
```

The property type of ObjectId (or an array of ObjectIds) is always used to implement referencing.

The ref: 'User' is optional, but allows us to use the unicorn of Mongoose methods - populate.

Let's do the same for the User model and give it a property called `restaurants` (in `models/user.js`), which will also be an array that holds the ObjectIds of the restaurants the user has patronized. We'll do this because we might want to access the data of which restaurants a user has visited through the User model as well.

```js
const UserSchema = new mongoose.Schema({
	name: String,
	// add this line!
	restaurants: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Restaurant' }],
});
```

#### Contrasting One-to-Many (1:M) and Many-to-Many (M:M) Relationships

The key difference between a 1:M and a M:M relationship:

In a 1:M relationship, each of the many (child) documents belongs to only one (parent) document. Each time we want add a new relationship - the child document must be created.
In a M:M relationship, existing documents are referenced and the same document can be referenced over and over. New documents are created only if it's the first of its kind.

#### Many:Many CRUD

So, before a many-to-many relationship can be created between two documents (often called an association), those two documents must first exist.

This requires that the app first provide the functionality to create the two resources independent of each other.

Then, creating the association is a matter of adding the ObjectId to an array on the other side of the relationship.

The array property can be on either side (even both, but that's not usually recommended). Usually, the app's functionality reveals which side makes more sense. For example, the viewing of a restaurant with its customers is slightly easier to code by putting the customers array on the Restaurant Model vs. a restaurants array on the User Model. But we can do both, and we'll see the implementation for both in this code example.

> Note: When a relationship exists between the logged in user, for example:
> User ---< Post, it's usually a better practice to add the property that holds the relationship to the other Model, not the User Model.

❓ **Review Questions**:

- What property type is used in schemas to reference other documents?

- True or False: Assuming Movie >---< Performer, when associating a "performer" document with a "movie" document, both documents must already exist in the database.

### Associating Restaurants and Customers

Now that we've added the customers property to the Restaurant model, we're ready to implement the M:M relationship between restaurants and users.

Let's turn to our controllers files, which handle all the business logic for our application. First let's make an endpoint to add a customer to a Restaurant's customers array. We'll want to use `PUT` to update a Restaurant document and add to its customers collection. We also need to import the User model and make sure to add the restaurant to the user's restaurants array. Add the following to your `controllers/restaurants.js` file.

```js
const User = require('../models/user.js');
// add user to customers array
// PUT /restaurants/:restaurantID/users/:userID
router.put('/:id/users/:userId', (req, res, next) => {
	let updatedRestaurant;
	Restaurant.findByIdAndUpdate(
		req.params.id,
		{ $push: { customers: req.params.userId } },
		{ new: true }
	)
		.then((restaurant) => {
			updatedRestaurant = restaurant;
		})
		.then(() => {
			User.findByIdAndUpdate(
				req.params.userId,
				{ $push: { restaurants: req.params.id } },
				{ new: true }
			).then(() => res.json(updatedRestaurant));
		})
		.catch(next);
});
```

While we're here, let's also add the `populate` method onto the Restaurant `GET` show route so that we can see the customers information when we request an individual Restaurant's data.

```js
// SHOW
// GET /restaurants/:id
router.get('/:id', (req, res, next) => {
	const id = req.params.id;
	Restaurant.findById(id)
		.populate('customers')
		.then((restaurant) => res.json(restaurant))
		.catch(next);
});
```

Excellent! Now since we also want to be able to see this data on the User side, let's turn to the User controller and add the `populate` method onto the get request so that we can see the full documents for which restaurants a user has visited.

```js
router.get('/', (req, res, next) => {
	User.find({})
		.populate('restaurants')
		.then((users) => res.json(users))
		.catch(next);
});
```

In Postman, check and see whether your API request for `PUT /restaurants/:restaurantID/users/:userID` works, and allows you to add a User to a Restaurant's `customers` array, and that it also adds the Restaurant to the User's `restaurants` array.

```json
{
	"customers": [
		{
			"restaurants": ["60e9e0eaaa4dea11486f1ae4"],
			"_id": "615b9581d44ff33ce3169490",
			"name": "jimboptimus prime",
			"__v": 0
		}
	],
	"_id": "60e9e0eaaa4dea11486f1ae4",
	"name": "Taco Cabana (aka \"Taco C\")",
	"cuisine": "Tex-Mex",
	"reviews": [
		{
			"_id": "60e9e378fa53001187b576d8",
			"title": "Do yourself a favor and grab some tacos from Taco C",
			"body": "The tortillas are so soft and addictive. I hate how much I love Taco C.",
			"reviewer": "60e9e185aa4dea11486f1ae6",
			"createdAt": "2021-07-15T17:01:00.860Z",
			"updatedAt": "2021-07-15T17:01:00.860Z"
		},
		{
			"_id": "60e9e404fa53001187b576db",
			"title": "Taco Tuesday is the highlight of my week",
			"body": "Give me all your tacos. I’d have to say, those tacos are on fleek. Side of rice and beans, please.",
			"reviewer": "60e9e185aa4dea11486f1ae6",
			"createdAt": "2021-07-15T17:01:00.860Z",
			"updatedAt": "2021-07-15T17:01:00.860Z"
		},
		{
			"_id": "60f07e9800143c00152a6ccb",
			"title": "Deliciosity",
			"body": "Why u so good?!?!?!?!",
			"createdAt": "2021-07-15T18:29:44.543Z",
			"updatedAt": "2021-07-15T18:29:44.543Z"
		}
	],
	"createdAt": "2021-07-10T18:03:22.797Z",
	"updatedAt": "2021-10-05T00:01:32.590Z",
	"__v": 7
}
```

Now as you can imagine, this set up includes a bit of redundancy, since both Users and Restaurants track each other. But there might be use cases where it's useful to have a set up like this. Another example that would require a Many-to-Many model might be Blog Posts and Tags. Having both resources track the other's unique ObjectIds helps make sure that we don't have errors, and we gain some flexibility in terms of how we want to access the data models.

**❓ Essential Questions**

1. True or False: If a "book" document needs to reference its "author" document, a property of type ObjectId must be included in the bookSchema.

2. Describe the difference between 1:M & M:M relationships.

3. What's the name of the method used to replace an ObjectId with the document it references?

## Additional Resources

- [Entity Relationship Diagrams](https://www.smartdraw.com/entity-relationship-diagram/)
- [Subdocuments vs References Tradeoffs](https://stackoverflow.com/questions/21302279/embedded-document-vs-reference-in-mongoose-design-model)
- [Mongoose Subdocuments Docs](http://mongoosejs.com/docs/subdocs.html)
- [Mongoose Populate Docs](http://mongoosejs.com/docs/populate.html)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
   alternative licensing, please contact legal@ga.co.
