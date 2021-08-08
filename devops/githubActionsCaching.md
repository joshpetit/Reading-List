# [Caching Dependencies in Workflows](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)

To cache dependencies you need to use github's
[`cache`](https://github.com/actions/cache) action. If you're using ruby check
[this
out](https://github.com/ruby/setup-ruby#caching-bundle-install-automatically)

To cache and restore dependencies for node managers, use
[this](https://github.com/actions/setup-node)

Be careful not saving sensitive information in the cache of public repos. When
people make PRs to your repo with actions, cli programs like docker login save
credentials to a config file people will be able to see.

## Artifacts vs cache

Use caching when you want to reuse files that don't change often between jobs

Use artifacts when you want to save files produced by a job to view after a
workflow has ended.

## Accessing Caches

Branches have access to the cache of the base branch and current branch.

## Cache action

The cache action restores caches based on a provided `key`. When a cache is
found, the action resores it to the `path` specified in the configurations

### Input Params

? means optional.

- `key` - Created when saving cache, used to search for a cache.
- `path` - Path to restore or cache. Can be absolute or relative.
  They can be single files, directories, or glob patterns. Example:

  ```yaml
  - name: Cache Gradle packages
  	uses: actions/cache@v2
  	with:
  	path: |
  	~/.gradle/caches
  	~/.gradle/wrapper
  ```

  v1 only supports directories though.

- `restore-keys?` - List of alternative keys to find cache if no hit occured for
  a key

### Output Params

`cache-hit` - Boolean that indicates match was found for key

Example `cache` action:

```yaml
name: Caching with npm

on: push

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Cache node modules
        uses: actions/cache@v2
        env:
          cache-name: cache-node-modules
        with:
          # npm cache files are stored in `~/.npm` on Linux/macOS
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
            ${{ runner.os }}-build-
            ${{ runner.os }}-

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm build
 
      - name: Test
        run: npm test
```
