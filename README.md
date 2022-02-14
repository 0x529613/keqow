# I. Using emscripten to compile c++ code to wasm:

```cmd
em++ --no-entry -O3 -DNDEBUG --bind bindings/*Bindings.cpp -I(headerDir) source/*.cpp -s WASM=1 -s EXPORT_ES6=1 -s MODULARIZE=1 -o *.mjs
```

# II. Modify output file _(\*.mjs)_:

-   Add `/* eslint-disable */` on **top**, to avoid syntax errors.

-   **Replace:**

```js
var _scriptDir = import.meta.url;
```

_to:_

```js
var _scriptDir =
	typeof document !== 'undefined' && document.currentScript
		? document.currentScript.src
		: undefined;
if (typeof __filename !== 'undefined') _scriptDir = _scriptDir || __filename;
```

-   **Replace:**

```js
scriptDirectory = self.location.href;
```

_to:_

```js
scriptDirectory = window.self.location.href;
```

-   **Replace:**

```js
var wasmBinaryFile;

if (Module['locateFile']) {
	wasmBinaryFile = '*.wasm';
	if (!isDataURI(wasmBinaryFile)) {
		wasmBinaryFile = locateFile(wasmBinaryFile);
	}
} else {
	wasmBinaryFile = new URL('*.wasm', import.meta.url).toString();
}
```

_to:_

```js
var wasmBinaryFile = '';

if (!isDataURI(wasmBinaryFile)) {
	wasmBinaryFile = locateFile(wasmBinaryFile);
}
```

-   **Remove** `getBinary` function

-   **Replace** `getBinaryPromise` functions

_with:_

```js
const getBinaryPromise = () =>
	new Promise((resolve, reject) => {
		fetch(wasmBinaryFile, { credentials: 'same-origin' })
			.then((response) => {
				if (!response['ok']) {
					throw (
						"failed to load wasm binary file at '" +
						wasmBinaryFile +
						"'"
					);
				}
				return response['arrayBuffer']();
			})
			.then(resolve)
			.catch(reject);
	});
```

-   **Replace:**

```js
if (
	!wasmBinary &&
	typeof WebAssembly.instantiateStreaming === 'function' &&
	!isDataURI(wasmBinaryFile) &&
	!isFileURI(wasmBinaryFile) &&
	typeof fetch === 'function'
);
```

_to:_

```js
if (
	!wasmBinary &&
	typeof WebAssembly.instantiateStreaming === 'function' &&
	typeof fetch === 'function'
);
```

# III. Using wasm in javascript:

```js
import Sample from 'path/to/output.mjs';

const sample = Sample({
	locateFile: () => 'path/to/*.wasm',
});

sample.then((core) => {
	core.cppFunction(...myParams);
});
```
