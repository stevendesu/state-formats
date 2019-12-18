# Method 3

The relations are broken out into a separate root element instead of being a
child of the model. This facilitates things like having a "models" page and a
"relations" page on the app. It also removes an implicit dependency that
relation has on model and replaces it with an explicit one (using the "on" key).

## State Format

```
{
	models: {
		Employee: {
			fields: {
				firstName: { kind: "string" },
				lastName: { kind: "string" }
			}
		},
		Language: {
			fields: {
				name: { kind: "string" }
			}
		}
	},
	relations: [
		{ name: "speaks", on: "Employee", type: "Language", kind: "toMany" },
		{ name: "speakers", on: "Language", type: "Employee", kind: "toMany" }
	]
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
state.relations
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
state.relations.filter(r => r.on == XXX)
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
let counts = state.relations.reduce((acc, relation) => ({
	...acc,
	[relation.on + "|" + relation.name]: (acc[relation.on + "|" + relation.name] || 0) + 1
}), {});
Object.keys(counts).filter(name => counts[name] > 1);
```

### Ensure no namespace collisions

```
Object.keys(state.models).forEach(name => {
	let model = state.models[name];
	let names = [].concat.apply([], [
		["type", "id"],
		Object.keys(model.fields),
		Object.keys(state.relations.filter(r => r.on == name))
	]);
	let counts = names.reduce((acc, name) => ({
		...acc,
		[name]: (acc[name] || 0) + 1
	}), {});
	Object.keys(counts).filter(name => counts[name] > 1);
})
```

## Pros

 - Separating out the relations make them easier to list
 - Separating out the relations means they can be created separately without
   weird implicit dependencies on models. This eliminates edge cases when a
   model is delete, for instance
 - We retain a lot of the simplicity from method 2

## Cons

 - Finding namespace collisions is *still* hard
 - We made finding duplicate relations hard again
 - Models still lack an order, although relations do have an order now
 - We still reference models by name in relations
