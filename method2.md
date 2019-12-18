# Method 2

This replaces the models array with a models OBJECT - same for fields and
relations. This actually looks almost exactly like the schema specification
used by OrbitJS.

## State Format

```
{
	models: {
		Employee: {
			fields: {
				firstName: { kind: "string" },
				lastName: { kind: "string" }
			},
			relations: {
				speaks: { type: "Language", kind: "toMany" }
			}
		},
		Language: {
			fields: {
				name: { kind: "string" }
			},
			relations: {
				speakers: { type: "Employee", kind: "toMany" }
			}
		}
	}
}
```

## Example Uses

### List all models

```
// As an array (mimic previous example output)
Object.keys(state.models).map(key => state.models[key])

// As an object (different format, same data)
state.models
```

### List all relations

```
let models = Object.keys(state.models).map(key => models[key]);

[].concat.apply([], models.map(m => m.relations))

// Although the above isn't very useful. Better would be:

[].concat(apply([], models.map(m => (
	m.relations.map(r => {...r, on: m})
))))
```

### List all fields *for a model*

```
// As an array (mimic previous example output)
Object.keys(state.models[XXX].fields).map(key => models[XXX].fields[key])

// As an object (different format, same data)
state.models[XXX].fields
```

### List all relations *for a model*

```
// As an array (mimic previous example output)
Object.keys(state.models[XXX].relations).map(key => models[XXX].relations[key])

// As an object (different format, same data)
state.models[XXX].relations
```

### Count models

```
Object.keys(state.models).length
```

### Find duplicate model names

```
// Duplicate models are impossible with this state mapping
```

### Find duplicate relations

```
// Duplicate relations are impossible with this state mapping
```

### Ensure no namespace collisions

```
Object.keys(state.models).forEach(name => {
	let model = state.models[name];
	let names = [].concat.apply([], [
		["type", "id"],
		Object.keys(model.fields),
		Object.keys(model.relations)
	]);
	let counts = names.reduce((acc, name) => ({
		...acc,
		[name]: (acc[name] || 0) + 1
	}), {});
	Object.keys(counts).filter(name => counts[name] > 1);
})
```

## Pros

 - This schema makes duplicate model names, field names, and relation names
   an impossibility -- which I **freaking love**
 - Listings all models or model fields is still really easy

## Cons

 - Relations are still buried under the model (implicit "on" field)
 - Finding namespace collisions is still hard since a duplicate key could exist
   across "field" and "relation" -- but still more efficient than the previous
   method
 - Order is lost with this method - you can no longer definitively say "model 1
   is Employee" and "model 2 is Language". This also means there's no trivial
   mapping to a table
 - Since we're referencing models by name in relations, if you change a modle's
   name you need to update all relations
