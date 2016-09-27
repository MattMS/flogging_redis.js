# Main module

This contain the main module exports for this package.


## Library imports

	redis = require 'redis'

	through2 = require 'through2'


## Make log message ID

	make_id = (chunk)->
		if chunk.id?
			return chunk.id

		process_id =
			if chunk.process_id?
				chunk.process_id
			else
				process.pid

		time =
			if chunk.time?
				chunk.time
			else
				process.uptime()
				# process.hrtime().join(',')

		"#{process_id}:#{time}"


## Exports

Use `options.prefix` to add a prefix to all keys.
This is one of the [Node Redis options](https://github.com/NodeRedis/node_redis#options-object-properties).


### HMSET

	module.exports.with_hmset = (options)->
		client = redis.createClient options

		transform = (chunk, encoding, callback)->
			id = make_id chunk

			client.hmset id, chunk, (err, response)->
				callback()

		flush = (callback)->
			client.quit()

			callback()

		through2.obj transform, flush


### SET

	module.exports.with_set = (options)->
		client = redis.createClient options

		transform = (chunk, encoding, callback)->
			id = make_id chunk

			json_text = JSON.stringify chunk

			client.set id, json_text, (err, response)->
				if err
					console.error err

				callback()

		flush = (callback)->
			client.quit()

			callback()

		through2.obj transform, flush
