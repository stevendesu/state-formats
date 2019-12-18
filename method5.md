# Method 5 ?

This is what I'm trying to figure out. Some perfect balance between methods 2
(which automatically prevented duplicates) and 5 (which was normalized and
therefore prevented a lot of weird edge cases)

In a perfect world this state would also make it easy to detect namespace
collisions, because I'm getting sick of how hard it is to detect those. I want
to throw up a red flag if you give a relation the same name as an existing
field, but that's **hard** for some reason. Does it have to be? I think not

## State Format

```
-- In progress, but maybe something like:

{
	models: {
		byId: {
			Employee: {
				id: "Employee",
				fields: {
					byId: {
						firstName: {
							id: "firstName",
							kind: "string"
						},
						lastName: {
							id: "lastName",
							kind: "string"
						}
					},
					allIds: ["firstName", "lastName"]
				},
				forwardRelations: ["{abc-relation-guid}"],
				backRelations: [],
				namespace: {
					type: true,
					id: true,
					firstName: true,
					lastName: true,
					speaks: true
				}
			},
			Language: {
				id: "Language",
				fields: {
					byId: {
						name: {
							id: "name",
							kind: "string"
						}
					},
					allids: ["name"]
				},
				forwardRelations: [],
				backRelations: ["{abc-relation-guid}"],
				namespace: {
					type: true,
					id: true,
					name: true,
					speakers: true
				}
			}
		},
		allIds: ["Employee", "Language"]
	},
	relations: {
		byId: {
			"{abc-relation-guid}": {
				id: "{abc-relation-guid}",
				name: "speaks",
				on: "Employee",
				types: ["Language"],
				kind: "many-to-many",
				inverse: "speakers"
			}
		},
		allIds: ["{abc-relation-guid}"]
	}
}
```

## Thoughts

 - By replacing "id" with "name" in the models, fields, and relations then we
   can make it impossible to have duplicates again
    - This re-introduces the issue that if we rename a model we need to update
      all references to it. But renaming models is probably done less often than
      checking for duplicates
 - Because relations are many-to-many, it makes sense to keep them separated
   from models. But fields are one-to-many (a field has one model, a model has
   many fields) meaning we may be able to de-normalize a bit there and actually
   SIMPLIFY our code since we'll embed the strong dependency (of a field
   "belonging to" a model) into our schema
 - In order to trivially detect namespace collisions, we'd need to keep an
   object with all fields AND relations and the two immutable properties "type"
   and "id" in a single place
    - The same methods used to prevent namespace collisions can be used to
      prevent duplicate models or relations (a namespace collision is just a
      duplicate detection problem, but looking at fields+relations instead of
      model names)
 - If we use an ID instead of a name, there's no reason the ID has to have any
   relation to the model. Saying "model1" is no different from saying "3" or
   "{abc-random-guid}"
 - I may be able to abstract the concepts of "field" and "relation". For a
   one-to-many the actual implementation details of a relation ARE a field (e.g.
   "employee_id" field on a photo model). For a many-to-many the actual
   implementation detail is a table. But in either case when "Employee" has a
   relation, the actual relation doesn't exist on the Employee table - yet the
   Employee **model** should have a member variable
 - While I don't **like** the idea of having to update multiple places in my
   state tree when one variable changes (e.g. if the relation's name changes, I
   don't want to have to update `state.relations[x].name` AND
   `state.models[x].namespace`) - I could do so using a getter / setter to hide
   the underlying process
    - I **REALLY** don't like the idea
 - Currently I act like "employee.speaks" and "language.speakers" are two
   DIFFERENT relationships. But there's no reason this has to be the case. The
   point of Sleeper API is to enforce best practices and establish standards so
   that API creation and use is quick, easy, and consistent. So I could just as
   easily establish the standard that "all relationships must be bi-directional"
    - If I did this, then I would need something like `fromName` and `toName`
      to distinguish "speaks" from "speakers"
 - I may drop the "id" field on the interior models. I get why it's there. If
   you pass a "field" to a function, you can know *which* field was passed by
   just checking `field.id`. But it's not the kind of thing I think I would use
   very often, and having it there leads to the risk of inconsistency (what
   happens if `field.id` doesn't match the field's actual ID?)

## Current Strategy

 - I went with using the name as the ID for models and fields, both of which
   are created rather frequently and have immediate namespace collision issues
   (you can't have two models of the same name or two fields of the same name on
   a single model)
 - I made fields a child of models since you *CAN* have two fields of the same
   name *across* models, and it makes it easier to grab a single model and then
   see all relevant info
 - Relations I used a GUID key since relations don't have a "name" that maps to
   a meaningful database entity - instead they may map in multiple ways
 - <strike>I removed the reference to relations under models. This means it will
   be less efficient to list all relations for a given model, but that's not a
   task I plan to do very often. It will only matter when deleting a model (to
   clean up existing relations), which is done less often than, say, displaying
   a table with model data</strike>
 - I split relations into `forwardRelations` and `backRelations` so I knew which
   field to use for namespacing
 - I kept the "byId" and "allIds" concept for ordering, <strike>but did not
   include a "namespace" object. While it would make checking for collisions
   easier, I only really need to check for collisions when a field or relation
   is created, which is, again, a trigger point where I can do something
   slightly slower or more complex</strike>
 - I went with the "namespace" object because if I'm going to update a
   `forwardRelations` and `backRelations`, anyway, why not update a namespace
   object, as well?
 - By deciding that all relations must be bi-directional I was able to eliminate
   half of the relations in the state but add an "inverse" field to them. I went
   with the name "inverse" to mimic the OrbitJS schema (same reason I went with
   "kind", "type", etc)
 - I established that relation.type(s) should be an array to allow for
   polymorphic relations

## Another Example

I wanted to demonstrate a more complex schema using this format. This comes
from the OrbitJS docs

```
{
	models: {
		byId: {
			Star: {
				id: "Star",
				fields: {
					byId: {
						name: {
							id: "name",
							kind: "string"
						}
					},
					allIds: ["name"]
				},
				forwardRelations: ["{abc-relation-guid}"],
				backRelations: [],
				namespace: {
					type: true,
					id: true,
					name: true,
					celestialObjects: true
				}
			},
			Planet: {
				id: "Planet",
				fields: {
					byId: {
						name: {
							id: "name",
							kind: "string"
						},
						classification: {
							id: "classification",
							kind: "string"
						},
						atmosphere: {
							id: "atmosphere",
							kind: "boolean"
						}
					},
					allids: ["name", "classification", "atmosphere"]
				},
				forwardRelations: ["{xyz-relation-guid}"],
				backRelations: ["{abc-relation-guid}"],
				namespace: {
					type: true,
					id: true,
					name: true,
					classification: true,
					atmosphere: true,
					star: true,
					moons: true
				}
			},
			Moon: {
				id: "Moon",
				fields: {
					byId: {
						name: {
							id: "name",
							kind: "string"
						}
					},
					allIds: ["name"]
				},
				forwardRelations: [],
				backRelations: ["{abc-relation-guid}", "{xyz-relation-guid}"],
				namespace: {
					type: true,
					id: true,
					name: true,
					star: true,
					planet: true
				}
			}
		},
		allIds: ["Star", "Planet", "Moon"]
	},
	relations: {
		byId: {
			"{abc-relation-guid}": {
				id: "{abc-relation-guid}",
				name: "celestialObjects",
				on: "Star",
				types: ["Planet", "Moon"],
				kind: "one-to-many",
				inverse: "star"
			},
			"{xyz-relation-guid}": {
				id: "{xyz-relation-guid}",
				name: "moons",
				on: "Planet",
				types: ["Moon"],
				kind: "one-to-many",
				inverse: "planet"
			}
		},
		allIds: ["{abc-relation-guid}", "{xyz-relation-guid}"]
	}
}
```

### List all models

```
// Names:
state.models.allIds

// Data:
state.models.byId

// Data as array:
state.models.allIds.map(id => state.models.byId[id])
```

### List all relations

```
// Names aren't too useful (GUIDs)

// Data:
state.relations.byId

// Date as array:
state.relations.allIds.map(id => state.relations.byId[id])
```

### List all fields *for a model*

```
// Names:
state.models.byId[XXX].fields.allIds

// Data:
state.models.byId[XXX].fields.byId

// Data as array:
state.models.byId[XXX].fields.allIds.map(id => state.models.byId[XXX].fields.byId[id])
```

### List all relations *for a model*

```
state.models.byId[XXX].forwardRelations
	.concat(state.models.byId[XXX].backRelations)
	.map(id => state.relations.byId[id])

// Or (same result, different method):

state.relations.allIds.map(id => state.relations.byId[id])
	.filter(r => r.on == XXX || r.types.includes(XXX))
```

### Count models

```
state.models.allIds.length
```

### Find duplicate model names

```
// Duplicate models are impossible with this state mapping
```

### Find duplicate relations

```
// Duplicate relations are possible, but maybe aren't a problem?
// As I thought about it, I realized that you may link a model to multiple
// distinct and named instances of a different model. For instance, consider
// linking Person to Photo with the following 3 relations:
//   - birthPhoto (one-to-one)
//   - selfies (one-to-many)
//   - pictures (one-to-many)
//
// The issue of two relations sharing a field name (e.g. person.selfies) is
// solved by checking for namespace collisions, below
```

### Ensure no namespace collisions

```
// SO LONG AS I keep the "namespace" object up to date (every time a field or
// relation is added, I add it to the namespace and every time one is removed
// I remove it from the namespace) then this schema will prevent collisions
```
