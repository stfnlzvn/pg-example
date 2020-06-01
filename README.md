# pg-example


server
```javascript

repl.init({
	dbObj: db,        //  pg-promise config
	pgp,
	ioObj: io,        //  socket io
  
  /* Server db handle */
  /* dbo contains all database tables and views */
  isReady: (dbo) => {
    dbo.tasks.find(
      { user_id: 234, title: { $ilike: '%coding%' } },
      { orderBy: { "last_updated": false } }
    ).then(tasks => ... );
  },
  
  /* Client authentication */
  onSocketConnect: async ({ socket, dbo }) => {
    const { username, password } = socket.handshake;
    const user = await dbo.users.find({ username, password });
    socket._user = user;
    return user;
  },
  onSocketDisconnect: ({ socket, dbo }) => { },
  
  /* Client request validation */
	publish: ({ socket, dbo }) => {
		const { _user } = socket;

    /* Must return a table/view name, then the allowed actions then action params/rules. 
        Tip: Use true or "*" to allow everything 
    */
		return {
      tasks: {
        insert: _user.admin || {
        
          // Fields allowed to be inserted.   Tip: Use false to exclude field
          fields: { user_id: false, id: false },
          
          // Data to include/overwrite on each insert
          forcedData: { user_id: _user.id }         
        },
        select: _user.admin || {
          fields: "name, description, task_duration, last_updated",
          forcedFilter: { user_id: _user.id },          // Filter to include/overwrite
          filterFields: "natural_id, last_updated"      // Fields allowed to be searched
        },
        update: _user.admin || {
          fields: "user_typing, user_connected, synced",
          forcedFilter: { user_id: _user.id },
          filterFields: "natural_id, last_updated"
        },
        delete: _user.admin,
        sync: {
          id_fields: "natural_id",
          synced_field: "last_updated"
        }
      },
      some_public_table: "*"
    }
 });
```


client
```javascript

const socket = io({ username: "usr", password: "pwd" });

psqlWS.init({ 
    socket, 
    isReady: (db) => {
      db.tasks.find({}).then(tasks => ... // Only tasks for the user_id);
      db.tasks.subscribe(
      	{ last_updated: { $gt: Date.now() - 24 * 3600 * 1000  },
	{ limit: 10 },
	tasks => {
      		...
      	}
      )
      
      
    },
    onDisconnect: (err, res) => {
        // location.reload();
    }
});
```
