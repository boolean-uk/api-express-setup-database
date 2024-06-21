## Express Setup with a Postgres Database
To set up a new express application, follow the [earlier guide](https://github.com/boolean-uk/api-express-setup)

When you have an API project running, now you can introduce your database!

## Choosing a DB

We'll focus only on Postgres for this guide. You have a few choices:

- Install [PostgreSQL](https://www.postgresql.org/) locally on your machine
- Use a cloud provider, for example:
  - [Neon](https://neon.tech)
  - [Supabase](https://supabase.com/)

This guide will use Neon as an example.

## Required Packages

You'll need two more packages:

1. [dotenv](https://www.npmjs.com/package/dotenv)
2. [pg](https://node-postgres.com/)

`npm install dotenv pg`

## Environment Variables

Do not git commit sensitive information such as database credentials! You should use _environment variables_ to keep them secret.

1. In the root directory of the project, create a `.env` file
2. Add `.env` to your `.gitignore` file to ensure it doesn't get committed into git
3. Put the below `key=value`'s into your .env file:
```bash
PGHOST=''
PGDATABASE=''
PGUSER=''
PGPASSWORD=''
ENDPOINT_ID=''
```

The values to attach to each of these keys can be obtained from your database connection string. The way a connection string is structured follows this format:

`postgresql://PGUSER:PGPASSWORD@ENDPOINT_ID.PGHOST/PGDATABASE`

Note that the ENDPOINT_ID is a portion of the PGHOST. For example, a PGHOST `ep-hidden-sky-a2rjgvvq-pooler.eu-central-1.aws.neon.tech` contains the ENDPOINT_ID up to the first `.`: `ep-hidden-sky-a2rjgvvq-pooler`

**Example Connection String**
```
postgresql://my_username:superSecretPassword@ep-hidden-sky-a2rjgvvq-pooler.eu-central-1.aws.neon.tech/mydatabase?sslmode=require
```

**Resulting .env file**
```bash
PGHOST='ep-hidden-sky-a2rjgvvq-pooler.eu-central-1.aws.neon.tech'
PGDATABASE='mydatabase'
PGUSER='my_username'
PGPASSWORD='superSecretPassword'
ENDPOINT_ID='ep-hidden-sky-a2rjgvvq-pooler'
```

## Easier Approach

Neon lets you generate a `.env` file in their UI:

![](./db_env.png)

_Figure 1: Choose Node.js from the dropdown_
_Figure 2: Check Pooled connection_
_Figure 3: Select the `.env` tab_

## Creating A Database Connection In Code

Create a file somewhere in the project that will contain your database connection code. It's common to create a `utils` directory with a `dbConnection.js` file, or something similar.

```js
const { Pool } = require('pg');

const { PGHOST, PGDATABASE, PGUSER, PGPASSWORD } = process.env;

const dbConnection = new Pool({
  host: PGHOST,
  database: PGDATABASE,
  username: PGUSER,
  password: PGPASSWORD,
  port: 5432,
  ssl: {
    require: true,
  },
})

module.exports = dbConnection
```

## Using The Database Connection To Make Queries

You should put database query code in its own place in your app, commonly in a file referred to as a _repository_. For example, a `userRepository.js` file might look like this:

```js
const dbConnection = require('../utils/dbConnection.js')

const getAllUsers = async () => {
	const db = await dbConnection.connect()

	try {
		const sqlQuery = 'select * from users'
		const result = await db.query(sqlQuery)

		return result.rows
	} catch (e) {
		console.log(e)
	} finally {
		db.release()
	}
}
```

You can then use that function in a controller:

```js
const { Router } = require('express')
const { getAllUsers } = require('../repositories/userRepo.js')

const router = Router()

// localhost:300/users
router.get('/', async (req, res) => {
	const users = await getAllUsers()

	res.json({
		users
	})
})

module.exports = router
```