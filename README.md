# Redis output for flogging.js

The `options` Object is sent directly to [createClient](https://github.com/NodeRedis/node_redis#rediscreateclient).


## Installing

	npm i flogging_redis --save


## Usage

	flogging = require('flogging')

	flogging_redis = require('flogging_redis')

	redis_stream = flogging_redis.with_set({
		host: '127.0.0.1',
		port: 6379
	})

	log = flogging.start_console_stream(redis_stream)

	log.info('Straight into Redis', 12)

	log.stop()


## License

[MIT](./LICENSE)
