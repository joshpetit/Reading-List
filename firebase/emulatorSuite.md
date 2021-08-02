# [Introduction](https://firebase.google.com/docs/emulator-suite)

Allows for unit, integration, and manual testing.

Cloud functions shell can be used to prototype in a REPL

There's a rules playground in the console rules specifier that lets you test
rules, but it's better to use an emulator and test there

# [Getting Started](https://firebase.google.com/docs/emulator-suite/connect_and_prototype)

Dev cycle typically looks like:

1. Prototype features interactively with the emulators and Emulator Suite UI.

2. If you're using a database emulator or the Cloud Functions emulator,
   perform a one-time step to connect your app to the emulators.

3. Automate your tests with the emulators and custom scripts.

### Personal thoughts

I think it makes more sense to have the app connected first and do the action
you are planning on making it do, that way if there is an error you want have to
be recreating documents (which if there is a large document is time consuming).

1. Connect app to the emulator and write the logic to call what you need

2. See that the logic works, then write script to test

From a TDD stand point however it makes sense to do the entire process in this
order (before writing implementation):

1. Write test script (that will fail)

2. Write implementation (just enough to pass script test)

3. See that the test script passes

4. Connect app to emulator and see works appropriately

5. Refactor to make the logic better, more solid.

The only thing that may be discouraging is that you don't see the app working
until step 4, but that's ok if you're going for a TDD approach

---

# Configure Suite

`firebase init emulators` to get the process started.

The security rules paths will be within the `firebase.json` and the rules for specific
services will look like:

```json
{
...
   "firestore": {
      "rules": "firestore.rules"
   }
}
```

Emulator configurations are specified as:

```json
{
...
   "emulators": {
      "firestore": {
         "port": "8080"
      },
      "ui": {
         "enabled": true,
         "port": "2091203921039021"
      }
   }
}
```

## Commands

`emulators:start` - **absolutely no idea**

-  `--only` Specific emulator to start, (firestore, functions, hosting...). Can
   list multiple
-  `--inspect-functions DEBUG_PORT?` Enables breakpoint debugging of functions at
   a port (defaults to `9229`)
-  `--export-on-exit=./DIR` Exports data to DIR specified. If the `--import` flag
   is used the export path will use the same directory specified for that
   command.
-  `--import=./DIR` Exports data to DIR specified. Uses data exported from
   `--export-on-exit`

`emulators:exec scriptPath` - **executes scirpt** Spins up emulators then stop
when script finishes running.

Contains all the same flags as `:start` and also:

-  `--ui` Which runs the emulator ui during execution.

`emulators:export EXPORT_DIR` **export data from a running emulator**. You can
also pass the `--export-on-exit` flag while the emulator is running.

# Containerized Suite Images

You can do
[caching](https://firebase.google.com/docs/emulator-suite/install_and_configure#running_containerized_emulator_suite_images) to speed up workflows. When you don't have `firebase.json` you have to add flags to specify which emulators to run. You should have a json file though honestly.

If you ci need sto use firbease hosting, you need to login to run
`emulators:exec`. Other emulators don't require login though. That can be
configured
[like this](https://firebase.google.com/docs/emulator-suite/install_and_configure#generate_an_auth_token_hosting_emulator_only)

# REST API

List current emulators

```
curl localhost:4400/emulators
```

You can temporarily disable functions (if you want to clear data for example and
not have `onDelete` triggered) by hitting:

```
curl -X PUT localhost:4400/functions/disableBackgroundTriggers
```

it'd be nice if there were a postman collection for this

# Auth

You can signup people from the cli using the [signup api](https://firebase.google.com/docs/reference/rest/auth#section-create-email-password)

When setting up emulator suite you can make them for real or demo project types.
Real project types will use your live services for anything you're not emulating
while demo will fail if you try to use a resource htat is not being emulated.
It's recommended to always use demo projects, herever possible.
