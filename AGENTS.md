# OAuth API Server — AI Assistant Guidelines

**For:** Claude Code, Cursor, GitHub Copilot, Gemini, Windsurf, and other AI coding assistants.

OpenShift OAuth API server: OAuth 2.0 token management and authentication. Manages OAuthAccessTokens, OAuthAuthorizeTokens, user identity resources.

## Build & Test Commands

```bash
make build
make test-unit                  # unit tests (./pkg/... ./cmd/...)
make test-e2e                   # E2E (requires cluster, 3h timeout, sequential -p 1)
make run-e2e-test WHAT=<test-name>

# OTE framework (not standard go test)
./oauth-apiserver-tests-ext run-suite "openshift/oauth-apiserver/all"
./oauth-apiserver-tests-ext run-test "test-name"

# Code generation
make update                     # regenerate conversions, deepcopy, defaults, openapi
make verify                     # verify before commit

# Verification
make verify && gofmt -s -w . && golangci-lint run

# Dependencies
go mod tidy && go mod vendor
```

## Key Resources

- **OAuthAccessToken** — OAuth 2.0 access tokens (`pkg/oauth/`, validation: `pkg/tokenvalidation/`)
- **OAuthAuthorizeToken** — Authorization codes (short-lived)
- **User/Identity** — User resources (`pkg/user/`)

## Token Validation

`pkg/tokenvalidation/` contains validation logic:
- `expirationvalidator.go`, `timeoutvalidator.go`, `uidvalidator.go`, `bootstrapauthenticator.go`

External OIDC integration: `pkg/externaloidc/`

## Code Generation

Generated files (never hand-edit): `zz_generated.conversion.go`, `zz_generated.deepcopy.go`, `zz_generated.defaults.go`, `pkg/openapi/zz_generated.openapi.go`

After modifying types in `pkg/api/`: `make update && make verify`

## Testing

**Unit tests:** `pkg/*_test.go`, run `make test-unit`, no cluster required

**E2E tests:** `test/e2e/` (Ginkgo + OTE framework)
- Key: `useroauthaccesstokens_test.go`, `tokenreviews.go`
- Requires live cluster
- 3h timeout, sequential (`-p 1`)
- OTE binary: `oauth-apiserver-tests-ext`

## PR / Commit Conventions

All PRs MUST have exactly 2 commits:
1. **Code commit:** Infrastructure changes (exclude go.mod, go.sum, vendor/, generated files)
2. **Generated/vendor commit:** Dependencies (go.mod, go.sum, vendor/) OR generated artifacts (`make update` output)

Base on `upstream/main`, not `origin/main`.

## Common Tasks

Modify token validation: Edit `pkg/tokenvalidation/`, add tests, `make test-unit`

Add OAuth feature: Update `pkg/api/`, `make update`, implement `pkg/oauth/`, add tests, `make verify && make test-unit && make test-e2e`

Debug: `oc get oauthaccesstokens`, `oc describe oauthaccesstoken/<name>`, `oc logs -n openshift-oauth-apiserver -l app=oauth-apiserver`

## Build System

- Go modules + vendor/
- OpenShift build-machinery-go (`golang.mk`, `targets/openshift/*.mk`)

## CI/CD

- Prow (OpenShift CI)
- Pre-merge: verify, test-unit, test-e2e
- Registry: registry.svc.ci.openshift.org/ocp/4.3:oauth-apiserver

## Known Issues

- E2E timeout: 3h, some tests slow
- Sequential execution: `-p 1`
- OTE framework (not go test)
- Always `make verify` before commit

## Links

- https://github.com/openshift/oauth-apiserver
- https://github.com/openshift-eng/openshift-tests-extension
