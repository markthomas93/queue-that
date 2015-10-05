Queue That
----------

A queue managed in localStorage for async tasks that may run immediately before page unload.
Queue That is built primarily to queue XHR requests for reporting.

### API

```javascript
var queueThat = require('queue-that')
var post = require('post')
var when = require('when')

/**
 * The queue will not process any more events
 * until done is called.
 */
var q = queueThat({
  process: function (items, done) {
    when.all(
      items.map(post.bind(null, 'https://somewhere.com/events'))
    ).then(done)
  }
})

/**
 * This will usually fail because the page
 * will unload and cancel the XHR.
 */
post('https://somewhere.com/events', {
  name: 'Ronald',
  description: 'Clicked on a link'
})

/**
 * By queueing the event in localStorage, if the
 * window is closed, the event can be processed
 * later.
 */
q({
  name: 'Ronald',
  event: 'Clicked on a link'
})
```

### Features

- When more than one tab is open, only one active queue will process tasks.
- If `options.process` calls back with an error, the tasks will be passed to `options.process`
  again after a second. This backoff time doubles for each sequential error. The backoff timer is
  stored in localStorage and so will not reset when Queue That is reinitialised.
- Tasks are batched with a default batch size of **20**. The option `batchSize` can be used to change
  the batch size. Disable batching by setting `batchSize` to `Infinity`.
- The active queue polls localStorage every **100ms** for new tasks.
- If the active queue does not poll for over **5 seconds**, the next tab to queue
  a process will become the active queue.

### Support

Supported and tested on

- IE8+
- Firefox
- Safari
- Opera
- Chrome