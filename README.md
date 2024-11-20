Try transpiling to CJS with default settings:

```
$ pnpm exec swc -C module.type=commonjs ts_package.ts > ts_package.cjs 
Successfully compiled 1 file with swc.
```

The resulting output correctly imports `cjs_with_interop.cjs`'s default export:

```
$ node ts_package.cjs                                                 
ts_package.cjs default
```

But node can't use the named exports from ts_package.cjs:

```
$ node esm_consumer.mjs                                               
file:///Users/john/figma/swc_interop/esm_consumer.mjs:1
import { d } from "./ts_package.cjs";
         ^
SyntaxError: Named export 'd' not found. The requested module './ts_package.cjs' is a CommonJS module, which may not support all module.exports as named exports.
CommonJS modules can always be imported via the default export, for example using:

import pkg from './ts_package.cjs';
const { d } = pkg;

    at ModuleJob._instantiate (node:internal/modules/esm/module_job:134:21)
    at async ModuleJob.run (node:internal/modules/esm/module_job:217:5)
    at async ModuleLoader.import (node:internal/modules/esm/loader:316:24)
    at async asyncRunEntryPointWithESMLoader (node:internal/modules/run_main:123:5)

Node.js v20.14.0
```

Okay, let's try using the `importInterop` option:

```
$ pnpm exec swc -C module.type=commonjs -C module.importInterop=node ts_package.ts > ts_package.cjs 
Successfully compiled 1 file with swc.
```

Now node can use the named exports from ts_package.cjs, but the default export from `cjs_with_interop.cjs` is broken!

```
$ node esm_consumer.mjs                                                                            
ts_package.cjs { __esModule: true, default: 'default' }
esm_consumer.mjs { __esModule: true, default: 'default' }
```

Let's try the suggested answer of using `importInterop=swc`:

```
$ pnpm exec swc -C module.type=commonjs -C module.importInterop=swc ts_package.ts > ts_package.cjs
Successfully compiled 1 file with swc.
```

Doesn't work. The result is the same as the default `importInterop` setting.

```
$ node esm_consumer.mjs                                                                            
file:///Users/john/figma/swc_interop/esm_consumer.mjs:1
import { d } from "./ts_package.cjs";
         ^
SyntaxError: Named export 'd' not found. The requested module './ts_package.cjs' is a CommonJS module, which may not support all module.exports as named exports.
CommonJS modules can always be imported via the default export, for example using:

import pkg from './ts_package.cjs';
const { d } = pkg;

    at ModuleJob._instantiate (node:internal/modules/esm/module_job:134:21)
    at async ModuleJob.run (node:internal/modules/esm/module_job:217:5)
    at async ModuleLoader.import (node:internal/modules/esm/loader:316:24)
    at async asyncRunEntryPointWithESMLoader (node:internal/modules/run_main:123:5)

Node.js v20.14.0
```
