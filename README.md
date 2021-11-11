# Untitled File Editor Project

A peer-to-peer text editor built on [Hypercore's new multiwriter Autobase](https://github.com/hypercore-protocol/autobase).

![screenshot.png](screenshot.png)

## Implementation notes

### Hypercore schemas

The repo is an Autobase which uses oplog inputs and a Hyperbee for the index index. All data is encoded using msgpack.

The Hyperbee index uses the following layout:

```
/_meta = {
  schema: 'untitled-file-editor-project',
  writerKeys: Buffer[]
}
/trees/$tree = {
  commit: string, // id of the commit that created this tree
  conflicts: number[], // seq numbers of currently-conflicting trees
  files: [
    // path        blob-ref (hash)
    ['/foo.txt', 'sha256-123ad..df'],
    ['/bar.txt', 'sha256-dkc22..12']
  ]
}
/commits/$tree/$id = {
  id: string, // random generated ID
  writer: Buffer, // key of the core that authored the commit
  parents: string[] // IDs of commits which preceded this commit
  message: string // a description of the commit
  timestamp: DateTime // local clock time of commit
  diff: {
    add: [[path: string, hash: string], ...],
    change: [[path: string, hash: string], ...],
    del: [path: string, ...]
  ]
}
/blobs/{hash} = {
  writer: Buffer // key of the input core which contains this blob
  bytes: number // number of bytes in this blob
  start: number // starting seq number
  end: number // ending seq number
}
```

The oplogs include one of the following message types:

```
Commit {
  op: 1
  id: string, // random generated ID
  parents: string[] // IDs of commits which preceded this commit
  message: string // a description of the commit
  timestamp: DateTime // local clock time of commit
  diff: {
    add: [[path: string, hash: string], ...],
    change: [[path: string, hash: string], ...],
    del: [path: string, ...]
  ]
}
Blob {
  op: 2
  hash: string // hash of this blob
  bytes: number // number of bytes in this blob
  length: number // number of chunks in this blob (which will follow this op)
}
BlobChunk {
  op: 3
  value: Buffer // content
}
```

### Managing writers

Only the creator of the Repo maintains the Hyperbee index as a hypercore. The owner updates the `/_meta` entry to determine the current writers.

This is a temporary design until Autoboot lands.
