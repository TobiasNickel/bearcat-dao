## Bearcat-dao -- an O/R mapping dao framework  
Bearcat-dao is an O/R mapping dao framework which provides O/R mapping, dao support for [node.js](http://nodejs.org/).  
It is written by [POJOs](https://github.com/bearcatnode/bearcat/wiki/POJOs-based-development) and can be nicely used with [Bearcat](https://github.com/bearcatnode/bearcat).  

## Features  
* O/R mapping  
* cacheTemplate  
* sqlTemplate  
* transaction  

## Usage
### Domain Definition
Domain is a POJO, express the relationship between table and object  
```
var simpleDomain = function() {
	this.id = 0;
	this.name = null;
}

module.exports = {
	func: Domain,
	primary: [{
		name: "id",
		type: "Long"
	}],
	fields: ["name"],
	tableName: "test"
}
```

more in details:  
* func : constructor function for the domain object
* primary : an array defines primary fields  
* fields : an array defines fields except primary fields  
  - field can be defined by object with ***name***, ***type*** properties, or simply by ***name*** string  
* tableName : the name of table to be mapped by the ORM object  
* key : the cached key for conjunctive query domain definition  

### Add to project
```
npm install bearcat-dao --save
```  

modify context.json used by your project  
[placeholds](https://github.com/bearcatnode/bearcat/wiki/Consistent-configuration) can be nicely used to switch between contexts  

```
"dependencies": {
	"bearcat-dao": "*"
},
"beans": [{
		"id": "mysqlConnectionManager",
		"func": "node_modules.bearcat-dao.lib.connection.sql.mysqlConnectionManager",
		"props": [{
			"name": "port",
			"value": "${mysql.port}"
		}, {
			"name": "host",
			"value": "${mysql.host}"
		}, {
			"name": "user",
			"value": "${mysql.user}"
		}, {
			"name": "password",
			"value": "${mysql.password}"
		}, {
			"name": "database",
			"value": "${mysql.database}"
		}]
	}, {
		"id": "redisConnectionManager",
		"func": "node_modules.bearcat-dao.lib.connection.cache.redisConnectionManager",
		"props": [{
			"name": "port",
			"value": "${redis.port}"
		}, {
			"name": "host",
			"value": "${redis.host}"
		}]
	}]
```

if you do not use redis, you can remove ***redisConnectionManager*** definition  

### Write daos  
Bearcat-dao provides ***domainDaoSupport*** wrapping basic sql and cache operations  
add it with properties dependency injection, and init it by invoking ***initConfig*** method  
then you can use domainDaoSupport convenient methods to wrap your own daos  

simpleDao.js
```
var SimpleDomain = require('simpleDomain');
var SimpleDao = function() {
	this.domainDaoSupport = null;
}

SimpleDao.prototype.init = function() {
	// init with SimpleDomain to set up O/R mapping
	this.domainDaoSupport.initConfig(SimpleDomain);
}

// query list all
// callback return mapped SimpleDomain array results
SimpleDao.prototype.getList = function(cb) {
	var sql = ' 1 = 1';
	return this.domainDaoSupport.getListByWhere(sql, null, null, cb);
}

module.exports = {
	id: "simpleDao",
	func: SimpleDao,
	props: [{
		name: "domainDaoSupport",
		ref: "domainDaoSupport"
	}],
	"init": "init"
}
```

api reference for [domainDaoSupport](http://bearcatnode.github.io/bearcat-dao/domainDaoSupport.js.html)  

## Transaction
Bearcat-dao provides transaction support based on [Bearcat AOP](https://github.com/bearcatnode/bearcat/wiki/Aspect-Object-Programming). The aspect is [transactionAspect](https://github.com/bearcatnode/bearcat-dao/blob/master/lib/aspect/transactionAspect.js) which provides around advice, when target transaction method calls cb function with ***err***, rollback will be emited, otherwise it will commit the operations.  
The pointcut defined is:  
```
"pointcut": "around:.*?Transaction"
```  
Therefore, any POJO method match this pointcut can a transcation method  
Since transaction must be within the same connection, in Bearcat-dao it is ***transactionStatus***, daos under the transaction method must hold the same transactionStatus  
```
SimpleService.prototype.testMethodTransaction = function(cb, txStatus) {
	var self = this;
	this.simpleDao.transaction(txStatus).addPerson(['aaa'], function(err, results) {
		if (err) {
			return cb(err); // if err occur, rollback will be emited
		}
		self.simpleDao.transaction(txStatus).getList([1, 2], function(err, results) {
			if (err) { 
				return cb(err); // if err occur, rollback will be emited
			}
			cb(null, results); // commit the operations
		});
	});
}
```

## Multi tables query
When doing querys, by default the mapping domain is what you pass into [domainDaoSupport.initConfig](http://bearcatnode.github.io/bearcat-dao/domainDaoSupport.js.html#initConfig) method  
In [domainDaoSupport.getList](http://bearcatnode.github.io/bearcat-dao/domainDaoSupport.js.html#getList) and [domainDaoSupport.getListByWhere](http://bearcatnode.github.io/bearcat-dao/domainDaoSupport.js.html#getListByWhere) method, you can pass mutli table specified domain to options to support O/R mapping when doing multi tables query.  
This domain is almost the same as [init domain](https://github.com/bearcatnode/bearcat-dao#domain-definition), except for the ***key*** specified as the cache key for this domain, and without needing to specify the ***tableName***

## License

(The MIT License)

Copyright (c) fantasyni and other contributors

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.