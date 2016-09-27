# Test main module

## Imports

### Library imports

	end_of_stream = require 'end-of-stream'

	flogging_base = require 'flogging.stream_base'

	pipe = require 'ramped.pipe'

	redis = require 'redis'

	tape = require 'tape'

	through2 = require 'through2'


### Relative imports

	flogging_redis = require './main'


## Helper functions

	delete_error = (client, t)->
		(err)->
			console.error err

			t.fail 'Error deleting key'

	delete_key = (client, key)->
		new Promise (resolve, reject)->
			client.del key, (err, result)->
				if err
					reject err

				else
					resolve result


## Run test

	tape 'Send only label and value with SET', (t)->
		t.plan 1

		desired_output =
			id: 'Flogging_Redis_test_message'
			label: 'Flogging Redis test message'
			value: 12

		logger = flogging_base.start()

		redis_connection =
			host: '127.0.0.1'
			port: 6379

		redis_stream = flogging_redis.with_set redis_connection

		flogging_base.pipe_to_stream logger, redis_stream

		log = flogging_base.make_note pipe [
			(message)->
				message.id = message.label.replace /\s/g, '_'

				message

			flogging_base.send logger
		]

		client = redis.createClient redis_connection

		end_of_stream logger, (err)->
		# logger.on 'end', ->
			console.log 'Logger got end'

			# if err
			# 	console.error err

			redis_stream.push null

NOTE: The `data` listener seems to be required for the Stream end event to fire.

		redis_stream.on 'data', ->
		# 	console.log arguments

		end_of_stream redis_stream, (err)->
		# redis_stream.on 'end', ->
			console.log 'Redis Stream got end'

			# if err
			# 	console.error 'Got error'
			# 	console.error err

			client.get desired_output.id, (err, result)->
				if err
					t.fail err

				else
					actual_output = JSON.parse result

					delete_key client, desired_output.id
					.catch delete_error client, t
					.then (result)->
						client.quit()

						t.deepEqual actual_output, desired_output

		delete_key client, desired_output.id
		.catch delete_error client, t
		.then (result)->
			console.log 'Deleted key'

			log desired_output.label, desired_output.value

			setTimeout ->
				console.log 'ending'

				flogging_base.stop logger
			, 500
