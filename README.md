mongoose-flatmatcher
====================

An query pre-processing plugin for Mongoose node.js ORM.

mongoose-flatmatcher allows you fully encapsulate client access to your Mongoose Schemas by freeing them from needing to provide JSON objects with fully qualified MongodDB dot-notation paths and `$in` syntax for nested array document properties.

Effectively, clients simply query for the document properties and values they care about with flat key/value JSON, without needing to know the Mongoose Schema or MongoDB document model it maps to.

Schemas decorated with mongoose-flatmatcher can accept JSON objects and return JSON with dot-notation qualified paths and `$in` syntax to be used as a match predicate in find(), update() and remove() calls by Mongoose against MongoDB document collections.  

## Schema Decoration

mongoose-flatmatcher decorates your Schemas with three methods to support flat matching. They are: 

1. `getFlatMatcher()` accepts flat JSON and returns the correct match predicate JSON, as described above. In normal usage, this is the only call you will need to use.

2. `setFlatMatcherMaxDepth()` sets the maximum recursion allowed when building a matcher for a Schema.  The default is 5. This setting prevents the matcher from recursing indefinitely since cycles in Mongoose Schemas are possible.  The depth equates to the depth of nesting in your Schema, so you should only need to change this if you have a highly nested schema.

3. `getFlatMatcherMaxDepth()` gets the maximum recursion allowed when building a matcher for a Schema.

To decorate your schema:

```javascript
		var Schema = mongoose.Schema
				, flatMatcher = require('./flat_matcher.js');

		var BlogPost = new Schema({
		    title     : String
		  , author    : String
		  , meta      : {
		      , visitors  : Number
		    }
		});
		... 
		...
		// FlatMatcher takes care of the dot-notation qualified path for us, visitors => meta.visitors
		var matchArgs = {title : testTitle, published : true, visitors : 5};    
		BlogPost.plugin(flatMatcher, {}); 								// empty opts hash
		var matcher = BlogPost.getMatcher(matchArgs);
		...
		...
		// Now use the matcher in a query
		BlogPost.find(matcher, function (err, found) {
		...
		...
```

## Motivation

If all property names at all levels in a Schema are unique, then clients can query on any combination of properties in the Schema by passing only a flat JSON of of keys and values.

This is a convenient idiom in any case where:
- Data is accessed through a REST API and you want to support a minimum of mapped accessor resources and allow name/value access through the query string. Since node web app frameworks typically expose the query string as a JSON object, this seamlessly extends that functionality to map the query string into an arbitrary Mongoose schema.
- Data is read or written by various front end calls from Ajax or form submits, and you don't want to leak the abstraction of full dot-notation and `$in` syntax qualification into the client code in many places
- The Schema may change often
- Data may be dynamically selected, such as ad-hoc report querying, and you want to support simple query-string (or other flat access by key-value pair) for different combinations of properties

## Limitations and Issues

- If a property name in a Schema is not unique, clients must, as they would without mongoose-flatmatcher, pass in a fully-qualified property name with dot notation to `getFlatMatcher`. flatmatcher cannot magically disambiguate properties at different levels in a Schema with the same name.
- Retrieving from Mongoose Schema objects in an embedded array using `$in` is not supported fully, and the test for this case is incomplete. This is because retrieving objects in embedded arrays in MongoDB requires passing literals that fully match the objects in the database, but constructed Schema objects in Mongoose add properties for internal implementation that are then in the records in the DB but not easily available to the client needing to match those objects in those documents exactly.
- mongoose-flatmatcher depends on the private implementation of the Mongoose lib Schema class, in particular the 'tree' property of that class.

## License

MIT License

## Author

Mark Weiss