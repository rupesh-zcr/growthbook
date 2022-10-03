# GrowthBook Back-End

This document is meant for developers who want to contribute to the GrowthBook platform. It covers the following topics:

- Editing data models

## Editing data models

In order to preform CRUD operations on the data model there are 3 places you will need to contribute to and a potential 4th.  
`models/exampleModel.ts`, `types/example.d.ts`, `controllers/example.ts`, and optionally if you are gathering data from the
front-end `front-end/pages/example/index.tsx`.

### `models/exampleModel.ts`

defines the model for mongoose ORM. Complex datatypes are NOT allowed.

```tsx
const exampleSchema = new mongoose.Schema({
  foo: String,
  bar: {
    x: String,
    y: Boolean,
    z: Date,
    w: Number,
  },
});
```

### `types/example.d.ts`

defines the type that can be used internally. Complex datatypes ARE allowed.

```tsx
interface BarI {
  x: String;
  y: Boolean;
  z: Date;
  w: Number;
}

const exampleSchema = new mongoose.Schema({
  foo: String,
  bar: BarI,
});
```

### `controllers/example.ts`

contains the definitions for various CRUD operations. Reference existing controllers for example code. If you need to add
a new endpoint add it to `backend/src/app.ts`.

### `front-end/pages/example/index.tsx`

if data is changing via the front-end you will need to make sure and provide the change wherever the API request is being
made via the `front-end`


## Sample Data

To use the analysis parts of GrowthBook, you need to connect to a data source with experimentation events.

We have a sample data generator script you can use to seed a Postgres database with realistic website traffic:

1. Run `yarn workspace back-end generate-dummy-data`.  This will create CSV files in `/tmp/csv`
2. Start Postgres locally
3. Connect to your local Postgres instance using `psql`
4. Run the SQL commands in `packages/back-end/test/data-generator/create.sql` to create the tables and upload the generated data

Next, you'll need to set up the Postgres connection within GrowthBook.

1. Under Analysis->Data Sources, add a new data source
2. Select "Custom Event Source"
3. Select Postgres and enter your connection info. If you are running with `yarn dev`, you can use `localhost` safely and ignore the warning in the UI about docker.
4. On the data source page, you can add two identifier types: `user_id` and `anonymous_id`
5. Add an Identifier Join table:
  ```sql
  SELECT
    userId as user_id,
    anonymousId as anonymous_id
  FROM
    experiment_viewed
  ```
6. Then, define an assignment query for logged-in users:
   - Identifier type: `user_id`
   - SQL:
    ```sql
    SELECT
      userId as user_id,
      timestamp as timestamp,
      experimentId as experiment_id,
      variationId as variation_id,
      browser,
      country
    FROM
      experiment_viewed
    ```
   - Dimension columns: `browser`, `country`
7. And another assignment query for anonymous visitors:
   - Identifier type: `anonymous_id`
   - SQL:
    ```sql
    SELECT
      anonymousId as anonymous_id,
      timestamp as timestamp,
      experimentId as experiment_id,
      variationId as variation_id,
      browser,
      country
    FROM
      experiment_viewed
    ```
   - Dimension columns: `browser`, `country`
8. Create a metric:
   - Name: `Purchased`
   - Type: `binomial`
   - Identifier Types: Both `user_id` and `anonymous_id`
   - SQL:
    ```sql
    SELECT
      userId as user_id,
      anonymousId as anonymous_id,
      timestamp as timestamp
    FROM
      orders
    ```
9. Go to add an experiment. You should see a few that are ready to be imported from the data source.

There are a lot more metrics you can define with the sample data besides just "Purchased". Here are a few ideas:
- Revenue per User
- Average Order Value
- Page Views per User
- Retention Rate
- Session Duration
- Pages per Session
- Sessions with Searches