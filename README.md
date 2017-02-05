DiamondDB
=========
DiamondDB is a zero dependency modular database system for Node.js. It consists of three parts: the API, the cache, and the persistence layer. The API (which I usually just call the "database") is the controller of the system: the client interacts directly with this module which in turn communicates with the cache and the persistence layer via a standard set of messages. This makes it easy to switch out the cache or persistence layer with a module of your own. The cache and persistence modules can interface with any other library so long as they implement the message interface.

Message Interface
----------------
The cache and persistence layer have a message method that recieves a message and does some action based on the message. The return value is important: some messages expect a resolves promise, a payload, or nothing. Below is an overview of the message interface expected for the cache and persistence module.

**CACHE**
```javascript
class Cache {
  /* the cache module must implement a method called message to receive messages */
  message(message){
    switch(message.operation){
      case STORE_RECORD:
        /* the cache should have a method for storing data */
	    this.storeRecord(message.data)
	    /* the STORE_RECORD message simply expects an empty success message */
	    return Promise.resolve(success())
      case FETCH_RECORD:
        /* the cache should have a method for fetching data */
	    const result = this.fetchRecord(message.data)
	    /* it should return a success message with the result, even if the result is null */
	    return Promise.resolve(success(result))
    }
  }
}
```

**STORE**
```javascript
class Store {
  message(message){
    switch(message.operation){
      case UPDATE_META:
	  /* the database sends the tables meta-data to the store to be persisted
	   * since all tables are sent with the message, it might be wise to only store
	   * the most recent payload and persist on some interval
	   * this message does not expect a response
	   */
        return Promise.resolve()
      case STORE_RECORD:
	    /* this message should either persist a record or store it for later batch persistence */
        return Promise.resolve()
      case FETCH_RECORD:
        /* this message is sent if a record is not in the cache, it should fetch from the disk */
        return this.fetch(message.data)
      case MAKE_TABLE:
        /* persists a table and expects a success or fail message */
        return this.makeTable(message.data)
      case INITIALIZE_PERSISTANCE:
        /* a store should be able to fetch the tables meta-data */
        const tables = this.initialize()
	    /* tables should be returned in a success message */
        return Promise.resolve(success(tables))
      case PERSIST_ALL:
        /* this is message is sent on intervals if intermittent batch-persistence is configured */
	    const result = this.persist()
	    /* the result should be either a success or failure message */
	    return Promise.resolve(result)
    }
  }
}
```
