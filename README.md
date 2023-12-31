## Why Does It Exist?
If you generate your backend api with protobuf, for example, I generate openapi scheme for the api. And also I have option for grpc if needed or even for grpc-web.

### Why not use grpc-web instead ?
To use grpc-web, you need already implemented grpc service and need to integrate it with grpc-web server. 
Secondly, you need proxy for serialization/deserialization protobuf specs
Also, you need bring the protobuf to the frontend, which may be not trivial in compared to just generating types for typescript.

### Why not generate client side with openapi ?
It may actually be a good approach, but I don't know how good it. 
There could be issues with code generation approach, that may lead to losing the control of your fetch client.
It even might require to integrate it into the system, replacing the logic of how your fetch client interacts with the system.

### What I do 
I take the http annotation values and generate `Request` for fetch.

Example of the result:
https://github.com/romashorodok/protoc-gen-fetch-types/blob/376dd630e3e7961c6b9d41d06943dcbb95079ed2/examples/gen/product.ts#L1-L16

Other examples can be found at [examples](https://github.com/romashorodok/protoc-gen-fetch-types/tree/main/examples) folder.
  
## How to use it ?
Currently, you need build it by hand:
```shell
go build -o protoc-gen-fetch-types main.go
```

buf.gen.yaml
```yaml
version: v1
plugins:
  - plugin: protoc-gen-fetch-types
    path: ./protoc-gen-fetch-types
    # The output folder for the code, where it can be bundled
    out: ./examples/gen
    opt:
      # Exclude google protobuf annotations when generate imports
      # NOTE: name must be a package first prefix name. Not a file name or folder
      # Also may add own ignore `["google" "openapiv3"]`
      - import_ignore=["google"]
```

### With nix package manager
In the root folder:
```shell
mkdir protoc-gen-fetch-types
cat <<EOF > protoc-gen-fetch-types/default.nix
{ lib, buildGoModule, fetchFromGitHub }:

# to get rev field
# git ls-remote https://github.com/romashorodok/protoc-gen-fetch-types main

buildGoModule rec {
  name = "protoc-fetch-types";

  src = fetchFromGitHub {
    owner = "romashorodok";
    repo = name;
    rev = "0fc0698ac58c2de8522c2f5da66ed2789e2cec60";
    sha256 = "c0HY4xYYgpZ/ecq/FMMbw8baTuuEMfFlxfKiCuHSARY=";
  };

  vendorHash = "sha256-l/tjPEkl/RimBxkkR12xrMNI32SnbK74k7gagHdL9k4=";

  doCheck = false;
}
EOF
```

Then add it to the nix-shell env:
```shell
cat <<EOF > shell.nix
{ pkgs ? import <nixpkgs> { } }:

let
  protoc-gen-fetch-types-main = pkgs.callPackage ./protoc-gen-fetch-types { };
in
pkgs.mkShell {
  buildInputs = with pkgs; [
    protoc-gen-fetch-types-main
  ];
}
EOF
```
Finally, run it:
```shell
nix-shell
```
