# miniplex

## 1.0.0-next.6

### Patch Changes

- efa21f2: Typing tweaks.

## 1.0.0-next.5

### Patch Changes

- c12dfc1: **Fixed:** Improved typing of `createEntity`.

## 1.0.0-next.1

### Major Changes

- 4016fb2: 1.0!

### Patch Changes

- 410e0f6: The `World` class can now be instantiated with an initial list of entities like so:

  ```js
  const world = new World({ entities: [entity1, entity2] })
  ```

## 0.11.0-next.0

### Minor Changes

- 769dba7: **Major Breaking Change:** The signature of `addComponent` has been simplified to accept an entity, a component name, and the value of the component:

  ```ts
  /* Before */
  world.addComponent(entity, { position: { x: 0, y: 0 } })

  /* After */
  world.addComponent(entity, "position", { x: 0, y: 0 })
  ```

  The previous API for `addComponent` is now available as `extendEntity`, but _marked as deprecated_.

- b8b2c9b: **Breaking Change:** The API signature of `createEntity` has been simplified in order to improve clarity of the API and reduce complexity in both implementation and types. `createEntity` now only supports a single argument, which _must_ satisfy the world's entity type.

  This will only affect you if you have been using `createEntity` with more than one argument in order to compose entities from partial entities, like so:

  ```js
  const entity = createEntity(position(0, 0), velocity(1, 1), health(100))
  ```

  This always had the issue of `createEntity` not checking the initial state of the entity against the world's entity type. Theoretically, the library could invest some additional effort into complex type assembly to ensure that the entity is valid, but there are enough object composition tools available already, so it felt like an unneccessary duplication.

  Instead, composition is now deferred into userland, where one of the most simple tools is the spread operator:

  ```js
  const entity = createEntity({
    ...position(0, 0),
    ...velocity(1, 1),
    ...health(100)
  })
  ```

- cb6d078: **Breaking Change:** When destroying entities, they are now removed from the world's global list of entities as well as the archetypes' lists of entities using the shuffle-and-pop pattern. This has the following side-effects that _may_ impact your code:

  - Entities are no longer guaranteed to stay in the same order.
  - The entity ID storied in its internal `__miniplex` component no longer corresponds to its index in the `entities` array.

  This change provides significantly improved performance in situations where a large number of entities are continuously being created and destroyed.

- 4d9e51b: **Breaking Change:** Removed the `EntityID` and `ComponentData` types.

## 0.10.5

### Patch Changes

- 5ef5f95: Included RegisteredEntity in ArchetypeEntity. (@benwest)
- c680bdd: Narrowed return type for createEntity (with one argument). (@benwest)

## 0.10.4

### Patch Changes

- 74e34c7: **Fixed:** Fixed an issue with the new iterator syntax on archetypes. (@benwest)

## 0.10.3

### Patch Changes

- 1cee12c: Typing improvements, thanks to @benwest.
- 65d2b77: **Added:** Archtypes now implement a `[Symbol.iterator]`, meaning they can be iterated over directly:

  ```js
  const withVelocity = world.archetype("velocity")

  for (const { velocity } of withVelocity) {
    /* ... */
  }
  ```

  (Thanks @benwest.)

## 0.10.2

### Patch Changes

- 821a45c: **Fixed:** When the world is cleared, archetypes now also get their entities lists cleared.

## 0.10.1

### Patch Changes

- cca39cd: **New:** Archetypes now expose a `first` getter that returns the first of the entities in the archetype (or `null` if it doesn't have any entities.) This streamlines situations where you deal with singleton entities (like a player, camera, and so on.) For example, in `miniplex-react`, you can now do the following:

  ```tsx
  export const CameraRigSystem: FC = () => {
    const player = ECS.useArchetype("isPlayer").first
    const camera = ECS.useArchetype("isCamera").first

    /* Do things with player and camera */
  }
  ```

## 0.10.0

### Minor Changes

- cc4032d: **Breaking Change:** `createEntity` will now, like in earlier versions of this library, mutate the first argument that is passed into it (and return it). This allows for patterns where you create the actual entity _object_ before you actually convert it into an entity through _createEntity_.

### Patch Changes

- b93a831: The internal IDs that are being generated for entities have been changed slightly, as they now start at `0` (instead of `1`) and are always equal to the position of the entity within the world's `entities` array. The behavior of `destroyEntity` has also been changed to `null` the destroyed entity's entry in that array, instead of cutting the entity from it.

  This change allows you to confidently and reliably use the entity ID (found in the internal miniplex component, `entity.__miniplex.id`) when integrating with non-miniplex systems, including storing data in TypedArrays (for which miniplex may gain built-in support at some point in the future; this change is also in preparation for that.)

## 0.9.2

### Patch Changes

- 92cf103: Safer sanity check in `addComponent`

## 0.9.1

### Patch Changes

- 48e785d: Fix sanity check in `removeComponent`

## 0.9.0

### Minor Changes

- b4cee80: **Breaking Change:** `createEntity` will now always return a new object, and not return the one passed to it.
- 544f231: **Typescript:** You no longer need to mix in `IEntity` into your own entity types, as part of a wider refactoring of the library's typings. Also, `createWorld` will now return a `RegisteredEntity<YourEntity>` type that reflects the presence of the automatically added internal `__miniplex` component, and makes a lot of interactions with the world instance safer than it was previously.
- 544f231: **Breaking Change:** Miniplex will no longer automatically add an `id` component to created entities. If your project has been making use of these automatically generated IDs, you will now need to add them yourself.

  Example:

  ```js
  let nextId = 0

  /* Some component factories */
  const id = () => ({ id: nextId++ })
  const name = (name) => ({ name })

  const world = new World()
  const entity = world.createEntity(id(), name("Alice"))
  ```

  **Note:** Keep in mind that Miniplex doesn't care about entity IDs much, since all interactions with the world are done through object references. Your project may not need to add IDs to entities at all; if it does, this can now be done using any schema that your project requires (numerical IDs, UUIDs, ...).

### Patch Changes

- b4cee80: `createEntity` now allows you to pass multiple parameters, each representing a partial entity. This makes the use of component factory functions more convenient. Example:

  ```js
  /* Provide a bunch of component factories */
  const position = (x = 0, y = 0) => ({ position: { x, y } })
  const velocity = (x = 0, y = 0) => ({ velocity: { x, y } })
  const health = (initial) => ({
    health: { max: initial, current: initial }
  })

  const world = new World()

  const entity = world.createEntity(
    position(0, 0),
    velocity(5, 7),
    health(1000)
  )
  ```

  **Typescript Note:** The first argument will always be typechecked against your entity type, so if your entity type has required components, you will need to pass a first argument that satisfies these. The remaining arguments are expected to be partials of your entity type.

- b4cee80: **Breaking Change:** `world.queue.createEntity` no longer returns an entity (which didn't make a whole lot of semantic sense to begin with.)

## 0.8.1

### Patch Changes

- 011c384: Change the API signature of `addComponent` to expect a partial entity instead of name and value, to provide a better interface for component factories:

  ```ts
  const position = (x: number = 0, y: number = 0) => ({ position: { x, y } })
  const health = (amount: number) => ({
    health: { max: amount, current: amount }
  })

  world.addComponent(entity, { ...position(), ...health(100) })
  ```
