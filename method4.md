# Method 4

This is a normalized state shape, as recomended by the
[Redux docs](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape)

## State Format

```
{
	models: {
		byId: {
			"model1": {
				id: "model1",
				name: "Employee",
				fields: ["field1", "field2"],
				relations: ["relation1"]
			},
			"model2": {
				id: "model2",
				name: "Language",
				fields: ["field3"],
				relations: ["relation2"]
			}
		},
		allIds: ["model1", "model2"]
	},
	fields: {
		byId: {
			"field1": {
				id: "field1",
				name: "firstName",
				kind: "string"
			},
			"field2": {
				id: "field2",
				name: "lastName",
				kind: "string"
			},
			"field3": {
				id: "field3",
				name: "name",
				kind: "string"
			}
		},
		allIds: ["field1", "field2", "field3"]
	},
	relations: {
		byId: {
			"relation1": {
				id: "relation1",
				name: "speaks",
				on: "model1",
				type: "model2",
				kind: "toMany"
			},
			"relation2": {
				id: "relation2",
				name: "speakers",
				on: "model2",
				type: "model1",
				kind: "toMany"
			}
		},
		allIds: ["relation1", "relation2"]
	}
}
```

## Example Uses

### List all models

```
// As an array (mimic previous example output)
state.models.allIds.map(id => state.models.byId[id])

// As an object (different format, same data)
state.models.byId
```

### List all relations

```
// As an array (mimic previous example output)
state.relations.allIds.map(id => state.relations.byId[id])

// As an object (different format, same data)
state.relations.byId
```

### List all fields *for a model*

```
state.models.byId[XXX].fields.map(id => state.fields.byId[id])
```

### List all relations *for a model*

```
state.models.byId[XXX].relations.map(relation => state.relations.byId[relation])
```

### Count models

```
state.models.allIds.length
```

### Find duplicate model names

```
let counts = state.models.allIds.reduce((acc, id) => ({
	...acc,
	[state.models.byId[id].name]: (acc[state.models.byId[id].name] || 0) + 1
}), {});
Object.keys(counts).filter(name => counts[name] > 1);
```

### Find duplicate relations

```
let counts = state.relations.allIds.reduce((acc, id) => ({
	...acc,
	[state.relations.byId[id].on + "|" + state.relations.byId[id].name]: (acc[state.relations.byId[id].on + "|" + state.relations.byId[id].name] || 0) + 1
}), {});
Object.keys(counts).filter(name => counts[name] > 1);
```

### Ensure no namespace collisions

```
state.models.allIds.forEach(id => {
	let model = state.models.byId[id];
	let names = [].concat.apply([], [
		["type", "id"],
		model.fields.map(f_id => state.fields.byId[f_id].name),
		model.relations.map(r_id => state.relations.byId[r_id].name)
	]);
	let counts = names.reduce((acc, name) => ({
		...acc,
		[name]: (acc[name] || 0) + 1
	}), {});
	Object.keys(counts).filter(name => counts[name] > 1);
})
```

## Pros

 - We retain all of the benefits of separating our models and relations
 - We retain an ordering of models, fields, and relations (allowing for a
   proper mapping to table rows or other things)
 - Since we reference models by ID now and not by name, if we update a model's
   name it won't risk breaking existing relations

## Cons

 - Finding namespace collisions is ***still*** hard - *Jesus*
 - Finding duplicate models *or* relations is hard now... did we just take a
   step backwards by normalizing our data?
 - I don't like the fact that the ID gets repeated twice in the state, but I
   see the reason why
