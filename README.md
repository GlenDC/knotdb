# knotdb

A Free and Open Source E2E Database written in Rust

## Draft: Design

Storage will be done using https://github.com/chrislusf/seaweedfs

- Features enabled:
    - erasure coding
    - use level db: https://github.com/chrislusf/seaweedfs/wiki/Optimization#insert-with-your-own-keys
    - https://github.com/chrislusf/seaweedfs/wiki/Optimization#insert-with-your-own-keys
        - volume id (check what volume is possible)
        - needle id: public key of owner (??)
        - file cookie: signed hash of file content
    - https://github.com/chrislusf/seaweedfs/wiki/Optimization#collection-as-a-simple-name-space: could be useful for our different files
    - enable logging: https://github.com/chrislusf/seaweedfs/wiki/Optimization#collection-as-a-simple-name-space

Test database we can run locally. For Bucket (first app that will use it for Plabayo)
we can setup a kubernetes cluster (Probably) via digital ocean. Simplest plans will
suffice for now.

### Files

Everything is a file, that's the philosophy here. As it's meant as an E2E database,
there is no schema validation. What is validated:

- always: hash of the file, does it match the content and is it from the used public key (owner);
- opt-in: file size limitation

At its simplest a file (= object) is owned by a single person in that case it is stored as:

```
/<pk>/<hash>
```

Where `pk` is the public key of the owner, and the hash is a signed hash (by that owner) and from the file content.

Files can also be owned by multiple people however. In that case the file content is a symbolic link to the actual file:

> TODO: work out digitally

Goals: Ensure all data for a user can always be found simply by known its PK.

Questions to answer:

- What if the group "owning" a file changes, can we do so without having to update the info for all users?
- Can we keep the indirection to one layer?
- Can we have multiple owners?
- What about other permission changes, can we easily have Read-only? and what about R/W but not owner?
- Should hash really be part of name? or perhaps better to have it as a shorter random id that never changes? Otherwise we will constantly need to delete/create each time file content changes. If so instead we would work with a schema `<hash><blob>` instead, where the binary file (blob) is prefixed with a hash. Optionally we can also put a header in the middle, but only if we really require extra metadata, and not just for fun...
