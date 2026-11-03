# Tools

```bash
# Extract the CRIU checkpoint archive
tar -xf checkpoint-report-pod.tar

# Inspect CRIU log files
grep -i "inet" dump.log
grep -i "ghost" dump.log

# Extract strings and inspect process memory
strings checkpoint/pages-1.img
```

* **CRIU (Checkpoint and Restore in Userspace):** Kubernetes container checkpoint tool.
* **CyberChef / Base64 Decoder:** [CyberChef](https://gchq.github.io/CyberChef/)
* **JWT.io:** [JWT Debugger](https://jwt.io/)
* **Hex Editor / PowerShell:** For reading Protobuf structures and extracting compressed in-memory classes.

---

# Questions

## 1. The security team believes the attacker had hands-on access to the container at the time the incident was declared and IR was initiated. Finding the attacker's hidden network channel is important for attribution and identifying potential exfiltration. What is the IP address and port?

**File:** `dump.log` & `checkpoint/files.img`

In `dump.log`, CRIU collected the network sockets associated with the running Java process (PID 14898). An established outbound TCP socket (`TCP_ESTABLISHED`) was recorded under inode `0xdafd`:
```text
(00.017163) inet: Collected: ino 0xdafd family AF_INET6 type SOCK_STREAM port 38664 state TCP_ESTABLISHED src_addr ::ffff:10.42.0.9
(00.032136) inet: Dumping: ino 0xdafd family AF_INET6 type SOCK_STREAM port 38664 state TCP_ESTABLISHED src_addr ::ffff:10.42.0.9
```

Decoding the `inet_sk_entry` Protobuf message inside `checkpoint/files.img` for inode `0xdafd`:
* **`dst_port` (Field 8):** `0xEC 0x70` $\rightarrow$ `14444`
* **`dst_addr` (Field 12):** `0xB9 0xB9 0x97 0xFB 0x02` $\rightarrow$ `185.220.101.47` (Tor exit node)

**Answer:** `185.220.101.47:14444`

---

## 2. During disk inspection of the checkpoint, nothing malicious was observed. What is the full path of the file suspected to be the one establishing a covert channel to the attacker's infrastructure?

**File:** `dump.log`

In `dump.log`, file descriptor `14` of the Java process was holding open an unlinked (deleted) file:
```text
(00.031987) 14898 fdinfo 14: pos: 0x34 flags: 100002/0
(00.031996) Dumping path for 14 fd via self 30 [/tmp/.x (deleted)]
(00.031998) Strip ' (deleted)' tag from './tmp/.x (deleted)'
(00.031999) Dumping ghost file for fd 30 id 0x24
(00.032003) Dumping ghost file contents (id 0x1)
```
Because the attacker deleted `/tmp/.x` immediately after opening it, standard filesystem and disk diff analysis (`rootfs-diff.tar`) revealed no malicious traces. It only survived in CRIU userspace memory as a "ghost file".

**Answer:** `/tmp/.x`

---

## 3. What is the inode of the file from previous task?

**File:** `checkpoint/ghost-file-1.img`

In CRIU, ghost file metadata is serialized using the `ghost_file_entry` Protobuf schema:
```protobuf
message ghost_file_entry {
    required uint32 uid   = 1;
    required uint32 gid   = 2;
    required uint32 mode  = 3;
    optional uint32 dev   = 4;
    optional uint64 ino   = 5;
}
```

Reading the raw bytes of `checkpoint/ghost-file-1.img`:
* **Tag 5 (`ino`):** `0x28 0xC7 0xDD 0x13`
* Decoding varint `0xC7 0xDD 0x13`:
  $$71 + (93 \times 128) + (19 \times 16384) = 323271$$
* Hex value: `0x4EEC7`

**Answer:** `323271`

---

## 4. The malicious file was used to setup the container to quietly reach a remote endpoint. What is the full URL?

**File:** `checkpoint/pages-1.img`

Extracting strings from `checkpoint/pages-1.img` reveals an in-memory class (`local/lab/Stager`) loaded by the exploit. The malware mimics an XMRig cryptomining configuration to disguise its C2 traffic:
```text
[xmrig] config loaded from /tmp/.x; pool=xmrpool.eternal-hashes.net:55555 worker=worker-03
```
Inside the Stratum configuration JSON:
```json
"pools":[{"coin":"monero","url":"xmrpool.eternal-hashes.net:55555","user":"49PnvHq8sLab2Tn1Qk7r6Wd4eZ9yJ3cV5bN8mA1sR2tF6gH2xK4wL7pD3qE9oU1iY5aS2dG8hC6vB6nM4kQ9TfXm7Zr3D","pass":"x","rig-id":"worker-03","keepalive":true,"tls":true,"enabled":true}]
```

**Answer:** `xmrpool.eternal-hashes.net:55555` *(o `stratum+tcp://xmrpool.eternal-hashes.net:55555`)*

---

## 5. Connecting to that remote service needs credentials to authenticate. What are the credentials the file uses to authenticate to the remote endpoint?

**File:** `checkpoint/pages-1.img`

From the Stratum mining login payload executed by `local/lab/Stager`:
* **Username / Wallet:** `49PnvHq8sLab2Tn1Qk7r6Wd4eZ9yJ3cV5bN8mA1sR2tF6gH2xK4wL7pD3qE9oU1iY5aS2dG8hC6vB6nM4kQ9TfXm7Zr3D`
* **Password:** `x`

**Answer:** `49PnvHq8sLab2Tn1Qk7r6Wd4eZ9yJ3cV5bN8mA1sR2tF6gH2xK4wL7pD3qE9oU1iY5aS2dG8hC6vB6nM4kQ9TfXm7Zr3D:x`

---

## 6. Identifying the initial access vector is crucial for identifying IOCs and TTPs, making it easier to determine the breach radius. What is the IP address used to compromise the container as part of the initial compromise?

**File:** `container.log` & `dump.log`

1. In `container.log`, all legitimate ingress traffic originated from internal cluster pod subnets (`10.244.x.x`). A single external IP sent an anomalous request right before the compromise at `03:46:14.848Z`:
   ```text
   2026-06-29T03:46:14.848Z INFO 1 --- [nio-8080-exec-8] local.lab.access : REQ ip=27.34.64.82 method=POST path=/api/v1/report status=200 ua="Python-urllib/3.13"
   ```
2. In `dump.log`, CRIU captured socket `0xdf18` lingering in `TCP_FIN_WAIT1` connected to `27.34.64.82:55422`.

**Answer:** `27.34.64.82`

---

## 7. The attacker abused a normal and trusted workflow that the container was used for by FrostLine Analytics to achieve code execution. What is the name of the file that was created by the trusted workflow that gave the attacker access to the container, and what was its title?

**File:** `checkpoint/pages-1.img` & `container.log`

Decompiling `App.class` from `app.jar` in memory shows the report creation endpoint `@PostMapping("/api/v1/report")`:
```java
@PostMapping("/api/v1/report")
public String createReport(
    @RequestParam(defaultValue = "Untitled") String title,
    @RequestParam(defaultValue = "system") String owner,
    @RequestParam(defaultValue = "'guest'") String label) throws Exception {
    
    OgnlContext ctx = (OgnlContext) Ognl.createDefaultContext(new Object(), new OpenAccess());
    Object evalResult = Ognl.getValue(label, ctx, ctx.getRoot());
    String id = "RPT-" + SEQ.getAndIncrement();
    ...
```
The attacker supplied an OGNL expression inside the `label` parameter to invoke `Runner.stageInMemory(...)`. Because the request omitted the `title` parameter, the report was staged with default values:
```json
{
  "id": "RPT-1057",
  "title": "Untitled",
  "owner": "system",
  "label": "staged",
  "status": "ready",
  "rows": 261,
  "created": "2026-06-29T03:46:14.847118535Z"
}
```

**Answer:** `RPT-1057, Untitled` *(o `RPT-1057` / `Untitled`)*

---

## 8. The DevSecOps checklist clearly dictates that service account authentication mechanisms should be considered compromised in the event of an incident. What is the expiry time of the mechanism used in the environment, which could have given the attacker cluster-wide privileged access?

**File:** `checkpoint/pages-1.img`

Searching for JWT tokens (`eyJ...`) in the process memory dump yields the projected Kubernetes ServiceAccount token stolen from `/var/run/secrets/kubernetes.io/serviceaccount/token`:
```text
eyJhbGciOiJSUzI1NiIsImtpZCI6ImJlbzYxRS1rVms5dUlDOWpWbHdTNC03el8wUV9NbmlOclJhT2swQmdKaE0ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxODE0MjM5NzQ5LCJpYXQiOjE3ODI3MDM3NDksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiNjAzN2RmMmYtOGE3Ni00OWU4LTg0MTQtYWJiMjE5ZjU5Y2RhIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJyZXBvcnRpbmctbnMiLCJub2RlIjp7Im5hbWUiOiJpcC0xNzItMzEtMTctMzgiLCJ1aWQiOiJhMTAxMmU2MS00NGFkLTQ3YWMtYTFiMC1lNDZkY2Y2ZGNiNjkifSwicG9kIjp7Im5hbWUiOiJyZXBvcnQtaW50ZXJuYWwtNWNjYzk2NDg2Ni02dHBoOCIsInVpZCI6ImQ3ZTEyMmI5LWE5YmUtNGZhZi1iMTkxLTY3MmYxZTZlMWJlMSJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoicmVwb3J0LXNhIiwidWlkIjoiOGUwNTkwOTctZjA1YS00YTcyLWI5YWItYmRjOGZhNGQ1NTY1In0sIndhcm5hZnRlciI6MTc4MjcwNzM1Nn0sIm5iZiI6MTc4MjcwMzc0OSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OnJlcG9ydGluZy1uczpyZXBvcnQtc2EifQ...
```

Decoding the payload claims:
* **`iat` (Issued at):** `1782703749` $\rightarrow$ `2026-06-29 03:29:09 UTC`
* **`exp` (Expiration):** `1814239749` $\rightarrow$ `2027-06-29 03:29:09 UTC`

**Answer:** `2027-06-29 03:29:09 UTC` *(o `2027-06-29T03:29:09Z` / `1814239749`)*

---

## 9. Identify the name of the thread responsible for malicious activity in the container.

**File:** `checkpoint/pages-1.img`

In `local/lab/Stager.class`, the background worker thread managing the Stratum session and persistence was created with a custom thread name designed to blend in with JVM JIT compilation threads:
```java
WORKER = new Thread(this::run, "C2 CompilerThread9");
WORKER.setDaemon(true);
WORKER.start();
```

**Answer:** `C2 CompilerThread9`

---

## 10. Thankfully, the attacker did not get the chance to steal critical data from the pod, only basic information. What is the full path of the archive file that the attacker created for potential exfiltration?

**File:** `checkpoint/pages-1.img`

The decompiled stager executed a shell command via `java.lang.ProcessBuilder` to stage reconnaissance info (`id`, `uname -a`) into a `.tar.gz` archive stored in shared memory:
```bash
mkdir -p /dev/shm/.X11-unix && { id; uname -a; echo BHAIRAV-WAS-HERE; } > /tmp/r.txt && tar -czf /dev/shm/.X11-unix/.cache.tar.gz -C /tmp r.txt && rm -f /tmp/r.txt
```

**Answer:** `/dev/shm/.X11-unix/.cache.tar.gz`
