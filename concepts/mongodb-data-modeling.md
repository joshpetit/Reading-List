[Article](https://docs.mongodb.com/manual/core/data-modeling-introduction/)

Desinging data models center around how application represents the relationship
between data.

### Denormalized

Embedded documents capture a relationship by beinga single document structure.
These embeded documents are called "denormalized data models".

```json
{
	"id": id,
	"username": "1rqwaf",
	"contact": {
		"phone": "aksfiasf",
		"email": "asjfias"
	},
	"access": {
		"level": 1,
		"group": "dev"
	}:
}
```


### References

References store relationship through a link from one doc to another

```json
// user doc
{
	id: "asofka",
	username: "jasfija"
}

// Contact Doc
{
	id: "obj id",
	phone: "asfikaisf",
	email: "asfiapsf"
}

//Access Doc
{
	...
	userid: "asifjasif",

}
```
