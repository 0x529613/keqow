# Using emscripten to compile c++ code to wasm:

em++ --no-entry -O3 -DNDEBUG --bind bindings/\_\_Bindings.cpp -I(include header) source/_.cpp -s WASM=1 -s EXPORT_ES6=1 -s MODULARIZE=1 -o _.mjs

# Modify output file (\*.mjs):

Add /_ eslint-disable _/ at top of the file, to avoid syntax errors.

Replace var \_scriptDir = import.meta.url; with:

var \_scriptDir =
typeof document !== 'undefined' && document.currentScript
? document.currentScript.src
: undefined;
if (typeof **filename !== 'undefined')
\_scriptDir = \_scriptDir || **filename;

Replace scriptDirectory = self.location.href; with scriptDirectory = window.self.location.href;

Replace:

var wasmBinaryFile;
if (Module['locateFile']) {
wasmBinaryFile = 'hello2.wasm';
if (!isDataURI(wasmBinaryFile)) {
wasmBinaryFile = locateFile(wasmBinaryFile);
}
} else {
wasmBinaryFile = new URL('hello2.wasm', import.meta.url).toString();
}

with:

var wasmBinaryFile = '';
if (!isDataURI(wasmBinaryFile)) {
wasmBinaryFile = locateFile(wasmBinaryFile);
}

Remove getBinary function;

Replace getBinaryPromise functions with:

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

Replace if (!wasmBinary && typeof WebAssembly.instantiateStreaming === 'function' && typeof fetch === 'function'); with:
if (!wasmBinary && typeof WebAssembly.instantiateStreaming === "function" && typeof fetch === "function");

# Using wasm in javascript:

import Sample from 'path/to/output.mjs';

const sample = Sample({
locateFile: () =>
'path/to/\*.wasm',
});

sample.then((core) => {
core.my_cpp_function(...my_params);
});

(This file is demo and will be edited)
