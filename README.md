# Form with conditionals

This project extends [react-jsonschema-form](https://github.com/mozilla-services/react-jsonschema-form) with 
conditional logic, which allow to have more complicated logic expressed and controlled with JSON schema.
This is primarily useful for complicated schemas with extended business logic,
which are suspect to changes and need to be manageable and changeable without modifying running application.

If you need simple rule logic, that does not change a lot, you can use original [mozilla project](https://github.com/mozilla-services/react-jsonschema-form),
by following examples like https://jsfiddle.net/69z2wepo/68259/

The project is done to be fully compatible with mozilla, 
without imposing additional limitations.

## Features

- Support for [Json Rules Engine](https://github.com/CacheControl/json-rules-engine) and [json-rules-engine-simplified](https://github.com/RxNT/json-rules-engine-simplified) 
- Extensible action mechanism
- Configuration over coding
- Lightweight and extensible

## Installation

Install `react-jsonschema-form-conditionals` by running:

```bash
npm install --s react-jsonschema-form-conditionals
```

## Usage

The simplest example of using `react-jsonschema-form-conditionals`

```jsx
import applyRules from 'react-jsonschema-form-conditionals';
import Engine from 'react-jsonschema-form-conditionals/engine/SimplifiedRuleEngineFactory';
import Form from "react-jsonschema-form";
let FormWithConditionals = applyRules(Form);

...

const rules = [{
    ...
}];

let FormWithConditionals = applyRules(Form);

ReactDOM.render(
  <FormWithConditionals
        rules = {rules}
        rulesEngine={Engine}
        schema = {schema}
        ...
  />,
  document.querySelector('#app')
);
```

To show case uses for this library we'll be using simple registration schema example 

```jsx

import applyRules from 'react-jsonschema-form-conditionals';
import Form from "react-jsonschema-form";

let schema = {
  definitions: {
    hobby: {
      type: "object",
      properties: {
        name: { type: "string" },
        durationInMonth: { "type": "integer" },
      }
    }
  },
  title: "A registration form",
  description: "A simple form example.",
  type: "object",
  required: [
    "firstName",
    "lastName"
  ],
  properties: {
    firstName: {
      type: "string",
      title: "First name"
    },
    lastName: {
      type: "string",
      title: "Last name"
    },
    age: {
      type: "integer",
      title: "Age",
    },
    bio: {
      type: "string",
      title: "Bio",
    },
    country: {
      type: "string",
      title: "Country" 
    },
    state: {
      type: "string",
      title: "State" 
    },
    zip: {
      type: "string",
      title: "ZIP" 
    },
    password: {
      type: "string",
      title: "Password",
      minLength: 3
    },
    telephone: {
      type: "string",
      title: "Telephone",
      minLength: 10
    },
    hobbies: {
        type: "array",
        items: { "$ref": "#/definitions/hobby" }
    }
  }
}

let rules = [{
    ...
}]

let FormWithConditionals = applyRules(Form);

render((
  <FormWithConditionals
    schema={schema}
    rules={rules}
  />
), document.getElementById("app"));
```

Conditionals functionality is build using 2 things
- Rules engine ([Json Rules Engine](https://github.com/CacheControl/json-rules-engine) or [Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified))
- Schema action mechanism 

Rules engine responsibility is to trigger events, action mechanism 
performs needed actions on the requests.

## Rules engine

Project supports 2 rules engines out of the box:
- [Json Rules Engine](https://github.com/CacheControl/json-rules-engine) 
- [Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified)

### Enabling rules engine
 
In order to use one or another you need to specify `EngineFactory` to use,
which provides additional optimizations above the original rules engine.

### [Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified) factory

To use [Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified), you need to specify `SimplifiedRulesEngineFactory` 
as a `rulesEngine` in `FormWithConditionals`

For example:
```jsx

import applyRules from 'react-jsonschema-form-conditionals';
import SimplifiedRulesEngineFactory from 'react-jsonschema-form-conditionals/engine/SimplifiedRuleEngineFactory';
import Form from "react-jsonschema-form";
let FormWithConditionals = applyRules(Form);

...

let FormWithConditionals = applyRules(Form);

ReactDOM.render(
  <FormWithConditionals
        rules = {rules}
        rulesEngine={SimplifiedRulesEngineFactory}
        schema = {schema}
        ...
  />,
  document.querySelector('#app')
);
```

That is it, now rules are expected to be in accordance with [Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified)

### Cache Control [Json Rules Engine](https://github.com/CacheControl/json-rules-engine) 

To use [Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified), you need to specify `CacheControlRulesEngineFactory` 
as a `rulesEngine` in `FormWithConditionals`

For example: 
```js

import applyRules from 'react-jsonschema-form-conditionals';
import CacheControlRulesEngineFactory from 'react-jsonschema-form-conditionals/engine/CacheControlRulesEngineFactory';
import Form from "react-jsonschema-form";

// ...

let FormWithConditionals = applyRules(Form);

ReactDOM.render(
  <FormWithConditionals
        rulesEngine={CacheControlRulesEngineFactory}
        // ...
  />,
  document.querySelector('#app')
);
```

### Extending rules engine

If non of the provided engines satisfies, your needs, you can 
implement your own `EngineFactory` and `Engine` which should 
comply to following:

```js
class EngingFactory {
  getEngine(rules, schema) {
    return Engine;
  }
}

class Engine {
  run() {
    return Promise[Event];
  }
}
```

Original `rules` and `schema` is used as a parameter for a factory call, 
in order to be able to have additional functionality, such as rules to schema compliance validation, 
like it's done in Simplified Json Rules Engine](https://github.com/RxNT/json-rules-engine-simplified)

## Schema action mechanism

Rules engine emits events, which are expected to have a `type` and `params` field,
`type` is used to distinguish action that is needed, `params` are used as input for that action:

```json
{
  "type": "remove",
  "params": {
    "field": "name"
  }
}
```

By default action mechanism defines a supported set of rules, which you can extend as needed:

- `remove` removes a field or set of fields from `schema` and `uiSchema`
- `require` makes a field or set of fields required
- `replaceUi` replaces a field or set of fields `uiSchema` entrance

### Remove action

If you want to remove a field, your configuration should look like this:

```json
    {
      "conditions": { },
      "event": {
        "type": "remove",
        "params": {
          "field": "password"
        }
      }
    }
```
When `condition` is meet, `password` will be removed from both `schema` and `uiSchema`.
 
In case you want to remove multiple fields `name`, `password`, rule should look like this:

```json
    {
      "conditions": { },
      "event": {
        "type": "remove",
        "params": {
          "field": [ "name", "password" ]
        }
      }
    }
```
### Require action

The same convention goes for `require` action

For a single field:

```json
    {
      "conditions": { },
      "event": {
        "type": "require",
        "params": {
          "field": "password"
        }
      }
    }
```

For multiple fields:

```json
    {
      "conditions": { },
      "event": {
        "type": "require",
        "params": {
          "field": [ "name", "password"]
        }
      }
    }
```

### Replace UI action

The same convention goes for `require` action

For a single field:

```json
    {
      "conditions": { },
      "event": {
        "type": "replaceUi",
        "params": {
          "field": "password",
          "conf": {
            "classNames": "col-md-12"
          }
        }
      }
    }
```

After this event is triggered, `uiSchema` for the `password`, will be replaced with `conf` content, so it will become `col-md-12`.

For multiple fields:

```json
    {
      "conditions": { },
      "event": {
        "type": "require",
        "params": {
          "field": [ "name", "password"],
          "conf": {
            "classNames": "col-md-12"
          }
        }
      }
    }
```
After this event is triggered, `uiSchema` for both `password` & `name`, will be replaced with `conf` content.

### Extension mechanism

You can extend existing actions list, by specifying `extraActions` on the form.

Let's say we need to introduce `replaceClassNames` action, that 
would just specify `classNames` `col-md-4` for all fields except for `ignore`d one.
We also want to trigger it only when `password` is `empty`.

This is how we can do this:

```jsx
import applyRules from 'react-jsonschema-form-conditionals';
import Engine from 'react-jsonschema-form-conditionals/engine/SimplifiedRuleEngineFactory';
import Form from "react-jsonschema-form";
let FormWithConditionals = applyRules(Form);

...

const rules = [
    {
        conditons: {
            password: "empty"
        },
        event: {
            type: "replaceClassNames",
            params: {
                classNames: "col-md-4",
                ignore: [ "password" ]
            }
        }
    }
];

let FormWithConditionals = applyRules(Form);

let extraActions = {
    replaceClassNames: function(params, schema, uiSchema) {
        Object.keys(schema.properties).forEach((field) => {
            if (uiSchema[field] === undefined) {
                uiSchema[field] = {}
            }
            uiSchema[field].classNames = params.classNames;
        }
    }
};

ReactDOM.render(
  <FormWithConditionals
        rules = {rules}
        rulesEngine={Engine}
        schema = {schema}
        extraActions = {extraActions}
        ...
  />,
  document.querySelector('#app')
);
```

Provided snippet does just that.

## Action validation mechanism

All default actions are validated by default, checking that field exists in the schema, to save you some headaches. 
There are 2 levels of validation

- `propTypes` validation, using FB `prop-types` package
- explicit validation

You can define those validations in your actions as well, to improve actions usability.

All validation is disabled in production.
 
### Prop types action validation
 
This is reuse of familiar `prop-types` validation used with React components, and it's used in the same way:

In case of `require` it can look like this:
```js
require.propTypes = {
  field: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.arrayOf(PropTypes.string),
  ]).isRequired,
};
```

The rest is magic.

WARNING, the default behavior of `prop-types` is to send errors to console,
which you need to have running in order to see them.

For our `replaceClassNames` action, it can look like this:

```js
replaceClassNames.propTypes = {
  classNames: PropTypes.string.isRequired,
  ignore: PropTypes.arrayOf(PropTypes.string)
};
```

## Explicit validation

In order to provide more granular validation, you can specify validate function on 
your action, that will receive `params` and `schema` and can provide appropriate validation.

For example, validation for `require` can be done like this:

```js
  require.validate = function({ field }, schema) {
    if (Array.isArray(field)) {
      field
        .filter(f => schema && schema.properties && schema.properties[f] === undefined)
        .forEach(f => console.error(`Field  "${f}" is missing from schema on "require"`));
    } else if (
      schema &&
      schema.properties &&
      schema.properties[field] === undefined
    ) {
      console.error(`Field  "${field}" is missing from schema on "require"`);
    }
  };
```

Validation is not mandatory, and will be done only if field is provided.

For our `replaceClassNames` action, it would look similar: 
```js
  replaceClassNames.validate = function({ ignore }, schema) {
    if (Array.isArray(field)) {
      ignore
        .filter(f => schema && schema.properties && schema.properties[f] === undefined)
        .forEach(f => console.error(`Field  "${f}" is missing from schema on "replaceClassNames"`));
    } else if (
      schema &&
      schema.properties &&
      schema.properties[ignore] === undefined
    ) {
      console.error(`Field  "${ignore}" is missing from schema on "replaceClassNames"`);
    }
  };
```

## Contribute

- Issue Tracker: github.com/RxNT/form-with-rules/issues
- Source Code: github.com/RxNT/form-with-rules

## Support

If you are having issues, please let us know.
We have a mailing list located at: ...

## License

The project is licensed under the ... license.
