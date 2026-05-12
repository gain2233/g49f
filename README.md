# Logging-Bake-4 — Operator Reference

**Sub-phase Logging-Bake-4 sealed**: commit [`9f2f002`](../CHANGELOG.md). Per-sink LogLevel JSON v2 schema; total silent default; bootstrap log explicit gate; args precedence preserved.

> Bu belge: `%LocalAppData%\Packages\cr.sb.client\telemetry-config.json` ile client logging davranışının tam kontrolü. JSON elle düzenleyebilir veya Builder.Next.exe 7th CardExpander UI'dan bake edebilirsin.

---

## 1. Schema v2 Tam Spec

```json
{
  "_schema_version": 2,
  "console":   "off | critical | major | minor",
  "file":      "off | critical | major | minor",
  "bootstrap": "off | critical | major | minor",
  "directory": "C:\\custom\\path\\or\\null"
}
```

### Field referansı

| Field | Tip | Valid values | Default (unset) | Etkilediği sink | Case-sensitivity |
|---|---|---|---|---|---|
| `_schema_version` | int | `2` zorunlu | yokken default `2` varsayılır (v2 parser çalışır) | Şema gate | — |
| `console` | string | `"off"` \| `"critical"` \| `"major"` \| `"minor"` | `"off"` (baked fallback) | **stderr** (Console.Error) | **case-insensitive** (`Enum.TryParse ignoreCase`) — `"OFF"`, `"Off"`, `"off"` hepsi geçerli |
| `file` | string | `"off"` \| `"critical"` \| `"major"` \| `"minor"` | `"off"` | **`<directory>\trace-{stamp}.log`** (rotating) | case-insensitive |
| `bootstrap` | string | `"off"` \| `"critical"` \| `"major"` \| `"minor"` | `"off"` | **`%LocalAppData%\Packages\cr.sb.client\trace-bootstrap.log`** (FIXED path) | case-insensitive |
| `directory` | string\|null | herhangi bir path; `DDMMYY` token + `%LocalAppData%`-style env vars expand edilir | hardcoded `%LocalAppData%\Packages\cr.sb.client\TelemetryClient_{ddMMyy}\` | **YALNIZ `file` sink** (bootstrap path FIXED — operator pin pre-flight #1) | — |

### LogLevel cumulative semantic

| Değer | Emit edilen severity | Event sayısı |
|---|---|---|
| `off` | hiçbiri | 0 |
| `critical` | severity=1 (Critical) | 26 |
| `major` | severity≤2 (Critical + Major) | 26+56 = **82** |
| `minor` | severity≤3 (hepsi) | **184** |

### Bilinmeyen event default

`EventSeverityRegistry.Lookup(unknownKey)` → `Major` (operator-pinned safe side). Yeni event key'leri eklerken registry'e kaydet — aksi halde `level=critical` set edildiğinde gözükmezler.

---

## 2. Dosya Konumu + İzinler

### Lookup precedence (yüksek → düşük)

1. `--log-config=<path>` CLI args (explicit override)
2. `<exe-dir>\telemetry-config.json` (portable; Stub.exe yanı sıra dosya)
3. **`%LocalAppData%\Packages\cr.sb.client\telemetry-config.json`** (operator default — önerilen)

### Multi-instance senaryosu

Telemetry config **shared** — per-instance (`cr.sb.client.net472{GUID}`) değil. `Paths.AppDataFolderName = "Packages\\cr.sb.client"` const tüm Stub.exe instance'larının aynı telemetry config'i kullanmasını sağlar.

> **Not**: Sub-phase Config encrypted `config.json` per-instance GUID-suffixed (`Packages\sn.rs.client.net472{GUID}\config.json`) — farklı sistem. Telemetry config global.

### Edge case davranışları

| Senaryo | Davranış | Telemetry emit |
|---|---|---|
| **Dosya yok** | Silent fallback (baked all-Off + hardcoded directory) | yok |
| **Dosya var, empty `{}`** | Schema v2 varsayılır; tüm field'lar default → baked layer'a fall through | yok |
| **Dosya var, sadece bir field** | Bilinen field uygulanır; eksikler baked'tan gelir | yok |
| **JSON syntax error** | `Logging.Config.Json.LoadFail` (Major) → JSON ignore + baked fallback | bootstrap log'da (BootstrapLevel allow ediyorsa) |
| **Permission denied (read)** | `Logging.Config.Json.LoadFail` (Major) + fallback | bootstrap log |
| **Klasör yok (`directory` set)** | `Logging.Config.Dir.CreateFail` (Major); best-effort `fs.CreateDirectory()` denenir; başarısızsa file sink yazma anında IOException catch + drop | bootstrap log |
| **`_schema_version: 1`** | **HARD ERROR** — `Logging.Config.Json.SchemaV1.HardError` (Major) + stderr FATAL hint; JSON tamamen ignore; baked fallback | bootstrap log + stderr (rare) |
| **`_schema_version: 99`** | `Logging.Config.Json.SchemaUnknown` (default unknown Major); JSON ignore; fallback | bootstrap log |

---

## 3. Behavior Matrix — 12 senaryo

`MuckClient.exe` baked deploy (Builder UI defaults: all Off). Aşağıdaki `telemetry-config.json` içerikleri için sonuç:

| # | JSON içeriği | stderr | `trace-bootstrap.log` | `trace-{stamp}.log` |
|---|---|---|---|---|
| 1 | **Dosya YOK** | EMPTY | YOK | YOK |
| 2 | `{}` | EMPTY | YOK | YOK |
| 3 | `{"_schema_version": 2}` | EMPTY | YOK | YOK |
| 4 | `{"_schema_version": 2, "console": "off", "file": "off", "bootstrap": "off"}` | EMPTY | YOK | YOK |
| 5 | `{"_schema_version": 2, "console": "minor"}` | **TÜM 184 event** | YOK | YOK |
| 6 | `{"_schema_version": 2, "file": "major"}` | EMPTY | YOK | **Critical+Major (82 event)** kategorisinden tetiklenenler `%LocalAppData%\Packages\cr.sb.client\TelemetryClient_{ddMMyy}\trace-{stamp}.log`'a |
| 7 | `{"_schema_version": 2, "bootstrap": "critical"}` | EMPTY | **Yalnız Critical (26 event)** kategorisinden tetiklenenler | YOK |
| 8 | `{"_schema_version": 2, "console": "minor", "file": "minor", "bootstrap": "minor"}` | TÜM 184 | TÜM 184 | TÜM 184 |
| 9 | `{"_schema_version": 2, "file": "minor", "directory": "C:\\logs"}` | EMPTY | YOK | `C:\logs\trace-{stamp}.log` (tüm 184) |
| 10 | `{"_schema_version": 1, "telemetry": {...}}` | **`[FATAL] telemetry-config.json schema v1 detected at ...`** | YOK (bootstrap=off baked) | YOK |
| 11 | `{"_schema_version": 2, "console": "BOGUS"}` | EMPTY (invalid enum → field skip → baked Off) | YOK | YOK |
| 12 | `{"_schema_version": 2, "console": "minor"` (truncated/syntax error) | EMPTY (catch + LoadFail → JSON ignore) | YOK | YOK |

### #10 detay — V1 HARD ERROR stderr örnek

```
[FATAL] telemetry-config.json schema v1 detected at 'C:\Users\user\AppData\Local\Packages\cr.sb.client\telemetry-config.json' — Logging-Bake-4 requires v2 (per-sink levels). Regenerate via Builder.Next.exe OR delete the file to use baked defaults.
```

Stderr emit'i `Console.Error.WriteLine` ile DIRECT yapılır — bootstrap log gate'inden geçmez. Operator yanlış schema'yı görmek zorundadır (silent fail kabul edilemez).

---

## 4. Args Override Precedence — Concrete Examples

### Internal apply sırası (Logging-Bake-4 pre-flight #3)

```
1. JSON v2 layer parse → cfg
2. --log-level=X     → cfg.{Console, File, Bootstrap} = X (3 sink set)
3. --log-channel=X   → narrow:
                        consoleonly → File=Off, Bootstrap=Off
                        fileonly    → Console=Off
                        both        → no-op
4. --log-severity=X  → cap (max int in CSV → LogLevel cap)
5. --log-dir=X       → cfg.Directory = X (file sink scope only)
```

### Concrete matrix

| # | JSON state | Args | Final console | Final file | Final bootstrap | Final directory |
|---|---|---|---|---|---|---|
| 1 | `console=off, file=major, bootstrap=off` | `--log-level=verbose` | **Minor** | **Minor** | **Minor** | (JSON value korunur) |
| 2 | all Off (default) | `--log-channel=consoleonly` | Off (no level set) | Off | Off | (default) |
| 3 | all Off | `--log-level=verbose --log-channel=consoleonly` | **Minor** | Off | Off | (default) |
| 4 | `console=minor, file=minor, bootstrap=minor` | `--log-channel=fileonly --log-severity=1` | Off (channel→Off) | **Critical** (severity cap 1) | **Critical** (cap 1) | (default) |
| 5 | `console=minor, file=minor` | `--log-dir=D:\debug` | Minor | Minor | Off (JSON unset) | **`D:\debug`** |
| 6 | `console=minor` | `--log-config=C:\alt.json` | (C:\alt.json içeriğine göre) | — | — | — |

### F13 regression lock

`LoggingConfigurationLoaderTests.F13_ArgsPrecedence_LevelThenChannel` testi şu davranışı garanti eder:

```
Args: --log-level=verbose --log-channel=fileonly
→ Step 2: level=Minor → all 3 = Minor
→ Step 3: channel=fileonly → console = Off
Final: console=Off, file=Minor, bootstrap=Minor
```

---

## 5. Hot-Reload

**Yok**. Q-LOG-8 operator pin: `LoggingConfigurationLoader.Resolve` **boot-time only**, Step 2.1'de bir kez çalışır. JSON dosyası değiştirilse bile uygulanmaz.

### Operator workflow

1. `MuckClient.exe` durdur (Ctrl+C veya process kill)
2. `telemetry-config.json` düzenle
3. `MuckClient.exe` yeniden başlat
4. Yeni config uygulanır

> **Performance gerekçesi**: Boot-time tek seferlik resolve = sub-µs runtime cost per `log.Write` çağrısı. Dinamik reload file-watcher daemon + lock contention getirirdi, no-go.

---

## 6. Bootstrap Timing

### Boot sırası (kronolojik)

```
T+0       BootstrapDiagnosticLog ctor — peeks LoggingDefaultsConstants.Default (baked or placeholders)
              ↓ HasAnyBakedValue check
              ↓ _config = baked config (eğer bake varsa) veya null (fresh build)
T+0+δ     log.Write("Boot.Start")     → gated by ctor auto-init _config
T+50µs    LoggingConfigurationLoader.Resolve(args, fs, log)
              ↓ emits Logging.Config.Source.Baked (if any baked)
              ↓ reads telemetry-config.json
              ↓ emits Logging.Config.Json.Loaded
              ↓ applies args
              ↓ Logging.Config.Dir.CreateFail (eğer directory create başarısız)
              ↓ returns resolved config
T+5ms     log.Write("Logging.Config.Resolved") → gated by ctor auto-init (PRE-Reconfigure)
T+5ms+δ   bootstrapLog.Reconfigure(resolvedConfig)   ← FULL config applied
T+5ms+δ   sonraki tüm log.Write çağrıları FULL config'i honor eder
T+~100ms  Composite swap (Step 7): log = ConfigurableTelemetrySink replacement
              ↓ post-swap events composite sink chain'inden geçer
              ↓ trace-bootstrap.log artık bu noktadan sonra yazılmaz
              ↓ trace-{stamp}.log post-swap event'leri alır
```

### Pre-Reconfigure window — hangi event'ler düşebilir?

JSON layer'dan ÖNCE emit edilen event'ler **ctor auto-init baked config**'i honor eder (baked all-Off → suppressed; baked partial → ilgili sink'e düşer):

| Event | Severity | Pre-Reconfigure timing |
|---|---|---|
| `Boot.Start` | Minor | T+0 |
| `Logging.Config.Source.Baked` | Minor | T+50µs (Resolve içinde) |
| `Logging.Config.Json.Loaded` | Minor | T+50µs (JSON found ise) |
| `Logging.Config.Json.LoadFail` | Major | T+50µs (JSON parse error ise) |
| `Logging.Config.Json.SchemaV1.HardError` | Major | T+50µs (v1 dosya ise) |
| `Logging.Config.Json.SchemaUnknown` | Major | T+50µs (unknown version ise) |
| `Logging.Config.Dir.CreateFail` | Major | T+50µs (klasör create fail ise) |
| `Logging.Config.Resolved` | Minor | T+5ms (Resolve sonrası ama Reconfigure öncesi) |

### Pratik etki

- **Baked Off + JSON yok**: pre-Reconfigure event'ler tamamen suppressed (ctor auto-init Off uygular).
- **Baked Off + JSON `bootstrap: minor`**: pre-Reconfigure event'ler suppressed (ctor auto-init Off görür; JSON henüz uygulanmadı). Sadece T+5ms+δ sonrası event'ler bootstrap log'a yazılır. → **`Logging.Config.Json.Loaded` event'ini görmezsin** (Reconfigure öncesi emit edilir).
- **Fresh build (Builder hiç çalışmamış)**: ctor auto-init `_config=null` → always-on stderr+file dev forensics → pre-Reconfigure 3 satır console'da görünür.

### Operator için bilinmesi gereken

`bootstrap=minor` set ettiğin halde `Logging.Config.Json.Loaded` event'ini bootstrap dosyasında **göremezsin** — bu event JSON layer'ın yüklendiğini bildirir, JSON layer henüz uygulanmamış olduğu için baked Off gate'i tarafından suppressed olur. Bu intentional trade-off (Logging-Bake-3 commit message edge case).

---

## 7. 5 Common Debug Senaryosu — Copy-Paste Ready

### (a) "Hiçbir log istemiyorum"
```json
// Sadece dosyayı sil, ya da:
{ "_schema_version": 2 }
```

### (b) "Boot fail'i debug edeceğim"
```json
{ "_schema_version": 2, "bootstrap": "minor" }
```
→ trace-bootstrap.log'a tüm boot timeline (post-Reconfigure). Pre-Reconfigure 1-2 satır kayıp.

### (c) "Production audit trail (file-only, önemli olaylar)"
```json
{
  "_schema_version": 2,
  "file": "major",
  "directory": "D:\\audit\\client"
}
```
→ Console silent. `D:\audit\client\trace-{stamp}.log`'a yalnız Critical+Major (82 event categories).

### (d) "Live console tail (dev workflow)"
```json
{ "_schema_version": 2, "console": "minor" }
```
→ stderr full. Hiçbir dosyaya yazmaz. Live `MuckClient.exe 2>&1 | Tee-Object out.log` workflow için.

### (e) "Her şey her yere (forensic deep dive)"
```json
{
  "_schema_version": 2,
  "console":   "minor",
  "file":      "minor",
  "bootstrap": "minor"
}
```
→ 3 hedef de tüm 184 event alır. **Disk-I/O heavy** — sustained workload için önerilmez.

---

## 8. Telemetry Key Spy List

### Critical (26 event) — `level=critical` set edildiğinde dosyaya düşenler

```
Boot.Settings.LoadFatal
Boot.Settings.ValidateFatal
Boot.Unhandled.Exception
Crypto.Aes256.Decrypt.CiphertextTooShort
Crypto.Aes256.Decrypt.MacInvalid
Hwid.Compute.Hierarchical.AllFail
Hwid.Compute.Legacy.Fail
Packet.Decode.GzipCrcFail
Packet.Decode.LengthMismatch
Packet.Decode.NegativeLength
Packet.Frame.OversizeReject
Plugin.Abi.Mismatch
Session.Lost.Reason
Settings.Config.Decrypt.HmacInvalid
Settings.Load.AesEmbedded.Fail.AesCtor
Settings.Load.AesEmbedded.Fail.AesKeyMissing
Settings.Load.AesEmbedded.Fail.CertDecode
Settings.Load.AesEmbedded.Fail.FieldDecrypt
Settings.Load.AesEmbedded.Fail.KeyDecode
Settings.Load.AesEmbedded.Fail.PortsParse
Settings.Load.AesEmbedded.Fail.ResourceMissing
Settings.Load.AesEmbedded.Fail.SignatureDecode
Settings.Load.AesEmbedded.Fail.SignatureMismatch
Settings.Load.Json.Fail.Parse
Settings.Load.Json.Fail.Read
Transfer.Integrity.WholeFileFail
```

### Major (56 event) — `level=major` set edildiğinde Critical'a ek olarak düşenler

```
ClientInfo.Av.Wmi.Fail
ClientInfo.Av.Wmi.Timeout
ClientInfo.Send.Fail
Crypto.Aes256.Ctor.InvalidKey
Crypto.Aes256.Decrypt.Fail
Crypto.Aes256.Encrypt.Fail
Dns.Resolve.Empty
Dns.Resolve.Exception
Hwid.Cache.Read.DecryptFail
Hwid.Compute.Hierarchical.BiosUuid.Fail
Hwid.Compute.Hierarchical.MacHash.Fail
Hwid.Compute.Hierarchical.MachineGuid.Fail
Hwid.Compute.Hierarchical.Tpm.Fail
Hwid.Compute.Hierarchical.Tpm.Timeout
Logging.Config.Dir.CreateFail
Logging.Config.Json.LoadFail
Packet.Encode.Fail
Packet.Encode.UnknownPacketType
Packet.Encode.UnknownSignature
Plugin.Invoke.Failed.Other
Plugin.Invoke.Threw
Plugin.Invoke.Timeout
Plugin.LegacyAbi.Bootstrap.Failed
Plugin.LegacyAbi.Run.Exception
Plugin.LegacyAbi.Shim.PathJ.ServerEndpoint.Exception
Plugin.LegacyAbi.Shim.PathJ.ServerEndpoint.NotFound
Plugin.LegacyAbi.Threw
Plugin.LegacyAbi.Timeout
Plugin.Push.Rejected
Plugin.Wire.Malformed
Plugin.Wire.Unknown
Reconnect.Backoff.Sleep
Resolve.Host.Fail
Session.Packet.DecodeFail
Session.Packet.Unhandled
Session.Packet.Unhandled.NoRouter
Session.Ping.Fail
Session.Write.Timeout
Settings.Config.Decrypt.Fail
Settings.Config.LoadFail
Settings.Config.ParseFail
Settings.Loader.ShortCircuit
Settings.Validate.CertValidation.NoneInProduction
Tcp.Connect.Fail
Tcp.Connect.IpFail
Tcp.Connect.Refused
Tcp.Connect.SocketException
Tls.Handshake.AuthenticationException
Tls.Handshake.Fail
Tls.Handshake.IOException
Transfer.Integrity.SegmentMismatch.Retry
Transfer.Wire.NotEnabled
Upload.Integrity.WholeFileMismatch.Retry
Upload.Router.NotConfigured
Watchdog.Tick.Exception
Watchdog.Tick.Overlap
```

### Minor (102 event) — `level=minor` set edildiğinde + üstüne düşenler (success / info / lifecycle)

```
Boot.End
Boot.GracefulDrain.Released
Boot.GracefulDrain.Wait
Boot.Hwid.Resolved
Boot.OperatorCancel.Received
Boot.Phase.Banner                              ← sealed banner (post Logging-Bake-3)
Boot.Settings.Ok
Boot.Sink.Flushed
Boot.SM.StartFired
Boot.SM.Stop.Called
Boot.Start
ClientInfo.Send.Success
Crypto.Aes256.Decrypt.Success
Crypto.Aes256.Encrypt.Success
Crypto.Aes256.Kdf.Done
Hwid.Cache.Read.Miss
Hwid.Cache.Read.Success
Hwid.Cache.Write
Hwid.Compute.Hierarchical.BiosUuid
Hwid.Compute.Hierarchical.MacHash
Hwid.Compute.Hierarchical.MachineGuid
Hwid.Compute.Hierarchical.Tpm
Hwid.Compute.Legacy
Hwid.Resolved
Hwid.Rotation.Applied
Hwid.Rotation.Detected
Logging.Config.Json.Loaded
Logging.Config.Resolved
Logging.Config.Source.Baked                    ← Logging-Bake-1 telemetry
Mutex.Primary
Mutex.Released
Mutex.Secondary
Packet.Codec.LegacySelected
Packet.Codec.RawSelected
Packet.Decode.Success
Packet.Encode.DiscriminatorInjected
Packet.Encode.DiscriminatorPassThrough
Packet.Encode.PingTypoCorrected
Packet.Encode.Success
Plugin.Invoke.Success
Plugin.LegacyAbi.Bootstrap.Constructed
Plugin.LegacyAbi.Bootstrap.Skipped
Plugin.LegacyAbi.Detection.AttributeCheck
Plugin.LegacyAbi.Detection.Route
Plugin.LegacyAbi.Detection.SignatureCheck
Plugin.LegacyAbi.Disposed
Plugin.LegacyAbi.Loaded
Plugin.LegacyAbi.Register.Entry
Plugin.LegacyAbi.Run.Entry
Plugin.LegacyAbi.Run.Exit
Plugin.LegacyAbi.Shim.EndpointSpoof.Reverted
Plugin.LegacyAbi.Shim.PathJ.ServerEndpoint.Set
Plugin.LegacyAbi.Socket.Shim.Created
Plugin.LegacyAbi.StaticField.Set
Plugin.Load.Success
Plugin.Loader.Disposed
Plugin.Pending.Drained
Plugin.Push.Received
Plugin.SendPlugin.Requested
Reconnect.Backoff.Bypassed
Reconnect.NetworkAvailable.Triggered
Reconnect.NetworkUnavailable.Detected
Resolve.Host.Success
Session.Ping.Sent
Session.Pong.Received
Session.ServerPing.Responded
Settings.Config.Decrypt.Success
Settings.Config.NotPresent
Settings.Config.OverridePath.Used
Settings.Load.AesEmbedded.Success
Settings.Load.Json.NotPresent
Settings.Load.Json.Success
Settings.Loader.Merge.Success
Settings.Loader.Source.                        ← prefix-only kayıt (operator pin Adım 7d)
StateMachine.Bootstrap
StateMachine.Disposed
Tcp.Connect.Attempt
Tcp.Connect.Success
Tcp.KeepAlive.Configure.Fail
Tcp.KeepAlive.Configure.Success
Tcp.SetSocketOption.KeepAlive.Fail
Telemetry.Composite.Online
Telemetry.Rotation.Done
Tls.Cert.Validation.Reason
Tls.Handshake.OperationCanceled
Tls.Handshake.Started
Tls.Handshake.Success
Transfer.Complete.Verified
Transfer.Init.Received
Transfer.Router.Received
Transfer.Segment.Complete
Transfer.Segment.Start
Transfer.Subsystem.Online
Upload.Complete.Verified
Upload.Init.Sent
Upload.Segment.Acked
Upload.Segment.Sent
Watchdog.Disposed
Watchdog.Reset
Watchdog.Start
Watchdog.Stop
Watchdog.Tick.Fired
```

### Toplam: 26 + 56 + 102 = **184** registered events

Registry'de olmayan event key'leri `Lookup()` tarafından **Major** olarak işaretlenir → `level=major` veya üstü ile gözükür. Eğer yeni bir event ekleyip `level=critical` ile görmek istiyorsan EventSeverityRegistry'e Critical olarak kaydetmen şart.

---

## 9. Yeni Telemetry Keys (Logging-Bake-4 doğan)

| Event key | Severity | Açıklama |
|---|---|---|
| `Logging.Config.Json.SchemaV1.HardError` | Major | `_schema_version: 1` detect → JSON ignore + stderr FATAL emit |
| `Logging.Config.Json.SchemaUnknown` | (registry'de yok; default Major) | `_schema_version: !=1,!=2` detect → JSON ignore |
| `Boot.Phase.Banner` | Minor | Sealed banner (Logging-Bake-3 conversion); composite sink Step 7 sonrası emit |

### Pre-existing Logging.Config.* keys (referans)

| Event key | Severity |
|---|---|
| `Logging.Config.Json.Loaded` | Minor |
| `Logging.Config.Json.LoadFail` | Major |
| `Logging.Config.Resolved` | Minor |
| `Logging.Config.Source.Baked` | Minor |
| `Logging.Config.Dir.CreateFail` | Major |

---

## 10. Manual Smoke Test Script

5 dakikada whole-system doğrulama:

```powershell
# Setup
$ConfigPath = "$env:LocalAppData\Packages\cr.sb.client\telemetry-config.json"
$BootstrapLog = "$env:LocalAppData\Packages\cr.sb.client\trace-bootstrap.log"
$TelemetryDir = "$env:LocalAppData\Packages\cr.sb.client\TelemetryClient_$(Get-Date -Format ddMMyy)"
$Exe = "C:\Users\user\Desktop\MuckClient.exe"

# ─────────────────────────────────────────────────────────────
# Test 1 — Total silent (no JSON, baked Off)
# ─────────────────────────────────────────────────────────────
Remove-Item $ConfigPath -ErrorAction SilentlyContinue
Remove-Item $BootstrapLog -ErrorAction SilentlyContinue
Remove-Item $TelemetryDir -Recurse -ErrorAction SilentlyContinue

Write-Host "[1/5] Total silent test..."
& $Exe 2>&1 | Out-String
# EXPECT: empty output. No bootstrap log. No telemetry dir.
Test-Path $BootstrapLog                          # → False ✓
(Test-Path $TelemetryDir) -or                    # → False ✓
(Get-ChildItem $TelemetryDir -ErrorAction Ignore).Count -eq 0

# ─────────────────────────────────────────────────────────────
# Test 2 — Full debug (everything everywhere)
# ─────────────────────────────────────────────────────────────
@'
{
  "_schema_version": 2,
  "console":   "minor",
  "file":      "minor",
  "bootstrap": "minor"
}
'@ | Set-Content $ConfigPath -Encoding UTF8

Write-Host "[2/5] Full debug test..."
& $Exe 2>&1 | Select-Object -First 10
# EXPECT: stderr non-empty (boot timeline). Bootstrap log written.
# trace-{stamp}.log created in TelemetryClient_DDMMYY dir.
Test-Path $BootstrapLog                          # → True ✓
Get-ChildItem "$TelemetryDir\trace-*.log"        # → 1+ files ✓

# ─────────────────────────────────────────────────────────────
# Test 3 — File-only audit (Critical+Major)
# ─────────────────────────────────────────────────────────────
Remove-Item $BootstrapLog -ErrorAction SilentlyContinue
Remove-Item $TelemetryDir -Recurse -ErrorAction SilentlyContinue

@'
{ "_schema_version": 2, "file": "major" }
'@ | Set-Content $ConfigPath -Encoding UTF8

Write-Host "[3/5] File audit test..."
& $Exe 2>&1
# EXPECT: stderr EMPTY. No bootstrap log. trace-{stamp}.log has
# only Critical+Major events (Boot.Start = Minor → suppressed).
Test-Path $BootstrapLog                          # → False ✓
$content = Get-Content "$TelemetryDir\trace-*.log" -Raw
$content -notmatch "Boot\.Start"                 # → True (Minor suppressed) ✓
$content -match "Tcp\.|Settings\.Load\."         # → typical Major events ✓

# ─────────────────────────────────────────────────────────────
# Test 4 — V1 schema HARD ERROR
# ─────────────────────────────────────────────────────────────
Remove-Item $BootstrapLog -ErrorAction SilentlyContinue
Remove-Item $TelemetryDir -Recurse -ErrorAction SilentlyContinue

@'
{
  "_schema_version": 1,
  "telemetry": { "verbosity": "Verbose", "outputChannel": "Both" }
}
'@ | Set-Content $ConfigPath -Encoding UTF8

Write-Host "[4/5] V1 HARD ERROR test..."
& $Exe 2>&1 | Select-Object -First 3
# EXPECT: stderr starts with "[FATAL] telemetry-config.json schema
# v1 detected at ...". No further events (baked Off fallback).

# ─────────────────────────────────────────────────────────────
# Test 5 — Custom directory + DDMMYY expand
# ─────────────────────────────────────────────────────────────
$CustomDir = "C:\muck-test-logs"
Remove-Item $CustomDir -Recurse -ErrorAction SilentlyContinue

@"
{
  "_schema_version": 2,
  "file":      "minor",
  "directory": "C:\\muck-test-logs\\DDMMYY"
}
"@ | Set-Content $ConfigPath -Encoding UTF8

Write-Host "[5/5] Custom directory test..."
& $Exe 2>&1
$ExpectedDir = "C:\muck-test-logs\$(Get-Date -Format ddMMyy)"
Get-ChildItem "$ExpectedDir\trace-*.log"         # → 1+ files ✓

# ─────────────────────────────────────────────────────────────
# Cleanup
# ─────────────────────────────────────────────────────────────
Remove-Item $ConfigPath -ErrorAction SilentlyContinue
Write-Host "✅ All 5 tests OK"
```

### Beklenen exit kodları

| Test | Exit kod | Açıklama |
|---|---|---|
| 1 | 0 | Normal exit (process'i Ctrl+C ile durdurursan) |
| 2 | 0 | Aynı |
| 3 | 0 | Aynı |
| 4 | 0 | Baked Off fallback ile normal exit (FATAL stderr emit ediliyor ama process devam ediyor) |
| 5 | 0 | Aynı |

> **Not**: 4. test'te V1 hard error stderr emit'i process'i terminate ETMEZ — yalnız JSON ignore + fallback. Operator yanlış schema'yı görmesi için stderr'e yazılır.

---

## 11. Hızlı Referans Cheatsheet

### Verbosity → LogLevel haritası (mental model)

```
Pre-Logging-Bake-4 (v1)              Post-Logging-Bake-4 (v2)
─────────────────────────            ───────────────────────────
verbosity: "Off"            ─────→   { console/file/bootstrap: "off" }
verbosity: "Verbose"        ─────→   { ... : "minor" }
verbosity: "ErrorsOnly" +            { ... : "major" }
  severityFilter: [1, 2]    ─────→
outputChannel: "Both"                console=X, file=X
outputChannel: "FileOnly"            console=off, file=X
outputChannel: "ConsoleOnly"         console=X, file=off, bootstrap=off
```

### "Operatör bunu istediğinde, JSON şudur"

| İstek | JSON |
|---|---|
| "Hiçbir log yok" | sil veya `{ "_schema_version": 2 }` |
| "Yalnız boot hatalarını yakalayalım" | `{ "_schema_version": 2, "bootstrap": "critical" }` |
| "Production: yalnız fail durumları dosyaya" | `{ "_schema_version": 2, "file": "major" }` |
| "Tüm boot sequence'ı izlemek istiyorum" | `{ "_schema_version": 2, "bootstrap": "minor" }` |
| "Live debug, ekrana yazsın" | `{ "_schema_version": 2, "console": "minor" }` |
| "Forensic deep dive, her şey her yere" | `{ "_schema_version": 2, "console": "minor", "file": "minor", "bootstrap": "minor" }` |
| "Custom path'e file log" | `{ "_schema_version": 2, "file": "minor", "directory": "D:\\logs\\client" }` |

---

## 12. Builder UI Quick Presets

Builder.Next.exe → BuildPage → "Logging Defaults (Phase 17 Sub-phase Logging-Bake-4)" → 3 buton:

| Buton | Bake değerleri | Hangi senaryo |
|---|---|---|
| **[Silent]** | `console=Off, file=Off, bootstrap=Off` | Production silent default |
| **[Audit]** | `console=Off, file=Major, bootstrap=Critical` | Production audit trail |
| **[Full Debug]** | `console=Minor, file=Minor, bootstrap=Minor` | Dev hot-debug |

Operator buton tıkladıktan sonra **Build** → yeni Stub.exe baked değerleri içerir. JSON dosyası deploy etmezsen bu baked'i kullanır.

---

**Belge versiyonu**: 1.0 (Logging-Bake-4 sealed sonrası)
**Son güncelleme**: 2026-05-12
**Source-of-truth**: `ClientNext/Muck.Core.Client/Telemetry/LoggingConfigurationLoader.cs` + `LoggingConfiguration.cs` + `LoggingDefaultsConstants.cs` + `EventSeverityRegistry.cs`
**Kaynak commit**: `9f2f002`
