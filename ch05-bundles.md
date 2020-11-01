A bundle allows tasks and networks to be _configurable_, which means their behavior can be influenced by overriding parts of the bundle at instantiation. A bundle is a stateless entity that defines types, constants, and functions that are common to several tasks and networks.

    package com.synflow.sha256;

    /**
     * Description of the SHACommon bundle
     */
    bundle SHACommon {
      // type definition
      typedef u6 addr_t;

      // constant definition
      u9 HASH_SIZE = 256;

      u32 Ch(u32 x, u32 y, u32 z) {
        return (x & y) ^ (~x & z);
      }

      u32 Maj(u32 x, u32 y, u32 z) {
        return (x & y) ^ (x & z) ^ (y & z);
      }
    }

The constants, types, functions of a bundle can be referenced in other entities in three different ways. In the "SHACommon" bundle above, the "addr" type, the "HASH_SIZE" constant, and the "Ch" and "Maj" functions can be referenced as follows:

1. Typical, shortest way: import everything from the bundle, and reference them by their short name, like `addr` or `Ch`.
2. Reasonably short way: import the bundle, and reference objects by their qualified name `SHACommon.addr` or `SHACommon.Ch`.
3. Verbose way: Reference objects by their full name: `com.synflow.sha256.SHACommon.addr` or `com.synflow.sha256.SHACommon`


---
```
Copyright 2014-2020 Synflow SAS

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
