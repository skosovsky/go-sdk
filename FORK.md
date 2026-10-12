# Exact identities and cancellation integration

Branch: `integration/exact-ids-cancellation`.
Base: upstream `8c761a2d6711d0ad92b363fe6230547d579a8a23`.
The module path remains `github.com/modelcontextprotocol/go-sdk`.

Included patches (original authors and source commits retained):

- [#1375](https://github.com/modelcontextprotocol/go-sdk/pull/1375), `9174a3ffc71cf5c7617a6c64afca0802b855f770`: exact numeric identities, public `MakeID(json.Number)`, and precision-preserving `CancelledParams.RequestID` decoding.
- [#1253](https://github.com/modelcontextprotocol/go-sdk/pull/1253), `8beb4c65e0c03e7ef0932599182797abeb4bc293`: bounded decimal/exponent parser and regression coverage for response correlation and adjacent large IDs.
- [#1390](https://github.com/modelcontextprotocol/go-sdk/pull/1390), `f5811a6c02965f8373849a06199d06b693b307e7`: bind cancellation to the active request before the read loop advances and its ID can be reused.

## Integration decisions

The parser from #1253 replaces the overlapping big.Rat implementation from #1375. Both wire IDs and `MakeID(json.Number)` use this parser. Cancellation continues through `CancelledParams` and `MakeID`, preserving upstream reason/cause propagation and metadata behavior. Tests from #1253 are adapted to that path and the current logger requirement.

Numeric IDs must be integral signed int64 values; decimal and exponent forms are accepted when exact. Numeric lexemes are limited to 128 bytes. String IDs retain their separate identity and are not subject to this numeric bound. Decoded numeric `CancelledParams.RequestID` values are `json.Number`; metadata numbers retain their existing representation. Legacy explicit `MakeID(float64)` behavior is unchanged; callers requiring exact identities should pass `json.Number` or use `Int64ID`.

This integration does not include additional invalid-request error framing, STDIO recovery, or float64 hardening from application-specific dependency exception proposals. It does not by itself establish application migration acceptance.

## Validation

Validated with Go 1.27.1 on darwin/arm64:

- `go test ./... -timeout=10m`
- `go test -race ./internal/jsonrpc2 ./mcp -timeout=10m` (includes integration parser boundary cases)
- `go test -race ./mcp -run TestCancellation -count=20 -timeout=5m`
- `go vet ./...`
- `go build ./...`

All passed. External conformance scripts and application-level migration gates were not run.
