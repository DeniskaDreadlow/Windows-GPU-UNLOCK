# Lenovo Legion 7 15ACH6 discrete-MUX capture review — GA106 `24dc10de`

Date: 2026-10-28 (RTZ2). Laptop internal dGPU only; desktop RTX 3090/3090 Ti
routes are unaffected by this note.

Two cold boots, BIOS graphics set to discrete MUX, generic deep probe:
`D:/EVA/gpu_unlock/logs/recon-run2/DIAG00.TXT`, `DIAG01.TXT`, `PROM.BIN`
(copied verbatim from `E:\RTX3090-UNLOCK` root after each boot).
Both runs: `capture-complete=1` at line 6646 and
`diagnostic-result status=0x0000000000000000` at line 6647 of both files —
first clean deep-probe results on this machine after the
`diagnostic.c` protocol-Flush UNSUPPORTED tolerance fix (fork commit
`9619a80`; the previous 10:03 pair reported status=0x8000000000000006 with
byte-complete content).

## Pinned evidence

Every claim cites one line from each cold boot.

### GPU identity

| Fact | DIAG00 line | DIAG01 line |
|---|---|---|
| PCI config dwords `24dc10de 00100003 030000a1 00800010 6b000000 0000000c 00000040` (GA106, command 0x0003 + No-Snoop bit, class 0300 subclass/IFR a1, BAR0 16 MiB at 0x6b000000) | 5011 | 5011 |
| companion HDMI audio function `10de:228b`, dwords `228b10de 00100000 040300a1 00800010 6c000000 00000000 00000000` (class 0403, rev a1, BAR0 16 MiB at 0x6c000000) | 5056 | 5056 |
| BAR0 descriptor `granularity=20 address-min=0x6b000000 length=0x01000000 address-max=0x00ffffff` (relative address-max form; already accepted by the generic probe envelope after the 2026-10-28 review, fork commit `67ccb91`) | 80 | 80 |
| BAR0 base `0x6b000000` | 81 | 81 |

### Chip, VRAM layout, PROM

| Fact | DIAG00 line | DIAG01 line |
|---|---|---|
| chip register `0x00000000 b74000a1` → **GA106** (family nibble `4`; desktop envelope chip is GA102 `b72000a1`) | 83 | 83 |
| `0x00823804 ffffff8f` (FBMMA), `0x0082381c 50041100` and `0x00823820 00000002` (FBMTR layout 2), `0x00824140` = `50041121` / `50041124` | 95–98 | 95–98 |

`0x00824140` low bits differ between boots (run2-01=`21`, run2-02=`24`) —
live memory/boost counter, like the handle-address volatility already
documented for this machine; identity fields are byte-stable.

`PROM.BIN` sha256 `91db59aaf59d3ec5e822a3412536968ab5fcbfeb90392d1a57a2b28b37d4a368`
(DIAG line 99 in both boots) and byte-identical to the earlier 10:03 laptop
capture `D:/EVA/gpu_unlock/logs/recon/PROM.BIN`.
`fwsec-locate.exe` on this PROM parses cleanly:
`image_off=283104 image_size=59904 dmem_size=2048 data_off=57856
pkc_off=1444 ucode_id=9 engine_mask=1024 sig_count=3 sig_versions=7`
— satisfies the FWSEC-parse prerequisite, like the desktop identities.

### GOP visibility

`gops=0002` (line 102): two GOP handles; the RTX GOP has a device path
(line 103), the second GOP handle has no path (line 109). Framebuffer state
captured at lines 4855–4856 / 5717–5718 (2560x1600 panel, 16 modes).
No NVIDIA GOP driver image was dumped on this machine, so its driver
version/text hashes are **unknown**.

## Scope verdict

- Probe/hardware path is healthy: full 618-handle graph, all MMIO/PROM
  reads `status=0`, content complete, clean exit status.
- The automatic local-profile route remains **not applicable** here:
  `check-compatibility.py` reports `[gpu_scope]` for both captures because
  `research/profile/proven.json` documents only `220310de`/`220410de` with
  the GA102 chip and pinned nvgop-ga1xx text hashes. `24dc10de` is a GA106
  laptop identity with an unknown NVIDIA GOP driver image.
- No hardware/provenance, Windows-recovery or write eligibility is granted
  by this note. Extending write-enabled support to this identity is a
  PORTABILITY stage-4+ decision (`docs/PORTABILITY.en.md`: per-identity
  pinned records; hybrid-laptop paths need an explicit design).
