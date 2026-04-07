# Technical Architect Interview — Deployment Configuration, Defaults & Overrides

> All code references below are from Magento 2's `Magento\Framework\App\DeploymentConfig`
> (`lib/internal/Magento/Framework/App/DeploymentConfig.php`).

---

## 1. Core Configuration Concepts

### Q1. What is `DeploymentConfig`?

Configuration resolved and locked **at deploy time**, before the app serves requests.

| Aspect | DeploymentConfig | Runtime Config | App State |
|---|---|---|---|
| When resolved | Deploy time | Per-request | Continuously |
| Mutability | Immutable at runtime | Mutable via admin | Always changing |
| Storage | `env.php`, env vars | Database (core_config_data) | DB, cache, session |
| Examples | DB host, cache backend | Locale, tax rules | Cart, session data |

Magento's `DeploymentConfig` is intentionally immutable — the constructor docblock states:
> *"This object's public interface is intentionally immutable."*

**Why environment-specific:** Each environment connects to different infrastructure. Portability requires externalizing these values from the codebase.

**Use Case:** `app/etc/env.php` holds DB credentials and cache backend config. A single build artifact is promoted across environments by swapping only this file.

---

### Q2. Why is relying *only* on deployment config risky?

- **Bootstrap failure** — Missing key with no default = app won't start. A typo in `env.php` takes down the cluster.
- **Environment drift** — Every environment must define every key; forgotten keys surface as late-stage bugs.
- **DevOps bottleneck** — Devs can't run locally without a complete `env.php`.
- **Human error** — Missing a boolean flag silently changes behavior (e.g., cache disabled in production).

**Use Case:** A deploy omits `retry/max-attempts`. Without a code default of `3`, the service retries infinitely, causing a cascading outage.

---

### Q3. Advantages of defining defaults in code?

Magento's `get()` method already supports this pattern natively:

```php
// DeploymentConfig::get() signature
public function get($key = null, $defaultValue = null)
```

Calling `$deploymentConfig->get('graphql/query_depth_limit', 20)` returns `20` if the key is absent — no crash, no misconfiguration.

**Benefits:** Safe startup, self-documenting behavior, zero-config local dev, simpler testing.

---

## 2. Configuration Resolution Strategy

### Q4. Magento's actual precedence order

Inspecting `DeploymentConfig::reloadData()` reveals the real resolution chain:

```
1. Individual env vars (MAGENTO_DC_*)           ← highest priority
2. JSON env override (MAGENTO_DC__OVERRIDE)
3. Constructor-injected $overrideData
4. Config files merged by Reader::load()        ← lowest priority
5. $defaultValue passed to get()                ← fallback
```

Relevant code from `reloadInitialData()`:

```php
$this->data = array_replace(
    $this->readerLoad,       // files (env.php, config.php)
    $this->overrideData,     // constructor injection
    $this->getEnvOverride()  // MAGENTO_DC__OVERRIDE JSON
);
```

Then in `reloadData()`, individual `MAGENTO_DC_*` env vars take the highest priority:

```php
$this->flatData = $this->getAllEnvOverrides() + $this->flatData;
```

**Why this order:** Env vars allow container-level overrides without touching files — essential for Kubernetes, ECS, and CI/CD.

**Use Case:** Override DB host per pod: `MAGENTO_DC_DB__CONNECTION__DEFAULT__HOST=replica-db.internal` — no file change needed.

---

### Q5. Problems when precedence is undefined?

- **Hidden overrides** — A stale `MAGENTO_DC_*` env var silently masks a config file change.
- **Non-deterministic behavior** — Different pods with different env vars behave differently.
- **Debugging complexity** — "Which source provided this value?" is unanswerable without source-tagging.

**Use Case:** Ops updates `cache/backend` in `env.php`. A forgotten `MAGENTO_DC_CACHE__BACKEND=redis` env var overrides it. Cache migration fails silently.

---

## 3. Applied Design Scenario

### Q6. Designing a safe config class for `feature/enabled`

```php
class FeatureConfig
{
    private const PATH = 'feature/enabled';
    private const DEFAULT = false;

    public function __construct(private readonly DeploymentConfig $deploymentConfig) {}

    public function isEnabled(): bool
    {
        // Uses DeploymentConfig::get($key, $defaultValue) — built-in fallback
        $value = $this->deploymentConfig->get(self::PATH, self::DEFAULT);
        return filter_var($value, FILTER_VALIDATE_BOOLEAN);
    }
}
```

**Key design choices:** Single path constant (no typos), fail-closed default, `filter_var` for type coercion, leverages the native `$defaultValue` parameter of `DeploymentConfig::get()`.

---

### Q7. Handling completely missing config

| Strategy | When to use | Example |
|---|---|---|
| **Fail-closed** (default off) | Security/destructive features | `maintenance/enabled` defaults to `false` |
| **Fail-open** (default on) | Non-critical features | `logging/enabled` defaults to `true` |
| **Fail-loud** (throw) | Mandatory infra config | `db/connection/host` — no sane default exists |

Magento uses fail-loud for DB availability via `isDbAvailable()`:

```php
public function isDbAvailable() {
    return $this->getConfigData('db') !== null;
}
```

And for install detection via `isAvailable()` — checks `install/date` is present.

**Best practice:** Log when falling back. For critical keys, validate at startup (not at first use).

---

## 4. Implementation-Level Considerations

### Q8. Hardcoded defaults vs mandatory config

| Use hardcoded default | Use mandatory config |
|---|---|
| Page size, timeout, retry count | DB host, API keys, crypt key |
| Safe in all environments | No sane default exists |
| `$config->get('graphql/depth', 20)` | `$config->get('crypt/key')` — must be set |

Magento example: `ConfigOptionsListConstants::CONFIG_PATH_CRYPT_KEY` (`crypt/key`) has no default — the installer generates it. But `x-frame-options` defaults to `SAMEORIGIN`.

---

### Q9. Unit testing config resolution

```php
/** @dataProvider configProvider */
public function testIsEnabled(?string $configValue, bool $expected): void
{
    $deploymentConfig = $this->createMock(DeploymentConfig::class);
    $deploymentConfig->method('get')
        ->with('feature/enabled', false)
        ->willReturn($configValue ?? false);

    $this->assertSame($expected, (new FeatureConfig($deploymentConfig))->isEnabled());
}

public static function configProvider(): array
{
    return [
        'present and true'  => ['1', true],
        'present and false' => ['0', false],
        'string true'       => ['true', true],
        'missing (default)' => [null, false],
        'empty string'      => ['', false],
    ];
}
```

**Scenarios covered:** present + truthy, present + falsy, missing (null), empty, string variants.

---

## 5. Cloud & Infrastructure Scenarios

### Q10. Normalizing multiple sources

Magento already implements this internally:

```
Config files (env.php, config.php)     ← Reader::load() merges files via array_replace_recursive
     ↓
Constructor overrides ($overrideData)  ← injected at DI level
     ↓
JSON env override (MAGENTO_DC__OVERRIDE) ← getEnvOverride()
     ↓
Individual env vars (MAGENTO_DC_*)     ← getAllEnvOverrides()
     ↓
DeploymentConfig::get($key, $default)  ← single unified interface
```

Additionally, Magento supports `#env(VAR_NAME, "default")` syntax inside config values:

```php
// In env.php
'db' => ['connection' => ['default' => ['host' => '#env(DB_HOST, "localhost")']]]
```

The `flattenParams()` method resolves this at load time via regex: `~^#env\(\s*(?<name>\w+)\s*(,\s*"(?<default>[^"]+)")?\)$~`

**Use Case:** DB host from ConfigMap file, DB password from Vault via `#env(DB_PASSWORD)` — all normalized behind `$config->get('db/connection/default/host')`.

---

### Q11. Same key in multiple sources — risks

| Risk | Mitigation |
|---|---|
| Silent override | Magento's `getAllEnvOverrides()` uses `+` operator — env vars win silently. Log conflicts at startup. |
| Stale env var | CI step to audit active env vars against config files |
| Pod inconsistency | Standardize env vars via Kubernetes ConfigMaps, not per-pod settings |

**Use Case:** `MAGENTO_DC_CACHE__BACKEND=redis` overrides `env.php`'s `cache/backend = memcached`. Without logging, ops won't know why the migration failed.

---

## 6. Security & Governance Scenario

### Q12. Security-sensitive feature — disabled by default

```php
public function isEnabled(): bool
{
    $value = $this->deploymentConfig->get('admin/remote_exec/enabled', false);
    if (filter_var($value, FILTER_VALIDATE_BOOLEAN) && $this->isProduction()) {
        $this->logger->critical('Remote exec enabled in PRODUCTION — refusing to activate.');
        return false;
    }
    return filter_var($value, FILTER_VALIDATE_BOOLEAN);
}
```

**Controls:** Security team owns the policy. Pipeline enforces four-eyes review. Code guardrail blocks activation in production even if config says `true`.

---

### Q13. Preventing unsafe config changes in production

```
Layer 1: Startup    → ConfigValidator checks required keys (isAvailable(), isDbAvailable())
Layer 2: CI         → Schema validation of env.php before merge
Layer 3: CD         → Config diff flags removed keys
Layer 4: Runtime    → Health checks verify config sanity post-deploy
```

Magento already does Layer 1: `isAvailable()` checks `install/date`; `isDbAvailable()` checks the `db` key. If either returns `false`, the app enters setup/install mode.

---

## 7. Failure & Incident Scenario

### Q14. Outage from removed config key — prevention design

**Scenario:** Deploy removes `rate_limiting/enabled`. Code defaults to `false`. Traffic spike overwhelms the system.

**Solution — Config inventory audit at startup:**

```php
public function audit(DeploymentConfig $config): void
{
    $critical = ['rate_limiting/enabled', 'rate_limiting/rpm', 'db/connection/default/host'];
    foreach ($critical as $key) {
        if ($config->get($key) === null) {
            $this->metrics->increment('config.missing_key', ['key' => $key]);
            if ($this->isCritical($key)) {
                throw new MissingCriticalConfigException("$key must be set in production.");
            }
        }
    }
}
```

**Also:** CD pipeline diffs new config against running config. Removed critical keys block the deploy.

---

### Q15. Warn or fail on fallback in production?

| Severity | Behavior | Example |
|---|---|---|
| **Critical** | Fail startup | `db/connection/default/host`, `crypt/key` |
| **Important** | Warn + alert | `cache/frontend/default/backend`, retry counts |
| **Optional** | Debug log | Page size, UI preferences |

Magento's own pattern: `isAvailable()` and `isDbAvailable()` act as critical-key checks. Missing = system enters install mode (effectively a fail-loud).

---

## 8. Runtime vs Deployment Configuration

### Q16. Should deployment config be changeable at runtime?

**No.** Magento enforces this: `DeploymentConfig` has no setter. `resetData()` only clears the cache for a re-read — it does not mutate values.

| Concern | Magento's approach |
|---|---|
| Safety | Immutable public API — no `set()` method |
| Auditability | Values live in version-controlled `env.php` |
| Config vs state | Runtime changes go to `core_config_data` (DB), not `DeploymentConfig` |

**Exception:** Feature flags via dedicated services (LaunchDarkly, etc.) — separate system with its own audit trail.

---

### Q17. Documenting config expectations

Magento uses `ConfigOptionsListConstants` as the canonical reference:

```php
public const CONFIG_PATH_DB_CONNECTION_DEFAULT = 'db/connection/default';
public const CONFIG_PATH_CRYPT_KEY = 'crypt/key';
public const CONFIG_PATH_SESSION_SAVE = 'session/save';
public const CONFIG_PATH_CACHE_TYPES = 'cache_types';
```

**Best practice per key:** Type, default, required (yes/no), severity, override example, non-goals. Constants serve as self-documentation; pair with a markdown reference for ops.

---

## 9. Architectural Judgment

### Q18. Default-in-code + override vs. deployment-only

| Criteria | Default + Override | Deployment-only |
|---|---|---|
| **Maintainability** | High — defaults colocated with code | Low — config drifts from code |
| **Scalability** | New envs need minimal config | Every env needs full config |
| **Reliability** | Always boots | Missing key = down |
| **DX** | Clone → run | Clone → configure → run |

Magento's choice: **Default-in-code + override.** `DeploymentConfig::get($key, $defaultValue)` has the `$defaultValue` parameter for exactly this purpose. Infrastructure keys (`db/host`, `crypt/key`) have no default and are validated separately.

---

## 10. Architecture Decision Record

### Q19. ADR: Defaults in Code, Overrides in Deployment

```
# ADR-042: Config Strategy — Defaults in Code, Overrides in Deployment

## Status: Accepted

## Context
All config was in deployment files → bootstrap failures, onboarding friction, production incidents.

## Decision
1. Every config key has a code-level default via DeploymentConfig::get($key, $default).
2. Deployment config (env.php, MAGENTO_DC_* env vars) overrides defaults.
3. Critical keys (db/host, crypt/key) have no safe default → fail-loud if missing.

## Precedence
MAGENTO_DC_* env vars > MAGENTO_DC__OVERRIDE JSON > $overrideData > env.php > code default

## Consequences
(+) Reliable startup, zero-config local dev, self-documented defaults
(-) Poor defaults can mask missing production config (mitigate: logging + alerts)

## Guardrails
- isAvailable() / isDbAvailable() validate critical keys at startup
- ConfigOptionsListConstants documents all known paths
- CD pipeline diffs config and flags removed keys
```

---

## Complete Guide: All Ways to Override Configuration in Magento 2

Magento 2 has **multiple layers** of configuration, each with its own override mechanism. They fall into two broad categories: **deployment config** (file-based, immutable at runtime) and **system config** (DB-based, mutable via admin).

---

### Layer 1: Config Files — `env.php` and `config.php`

The foundation. `DeploymentConfig\Reader::load()` merges both files via `array_replace_recursive`.

| File | Pool Constant | Purpose | Version-controlled? |
|---|---|---|---|
| `app/etc/config.php` | `ConfigFilePool::APP_CONFIG` | Shared config: modules, themes, scopes, i18n | Yes |
| `app/etc/env.php` | `ConfigFilePool::APP_ENV` | Environment-specific: DB, cache, crypt key, queue | No (sensitive) |

**How it works:**

```php
// Reader::load() merges all pool files
foreach ($configFiles as $file) {
    $result = array_replace_recursive($result, include $configFile);
}
```

**Override method:** Edit the file directly or use CLI commands (see below).

**Use Case:** `config.php` holds module list and shared system config dumped by `app:config:dump`. `env.php` holds DB credentials and cache backend. Deploying to staging only swaps `env.php`.

---

### Layer 2: Constructor `$overrideData` Injection

`DeploymentConfig` accepts `$overrideData` in its constructor, merged on top of file data.

```php
public function __construct(DeploymentConfig\Reader $reader, $overrideData = [])
```

Used internally by the setup/install process to inject config before files are written.

**Override method:** DI configuration (`di.xml`) or programmatic instantiation.

---

### Layer 3: `MAGENTO_DC__OVERRIDE` JSON Environment Variable

A single env var containing a full JSON config blob, merged on top of file + constructor data.

```php
// DeploymentConfig::getEnvOverride()
$env = getenv('MAGENTO_DC__OVERRIDE');
return !empty($env) ? (json_decode($env, true) ?? []) : [];

// Applied in reloadInitialData():
$this->data = array_replace($this->readerLoad, $this->overrideData, $this->getEnvOverride());
```

**Override method:**

```bash
export MAGENTO_DC__OVERRIDE='{"db":{"connection":{"default":{"host":"staging-db.internal"}}}}'
```

**Use Case:** Override multiple nested keys in a single env var. Useful for Docker Compose or ECS task definitions where managing many individual env vars is cumbersome.

---

### Layer 4: `MAGENTO_DC_*` Individual Environment Variables (Highest Priority)

Individual env vars with the `MAGENTO_DC_` prefix. Double underscores (`__`) map to path separators (`/`).

```php
// DeploymentConfig::getAllEnvOverrides()
// MAGENTO_DC_DB__CONNECTION__DEFAULT__HOST → db/connection/default/host
$flatKey = strtolower(str_replace([self::MAGENTO_ENV_PREFIX, '__'], ['', '/'], $key));

// Applied LAST — wins over everything:
$this->flatData = $this->getAllEnvOverrides() + $this->flatData;
```

**Override method:**

```bash
export MAGENTO_DC_DB__CONNECTION__DEFAULT__HOST=replica-db.internal
export MAGENTO_DC_CACHE__FRONTEND__DEFAULT__BACKEND=Magento\\Cache\\Backend\\Redis
```

**Use Case:** Per-pod override in Kubernetes. Override DB host for a canary pod without touching config files.

---

### Layer 5: `#env(VAR, "default")` Inline Syntax in Config Values

Values inside `env.php` / `config.php` can reference OS-level env vars using a special syntax.

```php
// Resolved in DeploymentConfig::flattenParams() via regex:
// ~^#env\(\s*(?<name>\w+)\s*(,\s*"(?<default>[^"]+)")?\)$~

// Example in env.php:
'db' => ['connection' => ['default' => ['host' => '#env(DB_HOST, "localhost")']]]
```

The value is resolved at config load time: reads `DB_HOST` env var, falls back to `"localhost"`.

**Override method:** Set the referenced env var:

```bash
export DB_HOST=production-db.rds.amazonaws.com
```

**Use Case:** Keep `env.php` in version control with placeholder references. Each environment only needs to set the referenced env vars. Useful for Docker images with a baked-in `env.php`.

---

### Layer 6: `setup:config:set` CLI Command

Writes directly to `env.php` via `DeploymentConfig\Writer::saveConfig()`. Used for infrastructure-level deployment config.

```
bin/magento setup:config:set --db-host=newhost --db-name=newdb --cache-backend=redis
```

**Source:** `Magento\Setup\Console\Command\ConfigSetCommand`

```php
// Uses DeploymentConfig\Writer::saveConfig() internally
// Prompts for confirmation when overwriting existing values:
$needOverwrite = ($currentValue !== null) && ($inputOptions[$option->getName()] !== null)
    && ($inputOptions[$option->getName()] !== $currentValue);
```

**What it sets:** DB connection, session save, cache backend, amqp, crypt key — all `ConfigOptionsListConstants` paths.

**Use Case:** Automated provisioning scripts. `setup:config:set --db-host=db.internal --cache-backend=redis` during infrastructure setup.

---

### Layer 7: `config:set` CLI Command (with Lock)

Sets **system configuration** values (the `system` config type — paths like `section/group/field`). Can write to either the database OR lock the value into config files.

```
bin/magento config:set web/unsecure/base_url http://example.com/

# Lock to env.php (cannot be changed in Admin):
bin/magento config:set --lock-env web/unsecure/base_url http://example.com/

# Lock to config.php (shared + locked):
bin/magento config:set --lock-config web/unsecure/base_url http://example.com/
```

**Source:** `Magento\Config\Console\Command\ConfigSetCommand`

**Lock mechanism** (`LockProcessor`):

```php
// Writes to env.php (--lock-env) or config.php (--lock-config):
$this->deploymentConfigWriter->saveConfig(
    [$this->target => $this->arrayManager->set($configPath, [], $value)]
);
```

The locked value goes under `system/default/web/unsecure/base_url` inside the config file. Once locked, the Admin panel shows the value as read-only.

| Flag | Target file | Shared across envs? | Editable in Admin? |
|---|---|---|---|
| (none) | `core_config_data` DB table | No | Yes |
| `--lock-env` | `env.php` | No | No |
| `--lock-config` | `config.php` | Yes | No |

**Use Case:** Lock the base URL in `env.php` so it cannot be accidentally changed via the Admin panel. `bin/magento config:set --lock-env web/unsecure/base_url https://prod.example.com/`

---

### Layer 8: `config:sensitive:set` CLI Command

Sets sensitive/environment-specific system config values. Always writes to `env.php`.

```
bin/magento config:sensitive:set payment/paypal/api_key SECRETKEY123
```

**Source:** `Magento\Deploy\Console\Command\App\SensitiveConfigSetCommand`

Values are stored under the `system` key in `env.php`, same as `--lock-env`. The command also regenerates the config hash via `Hash::regenerate()`.

**Use Case:** Set payment gateway API keys, SMTP passwords, and other secrets during deployment without exposing them in `config.php` (which is version-controlled).

---

### Layer 9: `app:config:dump` CLI Command

Dumps the current system configuration from the database into config files.

```
bin/magento app:config:dump
bin/magento app:config:dump scopes themes i18n
```

**Source:** `Magento\Deploy\Console\Command\App\ApplicationDumpCommand`

```php
// Uses Writer::saveConfig() with override=true:
$this->writer->saveConfig([$pool => $dump], true, null, $comments);
```

Dumps non-sensitive values into `config.php` and sensitive values into `env.php` (based on pool assignment). Regenerates the config hash afterward.

**Use Case:** Pipeline deployment workflow:
1. Developer configures themes/scopes in Admin on a build instance
2. Runs `app:config:dump` to capture config into `config.php`
3. Commits `config.php` to version control
4. Production deploys and runs `app:config:import` to apply

---

### Layer 10: `app:config:import` CLI Command

Imports configuration from shared config files (`config.php`) into the database.

```
bin/magento app:config:import
```

**Source:** `Magento\Deploy\Console\Command\App\ConfigImportCommand`

Detects changes by comparing a config hash. If `config.php` has changed since the last import, it writes the new values into the database (themes, scopes, system config).

**Use Case:** Production server receives `config.php` via git pull. Running `app:config:import` applies the new theme/scope configuration without needing Admin access.

---

### Layer 11: Module-Level Defaults via `config.xml`

Every module can define default system configuration values in `etc/config.xml`:

```xml
<!-- app/code/Magento/Catalog/etc/config.xml -->
<config>
    <default>
        <catalog>
            <frontend>
                <grid_per_page>12</grid_per_page>
                <list_per_page>10</list_per_page>
                <flat_catalog_product>0</flat_catalog_product>
            </frontend>
        </catalog>
    </default>
</config>
```

These are the **lowest-priority** system config defaults. Overridden by `core_config_data` (Admin panel), which is overridden by locked config in `config.php`/`env.php`.

**Use Case:** A module ships with `catalog/frontend/grid_per_page = 12`. The merchant changes it to `24` in Admin (written to `core_config_data`). If the DB row is deleted, the system falls back to `12` from `config.xml`.

---

### Layer 12: Admin Panel → `core_config_data` Table

The Admin panel (`Stores > Configuration`) writes to the `core_config_data` database table. Read via `ScopeConfigInterface::getValue()`.

```php
// ScopeConfigInterface — the runtime config reader
$value = $scopeConfig->getValue('catalog/frontend/grid_per_page', 'store', 'default');
```

These are **runtime** (not deployment) configuration values. They support scoping: default, website, store view.

**Use Case:** Merchant changes store name, tax rules, shipping settings via the Admin panel. Values persist in the database and can differ per store view.

---

### Layer 13: `DeploymentConfig::get()` `$defaultValue` Parameter

The final fallback. Code-level default passed at the call site.

```php
$maxDepth = $this->deploymentConfig->get('graphql/query_depth_limit', 20);
```

If no file, env var, or override provides the value, `20` is returned.

---

### Complete Precedence Chain (Deployment Config)

```
MAGENTO_DC_* individual env vars              ← HIGHEST (Layer 4)
  ↓ overrides
MAGENTO_DC__OVERRIDE JSON env var             ← (Layer 3)
  ↓ overrides
Constructor $overrideData                     ← (Layer 2)
  ↓ overrides
env.php + config.php (merged by Reader)       ← (Layer 1)
  ↓ falls back to
get($key, $defaultValue) code default         ← LOWEST (Layer 13)
```

### Complete Precedence Chain (System Config — `ScopeConfigInterface`)

```
Locked in env.php (--lock-env / config:sensitive:set)    ← HIGHEST
  ↓ overrides
Locked in config.php (--lock-config / app:config:dump)
  ↓ overrides
core_config_data DB table (Admin panel / config:set)
  ↓ falls back to
Module etc/config.xml <default> values                   ← LOWEST
```

---

### CLI Commands Quick Reference

| Command | What it does | Target |
|---|---|---|
| `setup:config:set` | Set deployment config (DB, cache, etc.) | `env.php` |
| `config:set` | Set system config in DB | `core_config_data` |
| `config:set --lock-env` | Set + lock system config | `env.php` |
| `config:set --lock-config` | Set + lock + share system config | `config.php` |
| `config:sensitive:set` | Set sensitive system config | `env.php` |
| `config:show` | Read resolved system config | (read-only) |
| `app:config:dump` | Dump DB system config to files | `config.php` / `env.php` |
| `app:config:import` | Import file config into DB | `core_config_data` |

---

### `DeploymentConfig` API Quick Reference

| Method | Purpose |
|---|---|
| `get($key, $defaultValue)` | Read flat config with fallback. Core method for all config reads. |
| `getConfigData($key)` | Read non-flattened (nested) config data. |
| `isAvailable()` | Check if `install/date` is set — system is installed. |
| `isDbAvailable()` | Check if `db` config exists. |
| `resetData()` | Clear cached config (forces re-read on next `get()`). |

| Internal Mechanism | What it does |
|---|---|
| `Reader::load()` | Merges `env.php` + `config.php` via `array_replace_recursive` |
| `Writer::saveConfig()` | Writes config arrays to `env.php` / `config.php`, invalidates opcache |
| `getAllEnvOverrides()` | Reads `MAGENTO_DC_*` env vars, converts `__` to `/` for flat keys |
| `getEnvOverride()` | Reads `MAGENTO_DC__OVERRIDE` JSON env var |
| `flattenParams()` | Converts nested arrays to `path/key` format; resolves `#env()` syntax |
| `#env(VAR, "default")` | Inline env var reference inside config values |
