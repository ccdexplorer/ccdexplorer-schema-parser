## Schema parsing via Rust bindings

This repository holds the Rust bindings in `src/lib.rs`. They are simple exports of functionality
from `concordium-contracts-common`. To build them locally, you require [maturin](https://github.com/PyO3/maturin) and Rust installed.

We've tested with Rust 1.74 and 1.77.

## Project structure

The entrypoint for Python consumers is the `ccdexplorer_schema_parser` package which has a
single class `Schema`. The constructor will instantiate the schema from a
deployed Wasm module.

After that the constructed object can be used to parse events or return values
using the schema in the module.


```python
from ccdexplorer_schema_parser.Schema import Schema
from ccdexplorer_fundamentals.enums import NET
from ccdexplorer_fundamentals.GRPCClient import GRPCClient
from ccdexplorer_fundamentals.GRPCClient.types_pb2 import VersionedModuleSource

versioned_module: VersionedModuleSource = (
    self.grpcclient.get_module_source_original_classes(
        module_ref, "last_final", net=NET(net)
    )
)
schema = Schema(versioned_module.v1.value, 1) if versioned_module.v1
    else Schema(versioned_module.v0.value, 0)
```

To parse a logged event from an `account_transaction`, use the following call:
```python
event_json = schema.event_to_json(
    source_module_name, bytes.fromhex(event)
    )
```
Where `source_module_name` is the name of the module (corresponding to the `module_ref` you have used to parse the schema and `event` is the `hex` representation of the logged event).
If this can be parsed, the result will be a dictionary.

## Building

Run `pip install -r requirements.txt` to install dependencies.

Run `maturin build` to build the project.

This will produce a python wheel in `target/wheels` that will contain both the
compiled Rust binaries and python wrappers. The compiled package is platform
specific, so a package built on, e.g., Linux will not work on Windows.

## Deploy
This repository has a `CI.yml` that builds this package for various configurations and publishes this to Pypi.
