# Handyman / quality of life module

Handyman module allow you to:

- pass shorthands
- validate & secure plans
- pass schemas as namespace parameter
- pass schemas as plans
- pull sub schemas by $id
- calculate fields

## With module:

```js
// Without this plugin this syntax will throw an error
const opts = { namespace: { MyNamespaceSchema: new Schema(['winter', 'spring'])}};
const schema = new Schema({
  test: 'string',
  test2: { $id: 'SubSchema', test: 'string' },
  test3: new Schema('?string');
  test4: 'MyNamespaceSchema'
}, opts);

const subschema = schema.pull('SubSchema'); // Schema
const MyNamespaceSchema = schema.pull('MyNamespaceSchema'); // Schema
```

## Without module:

```js
const MyNamespaceSchema = new Schema({ $type: 'enum', enum: ['winter', 'spring'] });
const SubSchema = new Schema({ $type: 'object', properties: { test: { $type: 'string' } } });
const schema = new Schema({
  $type: 'object',
  properties: {
    test: { $type: 'string' },
    test2: { $type: 'schema', schema: SubSchema },
    test3: { $type: 'schema', schema: new Schema({ $type: 'string' }), $required: false };
    test4: { $type: 'schema', schema: MyNamespaceSchema }
  },
});

```

## Possible shorthands:

- string shorthand: <code>'string' | 'MySchema'</code> for prototype and namespace schemas
  shorthands
- object shorthand: <code>{ fieldA: 'string', fieldB: 'string' }</code>
- tuple shorthand: <code>['string', 'number']</code>
- enum shorthand: <code>['winter', 'spring']</code>
- schema shorthand: <code>new Schema('?string')</code>

### Example:

```js
const schema = new Schema(
  {
    a: 'string', //? scalar shorthand
    b: '?string', //? optional shorthand
    c: ['string', 'string'], //? tuple
    d: new Schema('?string'), //? Schema shorthand
    e: ['winter', 'spring'], //? Enum shorthand
    f: { a: 'number', b: 'string' }, //? Object shorthand
    g: { $type: 'array', items: 'string' }, //? Array items shorthand
    h: 'MyExternalSchema', //? Namespace schema shorthand
  },
  { namespace: { MyExternalSchema: new Schema('string') } },
);
```

> String shorthand is analog to <code>{ $type: type, required: type.includes('?') } </code>

## Calculated fields

Calculated fields supposed to do preprocessing of your schema;

> Warning: **experimental**. We do not support some types yet: Map, Set

### Example

```js
const schema = {
  $id: 'user',
  name: 'string',
  phrase: (sample, schema, root) => 'Hello ' + schema.name + ' !',
};
const sample = { name: 'Alexander' };
new Schema(schema).calc(sample); // { ..., name: 'Alexander', phrase: 'Hello Alexander !'};
schema; // { $id: 'user', name: 'Alexander', phrase: 'Hello Alexander !'};
```

### Writing calculated fields

Calculated fields is a function that receives two arguments:

- root: root object <code>{ input: Sample }</code>
- parent: assigned target object

> Warning: your return value will be assigned to samples

### Additional

Method <code>schema.calc</code> receives mode as second parameter; This method allow to specify
return value as:

- Schema.calc(sample, true); // Returns copy of sample with assigned values
- Schema.calc(sample); // Returns sample object with assigned values
