# [Firebase Security Rules](https://firebase.google.com/docs/rules)

# Intro

All rules are applied as `OR` statements, so overlapping rules will work give
most access always.

```firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if
          request.time < timestamp.date(2021, 8, 4);
    }
  }
}
```

-  `service` - The product rules apply to (cloud.firestore, firestore.storage)

-  `match` - The path in db or storage bucket rules apply to

You can have function declarations to wrap conditions

## Service

You can only include on service declaration per source files, so firestore and
storage rules must be in separate files.

## Match

The match block uses patterns compared to the `request.path`.

Within a match block, there must be one or more match blocks, allow statements,
or function declarations

-  Partial Match: The path pattern is a prefix match
-  Complete matches: The pattern exactly matches the path

Exampled

```firestore.rules
// Given request.path == /example/hello/nested/path the following
// declarations indicate whether they are a partial or complete match and
// the value of any variables visible within the scope.
service firebase.storage {
  // Partial match.
  match /example/{singleSegment} {   // `singleSegment` == 'hello'
    allow write;                     // Write rule not evaluated.
    // Complete match.
    match /nested/path {             // `singleSegment` visible in scope.
      allow read;                    // Read rule is evaluated.
    }
  }
  // Complete match.
  match /example/{multiSegment=**} { // `multiSegment` == /hello/nested/path
    allow read;                      // Read rule is evaluated.
  }
}
```

The partial match is found when parts (haha maybe that's y it's caled partial)
of the path are found within the request string. In the above example

`/example/hello` is only part of the requested path
(`/example/hello/nested/path`). If the requested path were to be
`/example/other` then it would have matched, but it wasn't so it doesn't :).
Adding the wild cards lets the match be for any deeper nested paths, as the
complete match shows with `{multiSegment=**}`

-  Single-segment Wilcard: things wrapped in curly braces, accessible as a string
   within statement
-  Recursive wildcard: Matches path segments at or below a path. You always
   declare it by adding `=**`

One note about recursive wildcards is that when you use them in conjunction with
a singleSegment, the singleSegment will be of the lowest relative document path.

I.e if you have

`/example/hello/{docId}` as a path structure in firestore and have

`/{example=**}/{greeting}` as a match statement

When you make a request of `/example/hello/doc1` the value of `{greeting}` will
be `{docId}`, not the document right bellow `example`

## Allow

Each match block can have an allow statement. Each allow statement can be
applied to one or more methods

(read, get, list, write, create, and update)

allow types:

-  read: any kind of read
   -  get: Read request for single docs or files
   -  list: Read req for queries
-  write: any kind of write
   -  create: Write new docs or files
   -  update: Write to existing docs or files

Example allow statement:

```
match /users/{userId}/{anyUserFile=**} {
   allow read, delete: if request.auth != null && request.auth.uid == userId;
}
```

You should be cognisant of how the rules may interplay. Try to avoid rules of
the same type from overlapping paths. FOr example

```firestore.rules
service cloud.firestore {
   match /this/overlaps {
      allow read: if condition;

      match subpath {
         allow get: if condition;
      }
   }
}
```

The subpath and superpath have overlapping condions

## Function

Works just like javascript functions :/

To declare a variable use `let`. Only available in v2, no more than 10, must
end with a return

> You can use `exists(path/to/$(var)/doc)` to see if something exists

## Building Conditions

**request**:

-  `.auth`: A jwt that has other auth credentials. More on that
   [here](https://firebase.google.com/docs/rules/rules-and-auth)
-  `.method`: Can be standard methods (i.e read and write) but also be custom
   methods
-  `.params`: Any data not rel
-  `.path`: The path for resource being targeted. non-url safe stuff is encoded
   so that's good.

**resource**:

The current value within the service. Referencing resource in a condition will
require a read.

> You can see whether thing is in list with `a` in `b`

## How security rules work

All match statements should point to documents, not collections

You can put wildcard match in any part of path

```
/{this=**}/is/the/path
```

## Securityt rules & Auth

Within the `request` there are two objects, `uid: string` (the id of the person
authing) and `token`

`token`:

-  email
-  email_verified
-  phone_number
-  name (the display name)
-  sub (firebase uid)
-  firebase.identities (map of identities associated with account)
-  firebase.sign_in_provider (sign in provider used to obtain token)

## User stuff

You can use info in the db to your advantage by querying fields.

```
service cloud.firestore {
  match /databases/{database}/documents/some_collection: {
    write: if request.auth != null && get(/databases/(database)/documents/users/$(request.auth.uid)).data.admin) == true;
   read: if request.auth != null;
  }
}
```

# Writing s.r


# DEBUGGING

To debug security rules you must use the emulator. Wrap a variable you'd like to
see the value of with `debug(VARIABLE)`.

```
    match /events/{userId}/{document=**} {
      allow read, write: if
        debug(
            request.auth.uid == debug(userId)
            );
    }
```

In this example it will first print the userId then print the results of the
entire debug statement (either true or false).
