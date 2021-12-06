---
title: "JS SDK"
layout: docs
category: "JS SDK"
order: 3
---

## JS SDK

Through Yorkie JS SDK, you can efficiently building collaborative applications. On the client-side implementation, you can create documents that are automatically synced with remote peers with minimal effort.

### Client

Client is a normal client that can communicate with the agent. It has documents and sends changes of the document in local to the agent to synchronize with other replicas in remote.

#### Creating a client

With `yorkie.createClient()` you can create a client. After the client is activated, it is connected to the agent and ready to use.

```javascript
const client = yorkie.createClient('AGENT_RPC_ADDR');
await client.activate();
```

#### Subscribing document events

We can use `client.subscribe` to subscribe client-based events, such as `status-changed`, `stream-connection-status-changed` and `peer-changed`. 

```javascript
const unsubscribe = client.subscribe((event) => {
  if (event.name === 'status-changed') {
    console.log(event.value); // 'activated' or 'deactivated'
  } else if (event.name === 'stream-connection-status-changed') {
    console.log(event.value); // 'connected' or 'disconnected'
  }
});
```

By using the value of the `stream-connection-status-changed` event, it is possible to determine whether the client is connected to the network.

#### Peer Awareness

When creating a client, we can pass information of the client to other peers attaching the same document with metadata.

```javascript
const client = yorkie.createClient('AGENT_RPC_ADDR', {
  metadata: {
    username: name,
  },
});
```

When a new peer registers or leaves, `peers-changed` event is fired, and the other peer's clientID and metadata can be obtained from the event.

```javascript
const unsubscribe = client.subscribe((event) => {
  if (event.name === 'peers-changed') {
    const peers = event.value[doc.getKey().toIDString()];
    for (const [clientID, metadata] of Object.entries(peers)) {
      console.log(clientID, metadata);
    }
  }
});
```

### Document

Document is primary data type in Yorkie, providing a JSON-like updating experience that makes it easy to represent your application's model. Document can be updated without attaching it to the client, and changes are automatically propagated to other peers when attaching it to the client or when the network is restored.

#### Editing the document

`Document.update(changeFn, message)` enables you to modify a document. The optional `message` allows you to keep a string to the change. If the document is attached to the client, all changes are automatically synchronized with other clients.

```javascript
doc.update((root) => {
  root.obj = {};              // {"obj":{}}
  root.obj.num = 1;           // {"obj":{"num":1}}
  root.obj.obj = {'str':'a'}; // {"obj":{"num":1,"obj":{"str":"a"}}}
  root.obj.arr = ['1', '2'];  // {"obj":{"num":1,"obj":{"str":"a"},"arr":[1,2]}}
}, 'update document for test');
```

Under the hood, `root` in the `update` function creates a `change`, a set of operations, using a JavaScript proxy. And Every element has a unique ID, created by the logical clock. This ID is used by Yorkie to track which object is which.

And you can get the contents of the document using `document.getRoot()`.

```javascript
const root = doc.getRoot();
console.log(root.obj);     // {"num":1,"obj":{"str":"a"},"arr":[1,2]}
console.log(root.obj.num); // 1
console.log(root.obj.obj); // {"str":"a"}
console.log(root.obj.arr); // [1,2]
```

#### Subscribing document events

A document is modified by changes generated remotely or locally in Yorkie. When the document is modified, change events occur and we can subscribe to it using `document.subscribe`. Here, we can do post-processing such as repaint in the application using the `path` of the change events.

```javascript
doc.subscribe((event) => {
  if (event.type === 'local-change') {
    console.log(event);
  } else if (event.type === 'remote-change') {
    for (const changeInfo of event.value) {
      // `message` delivered when calling document.update
      console.log(changeInfo.change.message);
      for (const path of changeInfo.paths) {
        if (path.startsWith('$.obj.num') {
          // root.obj.num is changed
        } else if (path.startsWith('$.obj')) {
          // root.obj is changed
        }
      }
    }
  }
});
```

### Custom CRDT types

#### Text

Text provides support for collaborative plain text editing. Under the hood, text is represented as a list of characters. Compared to using a regular JavaScript array, Text offers better performance. And it also has selection information for sharing the cursor position.

```javascript
doc.update((root) => {
  const text = root.createText('text');      // {"text":""}
  text.edit(0, 0, 'hello');                  // {"text":"hello"}
  text.edit(0, 1, 'H');                      // {"text":"Hello"}
  text.select(0, 1);                         // {"text":"^H^ello"}
});
```

An example of text co-editing with CodeMirror:

[CodeMirror example](https://github.com/yorkie-team/yorkie-js-sdk/blob/main/examples/index.html)

#### Counter
Counter support numeric types that change to addition and subtraction. If numeric data needs to be modified at the same time, Counter should be used instead of Primitive.

```javascript
doc.update((root) => {
  const counter = root.createCounter('counter', 1);     // {"counter":1}
  counter.increase(2);                                  // {"counter":3}
  counter.increase(3.5);                                // {"counter":6.5}
  counter.increase(-3.5);                               // {"counter":3}
});
```

### Reference

[JS SDK Reference](https://yorkie.dev/yorkie-js-sdk)