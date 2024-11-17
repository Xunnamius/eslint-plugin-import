# import/order

🔧 This rule is automatically fixable by the [`--fix` CLI option](https://eslint.org/docs/latest/user-guide/command-line-interface#--fix).

<!-- end auto-generated rule header -->

Enforce a convention in the order of `require()` / `import` statements.

With the [`groups`](#groups-array) option set to `["builtin", "external", "internal", "parent", "sibling", "index", "object", "type"]` the order is as shown in the following example:

```ts
// 1. node "builtin" modules
import fs from 'fs';
import path from 'path';
// 2. "external" modules
import _ from 'lodash';
import chalk from 'chalk';
// 3. "internal" modules
// (if you have configured your path or webpack to handle your internal paths differently)
import foo from 'src/foo';
// 4. modules from a "parent" directory
import foo from '../foo';
import qux from '../../foo/qux';
// 5. "sibling" modules from the same or a sibling's directory
import bar from './bar';
import baz from './bar/baz';
// 6. "index" of the current directory
import main from './';
// 7. "object"-imports (only available in TypeScript)
import log = console.log;
// 8. "type" imports (only available in Flow and TypeScript)
import type { Foo } from 'foo';
```

In other words, if the [specifier](https://nodejs.org/api/esm.html#terminology)...

- Has no corresponding identifiers (e.g. `import './my/thing.js'`) and `warnOnUnassignedImports` is disabled, or is an unsupported use of `require()`, it will be ignored entirely since the order of these imports may be important for their side-effects
- Is an arcane TypeScript import declaration (e.g. `import log = console.log`), it will be considered **object**. However, note that external module references (e.g. `import x = require('z')`) are treated as normal `require()`s and import-exports (e.g. `export import w = y;`) are always ignored
- Is a type-only import, `"type"` is in `groups`, and `sortTypesAmongThemselves` is disabled, it will be considered **type** (with additional implications if using `pathGroups` and `"type"` is in `pathGroupsExcludedImportTypes`)
- Matches [`import/internal-regex`](../../README.md#importinternal-regex), it will be considered **internal**
- Is an absolute path, it will be considered **unknown**
- Has the name of a Node.js core module (using [is-core-module](https://www.npmjs.com/package/is-core-module)), it will be considered **builtin**
- Matches [`import/core-modules`](../../README.md#importcore-modules), it will be considered **builtin**
- Is a path relative to the parent directory (e.g. starts with `../`), it will be considered **parent**
- Is one of `['.', './', './index', './index.js']`, it will be considered **index**
- Is a path relative to the current directory (e.g. starts with `./`), it will be considered **sibling**
- Is a path pointing to a file outside the current package's root directory (determined using [package-up](https://www.npmjs.com/package/package-up)), it will be considered **external**
- Matches [`import/external-module-folders`](../../README.md#importexternal-module-folders) (defaults to matching anything pointing to files within the current package's `node_modules` directory), it will be considered **external**
- Is a path pointing to a file within the current package's root directory (determined using [package-up](https://www.npmjs.com/package/package-up)), it will be considered **internal**
- Has a name that looks like a scoped package (e.g. `@scoped/package-name`), it will be considered **external**
- Has a name that starts with a word character, it will be considered **external**
- Reaches this point, it will be ignored entirely

At the end of the process, if they co-exist in the same file, all top-level `require()` statements that haven't been ignored are shifted (with respect to their order) below any ES6 `import` or similar declarations. Finally, any type-only declarations are potentially reorganized according to [`sortTypesAmongThemselves`](#sorttypesamongthemselves-truefalse).

## Fail

```ts
import _ from 'lodash';
import path from 'path'; // `path` import should occur before import of `lodash`

// -----

var _ = require('lodash');
var path = require('path'); // `path` import should occur before import of `lodash`

// -----

var path = require('path');
import foo from './foo'; // `import` statements must be before `require` statement
```

## Pass

```ts
import path from 'path';
import _ from 'lodash';

// -----

var path = require('path');
var _ = require('lodash');

// -----

// Allowed as ̀`babel-register` is not assigned.
require('babel-register');
var path = require('path');

// -----

// Allowed as `import` must be before `require`
import foo from './foo';
var path = require('path');
```

## Limitations of `--fix`

Unbound imports are assumed to have side effects, and will never be moved/reordered. This can cause other imports to get "stuck" around them, and the fix to fail.

```javascript
import b from 'b'
import 'format.css';  // This will prevent --fix from working.
import a from 'a'
```

As a workaround, move unbound imports to be entirely above or below bound ones.

```javascript
import 'format1.css';  // OK
import b from 'b'
import a from 'a'
import 'format2.css';  // OK
```

## Options

This rule supports the following options:

### `groups: [array]`

How groups are defined, and the order to respect. `groups` must be an array of `string` or [`string`]. The only allowed `string`s are:
`"builtin"`, `"external"`, `"internal"`, `"unknown"`, `"parent"`, `"sibling"`, `"index"`, `"object"`, `"type"`.
The enforced order is the same as the order of each element in a group. Omitted types are implicitly grouped together as the last element. Example:

```ts
[
  'builtin', // Built-in types are first
  ['sibling', 'parent'], // Then sibling and parent types. They can be mingled together
  'index', // Then the index file
  'object',
  // Then the rest: internal and external type
]
```

The default value is `["builtin", "external", "parent", "sibling", "index"]`.

You can set the options like this:

```ts
"import/order": [
  "error",
  {
    "groups": [
      "index",
      "sibling",
      "parent",
      "internal",
      "external",
      "builtin",
      "object",
      "type"
    ]
  }
]
```

### `pathGroups: [array of objects]`

To be able to group by paths mostly needed with aliases pathGroups can be defined.

Properties of the objects

| property       | required | type   | description   |
|----------------|:--------:|--------|---------------|
| pattern        |     x    | string | minimatch pattern for the paths to be in this group (will not be used for builtins or externals) |
| patternOptions |          | object | options for minimatch, default: { nocomment: true } |
| group          |     x    | string | one of the allowed groups, the pathGroup will be positioned relative to this group |
| position       |          | string | defines where around the group the pathGroup will be positioned, can be 'after' or 'before', if not provided pathGroup will be positioned like the group |

```json
{
  "import/order": ["error", {
    "pathGroups": [
      {
        "pattern": "~/**",
        "group": "external"
      }
    ]
  }]
}
```

### `distinctGroup: [boolean]`

This changes how `pathGroups[].position` affects grouping. The property is most useful when `newlines-between` is set to `always` and at least 1 `pathGroups` entry has a `position` property set.

By default, in the context of a particular `pathGroup` entry, when setting `position`, a new "group" will silently be created. That is, even if the `group` is specified, a newline will still separate imports that match that `pattern` with the rest of the group (assuming `newlines-between` is `always`). This is undesirable if your intentions are to use `position` to position _within_ the group (and not create a new one). Override this behavior by setting `distinctGroup` to `false`; this will keep imports within the same group as intended.

Note that currently, `distinctGroup` defaults to `true`. However, in a later update, the default will change to `false`

Example:

```json
{
  "import/order": ["error", {
    "newlines-between": "always",
    "pathGroups": [
      {
        "pattern": "@app/**",
        "group": "external",
        "position": "after"
      }
    ],
    "distinctGroup": false
  }]
}
```

### `pathGroupsExcludedImportTypes: [array]`

This defines import types that are not handled by configured pathGroups.
If you have added path groups with patterns that look like `"builtin"` or `"external"` imports, you have to remove this group (`"builtin"` and/or `"external"`) from the default exclusion list (e.g., `["builtin", "external", "object"]`, etc) to sort these path groups correctly.

Example:

```json
{
  "import/order": ["error", {
    "pathGroups": [
      {
        "pattern": "@app/**",
        "group": "external",
        "position": "after"
      }
    ],
    "pathGroupsExcludedImportTypes": ["builtin"]
  }]
}
```

[Import Type](https://github.com/import-js/eslint-plugin-import/blob/HEAD/src/core/importType.js#L90) is resolved as a fixed string in predefined set, it can't be a `patterns`(e.g., `react`, `react-router-dom`, etc). See [#2156] for details.

### `newlines-between: [ignore|always|always-and-inside-groups|never]`

Enforces or forbids new lines between import groups:

 - If set to `ignore`, no errors related to new lines between import groups will be reported.
 - If set to `always`, at least one new line between each group will be enforced, and new lines inside a group will be forbidden. To prevent multiple lines between imports, core `no-multiple-empty-lines` rule can be used.
 - If set to `always-and-inside-groups`, it will act like `always` except newlines are allowed inside import groups.
 - If set to `never`, no new lines are allowed in the entire import section.

The default value is `"ignore"`.

With the default group setting, the following will be invalid:

```ts
/* eslint import/order: ["error", {"newlines-between": "always"}] */
import fs from 'fs';
import path from 'path';
import index from './';
import sibling from './foo';
```

```ts
/* eslint import/order: ["error", {"newlines-between": "always-and-inside-groups"}] */
import fs from 'fs';

import path from 'path';
import index from './';
import sibling from './foo';
```

```ts
/* eslint import/order: ["error", {"newlines-between": "never"}] */
import fs from 'fs';
import path from 'path';

import index from './';

import sibling from './foo';
```

while those will be valid:

```ts
/* eslint import/order: ["error", {"newlines-between": "always"}] */
import fs from 'fs';
import path from 'path';

import index from './';

import sibling from './foo';
```

```ts
/* eslint import/order: ["error", {"newlines-between": "always-and-inside-groups"}] */
import fs from 'fs';

import path from 'path';

import index from './';

import sibling from './foo';
```

```ts
/* eslint import/order: ["error", {"newlines-between": "never"}] */
import fs from 'fs';
import path from 'path';
import index from './';
import sibling from './foo';
```

### `named: true|false|{ enabled: true|false, import: true|false, export: true|false, require: true|false, cjsExports: true|false, types: mixed|types-first|types-last }`

Enforce ordering of names within imports and exports:

 - If set to `true`, named imports must be ordered according to the `alphabetize` options
 - If set to `false`, named imports can occur in any order

`enabled` enables the named ordering for all expressions by default.
Use `import`, `export` and `require` and `cjsExports` to override the enablement for the following kind of expressions:

 - `import`:

   ```ts
   import { Readline } from "readline";
   ```

 - `export`:

   ```ts
   export { Readline };
   // and
   export { Readline } from "readline";
   ```

 - `require`

   ```ts
   const { Readline } = require("readline");
   ```

 - `cjsExports`

   ```ts
   module.exports.Readline = Readline;
   // and
   module.exports = { Readline };
   ```

The `types` option allows you to specify the order of `import`s and `export`s of `type` specifiers.
Following values are possible:

 - `types-first`: forces `type` specifiers to occur first
 - `types-last`: forces value specifiers to occur first
 - `mixed`: sorts all specifiers in alphabetical order

The default value is `false`.

Example setting:

```ts
{
  named: true,
  alphabetize: {
    order: 'asc'
  }
}
```

This will fail the rule check:

```ts
/* eslint import/order: ["error", {"named": true, "alphabetize": {"order": "asc"}}] */
import { compose, apply } from 'xcompose';
```

While this will pass:

```ts
/* eslint import/order: ["error", {"named": true, "alphabetize": {"order": "asc"}}] */
import { apply, compose } from 'xcompose';
```

### `alphabetize: {order: asc|desc|ignore, orderImportKind: asc|desc|ignore, caseInsensitive: true|false}`

Sort the order within each group in alphabetical manner based on **import path**:

 - `order`: use `asc` to sort in ascending order, and `desc` to sort in descending order (default: `ignore`).
 - `orderImportKind`: use `asc` to sort in ascending order various import kinds, e.g. imports prefixed with `type` or `typeof`, with same import path. Use `desc` to sort in descending order (default: `ignore`).
 - `caseInsensitive`: use `true` to ignore case, and `false` to consider case (default: `false`).

Example setting:

```ts
alphabetize: {
  order: 'asc', /* sort in ascending order. Options: ['ignore', 'asc', 'desc'] */
  caseInsensitive: true /* ignore case. Options: [true, false] */
}
```

This will fail the rule check:

```ts
/* eslint import/order: ["error", {"alphabetize": {"order": "asc", "caseInsensitive": true}}] */
import React, { PureComponent } from 'react';
import aTypes from 'prop-types';
import { compose, apply } from 'xcompose';
import * as classnames from 'classnames';
import blist from 'BList';
```

While this will pass:

```ts
/* eslint import/order: ["error", {"alphabetize": {"order": "asc", "caseInsensitive": true}}] */
import blist from 'BList';
import * as classnames from 'classnames';
import aTypes from 'prop-types';
import React, { PureComponent } from 'react';
import { compose, apply } from 'xcompose';
```

### `warnOnUnassignedImports: true|false`

 - default: `false`

Warns when unassigned imports are out of order.  These warning will not be fixed
with `--fix` because unassigned imports are used for side-effects and changing the
import of order of modules with side effects can not be done automatically in a
way that is safe.

This will fail the rule check:

```ts
/* eslint import/order: ["error", {"warnOnUnassignedImports": true}] */
import fs from 'fs';
import './styles.css';
import path from 'path';
```

While this will pass:

```ts
/* eslint import/order: ["error", {"warnOnUnassignedImports": true}] */
import fs from 'fs';
import path from 'path';
import './styles.css';
```

### `sortTypesAmongThemselves: true|false`

> \[!NOTE]
>
> This setting is only meaningful when `"type"` is included in `groups`.

Sort type-only imports separately from normal non-type imports.

When enabled, the intragroup sort order of type-only imports will mirror the intergroup ordering of normal imports as defined by `groups`, `pathGroups`, etc.

Given the following settings:

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index']
}
```

This example will fail the rule check:

```ts
import type A from "fs";
import type B from "path";
import type C from "../foo.js";
import type D from "./bar.js";
import type E from './';

import a from "fs";
import b from "path";
import c from "../foo.js";
import d from "./bar.js";
import e from "./";
```

However, if we add `sortTypesAmongThemselves: true`:

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index'],
  sortTypesAmongThemselves: true
}
```

The same example will pass.

### `newlines-between-types: [ignore|always|always-and-inside-groups|never]`

> \[!NOTE]
>
> This setting is only meaningful when `sortTypesAmongThemselves` is enabled.

Enforces or forbids new lines between _type-only_ import groups. It is otherwise identical to [`newlines-between`](#newlines-between-ignorealwaysalways-and-inside-groupsnever).

The default value is the value of `newlines-between`. When determining if a new line is enforceable or forbidden between the type-only imports and the normal imports, `newlines-between-types` takes precedence over `newlines-between`.

Given the following settings:

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index'],
  sortTypesAmongThemselves: true,
  'newlines-between': 'always'
}
```

This example will fail the rule check:

```ts
import type A from "fs";
import type B from "path";
import type C from "../foo.js";
import type D from "./bar.js";
import type E from './';

import a from "fs";
import b from "path";

import c from "../foo.js";

import d from "./bar.js";

import e from "./";
```

However, if we add `newlines-between-types: 'ignore'`:

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index'],
  sortTypesAmongThemselves: true,
  'newlines-between': 'always',
  'newlines-between-types': 'ignore'
}
```

The same example will pass.

Note the newline after `import type E from './';` but before `import a from "fs";`. This newline separates the type-only imports from the normal imports. Its existence is governed by `newlines-between-types`.

For example, the following will pass even though there's a newline preceding the normal import and `newlines-between` is set to "never":

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index'],
  sortTypesAmongThemselves: true,
  'newlines-between': 'never',
  'newlines-between-types': 'always'
}
```

```ts
import type A from "fs";

import type B from "path";

import type C from "../foo.js";

import type D from "./bar.js";

import type E from './';

import a from "fs";
import b from "path";
import c from "../foo.js";
import d from "./bar.js";
import e from "./";
```

While the following fails due to the newline between the last type import and the first normal import:

```ts
{
  groups: ['type', 'builtin', 'parent', 'sibling', 'index'],
  sortTypesAmongThemselves: true,
  'newlines-between': 'always',
  'newlines-between-types': 'never'
}
```

```ts
import type A from "fs";
import type B from "path";
import type C from "../foo.js";
import type D from "./bar.js";
import type E from './';

import a from "fs";

import b from "path";

import c from "../foo.js";

import d from "./bar.js";

import e from "./";
```

## Related

 - [`import/external-module-folders`] setting

 - [`import/internal-regex`] setting

[`import/external-module-folders`]: ../../README.md#importexternal-module-folders

[`import/internal-regex`]: ../../README.md#importinternal-regex
