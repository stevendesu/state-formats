# Method 1

This is a "simple" mapping, first-pass attempt, just plugging in what made sense
to me for the state format. We have models, models have relations and fields.

## State Format

```
{
	models: [
		{
			name: "Employee",
			fields: [
				{ name: "firstName", kind: "string" },
				{ name: "lastName", kind: "string" }
			],
			relations: [
				{ name: "speaks", type: "Language", kind: "toMany" }
			]
		},
		{
			name: "Language",
			fields: [
				{ name: "name", kind: "string" }
			],
			relations: [
				{ name: "speakers", type: "Employee", kind: "toMany" }
			]
		}
	]
}
```

## Example Uses

### List all models

```
state.models
```

### List all relations

```
[].concat.apply([], state.models.map(m => m.relations))

// Although the above isn't very useful. Better would be:

[].concat(apply([], state.models.map(m => (
	m.relations.map(r => {...r, on: m})
))))
```

### List all fields *for a model*

```
state.models[XXX].fields
```

### List all relations *for a model*

```
state.models[XXX].relations
```

### Count models

```
state.models.length
```

### Find duplicate model names

```
let counts = state.models.reduce((acc, model) => ({
	...acc,
	[model.name]: (acc[model.name] || 0) + 1
}), {});
Object.keys(counts).filter(model => counts[model] > 1);
```

### Find duplicate relations

```
let dupes = [];
state.models.forEach(model => {
	let counts = model.relations.reduce((acc, relation) => ({
		...acc,
		[relation.name]: (acc[relation.name] || 0) + 1
	}), {});
	dupes = dupes.concat(
		Object.keys(counts).filter(relation => counts[relation] > 1)
	);
});
```

### Ensure no namespace collisions

```
state.models.forEach(model => {
	let names = [].concat.apply([], [
		["type", "id"],
		model.fields.map(field => field.name),
		model.relations.map(relation => relation.name)
	]);
	let counts = names.reduce((acc, name) => ({
		...acc,
		[name]: (acc[name] || 0) + 1
	}), {});
	Object.keys(counts).filter(name => counts[name] > 1);
})
```

## Pros

 - As you can tell from above, listing all models is stupidly simple with this
   setup
 - Listing all of the data *about* a model (all fields *for a model* or all
   relations *for a model*) is also stupidly simple
 - Since we're dealing with arrays directly (an array of models, an array of
   fields, an array of relations) there's an easy mapping to table rows, we can
   trivially count, and the models retain their order
 - The "on" field for a relation is implicit (via its parent) which can reduce
   the burden of creating new relations

## Cons

 - Since the "name" field is buried within each array element, we have to
   traverse the entire array to find duplicates. This makes namespace collision
   detection slow and difficult
 - Arrays allow for duplicate elements, necessitating checks for models with the
   same name, fields with the same name, relations with the same name, etc
 - The "on" field for a relation is implicit, which is implicit design
 - Since a relation requires two models to exist, putting it within a single
   model will lead to weird edge cases (if you delete a model, you must find
   any relations which mention is *in all models* to clean up)
