# Third-party licenses for the Prefex binary

Every third-party Go module compiled into the released Prefex binary, with its
license. Generated from the module graph of the release build
(`GOFLAGS=-tags=pro`, license texts read from the Go module cache). Prefex's own
source is not listed here. Licenses for the optional Kompress sidecar
(Python/model) are in [`NOTICE`](NOTICE).

The largest addition since the 22-module list: the embedded
[Open Policy Agent](https://github.com/open-policy-agent/opa) engine
(Apache-2.0) that powers the policy gateway, and its dependency tree.

| Module | License |
|--------|---------|
| `github.com/agnivade/levenshtein` | MIT |
| `github.com/cespare/xxhash/v2` | MIT |
| `github.com/coreos/go-oidc/v3` | Apache-2.0 |
| `github.com/dlclark/regexp2/v2` | MIT |
| `github.com/dustin/go-humanize` | MIT |
| `github.com/elazarl/goproxy` | BSD-3-Clause |
| `github.com/go-jose/go-jose/v4` | Apache-2.0 |
| `github.com/gobwas/glob` | MIT |
| `github.com/google/uuid` | BSD-3-Clause |
| `github.com/lestrrat-go/blackmagic` | MIT |
| `github.com/lestrrat-go/dsig` | MIT |
| `github.com/lestrrat-go/httpcc` | MIT |
| `github.com/lestrrat-go/httprc/v3` | MIT |
| `github.com/lestrrat-go/jwx/v3` | MIT |
| `github.com/lestrrat-go/option/v2` | MIT |
| `github.com/mattn/go-isatty` | MIT |
| `github.com/ncruces/go-strftime` | MIT |
| `github.com/open-policy-agent/opa` | Apache-2.0 |
| `github.com/rcrowley/go-metrics` | BSD-2-Clause |
| `github.com/redis/go-redis/v9` | BSD-2-Clause |
| `github.com/remyoudompheng/bigfft` | BSD-3-Clause |
| `github.com/sirupsen/logrus` | MIT |
| `github.com/tchap/go-patricia/v2` | MIT |
| `github.com/tiktoken-go/tokenizer` | MIT |
| `github.com/valyala/fastjson` | MIT |
| `github.com/vektah/gqlparser/v2` | MIT |
| `github.com/xeipuuv/gojsonpointer` | Apache-2.0 |
| `github.com/xeipuuv/gojsonreference` | Apache-2.0 |
| `github.com/yashtewari/glob-intersection` | Apache-2.0 |
| `go.uber.org/atomic` | MIT |
| `go.yaml.in/yaml/v2` | Apache-2.0 |
| `go.yaml.in/yaml/v3` | Apache-2.0 |
| `golang.org/x/crypto` | BSD-3-Clause |
| `golang.org/x/net` | BSD-3-Clause |
| `golang.org/x/oauth2` | BSD-3-Clause |
| `golang.org/x/sync` | BSD-3-Clause |
| `golang.org/x/sys` | BSD-3-Clause |
| `golang.org/x/text` | BSD-3-Clause |
| `google.golang.org/protobuf` | BSD-3-Clause |
| `gopkg.in/yaml.v3` | Apache-2.0 |
| `modernc.org/libc` | BSD-3-Clause |
| `modernc.org/mathutil` | BSD-3-Clause |
| `modernc.org/memory` | BSD-3-Clause |
| `modernc.org/sqlite` | BSD-3-Clause |
| `sigs.k8s.io/yaml` | MIT |

_45 third-party modules._
