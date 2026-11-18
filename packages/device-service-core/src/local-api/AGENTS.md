# local-api — device-relay wire protocol (vendored v1)

**Score 11** (46 TS modules, ~6.2k LOC; distinct domain — this file and `docs/local-api.md` are the only guidance in `device-service-core`).

## OVERVIEW

Typed v1 wire protocol + client for the device-relay daemon: versioned handshake, framed requests/events, listing deltas, uploads. Vendored from `sisyphuslabs/omo-relay-daemon@edd2ee49` — every source header carries the provenance line and `PIN.json` records the upstream commit and import rewrites. Consumers: `../device-service.ts`, `../delivery-consumer.ts`, `../registrant.ts`, `../bootstrap.ts`; behavior suites live in `../../test/local-api/` (conformance, dacl-verdict, deliveries-decode, fake-owner-modes, ...).

## STRUCTURE (file families)

| Prefix | Domain |
|--------|--------|
| `client*.ts` (5) | `client.ts` facade (`connectLocalApi`) over `client-open.ts` (handshake/open, timeouts), `client-connection.ts` (request deadlines), `client-listing.ts`, `client-assembly.ts` |
| `protocol*.ts` (8) | `protocol.ts` type barrel composing the `protocol-{events,control,machine,delegated,presenter,deliveries,command}.ts` shape modules |
| `validate*.ts` (17) | `validate.ts` facade (`validateHello`/`Welcome`/`Request`/`Response`/`Event` + snapshot) over per-family validators (`validate-{fields,requests,events,machine,delegated,deliveries,command,update,shapes,...}.ts`) |
| `vocabulary.ts`, `framing.ts`, `bytes-encoding.ts`, `chunking.ts`, `errors.ts` | Literal vocabularies, frame encoding, byte codecs, `LocalApiError` |
| `transport.ts`, `access.ts`, `dacl.ts`, `proof.ts`, `strict` helpers | Connection transport, POSIX access checks, ACL verdicts, handshake proofs |
| `upload.ts`, `listing.ts`, `session-ref.ts`, `command-text.ts`, `spec-proto.ts` | Upload parts (`LOCAL_UPLOAD_PARTS_MAX`), listing rows/limits, session refs, relay-spec proto shim |
| `PIN.json` | Vendoring provenance: upstream repo + commit, import rewrites |

## CONVENTIONS

- Read `v` (version) BEFORE shape: a hello/welcome from a newer peer is refused `"version"`, never `"malformed"` (documented at `validate.ts:67`; enforced in `validateHello`).
- Wire object properties are `readonly`; vocabularies are literal tuples with derived unions.
- Relative imports keep the `.ts` extension (vendored upstream style — differs from the rest of the repo's omo-opencode tree).
- Unknown request fields are refused; malformed-request reasons name the FIELD, never the value. API names/values/doc sentences stay free of transport-between-devices vocabulary (`docs/local-api.md`).
- No colocated tests — suites live in `packages/device-service-core/test/local-api/`.

## ANTI-PATTERNS

- Do not hand-edit vendored files without updating `PIN.json` and each header's provenance line.
- Do not reorder validation: shape errors on a versioned message must not mask the `"version"` verdict.

## COMMANDS

- `bun run --cwd packages/device-service-core typecheck` / `bun run --cwd packages/device-service-core test` (covers the `test/local-api/` conformance suites).
