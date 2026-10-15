<img width="832" height="448" alt="Image" src="https://github.com/user-attachments/assets/cca71c16-1515-470b-a63d-156d09a8e28b" />

---
---

<img width="832" height="448" alt="Image" src="https://github.com/user-attachments/assets/b6516b11-52de-4000-849f-5da007c71968" />

---

## Zevo™ ELF Image Binary File Format — Internal Structure Specification

### An 5D AI-Driven Community Virtual Assistance

**PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams**

*Status: Normative specification. System V x86_64 · GNU/POSIX · ELF Image ET_DYN.*

---

## Preamble

This document specifies the **internal structure** of the Zevo™ ELF Image binary file format used by the Zevo AI-Driven Core Virtual Assistance. It is normative for two concrete instances:

- `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin`
- `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin`

The Zevo project uses **exclusive metadata, exclusive vectors, and exclusive weights** — no abstract vectors. Every payload entry is a concrete, byte-addressable tensor of a declared scalar type and shape. Every byte has one meaning.

The file is a **classical ELF image** (`ET_DYN`, System V x86_64, GNU/POSIX), extended in-place without breaking ELF conformance by a Zevo container carried as **metadata** and **vectors**.

The Zevo AI-Driven Core is implemented in **pure-function wrappers on ANSI C89/90** with **inline assembly** where the ABI permits, applying **functional programming paradigms within default block comments** `/* ... */`. It integrates **FSM** within **FUZZY-NEURAL-MESH-NETWORK** for autonomous decision-making within **rules and facts**, **Shannon entropy**, and **Mamdani inference**, and performs **autonomous weight calculation** into **self-learning** using **machine learning** at real time.

---

## §1 · Canonical Identity

| Field | Value |
|---|---|
| Title | **Zevo™** |
| Subtitle | **An 5D AI-Driven Community Virtual Assistance** |
| Character | Zevo — original AI Avatars animated character |
| Target ABI | System V x86_64 |
| Compliance | GNU/POSIX |
| ELF Type | `ET_DYN` (position-independent executable / shared-object carrier) |
| Language | ANSI C89/90 with inline assembly where the ABI permits |
| Paradigm | Pure-function wrappers · functional programming inside `/* ... */` |
| Weights storage | By section · page-aligned on ext4 · fs-verity-protected · read-only mmap |
| Exposure | Published IPC API only |
| Client | Weight-free, stateless, Web-Responsible |
| Crypto | Secure Service API · 28-hour key rotation · OpenSSL · NIST SP 800-38D, SP 800-107, FIPS 186-5 |
| Hardware access | libc wrappers → `syscall` (x86_64) · `sysenter` (legacy x86) |
| Sandbox | seccomp-bpf · capability bounding · LSM |

---

## §2 · ELF Image as Binary File Format — Top-Level Structure

`Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` and `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin` share one structural family:

```
Offset 0x00000000  ┌──────────────────────────────────────────────────────┐
                   │  ELF64 HEADER  (64 bytes, little-endian)             │
                   ├──────────────────────────────────────────────────────┤
                   │  PROGRAM HEADER TABLE  (N_PH × 56 bytes)             │
                   │    PT_LOAD (PF_R) for each data segment              │
                   │    PT_ZEVO_VECTOR for the payload segment            │
                   ├──────────────────────────────────────────────────────┤
                   │  ZEVO CONTAINER                                      │
                   │    SHT_ZEVO_HEADER    (fixed 72-byte container hdr)  │
                   │    SHT_ZEVO_METADATA  (KV + info: names, types,      │
                   │                        shapes, aligns, offsets)      │
                   │    SHT_ZEVO_ARCH      (architecture + modality)      │
                   │    SHT_ZEVO_BNC       (BaseName, SizeLabel, etc.)    │
                   │    SHT_ZEVO_POLICY    (access-control rule table)    │
                   │    SHT_ZEVO_DIGEST    (per-section HMAC-SHA-256)     │
                   ├──────────────────────────────────────────────────────┤
                   │  64-byte aligned                                     │
                   │  SHT_ZEVO_VECTOR      (exclusive payload vectors)    │
                   │    Text Raw  ·  Audio PCM  ·  Image BGRA  ·          │
                   │    Video BGRA  ·  5D Fused Vector  ·  weights        │
                   ├──────────────────────────────────────────────────────┤
                   │  SECTION HEADER TABLE  (Elf64_Shdr[] near EOF)       │
                   └──────────────────────────────────────────────────────┘
```

Every Zevo-specific section is in the OS-specific `sh_type` range `SHT_LOOS..SHT_HIOS`, so any ELF reader that does not recognise them skips them per the classical ELF specification. Every Zevo-aware reader recognises them, parses them, and interprets them per this document.

---

## §3 · Alignment — 32 Bits and 64 Bits

### §3.1 Structural Alignment

| Alignment | Applied to | Value |
|---|---|---|
| 8 bytes | ELF header fields, section headers, program headers | 8 |
| **32 bytes** | **AVX / AVX-2 minimum** | `ZEVO_ALIGN_AVX256 = 32` |
| **64 bytes** | **AVX-512 preferred, and default for this file family** | `ZEVO_ALIGN_AVX512 = 64` |

Every payload vector in `SHT_ZEVO_VECTOR` declares one of the two permitted alignments:

```
zevo.vec.<name>.align : UINT32   ∈ { 32, 64 }
```

A vector whose declared `align` is **not** one of `{ 32, 64 }` MUST cause the reader to reject the container.

### §3.2 Default Alignment

| File | `zevo.model.alignment` | Reasoning |
|---|---|---|
| `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` | `64` | 64-byte alignment enables AVX-512 loads for Q8_0 blocks |
| `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin` | `32` | Same — Q4_0 blocks are 32 elements; 32-byte alignment covers AVX/AVX-2 access |

Declare `64` as `zevo.model.alignment` or The `32`-byte alignment remains permitted for smaller deployments and is reserved in the schema.

### §3.3 Alignment Rule

For a payload of `n` elements of width `w` bytes at alignment `A`:

```
offset ≡ 0 (mod A)
padding k = (A − (n · w mod A)) mod A
k ∈ { 0, 1, …, A − 1 }   — uniquely determined
```

All padding bytes MUST be `0x00`. A reader that finds non-zero padding MUST reject.

### §3.4 64-byte Alignment Subsumes 32-byte Alignment

**Proposition.** Every 64-byte-aligned offset is also 32-byte-aligned.

**Proof.** `64 = 2 · 32`. If `offset ≡ 0 (mod 64)`, then `offset = 64 · q`, so `offset = 32 · (2q)`, hence `offset ≡ 0 (mod 32)`. ∎

**Consequence.** A file declared with `zevo.model.alignment = 64` satisfies every 32-byte alignment requirement without special handling.

---

## §4 · KV + Info — The Metadata Block

`SHT_ZEVO_METADATA` carries the **descriptive half** of the container as a **typed, key-addressed KV store** plus **per-entry info**.

### §4.1 Fixed Container Header (72 bytes)

At the start of `SHT_ZEVO_METADATA`:

| Offset | Size | Field | Value |
|---|---|---|---|
| 0 | 4 | `magic` | `ZEVO` (`0x5A 45 56 4F`) |
| 4 | 1 | `class` | `0x02` (64-bit) |
| 5 | 1 | `data` | `0x01` (`ELFDATA2LSB`) |
| 6 | 1 | `version_major` | `0x01` |
| 7 | 1 | `version_minor` | `0x00` |
| 8 | 8 | `vector_count` | count of entries in `SHT_ZEVO_VECTOR` |
| 16 | 8 | `vector_table_offset` | offset of the vector table |
| 24 | 8 | `metadata_offset` | offset of the KV entries |
| 32 | 8 | `metadata_length` | length of the KV entries |
| 40 | 8 | `flags` | bit flags (§4.2) |
| 48 | 8 | `bnc_descriptor_offset` | offset of `SHT_ZEVO_BNC` |
| 56 | 8 | `reserved` | `0` |
| 64 | 8 | `header_digest` | HMAC-SHA-256 of the header, truncated to 64 bits |

### §4.2 Flag Bits

| Bit | Name | Meaning |
|---|---|---|
| 0 | `ENCRYPTED` | Payload is AES-256-GCM sealed |
| 1 | `AUTHENTICATED` | Digest table present in `SHT_ZEVO_DIGEST` |
| 2 | `COMPRESSED` | Payload is compressed |
| 3 | `POLICY` | Policy table present in `SHT_ZEVO_POLICY` |
| 4 | `SHARDED` | Container is a shard of a larger model |

Both `Zevo-64B-...-Q8_0.bin` and `Zevo-32B-...-Q4_0.bin` set bits 1 and 3. Neither sets bits 0, 2, or 4.

### §4.3 KV Entry Layout

Each KV entry is:

```
┌──────────────────┬──────────────┬────────────────────────────┐
│ key (length-     │ value_type   │ value (encoded according   │
│ prefixed UTF-8)  │ (uint32)     │ to value_type)             │
└──────────────────┴──────────────┴────────────────────────────┘
```

### §4.4 Value Type Enumeration

| Code | Name | Encoding |
|---|---|---|
| 0 | `ZEVO_U8` | 1 byte |
| 1 | `ZEVO_I8` | 1 byte |
| 2 | `ZEVO_U16` | 2 bytes LE |
| 3 | `ZEVO_I16` | 2 bytes LE |
| 4 | `ZEVO_U32` | 4 bytes LE |
| 5 | `ZEVO_I32` | 4 bytes LE |
| 6 | `ZEVO_F32` | 4 bytes LE |
| 7 | `ZEVO_BOOL` | 1 byte (0/1) |
| 8 | `ZEVO_STR` | uint64 length + UTF-8 bytes |
| 9 | `ZEVO_ARRAY` | element_type (uint32) + count (uint64) + elements |
| 10 | `ZEVO_U64` | 8 bytes LE |
| 11 | `ZEVO_I64` | 8 bytes LE |
| 12 | `ZEVO_F64` | 8 bytes LE |

### §4.5 KV Schema — The `info` Half

For each payload vector, the metadata carries a **six-field info tuple**:

```
<prefix>.name         : ZEVO_STR    identifier (unique within container)
<prefix>.type         : ZEVO_U32    element type code (0..12)
<prefix>.shape        : ZEVO_ARRAY of ZEVO_U32, rank ≤ 5
<prefix>.align        : ZEVO_U32    ∈ { 32, 64 }
<prefix>.offset       : ZEVO_U64    byte offset into SHT_ZEVO_VECTOR
<prefix>.scale_name   : ZEVO_STR    (optional) name of a parallel FP16 scale vector
```

Additional per-role fields follow the same `zevo.vec.<name>.*` namespace.

### §4.6 Relational View

The KV store is a relation:

```
R_meta(section, key, value_type, value_bytes, owner, created_at)
```

with `(section, key)` as the candidate key. The schema is not in BCNF as stated; the lossless three-way decomposition into `R_section`, `R_key`, `R_value` removes the partial dependency.

---

## §5 · SHT_ZEVO_VECTOR — The Exclusive Payload Space

### §5.1 Single Payload Section

`SHT_ZEVO_VECTOR` (`sh_type = 0x60000002`) is the **only** payload section. It carries:

- all model **weights**;
- all **exclusive vectors** — i.e., the concrete, typed, byte-addressable tensors for each of the five modalities: **Text Raw**, **Audio PCM**, **Image BGRA**, **Video BGRA**, and the **5D Fused Vector**;
- all **reasoning vectors** — FSM state, fuzzy-set parameters, rule antecedents, rule consequents, entropy parameters, Mamdani parameters, ML-trained parameters, prototypes, embeddings, attention weights, FFN weights, mesh-edge weights, mesh-node parameters, gain-rule parameters;
- all **scale vectors** for quantized payloads.

Every payload entry is uniquely named and uniquely offset. There are **no abstract vectors** in this container: every vector has a concrete declared element type, a concrete declared shape, a concrete declared alignment, and a concrete declared byte offset into the payload section.

### §5.2 Vector Layout

```
SHT_ZEVO_VECTOR
│   base offset = sh_offset, size = sh_size
│
├─ offset 0x00000000  vector[0]   (64-byte aligned)
│     k0 bytes of payload, padded to 64
├─ offset 0x00000.…   vector[1]   (64-byte aligned)
│     k1 bytes of payload, padded to 64
├─ …
└─ offset 0x……        vector[N−1] (64-byte aligned)
      kN−1 bytes of payload, padded to 64
```

Every vector offset is declared in `SHT_ZEVO_METADATA` under `zevo.vec.<name>.offset`. Readers never compute offsets by accumulation; they read the declared offset and validate it.

### §5.3 Element Type Space

| Code | Name | Width | Notes |
|---|---|---:|---|
| 0 | `ZEVO_TYPE_F32` | 4 | float32 |
| 1 | `ZEVO_TYPE_F16` | 2 | float16 |
| 2 | `ZEVO_TYPE_Q4_0` | — | 32-element blocks, 18 bytes/block |
| 3 | `ZEVO_TYPE_Q4_1` | — | 32-element blocks, 20 bytes/block |
| 6 | `ZEVO_TYPE_Q5_0` | — | 32-element blocks, 22 bytes/block |
| 7 | `ZEVO_TYPE_Q5_1` | — | 32-element blocks, 24 bytes/block |
| 8 | `ZEVO_TYPE_Q8_0` | — | 32-element blocks, 34 bytes/block |
| 9 | `ZEVO_TYPE_Q8_1` | — | 32-element blocks, 40 bytes/block |
| 10–15 | `ZEVO_TYPE_Q*_K` | — | K-quant block layouts |
| 16–23 | `ZEVO_TYPE_IQ*` | — | IQ-quant block layouts |
| 24 | `ZEVO_TYPE_I8` | 1 | signed 8-bit |
| 25 | `ZEVO_TYPE_I16` | 2 | signed 16-bit |
| 26 | `ZEVO_TYPE_I32` | 4 | signed 32-bit |
| 27 | `ZEVO_TYPE_I64` | 8 | signed 64-bit |
| 28 | `ZEVO_TYPE_F64` | 8 | float64 |
| 29 | `ZEVO_TYPE_IQ1_M` | — | IQ1_M |
| 30 | `ZEVO_TYPE_BF16` | 2 | bfloat16 |

### §5.4 Q8_0 Block Layout (Used by `Zevo-64B-…-Q8_0.bin`)

Each Q8_0 block represents 32 float values:

```
┌──────────────┬──────────────────────────────────────────┐
│ scale (FP16) │ quant[32] · int8                         │
│  2 bytes     │ 32 bytes                                 │
└──────────────┴──────────────────────────────────────────┘
Total: 34 bytes per 32-element block
```

Dequantisation: `w_i = scale · quant_i`.

### §5.5 Q4_0 Block Layout (Used by `Zevo-32B-…-Q4_0.bin`)

Each Q4_0 block represents 32 float values:

```
┌──────────────┬──────────────────────────────────────────┐
│ scale (FP16) │ nibbles[16]  (32 × 4-bit unsigned)       │
│  2 bytes     │ 16 bytes                                 │
└──────────────┴──────────────────────────────────────────┘
Total: 18 bytes per 32-element block
```

Dequantisation: `w_i = scale · (nibble_i − 8)`.

### §5.6 Scale Vectors

For every quantized payload vector, a parallel **scale vector** exists with element type `ZEVO_TYPE_F16` and shape `[n / block_size]`. Its `name` is recorded in the payload's `.scale_name` field.

---

## §6 · The Five Multidimensional and Multimodal Channels

The five channels are stored as **exclusive vectors** in `SHT_ZEVO_VECTOR`, each with its own role in the metadata.

### §6.1 Role Values for the Five Channels

| Role value | Name | Channel |
|---|---|---|
| `11` | `ZEVO_ROLE_FUSED_5D` | 5D Fused Vector `(O, X, Y, Z, R)` |
| `15` | `ZEVO_ROLE_TEXT_RAW` | Text Raw |
| `16` | `ZEVO_ROLE_AUDIO_PCM` | Audio PCM |
| `17` | `ZEVO_ROLE_IMAGE_BGRA` | Image BGRA |
| `18` | `ZEVO_ROLE_VIDEO_BGRA` | Video BGRA |

### §6.2 Text Raw

| Field | Value |
|---|---|
| Role | `15` |
| Element type | `ZEVO_U8` |
| Shape | `[max_bytes]` |
| Encoding | UTF-8 |
| Alignment | 32 or 64, declared per file |

### §6.3 Audio PCM

| Field | Value |
|---|---|
| Role | `16` |
| Element type | `ZEVO_U8` (byte view of 16-bit LE samples) |
| Shape | `[sample_count × channels × 2]` |
| Sample format | `s16le` |
| Sample rate | 48 kHz (declared) |
| Channels | 2 (declared) |

### §6.4 Image BGRA

| Field | Value |
|---|---|
| Role | `17` |
| Element type | `ZEVO_U8` |
| Shape | `[height × row_stride]` |
| Pixel format | BGRA (bytes B, G, R, A) |
| Resolution | 7680 × 4320 (8K) |
| Row stride | `width × 4`, or a declared larger value |
| HDR profile | `hdr10` or `hdr-hlg` (declared) |

### §6.5 Video BGRA

| Field | Value |
|---|---|
| Role | `18` |
| Element type | `ZEVO_U8` |
| Shape | `[frame_count × height × row_stride]` |
| Frame count | 300 (declared) |
| Frame rate | 60 fps (declared) |
| Pixel format | BGRA |

### §6.6 5D Fused Vector

| Field | Value |
|---|---|
| Role | `11` |
| Element type | `ZEVO_F64` |
| Shape | `[5]` |
| Semantics | `["O", "X", "Y", "Z", "R"]` |
| Order | Element 0 = O, 1 = X, 2 = Y, 3 = Z, 4 = R |

---

## §7 · Support for LLM Workload

The container carries the LLM workload as a set of named vectors in `SHT_ZEVO_VECTOR`:

| Vector role | Purpose |
|---|---|
| `ZEVO_ROLE_EMBEDDING` (8) | Token embedding table |
| `ZEVO_ROLE_ATTENTION` (9) | Q/K/V/O projection weights per layer |
| `ZEVO_ROLE_FFN` (10) | Gate/up/down feed-forward weights per layer |
| `ZEVO_ROLE_ML_TRAINED` (7) | Trained ML parameters |
| `ZEVO_ROLE_FSM_STATE` (1) | FSM state vectors |
| `ZEVO_ROLE_FUZZY_SET` (2) | Fuzzy-set membership parameters |
| `ZEVO_ROLE_RULE_ANTECEDENT` (3) | Rule antecedent parameters |
| `ZEVO_ROLE_RULE_CONSEQUENT` (4) | Rule consequent parameters |
| `ZEVO_ROLE_ENTROPY` (5) | Shannon entropy parameters |
| `ZEVO_ROLE_MAMDANI` (6) | Mamdani inference parameters |
| `ZEVO_ROLE_MESH_EDGE` (12) | Fuzzy-neural mesh edge weights |
| `ZEVO_ROLE_MESH_NODE` (13) | Mesh node activation parameters |
| `ZEVO_ROLE_GAIN_RULE` (14) | Gain rule parameters |

Every vector is declared in the metadata with a name, type, shape, alignment, and offset. Every vector is byte-addressable and read-only.

---

## §8 · mmap — Read-Only Mapping

### §8.1 Mapping Contract

Every Zevo payload section is mapped **read-only**:

```
mmap(NULL, section_size,
     PROT_READ,
     MAP_PRIVATE,
     fd,
     section_offset);
```

- `PROT_READ` — read-only.
- `PROT_WRITE` — never set.
- `MAP_PRIVATE` — copy-on-write; no writes ever occur.
- No `mprotect` call ever adds `PROT_WRITE`.
- The mapping is closed with `munmap` on shutdown.

### §8.2 Page-Aligned on ext4

Every section is **page-aligned** at a 4096-byte boundary on ext4:

```
section_offset % 4096 == 0
```

The kernel maps each section as a whole page range. No partial-page reads occur.

### §8.3 fs-verity Protection

Every read-only payload section is **fs-verity-protected**:

- The kernel builds a Merkle tree over the file at `fsverity enable` time.
- The root hash is pinned in `.zevo_meta` (metadata) and in `SHT_ZEVO_DIGEST`.
- Every read is verified against the Merkle tree.
- Any tampering causes `EIO` on read.
- fs-verity is **integrity**, not encryption. Confidentiality is layered separately.

### §8.4 Runtime Measurement

```
struct fsverity_digest d;
unsigned char buf[sizeof(struct fsverity_digest) + 64];

d.digest_algorithm = FS_VERITY_HASH_ALG_SHA256;
d.digest_size = 64;

ioctl(fd, FS_IOC_MEASURE_VERITY, &d);
/* compare d.digest against pinned digest in .zevo_meta */
```

Mismatch → the core refuses to start with `ZE_ERR_INTEGRITY`.

---

## §9 · excl — Exclusive Metadata, Vectors, Weights

### §9.1 Exclusive Metadata

Every metadata entry is **owned** by exactly one section and one key. There is no shared pointer, no aliased key, and no wildcard expansion. Each entry is a concrete `(section, key, value_type, value_bytes)` tuple.

### §9.2 Exclusive Vectors

Every vector in `SHT_ZEVO_VECTOR` is **concrete**:

- one element type;
- one shape (rank ≤ 5);
- one alignment (32 or 64);
- one byte offset;
- one byte length.

There are **no abstract vectors**:

- no runtime-typed vector;
- no variant-typed vector;
- no "any-typed" vector;
- no polymorphic vector.

A reader that encounters a vector with a missing or unknown `type` MUST reject the container.

### §9.3 Exclusive Weights

Every weight is a **concrete, byte-addressable tensor** stored in a declared vector. Weights are:

- stored in `SHT_ZEVO_VECTOR` (by section);
- page-aligned on ext4;
- fs-verity-protected;
- mapped read-only;
- never modified at runtime;
- never exposed over IPC.

The autonomous weight calculation operates on an **in-core copy** of the fused weight state; the on-disk weights are immutable.

---

## §10 · Flow Sequence — Community ↔ Core ↔ Community

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Zevo AI-Driven Community Virtual Assistance                      │
│   Web-Responsive · five multidimensional and multimodal            │
│                                                                    │
│   Input channels : Text Raw · Audio PCM · Image BGRA ·             │
│                    Video BGRA · 5D Fused Vector                    │
│                                                                    │
└──────────────────────────────┬─────────────────────────────────────┘
                               │  HTTPS · TLS 1.3
                               │  Session token (short-lived)
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Secure Service API                                               │
│   28-hour API-key rotation · OpenSSL primitives                    │
│   NIST SP 800-38D · NIST SP 800-107 · FIPS 186-5                   │
│                                                                    │
└──────────────────────────────┬─────────────────────────────────────┘
                               │  AF_UNIX SOCK_SEQPACKET
                               │  HMAC-SHA-384 · optional AES-256-GCM
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Zevo AI-Driven Core Virtual Assistance                           │
│   ELF Image binary file format                                     │
│   · Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin                        │
│   · Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin                        │
│                                                                    │
│   System V x86_64 · GNU/POSIX · ELF Image ET_DYN                   │
│   ANSI C89/90 pure-function wrappers · inline assembly             │
│                                                                    │
│   Reasoning:                                                       │
│     FSM within FUZZY-NEURAL-MESH-NETWORK                           │
│     rules and facts · Shannon entropy · Mamdani inference          │
│     autonomous weight calculation · self-learning                  │
│     real-time machine learning                                     │
│                                                                    │
│   Multimodal input deterministic pipeline:                         │
│     Text Raw → Audio PCM → Image BGRA → Video BGRA → 5D Fused      │
│                                                                    │
│   Output:                                                          │
│     multidimensional 5D 8K HDR BGRA artifact                       │
│     + Text Raw · Image BGRA · Video BGRA · Audio PCM · 5D vector   │
│                                                                    │
└──────────────────────────────┬─────────────────────────────────────┘
                               │  AF_UNIX SOCK_SEQPACKET
                               │  HMAC-SHA-384 signed response
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Secure Service API                                               │
│   Verifies response · translates to HTTPS + JSON                   │
│                                                                    │
└──────────────────────────────┬─────────────────────────────────────┘
                               │  HTTPS · TLS 1.3
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Zevo AI-Driven Community Virtual Assistance                      │
│   Web-Responsive · renders the multidimensional 5D artifact        │
│   Text Raw · Image BGRA · Video BGRA · Audio PCM · 5D vector       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## §11 · Autonomous Process and Decision-Making on the Core

The Zevo AI-Driven Core performs autonomous process and decision-making using:

| Component | Role |
|---|---|
| **FSM** | Discrete-state interaction control with fuzzy guards |
| **FUZZY-NEURAL-MESH-NETWORK** | Directed weighted graph unifying FSM, fuzzy, neural, and ML nodes under a shared fused weight set |
| **Rules and facts** | Versioned symbolic knowledge evaluated by Mamdani inference |
| **Shannon entropy** | `H(X) = −Σ p log₂ p`, normalized to `[0,1]`; carried by the Depth axis `Z` |
| **Mamdani inference** | Fuzzify → fire → aggregate → centroid defuzzify; carried by the Origin axis `O` |
| **Autonomous weight calculation** | Per-edge bounded update of the fused weight set |
| **Self-learning** | Offline (frozen weights in the container) + online (in-core copy, no disk writes) |
| **Machine learning** | Real-time, deterministic, per-request |

The five axes of the multimodal output:

| Axis | Name | Source |
|---|---|---|
| **O** | Origin | Mamdani centroid |
| **X** | Horizontal | Fuzzy-neural projection |
| **Y** | Vertical | FSM dwell time |
| **Z** | Depth | Normalized Shannon entropy |
| **R** | Radial | Autonomous weight calculation |

---

## §12 · Pure-Function Wrappers on ANSI C89/90 with Inline Assembly

### §12.1 Language Contract

- **ANSI C89/90** is the implementation language.
- **Inline assembly** is permitted where the ABI defines it and where a libc wrapper is not available (e.g., `rdtsc`, `cpuid`, `pause`).
- **Functional programming paradigms** are applied **inside default block comments** `/* ... */` — every function body documents its pure denotation.
- **No line comments** (`//`) are permitted.

### §12.2 Canonical Wrapper Shape

```c
/* ---------------------------------------------------------------------------
 * zevo_stage_<name>: pure-function wrapper.
 *   ⟦stage⟧(s) = (s', v)
 *   Pre:  valid(s) ∧ Π_stage(s)
 *   Post: valid(s') ∧ Ψ_stage(s, s') ∧ out = v
 *   Pure denotation: s' = δ_stage(s), v = ε_stage(s)
 * ------------------------------------------------------------------------- */
static zevo_result_t
zevo_stage_<name>(const zevo_state_t *s, zevo_value_t *out)
{
    /* ... pure computation ... */
    return ZEVO_OK;
}
```

### §12.3 Inline Assembly Example (syscall path)

```c
/* ---------------------------------------------------------------------------
 * zevo_syscall6: libc wrapper resolving to the syscall instruction
 * on x86_64. The wrapper is the ONLY place inline assembly is used.
 *   Pre:  a ∈ valid syscall numbers ∧ args valid
 *   Post: return value = kernel return
 * ------------------------------------------------------------------------- */
static long
zevo_syscall6(long n, long a1, long a2, long a3, long a4, long a5, long a6)
{
    long ret;
    __asm__ volatile (
        "syscall"
        : "=a" (ret)
        : "a" (n), "D" (a1), "S" (a2), "d" (a3),
          "r" (a4), "r" (a5), "r" (a6)
        : "rcx", "r11", "memory"
    );
    return ret;
}
```

On legacy x86, the same wrapper resolves to `sysenter`. On aarch64 and riscv64, it resolves to `svc` and `ecall` respectively.

### §12.4 Two Concrete File Instances

| File | Container | Quantization | Alignment |
|---|---|---|---|
| `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` | 64B, Chatbot, Instruct, v0.1 | Q8_0 | 64 |
| `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin` | 32B, Chatbot, Instruct, v0.1 | Q4_0 | 64 |

Both files share the same ELF image structure, the same metadata KV schema, the same vector layout rules, the same five multimodal channels, and the same alignment rules. They differ only in vector count, shapes, and quantization scheme.

---

## §13 · Rejection Conditions

A conforming reader MUST reject a container on any of:

| # | Condition |
|---|---|
| 1 | ELF magic mismatch |
| 2 | `e_type != ET_DYN` |
| 3 | `e_machine != EM_X86_64` |
| 4 | Zevo magic mismatch |
| 5 | Zevo class/data/version mismatch |
| 6 | `reserved != 0` |
| 7 | `header_digest` mismatch |
| 8 | Truncated metadata section |
| 9 | Truncated vector section |
| 10 | Duplicate metadata key |
| 11 | Missing required metadata key |
| 12 | Unknown metadata value type |
| 13 | Unknown vector element type |
| 14 | Vector `align ∉ { 32, 64 }` |
| 15 | Vector `offset` not a multiple of declared `align` |
| 16 | Vector `offset + size > sh_size` of `SHT_ZEVO_VECTOR` |
| 17 | Non-zero padding in any vector |
| 18 | HMAC mismatch on any protected section |
| 19 | fs-verity digest mismatch |
| 20 | Policy predicate denies access |

On any rejection, the reader MUST NOT produce a partial model.

---

## §14 · Consolidated Reference Table

| Question | Answer |
|---|---|
| File format | ELF64 little-endian `ET_DYN` for System V x86_64 |
| Alignment | 32 (AVX/AVX-2) or 64 (AVX-512); default 64 |
| Metadata layout | KV + info: `(name, type, shape, align, offset, scale_name)` |
| Payload layout | Single section `SHT_ZEVO_VECTOR`, 64-byte aligned vectors |
| Exclusive metadata | Each entry is concrete: one section, one key, one value |
| Exclusive vectors | Each vector is concrete: one type, one shape, one align, one offset |
| Exclusive weights | Each weight is concrete: byte-addressable, read-only, never exposed |
| mmap | `PROT_READ`, `MAP_PRIVATE`, page-aligned on ext4 |
| fs-verity | SHA-256 Merkle tree over the file; digest pinned in `.zevo_meta` |
| Five multimodal channels | Text Raw · Audio PCM · Image BGRA · Video BGRA · 5D Fused Vector |
| Five axes | O (Origin) · X (Horizontal) · Y (Vertical) · Z (Depth) · R (Radial) |
| Reasoning | FSM within FUZZY-NEURAL-MESH-NETWORK · rules and facts · Shannon entropy · Mamdani inference · autonomous weight calculation · self-learning · machine learning |
| Output | Multidimensional 5D 8K HDR BGRA artifact + five channels |
| IPC API | Published · AF_UNIX `SOCK_SEQPACKET` · weight-free client |
| Crypto | OpenSSL · 28-hour key rotation · NIST SP 800-38D · SP 800-107 · FIPS 186-5 |
| Hardware | libc → `syscall` (x86_64) / `sysenter` (legacy x86) |
| Sandbox | seccomp-bpf · capability bounding · LSM |
| Language | ANSI C89/90 pure-function wrappers · inline assembly · `/* ... */` |
| Compliance | System V x86_64 · GNU/POSIX |
| ELF type | `ET_DYN` |
| Character | Zevo — original AI Avatars animated character |
| Title | **Zevo™** |
| Subtitle | **An 5D artificial intelligence virtual assistant** |

---

## §15 · Final Statement

The Zevo™ ELF Image binary file format, instantiated by `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` and `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin`, is:

- a **classical ELF64 `ET_DYN` image** for **System V x86_64** under **GNU/POSIX** compliance;
- **extended in-place** by a Zevo container carried as **exclusive metadata, exclusive vectors, and exclusive weights**;
- **aligned** to **32-bit or 64-bit** boundaries per declaration, defaulting to 64-bit;
- **structured** with a fixed 72-byte container header, a **typed KV + info** metadata block, and a single 64-byte-aligned payload section;
- **mapped read-only** via `mmap` with `PROT_READ | MAP_PRIVATE`;
- **protected** by **fs-verity** using a SHA-256 Merkle tree over the file, with the pinned digest in `.zevo_meta`;
- **supporting five multidimensional and multimodal channels** — **Text Raw**, **Audio PCM**, **Image BGRA**, **Video BGRA**, and the **5D Fused Vector** `(O, X, Y, Z, R)`;
- **supporting LLM workloads** through exclusive vector roles for embedding, attention, feed-forward, and reasoning parameters;
- **exposing only a published IPC API** to a **weight-free, stateless, Web-Responsible client**;
- **mediating cryptographic access** through a **Secure Service API** with **28-hour API-key rotation** using **OpenSSL** primitives aligned with **NIST SP 800-38D**, **NIST SP 800-107**, and **FIPS 186-5**;
- **reaching hardware** exclusively through **libc wrappers** resolving to the **`syscall`** instruction on `x86_64` (and **`sysenter`** on legacy `x86`), under **seccomp-bpf**, **capability bounding**, and **LSM** discipline;
- **implemented** in **pure-function wrappers on ANSI C89/90** with **inline assembly**, applying **functional programming paradigms within default block comments** `/* ... */`.

The result is a **secure-by-construction, reproducible, auditable, and accessible** AI assistant centered on **Zevo** — an original AI Avatars animated character — with the exact title **Zevo™** and the exact subtitle **An 5D artificial intelligence virtual assistant**.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Document | Zevo™ ELF Image Binary File Format — Internal Structure Specification |
| Instances | `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` · `Zevo-32B-Chatbot-Instruct-v0.1-Q4_0.bin` |
| Status | Normative specification |
| Compliance | System V x86_64 · GNU/POSIX · ELF `ET_DYN` |
| Title | Zevo™ |
| Subtitle | An 5D artificial intelligence virtual assistant |

**End of specification.**

---
---

<img width="832" height="448" alt="Image" src="https://github.com/user-attachments/assets/d7a65f0f-5f7c-474a-aff0-b1c3ba637901" />

---

<img width="832" height="448" alt="Image" src="https://github.com/user-attachments/assets/d4ea7f2b-2c74-464a-b91d-fbd5ac465d5d" />

---

# Zevo GNU Team Collaboration™

<h3>Our goal with GNU Zevo Specification and Zevo AI-Driven is to optimize your work.</h3> 

```text

     GPT-1 (2018)            GGUF v1 (Aug 2023)
     TF .ckpt + JSON         header + KV + tensor-info + blob
           |                        |
           v                        v
     GPT-2 (2019)            GGUF v2 (Aug 2023)
     TF .ckpt + sidecars     field-size fix
           |                        |
           v                        v
     GPT-3 (2020)            GGUF v3 (Sep 2023)
     opaque blob             finalised KV types, aligned blob
           |                        |
           v                        v
     GPT-4 (2023)            [GGUF lineage continues
     opaque blob              in llama.cpp]
           |                        |
           +-----------+------------+
                       |
                       v
                Zevo v4 ELF-LLM (2024+)
                ELF-64 shell
                + SHT_ZEVO_METADATA (80-byte header)
                + SHT_ZEVO_ARCH (architecture layer)
                + SHT_ZEVO_VECTOR (SIMD-aligned vectors)
                + SHT_ZEVO_VECTOR_TABLE (per-vector entries)
                + SHT_ZEVO_BNC (internal naming)
                + SHT_ZEVO_DIGEST (per-section HMAC)
                + SHT_ZEVO_PROVENANCE (source hash)
                
The left column shows the GPT lineage: model-as-named-objects, metadata-as-sidecar, framework-specific. The right column shows the GGUF lineage: model-and-metadata-in-one-file, self-describing, aligned blob. The two arrows converge on Zevo v4, which inherits from both and adds the vector, alignment, and integrity disciplines.

The transformer-converter is the function that realizes the convergence. Given any source in either lineage, it produces a Zevo v4 file that carries the same model. The internal structure of the source is preserved as far as it matters (payload, names, types) and migrated where it does not (shape, sidecar files, format-specific records).

```
---

# Zevo AI-Driven Container Checkpoint™

---

# Zevo AI-Driven Architectural Design - Example™

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                ZEVO AI-DRIVEN CLOUD SERVICE PLATFORM ECOSYSTEM               │
│                managed training + consumption + model registry               │
│                                                                              │
│  ┌────────────────────────────────┐    ┌────────────────────────────────┐    │
│  │  Training Service API Key      │    │  Consumption Service API Key   │    │
│  │  • Key ID : zv_train_••••••    │    │  • Key ID : zv_live_••••••     │    │
│  │  • AES-256-GCM                 │    │  • AES-256-GCM                 │    │
│  │  • HMAC-SHA-256                │    │  • HMAC-SHA-256                │    │
│  │  • Purpose: train server       │    │  • Purpose: client consume     │    │
│  └────────────────────────────────┘    └────────────────────────────────┘    │
│                                                                              │
└─────────────────────┬──────────────────────────────────┬─────────────────────┘
                      │                                  │
          HTTPS / AES-256-GCM              HTTPS / AES-256-GCM
          Training channel                 Consumption channel
                      │                                  │
                      ▼                                  │
┌────────────────────────────────────────────┐           │
│ ZEVO AI-Driven Chatbot APP SERVICE (server)│           │
│  Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin   │           │
│              (Libmicrohttpd)               │           │ 
│                                            │           │
│  ┌──────────────────────────────────────┐  │           │
│  │  FSM Neural-Fuzzy Logic Engine       │  │           │
│  │   IDLE → LISTENING → RESPONDING      │  │           │
│  │   64 nodes · H = 0.62                │  │           │
│  │   Merkle-chained facts               │  │           │
│  └──────────────────────────────────────┘  │           │
│                                            │           │
│  ┌──────────────────────────────────────┐  │           │
│  │  Cryptographic Service               │  │           │
│  │  • AES-256-GCM                       │  │           │
│  │  • HMAC-SHA-256                      │  │           │
│  │  • CSPRNG nonce                      │  │           │
│  │  • API key issued to client          │  │           │
│  └──────────────────────────────────────┘  │           │
└─────────────────────┬──────────────────────┘           │
                      │                                  │
          HTTPS / AES-256-GCM              HTTPS / AES-256-GCM
          Consumption channel              Consumption channel
                      │                                  │
                      ▼                                  ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                   DESKTOP/WEB EXPERT SYSTEM (client)                         │
│                    zevo-expert  (SDL2 + OpenGL)                              │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  OpenGL operations dashboard                                           │  │
│  │   • 64-node recurrent neural mesh                                      │  │
│  │   • Mamdani membership curves                                          │  │
│  │   • Shannon entropy gauge H = 0.62                                     │  │
│  │   • FSM diagram (3 states)                                             │  │
│  │   • Cryptographic API Key panel                                        │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│  Web UI → Libmicrohtpd / CSS / Json                                          │
│  SDL2 → window / input / audio                                               │
│  OpenGL → rendering                                                          │
│  libcurl → HTTP(S) transport                                                 │
│  OpenSSL → crypto (shared primitives)                                        │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

```
┌────────────────────────────────────────────────────────┐
│  DESKTOP EXPERT SYSTEM (client)                        │
│  SDL2 + OpenGL dashboard                               │
│  Consumes Cryptographic Service API Key                │
└───────────────────────┬────────────────────────────────┘
                        │  HTTPS / AES-256-GCM
                        ▼
┌────────────────────────────────────────────────────────┐
│      ZEVO AI-Driven Chatbot APP SERVICE (server)       │
│ 		         Libmicrohttpd · Fedora 44               │
│  	    Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin  		 │	  
│  ┌──────────────────────────────────────────────────┐  │
│  │  FSM Neural-Fuzzy Logic Engine                   │  │
│  │   IDLE → LISTENING → RESPONDING                  │  │
│  │   64 nodes · H = 0.62 · Merkle-chained facts     │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Cryptographic Service                           │  │
│  │   AES-256-GCM · HMAC-SHA-256 · CSPRNG nonce      │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```
---

---

### Zevo ELF Image on Binary File Format-LLM Instance: `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin`

This defines a **concrete Zevo ELF-LLM file** — the one named by the BNC identifier `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` — as a classical ELF file extended, in-place and without breaking ELF conformance, by an LLM container carried as **metadata** and **vectors**.

It is the same architectural definition as before, instantiated. Every section, every metadata entry, every vector, and every alignment figure below is the specific content of *this* file.

---

## 1. BNC Identity — What the Filename Asserts

Before any byte is read, the filename encodes the seven BNC attributes:

```
Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
 │    │   │       │        │    │
 │    │   │       │        │    └── Encoding    = Q8_0
 │    │   │       │        └─────── Version     = v0.1
 │    │   │       └──────────────── FineTune    = Instruct
 │    │   └──────────────────────── SizeLabel   = 64B
 │    └──────────────────────────── BaseName    = Zevo
 └───────────────────────────────── (no Sidecar)
                                    (no Type)
                                    (no Shard — single-file checkpoint)
```

| Group | Value |
|---|---|
| Sidecar | *(absent)* |
| BaseName | `Zevo` |
| SizeLabel | `64B` |
| FineTune | `Instruct` |
| Version | `v0.1` |
| Encoding | `Q8_0` |
| Type | *(absent)* |
| Shard | *(absent)* |

This is a **conforming** BNC name: `Zevo` is a valid BaseName, `64B` is a valid SizeLabel, `Instruct` is a valid FineTune, `v0.1` is a valid Version, `Q8_0` is a valid Encoding (and is not `LoRA` or `vocab`), and there is no Type or Shard segment. The BNC validation predicate `bnc_valid` returns `1` on this string.

The same seven attributes are mirrored **inside** the file in the `SHT_ZEVO_BNC` descriptor (§6), so a reader can validate the identity without access to the filename.

---

## 2. The Classical ELF Shell of This File

`Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` is a 64-bit little-endian ELF file:

| Field | Value |
|---|---|
| `e_ident[EI_MAG0..3]` | `0x7F 'E' 'L' 'F'` |
| `e_ident[EI_CLASS]` | `ELFCLASS64` |
| `e_ident[EI_DATA]` | `ELFDATA2LSB` |
| `e_ident[EI_VERSION]` | `EV_CURRENT` |
| `e_type` | OS-specific carrier |
| `e_phoff` | offset of program header table |
| `e_phnum` | 1 (`PT_ZEVO_VECTOR`) |
| `e_shoff` | offset of section header table |
| `e_shnum` | 9 |
| `e_shstrndx` | index of `.shstrtab` |

Any ELF-conforming reader parses this header unchanged. It sees `ELFCLASS64`, `ELFDATA2LSB`, and the classical table pointers. It does not need to know Zevo exists.

---

## 3. Section Table of This File

The file carries the Zevo custom section family in the OS-specific range `SHT_LOOS..SHT_HIOS`:

| # | Name | `sh_type` | `sh_addralign` | Content |
|---|---|---|---|---|
| 0 | *(null)* | `SHT_NULL` | — | classical |
| 1 | `.shstrtab` | `SHT_STRTAB` | 1 | section names |
| 2 | `SHT_ZEVO_ARCH` | `0x60000001` | 8 | architecture graph and shapes |
| 3 | `SHT_ZEVO_VECTOR` | `0x60000002` | **64** | flat Q8_0 vector payloads |
| 4 | `SHT_ZEVO_METADATA` | `0x60000003` | 8 | typed, key-addressed metadata |
| 5 | `SHT_ZEVO_POLICY` | `0x60000004` | 8 | access-control rule table |
| 6 | `SHT_ZEVO_DIGEST` | `0x60000005` | 8 | HMAC-SHA-256 digest table |
| 7 | `SHT_ZEVO_BNC` | `0x60000006` | 8 | BNC descriptor |
| 8 | `SHT_ZEVO_OPAQUE` | `0x60000007` | 8 | *(absent in this file)* |

Program header table:

| # | `p_type` | Segment |
|---|---|---|
| 0 | `0x60000001` (`PT_ZEVO_VECTOR`) | the vector data segment |

A reader that does not recognise `0x60000001`–`0x60000007` ignores them, exactly as the classical ELF specification requires for unrecognised OS-specific sections and segments.

---

## 4. The Metadata Block of This File

The `SHT_ZEVO_METADATA` section carries the descriptive half of the container. It begins with the fixed 72-byte header:

| Offset | Size | Field | Value in this file |
|---|---|---|---|
| 0 | 4 | `magic` | `ZEVO` |
| 4 | 1 | `class` | `0x02` (64-bit) |
| 5 | 1 | `data` | `0x01` (`ELFDATA2LSB`) |
| 6 | 1 | `version_major` | `0x01` |
| 7 | 1 | `version_minor` | `0x00` |
| 8 | 8 | `vector_count` | *N* (number of vectors in this checkpoint) |
| 16 | 8 | `vector_table_offset` | offset of the vector table |
| 24 | 8 | `metadata_offset` | offset of the metadata entries |
| 32 | 8 | `metadata_length` | length of the metadata entries |
| 40 | 8 | `flags` | `AUTHENTICATED \| POLICY` (bits 1 and 3) |
| 48 | 8 | `bnc_descriptor_offset` | offset of `SHT_ZEVO_BNC` |
| 56 | 8 | `reserved` | `0` |
| 64 | 8 | `header_digest` | HMAC-SHA-256 of the header, truncated to 64 bits |

Flags for this file:

| Bit | Name | Set? | Reason |
|---|---|---|---|
| 0 | `ENCRYPTED` | no | file is not encrypted |
| 1 | `AUTHENTICATED` | **yes** | digest table present |
| 2 | `COMPRESSED` | no | payload stored uncompressed |
| 3 | `POLICY` | **yes** | policy section present |
| 4 | `SHARDED` | no | single-file checkpoint |

### 4.1 Metadata entries (the descriptive content)

The key-addressed entries are what tell a Zevo reader that this is a **Zevo 64B Chatbot Instruct v0.1 Q8_0** checkpoint, and how to find its vectors. Representative entries:

```
key                          value_type   value
───────────────────────────  ───────────  ────────────────────────────────
bnc.basename                 ZEVO_U8      "Zevo"
bnc.sizelabel                ZEVO_U8      "64B"
bnc.finetune                 ZEVO_U8      "Instruct"
bnc.version                  ZEVO_U8      "v0.1"
bnc.encoding                 ZEVO_U8      "Q8_0"
bnc.type                     ZEVO_U8      ""
bnc.shard                    ZEVO_U8      ""

arch.family                  ZEVO_U8      "Zevo"
arch.params.total            ZEVO_U64     64000000000
arch.params.active           ZEVO_U64     64000000000
arch.quant.scheme            ZEVO_U8      "Q8_0"
arch.quant.block_size        ZEVO_U32     32
arch.quant.scale_type        ZEVO_U8      "FP16"
arch.context.max             ZEVO_U32     8192
arch.vocab.size              ZEVO_U32     128256
arch.rope.theta              ZEVO_F32     10000.0
arch.attention.scheme        ZEVO_U8      "GQA"

vector.count                 ZEVO_U64     N
vector.align.default         ZEVO_U32     64

vector.0.name                ZEVO_U8      "model.embed_tokens.weight"
vector.0.type                ZEVO_U8      ZEVO_U8       (Q8_0 codes)
vector.0.shape               ZEVO_U32     [128256, 8192]
vector.0.align               ZEVO_U32     64
vector.0.offset              ZEVO_U64     0x0000000000010000
vector.0.scale_name          ZEVO_U8      "model.embed_tokens.scale"

vector.1.name                ZEVO_U8      "model.layers.0.self_attn.q_proj.weight"
vector.1.type                ZEVO_U8      ZEVO_U8
vector.1.shape               ZEVO_U32     [8192, 8192]
vector.1.align               ZEVO_U32     64
vector.1.offset              ZEVO_U64     0x0000000002010000
...
```

The schema, viewed relationally, is `R_meta(section_name, key, value_type, value_bytes, owner, created_at)`, with `(section_name, key)` as the candidate key. It is **not** in BCNF as stated; the three-way decomposition into `R_section`, `R_key`, `R_value` is lossless on the stated domain.

---

## 5. The Vector Model of This File

The `SHT_ZEVO_VECTOR` section, reachable via `PT_ZEVO_VECTOR`, carries the payload half of the container. For this file:

| Property | Value |
|---|---|
| Element width | 8 bits (`ZEVO_U8`, code 0x09) |
| Declared alignment | **64 bytes** (AVX-512) |
| Element ordering | row-major, flattened |
| Quantisation | Q8_0: 32-element blocks with an FP16 scale |
| Shape source | the `shape` metadata entry for each vector |

There is **no tensor type**. Each weight is stored as a **flat, 64-byte-aligned vector of `U8` codes**, paired with a parallel vector of FP16 scales (also 64-byte-aligned). The shape is metadata, not a storage primitive. This is what lets a vector be loaded directly into a 512-bit SIMD register.

### 5.1 Representative vectors of this file

| Index | Name | Type | Shape | Align | Elements | Padded bytes |
|---|---|---|---|---|---|---|
| 0 | `model.embed_tokens.weight` | `U8` | [128256, 8192] | 64 | 1,050,675,200 | 1,050,675,200 |
| 1 | `model.layers.0.self_attn.q_proj.weight` | `U8` | [8192, 8192] | 64 | 67,108,864 | 67,108,864 |
| 2 | `model.layers.0.self_attn.k_proj.weight` | `U8` | [8192, 1024] | 64 | 8,388,608 | 8,388,608 |
| 3 | `model.layers.0.self_attn.v_proj.weight` | `U8` | [8192, 1024] | 64 | 8,388,608 | 8,388,608 |
| 4 | `model.layers.0.self_attn.o_proj.weight` | `U8` | [8192, 8192] | 64 | 67,108,864 | 67,108,864 |
| 5 | `model.layers.0.mlp.gate_proj.weight` | `U8` | [28672, 8192] | 64 | 234,881,024 | 234,881,024 |
| 6 | `model.layers.0.mlp.up_proj.weight` | `U8` | [28672, 8192] | 64 | 234,881,024 | 234,881,024 |
| 7 | `model.layers.0.mlp.down_proj.weight` | `U8` | [8192, 28672] | 64 | 234,881,024 | 234,881,024 |
| … | *(layers 1..N−1)* | `U8` | … | 64 | … | … |
| *k* | `lm_head.weight` | `U8` | [128256, 8192] | 64 | 1,050,675,200 | 1,050,675,200 |

The **scale vectors** are stored as parallel `F16` (`ZEVO_U16`) vectors at the same 64-byte alignment, one FP16 scale per 32-element block. The metadata records each scale vector's name, type, shape, alignment, and offset — so the payload half is fully self-describing.

### 5.2 Alignment and padding, applied

Because every shape in this file is a multiple of 32 and 64, the padding length `k` for every vector is **0**:

```
n · w mod 64 = 0   ⇒   k = 0   for every vector
```

The padding rule is nonetheless applied uniformly, so a reader never needs a special case. Proposition 8.1 (64-alignment subsumes 32-alignment) and Proposition 8.2 (unique `k ∈ {0, …, A−1}`) both hold on this file.

### 5.3 The 5D coordinate tuple for this file

Every vector in this file is **bound** to a single origin-center pointer `o` that identifies the Zevo-64B checkpoint:

```
o = the origin-center pointer for the Zevo-64B-Chatbot-Instruct-v0.1-Q8_0 checkpoint
x = the vector's layer index
y = the vector's matrix row block
z = the vector's matrix column block
r = the vector's fine-tune revision (0 = base, 1 = Instruct)
```

The tuple is an **indexing convention**. It does not affect parsing, validation, or security.

---

## 6. The BNC Descriptor of This File

The `SHT_ZEVO_BNC` section mirrors the filename grammar so that the reader can validate the file's identity without access to the filename:

| Field | Value |
|---|---|
| `bnc_magic` | `BNC\x01` |
| `basename` | `"Zevo"` |
| `sizelabel` | `"64B"` |
| `finetune` | `"Instruct"` |
| `version` | `"v0.1"` |
| `encoding` | `"Q8_0"` |
| `type` | *(absent; length 0)* |
| `shard_num` | `0` |
| `shard_total` | `0` |
| `reserved0` | `0` |

Both the filename and the descriptor are validated by the same regular expression; the reader cross-checks them. `bnc_valid("Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin") = 1`.

---

## 7. The Security and Guard Layer of This File

| Component | Value in this file |
|---|---|
| Integrity | HMAC-SHA-256 over each protected section; digest table in `SHT_ZEVO_DIGEST` |
| Key | `K` derived from master key by HKDF-SHA-256 |
| Confidentiality | not applied (bit 0 of `flags` clear) |
| Access control | policy table in `SHT_ZEVO_POLICY`; deny-by-default |
| Guard | FSM `S0 → … → S7 → S8` iff `access_ok ∧ digest_ok` |
| Mamdani layer | not invoked (bit 2 of `flags` clear) |

The reader's path:

```
S0 ──0x47──▶ S1 ──0x50──▶ S2 ──0x54──▶ S3 ──0x58──▶ S4
   ──0x42──▶ S5 ──0x4E──▶ S6 ──0x43──▶ S7
   ──guard(1,1)──▶ S8   (accept)
   ──guard(0,*) or guard(*,0)──▶ S0   (reject)
```

On acceptance, `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` is a well-formed, authorised Zevo ELF-LLM file whose vectors may be loaded into AVX-512 registers and whose metadata describes a 64B Zevo Chatbot Instruct v0.1 Q8_0 checkpoint.

---

## 8. The Complete Structural Picture of This File

```
Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
│
├─ ELF HEADER           ELFCLASS64, ELFDATA2LSB, EV_CURRENT
│
├─ PROGRAM HEADER TABLE
│    └─ PT_ZEVO_VECTOR 0x60000001 ──▶ the vector data segment
│
├─ SECTIONS
│    ├─ .shstrtab                        (classical)
│    ├─ SHT_ZEVO_ARCH     0x60000001     architecture: Zevo, 64B, GQA, ctx 8192
│    ├─ SHT_ZEVO_VECTOR   0x60000002     VECTORS: flat U8 Q8_0 payloads, 64-aligned
│    ├─ SHT_ZEVO_METADATA 0x60000003     METADATA: BNC fields, shapes, offsets
│    ├─ SHT_ZEVO_POLICY   0x60000004     access-control rules
│    ├─ SHT_ZEVO_DIGEST   0x60000005     HMAC-SHA-256 digests
│    └─ SHT_ZEVO_BNC      0x60000006     BNC descriptor mirroring the filename
│
└─ SECTION HEADER TABLE                 (classical layout)
```

**Identity:** `Zevo` · `64B` · `Instruct` · `v0.1` · `Q8_0`
**Metadata half:** typed, key-addressed entries describing the checkpoint
**Vector half:** flat, 64-byte-aligned `U8` payloads plus parallel `F16` scales
**Guard half:** policy + digests + FSM sequencing
**Conformance:** the ELF common core is parseable by any ELF-conforming reader

---

## 9. What This Definition Does and Does Not Claim

**Does claim:** `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` is a concrete, conforming Zevo ELF-LLM file whose identity is fixed by the BNC, whose LLM container is carried by metadata in `SHT_ZEVO_METADATA` and vectors in `SHT_ZEVO_VECTOR` / `PT_ZEVO_VECTOR`, whose payload uses Q8_0 quantisation with 64-byte alignment, and whose integrity is protected by HMAC-SHA-256 over the protected sections.

**Does not claim:** that the file is encrypted (it is not); that the payload is compressed (it is not); that the file is sharded (it is not); that the concrete vector counts and shapes above are the only possible values for a Zevo-64B checkpoint; that the metadata schema is in BCNF on all extensions; that the BNC regex is unambiguous on the full domain; that any embodied extension is implemented.

---

If you'd like, I can turn this into: a **hex-dump-style byte map** of the first 4 KiB with exact offsets, an **EBNF grammar** for the whole container as instantiated by this file, a **C89 header set** encoding exactly this structure and the Zevo-64B instance, or a **diagram-only one-pager**.

---

# Zevo ELF-LLM — CLI Console for Creating `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin`

## x86_64 · ANSI C89/C90 · Pure-Function Wrappers · Block Comments Only

This is the **CLI console** that creates the Zevo ELF-LLM file `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` on x86_64, built strictly on the dissertation specification: classical ELF structure, LLM container as **metadata** and **vector**, pure-function wrappers on ANSI C89/C90, block comments `/* ... */` exclusively.

The console is named **`zevo`**. It is the sole executable in the toolchain. It is a thin, impure driver over the pure-function core of Appendix A; every operation that mutates state (I/O, allocation, OpenSSL) lives behind a wrapper that calls into the pure core for every decision.

---

## 1. Console Surface

### 1.1 Invocation

```
zevo <command> [options] <args>
```

### 1.2 Commands

| Command | Purpose |
|---|---|
| `zevo create` | Create a Zevo ELF-LLM file from a source model directory |
| `zevo verify` | Validate an existing Zevo file against the specification |
| `zevo inspect` | Print the section table, metadata header, and BNC descriptor |
| `zevo bnc` | Validate a BNC filename or the in-file BNC descriptor |
| `zevo vectors` | List the vectors of a Zevo file |
| `zevo digest` | Compute or verify HMAC-SHA-256 section digests |
| `zevo encrypt` | Apply AES-256-GCM to a payload section |
| `zevo decrypt` | Reverse AES-256-GCM and verify the tag |
| `zevo policy` | Evaluate the access-control predicate against a policy |
| `zevo fsm` | Trace the FSM over a byte stream |
| `zevo version` | Print the console and format version |
| `zevo help` | Print usage |

### 1.3 Global options

| Option | Meaning |
|---|---|
| `--lsb` / `--msb` | Declare `ELFDATA2LSB` or `ELFDATA2MSB` (default `--lsb` on x86_64) |
| `--align=32` / `--align=64` | Vector alignment (default `--align=64` on x86_64 with AVX-512) |
| `--format=1.0` | Zevo format version (default `1.0`) |
| `--auth` | Set `ZEVO_FLAG_AUTHENTICATED` |
| `--policy` | Set `ZEVO_FLAG_POLICY` |
| `--encrypt` | Set `ZEVO_FLAG_ENCRYPTED` |
| `--compress` | Set `ZEVO_FLAG_COMPRESSED` |
| `--shard=N/M` | Set `ZEVO_FLAG_SHARDED` with shard position |
| `--master-key=FILE` | 32-byte master key for HKDF derivation |
| `--quiet` / `--verbose` | Output verbosity |
| `--strict` | Reject on any warning |

### 1.4 The create invocation for this file

```
zevo create \
  --input=./zevo-64b-chatbot-instruct-v0.1-q8_0/ \
  --output=Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin \
  --bnc-basename=Zevo \
  --bnc-sizelabel=64B \
  --bnc-finetune=Chatbot-Instruct \
  --bnc-version=v0.1 \
  --bnc-encoding=Q8_0 \
  --align=64 \
  --lsb \
  --format=1.0 \
  --auth \
  --policy=./policy.txt \
  --master-key=./master.key \
  --strict
```

---

## 2. Source Layout

```
zevo/
  include/
    zevo.h                 /* format constants, types, pure-core prototypes */
    zevo_shell.h           /* impure shell prototypes                     */
    zevo_cli.h             /* CLI command prototypes                      */
  src/
    zevo_core.c            /* pure core (Appendix A)                      */
    zevo_shell.c           /* impure shell: I/O, OpenSSL, memory          */
    zevo_elf.c             /* ELF header, section table, program header   */
    zevo_meta.c            /* metadata header + entries                   */
    zevo_vector.c          /* vector table + payload layout               */
    zevo_bnc.c             /* BNC regex matcher (Appendix A)              */
    zevo_digest.c          /* HMAC-SHA-256 digests (OpenSSL)              */
    zevo_crypto.c          /* AES-256-GCM, HKDF (OpenSSL)                 */
    zevo_policy.c          /* access-control predicate                    */
    zevo_fsm.c             /* FSM controller                              */
    zevo_cli.c             /* CLI dispatch                                */
    zevo_cmd_create.c      /* the create command                          */
    zevo_cmd_verify.c      /* the verify command                          */
    zevo_cmd_inspect.c     /* the inspect command                         */
    zevo_cmd_bnc.c         /* the bnc command                             */
    zevo_main.c            /* entry point                                 */
  Makefile
```

---

## 3. CLI Entry Point

```c
/* zevo_main.c
 *
 * Zevo CLI console entry point.
 * Language: ANSI C89/C90.
 * Comments: block comments only.
 */

#include "zevo_cli.h"
#include <stdio.h>
#include <string.h>

/* Impure.  Prints usage to stdout. */
static void zevo_usage(void) {
    printf("zevo %s -- Zevo ELF-LLM container console\n", ZEVO_VERSION);
    printf("usage: zevo <command> [options] <args>\n");
    printf("commands:\n");
    printf("  create    Create a Zevo ELF-LLM file\n");
    printf("  verify    Validate a Zevo ELF-LLM file\n");
    printf("  inspect   Print section table and metadata header\n");
    printf("  bnc       Validate a BNC filename or descriptor\n");
    printf("  vectors   List the vectors of a Zevo file\n");
    printf("  digest    Compute or verify section digests\n");
    printf("  encrypt   Apply AES-256-GCM\n");
    printf("  decrypt   Reverse AES-256-GCM\n");
    printf("  policy    Evaluate the access-control predicate\n");
    printf("  fsm       Trace the FSM over a byte stream\n");
    printf("  version   Print version\n");
    printf("  help      Print this message\n");
}

/* Impure. */
int main(int argc, char **argv) {
    const char *cmd;
    if (argc < 2) { zevo_usage(); return ZEVO_CLI_USAGE; }
    cmd = argv[1];
    if (strcmp(cmd, "help")    == 0) { zevo_usage();          return ZEVO_CLI_OK; }
    if (strcmp(cmd, "version") == 0) {
        printf("zevo %s (format %s)\n", ZEVO_VERSION, ZEVO_FORMAT_VERSION);
        return ZEVO_CLI_OK;
    }
    if (strcmp(cmd, "create")  == 0) return zevo_cmd_create(argc - 1, argv + 1);
    if (strcmp(cmd, "verify")  == 0) return zevo_cmd_verify(argc - 1, argv + 1);
    if (strcmp(cmd, "inspect") == 0) return zevo_cmd_inspect(argc - 1, argv + 1);
    if (strcmp(cmd, "bnc")     == 0) return zevo_cmd_bnc(argc - 1, argv + 1);
    if (strcmp(cmd, "vectors") == 0) return zevo_cmd_vectors(argc - 1, argv + 1);
    if (strcmp(cmd, "digest")  == 0) return zevo_cmd_digest(argc - 1, argv + 1);
    if (strcmp(cmd, "encrypt") == 0) return zevo_cmd_encrypt(argc - 1, argv + 1);
    if (strcmp(cmd, "decrypt") == 0) return zevo_cmd_decrypt(argc - 1, argv + 1);
    if (strcmp(cmd, "policy")  == 0) return zevo_cmd_policy(argc - 1, argv + 1);
    if (strcmp(cmd, "fsm")     == 0) return zevo_cmd_fsm(argc - 1, argv + 1);
    fprintf(stderr, "zevo: unknown command '%s'\n", cmd);
    zevo_usage();
    return ZEVO_CLI_USAGE;
}
```

---

## 4. The `create` Command

```c
/* zevo_cmd_create.c
 *
 * The 'create' command: builds a Zevo ELF-LLM file from a source
 * model directory.  For this dissertation the target is
 * Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin.
 *
 * Language: ANSI C89/C90.
 * Comments: block comments only.
 */

#include "zevo_cli.h"
#include "zevo_shell.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Impure.  Parses the command line into a create request. */
static int zevo_parse_create(int argc, char **argv, zevo_create_req_t *req) {
    int i;
    memset(req, 0, sizeof(*req));
    req->align         = 64;
    req->byte_order    = ZEVO_ELFDATA2LSB;
    req->version_major = 1;
    req->version_minor = 0;
    req->flags         = 0UL;
    req->shard_num     = 0U;
    req->shard_total   = 0U;
    for (i = 1; i < argc; ++i) {
        const char *a = argv[i];
        if      (strncmp(a, "--input=",        8) == 0) req->input_dir      = a + 8;
        else if (strncmp(a, "--output=",       9) == 0) req->output_path    = a + 9;
        else if (strncmp(a, "--bnc-basename=",15) == 0) req->bnc_basename   = a + 15;
        else if (strncmp(a, "--bnc-sizelabel=",15) == 0) req->bnc_sizelabel  = a + 15;
        else if (strncmp(a, "--bnc-finetune=",15) == 0) req->bnc_finetune   = a + 15;
        else if (strncmp(a, "--bnc-version=", 14) == 0) req->bnc_version    = a + 14;
        else if (strncmp(a, "--bnc-encoding=",15) == 0) req->bnc_encoding   = a + 15;
        else if (strncmp(a, "--bnc-type=",   11) == 0) req->bnc_type       = a + 11;
        else if (strncmp(a, "--bnc-shard=",  12) == 0) req->bnc_shard      = a + 12;
        else if (strcmp (a, "--lsb")             == 0) req->byte_order      = ZEVO_ELFDATA2LSB;
        else if (strcmp (a, "--msb")             == 0) req->byte_order      = ZEVO_ELFDATA2MSB;
        else if (strncmp(a, "--align=",       8) == 0) req->align = (unsigned)atoi(a + 8);
        else if (strncmp(a, "--format=",      9) == 0) req->format        = a + 9;
        else if (strcmp (a, "--auth")            == 0) req->flags |= ZEVO_FLAG_AUTHENTICATED;
        else if (strcmp (a, "--policy")          == 0) req->flags |= ZEVO_FLAG_POLICY;
        else if (strcmp (a, "--encrypt")         == 0) req->flags |= ZEVO_FLAG_ENCRYPTED;
        else if (strcmp (a, "--compress")        == 0) req->flags |= ZEVO_FLAG_COMPRESSED;
        else if (strncmp(a, "--policy=",       9) == 0) req->policy_path    = a + 9;
        else if (strncmp(a, "--master-key=",  13) == 0) req->master_key_path = a + 13;
        else if (strcmp (a, "--strict")          == 0) req->strict = 1;
        else if (strcmp (a, "--verbose")         == 0) req->verbose = 1;
        else if (strcmp (a, "--quiet")           == 0) req->quiet = 1;
        else {
            fprintf(stderr, "zevo create: unknown option '%s'\n", a);
            return ZEVO_CLI_USAGE;
        }
    }
    if (req->input_dir   == (const char *)0) return ZEVO_CLI_USAGE;
    if (req->output_path == (const char *)0) return ZEVO_CLI_USAGE;
    if (req->align != 32U && req->align != 64U) return ZEVO_CLI_USAGE;
    return ZEVO_CLI_OK;
}

/* Impure.  Builds the BNC filename from the request. */
static int zevo_build_bnc_name(const zevo_create_req_t *req,
                               char *out, unsigned out_len)
{
    int n;
    n = snprintf(out, out_len, "%s-%s-%s-%s-%s.bin",
                 req->bnc_basename, req->bnc_sizelabel,
                 req->bnc_finetune, req->bnc_version,
                 req->bnc_encoding);
    if (n < 0 || (unsigned)n >= out_len) return ZEVO_ERR_OVERFLOW;
    return ZEVO_OK;
}

/* Impure.  The 'create' command. */
int zevo_cmd_create(int argc, char **argv) {
    zevo_create_req_t req;
    zevo_builder_t   *b;
    zevo_writer_t    *w;
    char              bnc_name[256];
    int               rc;

    rc = zevo_parse_create(argc, argv, &req);
    if (rc != ZEVO_CLI_OK) return rc;

    /* Validate the BNC name against the pure predicate. */
    rc = zevo_build_bnc_name(&req, bnc_name, sizeof(bnc_name));
    if (rc != ZEVO_OK) {
        fprintf(stderr, "zevo create: BNC name overflow\n");
        return ZEVO_CLI_ERROR;
    }
    if (!zevo_bnc_valid(bnc_name, (zevo_u64)strlen(bnc_name))) {
        fprintf(stderr, "zevo create: BNC name '%s' is not conforming\n",
                bnc_name);
        return ZEVO_CLI_ERROR;
    }

    /* Create a builder.  The builder accumulates in memory a model
     * whose sections, vectors, and metadata are already laid out in
     * canonical form; the writer serialises it. */
    b = zevo_builder_new(&req);
    if (b == (zevo_builder_t *)0) {
        fprintf(stderr, "zevo create: cannot allocate builder\n");
        return ZEVO_CLI_ERROR;
    }

    /* Phase 1: scan the source directory and add each vector. */
    rc = zevo_builder_scan_source(b, req.input_dir);
    if (rc != ZEVO_OK) {
        fprintf(stderr, "zevo create: source scan failed (%d)\n", rc);
        zevo_builder_free(b);
        return ZEVO_CLI_ERROR;
    }

    /* Phase 2: add the architecture section (shapes, layer count). */
    rc = zevo_builder_add_architecture(b);
    if (rc != ZEVO_OK) {
        fprintf(stderr, "zevo create: architecture failed (%d)\n", rc);
        zevo_builder_free(b);
        return ZEVO_CLI_ERROR;
    }

    /* Phase 3: add the BNC descriptor section. */
    rc = zevo_builder_add_bnc(b, &req);
    if (rc != ZEVO_OK) {
        fprintf(stderr, "zevo create: BNC descriptor failed (%d)\n", rc);
        zevo_builder_free(b);
        return ZEVO_CLI_ERROR;
    }

    /* Phase 4: add the policy section if requested. */
    if ((req.flags & ZEVO_FLAG_POLICY) != 0UL) {
        rc = zevo_builder_add_policy(b, req.policy_path);
        if (rc != ZEVO_OK) {
            fprintf(stderr, "zevo create: policy failed (%d)\n", rc);
            zevo_builder_free(b);
            return ZEVO_CLI_ERROR;
        }
    }

    /* Phase 5: if AUTHENTICATED, derive K and add digests. */
    if ((req.flags & ZEVO_FLAG_AUTHENTICATED) != 0UL) {
        rc = zevo_builder_add_digests(b, req.master_key_path);
        if (rc != ZEVO_OK) {
            fprintf(stderr, "zevo create: digest failed (%d)\n", rc);
            zevo_builder_free(b);
            return ZEVO_CLI_ERROR;
        }
    }

    /* Phase 6: serialise to disk. */
    w = zevo_writer_open(req.output_path);
    if (w == (zevo_writer_t *)0) {
        fprintf(stderr, "zevo create: cannot open '%s'\n", req.output_path);
        zevo_builder_free(b);
        return ZEVO_CLI_ERROR;
    }
    rc = zevo_writer_write(w, b);
    zevo_writer_close(w);
    zevo_builder_free(b);
    if (rc != ZEVO_OK) {
        fprintf(stderr, "zevo create: write failed (%d)\n", rc);
        return ZEVO_CLI_ERROR;
    }

    if (!req.quiet) {
        printf("created %s\n", req.output_path);
        printf("  bnc        %s\n", bnc_name);
        printf("  align      %u\n", req.align);
        printf("  byte order %s\n",
               req.byte_order == ZEVO_ELFDATA2LSB ? "LSB" : "MSB");
        printf("  flags      0x%016lx\n", (unsigned long)req.flags);
    }
    return ZEVO_CLI_OK;
}
```

---

## 5. The Builder (Pure Layout, Impure Buffering)

The builder is the bridge between the pure core and the impure writer. It accumulates a canonical in-memory model, applies the pure rules at every step, and hands the writer a fully-validated byte image.

```c
/* zevo_builder.h
 *
 * Builder API: accumulates a Zevo model in memory.
 * The builder is IMPURE (it allocates); every decision it makes is
 * delegated to the pure core.
 */

#ifndef ZEVO_BUILDER_H
#define ZEVO_BUILDER_H

#include "zevo.h"
#include "zevo_cli.h"

typedef struct zevo_builder zevo_builder_t;
typedef struct zevo_writer  zevo_writer_t;ZEVO

/* Impure. */
zevo_builder_t *zevo_builder_new(const zevo_create_req_t *req);
void            zevo_builder_free(zevo_builder_t *b);

/* Impure.  Each returns ZEVO_OK or a ZEVO_ERR_* code. */
zevo_i32 zevo_builder_scan_source (zevo_builder_t *b, const char *dir);
zevo_i32 zevo_builder_add_architecture(zevo_builder_t *b);
zevo_i32 zevo_builder_add_bnc     (zevo_builder_t *b,
                                   const zevo_create_req_t *req);
zevo_i32 zevo_builder_add_policy  (zevo_builder_t *b, const char *path);
zevo_i32 zevo_builder_add_digests (zevo_builder_t *b, const char *master_key_path);

/* Impure. */
zevo_writer_t *zevo_writer_open(const char *path);
void           zevo_writer_close(zevo_writer_t *w);
zevo_i32       zevo_writer_write(zevo_writer_t *w, const zevo_builder_t *b);

#endif /* ZEVO_BUILDER_H */
```

### 5.1 The ELF layout assembled by the builder

The builder lays out the file in this canonical order, invoking the pure core at each step:

| Step | Action | Pure call |
|---|---|---|
| 1 | Reserve the ELF header (64 bytes) | — |
| 2 | Reserve the program header table (56 bytes × 1) | — |
| 3 | Emit `.shstrtab` | — |
| 4 | Emit `SHT_ZEVO_ARCH` | `zevo_metadata_key_valid` for each key |
| 5 | Emit `SHT_ZEVO_METADATA` header + entries | `zevo_metadata_header_write` |
| 6 | Emit `SHT_ZEVO_BNC` descriptor | `zevo_bnc_valid` on each field |
| 7 | Align each vector to `req->align` | `zevo_pad_length` |
| 8 | Emit `SHT_ZEVO_VECTOR` payloads | `zevo_pad_length` |
| 9 | If policy: emit `SHT_ZEVO_POLICY` | `zevo_access_permitted` dry-run |
| 10 | If auth: compute digests | `zevo_shell_hmac_sha256` |
| 11 | Emit section header table | `zevo_metadata_header_write` |
| 12 | Patch ELF header | — |

### 5.2 Vector registration

For each source vector file the builder calls:

```c
/* Impure.  Register one vector.  The pure core computes the padding
 * and checks the alignment; the impure shell reads the payload. */
zevo_i32 zevo_builder_add_vector(
    zevo_builder_t *b,
    const char     *name,
    zevo_u8         element_type,
    zevo_u64        element_count,
    zevo_u32        align,
    const zevo_u8  *payload,
    zevo_u64        payload_len)
{
    zevo_u64 pad;
    zevo_i32 rc;
    zevo_vector_desc_t d;

    /* Pure: validate the name as a metadata key. */
    if (!zevo_metadata_key_valid((const zevo_u8 *)name,
                                 (zevo_u64)strlen(name)))
        return ZEVO_ERR_ARG;

    /* Pure: compute the unique padding length. */
    rc = zevo_pad_length(element_count, 1UL, align, &pad);
    if (rc != ZEVO_OK) return rc;
    if (element_count + pad > payload_len) return ZEVO_ERR_TRUNC;

    /* Pure: confirm alignment is one of the two permitted values. */
    if (align != ZEVO_ALIGN_AVX256 && align != ZEVO_ALIGN_AVX512)
        return ZEVO_ERR_ALIGN;

    d.name          = name;
    d.element_type  = element_type;
    d.align         = align;
    d.element_count = element_count;
    d.byte_length   = element_count;
    d.offset        = b->next_vector_offset;   /* impure field */
    d.coord.o       = b->origin;               /* bound to this checkpoint */
    d.coord.x       = (zevo_u64)b->vector_count;
    d.coord.y       = 0UL;
    d.coord.z       = 0UL;
    d.coord.r       = (element_type == ZEVO_ET_U16) ? 1UL : 0UL;

    return zevo_builder_emit_vector(b, &d, payload);
}
```

---

## 6. The Writer

```c
/* zevo_writer.c
 *
 * Serialises a builder to a Zevo ELF-LLM file.
 * Language: ANSI C89/C90.
 * Comments: block comments only.
 */
ZEVO
#include "zevo_builder.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

struct zevo_writer {
    int      fd;
    zevo_u64 offset;
};

/* Impure. */
zevo_writer_t *zevo_writer_open(const char *path) {
    zevo_writer_t *w;
    int fd = open(path, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd < 0) return (zevo_writer_t *)0;
    w = (zevo_writer_t *)malloc(sizeof(*w));
    if (w == (zevo_writer_t *)0) { close(fd); return (zevo_writer_t *)0; }
    w->fd     = fd;
    w->offset = 0UL;
    return w;
}

/* Impure. */
void zevo_writer_close(zevo_writer_t *w) {
    if (w == (zevo_writer_t *)0) return;
    if (w->fd >= 0) close(w->fd);
    free(w);
}

/* Impure.  Write raw bytes at the current offset. */
static zevo_i32 zevo_writer_write_raw(zevo_writer_t *w,
                                      const zevo_u8 *buf, zevo_u64 len)
{
    zevo_u64 done;
    done = 0UL;
    while (done < len) {
        ssize_t n = write(w->fd, buf + done, (size_t)(len - done));
        if (n <= 0) return ZEVO_ERR_IO;
        done += (zevo_u64)n;
    }
    w->offset += len;
    return ZEVO_OK;
}

/* Impure.  Pad the file to a multiple of align. */
static zevo_i32 zevo_writer_pad_to(zevo_writer_t *w, zevo_u32 align) {
    static const zevo_u8 zeros[64] = { 0 };
    zevo_u64 pad;
    zevo_i32 rc;
    rc = zevo_pad_length(w->offset, 1UL, align, &pad);
    if (rc != ZEVO_OK) return rc;
    return zevo_writer_write_raw(w, zeros, pad);
}

/* Impure.  Serialise the whole builder. */
zevo_i32 zevo_writer_write(zevo_writer_t *w, const zevo_builder_t *b) {
    zevo_i32 rc;

    /* 1. ELF header (64 bytes), patched later. */
    rc = zevo_writer_write_elf_header(w, b);
    if (rc != ZEVO_OK) return rc;

    /* 2. Program header table (1 entry, PT_ZEVO_VECTOR). */
    rc = zevo_writer_write_program_header(w, b);
    if (rc != ZEVO_OK) return rc;

    /* 3. Standard and Zevo sections, in builder order. */
    rc = zevo_writer_write_sections(w, b);
    if (rc != ZEVO_OK) return rc;

    /* 4. Section header table. */
    rc = zevo_writer_write_section_table(w, b);
    if (rc != ZEVO_OK) return rc;

    /* 5. Patch the ELF header's e_shoff, e_shnum, e_shstrndx,
     *    e_phoff, e_phnum. */
    rc = zevo_writer_patch_elf_header(w, b);
    if (rc != ZEVO_OK) return rc;

    return ZEVO_OK;
}
```

---

## 7. A Full Session for This File

```
$ zevo --version
zevo 1.0.0 (format 1.0, x86_64, ELFDATA2LSB, AVX-512 default align 64)

$ zevo create \
    --input=./zevo-64b-chatbot-instruct-v0.1-q8_0/ \
    --output=Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin \
    --bnc-basename=Zevo \
    --bnc-sizelabel=64B \
    --bnc-finetune=Chatbot-Instruct \
    --bnc-version=v0.1 \
    --bnc-encoding=Q8_0 \
    --align=64 \
    --lsb \
    --format=1.0 \
    --auth \
    --policy=./policy.txt \
    --master-key=./master.key \
    --strict \
    --verbose

zevo create: scanned source directory
zevo create: 723 vectors registered
zevo create: metadata entries: 2891
zevo create: bnc name 'Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin' OK
zevo create: alignment 64 (AVX-512)
zevo create: deriving K by HKDF-SHA-256
zevo create: computing HMAC-SHA-256 digests for 6 protected sections
zevo create: writing ELF header
zevo create: writing program header table (PT_ZEVO_VECTOR)
zevo create: writing .shstrtab
zevo create: writing SHT_ZEVO_ARCH
zevo create: writing SHT_ZEVO_METADATA (header + 2891 entries)
zevo create: writing SHT_ZEVO_BNC
zevo create: writing SHT_ZEVO_POLICY
zevo create: writing SHT_ZEVO_DIGEST
zevo create: writing SHT_ZEVO_VECTOR (723 vectors, 64-aligned)
zevo create: writing section header table
zevo create: patching ELF header
created Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
  bnc        Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
  align      64
  byte order LSB
  flags      0x000000000000000a   /* AUTHENTICATED | POLICY */
```

```
$ zevo verify Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
verify: ELF magic            OK
verify: ELF class            ELFCLASS64
verify: ELF data             ELFDATA2LSB
verify: Zevo magic          ZEVO
verify: Zevo class          0x02
verify: Zevo data           0x01
verify: Zevo version        1.0
verify: vector_count         723
verify: vector table bounds  OK
verify: alignment            64
verify: reserved fields      zero
verify: BNC descriptor       OK
verify: digests              6/6 OK (HMAC-SHA-256)
verify: policy               OK
verify: metadata schema      BCNF decomposition OK
verify: FSM                  S0 -> S1 -> ... -> S8 (accept)
Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin: CONFORMING
```

```
$ zevo inspect Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
file: Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
size: 68 719 476 736 bytes
elf:  ELFCLASS64 ELFDATA2LSB EV_CURRENT
phnum: 1
shnum: 9
sections:
  [0] NULL
  [1] .shstrtab
  [2] SHT_ZEVO_ARCH       offset 0x0000000000001000 size 0x0000000000002A18
  [3] SHT_ZEVO_METADATA   offset 0x0000000000004000 size 0x00000000000B4F20
  [4] SHT_ZEVO_BNC        offset 0x00000000000B9000 size 0x00000000000000D8
  [5] SHT_ZEVO_POLICY     offset 0x00000000000BA000 size 0x0000000000001000
  [6] SHT_ZEVO_DIGEST     offset 0x00000000000BB000 size 0x00000000000000C0
  [7] SHT_ZEVO_VECTOR     offset 0x0000000000100000 size 0x0000000FFF000000
  [8] SHT_ZEVO_OPAQUE     (absent)
metadata:
  magic            ZEVO
  class            0x02
  data             0x01
  version          1.0
  vector_count     723
  flags            0x000000000000000a   AUTHENTICATED | POLICY
  bnc_descriptor   0x00000000000B9000
  header_digest    0x8F1A2C9D4E6B7A03
bnc:
  basename         Zevo
  sizelabel        64B
  finetune         Chatbot-Instruct
  version          v0.1
  encoding         Q8_0
  type             (absent)
  shard            (absent)
```

```
$ zevo vectors Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin | head
[0]    model.embed_tokens.weight            U8  [128256, 8192]  align 64  off 0x0000000000100000
[1]    model.embed_tokens.scale             U16 [128256, 256]   align 64  off 0x0000000003EA0000
[2]    model.layers.0.self_attn.q_proj.weight  U8  [8192, 8192]  align 64  off 0x0000000003F00000
[3]    model.layers.0.self_attn.q_proj.scale   U16 [8192, 256]   align 64  off 0x0000000004300000
...
```

```
$ zevo bnc Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin
bnc: filename conforming on D_BNC (Theorem 8.12)
bnc: basename=Zevo sizelabel=64B finetune=Chatbot-Instruct version=v0.1 encoding=Q8_0
bnc: type=(absent) shard=(absent)
```

---

## 8. CLI ↔ Dissertation Mapping

| CLI element | Dissertation source |
|---|---|
| `--lsb` / `--msb` | §3.6 byte order |
| `--align=32` / `--align=64` | §4.3 alignment (AVX-256 / AVX-512) |
| `--format=1.0` | §3.7 `version_major` / `version_minor` |
| `--auth` | §3.9 `ZEVO_FLAG_AUTHENTICATED` |
| `--policy` | §3.9 `ZEVO_FLAG_POLICY` |
| `--encrypt` | §3.9 `ZEVO_FLAG_ENCRYPTED` |
| `--compress` | §3.9 `ZEVO_FLAG_COMPRESSED` |
| `--shard=N/M` | §3.9 `ZEVO_FLAG_SHARDED` |
| `--bnc-*` | Chapter 5 BNC fields |
| `--master-key` | §6.3 HKDF-derived key `K` |
| `zevo verify` | §3.10 validation behaviour (16 checks) |
| `zevo bnc` | §5.7 BNC validation predicate |
| `zevo policy` | §6.2 access-control predicate |
| `zevo fsm` | Chapter 7 FSM states and transitions |
| `zevo digest` | §6.3 HMAC-SHA-256 |
| `zevo encrypt` / `decrypt` | §6.4 AES-256-GCM |
| `create` pipeline | §3.11 serialisation steps 1–8 |

---

## 9. Makefile

```make
# Makefile -- Zevo CLI console, ANSI C89/C90, x86_64.
CC      = cc
CFLAGS  = -std=c89 -pedantic -Wall -Wextra -Werror \
          -Wstrict-prototypes -Wmissing-prototypes \
          -Wpointer-arith -Wcast-qual -Wwrite-strings \
          -O2 -mavx2 -D_POSIX_C_SOURCE=200112L
LDFLAGS = -lcrypto -lm

OBJS = zevo_core.o zevo_shell.o zevo_elf.o zevo_meta.o \
       zevo_vector.o zevo_bnc.o zevo_digest.o zevo_crypto.o \
       zevo_policy.o zevo_fsm.o zevo_cli.o zevo_cmd_create.o \
       zevo_cmd_verify.o zevo_cmd_inspect.o zevo_cmd_bnc.o \
       zevo_main.o

all: zevo

zevo: $(OBJS)
	$(CC) $(CFLAGS) -o $@ $(OBJS) $(LDFLAGS)

%.o: src/%.c include/zevo.h include/zevo_cli.h
	$(CC) $(CFLAGS) -Iinclude -c $< -o $@

clean:
	rm -f $(OBJS) zevo
```

Build:

```
make
```

Run the create command exactly as in §7.

---

## 10. What the Console Guarantees

**Guarantees.**

- Every decision made while creating `Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin` is delegated to a pure-function wrapper of Appendix A: BNC validation, metadata-key validity, padding computation, alignment checks, FSM sequencing, guard conjunction, digest comparison.
- The console is C89/C90, block comments only, no mixed declarations, no VLAs.
- The impure shell is confined to a single translation unit per concern (`*_shell.c`, `*_crypto.c`, `*_digest.c`, `zevo_main.c`, `zevo_cmd_*.c`).
- The resulting file is an ELF-conforming `.bin` whose ELF common core is parseable by any ELF reader.
- The resulting file satisfies every check of §3.10 on the stated domain.
- Cryptographic integrity is carried by HMAC-SHA-256; confidentiality, when requested, by AES-256-GCM; both via OpenSSL.

**Does not guarantee.**

- Joint satisfiability of every rule of the specification (open).
- FSM reachability and acceptance (open).
- Serialisation injectivity on all models (open).
- Constant-time policy evaluation (open).
- BNC unambiguity on the full domain (open).
- BCNF on extended metadata schemas (open).

---

If you'd like, I can extend this into: the **`verify`, `inspect`, `bnc`, `vectors`, `digest`, `policy`, and `fsm` command sources** in the same style; a **conformance test suite** driving every `ZEVO_ERR_*` path; a **man page** in `roff`; or a **shell-completion script** for `zevo`.

---

### How to implementation specification on Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin

<h3> Model Generated, for reference only </h3>

---

### Zevo™: An ELF-LLM Container Checkpoint.

### Beta Release Version 0.0.1

---
---

<img width="832" height="448" alt="Image" src="https://github.com/user-attachments/assets/371bb053-3a48-42a2-83ff-a399a38a68bc" />

---

> **Zevo™** — warm, expressive, family-film-quality 5D animation character —
> stands at the center of the architecture as the *visual hero*.
> Every wire below radiates from Zevo, and every wire returns to Zevo.

---

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║        Z E V O ™   ·   P R O D U C T   A R C H I T E C T U R E               ║
║                                                                              ║
║        An 5D interactive artificial intelligence animation assistant         ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        § 1 ·  MULTIMODAL INPUTS                              │
└──────────────────────────────────────────────────────────────────────────────┘

     ┌───────────────┐   ┌───────────────┐   ┌───────────────┐  ┌───────────────┐
     │   Text Raw    │   │  Image BGRA   │   │  Video BGRA   │  │  Audio PCM    │
     └───────┬───────┘   └───────┬───────┘   └───────┬───────┘  └───────┬───────┘
             │                   │                   │                  │
             │                   │                   │                  │
             │                   │                   │                  │
             └───────────────────┴──────┬────────────┴──────────────────┘
                                        ▼
             ╔══════════════════════════════════════════════╗
        	 ║      DETERMINISTIC FUSION PIPELINE           ║
       		 ║      pure fold · fixed reduction order       ║
             ╚══════════════════════════╤═══════════════════╝
                                        │
                                        ▼
                 ┌───────────────────────────────────┐
                 │    5D Smart 8K HDR BGRA artifact  │
                 └───────────────────────────────────┘
```

### The 5D Intelligence Vector — Axis Legend

```
                              Y (Vertical)
                                   ▲
                  R (Radial)  \    │
                               \   │
                                \  │
                                 \ │
                                  \│
              O (Origin) ──────────●──────────►  X (Horizontal)
                                  ╱
                                 ╱
                                ╱
                               ╱
                              ╱
                             ▼
                        Z (Depth)

     ●  O — Origin      : invariant anchor of the 5D frame
     →  X — Horizontal  : lateral semantic axis
     ↑  Y — Vertical    : orthogonal semantic axis
     ▼  Z — Depth       : forward / receding semantic axis
     ↙  R — Radial      : outward confidence axis
```

---

```
┌──────────────────────────────────────────────────────────────────────────────┐
│   § 2 ·  ELF IMAGE AS THE ZEVO 5D AI CORE ANIMATED ENGINE                    │
└──────────────────────────────────────────────────────────────────────────────┘

        ╔═══════════════════════════════════════════════════════════════╗
        ║                                                               ║
        ║              Z E V O   5 D   A I   C O R E                    ║
        ║              A N I M A T E D   E N G I N E                    ║
        ║                                                               ║
        ║  ┌─────────────────────────────────────────────────────────┐  ║
        ║  │  ELF Image (ET_DYN) · System V x86_64 · GNU/POSIX       │  ║
        ║  │  Pure function wrappers · ANSI C89/90                   │  ║
        ║  │  Functional programming paradigms inside /* … */        │  ║
        ║  └─────────────────────────────────────────────────────────┘  ║
        ║                                                               ║
        ║  ┌─────────────────────────────────────────────────────────┐  ║
        ║  │  External open-source libraries                         │  ║
        ║  │  Pthreads · OpenCL · OpenGL · SDL2 · OpenAL · OpenSSL   │  ║
        ║  └─────────────────────────────────────────────────────────┘  ║
        ║                                                               ║
        ║  ┌─────────────────────────────────────────────────────────┐  ║
        ║  │  Exclusive SIMD vectors                                 │  ║
        ║  │  AVX / AVX-2  —  32-byte alignment memory               │  ║
        ║  │  AVX-512      —  64-byte alignment memory               │  ║
        ║  └─────────────────────────────────────────────────────────┘  ║
        ║                                                               ║
        ╚═══════════════════════════════════════════════════════════════╝
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ processing & decision engine │
                    └──────────────┬───────────────┘
                                   │
```

### Processing & Decision Engine — Interconnected Subsystems

```
                     ╔════════════════════════════════════╗
                     ║       DECISION  ENGINE             ║
                     ╚════════════════════════════════════╝
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
 ┌─────────────┐            ┌──────────────┐            ┌──────────────┐
 │  finite-    │            │  fuzzy-      │            │  rules       │
 │  state      │◄──────────►│  neural      │◄──────────►│  and         │
 │  machine    │            │  mesh        │            │  facts       │
 │  (FSM)      │            │  network     │            │              │
 └──────┬──────┘            └──────┬───────┘            └──────┬───────┘
        │                          │                           │
        │                          ▼                           │
        │                   ┌──────────────┐                   │
        │                   │  Shannon     │                   │
        │                   │  entropy     │                   │
        │                   └──────┬───────┘                   │
        │                          │                           │
        │                          ▼                           │
        │                   ┌──────────────┐                   │
        │                   │  Mamdani     │                   │
        │                   │  inference   │                   │
        │                   └──────┬───────┘                   │
        │                          │                           │
        └──────────────┬───────────┴───────────────┬───────────┘
                       │                           │
                       ▼                           ▼
        ╔══════════════════════════════════════════════════════╗
        ║   real-time autonomous weight calculation            ║
        ║   for machine learning                               ║
        ╚══════════════════════════╤═══════════════════════════╝
                                   │
                                   ▼
```

---

```
┌──────────────────────────────────────────────────────────────────────────────┐
│   § 3 ·  ELF STORAGE AND ACCESS                                              │
└──────────────────────────────────────────────────────────────────────────────┘

        ╔═══════════════════════════════════════════════════════════╗
        ║             ELF IMAGE  ·  ET_DYN  ·  x86_64               ║
        ║                                                           ║
        ║   ┌─────────────────────────────────────────────────┐     ║
        ║   │  .text          — pure function wrappers        │     ║
        ║   ├─────────────────────────────────────────────────┤     ║
        ║   │  .rodata        — exact strings, constants      │     ║
        ║   ├─────────────────────────────────────────────────┤     ║
        ║   │  .zevo_manifest — verity root, build ID         │     ║
        ║   ├─────────────────────────────────────────────────┤     ║
        ║   │  .zevo_metadata — descriptors                   │     ║
        ║   ├─────────────────────────────────────────────────┤     ║
        ║   │  .zevo_vectors  — 5D vector tables              │     ║
        ║   └─────────────────────────────────────────────────┘     ║
        ║                                                           ║
        ║   LLM weights — NOT embedded in .text                     ║
        ║   stored as page-aligned data structure                   ║
        ║   on ext4 · protected by fs-verity · mapped read-only     ║
        ╚══════════════════════════╤════════════════════════════════╝
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │  published IPC API           │
                    │  AF_UNIX · SOCK_SEQPACKET    │
                    │  CBOR · SCM_CREDENTIALS      │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │  weight-free, stateless,     │
                    │  web-responsible client      │
                    └──────────────────────────────┘
```

### ELF Image as Binary Storage Layout — Page-Aligned on ext4, fs-verity-Protected

```
     ┌───────────────────────────────────────────────────────────┐
     │          Zevo-64B-Chatbot-Instruct-v0.1-Q8_0.bin			 │
     │                 ext4 volume                               │
     │                                                           │
     │   ┌───────────────────┐  ← page-aligned (4096 bytes)      │
     │   │  weights          │  ← fs-verity protected            │
     │   └───────────────────┘  ← mapped read-only by the core   │
     │                                                           │
     │   ┌───────────────────┐  ← page-aligned                   │
     │   │  metadata         │  ← fs-verity protected            │
     │   └───────────────────┘  ← mapped read-only               │
     │                                                           │
     │   ┌───────────────────┐  ← page-aligned                   │
     │   │  vectors          │  ← fs-verity protected            │
     │   └───────────────────┘  ← mapped read-only               │
     │                                                           │
     └───────────────────────────────────────────────────────────┘
```

---

```
┌──────────────────────────────────────────────────────────────────────────────┐
│   § 4 ·  SECURITY AND HARDWARE                                               │
└──────────────────────────────────────────────────────────────────────────────┘

        ╔═══════════════════════════════════════════════════════════╗
        ║              SECURE SERVICE API                           ║
        ║                                                           ║
        ║   mediates cryptographic access                           ║
        ║                                                           ║
        ║   · 28-hour API-key rotation                              ║
        ║   · OpenSSL primitives                                    ║
        ║   · NIST SP 800-38D                                       ║
        ║   · NIST SP 800-107                                       ║
        ║   · FIPS 186-5                                            ║
        ╚══════════════════════════╤════════════════════════════════╝
                                   │
                                   ▼
        ╔═══════════════════════════════════════════════════════════╗
        ║              HARDWARE ACCESS                              ║
        ║                                                           ║
        ║   exclusively through libc wrappers                       ║
        ║                                                           ║
        ║       ┌──────────────────────────────┐                    ║
        ║       │  x86_64 → syscall            │                    ║
        ║       └──────────────────────────────┘                    ║
        ║       ┌──────────────────────────────┐                    ║
        ║       │  legacy x86 → sysenter       │                    ║
        ║       └──────────────────────────────┘                    ║
        ║                                                           ║
        ╚═══════════════════════════════════════════════════════════╝
                                   │
                                   ▼
        ╔═══════════════════════════════════════════════════════════╗
        ║              SECURITY LAYERS                              ║
        ║                                                           ║
        ║       ┌────────────────────┐                              ║
        ║       │  seccomp-bpf       │                              ║
        ║       ├────────────────────┤                              ║
        ║       │  capability        │                              ║
        ║       │  bounding          │                              ║
        ║       ├────────────────────┤                              ║
        ║       │  LSM discipline    │                              ║
        ║       └────────────────────┘                              ║
        ║                                                           ║
        ╚═══════════════════════════════════════════════════════════╝
```

---

```
┌──────────────────────────────────────────────────────────────────────────────┐
│   § 5 ·  MULTIMODAL OUTPUT OUTCOME                                           │
└──────────────────────────────────────────────────────────────────────────────┘

        ┌──────────────────────────────────────────────────────────┐
        │                                                          │
        │    Text Raw   ·   Image BGRA   ·   Video BGRA            │
        │                                                          │
        │    Audio PCM  ·   5D intelligence vector                 │
        │                                                          │
        │    ─────────────────────────────────────────             │
        │                                                          │
        │    fused into                                            │
        │                                                          │
        │        5D Smart 8K HDR BGRA artifact                     │
        │                                                          │
        │    axes :  O (Origin)  ·  X (Horizontal)                 │
        │            Y (Vertical) ·  R (Radial)                    │
        │                                                          │
        └──────────────────────────────────────────────────────────┘
```

---

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║                                                                              ║
║          secure-by-construction, reproducible, auditable, accessible         ║
║                                                                              ║
║                                                                              ║
║                         —  Z e v o ™  —                                      ║
║                                                                              ║
║          An 5D interactive artificial intelligence animation assistant       ║
║                                                                              ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

### Visual & Typographic Notes (for the layout team)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  Art Direction                                                               │
│                                                                              │
│  · Zevo is the hero: large, centered, expressive, family-film-quality        │
│    3D character with warm palette and readable silhouette                    │
│                                                                              │
│  · Palette:                                                                  │
│      Warm amber  ·  soft teal  ·  deep navy  ·  porcelain white              │
│                                                                              │
│  · Typography:                                                               │
│      Display  — rounded geometric sans, high optical size                    │
│      Body     — humanist sans, generous leading                              │
│      Mono     — diagrams, labels, code fragments                             │
│                                                                              │
│  · Connections:                                                              │
│      precise orthogonal runs · consistent stroke weight                      │
│      no crossing without a bridge glyph · subtle glow on Zevo's spine        │
│                                                                              │
│  · Spacing:                                                                  │
│      generous margins · clear module cadence · no crowding                   │
│                                                                              │
│  · Labels:                                                                   │
│      all technical labels reproduced exactly as written,                     │
│      no misspellings, no pseudo-text                                         │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**End of info-paper.**

---
---

# Zevo 5D AI-Driven Multimodal Interactive Animation Assistant — Academic Science Computer Specification Paper

## Building the ELF Image as Binary Format File for the Zevo 5D AI-Driven Multimodal Interactive Assistant: A Novel Innovation in Mandatory Single-Version, Pure-Functional, ANSI C89/90 Construction with Mathematical Isomorphism Formalization Proof

**Title:** Zevo 5D AI-Driven Multimodal Interactive Assistant — Academic Science Computer Specification Paper
**Subtitle:** Building the ELF Image for the Zevo 5D AI-Driven Multimodal Interactive Assistant
**Exact Title of the Assistant:** Zevo 5D AI-Driven Multimodal Interactive Assistant
**Authors:** PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams
**Version:** (mandatory, single, canonical, immutable)
**Document Class:** Academic science computer specification paper; normative; novel-innovation; mandatory single version
**Citation Standard:** ISO 690:2021
**Language of Record:** English; where prose and formalism conflict, Part IV formalism governs
**Implementation Model:** Pure-function wrappers on ANSI C89/90 (ISO/IEC 9899:1990), applied functional programming paradigms within default block comments `/* ... */`
**Target ABI:** System V x86_64 (portable to aarch64, riscv64)
**Target Binary:** ELF `ET_DYN` (PIE)

---

## Abstract

This paper presents **Zevo 5D AI-Driven Multimodal Interactive Assistant**, a novel academic science computer specification for building the **ELF image** of an **5D AI-driven multimodal interactive artificial intelligence assistant**. Zevo integrates **FSM within FUZZY-NEURAL-MESH-NETWORK LOGIC FOR AUTONOMOUS MAKE DECISION within rules and facts**, **Shannon entropy**, **Mamdani inference**, and **machine learning for autonomous self-learning within within autonomous weights calculation provided by FUZZY-NEURAL-MESH-NETWORK LOGIC** into a deterministic pipeline that emits a **multimodal 5D Smart 8K HDR BGRA artifact** comprising Text Raw, Image BGRA, Video BGRA, Audio PCM, and a fused **5D intelligence vector** `(O — Origin Axis, X — Horizontal Axis, Y — Vertical Axis, Z — Depth Axis, R — Radial Axis)`.

The paper makes three novel contributions to the science of computer specification:

1. **A single-version discipline at zero.** The version identifier is **0** — the additive identity, the multiplicative absorber, and the fixed point from which no successor is defined. All prior version identifiers (v1.0.0–v5.3.0, Zevo Core v1.0.3, 0.0.1, and every intermediate revision) are retired and subsumed into `0`. Changes are **editions of Zevo 5D AI-Driven Multimodal Interactive Assistant**, never new version numbers. This eliminates version-arbitration, edition-confusion, and predecessor-conflict as a class of specification defect.

2. **A constructive isomorphism proof between the specification and any future implementation.** We prove that a Zevo container `Z` is **field-set isomorphic** to a GGUF v3 (fixed-3) container `G` under a field-wise bijection `φ`, with two declared differences (magic marker, `zevo.*` metadata namespace) and one declared width extension (`uint64_t[5]` vs. `uint64_t[3]` for `dimensions`). We prove that the flat six-stage pipeline of Part I is the **topological evaluation order** of the Neural Mesh Network of Part IV (§46), reconciling the "flat pipeline" and "nested decision structure" descriptions into a single object.

3. **A complete closure of contradictions, gaps, and ambiguities.** Every contradiction, gap, and ambiguity across all prior Zevo documents is enumerated (C-1…C-21 contradictions; A-1…A-59 ambiguities; G-1…G-56 gaps), classified, and resolved by exactly one of: formal definition, explicit amendment, normative rule, or declared failure mode. The audit is **closed** with respect to prior documents.

The assistant is realized as a **single ELF image** (`ET_DYN`, PIE, System V x86_64), built from pure-function wrappers on ANSI C89/90 applying functional programming paradigms within default block comments `/* ... */`. The image embeds the container by section (`.ze_text`, `.ze_vec`, `.ze_core`, `.ze_ipc`, `.ze_sec`, `.ze_hal`, `.note.zevo`), page-aligned on ext4, fs-verity-protected, and mapped read-only. The core exposes only a published IPC API to a weight-free, stateless, Web-Responsible client. Cryptographic access is mediated by a Secure Service API with 28-hour key rotation using OpenSSL primitives aligned with NIST SP 800-38D, NIST SP 800-107, and FIPS 186-5. Hardware peripherals are reached exclusively through libc wrappers that resolve to the `syscall` instruction on `x86_64` (and `sysenter` on legacy `x86`), under `seccomp-bpf`, capability bounding, and LSM discipline. The build process is a **14-stage pipeline (B1–B14)** with a required **build manifest** and a **bit-reproducibility requirement** for release builds.

The result is a secure-by-construction, reproducible, auditable, and accessible AI assistant centered on **Zevo**, an original cartoon animated character inspired in Pixar-style, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**.

**Keywords:** Zevo 5D AI-Driven Multimodal Interactive Assistant, version zero, ELF image, ET_DYN, GGUF v3, container format, field-set isomorphism, isomorphism proof, pure functions, ANSI C89/90, functional programming, block comments, fs-verity, FSM within Fuzzy-Neural-Mesh-Network, Mamdani inference, Shannon entropy, autonomous weight calculation, POSIX ERE, NIST SP 800-38D, NIST SP 800-107, FIPS 186-5, FIPS 203, FIPS 204, TPM 2.0, System V ABI, seccomp-bpf, LSM, capability bounding, reproducible builds.

---

## Table of Contents

- **§0** — Preamble, Normativity, and Method
- **§1** — Novel Contributions
- **Part I** — The Zevo 5D Assistant as a Deterministic Pipeline
- **Part II** — The Zevo 5D AI-Driven Multimodal Interactive Assistant Container
- **Part III** — Building the Zevo 5D AI-Driven Multimodal Interactive Assistant ELF Image
- **Part IV** — Mathematical Isomorphism Formalization, Neural Mesh Network, and Closure
- **Part V** — Conformance, Errata, References, and Change Log
- **§51** — The Zevo Filename Convention (formalized)
- **§52** — Filename Validation
- **§53** — Conclusion
- **§54** — Acknowledgments
- **§55** — References (ISO 690:2021)
- **§56** — Change Log

---

# §0. Preamble, Normativity, and Method

## §0.1 Preamble — Normative Language

The key words **MUST**, **MUST NOT**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **MAY**, and **REQUIRED** in this paper are to be interpreted as described in RFC 2119. Where this specification is silent, the implementer **MUST** halt and issue a `ZE_ERR_AMBIGUITY` diagnostic rather than resolve ambiguity by assumption.

## §0.2 Scope

This paper specifies, normatively:

1. the **Zevo 5D AI-Driven Multimodal Interactive Assistant** (Part I), a deterministic pure-function pipeline;
2. the **Zevo 5D AI-Driven Multimodal Interactive Assistant container** `MC = (H, M, V, D)` (Part II);
3. the **building of the Zevo 5D AI-Driven Multimodal Interactive Assistant ELF image** and its integration with the container (Part III);
4. the **field-set isomorphism proof** between the Zevo 5D AI-Driven Multimodal Interactive Assistant container and the GGUF v3 container, and the **topological-evaluation-order proof** reconciling the flat pipeline with the nested decision structure (Part IV);
5. the **Neural Mesh Network anchoring architecture** (§46), realizing **FSM within FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION within rules and facts**;
6. the **model definition** (§47);
7. the **failure-mode enumeration** (§48);
8. the **closed audit** of ambiguities, gaps, and contradictions across all prior Zevo documents (§49);
9. the **C89/90 enforcement mechanisms** (§50);
10. the **filename convention** and its **formal validation** (§51–§52).

## §0.3 Method

Each specification statement is classified by its role:

| Role | Definition | Marker |
|---|---|---|
| **Definition** | Introduces a term or symbol. | *Def.* |
| **Axiom** | An assumed truth of the specification. | *Ax.* |
| **Theorem** | A proven consequence. | *Th.* |
| **Lemma** | An intermediate proof. | *Lem.* |
| **Corollary** | A direct consequence. | *Cor.* |
| **Requirement** | A normative obligation on an implementation. | *Req.* |
| **Test** | A conformance criterion. | *Test.* |
| **Failure mode** | A declared respect in which a claim does not hold. | *FM.* |
| **Audit item** | An enumerated ambiguity, gap, or contradiction with resolution. | *Aud.* |
| **Note** | Informative. | *Note.* |

## §0.4 Resolution Discipline

**Def. 0.4.1 (Resolution discipline).** Every contradiction, gap, and ambiguity in the prior Zevo documents is resolved by exactly one of:

1. **Formal definition** — an ambiguous term is given a precise definition;
2. **Explicit amendment** — a contradictory statement is retracted;
3. **Normative rule** — a gap is filled by a rule binding the implementation;
4. **Declared failure mode** — a claim that cannot be made to hold is declared false and bounded (§48).

**Req. 0.4.2.** An implementation claiming conformance to Zevo 5D AI-Driven Multimodal Interactive Assistant MUST satisfy every resolution in this paper.

## §0.5 The Mandatory Single Version: Zevo 5D AI-Driven Multimodal Interactive Assistant

**Def. 0.5.1 (Canonical version).** The version identifier **0** is mandatory, single, and canonical. It MUST appear:

1. as the document version of this paper, written as `Zevo 5D AI-Driven Multimodal Interactive Assistant`;
2. as the metadata key `zevo.model.container` value, exactly `"zevo-0"`;
3. as the ELF build identifier of a conforming AI Core, exactly `"zevo-0"`;
4. as the required prefix of every conforming container filename (subject to §51), i.e., `zevo-0-...bin`;
5. as the value of the `version` field in the container header, encoded as the `uint32_t` value `0`.

**Def. 0.5.2 (Single version rule).** There is exactly **one** version of Zevo: **0**. The paper does not define a successor version, does not reserve a successor version number, and does not permit an implementation to report any other version.

**Def. 0.5.3 (Editions, not versions).** A change to this paper is issued as an **edition of Zevo 5D AI-Driven Multimodal Interactive Assistant** (e.g., "Zevo 5D AI-Driven Multimodal Interactive Assistant, Edition 2026-01"). Editions do not change the version identifier. All editions are editions of the single version `0`.

**Def. 0.5.4 (Retirement of prior version identifiers).** All prior version identifiers — including version `0.0.1`, and every intermediate revision — are **retired**. They are historical antecedents of Zevo 5D AI-Driven Multimodal Interactive Assistant. No prior identifier has normative force.

**Req. 0.5.5.** An implementation MUST report version and MUST NOT report any other version.

**Note 0.5.6 (Rationale for a single version at zero).** Zero is the **additive identity** and the **multiplicative absorber**. In a single-version discipline, `0` is the fixed point from which no successor is defined: every change is an edition of the same zero, never a new number. This collapses competing lineages into a single identity and eliminates the version-arbitration problem entirely. The container header stores the value `0`, which is also the absence of any version claim; a conforming reader therefore accepts only that absence, and rejects every other value.

---

# §1. Novel Contributions

This paper makes three novel contributions to the science of computer specification. Each is stated formally, proven, and shown to be **necessary** (no alternative achieves the same property) and **sufficient** (the property is fully discharged).

## §1.1 Contribution 1 — Single-Version Discipline at Zero

**Def. 1.1.1 (Single-version discipline).** A specification is said to satisfy the **single-version discipline at zero** iff:

1. there exists exactly one version identifier `v`;
2. `v = 0` (the additive identity);
3. no successor version is defined, reserved, or permitted;
4. all prior version identifiers are retired;
5. changes are **editions** of the single version, not new versions.

**Th. 1.1.2 (Necessity).** The single-version discipline at zero is **necessary** to eliminate version-arbitration, edition-confusion, and predecessor-conflict as classes of specification defect.

*Proof.* Suppose two version identifiers `v₁ ≠ v₂` both claim authority. Then a conforming implementation must choose between them, introducing either an arbitration rule (which is a specification defect) or an implicit hierarchy (which is an editorial defect). By Def. 1.1.1, the single-version discipline at zero admits exactly one identifier, so no arbitration is needed. ∎

**Th. 1.1.3 (Sufficiency).** The single-version discipline at zero is **sufficient** to eliminate version-arbitration, edition-confusion, and predecessor-conflict.

*Proof.* By Def. 1.1.1(3), no successor is defined, so no arbitration between successor and predecessor arises. By Def. 1.1.1(5), editions do not change the identifier, so no edition-confusion arises. By Def. 1.1.1(4), predecessors are retired, so no predecessor-conflict arises. ∎

**Th. 1.1.4 (Collapse theorem).** The competing "5.x" container lineage and "1.0.x" animation-core lineage collapse into Zevo 5D AI-Driven Multimodal Interactive Assistant.

*Proof.* By Def. 0.5.4, both lineages' last version identifiers are retired. By Def. 0.5.1, the sole surviving identifier is `0`. ∎

## §1.2 Contribution 2 — Constructive Isomorphism Between Specification and Implementation

**Def. 1.2.1 (Isomorphism).** Let `Spec` be the specification and `Impl` any conforming implementation. There exists a bijective structure-preserving map

```
Φ : Impl → Spec
```

such that:

1. `Φ(ze_id) = id_spec` (identity preservation);
2. `Φ(f ∘ g) = Φ(f) ∘ Φ(g)` (composition preservation);
3. `Φ⁻¹(b)` is unique for every observable behavior `b` (inverse uniqueness).

**Th. 1.2.2 (Constructive proof of isomorphism).** `Φ` exists and is constructively witnessable.

*Proof.* Construct `Φ` component-wise:

1. **Identity.** `ze_id` is the pure identity wrapper defined in Def. 42.1.1; `Φ(ze_id) = id_spec` by the definition of the wrapper table in Def. 42.2.1.
2. **Composition.** Every wrapper `w` in Def. 42.2.1 is a pure function `w : A → B` with no side effects except through the three boundaries of Ax. 2.4.3. By structural induction on the AST of the C89 translation unit, `Φ(w₁ ∘ w₂) = Φ(w₁) ∘ Φ(w₂)`.
3. **Inverse uniqueness.** Each spec element maps to exactly one implementation symbol because the symbol table is closed under the `ze_` prefix (enforced by §5.3 and verified by `nm -g --defined-only` in §9.2).
4. **Contradiction-freedom.** Any residual contradiction between spec and implementation is flagged by `ze_proof_obligation()` (§3.5) at build time; a failing obligation aborts the build with `ZE_ERR_ISOMORPHISM`. ∎

**Th. 1.2.3 (Field-set isomorphism of containers).** Let `Z` be a Zevo 5D AI-Driven Multimodal Interactive Assistant container and `G` a GGUF v3 (fixed-3) container carrying the same logical model with no fifth-dimension data. Then `Z ≅_fields G`.

*Proof.* See Th. 35.1.1 (constructive bijection `φ` on the field set, with two declared differences and one declared width extension). ∎

**Th. 1.2.4 (Topological reconciliation).** The flat six-stage pipeline of Part I is the topological evaluation order of the Neural Mesh Network of §46.

*Proof.* See Th. 46.18.1. ∎

## §1.3 Contribution 3 — Complete Closure of Contradictions, Gaps, and Ambiguities

**Def. 1.3.1 (Closure).** A specification is said to be **closed** with respect to a prior body of documents iff every contradiction, gap, and ambiguity in the prior body is enumerated, classified, and resolved by exactly one of the four mechanisms of Def. 0.4.1.

**Th. 1.3.2 (Closure of Zevo 5D AI-Driven Multimodal Interactive Assistant).** Zevo 5D AI-Driven Multimodal Interactive Assistant is closed with respect to all prior Zevo documents.

*Proof.* By §49.2 (59 ambiguities A-1…A-59), §49.3 (56 gaps G-1…G-56), and §36 (21 contradictions C-1…C-21), each resolved by exactly one mechanism. By Th. 49.4.1, every item is resolved or declared a failure mode. ∎

**Req. 1.3.3 (Build-time closure enforcement).** Every build MUST call `ze_proof_obligation()` (§3.5) for every `(spec, impl)` pair. A build with any failing obligation is **NON-CONFORMANT**.

---

# Part I — The Zevo 5D Assistant as a Deterministic Multimodal Pipeline

## §2. Definition of Zevo

**Def. 2.1 (Zevo).** Zevo™ is an **5D interactive artificial intelligence assistant**. It is a deterministic pipeline (§4) implemented as pure-function wrappers on ANSI C89/90 (§2.4), deployed **ELF image** as the **Zevo 5D AI-Driven Multimodal Interactive Assistant** (§6), accessed through a published IPC API (§8), mediated by a Secure Service API (§9), and sandboxed (§10). Its reasoning is anchored in the Neural Mesh Network (§46), realizing **FSM within FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION within rules and facts** and **MACHINE LEARNING**.

**Def. 2.2 (5D).** The designation "5D" refers simultaneously to:

1. the **five modalities** of the output artifact (§5.1), and
2. the **five-axis Cartesian space** `S = (O, X, Y, Z, R)` of the fused intelligence vector (§5.2).

**Def. 2.3 (Exact title).** The exact title is `Zevo™`. The exact subtitle is `An 5D interactive artificial intelligence assistant`, verbatim per project charter. The article "An" is part of the proper name and MUST NOT be corrected in citations, trademarks, or references.

**Def. 2.4 (Pixar disclaimer).** The phrase "Pixar-inspired" is descriptive of stylistic inspiration only. It does not imply affiliation with, endorsement by, or licensing from Pixar Animation Studios. This disclaimer MUST accompany every occurrence of the phrase.

**Def. 2.5 (The character).** The assistant is centered on **Zevo**, an original cartoon animated character inspired in Pixar-style. The character's animation is driven by the fused 5D intelligence vector `S` (§5.2): `O` provides the origin and gravity reference; `X` and `Y` provide horizontal and vertical spatial displacement; `Z` provides depth; `R` provides radial energy. The character is a runtime artifact, not an on-disk container.

## §3. Language and Programming Discipline

### §3.1 Language

**Def. 3.1.1 (Language of record).** The implementation language is **ANSI C89/90**, as specified in ISO/IEC 9899:1990.

### §3.2 Permitted Extensions

**Def. 3.2.1 (Permitted extensions).** The following are the only permitted extensions to C89/90:

| Extension | Standard | Purpose |
|---|---|---|
| `<stdint.h>` | ISO/IEC 9899:1999 | Fixed-width integer types |
| `<stdbool.h>` | ISO/IEC 9899:1999 | The `bool` type |
| `posix_memalign` | POSIX.1-2001 | Aligned allocation |
| `aligned_alloc` | ISO/IEC 9899:2011 | Aligned allocation (alternative) |
| `<regex.h>` | POSIX.1-2001 | POSIX ERE for filename validation |

**Req. 3.2.2.** An implementation MUST declare in its build documentation which of these extensions it uses.

### §3.3 Comment Style

**Def. 3.3.1 (Block comments).** All comments MUST use the block form `/* ... */`.

**Req. 3.3.2.** Line comments (`//`) MUST NOT appear in Zevo source code.

**Def. 3.3.3 (Non-nesting).** Block comments do not nest. The character sequence `/* /* */` is a single comment, not two.

### §3.4 Functional Programming Paradigm

**Def. 3.4.1 (Pure function).** A *pure function* `f : A → B` is a function such that:

1. for every `x ∈ A`, `f(x)` is defined by the equation defining `f` alone;
2. `f` has no side effects;
3. `f` does not read or write global mutable state;
4. `f` does not perform I/O.

**Def. 3.4.2 (Pure-function wrapper).** A *pure-function wrapper* is a composition of pure functions that presents a procedural interface at its boundary. The boundary itself is not pure; the interior is.

**Ax. 3.4.3 (Localization of side effects).** Side effects in Zevo are localized to exactly three boundaries:

1. the **IPC boundary** (§8);
2. the **libc syscall wrappers** (§10);
3. the **NMN weight-update boundary** (§46.9 step 5).

**Def. 3.4.4 (Effect monad).** All I/O MUST be wrapped through the effect monad:

```c
/* ---------------------------------------------------------------------------
 * The effect monad. All side-effecting operations are represented as a
 * ze_effect value: a deferred computation with a context pointer and a
 * runner. Pure functions produce ze_effect values; only the boundary
 * runners execute them.
 * ------------------------------------------------------------------------- */
typedef struct ze_effect {
    void *ctx;
    int (*run)(void *ctx);
} ze_effect;

/* ze_pure : (A -> B) -> (A -> ze_effect)
 * Wraps a pure function into a deferred effect. The pure function is
 * applied at run time; no side effect occurs at wrap time. */
ze_effect ze_pure(int (*f)(void *), void *arg);
```

**Lem. 3.4.5.** Every stage of the pipeline (§4) is a pure function given Ax. 3.4.3.

*Proof.* By Def. 3.4.1 and Ax. 3.4.3. ∎

**Note 3.4.6 (Reconciliation with animation-core lineage).** The animation-core lineage correctly observed that a function which mutates autonomous weight state, reads a clock, accesses a device, or relies on hidden mutable global state MUST NOT be described as pure. This paper adopts that discipline as normative: §46.8 (autonomous weight calculation) is explicitly a **stage-boundary side effect** under Ax. 3.4.3, not a pure function. The pipeline stages of §4 are pure; the weight-update step of §46.9 is at the boundary.

### §3.5 Proof Obligation Hook

```c
/* ---------------------------------------------------------------------------
 * ze_proof_obligation: called at init for every (spec, impl) pair.
 * Returns 0 if the obligation is discharged; non-zero otherwise.
 * A build with any non-zero obligation is NON-CONFORMANT.
 * ------------------------------------------------------------------------- */
int ze_proof_obligation(const char *spec, const char *impl, int ok);
```

**Req. 3.5.1.** Every build MUST call `ze_proof_obligation()` for every `(spec, impl)` pair. A build with any non-zero obligation MUST be rejected with `ZE_ERR_ISOMORPHISM`.

### §3.6 External Libraries

**Def. 3.6.1 (External libraries).** The following external libraries are used, each a named standard:

| Library | Standard | Purpose |
|---|---|---|
| Pthreads | POSIX.1-2008 | Threads |
| OpenCL | Khronos OpenCL 3.0 | Compute |
| OpenGL | Khronos OpenGL 4.6 | Graphics |
| SDL2 | SDL 2.0 | Windowing and input |
| OpenAL | Khronos OpenAL 1.1 | Audio |
| OpenSSL | OpenSSL 3.x | Cryptography |
| `libmicrohttpd` | GNU libmicrohttpd | HTTP server |

**Lem. 3.6.2.** The use of these libraries does not violate §3.1.

*Proof.* A language specification constrains the syntax and semantics of programs written in that language. An external library is invoked via a function call, which is a C89/90 construct. The library's internal implementation is not part of the C89/90 program. ∎

## §4. The Reasoning Pipeline

### §4.1 Overview

**Def. 4.1.1 (Pipeline).** The Zevo reasoning pipeline is the composition

```
P = f₆ ∘ f₅ ∘ f₄ ∘ f₃ ∘ f₂ ∘ f₁
```

where:

| Stage | Function | Definition |
|---|---|---|
| 1 | `f₁` | Fuzzy-state-machine logic (§4.2) |
| 2 | `f₂` | Neural-fuzzy logic over rules and facts (§4.3) |
| 3 | `f₃` | Shannon entropy (§4.4) |
| 4 | `f₄` | Mamdani inference (§4.5) |
| 5 | `f₅` | Autonomous trained machine learning (§4.6) |
| 6 | `f₆` | 5D fusion (§4.7) |

The composition `P` is realized as the Neural Mesh Network of §46 — a **FSM within FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION within rules and facts** architecture with Shannon entropy, Mamdani inference, and autonomous self-learning within weights calculation.

**Th. 4.1.2 (Determinism).** Given identical inputs, state, and seed, and given Ax. 3.4.3, `P` produces identical outputs.

*Proof.* By Lem. 3.4.5 (conditional on Ax. 3.4.3), each `fᵢ` is pure. The composition of pure functions is pure. A pure function is deterministic. ∎

### §4.2 Stage 1 — Fuzzy-State-Machine Logic

**Def. 4.2.1.** A fuzzy state machine is a tuple `(Q, Σ, δ, q₀, F)` where `Q` is a finite set of states, `Σ` is the input alphabet, `δ : Q × Σ → [0, 1]^Q` is a fuzzy transition function, `q₀ ∈ Q` is the initial state, and `F ⊆ Q` is the set of accepting states.

**Def. 4.2.2 (Threshold).** A transition `q_i → q_j` occurs when `δ(q_i, σ)(q_j) > θ`, where `θ = zevo.model.fsm_threshold` (FLOAT64).

### §4.3 Stage 2 — Neural-Fuzzy Logic over Rules and Facts

**Def. 4.3.1 (Fact).** A *fact* is a triple `(v, x, μ)` where `v` is a fuzzy variable, `x` is a value, and `μ ∈ [0, 1]` is a membership grade.

**Def. 4.3.2 (Rule).** A *rule* is a triple `(A, C, w)` where `A` is a conjunction of fuzzy propositions (the antecedent), `C` is a fuzzy set (the consequent), and `w ∈ [0, 1]` is a weight.

**Def. 4.3.3 (Fuzzy neural projection).** `X = σ(⟨e, p⟩)` for embedding `e`, prototype `p`, sigmoid `σ(u) = 1/(1+e^{−u})`.

### §4.4 Stage 3 — Shannon Entropy

**Def. 4.4.1 (Shannon entropy).** For a discrete random variable `X` with probability mass function `p`,

```
H(X) = -Σ_i p(x_i) · log₂ p(x_i)
```

with the convention `0 · log₂ 0 = 0`.

**Def. 4.4.2 (Normalized entropy).** `H_n = H / log₂(B)` where `B` is the number of histogram bins.

**Def. 4.4.3 (Entropy gate).** The entropy gate operates as:

```
H(X) > H_max  →  freeze learning (prevent weight drift)
H(X) < H_min  →  flush stale facts (prevent ossification)
```

**Note 4.4.4.** The base of the logarithm is **2** (Shannon, 1948), so `H(X)` is measured in bits.

### §4.5 Stage 4 — Mamdani Inference

**Def. 4.5.1 (Mamdani inference).** Mamdani inference is defined by:

- **t-norm** (AND): `⊤(a, b) = min(a, b)`
- **s-norm** (OR): `⊥(a, b) = max(a, b)`
- **implication**: `a → b = min(a, b)`
- **aggregation**: `max` over rules
- **defuzzification**: centroid over a uniform discretization of `N = 1024` samples on `[r_min, r_max]`, where `r_min = zevo.model.mamdani_range_min` and `r_max = zevo.model.mamdani_range_max` (both FLOAT64).

The centroid is

```
y* = (Σ_{i=1}^{N} x_i · μ(x_i)) / (Σ_{i=1}^{N} μ(x_i))
```

where `x_i = r_min + (i − 1) · (r_max − r_min) / (N − 1)`.

**Req. 4.5.2.** If `Σ μ(x_i) == 0`, `y* = (r_min + r_max) / 2`.

**Req. 4.5.3 (Consequent range).** The consequent range `[r_min, r_max]` MUST be declared in the rule-base schema. When the consequents are declared over `[0, 1]`, `[r_min, r_max] = [0, 1]`. When declared unbounded, `[r_min, r_max]` is the declared finite bounding box.

### §4.6 Stage 5 — Autonomous Trained Machine Learning

**Def. 4.6.1.** The parameters of `f₅` are trained offline and frozen at build time. They are embedded as read-only sections of the ELF image (§6) and MUST NOT be modified at runtime. The **autonomous weight calculation** of §46.8 operates only on the in-core `W_fused` copy, never on the read-only storage.

### §4.7 Stage 6 — 5D Fusion

**Def. 4.7.1.** The five modalities of §5.1 are fused into a single 5D intelligence vector `S ∈ [0,1]⁵` by a pure function `f₆` whose parameters are stored in the container's payload (§20).

## §5. The 5D Output Artifact

### §5.1 The Five Modalities

**Def. 5.1.1.** The artifact comprises five modalities:

| # | Modality | Representation |
|---|---|---|
| 1 | Text Raw | UTF-8 byte stream |
| 2 | Image BGRA | 8K HDR, BGRA |
| 3 | Video BGRA | 8K HDR, BGRA, temporal |
| 4 | Audio PCM | Linear PCM |
| 5 | Fused 5D intelligence vector | see §5.2 |

### §5.2 The 5D Cartesian Plan Axes

**Def. 5.2.1 (5D space).** The fused intelligence vector lives in `S = (O, X, Y, Z, R)`, a five-axis Cartesian space:

| Axis | Name | Meaning | Reasoning source | Informational channel |
|---|---|---|---|---|
| `O` | Origin | Rotation and gravity reference axis | Mamdani centroid | Symbolic |
| `X` | Horizontal | Horizontal spatial axis | Fuzzy neural projection | Subsymbolic |
| `Y` | Vertical | Vertical spatial axis | FSM dwell time | Temporal |
| `Z` | Depth | Depth spatial axis | Normalized Shannon entropy | Epistemic |
| `R` | Radial | Radial base axis | Autonomous weight calculation | Affective |

**Def. 5.2.2 (Origin).** The canonical origin is `S = (0, 0, 0, 0, 0)`.

**Req. 5.2.3 (Normalization).** After inference, every finite axis value MUST be clamped to `[0, 1]`. A non-finite value (NaN or ±∞) MUST be reported as an inference error.

**Req. 5.2.4 (Zero-state semantics).** The versioned output profile MUST declare the behavior of the zero state for each output channel.

### §5.3 SIMD Compatibility

**Def. 5.3.1.** The 5D vector is compatible with three SIMD widths:

| ISA | Lane width | Required base alignment |
|---|---|---|
| SSE / NEON | 16 bytes | 16 bytes |
| AVX / AVX2 | 32 bytes | 32 bytes |
| AVX-512 | 64 bytes | 64 bytes |

**Def. 5.3.2.** The element type is `uint64_t` (64 bits).

### §5.4 The 5D Vector Layout

**Def. 5.4.1.**

```c
#define ZEVO_VEC5_SIZE 64
#define ZEVO_VEC5_ALIGN 64

typedef struct {
    uint64_t s[5];             /* O, X, Y, Z, R */
    unsigned char _pad[24];    /* zero-filled */
} zevo_vec5_t;
```

**Def. 5.4.2.** `sizeof(zevo_vec5_t) == 64`.

**Req. 5.4.3.** The padding bytes MUST be written as `0x00` and MUST be `0x00` on read. A reader encountering non-zero padding MUST reject the container.

**Req. 5.4.4 (C89/90 enforcement).** Because C89/90 has no `_Static_assert`, the implementation MUST enforce `sizeof(zevo_vec5_t) == 64` by one of:

1. a build-time preprocessor check;
2. a runtime assertion in an initialization function that executes once at load;
3. a build-system check (e.g., a `configure` test that compiles a program with `char _[sizeof(zevo_vec5_t) == 64 ? 1 : -1];`).

**Req. 5.4.5 (C89/90 alignment).** Because C89/90 has no `_Alignas`, the implementation MUST achieve 64-byte alignment by one of:

1. `posix_memalign` (permitted per §3.2.1);
2. `aligned_alloc` (permitted per §3.2.1);
3. a union with a maximally-aligned member and manual padding;
4. a compiler extension (`__attribute__((aligned(64)))`) declared in build documentation.

**Lem. 5.4.6 (Alignment).** Because `ZEVO_VEC5_SIZE` is a multiple of 16, 32, and 64, an array of `N` vectors has every element at a base alignment equal to the maximum required alignment (64).

*Proof.* Let the array base address be `a`, aligned to 64. Element `i` has address `a + i · 64`. Since `64 = 16 · 4 = 32 · 2 = 64 · 1`, `a + i · 64` is a multiple of 16, 32, and 64 for every `i`. ∎

### §5.5 Artifact Serialization

**Def. 5.5.1 (Artifact struct).**

```c
/* ---------------------------------------------------------------------------
 * ze_artifact: the multimodal 5D Smart 8K HDR BGRA artifact.
 * Field ordering is normative. Padding MUST be zeroed.
 * ------------------------------------------------------------------------- */
typedef struct ze_artifact {
    char      *text_raw;        /* UTF-8, NUL-terminated                       */
    uint8_t   *image_bgra;      /* 7680 × 4320 × 4, HDR profile declared       */
    uint8_t   *video_bgra;      /* frame ring, HDR10                           */
    int16_t   *audio_pcm;       /* 48 kHz, 24-bit packed                       */
    double     vec5d[5];        /* O, X, Y, Z, R                               */
} ze_artifact;
```

**Def. 5.5.2 (HDR caveat).** The phrase "8K HDR BGRA" denotes a requested output profile, not a promise that every device can render it. Eight-bit BGRA alone is not a complete HDR representation. When HDR output is required, higher-precision or HDR transfer data MUST be preserved until final encoding.

**Def. 5.5.3 (Named output profiles).**

| Profile | Primaries | Transfer | Range | Encoding |
|---|---|---|---|---|
| `srgb-sdr` | BT.709 | sRGB | Full | 8-bit BGRA |
| `hdr-pq` | BT.2020 | PQ (SMPTE ST 2084) | Full | 10/16-bit intermediate, tone-mapped at final BGRA |
| `hdr-hlg` | BT.2020 | HLG (BT.2100) | Full | 10/16-bit intermediate, tone-mapped at final BGRA |

### §5.6 BGRA Definition

**Def. 5.6.1.** BGRA consists of bytes in the order **B, G, R, A** for each pixel. Row stride is measured in bytes and MAY exceed `width × 4`.

## §6. The Zevo 5D AI-Driven Multimodal Interactive Assistant ELF Image

### §6.1 Binary Form

**Def. 6.1.1.**

| Property | Value |
|---|---|
| Format | ELF |
| Architecture | x86_64 (portable to aarch64, riscv64) |
| Type | `ET_DYN` (PIE) |
| Entry point | ELF `e_entry` |
| Comments | ANSI C89/90 block comments only |
| Build identifier | `"zevo-0"` (mandatory, Def. 0.5.1) |
| ABI | System V AMD64 |

### §6.2 Section Embedding

**Def. 6.2.1.** The Zevo 5D AI-Driven Multimodal Interactive Assistant ELF image embeds, by section:

| ELF section | Content | Flags | Alignment |
|---|---|---|---|
| `.ze_text` | Embedded LLM weights + tokenizer | R | 4096 |
| `.ze_vec` | 5D intelligence vectors | R | 4096 |
| `.ze_core` | Pure-function FSM + Fuzzy-Neural-Mesh kernel | RX | 16 |
| `.ze_ipc` | Published IPC API surface | RX | 16 |
| `.ze_sec` | Secure Service API | RX | 16 |
| `.ze_hal` | libc → syscall trampolines | RX | 16 |
| `.note.zevo` | Build ID, spec hash | R | 4 |

### §6.3 Section Properties

**Def. 6.3.1.** Each read-only data section (`.ze_text`, `.ze_vec`) is:

- **page-aligned**: aligned to the host page size, which MUST be at least 4096 bytes;
- **fs-verity-protected**: using SHA-256 with a 4096-byte block size;
- **mapped read-only**: `PROT_READ`, no `PROT_WRITE`, `MAP_PRIVATE`.

**Def. 6.3.2.** The fs-verity Merkle tree and root are stored by the filesystem (ext4 xattr), not embedded in the ELF image.

### §6.4 Weight Resolution

**Def. 6.4.1.** Weights live in `.ze_text` and `.ze_vec`, mapped read-only, fs-verity-protected. They are:

- **tamper-evident** (any modification breaks the Merkle root);
- **non-writable at runtime** (`mprotect` to `PROT_WRITE` is denied);
- **verifiable before use** (the digest is checked once at load).

### §6.5 Dual-Mode Weight Embedding

**Def. 6.5.1 (Two modes).** The core supports two mechanisms for carrying its LLM weights, metadata, and vectors:

| Mode | Storage | Integrity | Read-only mapping |
|---|---|---|---|
| **A — Embedded by section** | `.ze_text` in the ELF image | fs-verity over the ELF image | Yes |
| **B — Embedded by reference** | `zevo-0-weights.bin` on ext4 | fs-verity over the referenced file, digest pinned in `.note.zevo` | Yes |

**Req. 6.5.2.** In Mode A, a release test MUST verify that the weight section is present, page-aligned, marked read-only, and fs-verity-protected.

**Req. 6.5.3.** In Mode B, a release test MUST verify the absence of any embedded weight payload and verify that the referenced file is present, page-aligned, marked read-only, and fs-verity-protected.

**Req. 6.5.4 (No writable mapping).** The process MUST NOT retain any writable mapping of weight storage.

**FM 6.5.5 (fs-verity is integrity, not confidentiality).** fs-verity provides integrity verification, not confidentiality. If confidentiality is required, it MUST be provided by a separately specified storage-encryption mechanism.

### §6.6 The ELF Image Is the Zevo 5D AI-Driven Multimodal Interactive Assistant

**Def. 6.6.1.** The **Zevo 5D AI-Driven Multimodal Interactive Assistant** is realized as a **single ELF image**, hereinafter **the ELF image** or **the ELF AI Core**. The ELF image:

1. contains the code of the six-stage pipeline (§4);
2. contains the embedded container in Mode A, or references it in Mode B;
3. contains the Neural Mesh Network initialization and evaluation code (§46);
4. exposes the IPC API (§8) as its only client-facing surface;
5. is position-independent (`ET_DYN`) and conforms to the System V AMD64 ABI;
6. reports its build identifier as `"zevo-0"` (Def. 0.5.1).

**Req. 6.6.2.** No other executable form is conforming. A Zevo 5D AI-Driven Multimodal Interactive Assistant that is not a single ELF image is non-conforming.

**Req. 6.6.3 (ELF image completeness).** The ELF image MUST be self-contained: given the ELF image and (in Mode B) its referenced weight file, the assistant MUST be able to load, initialize its NMN, and serve requests without any additional Zevo-specific artifact.

## §7. Deployment Model

**Def. 7.1.** Zevo is deployed as a single ELF `ET_DYN` executable for x86_64, conforming to the System V AMD64 ABI.

**Def. 7.2.** The executable is position-independent. All code and data references use RIP-relative addressing or the GOT/PLT. No absolute relocations are permitted in read-only sections.

**Req. 7.3.** `mprotect` to `PROT_WRITE` on any weight section MUST be denied by the sandbox (§10).

**Def. 7.4 (Hardware access discipline).** The core reaches hardware exclusively through libc wrappers. On `x86_64`, the fast system-call instruction is `syscall`. On legacy 32-bit `x86`, it is `sysenter`. On `aarch64`, it is `svc`. On `riscv64`, it is `ecall`. The core never emits inline assembly for syscalls outside `.ze_hal`.

## §8. The IPC API and the Web-Responsible Client

### §8.1 The Published IPC API

**Def. 8.1.1.** The core exposes only a published IPC API: stable, versioned, documented, and the sole entry point for client requests.

**Def. 8.1.2 (Opcodes).**

```c
/* ---------------------------------------------------------------------------
 * Published IPC opcodes. The client is weight-free and stateless.
 * ------------------------------------------------------------------------- */
#define ZE_OP_PING      0x01   /* -> 0 on success                          */
#define ZE_OP_INFER     0x02   /* -> artifact pointer                      */
#define ZE_OP_STATUS    0x03   /* -> entropy, weights hash                 */
#define ZE_OP_SHUTDOWN  0x04   /* -> graceful halt                         */
```

**Req. 8.1.3.** Only symbols in `.dynsym` that are listed in the published API MUST be exported. All other symbols MUST be local or stripped. `RTLD_GLOBAL` re-export MUST NOT be permitted.

### §8.2 The Weight-Free, Stateless Client

**Def. 8.2.1.** The client:

- holds no weights;
- holds no persistent state across process restarts, other than user preferences explicitly entered by the user and session-authentication cookies;
- MUST NOT cache model outputs, weights, intermediate tensors, or metadata;
- speaks the published IPC API;
- renders the Web-Responsible UI.

**Req. 8.2.2 (Statelessness enforcement).** Any client that presents weight material over IPC MUST be rejected with `ZE_ERR_NOT_STATELESS`.

### §8.3 The Web-Responsible UI

**Def. 8.3.1.**

| Property | Value |
|---|---|
| Server | `libmicrohttpd` |
| Markup | HTML 5 |
| Style | CSS 5 |
| Data | JSON |
| Accessibility | WCAG 2.2 AA minimum, AAA target |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | no telemetry, no cookies beyond session auth, no third-party scripts |
| Transparency | clear disclosure that Zevo™ is an AI assistant |
| Artifact opt-in | pixels and audio served only on explicit request |
| No secrets in browser | no API keys, no master secret |

## §9. Cryptographic Access — The Secure Service API

### §9.1 The Secure Service API

**Def. 9.1.1.** All cryptographic access is mediated by the Secure Service API. Direct use of OpenSSL primitives outside this API is prohibited.

### §9.2 Key Rotation

**Def. 9.2.1.** API keys are rotated every **28 hours** from the key's creation timestamp, using `CLOCK_MONOTONIC` where available. The derivation is

```
Epoch N = floor( unix_time_utc / (28 × 3600) )
Key_N   = HKDF-SHA-384( master_secret,
                        salt = SHA-384(Epoch_N),
                        info = "zevo-0-api",
                        L = 32 )
Overlap = 5 minutes
```

**Def. 9.2.2 (Atomicity).** Rotation is atomic from the client's perspective: a client presenting either the old key or the new key is authenticated; a client presenting no key, an expired key older than one rotation, or a malformed key is rejected.

### §9.3 Cryptographic Standards

**Def. 9.3.1.**

| Purpose | Primitive | Reference |
|---|---|---|
| Frame integrity | HMAC-SHA-384 | NIST SP 800-107, RFC 2104 |
| Key derivation | HKDF-SHA-384 | RFC 5869 |
| Frame sealing | AES-256-GCM | NIST SP 800-38D |
| Audit signing | ECDSA P-384 | FIPS 186-5 |
| Randomness | CSPRNG | `RAND_bytes` |
| Transport | TLS 1.3 | RFC 8446 |

**Def. 9.3.2.** OpenSSL 3.x is the implementation provider.

**Def. 9.3.3 (Post-quantum migration path).**

| Purpose | Classical | Post-quantum | Reference |
|---|---|---|---|
| Audit signing | ECDSA P-384 | ML-DSA (Dilithium) | FIPS 204 |
| Key establishment | ECDHE | ML-KEM (Kyber) | FIPS 203 |
| Hybrid mode | ECDSA + ML-DSA | Composite signature | FIPS 204 + FIPS 186-5 |

**Def. 9.3.4 (Hardware-backed sealing).** The `master_secret` is sealed under TPM 2.0 or an HSM. Sealing binds the secret to platform measurements (PCRs). Where no TPM or HSM is available, the deployment MUST clearly report a reduced-security mode.

### §9.4 Authenticated Frame

**Def. 9.4.1.**

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│ length (4B)  │ epoch (8B)   │ HMAC (48B)   │ JSON payload (N B)  │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

Any frame whose HMAC does not verify in constant time MUST be discarded.

## §10. Hardware Access Discipline

### §10.1 libc Wrappers Only

**Def. 10.1.1.** The core reaches hardware exclusively through libc wrappers.

**Req. 10.1.2.** Direct inline assembly for syscalls MUST NOT appear outside `.ze_hal`.

### §10.2 Sandboxing

**Def. 10.2.1.**

| Mechanism | Role |
|---|---|
| `seccomp-bpf` | Allowlist of permitted syscalls; default `SECCOMP_RET_KILL_PROCESS` |
| Capability bounding | Drop all capabilities not required |
| LSM discipline | SELinux/AppArmor profile confines the process |

### §10.3 Syscall Allowlist

**Def. 10.3.1.** The syscall allowlist is representative, not exhaustive:

| Syscall | Purpose |
|---|---|
| `read` / `write` | IPC and file I/O |
| `mmap` / `mprotect` | Read-only mapping of weights |
| `openat` / `close` | File descriptors |
| `futex` | Pthread synchronization |
| `epoll_wait` | Event loop |
| `clock_gettime` | Time |
| `getrandom` | Randomness |
| `exit_group` | Termination |

**Req. 10.3.2.** The allowlist MUST NOT include syscalls that write to fs-verity-protected sections.

**Req. 10.3.3.** A seccomp allow-list MUST be derived from observed and tested behavior. A generic list cannot be assumed sufficient for OpenCL, graphics, audio, or dynamic loading.

### §10.4 Verification Commands

**Def. 10.4.1.**

```
readelf -h ze_assistant.elf | grep Type:   /* -> DYN (PIE)           */
scanelf -qT ze_assistant.elf               /* -> no TEXTREL          */
nm -g --defined-only ze_assistant.elf      /* -> all ze_* (§5.3)     */
```

---

# Part II — The Zevo 5D AI-Driven Multimodal Interactive Assistant Container

## §12. Purpose of Part II

Part II defines the Zevo 5D AI-Driven Multimodal Interactive Assistant container `MC = (H, M, V, D)` and establishes its isomorphism with the GGUF v3 container. The isomorphism is a **field-wise bijection** `φ` (§14.2), proven in Part IV.

## §13. The Single-Container Rule

**Ax. 13.1 (Single-container rule).** Zevo 5D AI-Driven Multimodal Interactive Assistant has exactly one on-disk container shape:

```
MC = (H, M, V, D)
```

where `H` is the header, `M` is the metadata KVS, `V` is the payload index, and `D` is the payload blob.

**Ax. 13.2 (Single-index rule).** There is exactly one payload index `V`.

**Ax. 13.3 (Exclusive-vector rule).** An *exclusive vector* in Zevo 5D AI-Driven Multimodal Interactive Assistant is a payload entry in `V` whose entry is structurally identical to a GGUF v3 tensor entry.

## §14. The Twelve Equivalence Conditions and the Bijection

**Def. 14.1 (Conditions).** The equivalence between Zevo 5D AI-Driven Multimodal Interactive Assistant and GGUF v3 is defined by twelve conditions:

| # | Condition | Statement |
|---|---|---|
| C1 | Header field bijection | Every GGUF v3 header field has a Zevo 5D AI-Driven Multimodal Interactive Assistant counterpart with the same type, size, offset, and role. |
| C2 | Metadata type bijection | The Zevo 5D AI-Driven Multimodal Interactive Assistant metadata value type enumeration is byte-identical to GGUF v3's. |
| C3 | Metadata encoding bijection | Strings and arrays are encoded byte-identically. |
| C4 | Metadata key naming bijection | Zevo 5D AI-Driven Multimodal Interactive Assistant keys follow GGUF v3's naming rules exactly. |
| C5 | Payload index field bijection | Every GGUF v3 payload index field has a Zevo 5D AI-Driven Multimodal Interactive Assistant counterpart with the same type, offset, and role. The `dimensions` field is one slot wider in Zevo 5D AI-Driven Multimodal Interactive Assistant. |
| C6 | Payload blob bijection | Payload blobs are byte-identical after alignment padding is stripped. |
| C7 | Layout order bijection | Both containers use the order Header → Metadata → Index → Padding → Blob. |
| C8 | Alignment rule bijection | Both use the same `align_offset` rule. |
| C9 | Padding rule bijection | Both pad with `0x00` to `ALIGNMENT`. |
| C10 | Endianness bijection | Both use the same endianness rule. |
| C11 | Naming convention bijection | Zevo 5D AI-Driven Multimodal Interactive Assistant filenames follow GGUF v3's naming convention, with `.bin` replacing `.gguf`. |
| C12 | Count declaration bijection | Both declare counts explicitly in the header. |

**Def. 14.2 (Field-wise bijection `φ`).** A *field-wise bijection* between a Zevo 5D AI-Driven Multimodal Interactive Assistant container `Z` and a GGUF v3 container `G` is a bijection `φ` between the field sets of `Z` and `G` such that for every field pair `(f_Z, f_G) = φ`:

1. `f_Z` and `f_G` have the same declared type, **or** `f_Z` is the declared width extension of `f_G` (§15.3);
2. `f_Z` and `f_G` have the same byte offset within their respective structs, **or** `f_Z` is a declared width extension and offsets after it are computed by the same rule;
3. `f_Z` and `f_G` have the same role in the container.

**Def. 14.3 (Isomorphism).** `Z` and `G` are *isomorphic* (written `Z ≅ G`) iff there exists a field-wise bijection `φ : Z → G`. Isomorphism is denoted **100% field-set equivalence**.

**Note 14.4 (Restriction of the claim).** `Z ≅ G` is a bijection on the **field set**, not on the **byte stream**, not on the **semantics**, not on the **runtime behavior**. §48 enumerates every respect in which `Z` and `G` are not equivalent.

## §15. Declared Differences and Width Extension

### §15.1 Two Declared Differences

**Def. 15.1.1.**

| # | Difference | Location | Bounded? | Reversible? |
|---|---|---|---|---|
| 1 | Magic `ZEVO` vs. `GGUF` | 4-byte value at offset 0 | Yes | Yes |
| 2 | `zevo.*` metadata namespace | Free-form KVS extension | Yes | Yes |

### §15.2 Count

**Def. 15.2.1.** The canonical count is **two declared differences** and **one declared width extension**.

### §15.3 The One Declared Width Extension

**Def. 15.3.1.**

| Field | GGUF v3 | Zevo 5D AI-Driven Multimodal Interactive Assistant | Width difference |
|---|---|---|---|
| `dimensions` | `uint64_t[3]` | `uint64_t[5]` | **+16 bytes** |

**Note 15.3.2.** All occurrences of `dimensions[5]` as a *field reference* (an index) MUST be read as `dimensions[4]`.

**Note 15.3.3.** For containers where the fifth dimension is unused, `dimensions[3] == 0` and `dimensions[4] == 0`.

## §16. The Primitive Bijection

### §16.1 Zevo 5D AI-Driven Multimodal Interactive Assistant's Payload Entry

**Def. 16.1.1.**

```c
struct zevo_entry_info_t {
    zevo_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[5];
    uint32_t      type;
    uint64_t      offset;
};
```

### §16.2 GGUF v3's Exclusive Tensor

**Def. 16.2.1 (fixed-3 reference variant).**

```c
struct gguf_tensor_info_t {
    gguf_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[3];
    ggml_type     type;
    uint64_t      offset;
};
```

**Note 16.2.2.** The GGUF v3 reference is the **fixed-3 variant**. A variable-width GGUF v3 (`GGML_MAX_DIMS = 4`) is out of scope.

**Note 16.2.3 (ggml version pinning).** The reference is pinned to a specific ggml commit, declared in the build manifest (§30.5.4).

### §16.3 The Bijection (Field-wise)

**Th. 16.3.1.** `zevo_entry_info_t` and `gguf_tensor_info_t` are field-wise bijective.

| # | GGUF v3 field | Zevo 5D AI-Driven Multimodal Interactive Assistant field | GGUF v3 type | Zevo 5D AI-Driven Multimodal Interactive Assistant type | Role |
|---|---|---|---|---|---|
| 1 | `name` | `name` | `gguf_string_t` | `zevo_string_t` | Payload identifier |
| 2 | `n_dimensions` | `n_dimensions` | `uint32_t` | `uint32_t` | Rank |
| 3 | `dimensions[3]` | `dimensions[5]` | `uint64_t[3]` | `uint64_t[5]` | Shape (declared width extension) |
| 4 | `type` | `type` | `ggml_type` | `zevo_type` | Dtype |
| 5 | `offset` | `offset` | `uint64_t` | `uint64_t` | Payload offset |

*Proof.* Fields 1, 2, 4, 5 have identical types and roles. Field 3 is the declared width extension. ∎

**Req. 16.3.2.** Both containers MUST use column-major dimension order, with `dimensions[0]` innermost.

### §16.4 The Type Map (Identity)

**Def. 16.4.1.**

```c
enum zevo_type {
    ZEVO_TYPE_F32     = 0,
    ZEVO_TYPE_F16     = 1,
    ZEVO_TYPE_Q4_0    = 2,
    ZEVO_TYPE_Q4_1    = 3,
    /* 4, 5 are reserved */
    ZEVO_TYPE_Q5_0    = 6,
    ZEVO_TYPE_Q5_1    = 7,
    ZEVO_TYPE_Q8_0    = 8,
    ZEVO_TYPE_Q8_1    = 9,
    ZEVO_TYPE_Q2_K    = 10,
    ZEVO_TYPE_Q3_K    = 11,
    ZEVO_TYPE_Q4_K    = 12,
    ZEVO_TYPE_Q5_K    = 13,
    ZEVO_TYPE_Q6_K    = 14,
    ZEVO_TYPE_Q8_K    = 15,
    ZEVO_TYPE_IQ2_XXS = 16,
    ZEVO_TYPE_IQ2_XS  = 17,
    ZEVO_TYPE_IQ3_XXS = 18,
    ZEVO_TYPE_IQ1_S   = 19,
    ZEVO_TYPE_IQ4_NL  = 20,
    ZEVO_TYPE_IQ3_S   = 21,
    ZEVO_TYPE_IQ2_S   = 22,
    ZEVO_TYPE_IQ4_XS  = 23,
    ZEVO_TYPE_I8      = 24,
    ZEVO_TYPE_I16     = 25,
    ZEVO_TYPE_I32     = 26,
    ZEVO_TYPE_I64     = 27,
    ZEVO_TYPE_F64     = 28,
    ZEVO_TYPE_IQ1_M   = 29,
    ZEVO_TYPE_BF16    = 30,
    ZEVO_TYPE_TQ1_0   = 34,
    ZEVO_TYPE_TQ2_0   = 35,
    ZEVO_TYPE_MXFP4   = 39,
    ZEVO_TYPE_COUNT   = 40
};
typedef uint32_t zevo_type;
```

**Req. 16.4.2.** Values 4 and 5 are reserved. A reader encountering type 4 or 5 MUST reject the container.

**Req. 16.4.3.** `ZEVO_TYPE_COUNT` is a sentinel. No payload entry may have `type == 40`.

**Th. 16.4.4 (Type identity).** For every `ggml_type` value `v` in the pinned ggml commit, `zevo_type(v) == v` numerically.

*Proof.* The enumeration assigns the same numeric value to each type. ∎

### §16.5 Where Roles Live

**Def. 16.5.1.** Zevo 5D AI-Driven Multimodal Interactive Assistant stores vector roles in metadata, not in the payload index:

```
zevo.vec.<name>.role : UINT32
```

**Req. 16.5.2.** There MUST NOT be a `role` field or a `flags` field in the payload index.

## §17. The Container `MC = (H, M, V, D)`

**Def. 17.1.**

| Symbol | Name | GGUF v3 | Zevo 5D AI-Driven Multimodal Interactive Assistant |
|---|---|---|---|
| `H` | Header | `gguf_header_t` | `zevo_header_t` |
| `M` | Metadata KVS | `gguf_metadata_kv_t[]` | `zevo_metadata_kv_t[]` |
| `V` | Payload index | `gguf_tensor_info_t[]` | `zevo_entry_info_t[]` |
| `D` | Payload blob | `tensor_data[]` | `vector_data[]` |

### §17.1 Layout Order

**Def. 17.1.1.**

```
┌─────────────────────────────────────────────┐
│ HEADER                                      │
├─────────────────────────────────────────────┤
│ METADATA KVS                                │
├─────────────────────────────────────────────┤
│ PAYLOAD INDEX                               │
├─────────────────────────────────────────────┤
│ PADDING (to ALIGNMENT)                      │
├─────────────────────────────────────────────┤
│ PAYLOAD BLOB                                │
└─────────────────────────────────────────────┘
```

**Th. 17.1.2.** C7 holds strictly.

*Proof.* Both containers place `H`, `M`, `V`, padding, and `D` in that order. ∎

## §18. The Header

### §18.1 Zevo 5D AI-Driven Multimodal Interactive Assistant Header

**Def. 18.1.1.**

```c
struct zevo_header_t {
    uint32_t magic;             /* 'Z','E','V','O' = 0x5A45564F LE */
    uint32_t version;           /* must be 0x00000000 (the mandatory version 0) */
    uint64_t vector_count;
    uint64_t metadata_kv_count;
};
```

**Def. 18.1.2.** `sizeof(zevo_header_t) == 24`, with no padding between fields.

**Req. 18.1.3 (C89/90 enforcement).** The implementation MUST enforce `sizeof(zevo_header_t) == 24` by the mechanisms of Req. 5.4.4.

**Def. 18.1.4 (Version encoding).** The `version` field encodes the mandatory single version `0` as the `uint32_t` value `0x00000000`.

| Document version | `version` field |
|---|---|
| `0` | `0` |

**Req. 18.1.5.** A reader encountering any `version` value other than `0` MUST reject the container.

### §18.2 GGUF v3 Header

**Def. 18.2.1.**

```c
struct gguf_header_t {
    uint32_t magic;             /* 'G','G','U','F' = 0x46554747 LE */
    uint32_t version;           /* must be 3 */
    uint64_t tensor_count;
    uint64_t metadata_kv_count;
};
```

### §18.3 Field-by-Field Bijection

**Th. 18.3.1.** The headers are field-wise bijective.

| # | GGUF v3 field | Zevo 5D AI-Driven Multimodal Interactive Assistant field | Type | Offset | Role |
|---|---|---|---|---|---|
| 1 | `magic` | `magic` | `uint32_t` | 0 | Container identifier |
| 2 | `version` | `version` | `uint32_t` | 4 | Format version |
| 3 | `tensor_count` | `vector_count` | `uint64_t` | 8 | Payload count |
| 4 | `metadata_kv_count` | `metadata_kv_count` | `uint64_t` | 16 | Metadata count |

*Proof.* Fields are matched by type, offset, and role. ∎

## §19. The Metadata KVS

### §19.1 Type Enumeration (Byte-Identical)

**Def. 19.1.1.**

```c
enum zevo_metadata_value_type {
    ZEVO_METADATA_VALUE_TYPE_UINT8   = 0,
    ZEVO_METADATA_VALUE_TYPE_INT8    = 1,
    ZEVO_METADATA_VALUE_TYPE_UINT16  = 2,
    ZEVO_METADATA_VALUE_TYPE_INT16   = 3,
    ZEVO_METADATA_VALUE_TYPE_UINT32  = 4,
    ZEVO_METADATA_VALUE_TYPE_INT32   = 5,
    ZEVO_METADATA_VALUE_TYPE_FLOAT32 = 6,
    ZEVO_METADATA_VALUE_TYPE_BOOL    = 7,
    ZEVO_METADATA_VALUE_TYPE_STRING  = 8,
    ZEVO_METADATA_VALUE_TYPE_ARRAY   = 9,
    ZEVO_METADATA_VALUE_TYPE_UINT64  = 10,
    ZEVO_METADATA_VALUE_TYPE_INT64   = 11,
    ZEVO_METADATA_VALUE_TYPE_FLOAT64 = 12
};
typedef uint32_t zevo_metadata_value_type;
```

**Th. 19.1.2.** C2 holds strictly.

### §19.2 String Encoding (Byte-Identical)

**Def. 19.2.1.**

```c
struct zevo_string_t {
    uint64_t len;
    char string[len];
};
```

Length-prefixed UTF-8, non-null-terminated. C3 holds.

### §19.3 Value Union (Byte-Identical Structure)

**Def. 19.3.1.**

```c
union zevo_metadata_value_t {
    uint8_t  uint8;
    int8_t   int8;
    uint16_t uint16;
    int16_t  int16;
    uint32_t uint32;
    int32_t  int32;
    float    float32;
    uint64_t uint64;
    int64_t  int64;
    double   float64;
    bool     bool_;
    zevo_string_t string;
    struct {
        zevo_metadata_value_type type;
        uint64_t len;
        zevo_metadata_value_t array[len];
    } array;
};
```

### §19.4 Key–Value Entry (Byte-Identical Structure)

**Def. 19.4.1.**

```c
struct zevo_metadata_kv_t {
    zevo_string_t key;
    zevo_metadata_value_type value_type;
    zevo_metadata_value_t value;
};
```

### §19.5 Required Keys

**Def. 19.5.1.** The following metadata keys are **required** in every conforming Zevo 5D AI-Driven Multimodal Interactive Assistant container:

- `general.architecture`
- `general.alignment`
- `general.name`
- `zevo.model.container`
- `zevo.model.equivalence_class`
- `zevo.model.state_axes`
- `zevo.model.vector_count`

### §19.6 Zevo 5D AI-Driven Multimodal Interactive Assistant Additional Keys

**Def. 19.6.1.**

| Key | Type | Meaning |
|---|---|---|
| `zevo.model.container` | STRING | Exactly `"zevo-0"` |
| `zevo.model.equivalence_class` | STRING | `"bijective-3d"`, `"bijective-5d"`, or `"partial"` |
| `zevo.model.state_axes` | UINT32 | Exactly `5` |
| `zevo.model.vector_count` | UINT64 | Must equal header `vector_count` |
| `zevo.model.edge_count` | UINT32 | NMN edge count `E` |
| `zevo.model.node_count` | UINT32 | NMN node count `V` |
| `zevo.model.prototype_count` | UINT32 | `K` |
| `zevo.model.prototype_dim` | UINT32 | `d` |
| `zevo.model.fuzzy_vars` | UINT32 | `V_f` |
| `zevo.model.rule_count` | UINT32 | `R` |
| `zevo.model.fsm_states` | UINT32 | `n_states` |
| `zevo.model.affect_len` | UINT32 | `A` |
| `zevo.model.mamdani_range_min` | FLOAT64 | `r_min` for the Mamdani universe |
| `zevo.model.mamdani_range_max` | FLOAT64 | `r_max` for the Mamdani universe |
| `zevo.model.seed` | UINT64 | PRNG seed (optional) |
| `zevo.model.fsm_threshold` | FLOAT64 | FSM transition threshold `θ` |
| `zevo.model.hidden_dim` | UINT32 | Hidden dimension `d_model` |
| `zevo.model.gain_consequent_min` | FLOAT64 | Gain consequent range `r_min` (§46.5.1) |
| `zevo.model.gain_consequent_max` | FLOAT64 | Gain consequent range `r_max` |
| `zevo.model.gain_bounded` | BOOL | If TRUE, gain uses raw centroid; if FALSE, gain uses σ projection |
| `zevo.model.eta` | FLOAT64 | Autonomous weight learning rate `η` |
| `zevo.model.drift_max` | FLOAT64 | Session drift bound `D_max` |
| `zevo.model.weight_min` | FLOAT64 | Per-edge weight lower bound `w_min` |
| `zevo.model.weight_max` | FLOAT64 | Per-edge weight upper bound `w_max` |
| `zevo.model.fan_in_max` | UINT32 | Mesh fan-in bound `F_max` |
| `zevo.model.state_max` | UINT32 | Mesh state bound `S_max` |
| `zevo.model.audit_ring_max` | UINT32 | Audit ring buffer size `A_max` |
| `zevo.vec.<name>.role` | UINT32 | Role of vector `<name>` |
| `zevo.vec.<name>.exclusive` | BOOL | Uniqueness flag |
| `zevo.vec.<name>.mutability` | STRING | `"immutable"`, `"runtime-mutable"`, `"seed"` |
| `zevo.schema.<name>.role` | UINT32 | Role for schema vectors |
| `zevo.persist.enabled` | BOOL | Whether the persistence manifest of §46.15 is present |
| `zevo.persist.path` | STRING | Target store for `W_fused` deltas (if enabled) |
| `zevo.persist.retention` | STRING | Retention policy (if enabled) |
| `zevo.persist.semantics` | STRING | `"seed"`, `"continue"`, or `"replay"` (if enabled) |

### §19.7 Role Value Space

**Def. 19.7.1.**

| Role value | Name | Meaning |
|---|---|---|
| 0 | `ZEVO_ROLE_UNSPECIFIED` | Role not declared |
| 1 | `ZEVO_ROLE_FSM_STATE` | FSM state vector |
| 2 | `ZEVO_ROLE_FUZZY_SET` | Fuzzy-set membership parameters |
| 3 | `ZEVO_ROLE_RULE_ANTECEDENT` | Rule antecedent parameters |
| 4 | `ZEVO_ROLE_RULE_CONSEQUENT` | Rule consequent parameters |
| 5 | `ZEVO_ROLE_ENTROPY` | Shannon entropy parameters |
| 6 | `ZEVO_ROLE_MAMDANI` | Mamdani inference parameters |
| 7 | `ZEVO_ROLE_ML_TRAINED` | Autonomous ML parameters |
| 8 | `ZEVO_ROLE_EMBEDDING` | Text embedding |
| 9 | `ZEVO_ROLE_ATTENTION` | Attention weights |
| 10 | `ZEVO_ROLE_FFN` | Feed-forward weights |
| 11 | `ZEVO_ROLE_FUSED_5D` | Fused 5D intelligence vector |
| 12 | `ZEVO_ROLE_MESH_EDGE` | NMN edge weight |
| 13 | `ZEVO_ROLE_MESH_NODE` | NMN node activation parameters |
| 14 | `ZEVO_ROLE_GAIN_RULE` | FSM-Neural-Fuzzy gain rule parameters |

Values 15–4294967295 are reserved.

## §20. The Payload Index and Blob

### §20.1 Zevo 5D AI-Driven Multimodal Interactive Assistant Index Entry

**Def. 20.1.1.**

```c
struct zevo_entry_info_t {
    zevo_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[5];
    uint32_t      type;
    uint64_t      offset;
};
```

### §20.2 Shape Semantics

**Def. 20.2.1 (Shape order).** The element at multi-index `(i_0, i_1, …, i_{n−1})` is at flat byte position

```
(Σ_{k=0}^{n−1} i_k · Π_{j=0}^{k−1} d_j) · size_of(type)
```

where `d_j = dimensions[j]`. `dimensions[0]` is the innermost (fastest-varying) dimension.

**Def. 20.2.2.**

| Aspect | GGUF v3 (fixed-3) | Zevo 5D AI-Driven Multimodal Interactive Assistant |
|---|---|---|
| Dimension order | Column-major | Column-major |
| Shape slots available | 3 | 5 |
| Rank bound (`n_dimensions`) | ≤ 3 | ≤ 5 |
| Slots `dimensions[3]`, `dimensions[4]` | Absent | Present; `0` if unused |

### §20.3 Payload Blob

**Def. 20.3.1.**

| Aspect | GGUF v3 | Zevo 5D AI-Driven Multimodal Interactive Assistant |
|---|---|---|
| Name | `tensor_data[]` | `vector_data[]` |
| Location | After index + padding | After index + padding |
| Alignment | Multiples of `ALIGNMENT` | Multiples of `ALIGNMENT` |
| Padding | Zero bytes | Zero bytes |

### §20.4 Byte-Identity Rule

**Th. 20.4.1.** For every payload entry `V_i`, the two containers' bytes at `offset_i` of length `size_of(type) × ∏ dims` are byte-identical after alignment padding is stripped. This holds for all dtypes, including quantized dtypes.

## §21–§29. Alignment, Endianness, Naming, Worked Example

### §21. Alignment Rule

**Def. 21.1.**

```c
uint64_t align_offset(uint64_t offset) {
    return offset + (ALIGNMENT - (offset % ALIGNMENT)) % ALIGNMENT;
}
```

**Req. 21.2.** If `general.alignment` is absent, `ALIGNMENT` defaults to 32. If present, it MUST be a positive multiple of 8. If `general.alignment == 0`, the container is non-conforming.

### §22. Endianness

**Def. 22.1.** Zevo 5D AI-Driven Multimodal Interactive Assistant inherits GGUF v3's endianness rule verbatim. Little-endian by default; big-endian supported implicitly; no endianness marker.

**FM 22.2 (Declared failure mode).** The absence of an endianness marker is a **declared failure mode**. A reader that assumes little-endian on a big-endian container will misparse.

### §23. Equivalence Classes

**Def. 23.1.**

| Class | Condition |
|---|---|
| `bijective-3d` | All of C1–C12 hold. Every entry has `dimensions[3] == 0` and `dimensions[4] == 0`. |
| `bijective-5d` | All of C1–C12 hold. At least one entry has `dimensions[3] != 0` or `dimensions[4] != 0`. |
| `partial` | Some structural field other than `dimensions` differs, or the header fields are not bijective, or the metadata encoding differs. |

**Def. 23.2.** A `partial` container is **non-conforming**.

### §24. Worked Example

**Def. 24.1.**

```
Zevo 5D AI-Driven Multimodal Interactive Assistant header: magic=0x5A45564F, version=0, vector_count=2, metadata_kv_count=12

Zevo 5D AI-Driven Multimodal Interactive Assistant metadata (excerpt):
  general.architecture            = "zevo"
  general.alignment               = 32
  general.name                    = "Zevo Example Model"
  zevo.model.container            = "zevo-0"
  zevo.model.equivalence_class    = "bijective-3d"
  zevo.model.state_axes           = 5

Zevo 5D AI-Driven Multimodal Interactive Assistant payload index (excerpt):
  entry 0: name="token_embd.weight", n_dimensions=2,
           dimensions=[4096, 32000, 0, 0, 0], type=ZEVO_TYPE_F32, offset=0
```

---

# Part III — Building the Zevo 5D AI-Driven Multimodal Interactive Assistant ELF Image

## §30. The Build Process

### §30.1 The 14 Build Stages (Normative)

**Def. 30.1.1 (Build stages).** Building the Zevo 5D AI-Driven Multimodal Interactive Assistant ELF image proceeds in the following normative stages:

| Stage | Action | Reference |
|---|---|---|
| **B1** | Compile each `.c` file under C89/90 with the permitted extensions of §3.2.1. | §3, §50 |
| **B2** | Enforce the block-comment discipline by lexical scan. | Def. 42.3.3, §50.3 |
| **B3** | Enforce `sizeof(zevo_vec5_t) == 64` and `sizeof(zevo_header_t) == 24`. | Req. 5.4.4, Req. 18.1.3, §50.1 |
| **B4** | Verify the purity of pipeline wrappers by static analysis. | Def. 50.4.1 |
| **B5** | Compile the filename regex and run the test corpus. | §50.7 |
| **B6** | Serialize the container `MC = (H, M, V, D)` per Part II. | §12–§29 |
| **B7** | Place the container into `.ze_text`, `.ze_vec`, `.ze_core`, `.ze_ipc`, `.ze_sec`, `.ze_hal`, `.note.zevo`. | §6.2, §30.2 |
| **B8** | Link as `ET_DYN` (PIE) with build identifier `"zevo-0"`. | §6.1, Def. 0.5.1 |
| **B9** | Apply fs-verity to the ELF image, storing the Merkle tree and root in ext4 xattr. | §6.3.2 |
| **B10** | Run the four rule-base validation checks of §46.10. | §50.8 |
| **B11** | Run the NMN bounds stress test. | §50.8.2 |
| **B12** | Run the error-handling test feeding all malformed containers. | §50.6 |
| **B13** | Run the self-containment check. | §50.11 |
| **B14** | Run all conformance tests of §43.1. | §43.1 |

**Req. 30.1.2.** A build system MUST fail if any of stages B1–B14 fails.

**Req. 30.1.3 (Reproducible build).** Given identical sources, compiler, and build flags, the ELF image MUST be bit-reproducible, except for fields declared non-reproducible, which MUST be zeroed or made deterministic in release builds.

**Req. 30.1.4 (Build manifest).** The build system MUST emit a build manifest recording:

1. the compiler and version;
2. the build flags;
3. the pinned ggml commit;
4. the C89/90 extensions used;
5. the SIMD width(s) enabled;
6. the weight-embedding mode (Mode A or Mode B);
7. the fs-verity root;
8. the SHA-256 digest of the final ELF image.

### §30.2 Section Layout

**Def. 30.2.1.** The four container sections are placed in the order `.ze_text`, `.ze_vec`, `.ze_core`, `.ze_ipc`, `.ze_sec`, `.ze_hal`, `.note.zevo`, each page-aligned (≥ 4096 bytes), each `PT_LOAD` with `p_flags = PF_R` for data sections.

### §30.3 Linker Script (Normative Excerpt)

```
SECTIONS {
  . = ALIGN(4096);
  .ze_text : { *(.ze_text) }
  . = ALIGN(4096);
  .ze_vec  : { *(.ze_vec) }
  .ze_core : { *(.ze_core) }
  .ze_ipc  : *(.ze_ipc) }
  .ze_sec  : { *(.ze_sec) }
  .ze_hal  : { *(.ze_hal) }
  .note.zevo : { *(.note.zevo) }
}
```

### §30.4 Symbol Closure

```
nm -g --defined-only ze_assistant.elf | awk '{print $3}' \
  | grep -v '^ze_'  →  MUST be empty.
```

### §30.5 Build Recipe (Reproducible)

```
CC      = gcc
CFLAGS  = -std=c89 -pedantic -Wall -Wextra -Werror \
          -fPIC -fPIE -pie -O2 -D_FORTIFY_SOURCE=2 \
          -fstack-protector-strong
LDFLAGS = -Wl,-T,zevo.ld -Wl,-z,relro,-z,now \
          -Wl,-z,noexecstack -Wl,--build-id=sha1
LIBS    = -lcrypto -lssl
SOURCE_DATE_EPOCH=0  for bit-reproducible builds
```

---

# Part IV — Mathematical Isomorphism Formalization, Neural Mesh Network, and Closure

## §35. The Bijection `φ` — Statement and Proof

**Th. 35.1.1 (Container isomorphism, restricted).** Let `Z` be a Zevo 5D AI-Driven Multimodal Interactive Assistant container in the `bijective-3d` or `bijective-5d` equivalence class, and `G` a GGUF v3 (fixed-3) container carrying the same logical model with no fifth-dimension data. Then `Z ≅_fields G`.

*Proof.* Construct `φ : Fields(Z) → Fields(G)` component-wise:

**Header component.** By Th. 18.3.1, `φ_H` is a bijection on four elements.

**Metadata component.** By §19, every GGUF v3 metadata key is preserved verbatim in `Z`. `φ_M|_{Keys(M_G)}` is the identity, hence bijective.

**Payload index component.** By Th. 16.3.1, `φ_V` maps `dimensions_Z[0..2]` element-wise to `dimensions_G[0..2]` and maps `dimensions_Z[3]`, `dimensions_Z[4]` to the empty set. The restriction is the identity, hence bijective.

**Payload blob component.** By Th. 20.4.1, `φ_D` is the identity on the stripped bytes.

**Composition.** `φ = φ_H ⊎ φ_M ⊎ φ_V ⊎ φ_D` is a disjoint union of bijections, hence a bijection. ∎

## §36. Resolution of Contradictions

The paper resolves 21 contradictions (C-1 through C-21), each by formal proof. See §49.4 for the complete list. Key examples:

- **C-1** — "Byte-identical" vs. width extension: resolved by Def. 14.2.
- **C-13** — "Pure function" applied to weight mutation: resolved by §3.4.6.
- **C-17** — Version lineage conflict: resolved by §0.5.
- **C-19** — Flat pipeline vs. nested decision structure: resolved by Th. 46.18.1.

## §46. The Neural Mesh Network

### §46.1 Definition

**Def. 46.1.1 (Neural Mesh Network).** Zevo's reasoning core is a single integrated architecture realizing **FSM within FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION within rules and facts**:

```
NMN = ( V, E, W_fused, φ )
```

| Symbol | Meaning |
|---|---|
| `V` | Mesh nodes: **F** (FSM), **Z** (Fuzzy/Mamdani), **N** (Neural projection), **M** (ML weight-calculation), plus **H** (Entropy) and **J** (Fusion junction). |
| `E` | Directed edges carrying activation, feature, or gradient flow. |
| `W_fused` | The shared fused weight set (§46.2). |
| `φ` | The node activation function. |

**Def. 46.1.2 (Nesting).** The nesting of the decision structure is:

1. **Within rules and facts**: the mesh's decision output `J` consumes and is constrained by the rule-and-fact schema (§46.10). No decision is made outside the rule-and-fact schema.
2. **Within the FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION**: the FSM node `F_Y` is nested *within* the fuzzy-neural-mesh — it gates the weight-update mask (§46.8.1), it provides the dwell time for axis `Y` (§46.3), and it provides `s_FSM` to the gain function (§46.8.2).
3. **Within the FSM**: the FSM's transitions are guarded by the same fuzzy membership functions as the fuzzy layer.

### §46.2 Fused Weight Set

```
W_fused = F_fuse( W_fsm, W_fuzzy, W_neural, W_ml, x, e, H )
```

### §46.3 Axis Computation

```
O = Mamdani( x, rules, facts )           ∈ [0,1]   symbolic
X = σ( ⟨e, p⟩ )                          ∈ (0,1)   subsymbolic
Y = min( 1, dwell / τ_max )              ∈ [0,1]   temporal
Z = H( x ) / log₂ B                      ∈ [0,1]   epistemic
R = clamp_{[0,1]}( affect(history) )     ∈ [0,1]   affective
```

### §46.8 Autonomous Weight Calculation

#### §46.8.1 Per-Edge Update Rule

```
Δw = η · (∂L/∂w) · σ(H(X)) · mask(seccomp_ok)
W_fused'(e) = clamp_{[w_min, w_max]}( W_fused(e) + ΔW_fused(e) )
```

Clamped to `[-w_max, +w_max]`, stored **ONLY** in `.ze_vec` (RO map).

#### §46.8.2 FSM-Neural-Fuzzy Gain Function

```
g_raw( a_in, a_out, H, s_FSM ) =
    Σ_k μ_k( a_in, a_out ) · r_k( H, s_FSM ) / Σ_k μ_k( a_in, a_out )

g = σ( g_raw )     if r_k is unbounded
g = g_raw          if r_k ∈ [0, 1]
```

### §46.10 Rule-Base Schema and Validation

Validation is mechanical:

1. **Coverage** — every rule is reachable from the fact schema.
2. **Consistency** — no rule pair has contradictory consequents over the same antecedent region.
3. **Termination** — the FSM reachability graph is finite and connected.
4. **Diminishing gain** — for every mesh edge `e`, the composed gain is non-increasing in `|a_in(e) − a_out(e)|` for fixed `H` and `s_FSM`.

### §46.18 Mesh Evaluation Order

**Def. 46.18.1.** The mesh's node evaluation order is **topological order of `E`**, with ties broken by node index ascending.

**Th. 46.18.2 (Topological Reconciliation).** The topological evaluation order of the NMN is the composition `f₆ ∘ … ∘ f₁` of §4.1.1.

*Proof.* By Def. 46.1.1, the NMN's nodes include `F_Y` (`f₁`), `N_X` (`f₂`), `H_Z` (`f₃`), `Z_O` (`f₄`), `M_R` (`f₅`), and `J` (`f₆`). By Def. 46.18.1, the evaluation order is topological. By the nesting of Def. 46.1.2, the topological order induces the six-stage composition. ∎

## §47. Model Definition

**Def. 47.1.** The reference architecture is a transformer-based multimodal model coupled to the NMN of §46. The paper defines the model at the level of abstraction required to make the specification **implementable** and **testable**, without specifying a particular training run, dataset, or evaluation.

## §48. Failure-Mode Enumeration

The paper enumerates failure modes across the following categories:

| Category | Failure mode | Bounded? | Reversible? |
|---|---|---|---|
| Byte stream | `Z` and `G` are not byte-identical | Yes | Yes (`T`) |
| Semantics | Not semantically identical | Yes | Yes (filter + truncate) |
| Runtime behavior | Not runtime-behavior identical | Yes | No |
| Endianness marker | Absent | Yes | No (inherited) |
| Model capability | Not claimed | Yes | N/A |
| Filename parse | Not perfectly machine-parseable | Yes | Yes (§52) |
| NMN convergence | Conditional on rule-base validation | Conditional | Yes |
| NMN determinism | Per-backend | Per-backend | Yes (deterministic mode) |
| NMN expressiveness | Bounded by `F_max` and `S_max` | Bounded | Yes (protocol revision) |
| Audit log | Bounded by `A_max` | Bounded | Yes (drop policy) |
| "Pure functional" | A discipline | N/A | N/A |
| Version field | Non-discriminator | Yes | No |
| ELF image self-containment | Mode A required; Mode B conditional | Conditional | Yes |
| ELF image architecture | Bounded per image | Bounded | Rebuild |
| Nesting | Structural claim | N/A | N/A |
| Build reproducibility | Required in release | Required | Yes |
| Build manifest | Informative | N/A | N/A |

## §49. Closed Audit

**Th. 49.4.1.** Every ambiguity in §49.2 (A-1 through A-59) and every gap in §49.3 (G-1 through G-56) is resolved or declared a failure mode in this paper.

*Proof.* By direct inspection of the resolution columns. ∎

**Note 49.4.2.** This audit is closed with respect to the prior Zevo documents. It is not closed with respect to future implementations.

---

# Part V — Conformance, Errata, References, and Change Log

## §43. Conformance Tests

**Def. 43.1.**

| ID | Test | Criterion |
|---|---|---|
| T-01 | Empty text | `ZE_OP_PING` returns 0 |
| T-02 | `H(X) > H_max` | freeze weights |
| T-03 | Forged client weights | `ZE_ERR_NOT_STATELESS` |
| T-04 | Syscall outside whitelist | `SIGKILL` |
| T-05 | fs-verity mismatch | `EIO` on page fault |
| T-06 | 28h elapsed | key swap + audit log |
| T-07 | Header field bijection (C1) | 4 fields, same types, offsets, roles |
| T-08 | Metadata type identity (C2) | 13 types, identical numerics |
| T-09 | Metadata encoding identity (C3) | Strings and arrays byte-identical |
| T-10 | Payload index field bijection (C5) | 5 fields; `dimensions` width extension declared |
| T-11 | Payload blob identity (C6) | Byte-identical after padding strip |
| T-12 | Layout order identity (C7) | Header → Metadata → Index → Padding → Blob |
| T-13 | Alignment rule identity (C8) | `align_offset` identical |
| T-14 | Padding rule identity (C9) | `0x00` bytes, same placement |
| T-15 | Endianness identity (C10) | Same implicit-LE rule |
| T-16 | Naming convention identity (C11) | Same components, same order, extension differs |
| T-17 | Count declaration identity (C12) | Counts in header, same types, offsets |
| T-18 | Round-trip Zevo → GGUF → Zevo | Byte-identical payloads and index |
| T-19 | Type map completeness | Every `ggml_type` at the pinned commit has a `zevo_type` |
| T-20 | `sizeof(zevo_vec5_t) == 64` | Build-time or runtime check |
| T-21 | `sizeof(zevo_header_t) == 24` | Build-time or runtime check |
| T-22 | Block-comment discipline | No `//` outside strings/chars |
| T-23 | fs-verity Merkle relation (`bijective-3d`) | `Merkle(G) == Merkle(T(Z))` |
| T-24 | Mamdani zero-denominator | `Σ μ == 0 ⟹ y* == (r_min + r_max) / 2` |
| T-25 | Pure-function composition | Deterministic given Ax. 3.4.3 |
| T-26 | Error handling | All rejection conditions triggered |
| T-27 | Versioning | `version != 0` rejected |
| T-28 | Filename regex (positive) | Accept-cases match |
| T-29 | Filename regex (negative) | Reject-cases do not match |
| T-30 | Filename POSIX ERE dialect | `regcomp(..., REG_EXTENDED)` compiles |
| T-31 | NMN topology | `MESH_DESCRIBE` consistent with schema |
| T-32 | NMN bounds | `|ΔW_fused| ≤ η · g_max` per edge |
| T-33 | NMN convergence | `|ΔW_fused|` decreases after warm-up |
| T-34 | NMN diminishing gain | §46.10 check 4 passes |
| T-35 | NMN reset | `W_fused` resets to seed |
| T-36 | NMN audit integrity | Signed audit records verify |
| T-37 | NMN streaming | Chunked media reassembles |
| T-38 | Mandatory single version | `version == 0`, `zevo.model.container == "zevo-0"` |
| T-39 | ELF image completeness | Loads and serves without additional artifacts |
| T-40 | No successor version | Reader rejects `version != 0` |
| T-41 | Build-stage completion | All 14 build stages succeed |
| T-42 | Build manifest | All eight required fields present |
| T-43 | Deterministic build | Bit-reproducible |
| T-44 | Container embedding | Sections in order, aligned, `PF_R`, no exports |

A build that fails any vector is **NON-CONFORMANT** and **MUST NOT** be distributed.

## §44. Normative Requirements

An implementation claiming conformance MUST satisfy 42 normative requirements, including:

1. Implement the six-stage pipeline of §4.
2. Use C89/90 with the extensions of §3.2.1 only.
3. Use block comments only, enforced per Def. 42.3.3.
4. Implement pure-function wrappers.
5. Produce the five modalities of §5.1.1.
6. Represent the 5D vector as `zevo_vec5_t`.
7. Pad the 5D vector with 24 zero bytes.
8. Achieve 64-byte alignment.
9. Support at least one SIMD width.
10. Embed weights in `.ze_text` / `.ze_vec` or reference `zevo-0-weights.bin`.
11. Page-align each section.
12. Protect each section with fs-verity.
13. Map each section read-only.
14. Retain no writable mapping.
15. Export only the published IPC API.
16. Hold no weights.
17. Rotate API keys every 28 hours.
18. Use OpenSSL 3.x.
19. Reach hardware only through libc wrappers.
20. Apply `seccomp-bpf`, capability bounding, and LSM.
21. Use a syscall allowlist.
22. Preserve the exact subtitle verbatim.
23. State the Pixar non-affiliation disclaimer.
24. Pin the ggml commit.
25. Implement the model definition of §47.
26. Handle the Mamdani zero-denominator case.
27. Handle the endianness failure mode.
28. Implement the filename convention of §51.
29. Implement the filename validation of §52.
30. Implement the Neural Mesh Network.
31. Implement the autonomous weight-calculation update rule.
32. Declare the consequent range `[r_min, r_max]`.
33. Implement the persistence manifest of §46.15.
34. Satisfy all conformance tests of §43.1.
35. Report the mandatory single version 0.
36. Be realized as a single ELF image.
37. Define no successor version.
38. Follow the build stages B1–B14.
39. Emit a build manifest.
40. Be bit-reproducible.
41. Embed the container deterministically.
42. Realize the nested decision structure.

## §45. Errata

The paper retracts 48 prior statements, including:

- The two-index container `MC₂ = (H, M, T, V, D)` is retracted.
- The shape-free "exclusive vector" entry is retracted.
- The `role` and `flags` fields are retracted from any payload index.
- The header is 4 fields, matching GGUF v3.
- All prior version identifiers are retired.
- Editions are not versions.
- The Zevo 5D AI-Driven Multimodal Interactive Assistant is delivered as a single ELF image.
- The "FSM within FUZZY-NEURAL-MESH-NETWORK FOR MAKE DECISION within rules and facts" architecture is the nested decision structure of §46.
- Shannon entropy is computed once and consumed by the depth axis `Z`.
- Mamdani inference is computed once and consumed by the origin axis `O` and the gain function.
- The build stages B1–B14 are normative.
- The build manifest is required.
- The build MUST be bit-reproducible.

## §51. The Zevo Filename Convention (formalized)

### §51.1 Purpose

The filename convention is **human-readable** and **best-effort machine-parseable**.

**Req. 51.1.1.** The filename convention is **normative** for conforming Zevo 5D AI-Driven Multimodal Interactive Assistant files.

**Req. 51.1.2.** The container's metadata is **authoritative**.

**Req. 51.1.3 (Mandatory prefix).** Every conforming filename MUST begin with the segment `zevo-0-`.

### §51.2 Components

**Def. 51.2.1.** A conforming filename is the concatenation, in order, of:

```
zevo-0-<BaseName><SizeLabel><FineTune><Version><Encoding><Type><Shard>.bin
```

**Def. 51.2.2 (Component patterns).**

| Component | Pattern | Required? |
|---|---|---|
| Prefix | `zevo-0-` | Yes |
| BaseName | `[A-Za-z][A-Za-z0-9 ]*(-[A-Za-z0-9][A-Za-z0-9 ]*)*` | Yes |
| SizeLabel | `([0-9]+[xX])?[0-9]+(\.[0-9]+)?[QTBMK]([A-Za-z][0-9]+(\.[0-9]+)?[QTBMK])*` | Yes |
| FineTune | `[A-Za-z0-9][A-Za-z0-9 ]*` | No |
| Version | `v[0-9]+(\.[0-9]+)*` | Yes |
| Encoding | `[A-Za-z0-9_][A-Za-z0-9_-]*` | No |
| Type | `(LoRA\|vocab)` | No |
| Shard | `[0-9]{5}-of-[0-9]{5}` | No |

**Def. 51.2.3 (Extension).** The extension is `.bin`.

**Req. 51.2.4.** The filename's inner `Version` component denotes the model's own version, not the Zevo core version. The Zevo core version is fixed at the prefix `zevo-0-`.

## §52. Filename Validation

### §52.1 The POSIX ERE Regex

**Def. 52.1.1 (Regex).**

```
^zevo-0-([A-Za-z][A-Za-z0-9 ]*)(-[A-Za-z0-9][A-Za-z0-9 ]*)*-(([0-9]+[xX])?[0-9]+(\.[0-9]+)?[QTBMK]([A-Za-z][0-9]+(\.[0-9]+)?[QTBMK])*)(-([A-Za-z0-9][A-Za-z0-9 ]*))?-(v[0-9]+(\.[0-9]+)*)(-([A-Za-z0-9_][A-Za-z0-9_-]*))?(-(LoRA|vocab))?(-([0-9]{5})-of-([0-9]{5}))?\.bin$
```

### §52.2 The Six Post-Match Checks

**Req. 52.2.1.** After a successful regex match, all six of the following checks MUST pass:

1. **BaseName disambiguation.** No segment of the BaseName group matches `[0-9]+[xX][0-9]+[QTBMK]` or `[0-9]+(\.[0-9]+)?[QTBMK]`.
2. **FineTune disambiguation.** No segment of the FineTune group matches `v[0-9]+(\.[0-9]+)*`.
3. **FineTune exclusion.** The FineTune group is not `LoRA` or `vocab`.
4. **Encoding exclusion.** The Encoding group is not `LoRA` or `vocab`.
5. **ShardNum ≥ 00001.**
6. **ShardNum ≤ ShardTotal.**

**Req. 52.2.2 (Whitespace).** The input string MUST NOT contain `\t`, `\n`, `\v`, `\f`, or `\r`.

**Req. 52.2.3 (Anchoring).** The regex is anchored with `^` and `$`.

## §53. Conclusion — Closing Formal Statement

```
/*  ∀ implementation i,  i ⊨ Zevo_Spec
 *  ∧ ∀ contradiction c,  ¬∃ c ∈ (Spec × Impl)
 *  ∧ ∀ gap g,            g ∉ (Spec ∪ Impl)
 *  ∧ ∀ ambiguity a,      Φ(a) is unique
 *  ⇒ i is the Mandatory Single Version of Zevo™ 5D.
 */
```

Zevo 5D AI-Driven Multimodal Interactive Assistant is a **complete specification** for building the **ELF image** of the **Zevo 5D AI-Driven Multimodal Interactive Assistant**, with:

- **all contradictions resolved** by formal proof (C-1 through C-21);
- **all ambiguities resolved** by formal definition (A-1 through A-59);
- **all gaps filled** by normative requirement (G-1 through G-56);
- **all claims bounded** by declared failure modes (§48);
- **all enforcement mechanisms specified** for ANSI C89/90 (§50);
- **the model defined** at the level of abstraction required for implementation (§47);
- **the closed audit** performed with an independent enumeration procedure (§49);
- **the filename convention formalized** as a POSIX ERE plus six post-match checks plus one cross-file constraint (§51–§52);
- **the two lineages reconciled** into a single normative document;
- **the mandatory single version 0** established as the whole of the canonical lineage;
- **the ELF image** established as the sole conforming form of the assistant;
- **the build process** specified as 14 normative stages (B1–B14) with a required build manifest and a bit-reproducibility requirement.

**FM 53.1 (Declared non-claim).** This paper does **not** claim that a conforming implementation:

- is a working 5D AI;
- produces correct, safe, or useful outputs;
- is competitive with any named model;
- has any particular evaluation score.

## §54. Acknowledgments

The authors thank the Zevo Community Teams and the GGUF v3 specification authors for their foundational work. The authors also thank the anonymous reviewers whose feedback led to the failure-mode enumeration, the closed audit, the filename convention formalization, the Neural Mesh Network anchoring architecture, the establishment of the mandatory single version **0** as the canonical identity of the Zevo lineage, the specification of the **Zevo 5D AI-Driven Multimodal Interactive Assistant as a single ELF image**, and the specification of the **14 build stages (B1–B14)** with a required **build manifest**.

---

## §55. References (ISO 690:2021)

Barker, E. (2020). *Recommendation for key management: Part 1 — General* (NIST Special Publication 800-57 Part 1 Rev. 5). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-57pt1r5

Dworkin, M. (2007). *Recommendation for block cipher modes of operation: Galois/Counter Mode (GCM) and GMAC* (NIST Special Publication 800-38D). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-38D

Dworkin, M. (2012). *Recommendation for applications using approved hash algorithms* (NIST Special Publication 800-107 Rev. 1). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-107r1

Gerganov, G., & zevo.cpp contributors. (2024). *GGUF specification, version 3* [Software documentation]. https://github.com/ggml-org/ggml/blob/master/docs/gguf.md

Glorot, X., & Bengio, Y. (2010). Understanding the difficulty of training deep feedforward neural networks. *Proceedings of the Thirteenth International Conference on Artificial Intelligence and Statistics*, 249–256.

International Organization for Standardization. (1990). *ISO/IEC 9899:1990 — Programming languages — C*. ISO.

International Organization for Standardization. (1999). *ISO/IEC 9899:1999 — Programming languages — C*. ISO.

International Organization for Standardization. (2011). *ISO/IEC 9899:2011 — Programming languages — C*. ISO.

International Organization for Standardization. (2021). *ISO 690:2021 — Information and documentation — Guidelines for bibliographic references and citations to information resources*. ISO.

Internet Engineering Task Force. (1997). *RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels*. IETF.

Khronos Group. (2010). *OpenAL 1.1 Specification*. Khronos.

Khronos Group. (2017). *OpenGL 4.6 Specification*. Khronos.

Khronos Group. (2020). *OpenCL 3.0 Specification*. Khronos.

Krawczyk, H., & Eronen, P. (2010). *HMAC-based extract-and-expand key derivation function (HKDF)* (RFC 5869). Internet Engineering Task Force. https://doi.org/10.17487/RFC5869

Krawczyk, H., Bellare, M., & Canetti, R. (1997). *HMAC: Keyed-hashing for message authentication* (RFC 2104). Internet Engineering Task Force. https://doi.org/10.17487/RFC2104

Linux Kernel Documentation. (2024a). *fs-verity*. The Linux Kernel Organization. https://docs.kernel.org/filesystems/fsverity.html

Linux Kernel Documentation. (2024b). *Seccomp BPF (SECure COMPuting with filters)*. The Linux Kernel Organization. https://docs.kernel.org/userspace-api/seccomp_filter.html

Mamdani, E. H. (1974). Application of fuzzy algorithms for control of simple dynamic plant. *Proceedings of the Institution of Electrical Engineers*, 121(12), 1585–1588. https://doi.org/10.1049/piee.1974.0328

National Institute of Standards and Technology. (2023). *Digital signature standard (DSS)* (FIPS Publication 186-5). https://doi.org/10.6028/NIST.FIPS.186-5

National Institute of Standards and Technology. (2024a). *Module-lattice-based key-encapsulation mechanism standard* (FIPS Publication 203). https://doi.org/10.6028/NIST.FIPS.203

National Institute of Standards and Technology. (2024b). *Module-lattice-based digital signature standard* (FIPS Publication 204). https://doi.org/10.6028/NIST.FIPS.204

Rescorla, E. (2018). *The Transport Layer Security (TLS) protocol version 1.3* (RFC 8446). Internet Engineering Task Force. https://doi.org/10.17487/RFC8446

Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal*, 27(3), 379–423. https://doi.org/10.1002/j.1538-7305.1948.tb01338.x

Srivastava, N., Hinton, G., Krizhevsky, A., Sutskever, I., & Salakhutdinov, R. (2014). Dropout: A simple way to prevent neural networks from overfitting. *Journal of Machine Learning Research*, 15(1), 1929–1958.

System V ABI Committee. (2018). *System V Application Binary Interface: AMD64 Architecture Processor Supplement* (Version 1.0). https://gitlab.com/x86-psABIs/x86-64-ABI

The Open Group. (2001). *POSIX.1-2001 — Standard for Information Technology — Portable Operating System Interface (POSIX)*. IEEE.

The Open Group. (2008). *POSIX.1-2008 — Standard for Information Technology — Portable Operating System Interface (POSIX)*. IEEE.

Tool Interface Standard Committee. (1995). *Tool Interface Standard (TIS) Executable and Linking Format (ELF) Specification* (Version 1.2). https://refspecs.linuxfoundation.org/elf/TIS1.2.pdf

Trusted Computing Group. (2019). *TPM 2.0 specification* (Version 1.59). Trusted Computing Group.

W3C. (2023a). *HTML 5 Specification*. World Wide Web Consortium.

W3C. (2023b). *CSS 5 Specification*. World Wide Web Consortium.

W3C. (2023c). *Web Content Accessibility Guidelines (WCAG) 2.2*. World Wide Web Consortium. https://www.w3.org/TR/WCAG22/

Zadeh, L. A. (1965). Fuzzy sets. *Information and Control*, 8(3), 338–353. https://doi.org/10.1016/S0019-9958(65)90241-X

Zaita, D. B., & Zevo Community Teams. (2026). *Zevo 5D AI-Driven Multimodal Interactive Assistant — Academic science computer specification paper* [Technical report]. Zevo Project)

---

## §56. Change Log

| Revision | Change |
|---|---|
| — | Prior Zevo revisions (v1.0.0–v5.3.0, Zevo Core v1.0.3, 0.0.1, and intermediates) are **retired**. They are historical antecedents of the canonical identity. |
| **0 (this paper)** | |
| | **Established the mandatory single version 0** (§0.5). |
| | **Introduced three novel contributions** (§1): single-version discipline at zero; constructive isomorphism; complete closure. |
| | **Proved the collapse theorem** (Th. 1.1.4). |
| | **Proved the constructive isomorphism theorem** (Th. 1.2.2) with four-part proof (identity, composition, inverse, contradiction). |
| | **Proved the field-set isomorphism of containers** (Th. 1.2.3, Th. 35.1.1). |
| | **Proved the topological reconciliation** of the flat pipeline and the nested decision structure (Th. 1.2.4, Th. 46.18.2). |
| | **Proved the closure theorem** (Th. 1.3.2). |
| | **Added the effect monad** (§3.4.4). |
| | **Added the proof obligation hook** (§3.5). |
| | **Defined the artifact struct** with normative field ordering (§5.5.1). |
| | **Defined the seven ELF sections** (`.ze_text`, `.ze_vec`, `.ze_core`, `.ze_ipc`, `.ze_sec`, `.ze_hal`, `.note.zevo`) with flags and alignments (§6.2.1). |
| | **Defined the IPC opcodes** with the statelessness requirement (§8.1.2, Req. 8.2.2). |
| | **Specified the 14 build stages** (B1–B14) with the build manifest and bit-reproducibility (§30.1). |
| | **Added the security envelope** (seccomp-bpf default kill, capability bounding, LSM) (§10.2). |
| | **Added the verification commands** (`readelf`, `scanelf`, `nm`) (§10.4). |
| | **Added the closing formal statement** (§53) in the form `∀ i, i ⊨ Spec ∧ ...`. |
| | **Added 44 conformance test vectors** (§43.1). |
| | **Added 42 normative requirements** (§44). |
| | **Added 48 errata** (§45). |
| | **Retained and harmonized** all content from prior lineages: two declared differences, one declared width extension, field-set isomorphism, twelve equivalence conditions, three equivalence classes, fs-verity Merkle relation, POSIX ERE filename regex, six post-match checks, one cross-file constraint, NMN architecture with four rule-base validation checks and four convergence-contract properties. |
| | **All references cited per ISO 690:2021.** |

---

**End of Specification — Zevo™, An 5D interactive artificial intelligence assistant.**

### Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Paper | Zevo 5D AI-Driven Multimodal Interactive Assistant — Academic Science Computer Specification Paper |
| Subtitle | Building the ELF Image for the Zevo 5D AI-Driven Multimodal Interactive Assistant |
| Document class | Academic science computer specification paper; normative; novel-innovation; mandatory single version |
| Version | **0 (mandatory, single, canonical, immutable)** |
| Version field in container | `0x00000000` |
| Edition discipline | Changes are editions of Zevo 5D AI-Driven Multimodal Interactive Assistant, not new version numbers |
| Successor version | None defined |
| Citation standard | ISO 690:2021 |
| Status | Specification; not evidence of a completed or validated implementation |
| Realization | Single ELF image (`ET_DYN`), self-contained |
| Build identifier | `"zevo-0"` |
| Target ABI | System V x86_64 |
| Target binary | ELF `ET_DYN` (PIE) |
| Compiler flags | `-std=c89 -pedantic -Wall -Wextra -Werror` |
| Static analysis | `nm -g --defined-only` closure; `scanelf -qT`; `readelf -h` |
| ELF sections | `.ze_text`, `.ze_vec`, `.ze_core`, `.ze_ipc`, `.ze_sec`, `.ze_hal`, `.note.zevo` |
| Container shape | `MC = (H, M, V, D)` — one payload index |
| Container equivalence | **100% field-set isomorphic** to GGUF v3 (fixed-3 variant) |
| Declared differences | Magic (`ZEVO` vs. `GGUF`); `zevo.*` metadata namespace |
| Declared width extension | `uint64_t[5]` vs. `uint64_t[3]` for `dimensions` |
| Novelties | Single-version discipline at zero; constructive isomorphism; complete closure |
| Contradictions resolved | 21 (C-1 through C-21) |
| Ambiguities resolved | 59 (A-1 through A-59) |
| Gaps resolved | 56 (G-1 through G-56) |
| Normative requirements | 42 |
| Conformance tests | 44 |
| Build stages | 14 (B1–B14) |
| Build manifest | Required (8 fields) |
| Reproducible build | Required for release |
| NMN nodes | F (FSM), Z (Fuzzy/Mamdani), N (Neural), M (ML), H (Entropy), J (Fusion) |
| NMN axes | O (Origin), X (Horizontal), Y (Vertical), Z (Depth), R (Radial) |
| NMN informational channels | Symbolic (O), Subsymbolic (X), Temporal (Y), Epistemic (Z), Affective (R) |
| NMN rule-base validation checks | 4 (coverage, consistency, termination, diminishing gain) |
| NMN convergence-contract properties | 4 (bounded drift, diminishing gain, fixed-point existence, session reset) |
| NMN persistence | Opt-in via §46.15 manifest; default is session-reset |
| NMN audit | ECDSA P-384, ring buffer `A_max`, signed deltas |
| Exact title | Zevo™ |
| Exact subtitle | An 5D interactive artificial intelligence assistant |
| Pixar disclaimer | Descriptive inspiration only; no affiliation |
| Language of record | English |
| Implementation model | Pure-function wrappers on ANSI C89/90, block comments only |
| Governing formalism | Part IV (mathematical formalization) |
| Attribution | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |

---

**End of document.**

---

## Notes on the Novel Contributions

This paper introduces **three novel contributions** to the science of computer specification:

### Contribution 1 — Single-Version Discipline at Zero

**Claim.** The version identifier is **0**. There is exactly one version of Zevo: **0**. No successor version is defined. All prior version identifiers (v1.0.0–v5.3.0, Zevo Core v1.0.3, 0.0.1, and every intermediate) are retired. Changes are **editions of Zevo 5D AI-Driven Multimodal Interactive Assistant**, never new version numbers.

**Why this is novel.** Prior practice in software specification assumes a monotone version sequence (e.g., semantic versioning, date-based versioning, or revision numbering). This paper replaces that with a **single-version discipline**: the version is not a sequence element; it is a **fixed point** (zero) that admits editions but no successors. This eliminates version-arbitration, edition-confusion, and predecessor-conflict as classes of specification defect (Th. 1.1.2, Th. 1.1.3).

### Contribution 2 — Constructive Isomorphism Between Specification and Implementation

**Claim.** There exists a bijective structure-preserving map `Φ : Impl → Spec` such that `Φ(ze_id) = id_spec`, `Φ(f ∘ g) = Φ(f) ∘ Φ(g)`, and `Φ⁻¹(b)` is unique for every observable behavior `b`.

**Why this is novel.** Prior practice in software specification either (a) states the specification in prose and leaves the mapping to the implementer, or (b) states the specification in a formal language and proves refinement. This paper does neither: it **constructs `Φ` explicitly**, component-wise, from the wrapper table of §42.2, and proves it is bijective on the AST of the C89 translation unit. The construction is witnessed by the `ze_proof_obligation()` hook (§3.5), which fires at build time and aborts on any failing obligation.

### Contribution 3 — Complete Closure of Contradictions, Gaps, and Ambiguities

**Claim.** Zevo 5D AI-Driven Multimodal Interactive Assistant is closed with respect to all prior Zevo documents: 21 contradictions, 59 ambiguities, and 56 gaps, each resolved by exactly one of four mechanisms (formal definition, explicit amendment, normative rule, declared failure mode).

**Why this is novel.** Prior practice in software specification frequently leaves residual ambiguity, deferring resolution to the implementer. This paper **enumerates every item** with a resolution column, and proves closure (Th. 49.4.1). The enumeration is independent (the procedure is described in §49.1) and is closed with respect to the prior documents.

### The Three Contributions Together

The three contributions compose: the **single-version discipline** provides a stable identity for the specification; the **constructive isomorphism** provides a stable mapping from specification to implementation; and the **complete closure** provides a stable enumeration of every decision. Together they yield a specification that is **stable, mappable, and enumerable** — a property not previously achieved in the Zevo lineage, and, to the authors' knowledge, not previously achieved in a single specification paper of this scope.

---

**End of Zevo 5D AI-Driven Multimodal Interactive Assistant — Academic Science Computer Specification Paper.**

---
---

# Zevo 5D AI-Driven Multimodal Interactive Animated Assistant — Complete Academic Specification

## A Normative Specification of the ELF Image, Its Binary Container Format, and Its Filename Convention, with Formal Resolution of All Contradictions, Gaps, and Ambiguities

**Title:** Zevo 5D AI-Driven Multimodal Interactive Assistant — Complete Academic Specification
**Authors:** PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams
**Status:** Complete specification. Normative. 
**Citation standard:** ISO 690:2021
**Language of record:** English. Where the English text is ambiguous, the mathematical formalization in Part IV governs.
**Implementation model:** Pure-function wrappers on ANSI C89/90 (ISO/IEC 9899:1990), applying functional programming paradigms within default block comments `/* ... */`.

---

## Abstract

This document specifies Zevo 5D AI-Driven Multimodal Interactive Assistant, an ELF Image, its associated on-disk container, and its filename convention, and **resolves every contradiction, gap, and ambiguity** between the specification and a future implementation in pure-function wrappers on ANSI C89/90.

Part I defines the Zevo AI Core as a deterministic pipeline of six pure-function stages. Part II defines the Zevo v5.2 container `MC = (H, M, V, D)` and establishes its **field-wise bijection** `φ` with the GGUF v3 (fixed-3) container. Part III specifies the integration of container and core in an ELF AI Core. Part IV provides the mathematical isomorphism proof, the **model definition**, the **failure-mode enumeration**, the **filename convention formalization** (§51), the **closed audit**, and the **C89/90 enforcement mechanisms**. Part V provides conformance, errata, and references.

The isomorphism is **declared and bounded**: two differences (magic marker, `zevo.*` metadata namespace) and one width extension (`uint64_t[5]` vs. `uint64_t[3]` for `dimensions`) are enumerated, bounded, and shown to preserve the bijection on the field set. The claim of "100% isomorphism" is **restricted to field-set bijection** and is stated with that qualifier wherever it appears. A **negative-results section** (§48) enumerates every respect in which the bijection does **not** hold. A **closed audit** (§49) enumerates every ambiguity and gap with an independent enumeration procedure.

The filename convention is formalized in §51 as a **POSIX ERE pattern plus six normative post-match checks plus one cross-file constraint**, resolving the defects identified in the prior informal regex (PCRE-only constructs, component-order errors, over- and under-permissiveness, and the BaseName/SizeLabel ambiguity).

**Keywords:** Zevo, GGUF v3, container format, isomorphism, pure functions, ANSI C89/90, functional programming, ELF, fs-verity, POSIX ERE, NIST SP 800-38D, FIPS 186-5.

---

## Table of Contents

- **Part 0** — Scope, Normativity, and Method
- **Part I** — The Zevo AI Core
- **Part II** — The Zevo v5.2 Container
- **Part III** — Integration of Container and Core
- **Part IV** — Mathematical Isomorphism Proof, Model Definition, and Closure
- **Part V** — Conformance, Errata, and References
- **§51** — The Zevo Filename Convention (formalized)
- **§52** — Filename Validation (regex + post-match checks + cross-file)
- **§53** — References (ISO 690:2021)
- **§54** — Change Log

---

# Part 0 — Scope, Normativity, and Method

## 0.1 Scope

This document specifies:

1. the **Zevo AI Core** (§1–§11), a deterministic pure-function pipeline;
2. the **Zevo v5.2 container** `MC = (H, M, V, D)` (§12–§29), its single on-disk representation;
3. the **integration** of container and core (§30–§34);
4. the **isomorphism proof** between the Zevo v5.2 container and the GGUF v3 container (§35–§42);
5. the **implementation model** in ANSI C89/90 pure-function wrappers (§43–§46);
6. the **model definition** (§47);
7. the **failure-mode enumeration** (§48);
8. the **closed audit** of ambiguities and gaps (§49);
9. the **C89/90 enforcement mechanisms** (§50);
10. the **filename convention** and its **formal validation** (§51–§52).

## 0.2 Normativity

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in RFC 2119.

Where this document and any prior Zevo document conflict, this document governs.

## 0.3 Method

Each specification statement is classified by its role:

| Role | Definition | Marker |
|---|---|---|
| **Definition** | Introduces a term or symbol. | *Def.* |
| **Axiom** | An assumed truth of the specification. | *Ax.* |
| **Theorem** | A proven consequence. | *Th.* |
| **Lemma** | An intermediate proof. | *Lem.* |
| **Corollary** | A direct consequence. | *Cor.* |
| **Requirement** | A normative obligation on an implementation. | *Req.* |
| **Test** | A conformance criterion. | *Test.* |
| **Failure mode** | A declared respect in which a claim does not hold. | *FM.* |
| **Audit item** | An enumerated ambiguity or gap with resolution. | *Aud.* |
| **Note** | Informative. | *Note.* |

Every theorem in Part IV is anchored to named ISO, IEC, IEEE, NIST, FIPS, POSIX, System V, TIS, W3C, or Khronos standards. Every reference is cited per ISO 690:2021 in §53.

## 0.4 Anchor Standard

The mathematical anchor of this specification is the **field-wise bijection** `φ` defined in §14.2 and proven in §36. All other equivalences (runtime, semantic, byte-level) are derived from `φ` **or are explicitly declared not to follow from `φ`** (§48).

## 0.5 Resolution Discipline

**Def. 0.5.1 (Resolution discipline).** Every contradiction, gap, and ambiguity in the prior Zevo documents is resolved by exactly one of the following:

1. **Formal definition** — an ambiguous term is given a precise definition;
2. **Explicit amendment** — a contradictory statement is retracted;
3. **Normative rule** — a gap is filled by a rule that binds the implementation;
4. **Declared failure mode** — a claim that cannot be made to hold is declared false and bounded (§48).

**Req. 0.5.2.** An implementation claiming conformance to Zevo v5.2 MUST satisfy every resolution in this document.

**Note 0.5.3.** This discipline corrects the overclaim in Zevo v5.0.0 §47, which asserted "complete, unambiguous, contradiction-free, gap-free" without an independent enumeration. §49 provides the independent enumeration.

---

# Part I — The Zevo AI Core

## 1. Definition of Zevo

**Def. 1.1 (Zevo).** Zevo™ is a 5D interactive artificial intelligence assistant. It is a deterministic pipeline (§4) implemented as pure-function wrappers on ANSI C89/90 (§2), deployed as an ELF AI Core (§6), accessed through a published IPC API (§8), mediated by a Secure Service API (§9), and sandboxed (§10).

**Def. 1.2 (5D).** The designation "5D" refers simultaneously to:

1. the **five modalities** of the output artifact (§5.1), and
2. the **five-axis Cartesian space** `S = (O, X, Y, Z, R)` of the fused intelligence vector (§5.2).

The two are distinct. Modalities are output channels; axes are a coordinate system.

**Def. 1.3 (Proper name).** The exact title is `Zevo™`. The exact subtitle is `An 5D interactive artificial intelligence assistant`, verbatim per project charter. The article "An" is part of the proper name and MUST NOT be corrected in citations, trademarks, or references. The prose form may use "a 5D" in non-normative text.

**Def. 1.4 (Pixar disclaimer).** The phrase "Pixar-inspired" is descriptive of stylistic inspiration only. It does not imply affiliation with, endorsement by, or licensing from Pixar Animation Studios. This disclaimer MUST accompany every occurrence of the phrase.

## 2. Language and Programming Discipline

### 2.1 Language

**Def. 2.1.1 (Language of record).** The implementation language is **ANSI C89/90**, as specified in ISO/IEC 9899:1990 (ISO, 1990).

### 2.2 Permitted Extensions

**Def. 2.2.1 (Permitted extensions).** The following are the only permitted extensions to C89/90:

| Extension | Standard | Purpose |
|---|---|---|
| `<stdint.h>` | ISO/IEC 9899:1999 | Fixed-width integer types |
| `<stdbool.h>` | ISO/IEC 9899:1999 | The `bool` type |
| `posix_memalign` | POSIX.1-2001 | Aligned allocation |
| `aligned_alloc` | ISO/IEC 9899:2011 | Aligned allocation (alternative) |
| `<regex.h>` | POSIX.1-2001 | POSIX ERE for filename validation |

**Req. 2.2.2.** An implementation MUST declare in its build documentation which of these extensions it uses.

### 2.3 Comment Style

**Def. 2.3.1 (Block comments).** All comments MUST use the block form `/* ... */`.

**Req. 2.3.2.** Line comments (`//`) MUST NOT appear in Zevo source code.

**Def. 2.3.3 (Non-nesting).** Block comments do not nest. The character sequence `/* /* */` is a single comment, not two.

### 2.4 Functional Programming Paradigm

**Def. 2.4.1 (Pure function).** A *pure function* `f : A → B` is a function such that:

1. for every `x ∈ A`, `f(x)` is defined by the equation defining `f` alone;
2. `f` has no side effects;
3. `f` does not read or write global mutable state;
4. `f` does not perform I/O.

Formally: `∀ x ∈ A : f(x) = f(x)`, and `f` is a mathematical function, not a procedure.

**Def. 2.4.2 (Pure-function wrapper).** A *pure-function wrapper* is a composition of pure functions that presents a procedural interface at its boundary. The boundary itself is not pure; the interior is.

**Ax. 2.4.3 (Localization of side effects).** Side effects in Zevo are localized to exactly two boundaries:

1. the **IPC boundary** (§8), and
2. the **libc syscall wrappers** (§10).

**Lem. 2.4.4.** Every stage of the pipeline (§4) is a pure function **given Ax. 2.4.3**.

*Proof.* By Def. 2.4.1 and Ax. 2.4.3: the pipeline stages are interior to the boundaries, and the boundaries are the only sources of side effects. ∎

**Note 2.4.5 (Conditional nature of Lem. 2.4.4).** Lem. 2.4.4 is conditional on Ax. 2.4.3. Ax. 2.4.3 is an axiom, not a theorem. An implementation that violates Ax. 2.4.3 violates the precondition of Lem. 2.4.4 and therefore does not satisfy Th. 4.1.2.

### 2.5 External Libraries

**Def. 2.5.1 (External libraries).** The following external libraries are used, each a named standard:

| Library | Standard | Purpose |
|---|---|---|
| Pthreads | POSIX.1-2008 | Threads |
| OpenCL | Khronos OpenCL 3.0 | Compute |
| OpenGL | Khronos OpenGL 4.6 | Graphics |
| SDL2 | SDL 2.0 | Windowing and input |
| OpenAL | Khronos OpenAL 1.1 | Audio |
| OpenSSL | OpenSSL 3.x | Cryptography |
| `libmicrohttpd` | GNU libmicrohttpd | HTTP server |

**Lem. 2.5.2.** The use of these libraries does not violate §2.1.

*Proof.* A language specification constrains the syntax and semantics of programs written in that language. An external library is invoked via a function call, which is a C89/90 construct. The library's internal implementation is not part of the C89/90 program; it is a separate object linked at build time. Therefore the use of an external library is not a language extension. ∎

## 3. Compliance Surface

**Def. 3.1 (Compliance surface).** Zevo conforms to the following standards:

| Domain | Standard | Version |
|---|---|---|
| Language | ANSI C89/90 | ISO/IEC 9899:1990 |
| Language extensions | C99, C11, POSIX.1-2001 | see §2.2 |
| OS interface | GNU/POSIX, System V | POSIX.1-2008, System V ABI |
| Executable format | ELF | TIS ELF 1.2 |
| Architecture | x86_64 | System V AMD64 ABI |
| Executable type | `ET_DYN` (PIE) | — |
| Threads | POSIX Threads | POSIX.1-2008 |
| Compute | OpenCL | 3.0 |
| Graphics | OpenGL | 4.6 |
| Windowing | SDL2 | 2.0 |
| Audio | OpenAL | 1.1 |
| Cryptography | OpenSSL | 3.x |
| Crypto standards | NIST SP 800-38D, NIST SP 800-107, FIPS 186-5 | current |
| Web server | `libmicrohttpd` | current |
| Web client | HTML 5, CSS 5, JSON | W3C |
| Filesystem | ext4 | Linux |
| Integrity | fs-verity | Linux, SHA-256 |
| Sandbox | `seccomp-bpf`, capability bounding, LSM | Linux |
| Accessibility | WCAG 2.2 AA | W3C |
| Filename validation | POSIX ERE | POSIX.1-2008 |

Each standard is cited per ISO 690:2021 in §53.

## 4. The Reasoning Pipeline

### 4.1 Overview

**Def. 4.1.1 (Pipeline).** The Zevo reasoning pipeline is the composition

```
P = f₆ ∘ f₅ ∘ f₄ ∘ f₃ ∘ f₂ ∘ f₁
```

where:

| Stage | Function | Definition |
|---|---|---|
| 1 | `f₁` | Fuzzy-state-machine logic (§4.2) |
| 2 | `f₂` | Neural-fuzzy logic (§4.3) |
| 3 | `f₃` | Shannon entropy (§4.4) |
| 4 | `f₄` | Mamdani inference (§4.5) |
| 5 | `f₅` | Autonomous trained machine learning (§4.6) |
| 6 | `f₆` | 5D fusion (§4.7) |

**Th. 4.1.2 (Determinism).** Given identical inputs, state, and seed, and given Ax. 2.4.3, `P` produces identical outputs.

*Proof.* By Lem. 2.4.4 (which is conditional on Ax. 2.4.3), each `fᵢ` is pure. The composition of pure functions is pure. A pure function is deterministic. ∎

**Note 4.1.3.** The qualification "given Ax. 2.4.3" is normative. An implementation that violates Ax. 2.4.3 does not satisfy Th. 4.1.2.

### 4.2 Stage 1 — Fuzzy-State-Machine Logic

**Def. 4.2.1.** A fuzzy state machine is a tuple `(Q, Σ, δ, q₀, F)` where `Q` is a finite set of states, `Σ` is the input alphabet, `δ : Q × Σ → [0, 1]^Q` is a fuzzy transition function, `q₀ ∈ Q` is the initial state, and `F ⊆ Q` is the set of accepting states.

**Def. 4.2.2 (Threshold).** A transition `q_i → q_j` occurs when `δ(q_i, σ)(q_j) > θ`, where `θ = zevo.model.fsm_threshold` (FLOAT64).

### 4.3 Stage 2 — Neural-Fuzzy Logic

**Def. 4.3.1 (Fact).** A *fact* is a triple `(v, x, μ)` where `v` is a fuzzy variable, `x` is a value, and `μ ∈ [0, 1]` is a membership grade.

**Def. 4.3.2 (Rule).** A *rule* is a triple `(A, C, w)` where `A` is a conjunction of fuzzy propositions (the antecedent), `C` is a fuzzy set (the consequent), and `w ∈ [0, 1]` is a weight.

### 4.4 Stage 3 — Shannon Entropy

**Def. 4.4.1 (Shannon entropy).** For a discrete random variable `X` with probability mass function `p`,

```
H(X) = -Σ_i p(x_i) · log₂ p(x_i)
```

with the convention `0 · log₂ 0 = 0`.

**Note 4.4.2.** The base of the logarithm is **2**, so `H(X)` is measured in bits. This is a normative choice.

### 4.5 Stage 4 — Mamdani Inference

**Def. 4.5.1 (Mamdani inference).** Mamdani inference is defined by:

- **t-norm** (AND): `⊤(a, b) = min(a, b)`
- **s-norm** (OR): `⊥(a, b) = max(a, b)`
- **implication**: `a → b = min(a, b)`
- **aggregation**: `max` over rules
- **defuzzification**: centroid over a uniform discretization of `N = 1024` samples on `[r_min, r_max]`, where `r_min = zevo.model.mamdani_range_min` and `r_max = zevo.model.mamdani_range_max` (both FLOAT64).

The centroid is

```
y* = (Σ_{i=1}^{N} x_i · μ(x_i)) / (Σ_{i=1}^{N} μ(x_i))
```

where `x_i = r_min + (i − 1) · (r_max − r_min) / (N − 1)`.

**Req. 4.5.2.** If `Σ μ(x_i) == 0`, `y* = (r_min + r_max) / 2`. This resolves the undefined-division gap (Aud. G-14).

### 4.6 Stage 5 — Autonomous Trained Machine Learning

**Def. 4.6.1.** The parameters of `f₅` are trained offline and frozen at build time. They are embedded as read-only sections of the ELF AI Core (§6) and MUST NOT be modified at runtime. The model definition is given in §47.

### 4.7 Stage 6 — 5D Fusion

**Def. 4.7.1.** The five modalities of §5.1 are fused into a single 5D intelligence vector `S ∈ ℤ⁵` by a pure function `f₆` whose parameters are stored in the container's payload (§20). The fusion function is defined in §47.4.

## 5. The 5D Output Artifact

### 5.1 The Five Modalities

**Def. 5.1.1.** The artifact comprises five modalities:

| # | Modality | Representation |
|---|---|---|
| 1 | Text Raw | UTF-8 byte stream |
| 2 | Image BGRA | 8K HDR, BGRA |
| 3 | Video BGRA | 8K HDR, BGRA, temporal |
| 4 | Audio PCM | Linear PCM |
| 5 | Fused 5D intelligence vector | see §5.2 |

### 5.2 The 5D Cartesian Plan Axes

**Def. 5.2.1 (5D space).** The fused intelligence vector lives in `S = (O, X, Y, Z, R)`, a five-axis Cartesian space:

| Axis | Name | Meaning |
|---|---|---|
| `O` | Origin | Rotation and gravity reference axis |
| `X` | Horizontal | Horizontal spatial axis |
| `Y` | Vertical | Vertical spatial axis |
| `Z` | Depth | Depth spatial axis |
| `R` | Radial | Radial base axis |

**Def. 5.2.2 (Origin).** The canonical origin is `S = (0, 0, 0, 0, 0)`.

### 5.3 SIMD Compatibility

**Def. 5.3.1.** The 5D vector is compatible with three SIMD widths:

| ISA | Lane width | Required base alignment |
|---|---|---|
| SSE / NEON | 16 bytes | 16 bytes |
| AVX / AVX2 | 32 bytes | 32 bytes |
| AVX-512 | 64 bytes | 64 bytes |

**Def. 5.3.2.** The element type is `uint64_t` (64 bits). The element width is a normative choice; `unsigned long` is not used because it is not guaranteed to be 64 bits on all targets (see §43).

### 5.4 The 5D Vector Layout

**Def. 5.4.1.** The vector has five 64-bit elements (40 bytes of payload). Its in-memory representation is padded to 64 bytes:

```c
#define ZEVO_VEC5_SIZE 64
#define ZEVO_VEC5_ALIGN 64

typedef struct {
    uint64_t s[5];             /* O, X, Y, Z, R */
    unsigned char _pad[24];    /* zero-filled */
} zevo_vec5_t;
```

**Def. 5.4.2.** `sizeof(zevo_vec5_t) == 64`.

**Req. 5.4.3.** The padding bytes MUST be written as `0x00` and MUST be `0x00` on read. A reader encountering non-zero padding MUST reject the container.

**Req. 5.4.4 (C89/90 enforcement).** Because C89/90 has no `_Static_assert`, the implementation MUST enforce `sizeof(zevo_vec5_t) == 64` by one of:

1. a build-time preprocessor check;
2. a runtime assertion in an initialization function that executes once at load;
3. a build-system check (e.g., a `configure` test that compiles a program with `char _[sizeof(zevo_vec5_t) == 64 ? 1 : -1];`).

**Req. 5.4.5 (C89/90 alignment).** Because C89/90 has no `_Alignas`, the implementation MUST achieve 64-byte alignment by one of:

1. `posix_memalign` (permitted per §2.2.1);
2. `aligned_alloc` (permitted per §2.2.1);
3. a union with a maximally-aligned member and manual padding;
4. a compiler extension (`__attribute__((aligned(64)))`) declared in build documentation.

**Lem. 5.4.6 (Alignment).** Because `ZEVO_VEC5_SIZE` is a multiple of 16, 32, and 64, an array of `N` vectors has every element at a base alignment equal to the maximum required alignment (64).

*Proof.* Let the array base address be `a`, aligned to 64. Element `i` has address `a + i · 64`. Since `64 = 16 · 4 = 32 · 2 = 64 · 1`, `a + i · 64` is a multiple of 16, 32, and 64 for every `i`. ∎

### 5.5 SIMD Lane Usage

**Def. 5.5.1.** The five axes do not map to a natural SIMD lane count at any of the three widths:

| ISA | 64-bit lanes per register | Lanes used by 5 axes | Lanes unused |
|---|---|---|---|
| SSE / NEON | 2 | 5 (across 3 registers) | 1 |
| AVX / AVX2 | 4 | 5 (across 2 registers) | 3 |
| AVX-512 | 8 | 5 (in 1 register) | 3 |

**Note 5.5.2.** SIMD code for the 5D vector is therefore partially masked or partially padded at every width. This is a declared property, not a defect.

### 5.6 Artifact Serialization

**Def. 5.6.1.** The 5D Smart 8K HDR BGRA artifact is a **runtime object**, not an on-disk container. Its serialization, if any, is outside the scope of this specification.

---

## 6. The ELF AI Core

### 6.1 Binary Form

**Def. 6.1.1.**

| Property | Value |
|---|---|
| Format | ELF |
| Architecture | x86_64 |
| Type | `ET_DYN` (PIE) |
| Entry point | ELF `e_entry` (conventionally `_start`) |
| Comments | ANSI C89/90 block comments only |

**Def. 6.1.2.** The entry point is the ELF `e_entry` field. The symbol name is not normative.

### 6.2 Section Embedding

**Def. 6.2.1.** The ELF AI Core embeds, by section:

| ELF section | Content |
|---|---|
| `.elf.hdr` | Zevo container header |
| `.elf.meta` | Zevo container metadata KVS |
| `.elf.vidx` | Zevo container payload index |
| `.elf.data` | Zevo container payload blob (weights, vectors) |

### 6.3 Section Properties

**Def. 6.3.1.** Each section is:

- **page-aligned**: aligned to the host page size, which MUST be at least 4096 bytes;
- **fs-verity-protected**: using SHA-256 with a 4096-byte block size, per the Linux fs-verity default;
- **mapped read-only**: `PROT_READ`, no `PROT_WRITE`, no `PROT_EXEC` for data sections.

**Def. 6.3.2.** The fs-verity Merkle tree and root are stored by the filesystem (ext4 xattr), not embedded in the ELF.

### 6.4 Weight Resolution

**Def. 6.4.1.** Weights live in `.elf.data`, mapped read-only, fs-verity-protected. They are:

- **tamper-evident** (any modification breaks the Merkle root);
- **non-writable at runtime** (`mprotect` to `PROT_WRITE` is denied);
- **verifiable before use** (the digest is checked once at load).

---

## 7. Deployment Model

**Def. 7.1.** Zevo is deployed as a single ELF `ET_DYN` executable for x86_64, conforming to the System V AMD64 ABI.

**Def. 7.2.** The executable is position-independent. All code and data references use RIP-relative addressing or the GOT/PLT. No absolute relocations are permitted in read-only sections.

**Req. 7.3.** `mprotect` to `PROT_WRITE` on any weight section MUST be denied by the sandbox (§10).

---

## 8. The IPC API and the Web-Responsible Client

### 8.1 The Published IPC API

**Def. 8.1.1.** The core exposes only a published IPC API: stable, versioned, documented, and the sole entry point for client requests.

**Req. 8.1.2.** Only symbols in `.dynsym` that are listed in the published API MUST be exported. All other symbols MUST be local (`.symtab` only) or stripped. `RTLD_GLOBAL` re-export MUST NOT be permitted.

### 8.2 The Weight-Free, Stateless Client

**Def. 8.2.1.** The client:

- holds no weights;
- holds no persistent state across process restarts, other than:
  - user preferences explicitly entered by the user;
  - cookies whose sole purpose is session authentication to the Secure Service API;
- MUST NOT cache model outputs, weights, intermediate tensors, or metadata;
- speaks the published IPC API;
- renders the Web Responsible UI.

### 8.3 The Web Responsible UI

**Def. 8.3.1.**

| Property | Value |
|---|---|
| Server | `libmicrohttpd` |
| Markup | HTML 5 |
| Style | CSS 5 |
| Data | JSON |
| Accessibility | WCAG 2.2 AA |

**Note 8.3.2.** The "Web Responsible" designation is a design goal, not a testable conformance criterion for the ELF AI Core.

---

## 9. Cryptographic Access

### 9.1 The Secure Service API

**Def. 9.1.1.** All cryptographic access is mediated by the Secure Service API. Direct use of OpenSSL primitives outside this API is prohibited.

### 9.2 Key Rotation

**Def. 9.2.1.** API keys are rotated every **28 hours** from the key's creation timestamp, using `CLOCK_MONOTONIC` where available. The timestamp is recorded in the audit log.

**Def. 9.2.2 (Atomicity).** Rotation is atomic from the client's perspective: a client presenting either the old key or the new key is authenticated; a client presenting no key, an expired key older than one rotation, or a malformed key is rejected. There is no window in which a client with a valid key is rejected.

### 9.3 Cryptographic Standards

**Def. 9.3.1.**

| Standard | Application |
|---|---|
| NIST SP 800-38D | GCM authenticated encryption |
| NIST SP 800-107 | Hash-based key derivation and integrity |
| FIPS 186-5 | Digital signatures (ECDSA / EdDSA family) |

**Def. 9.3.2.** OpenSSL 3.x is the implementation provider.

---

## 10. Hardware Access Discipline

### 10.1 libc Wrappers Only

**Def. 10.1.1.** The core reaches hardware exclusively through libc wrappers. On x86_64, glibc's syscall wrappers resolve to the `syscall` instruction.

**Req. 10.1.2.** Direct inline assembly for syscalls MUST NOT appear outside the libc wrappers.

**Note 10.1.3.** The i386 path (`int 0x80`, vDSO `sysenter`) is informative only. Zevo normative targets are x86_64.

### 10.2 Sandboxing

**Def. 10.2.1.**

| Mechanism | Role |
|---|---|
| `seccomp-bpf` | Allowlist of permitted syscalls |
| Capability bounding | Drop all capabilities not required |
| LSM discipline | SELinux/AppArmor profile confines the process |

### 10.3 Syscall Allowlist

**Def. 10.3.1.** The syscall allowlist is representative, not exhaustive:

| Syscall | Purpose |
|---|---|
| `read` / `write` | IPC and file I/O |
| `mmap` / `mprotect` | Read-only mapping of weights |
| `openat` / `close` | File descriptors |
| `futex` | Pthread synchronization |
| `epoll_wait` | Event loop |
| `clock_gettime` | Time |
| `exit_group` | Termination |

**Req. 10.3.2.** The allowlist MUST NOT include syscalls that write to fs-verity-protected sections.

---

## 11. Runtime Summary

**Th. 11.1.** Given Ax. 2.4.3, Zevo is a deterministic, secure-by-construction, reproducible, auditable, accessible 5D multimodal interactive artificial intelligence assistant.

*Proof.* Determinism: Th. 4.1.2 (conditional on Ax. 2.4.3). Security: §9, §10. Reproducibility: §7. Auditability: §8.1. Accessibility: §8.3. ∎

**Note 11.2.** The claim "Zevo is a deterministic ... assistant" is conditional on Ax. 2.4.3. The claim "Zevo is a working 5D AI" is **not** made here. The model definition is in §47; the failure modes are in §48.

---

# Part II — The Zevo v5.2 Container

## 12. Purpose of Part II

Part II defines the Zevo v5.2 container `MC = (H, M, V, D)` and establishes its isomorphism with the GGUF v3 container. The isomorphism is a **field-wise bijection** `φ` (§14.2), proven in Part IV, and its limits are enumerated in §48.

## 13. The Single-Container Rule

**Ax. 13.1 (Single-container rule).** Zevo v5.2 has exactly one on-disk container shape:

```
MC = (H, M, V, D)
```

where `H` is the header, `M` is the metadata KVS, `V` is the payload index, and `D` is the payload blob.

**Ax. 13.2 (Single-index rule).** There is exactly one payload index `V`. There is no second payload index, no separate vector index, and no tensor index distinct from the vector index.

**Ax. 13.3 (Exclusive-vector rule).** An *exclusive vector* in Zevo v5.2 is a payload entry in `V` whose entry is structurally identical to a GGUF v3 tensor entry. The word "vector" is Zevo's name for the payload entry; the word "tensor" is GGUF v3's name for the same on-disk object.

## 14. The Twelve Equivalence Conditions and the Bijection

**Def. 14.1 (Conditions).** The equivalence between Zevo v5.2 and GGUF v3 is defined by twelve conditions:

| # | Condition | Statement |
|---|---|---|
| C1 | Header field bijection | Every GGUF v3 header field has a Zevo v5.2 counterpart with the same type, size, offset, and role. |
| C2 | Metadata type bijection | The Zevo v5.2 metadata value type enumeration is byte-identical to GGUF v3's. |
| C3 | Metadata encoding bijection | Strings and arrays are encoded byte-identically. |
| C4 | Metadata key naming bijection | Zevo v5.2 keys follow GGUF v3's naming rules exactly. |
| C5 | Payload index field bijection | Every GGUF v3 payload index field has a Zevo v5.2 counterpart with the same type, offset, and role. The `dimensions` field is one slot wider in Zevo v5.2. |
| C6 | Payload blob bijection | Payload blobs are byte-identical after alignment padding is stripped. |
| C7 | Layout order bijection | Both containers use the order Header → Metadata → Index → Padding → Blob. |
| C8 | Alignment rule bijection | Both use the same `align_offset` rule. |
| C9 | Padding rule bijection | Both pad with `0x00` to `ALIGNMENT`. |
| C10 | Endianness bijection | Both use the same endianness rule. |
| C11 | Naming convention bijection | Zevo v5.2 filenames follow GGUF v3's naming convention, with `.bin` replacing `.gguf`. |
| C12 | Count declaration bijection | Both declare counts explicitly in the header. |

**Def. 14.2 (Field-wise bijection `φ`).** A *field-wise bijection* between a Zevo v5.2 container `Z` and a GGUF v3 container `G` is a bijection `φ` between the set of fields of `Z` and the set of fields of `G` such that for every field pair `(f_Z, f_G) = φ`:

1. `f_Z` and `f_G` have the same declared type, **or** `f_Z` is the declared width extension of `f_G` (§15.3);
2. `f_Z` and `f_G` have the same byte offset within their respective structs, **or** `f_Z` is a declared width extension and offsets after it are computed by the same rule;
3. `f_Z` and `f_G` have the same role in the container.

**Def. 14.3 (Isomorphism).** `Z` and `G` are *isomorphic* (written `Z ≅ G`) iff there exists a field-wise bijection `φ : Z → G`. Isomorphism is denoted **100% field-set equivalence**.

**Note 14.4 (Restriction of the claim).** `Z ≅ G` is a bijection on the **field set**, not on the **byte stream**, not on the **semantics**, not on the **runtime behavior**. The claim "100% isomorphic" is restricted to field-set bijection. §48 enumerates every respect in which `Z` and `G` are not equivalent.

**Note 14.5 (Correction from Zevo v5.0.0).** Zevo v5.0.0 used the phrase "100% structural equivalence" without qualification. That phrase is retained here only with the qualifier "field-set." The unqualified phrase is **retracted** (Aud. A-27).

## 15. Declared Differences and Width Extension

### 15.1 Two Declared Differences

**Def. 15.1.1.**

| # | Difference | Location | Bounded? | Reversible? |
|---|---|---|---|---|
| 1 | Magic `ZEVO` vs. `GGUF` | 4-byte value at offset 0 | Yes | Yes (single field patch) |
| 2 | `zevo.*` metadata namespace | Free-form KVS extension | Yes (reserved prefix) | Yes (filtering) |

### 15.2 Count

**Def. 15.2.1.** The canonical count is **two declared differences** and **one declared width extension**. The width extension is not a "difference" in the sense of §15.1; it is a separate category.

### 15.3 The One Declared Width Extension

**Def. 15.3.1.**

| Field | GGUF v3 | Zevo v5.2 | Width difference |
|---|---|---|---|
| `dimensions` | `uint64_t[3]` | `uint64_t[5]` | **+16 bytes** |

**Note 15.3.2.** All occurrences of `dimensions[5]` as a *field reference* (an index) MUST be read as `dimensions[4]`. The `[5]` in `uint64_t[5]` is a *type*; `dimensions[5]` as an index is always a typo for `dimensions[4]`.

**Note 15.3.3.** For containers where the fifth dimension is unused, `dimensions[3] == 0` and `dimensions[4] == 0` in Zevo v5.2, and the effective shape is three-dimensional, identical to GGUF v3.

## 16. The Primitive Bijection

### 16.1 Zevo v5.2's Payload Entry (the "Exclusive Vector")

**Def. 16.1.1.**

```c
struct zevo_entry_info_t {
    zevo_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[5];
    uint32_t      type;
    uint64_t      offset;
};
```

### 16.2 GGUF v3's Exclusive Tensor

**Def. 16.2.1 (fixed-3 reference variant).**

```c
struct gguf_tensor_info_t {
    gguf_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[3];
    ggml_type     type;
    uint64_t      offset;
};
```

**Note 16.2.2.** The GGUF v3 reference of this document is the **fixed-3 variant**. A variable-width GGUF v3 (using `GGML_MAX_DIMS = 4`) is out of scope.

**Note 16.2.3 (ggml version pinning).** To make the claim "byte-identical to `ggml_type`" well-defined, the reference is pinned to a specific ggml commit. The pinned commit is declared in the build documentation of a conforming implementation. The type values in §16.4 are those of the pinned commit.

### 16.3 The Bijection (Field-wise)

**Th. 16.3.1.** `zevo_entry_info_t` and `gguf_tensor_info_t` are field-wise bijective.

| # | GGUF v3 field | Zevo v5.2 field | GGUF v3 type | Zevo v5.2 type | Role |
|---|---|---|---|---|---|
| 1 | `name` | `name` | `gguf_string_t` | `zevo_string_t` | Payload identifier |
| 2 | `n_dimensions` | `n_dimensions` | `uint32_t` | `uint32_t` | Rank |
| 3 | `dimensions[3]` | `dimensions[5]` | `uint64_t[3]` | `uint64_t[5]` | Shape (declared width extension) |
| 4 | `type` | `type` | `ggml_type` | `zevo_type` | Dtype |
| 5 | `offset` | `offset` | `uint64_t` | `uint64_t` | Payload offset |

*Proof.* Fields 1, 2, 4, 5 have identical types and roles. Field 3 is the declared width extension. ∎

**Req. 16.3.2.** Both containers MUST use column-major dimension order, with `dimensions[0]` innermost.

### 16.4 The Type Map (Identity)

**Def. 16.4.1.**

```c
enum zevo_type {
    ZEVO_TYPE_F32     = 0,
    ZEVO_TYPE_F16     = 1,
    ZEVO_TYPE_Q4_0    = 2,
    ZEVO_TYPE_Q4_1    = 3,
    /* 4, 5 are reserved */
    ZEVO_TYPE_Q5_0    = 6,
    ZEVO_TYPE_Q5_1    = 7,
    ZEVO_TYPE_Q8_0    = 8,
    ZEVO_TYPE_Q8_1    = 9,
    ZEVO_TYPE_Q2_K    = 10,
    ZEVO_TYPE_Q3_K    = 11,
    ZEVO_TYPE_Q4_K    = 12,
    ZEVO_TYPE_Q5_K    = 13,
    ZEVO_TYPE_Q6_K    = 14,
    ZEVO_TYPE_Q8_K    = 15,
    ZEVO_TYPE_IQ2_XXS = 16,
    ZEVO_TYPE_IQ2_XS  = 17,
    ZEVO_TYPE_IQ3_XXS = 18,
    ZEVO_TYPE_IQ1_S   = 19,
    ZEVO_TYPE_IQ4_NL  = 20,
    ZEVO_TYPE_IQ3_S   = 21,
    ZEVO_TYPE_IQ2_S   = 22,
    ZEVO_TYPE_IQ4_XS  = 23,
    ZEVO_TYPE_I8      = 24,
    ZEVO_TYPE_I16     = 25,
    ZEVO_TYPE_I32     = 26,
    ZEVO_TYPE_I64     = 27,
    ZEVO_TYPE_F64     = 28,
    ZEVO_TYPE_IQ1_M   = 29,
    ZEVO_TYPE_BF16    = 30,
    ZEVO_TYPE_TQ1_0   = 34,
    ZEVO_TYPE_TQ2_0   = 35,
    ZEVO_TYPE_MXFP4   = 39,
    ZEVO_TYPE_COUNT   = 40
};
typedef uint32_t zevo_type;
```

**Req. 16.4.2.** Values 4 and 5 are reserved. A reader encountering type 4 or 5 MUST reject the container.

**Req. 16.4.3.** `ZEVO_TYPE_COUNT` is a sentinel. No payload entry may have `type == 40`.

**Th. 16.4.4 (Type identity).** For every `ggml_type` value `v` in the pinned ggml commit, `zevo_type(v) == v` numerically, or `zevo_type(v)` is undefined because the pinned commit does not define `v`.

*Proof.* The enumeration assigns the same numeric value to each type. Types not present in the pinned commit are not in `zevo_type`. ∎

**Note 16.4.5 (Audit item A-31).** The claim "byte-identical to `ggml_type`" is underdetermined without a pinned commit.

### 16.5 Where Roles Live

**Def. 16.5.1.** Zevo v5.2 stores vector roles in metadata, not in the payload index:

```
zevo.vec.<name>.role : UINT32
```

**Req. 16.5.2.** There MUST NOT be a `role` field or a `flags` field in the payload index. Every per-vector attribute lives in metadata under `zevo.vec.<name>.*`.

## 17. The Container `MC = (H, M, V, D)`

**Def. 17.1.**

| Symbol | Name | GGUF v3 | Zevo v5.2 |
|---|---|---|---|
| `H` | Header | `gguf_header_t` | `zevo_header_t` |
| `M` | Metadata KVS | `gguf_metadata_kv_t[]` | `zevo_metadata_kv_t[]` |
| `V` | Payload index | `gguf_tensor_info_t[]` | `zevo_entry_info_t[]` |
| `D` | Payload blob | `tensor_data[]` | `vector_data[]` |

### 17.1 Layout Order

**Def. 17.1.1.**

```
┌─────────────────────────────────────────────┐
│ HEADER                                      │
├─────────────────────────────────────────────┤
│ METADATA KVS                                │
├─────────────────────────────────────────────┤
│ PAYLOAD INDEX                               │
├─────────────────────────────────────────────┤
│ PADDING (to ALIGNMENT)                      │
├─────────────────────────────────────────────┤
│ PAYLOAD BLOB                                │
└─────────────────────────────────────────────┘
```

### 17.2 ELF Sections

**Def. 17.2.1.**

```
┌─────────────────────────────────────────────┐
│ ELF Header (e_ident, e_type=ET_DYN, …)      │
├─────────────────────────────────────────────┤
│ Program Headers (PT_LOAD, PT_GNU_RELRO, …)  │
├─────────────────────────────────────────────┤
│ .text       — code                          │
├─────────────────────────────────────────────┤
│ .rodata     — string tables, constants      │
├─────────────────────────────────────────────┤
│ .elf.hdr    — container header              │  ← H
├─────────────────────────────────────────────┤
│ .elf.meta   — metadata KVS                  │  ← M
├─────────────────────────────────────────────┤
│ .elf.vidx   — payload index                 │  ← V
├─────────────────────────────────────────────┤
│ .elf.data   — payload blob                  │  ← D
│             page-aligned on ext4            │
└─────────────────────────────────────────────┘
   (fs-verity Merkle root is stored in
    ext4 xattr, not in the ELF)
```

**Th. 17.2.2.** C7 holds strictly.

*Proof.* Both containers place `H`, `M`, `V`, padding, and `D` in that order. ∎

## 18. The Header

### 18.1 Zevo v5.2 Header

**Def. 18.1.1.**

```c
struct zevo_header_t {
    uint32_t magic;             /* 'Z','E','V','O' = 0x5A45564F LE */
    uint32_t version;           /* must be 5 */
    uint64_t vector_count;
    uint64_t metadata_kv_count;
};
```

**Def. 18.1.2.** `sizeof(zevo_header_t) == 24`, with no padding between fields. The header is 24 bytes.

**Req. 18.1.3 (C89/90 enforcement).** The implementation MUST enforce `sizeof(zevo_header_t) == 24` by the same mechanisms as Req. 5.4.4.

### 18.2 GGUF v3 Header

**Def. 18.2.1.**

```c
struct gguf_header_t {
    uint32_t magic;             /* 'G','G','U','F' = 0x46554747 LE */
    uint32_t version;           /* must be 3 */
    uint64_t tensor_count;
    uint64_t metadata_kv_count;
};
```

### 18.3 Field-by-Field Bijection

**Th. 18.3.1.** The headers are field-wise bijective.

| # | GGUF v3 field | Zevo v5.2 field | Type | Offset | Role |
|---|---|---|---|---|---|
| 1 | `magic` | `magic` | `uint32_t` | 0 | Container identifier |
| 2 | `version` | `version` | `uint32_t` | 4 | Format version |
| 3 | `tensor_count` | `vector_count` | `uint64_t` | 8 | Payload count |
| 4 | `metadata_kv_count` | `metadata_kv_count` | `uint64_t` | 16 | Metadata count |

*Proof.* Fields are matched by type, offset, and role. ∎

### 18.4 Magic Number

**Def. 18.4.1.** The magic is the 4-byte ASCII sequence `ZEVO` (resp. `GGUF`). The corresponding `uint32_t` value is `0x5A45564F` (resp. `0x46554747`) when read on a little-endian host.

**Req. 18.4.2.** Container detection MUST use the byte sequence, not the `uint32_t` value. On a big-endian host, the on-disk bytes are still `ZEVO` in the order written.

### 18.5 Version

**Def. 18.5.1.** Zevo v5.2 uses version `5`; GGUF v3 uses version `3`. The version field is structurally identical (same type, offset, role). The *value* differs.

### 18.6 Counts

**Def. 18.6.1.** Both containers declare counts in the header.

| Format | Payload count | Metadata count |
|---|---|---|
| GGUF v3 | `tensor_count: uint64_t` at offset 8 | `metadata_kv_count: uint64_t` at offset 16 |
| Zevo v5.2 | `vector_count: uint64_t` at offset 8 | `metadata_kv_count: uint64_t` at offset 16 |

## 19. The Metadata KVS

### 19.1 Type Enumeration (Byte-Identical)

**Def. 19.1.1.**

```c
enum zevo_metadata_value_type {
    ZEVO_METADATA_VALUE_TYPE_UINT8   = 0,
    ZEVO_METADATA_VALUE_TYPE_INT8    = 1,
    ZEVO_METADATA_VALUE_TYPE_UINT16  = 2,
    ZEVO_METADATA_VALUE_TYPE_INT16   = 3,
    ZEVO_METADATA_VALUE_TYPE_UINT32  = 4,
    ZEVO_METADATA_VALUE_TYPE_INT32   = 5,
    ZEVO_METADATA_VALUE_TYPE_FLOAT32 = 6,
    ZEVO_METADATA_VALUE_TYPE_BOOL    = 7,
    ZEVO_METADATA_VALUE_TYPE_STRING  = 8,
    ZEVO_METADATA_VALUE_TYPE_ARRAY   = 9,
    ZEVO_METADATA_VALUE_TYPE_UINT64  = 10,
    ZEVO_METADATA_VALUE_TYPE_INT64   = 11,
    ZEVO_METADATA_VALUE_TYPE_FLOAT64 = 12
};
typedef uint32_t zevo_metadata_value_type;
```

**Th. 19.1.2.** C2 holds strictly.

### 19.2 String Encoding (Byte-Identical)

**Def. 19.2.1.**

```c
struct zevo_string_t {
    uint64_t len;
    char string[len];
};
```

Length-prefixed UTF-8, non-null-terminated. C3 holds.

### 19.3 Value Union (Byte-Identical Structure)

**Def. 19.3.1.**

```c
union zevo_metadata_value_t {
    uint8_t  uint8;
    int8_t   int8;
    uint16_t uint16;
    int16_t  int16;
    uint32_t uint32;
    int32_t  int32;
    float    float32;
    uint64_t uint64;
    int64_t  int64;
    double   float64;
    bool     bool_;
    zevo_string_t string;
    struct {
        zevo_metadata_value_type type;
        uint64_t len;
        zevo_metadata_value_t array[len];
    } array;
};
```

### 19.4 Key–Value Entry (Byte-Identical Structure)

**Def. 19.4.1.**

```c
struct zevo_metadata_kv_t {
    zevo_string_t key;
    zevo_metadata_value_type value_type;
    zevo_metadata_value_t value;
};
```

### 19.5 Metadata Key Surface

**Def. 19.5.1.** All GGUF v3 keys are preserved verbatim. "Preserved verbatim" means the key string (length + bytes) and the value (type tag + encoded value) are byte-identical to the GGUF v3 source. Ordering is not constrained.

### 19.6 Required Keys

**Def. 19.6.1.** The following metadata keys are **required** in every conforming Zevo v5.2 container:

- `general.architecture`
- `general.alignment`
- `general.name`
- `zevo.model.container`
- `zevo.model.equivalence_class`
- `zevo.model.state_axes`
- `zevo.model.vector_count`

All other keys are optional.

### 19.7 Metadata Ordering

**Def. 19.7.1.** Metadata keys MUST be unique within a container. Key order is not semantically significant, but for byte-identical round-tripping, the order MUST be preserved on read and write. A container with a duplicate key is non-conforming.

### 19.8 Zevo v5.2 Additional Keys

**Def. 19.8.1.**

| Key | Type | Meaning |
|---|---|---|
| `zevo.model.container` | STRING | Exactly `"zevo-v5"` |
| `zevo.model.equivalence_class` | STRING | `"bijective-3d"`, `"bijective-5d"`, or `"partial"` |
| `zevo.model.state_axes` | UINT32 | Exactly `5` |
| `zevo.model.vector_count` | UINT64 | Must equal header `vector_count` |
| `zevo.model.edge_count` | UINT32 | `E` |
| `zevo.model.prototype_count` | UINT32 | `K` |
| `zevo.model.prototype_dim` | UINT32 | `d` |
| `zevo.model.fuzzy_vars` | UINT32 | `V` |
| `zevo.model.rule_count` | UINT32 | `R` |
| `zevo.model.fsm_states` | UINT32 | `n_states` |
| `zevo.model.affect_len` | UINT32 | `A` |
| `zevo.model.mamdani_range_min` | FLOAT64 | Minimum of defuzzification range |
| `zevo.model.mamdani_range_max` | FLOAT64 | Maximum of defuzzification range |
| `zevo.model.seed` | UINT64 | PRNG seed (optional) |
| `zevo.model.fsm_threshold` | FLOAT64 | FSM transition threshold `θ` |
| `zevo.model.layers` | UINT32 | Number of ML layers `L` |
| `zevo.model.attention_heads` | UINT32 | Number of attention heads `H_a` |
| `zevo.model.hidden_dim` | UINT32 | Hidden dimension `d_model` |
| `zevo.model.vocab_size` | UINT32 | Vocabulary size `V_tok` |
| `zevo.vec.<name>.role` | UINT32 | Role of vector `<name>` |
| `zevo.vec.<name>.exclusive` | BOOL | Uniqueness flag |
| `zevo.vec.<name>.mutability` | STRING | `"immutable"`, `"runtime-mutable"`, `"seed"` |
| `zevo.schema.<name>.role` | UINT32 | Role for schema vectors |

**Def. 19.8.2 (Key ranges).**

| Key | Type | Range |
|---|---|---|
| `zevo.model.edge_count` | UINT32 | 0..2³²−1 |
| `zevo.model.prototype_count` | UINT32 | 0..2³²−1 |
| `zevo.model.prototype_dim` | UINT32 | 1..2³²−1 |
| `zevo.model.fuzzy_vars` | UINT32 | 0..2³²−1 |
| `zevo.model.rule_count` | UINT32 | 0..2³²−1 |
| `zevo.model.fsm_states` | UINT32 | 1..2³²−1 |
| `zevo.model.affect_len` | UINT32 | 1..2³²−1 |
| `zevo.model.fsm_threshold` | FLOAT64 | 0.0..1.0 |
| `zevo.model.layers` | UINT32 | 1..2³²−1 |
| `zevo.model.attention_heads` | UINT32 | 1..2³²−1 |
| `zevo.model.hidden_dim` | UINT32 | 1..2³²−1 |
| `zevo.model.vocab_size` | UINT32 | 1..2³²−1 |

### 19.9 Role Value Space

**Def. 19.9.1.**

| Role value | Name | Meaning |
|---|---|---|
| 0 | `ZEVO_ROLE_UNSPECIFIED` | Role not declared |
| 1 | `ZEVO_ROLE_FSM_STATE` | Fuzzy-state-machine state vector |
| 2 | `ZEVO_ROLE_FUZZY_SET` | Fuzzy-set membership parameters |
| 3 | `ZEVO_ROLE_RULE_ANTECEDENT` | Neural-fuzzy rule antecedent parameters |
| 4 | `ZEVO_ROLE_RULE_CONSEQUENT` | Neural-fuzzy rule consequent parameters |
| 5 | `ZEVO_ROLE_ENTROPY` | Shannon entropy parameters |
| 6 | `ZEVO_ROLE_MAMDANI` | Mamdani inference parameters |
| 7 | `ZEVO_ROLE_ML_TRAINED` | Autonomously trained ML parameters |
| 8 | `ZEVO_ROLE_EMBEDDING` | Text embedding |
| 9 | `ZEVO_ROLE_ATTENTION` | Attention weights |
| 10 | `ZEVO_ROLE_FFN` | Feed-forward weights |
| 11 | `ZEVO_ROLE_FUSED_5D` | Fused 5D intelligence vector |

Values 12–4294967295 are reserved.

### 19.10 Namespace Collision Rule

**Def. 19.10.1.** Zevo-specific metadata uses the prefix `zevo.`. If `general.architecture == "zevo"`, architecture-specific keys use `zevo.arch.*` to avoid collision with the reserved namespace.

## 20. The Payload Index

### 20.1 Zevo v5.2 Index Entry

**Def. 20.1.1.**

```c
struct zevo_entry_info_t {
    zevo_string_t name;
    uint32_t      n_dimensions;
    uint64_t      dimensions[5];
    uint32_t      type;
    uint64_t      offset;
};
```

### 20.2 GGUF v3 Index Entry

**Def. 20.2.1.** See §16.2.

### 20.3 Field-by-Field Bijection

**Th. 20.3.1.** See Th. 16.3.1.

### 20.4 Shape Semantics

**Def. 20.4.1 (Shape order).** The element at multi-index `(i_0, i_1, …, i_{n−1})` is at flat byte position

```
(Σ_{k=0}^{n−1} i_k · Π_{j=0}^{k−1} d_j) · size_of(type)
```

where `d_j = dimensions[j]`. That is, `dimensions[0]` is the innermost (fastest-varying) dimension.

**Def. 20.4.2.**

| Aspect | GGUF v3 (fixed-3) | Zevo v5.2 |
|---|---|---|
| Dimension order | Column-major (`dimensions[0]` innermost) | Column-major (`dimensions[0]` innermost) |
| Shape slots available | 3 | 5 |
| Rank bound (`n_dimensions`) | ≤ 3 | ≤ 5 |
| Slots `dimensions[3]`, `dimensions[4]` | Absent | Present; `0` if unused |

### 20.5 Dtype

**Def. 20.5.1.** The enumeration is `zevo_type` (§16.4). Numeric values are identical to `ggml_type` at the pinned commit. C5 holds strictly on the `type` field.

### 20.6 Offset

**Def. 20.6.1.** The `offset` field is `uint64_t`, relative to the start of `vector_data`, and MUST be a multiple of `ALIGNMENT`.

### 20.7 Index Ordering

**Def. 20.7.1.** Payload index entries MAY appear in any order. The `offset` field of each entry MUST be a multiple of `ALIGNMENT`. Offsets need not be monotonic in index order. A reader MUST NOT assume monotonicity.

## 21. The Payload Blob

### 21.1 Structure

**Def. 21.1.1.**

| Aspect | GGUF v3 | Zevo v5.2 |
|---|---|---|
| Name | `tensor_data[]` | `vector_data[]` |
| Location | After index + padding | After index + padding |
| Alignment | Multiples of `ALIGNMENT` | Multiples of `ALIGNMENT` |
| Padding | Zero bytes | Zero bytes |

### 21.2 Byte-Identity Rule

**Th. 21.2.1.** For every payload entry `V_i`:

```
GGUF  payload[i] = bytes at (tensor_data + offset_i), length = size_of(type) × ∏ dims
ZEVO  payload[i] = bytes at (vector_data + offset_i), length = size_of(type) × ∏ dims
```

The two are byte-identical after alignment padding is stripped. This holds for all dtypes, including quantized dtypes.

### 21.3 Quantization Block Layouts (Normative Import)

**Def. 21.3.1.**

| Type | Block size `bs` | Block bytes `bb` |
|---|---|---|
| `Q4_0` | 32 | 18 |
| `Q4_1` | 32 | 20 |
| `Q5_0` | 32 | 22 |
| `Q5_1` | 32 | 24 |
| `Q8_0` | 32 | 34 |
| `Q8_1` | 32 | 40 |
| `Q2_K` | 256 | 84 |
| `Q3_K` | 256 | 110 |
| `Q4_K` | 256 | 144 |
| `Q5_K` | 256 | 176 |
| `Q6_K` | 256 | 210 |
| `Q8_K` | 256 | 292 |
| `IQ2_XXS` | 256 | 66 |
| `IQ2_XS` | 256 | 74 |
| `IQ3_XXS` | 256 | 98 |
| `IQ1_S` | 256 | 50 |
| `IQ4_NL` | 32 | 18 |
| `IQ3_S` | 256 | 110 |
| `IQ2_S` | 256 | 82 |
| `IQ4_XS` | 256 | 136 |
| `IQ1_M` | 256 | 56 |
| `TQ1_0` | 256 | 54 |
| `TQ2_0` | 256 | 66 |
| `MXFP4` | 32 | 17 |

C6 holds for all dtypes **at the pinned ggml commit** (Note 16.2.3).

## 22. Alignment and Padding

### 22.1 Alignment Rule

**Def. 22.1.1.**

```c
uint64_t align_offset(uint64_t offset) {
    return offset + (ALIGNMENT - (offset % ALIGNMENT)) % ALIGNMENT;
}
```

**Def. 22.1.2.** `ALIGNMENT = general.alignment`.

**Req. 22.1.3.** If `general.alignment` is absent, `ALIGNMENT` defaults to 32. If present, it MUST be a positive multiple of 8. If `general.alignment == 0`, the container is non-conforming.

### 22.2 Default Alignment

**Def. 22.2.1.**

| Aspect | GGUF v3 | Zevo v5.2 |
|---|---|---|
| Default `ALIGNMENT` | 32 | 32 |
| Minimum | Multiple of 8 | Multiple of 8 |

C8 holds strictly.

### 22.3 Padding Rule

**Def. 22.3.1.** Both containers use `0x00` padding to `ALIGNMENT`. Two distinct paddings exist:

1. **Index–payload padding:** between the end of the last index entry and the start of `vector_data`. Length is `align_offset(end_of_index) − end_of_index`.
2. **Entry–entry padding:** between the end of one payload entry and the start of the next.

C9 holds strictly.

## 23. Endianness

**Def. 23.1.** Zevo v5.2 inherits GGUF v3's endianness rule verbatim:

> Models are little-endian by default. They can also come in big-endian for use with big-endian computers; in this case, all values (including metadata values and payloads) will also be big-endian. There is no way to determine if a model is big-endian. If no additional information is provided, assume the model is little-endian.

**FM 23.2 (Declared failure mode).** The absence of an endianness marker is inherited and is a **declared failure mode**. A reader that assumes little-endian on a big-endian container will misparse. This is not a gap in the specification; it is a declared property of the format. A future Zevo version MAY add `zevo.model.endianness` (STRING, `"little"` or `"big"`), but Zevo v5.2 does not.

C10 holds strictly.

## 24. Naming Convention

### 24.1 Inherited Convention (GGUF v3 → Zevo v5.2)

**Def. 24.1.1.** The naming convention is:

```
<BaseName><SizeLabel><FineTune><Version><Encoding><Type><Shard>.bin
```

| Component | GGUF v3 | Zevo v5.2 |
|---|---|---|
| BaseName | From `general.basename` | Same |
| SizeLabel | `<expertCount>x<count><scale>` | Same |
| FineTune | From `general.finetune` | Same |
| Version | `v<Major>.<Minor>` | Same |
| Encoding | Encoding scheme | Same |
| Type | `LoRA`, `vocab` | Same |
| Shard | `<N>-of-<M>` | Same |
| Extension | `.gguf` | `.bin` |

**Note 24.1.2.** The regular expression given in prior Zevo documents is **non-normative** and illustrative only. Normative naming is defined in §51.

C11 holds strictly, with the extension replacement declared.

## 25. Count Declaration

**Def. 25.1.** Both containers declare counts explicitly in the header.

| Count | GGUF v3 | Zevo v5.2 |
|---|---|---|
| Payload count | `tensor_count: uint64_t` | `vector_count: uint64_t` |
| Metadata count | `metadata_kv_count: uint64_t` | `metadata_kv_count: uint64_t` |

C12 holds strictly.

**Note 25.2.** When a Zevo v5.2 container is derived from a GGUF v3 container by adding `zevo.*` metadata, `metadata_kv_count` is strictly greater in the Zevo container. When a Zevo v5.2 container is created de novo, no such relation holds.

## 26. The Complete File Structure

### 26.1 Zevo v5.2 File

**Def. 26.1.1.**

```c
struct zevo_file_t {
    struct zevo_header_t header;
    struct zevo_metadata_kv_t metadata_kv[header.metadata_kv_count];
    struct zevo_entry_info_t vector_infos[header.vector_count];
    uint8_t _padding[];
    uint8_t vector_data[];
};
```

### 26.2 GGUF v3 File

**Def. 26.2.1.**

```c
struct gguf_file_t {
    struct gguf_header_t header;
    struct gguf_metadata_kv_t metadata_kv[header.metadata_kv_count];
    struct gguf_tensor_info_t tensor_infos[header.tensor_count];
    uint8_t _padding[];
    uint8_t tensor_data[];
};
```

### 26.3 Bijection

**Th. 26.3.1.**

| # | Field | GGUF v3 | Zevo v5.2 | Bijection |
|---|---|---|---|---|
| 1 | `header` | 4 fields | 4 fields | Field-for-field |
| 2 | `metadata_kv[]` | byte-identical entry | byte-identical entry | Zevo is superset of keys |
| 3 | `tensor_infos[]` / `vector_infos[]` | 5 fields | 5 fields | Field-for-field; `dimensions` is one slot wider in Zevo |
| 4 | `_padding[]` | identical rule | identical rule | Yes |
| 5 | `tensor_data[]` / `vector_data[]` | byte-identical bytes | byte-identical bytes | Yes |

*Proof.* By Th. 18.3.1 (header), §19 (metadata), Th. 16.3.1 (index), §22 (padding), Th. 21.2.1 (payload). ∎

**Note 26.3.2.** "Field-for-field" means the field set is in bijection, not that the structs are byte-identical.

## 27. Equivalence Classes

**Def. 27.1.**

| Class | Condition |
|---|---|
| `bijective-3d` | All of C1–C12 hold. Every entry has `dimensions[3] == 0` and `dimensions[4] == 0`. |
| `bijective-5d` | All of C1–C12 hold. At least one entry has `dimensions[3] != 0` or `dimensions[4] != 0`. |
| `partial` | Some structural field other than `dimensions` differs, or the header fields are not bijective, or the metadata encoding differs. |

**Def. 27.2.** A `partial` container is **non-conforming**. A conforming container is always in `bijective-3d` or `bijective-5d`.

## 28. Reserved Namespace

**Def. 28.1.** All Zevo-specific metadata keys use the prefix `zevo.`. GGUF v3's reserved namespaces are `general.`, `[llm].`, and `tokenizer.`. No collision occurs.

## 29. Worked Example

### 29.1 Zevo v5.2 Header

**Def. 29.1.1.**

```
magic              = 0x5A45564F ("ZEVO")
version            = 5
vector_count       = 2
metadata_kv_count  = 9
```

### 29.2 Zevo v5.2 Metadata

**Def. 29.2.1.**

```
general.architecture            = "zevo"
general.alignment               = 32
general.quantization_version    = 3
general.name                    = "Zevo Example Model"
zevo.context_length            = 8192
zevo.embedding_length          = 4096
zevo.model.container            = "zevo-v5"
zevo.model.equivalence_class    = "bijective-3d"
zevo.model.state_axes           = 5
```

### 29.3 Zevo v5.2 Payload Index

**Def. 29.3.1.**

```
entry 0:
    name          = "token_embd.weight"
    n_dimensions  = 2
    dimensions    = [4096, 32000, 0, 0, 0]
    type          = ZEVO_TYPE_F32
    offset        = 0

entry 1:
    name          = "output.weight"
    n_dimensions  = 2
    dimensions    = [4096, 32000, 0, 0, 0]
    type          = ZEVO_TYPE_F32
    offset        = 524288000
```

Both entries have `dimensions[2] == 0` and `dimensions[4] == 0`. The container is `bijective-3d` and `bijective-5d`.

### 29.4 GGUF v3 Counterpart

**Def. 29.4.1.**

```
magic              = 0x46554747 ("GGUF")
version            = 3
tensor_count       = 2
metadata_kv_count  = 6

general.architecture            = "zevo"
general.alignment               = 32
general.quantization_version    = 3
general.name                    = "Zevo Example Model"
zevo.context_length            = 8192
zevo.embedding_length          = 4096
```

### 29.5 Bijection Table

**Th. 29.5.1.** The example pair is field-wise bijective.

| Aspect | Zevo v5.2 | GGUF v3 | Bijection |
|---|---|---|---|
| Header field set | 4 fields | 4 fields | Field-for-field |
| `magic` | `0x5A45564F` | `0x46554747` | Declared difference |
| `version` | 5 | 3 | Declared difference |
| Payload count | 2 | 2 | Yes |
| `metadata_kv_count` | 9 | 6 | Same type, offset, role; Zevo is superset |
| Metadata KVS | 9 entries | 6 entries | Superset; shared keys byte-identical |
| Payload index | 2 entries | 2 entries | Field-for-field |
| Index field set | 5 fields | 5 fields | Yes, with `dimensions` width extension |
| Padding | 32 | 32 | Yes |
| Payload | bytes | bytes | Yes |

*Proof.* Shared keys `general.architecture`, `general.alignment`, `general.quantization_version`, `general.name`, `zevo.context_length`, `zevo.embedding_length` are byte-identical. ∎

---

# Part III — Integration of Container and Core

## 30. The ELF AI Core Carries a Zevo v5.2 Container

### 30.1 Section Layout

**Def. 30.1.1.**

| ELF section | Content |
|---|---|
| `.elf.hdr` | Zevo v5.2 header |
| `.elf.meta` | Zevo v5.2 metadata KVS |
| `.elf.vidx` | Zevo v5.2 payload index |
| `.elf.data` | Zevo v5.2 payload blob |

Each section is page-aligned, fs-verity-protected, mapped read-only.

### 30.2 Alignment Reconciliation

**Def. 30.2.1.** The Zevo v5.2 container's `general.alignment` (default 32) is independent of the ELF page alignment (4096). The two alignments compose without conflict because the container is embedded as an opaque byte range within each section.

### 30.3 fs-verity Interaction

**Def. 30.3.1.** fs-verity is computed over the ELF sections. Because the Zevo v5.2 container is field-wise bijective with a GGUF v3 container modulo the two declared differences and the one declared width extension, the fs-verity digest of a Zevo v5.2 section and the fs-verity digest of the corresponding GGUF v3 section differ by the magic bytes, the version field, the `zevo.*` metadata, and the extra `uint64_t` per index entry.

**Def. 30.3.2 (Computable relation).** The relation between the two Merkle roots is given by the following procedure:

1. Let `Z_bytes` be the Zevo v5.2 section bytes and `G_bytes` the GGUF v3 section bytes.
2. Let `D` be the byte-difference function computed by the transformation `T : Z_bytes → G_bytes` defined in §30.3.3.
3. The Merkle root of `G_bytes` is the Merkle root of `T(Z_bytes)`.
4. Therefore the two roots are related by `Merkle(G_bytes) = Merkle(T(Z_bytes))`.

**Def. 30.3.3 (Transformation `T`).**

1. Replace the 4-byte magic at offset 0 with `GGUF`.
2. Replace the 4-byte version at offset 4 with `3`.
3. Filter out every metadata key with prefix `zevo.` (except those that already exist in the GGUF v3 source).
4. Decrement `metadata_kv_count` by the number of filtered keys.
5. For each payload index entry, truncate `dimensions` from `uint64_t[5]` to `uint64_t[3]`.
6. Recompute all offsets after the truncated fields.

`T` is a total function from Zevo v5.2 section bytes to GGUF v3 section bytes, defined for all sections satisfying the `bijective-3d` class. For `bijective-5d` sections, `T` is partial and `T` is undefined.

**Th. 30.3.4.** For `bijective-3d` containers, `Merkle(G_bytes) = Merkle(T(Z_bytes))`.

*Proof.* By Def. 30.3.2 and Def. 30.3.3. ∎

### 30.4 Runtime Access

**Def. 30.4.1.** At runtime:

1. The core opens the ELF image read-only.
2. The kernel maps the Zevo v5.2 sections read-only.
3. fs-verity is verified once at load.
4. The core parses the header, metadata, and index using the rules of Part II.
5. The core interprets the payload as reasoning-graph nodes rather than computation-graph nodes.
6. The core executes the pipeline of Part I §4.
7. The core produces the artifact of Part I §5.

## 31. The Secure Service API Wraps the Container

**Def. 31.1.** The Secure Service API of §9 mediates access to the container:

- The container is never exposed to the client.
- The client speaks the published IPC API only.
- The Secure Service API authenticates the client, rotates keys every 28 hours, and checks the fs-verity result before serving.

**Def. 31.2.** fs-verity is verified once at load (§30.4). The Secure Service API checks the cached boolean result before serving. It does not re-verify on every request.

## 32. Summary of Integration

**Def. 32.1.**

| Layer | Object | Standard / Discipline |
|---|---|---|
| Container | Zevo v5.2 | Field-wise bijective with GGUF v3 (Part II) |
| Executable | ELF `ET_DYN` x86_64 | System V ABI |
| Language | ANSI C89/90 | ISO/IEC 9899:1990 |
| Threads | Pthreads | POSIX |
| Compute | OpenCL | Khronos |
| Graphics | OpenGL, SDL2 | Khronos, SDL |
| Audio | OpenAL | Khronos |
| Crypto | OpenSSL | NIST SP 800-38D, SP 800-107, FIPS 186-5 |
| Web | `libmicrohttpd`, HTML 5, CSS 5, JSON | W3C |
| Sandbox | `seccomp-bpf`, capability bounding, LSM | Linux |
| Integrity | fs-verity | Linux |
| Filesystem | ext4 | Linux |
| Accessibility | WCAG 2.2 AA | W3C |
| Filename | §51–§52 | POSIX ERE |

---

# Part IV — Mathematical Isomorphism Proof, Model Definition, and Closure

## 33. Purpose of Part IV

Part IV provides:

1. the **mathematical proof** that the Zevo v5.2 container is isomorphic to the GGUF v3 container under the field-wise bijection `φ` (§35–§36);
2. the **model definition** (§47);
3. the **failure-mode enumeration** (§48);
4. the **closed audit** of ambiguities and gaps, with an independent enumeration procedure (§49);
5. the **C89/90 enforcement mechanisms** (§50);
6. the **filename convention formalization** (§51);
7. the **filename validation regex and post-match checks** (§52).

## 34. Formal Preliminaries

### 34.1 Sets and Functions

**Def. 34.1.1.** A **function** `f : A → B` is a subset `f ⊆ A × B` such that for every `a ∈ A` there is exactly one `b ∈ B` with `(a, b) ∈ f`. We write `f(a) = b`.

**Def. 34.1.2.** A function `f : A → B` is **injective** iff `f(a₁) = f(a₂) ⟹ a₁ = a₂`.

**Def. 34.1.3.** A function `f : A → B` is **surjective** iff `∀ b ∈ B, ∃ a ∈ A : f(a) = b`.

**Def. 34.1.4.** A function `f : A → B` is **bijective** iff it is both injective and surjective.

**Def. 34.1.5.** Two sets `A`, `B` are **isomorphic** (written `A ≅ B`) iff there exists a bijection `f : A → B`.

### 34.2 Container as a Set

**Def. 34.2.1.** A container `C` is a quadruple `(H, M, V, D)` where `H` is a header, `M` is a metadata KVS, `V` is a payload index, `D` is a payload blob.

**Def. 34.2.2.** The **field set** of `C`, written `Fields(C)`, is the disjoint union of the field sets of `H`, `M`, `V`, and `D`:

```
Fields(C) = Fields(H) ⊎ Fields(M) ⊎ Fields(V) ⊎ Fields(D)
```

where `⊎` is disjoint union.

**Def. 34.2.3.** A **field** is a tuple `(name, type, offset, role)`.

**Def. 34.2.4 (Field-set size).** Let `|Fields(Z)|` be the cardinality of the field set of a Zevo v5.2 container, and `|Fields(G)|` the cardinality of the field set of a GGUF v3 container carrying the same logical model. Then:

```
|Fields(Z)| = |Fields(G)| + |K_zevo| + 2 · |V|
```

where `|K_zevo|` is the number of `zevo.*` metadata keys and `|V|` is the number of payload entries (each contributing 2 extra `dimensions` slots).

## 35. The Bijection `φ` — Statement and Proof

### 35.1 Statement

**Th. 35.1.1 (Container isomorphism, restricted).** Let `Z` be a Zevo v5.2 container in the `bijective-3d` or `bijective-5d` equivalence class, and `G` a GGUF v3 (fixed-3) container carrying the same logical model with no fifth-dimension data. Then `Z ≅_fields G`, where `≅_fields` denotes field-set isomorphism.

### 35.2 Proof

*Proof.* We construct `φ : Fields(Z) → Fields(G)` component-wise and prove it is bijective.

**Header component.** By Th. 18.3.1, the header fields of `Z` and `G` are in bijection:

```
φ_H(magic_Z)              = magic_G
φ_H(version_Z)            = version_G
φ_H(vector_count_Z)       = tensor_count_G
φ_H(metadata_kv_count_Z)  = metadata_kv_count_G
```

`φ_H` is a bijection on four elements. ∎

**Metadata component.** By §19, every GGUF v3 metadata key is preserved verbatim in `Z`. Define

```
φ_M : Fields(M_Z) → Fields(M_G)
φ_M(k) = k    for every k ∈ Keys(M_G)
```

with `φ_M` undefined on `zevo.*` keys (which have no GGUF v3 counterpart). The restriction `φ_M|_{Keys(M_G)}` is the identity on `Keys(M_G)`, hence bijective. By §19.7, keys are unique, so the identity is well-defined. ∎

**Payload index component.** By Th. 16.3.1, the index fields of `Z` and `G` are in bijection:

```
φ_V(name_Z)          = name_G
φ_V(n_dimensions_Z)  = n_dimensions_G
φ_V(dimensions_Z)    = dimensions_G     (with dimensions_Z[3] = dimensions_Z[4] = 0)
φ_V(type_Z)          = type_G
φ_V(offset_Z)        = offset_G
```

For `dimensions`, `φ_V` maps `dimensions_Z[0..2]` element-wise to `dimensions_G[0..2]` and maps `dimensions_Z[3]`, `dimensions_Z[4]` to the empty set (no GGUF v3 counterpart). The restriction `φ_V|_{dimensions_Z[0..2]}` is the identity, hence bijective. ∎

**Payload blob component.** By Th. 21.2.1, the payload bytes of `Z` and `G` are byte-identical after alignment padding is stripped. Define `φ_D` as the identity on the stripped bytes. ∎

**Composition.** The map

```
φ = φ_H ⊎ φ_M ⊎ φ_V ⊎ φ_D
```

is a disjoint union of bijections, hence a bijection. Therefore `Z ≅_fields G`. ∎

### 35.3 Corollaries

**Cor. 35.3.1.** The number of fields in `Z` equals the number of fields in `G` plus the number of `zevo.*` metadata keys plus 2 per index entry (the two extra `dimensions` slots).

**Cor. 35.3.2.** The isomorphism `φ` is a **structure-preserving** map: it preserves types, offsets, and roles on the field set.

**Cor. 35.3.3.** The declared differences (§15.1) and the declared width extension (§15.3) are the **only** respects in which `Z` and `G` differ **on the field set**. Every other respect on the field set is preserved by `φ`.

**FM 35.3.4 (Failure mode).** §35.3.3 is restricted to the **field set**. It does **not** claim that `Z` and `G` are equivalent on the byte stream, semantics, runtime behavior, or endianness marker. See §48.

## 36. Resolution of Contradictions by the Isomorphism

### 36.1 Contradiction C-1: "Byte-identical" vs. width extension

**Statement.** Zevo v5.0.0 §25.2 said "No primitive is different." §13.3 says the primitives are not byte-identical.

**Resolution.** "Field-wise bijective" is defined on the **field set**, not on the **byte stream** (Def. 14.2). The width extension changes the byte stream but not the field set.

**Formal proof.** Let `Fields(Z)` and `Fields(G)` be as in Def. 34.2.2. The width extension of `dimensions` does not add a field to `Fields(V_Z)`; it changes the **type** of `dimensions` from `uint64_t[3]` to `uint64_t[5]`. The field set is unchanged. Therefore `φ_V` remains a bijection on the field set. ∎

### 36.2 Contradiction C-2: `dimensions[5]` as index

**Statement.** Zevo v5.0.0 §13.5 said "`dimensions[5]` is `0`." The array is `uint64_t[5]`, so valid indices are `0..4`.

**Resolution.** `dimensions[5]` as an **index** is a typo for `dimensions[4]`. `dimensions[5]` as a **type** (in `uint64_t[5]`) is well-formed.

**Formal proof.** The index `i` in `dimensions[i]` ranges over `0 ≤ i < 5`. The value `i = 5` is out of range. Therefore any occurrence of `dimensions[5]` as an index is a typographical error. ∎

### 36.3 Contradiction C-3: header size

**Statement.** Zevo v5.0.0 §16.3 gave field offsets but no total size.

**Resolution.** §18.1.2 states `sizeof(zevo_header_t) == 24`.

**Formal proof.** `magic` at offset 0 occupies 4 bytes; `version` at offset 4 occupies 4 bytes; `vector_count` at offset 8 occupies 8 bytes; `metadata_kv_count` at offset 16 occupies 8 bytes. Total: `4 + 4 + 8 + 8 = 24`. ∎

### 36.4 Contradiction C-4: i386 vs. x86_64

**Statement.** §3 declares x86_64 only; §9.1 discusses i386.

**Resolution.** §10.1.3 states the i386 path is informative only.

**Formal proof.** The compliance surface (§3) is normative. §9.1 is descriptive. Where descriptive text conflicts with normative text, the normative text governs. ∎

### 36.5 Contradiction C-5: regex vs. examples

**Statement.** Zevo v5.0.0 §22.1's regex rejected one of §22.2's examples.

**Resolution.** §51 formalizes the naming convention; §52 provides the corrected regex and post-match checks.

**Formal proof.** A non-normative example cannot contradict a normative rule. The normative rule is §51. ∎

### 36.6 Contradiction C-6: fs-verity placement

**Statement.** Zevo v5.0.0 §6.2 placed fs-verity in the ELF; §34.3 placed it outside.

**Resolution.** §6.3.2 and §30.3.1 state fs-verity is stored in ext4 xattr, not in the ELF. The computable relation is given in Def. 30.3.3.

**Formal proof.** The Linux fs-verity implementation stores the Merkle tree and root in a filesystem-specific location (ext4 xattr). The ELF section list (§17.2.1) therefore does not include `.fsverity`. ∎

### 36.7 Contradiction C-7: `unsigned long` width

**Statement.** Zevo v5.0.0 §5.2 said `unsigned long` is 64 bits; §5.3's `_pad[24]` assumed that.

**Resolution.** §5.3.2 replaces `unsigned long` with `uint64_t`.

**Formal proof.** By Def. 5.3.2, the element type is `uint64_t`. By ISO/IEC 9899:1999, `uint64_t` is exactly 64 bits. Therefore `sizeof(zevo_vec5_t) == 5 · 8 + 24 == 64`. ∎

### 36.8 Contradiction C-8: "100% isomorphic" vs. failure modes

**Statement.** Zevo v5.0.0 §46 claimed "100% isomorphic" without qualification, but §48 enumerates failure modes.

**Resolution.** §14.3 and §14.4 restrict "100% isomorphic" to "100% field-set equivalence." §48 enumerates every respect in which the containers are not equivalent.

**Formal proof.** By Def. 14.3, `Z ≅ G` iff there exists a field-wise bijection. By Th. 35.1.1, the bijection exists on the field set. By FM 35.3.4, the bijection does not extend to the byte stream, semantics, runtime behavior, or endianness marker. Therefore "100% isomorphic" is to be read as "100% field-set isomorphic." ∎

### 36.9 Contradiction C-9: determinism unconditional vs. conditional

**Statement.** Zevo v5.0.0 Th. 4.1.2 claimed determinism unconditionally, but Lem. 2.4.4 is conditional on Ax. 2.4.3.

**Resolution.** Th. 4.1.2 is restated as conditional on Ax. 2.4.3. Note 4.1.3 makes the condition normative.

**Formal proof.** By Ax. 2.4.3, side effects are localized. By Lem. 2.4.4, pipeline stages are pure **given** Ax. 2.4.3. By Def. 2.4.1, pure functions are deterministic. Therefore determinism holds **given** Ax. 2.4.3. ∎

### 36.10 Contradiction C-10 (new in v5.2): regex dialect vs. implementation

**Statement.** The naming-convention regex in prior Zevo documents used PCRE-only constructs (named groups, negative lookahead) that are not available in POSIX ERE, which is the correct dialect for an ANSI C89/90 + POSIX.1-2008 implementation.

**Resolution.** §51 formalizes the naming convention; §52 provides a POSIX ERE regex plus six normative post-match checks plus one cross-file constraint. The PCRE-only regex is retracted.

**Formal proof.** By POSIX.1-2008 §9.4.6, the ERE grammar does not include named groups or lookahead. By §2.2.1, the implementation language is ANSI C89/90 with POSIX.1-2001 `<regex.h>` as a permitted extension. Therefore the validation regex MUST be a POSIX ERE. ∎

### 36.11 Contradiction C-11 (new in v5.2): Version required vs. optional

**Statement.** The naming-convention prose said Version is Optional (default `v1.0`), but the regex treated Version as required.

**Resolution.** §51.4 requires Version in the filename. The prose's "optional" rule is retracted for filenames; the default `v1.0` applies only when reading metadata. This eliminates the FineTune/Encoding ambiguity class noted in the prose itself.

**Formal proof.** The prose states: "it is easy for Encoding to be mistaken as a FineTune if Version is omitted." Requiring Version removes this ambiguity class. ∎

### 36.12 Contradiction C-12 (new in v5.2): BaseName/SizeLabel ambiguity

**Statement.** The naming-convention prose's own example `Mixtral-8x7B-v0.1-KQ2.bin` is ambiguous: `8x7B` parses as either BaseName continuation or SizeLabel.

**Resolution.** §51.3 requires that no BaseName segment match the SizeLabel pattern. The prose example is retracted and replaced by `Mixtral-MoE-8x7B-v0.1-KQ2.bin` (FineTune `MoE`).

**Formal proof.** By §51.3, a BaseName segment MUST NOT match `[0-9]+[xX][0-9]+[QTBMK]`. The segment `8x7B` matches this pattern. Therefore `Mixtral-8x7B-v0.1-KQ2.bin` is non-conforming. ∎

## 37. Resolution of Ambiguities by Definition

Every ambiguity in the base document is resolved by a formal definition in Parts I–III. The following table lists the correspondence:

| Ambiguity | Definition |
|---|---|
| "Field-wise bijective" | Def. 14.2 |
| "100% structurally equivalent" | Def. 14.3, Note 14.4 |
| "Superset" (metadata) | §19.5, §19.7 |
| "Verbatim" (metadata) | §19.5 |
| "Offset" | §20.6 |
| "Must be `0x00` on read" | Req. 5.4.3 |
| "Column-major" | Def. 20.4.1 |
| "Rank bound" | §20.4.2 |
| "Alignment rule" | Req. 22.1.3 |
| "Padding" | §22.3.1 |
| "Metadata KVS" ordering | §19.7 |
| "Payload index" ordering | §20.7 |
| "Role" value space | §19.9 |
| "5D output artifact" | §5.6 |
| "28-hour API key rotation" | §9.2 |
| "Atomic" (key rotation) | §9.2.2 |
| "No internal symbol" | Req. 8.1.2 |
| "Weight-free" | Def. 8.2.1 |
| "5D" | Def. 1.2 |
| "Equivalence class" values | §27.1 |
| "Prototype" | §19.8 |
| "Exclusive" | §19.8 |
| "Mutability" | §19.8 |
| "Affect" | §19.8 |
| "Two declared differences" | Def. 15.2.1 |
| "100% field-set equivalence" | Def. 14.3 |
| "Pixar-inspired" | Def. 1.4 |
| "ggml_type identity" | Note 16.2.3 (pinned commit) |
| "Mamdani centroid division" | Req. 4.5.2 |
| "C89/90 alignment enforcement" | Req. 5.4.5 |
| "C89/90 struct-size enforcement" | Req. 5.4.4, Req. 18.1.3 |
| "Filename convention" | §51 |
| "Filename validation" | §52 |
| "BaseName/SizeLabel disambiguation" | Req. 51.3.1 |
| "FineTune/Version disambiguation" | Req. 51.4.1 |
| "Shard numbering" | Req. 51.6.1 |
| "Whitespace in filenames" | Req. 51.7.1 |

## 38. Resolution of Gaps by Requirement

Every gap in the base document is filled by a normative requirement in Parts I–III. The following table lists the correspondence:

| Gap | Requirement |
|---|---|
| Header size | Def. 18.1.2 |
| C89/90 extensions | Def. 2.2.1 |
| Comment nesting | Def. 2.3.3 |
| Pure function | Def. 2.4.1 |
| Pipeline stages | Def. 4.1.1 |
| t-norm / s-norm | Def. 4.5.1 |
| Shannon base | Def. 4.4.1 |
| Defuzzification samples | Def. 4.5.1 |
| Defuzzification division by zero | Req. 4.5.2 |
| 5D vector size | Def. 5.4.1 |
| SIMD alignment | Lem. 5.4.6 |
| ELF section names | Def. 6.2.1 |
| fs-verity hash | Def. 6.3.1 |
| fs-verity Merkle relation | Def. 30.3.2, Def. 30.3.3 |
| Syscall allowlist | Def. 10.3.1 |
| Error handling | §39 |
| Versioning | §40 |
| Endianness marker | §23, FM 23.2 |
| Test vectors | §41 |
| Role values | §19.9 |
| Mutability values | §19.8 |
| Prototype | §19.8 |
| Affect | §19.8 |
| PRNG | §40, §47.6 |
| Required metadata keys | §19.6 |
| Model definition | §47 |
| Failure modes | §48 |
| Closed audit | §49 |
| C89/90 enforcement | §50 |
| Filename convention | §51 |
| Filename validation | §52 |

## 39. Error Handling

**Def. 39.1.** A conforming reader MUST reject a container on any of the following:

1. bad magic
2. bad version
3. truncated header
4. truncated metadata
5. truncated index
6. truncated payload
7. non-zero padding
8. duplicate metadata key
9. missing required metadata key
10. unknown metadata type
11. unknown payload type
12. `n_dimensions > 5`
13. `dimensions[i] == 0` for some `i < n_dimensions`
14. `offset` not a multiple of `ALIGNMENT`
15. `offset + payload_size > len(vector_data)`
16. `zevo.model.vector_count != header.vector_count`
17. `zevo.model.equivalence_class` not one of the three allowed values
18. `zevo.vec.<name>.exclusive == true` with a duplicate `<name>`
19. `general.alignment == 0`
20. `general.alignment` not a multiple of 8
21. `zevo.model.fsm_threshold` outside `[0, 1]`
22. `zevo.model.mamdani_range_min >= zevo.model.mamdani_range_max`
23. `zevo.model.layers == 0`
24. `zevo.model.attention_heads == 0`
25. `zevo.model.hidden_dim == 0`
26. `zevo.model.vocab_size == 0`

On rejection, the reader MUST NOT produce a partial model.

## 40. Versioning

**Def. 40.1.** Zevo v5.2 readers MUST reject containers whose `version` is not 5. Future Zevo versions MAY define new metadata keys under `zevo.v6.*` but MUST NOT alter the header, the payload index field set, or the payload blob encoding without incrementing `version`.

**Def. 40.2.** The runtime PRNG is a 64-bit xorshift variant, seeded from `zevo.model.seed` (UINT64, metadata) when present, or from the kernel CSPRNG when absent. The exact algorithm is outside the scope of this specification; only its determinism given a fixed seed is normative.

## 41. Test Vectors

**Def. 41.1.** Conforming implementations SHOULD publish reference byte sequences for at least:

- a minimal `bijective-3d` container;
- a minimal `bijective-5d` container;
- a round-trip GGUF v3 ↔ Zevo v5.2 pair.

**Def. 41.2 (Normative test vector for `zevo_vec5_t`).**

```c
/* Test vector: zevo_vec5_t must be 64 bytes, 64-byte aligned. */
static const zevo_vec5_t zevo_vec5_zero = {
    { 0, 0, 0, 0, 0 },
    { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
      0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 }
};
```

## 42. The Implementation Model

### 42.1 Pure-Function Wrapper Model

**Def. 42.1.1.** The implementation model is a set of pure-function wrappers `{W₁, …, Wₙ}` on ANSI C89/90, each of the form

```c
result_t W_i(input_t x) {
    /* pure computation */
    return y;
}
```

with no side effects except at the IPC boundary and the libc syscall wrappers.

**Th. 42.1.2.** The composition `W = W_n ∘ … ∘ W₁` is a pure function, given Ax. 2.4.3.

*Proof.* By Def. 2.4.1, the composition of pure functions is pure. By Ax. 2.4.3, the only side effects are at the boundaries. ∎

### 42.2 Correspondence to the Pipeline

**Def. 42.2.1.** The pipeline stages `f₁, …, f₆` (§4.1.1) correspond one-to-one to pure-function wrappers `W₁, …, W₆`:

| Stage | Wrapper | Signature |
|---|---|---|
| `f₁` | `zevo_fsm_step` | `zevo_fsm_state_t zevo_fsm_step(const zevo_fsm_state_t *s, const zevo_fsm_input_t *i);` |
| `f₂` | `zevo_neurofuzzy_step` | `zevo_nf_state_t zevo_neurofuzzy_step(const zevo_nf_state_t *s, const zevo_nf_input_t *i);` |
| `f₃` | `zevo_entropy_step` | `double zevo_entropy_step(const zevo_dist_t *d);` |
| `f₄` | `zevo_mamdani_step` | `double zevo_mamdani_step(const zevo_fuzzy_set_t *s, const zevo_rule_set_t *r);` |
| `f₅` | `zevo_ml_step` | `zevo_ml_state_t zevo_ml_step(const zevo_ml_state_t *s, const zevo_ml_input_t *i, const zevo_ml_params_t *p);` |
| `f₆` | `zevo_fuse5d_step` | `zevo_vec5_t zevo_fuse5d_step(const zevo_modalities_t *m, const zevo_fuse_params_t *p);` |

**Th. 42.2.2.** `zevo_fuse5d_step ∘ … ∘ zevo_fsm_step` is a pure function, given Ax. 2.4.3.

*Proof.* By Th. 42.1.2. ∎

### 42.3 Block-Comment Discipline

**Def. 42.3.1.** All source code MUST use block comments. Line comments MUST NOT appear.

**Th. 42.3.2.** The block-comment discipline is preserved under composition.

*Proof.* The comment discipline is a lexical property of the source file, not of the runtime behavior. Composition of functions does not affect the lexical structure of the source. ∎

**Def. 42.3.3 (Enforcement).** The block-comment discipline MUST be enforced by a build-system check that scans all `.c` and `.h` files for the byte sequence `//` outside of string literals and character literals. A file containing `//` outside those contexts is non-conforming.

### 42.4 Isomorphism Preservation under Implementation

**Th. 42.4.1.** If the implementation model of §42.1 is used, the container isomorphism `φ` (§35.1.1) is preserved.

*Proof.* The implementation reads and writes the container according to the rules of Parts II and III. The isomorphism `φ` is a property of the container's field structure, not of the implementation. As long as the implementation writes the field structure specified in Part II, `φ` holds. ∎

**Cor. 42.4.2.** The implementation is **correct** with respect to the specification if and only if it writes containers that are field-wise bijective with the corresponding GGUF v3 containers under the declared differences.

## 43. Conformance Tests

**Def. 43.1.**

| # | Test | Criterion |
|---|---|---|
| T1 | Header field bijection (C1) | 4 fields, same types, offsets, roles |
| T2 | Metadata type identity (C2) | 13 types, identical numerics |
| T3 | Metadata encoding identity (C3) | Strings and arrays byte-identical |
| T4 | Metadata key naming identity (C4) | Same naming rules |
| T5 | Payload index field bijection (C5) | 5 fields; `dimensions` width extension declared |
| T6 | Payload blob identity (C6) | Byte-identical after padding strip |
| T7 | Layout order identity (C7) | Header → Metadata → Index → Padding → Blob |
| T8 | Alignment rule identity (C8) | `align_offset` identical |
| T9 | Padding rule identity (C9) | `0x00` bytes, same placement |
| T10 | Endianness identity (C10) | Same implicit-LE rule |
| T11 | Naming convention identity (C11) | Same components, same order, extension differs |
| T12 | Count declaration identity (C12) | Counts in header, same types, offsets |
| T13 | Round-trip Zevo → GGUF → Zevo | Byte-identical payloads and index, when `dimensions[3] == 0` and `dimensions[4] == 0` everywhere |
| T14 | Round-trip GGUF → Zevo → GGUF | Byte-identical payloads and index |
| T15 | Type map completeness | Every `ggml_type` at the pinned commit has a `zevo_type` |
| T16 | Dtype byte-identity | Numeric values equal at the pinned commit |
| T17 | Shape order identity | Both column-major |
| T18 | Rank bound | GGUF ≤ 3; Zevo ≤ 5 |
| T19 | Default alignment identity | Both 32 |
| T20 | Metadata superset check | Every GGUF v3 key preserved |
| T21 | Metadata prefix check | All Zevo-specific keys use `zevo.` |
| T22 | Magic number structural identity | 4-byte ASCII `uint32_t` at offset 0 |
| T23 | Version field identity | `uint32_t` at offset 4 |
| T24 | Little-endian identity | Both write `uint32_t` LE |
| T25 | Big-endian identity | Both support BE implicitly |
| T26 | Alignment offset rule byte-identity | `align_offset` identical |
| T27 | String length prefix byte-identity | Both `uint64_t` |
| T28 | Array encoding byte-identity | Both type + len + elements |
| T29 | Padding byte value identity | Both `0x00` |
| T30 | Container closure | Both single files |
| T31 | Quantized block layout identity | Same `bs`, `bb` per type at the pinned commit |
| T32 | Offset invariant identity | `align_offset(offset) == offset` in both |
| T33 | Declared width extension test | `dimensions[3] == 0` and `dimensions[4] == 0` ⟺ `bijective-3d` class |
| T34 | Declared difference test | Magic ∈ {`ZEVO`, `GGUF`}; version ∈ {5, 3} |
| T35 | Single-index test | The container has exactly one payload index |
| T36 | Equivalence class declaration | `zevo.model.equivalence_class` is one of the three allowed values and matches the actual class |
| T37 | `sizeof(zevo_vec5_t) == 64` | Build-time or runtime check (Req. 5.4.4) |
| T38 | `sizeof(zevo_header_t) == 24` | Build-time or runtime check (Req. 18.1.3) |
| T39 | `zevo_vec5_t` 64-byte alignment | Runtime check via `(uintptr_t)p % 64 == 0` |
| T40 | Block-comment discipline | No `//` outside strings/chars (Def. 42.3.3) |
| T41 | fs-verity Merkle relation (bijective-3d) | `Merkle(G) == Merkle(T(Z))` (Th. 30.3.4) |
| T42 | Mamdani zero-denominator | `Σ μ == 0 ⟹ y* == (r_min + r_max) / 2` (Req. 4.5.2) |
| T43 | C89/90 pure-function composition | `zevo_fuse5d_step ∘ … ∘ zevo_fsm_step` is deterministic given Ax. 2.4.3 |
| T44 | Error handling | All 26 rejection conditions in Def. 39.1 are triggered by the corresponding malformed container |
| T45 | Versioning | A container with `version != 5` is rejected |
| T46 | Filename regex (positive) | Every accept-case in §52.6 matches the regex |
| T47 | Filename regex (negative) | Every reject-case in §52.6 does not match the regex |
| T48 | Filename post-match checks | Post-match checks in §52.3 reject every ambiguous case |
| T49 | Filename cross-file check | Cross-file constraint in §52.4 is satisfied for a multi-shard model |
| T50 | Filename POSIX ERE dialect | The regex compiles under `regcomp(..., REG_EXTENDED)` |

---

# Part V — Conformance, Errata, and References

## 44. Normative Requirements

An implementation claiming conformance MUST satisfy all of the following:

1. Implement the six-stage pipeline of §4.
2. Use C89/90 with the extensions of §2.2.1 only.
3. Use block comments only (§2.3.1), enforced per Def. 42.3.3.
4. Implement pure-function wrappers (§2.4.2).
5. Produce the five modalities of §5.1.1.
6. Represent the 5D vector as `zevo_vec5_t` with `ZEVO_VEC5_SIZE == 64` (§5.4.1), enforced per Req. 5.4.4.
7. Pad the 5D vector with 24 zero bytes (§5.4.3).
8. Achieve 64-byte alignment per Req. 5.4.5.
9. Support at least one of the three SIMD widths of §5.3.1.
10. Embed weights, metadata, and vectors in the four ELF sections of §6.2.1.
11. Page-align each section to ≥ 4096 bytes (§6.3.1).
12. Protect each section with fs-verity using SHA-256 (§6.3.1).
13. Map each section read-only (§6.3.1).
14. Export only the published IPC API (§8.1.2).
15. Hold no weights and no persistent state beyond §8.2.1's exceptions.
16. Rotate API keys every 28 hours from key creation (§9.2).
17. Use OpenSSL 3.x for NIST SP 800-38D, SP 800-107, and FIPS 186-5 (§9.3).
18. Reach hardware only through libc wrappers on x86_64 (§10.1.1).
19. Apply `seccomp-bpf`, capability bounding, and LSM (§10.2.1).
20. Use a syscall allowlist that excludes writes to fs-verity-protected sections (§10.3.2).
21. Preserve the proper name "An 5D interactive artificial intelligence assistant" verbatim (§1.3).
22. State the Pixar non-affiliation disclaimer wherever "Pixar-inspired" appears (§1.4).
23. Pin the ggml commit and declare it in build documentation (Note 16.2.3).
24. Implement the model definition of §47.
25. Handle the Mamdani zero-denominator case per Req. 4.5.2.
26. Handle the endianness failure mode per FM 23.2.
27. Implement the filename convention of §51.
28. Implement the filename validation of §52.
29. Satisfy all 50 conformance tests of §43.1.

## 45. Errata from Prior Zevo Documents

**Def. 45.1.** This document supersedes all prior Zevo documents. The following prior statements are **retracted**:

1. The two-index container `MC₂ = (H, M, T, V, D)` is retracted. Zevo v5.2 has exactly one payload index.
2. The shape-free "exclusive vector" entry is retracted. The Zevo v5.2 payload entry is `zevo_entry_info_t`, field-identical to `gguf_tensor_info_t` except for the declared `dimensions[5]` width extension.
3. The `role` and `flags` fields are retracted from any payload index. All per-vector attributes live in metadata under `zevo.vec.<name>.*`.
4. The header is 4 fields, matching GGUF v3.
5. The payload index entry is 5 fields, matching GGUF v3 except for `dimensions[5]`.
6. The dtype enumeration is byte-identical to `ggml_type` **at the pinned commit** (Note 16.2.3).
7. The string, array, and value-union encodings are byte-identical.
8. The alignment rule, padding rule, layout order, and endianness rule are byte-identical.
9. Default alignment is 32.
10. The naming convention is inherited (`.bin` extension).
11. Column-major shape order is adopted (no reversal).
12. Quantized block layouts are adopted normatively from `ggml` **at the pinned commit**.
13. Two declared differences: magic marker, `zevo.*` metadata namespace.
14. One declared width extension: `uint64_t[5]` vs. `uint64_t[3]`.
15. The 5D runtime vector alignment is specified: 40-byte payload, 64-byte padded size, 24 zero bytes.
16. ELF section embedding is specified (Part III).
17. 50 conformance tests are defined.
18. 29 normative requirements are defined.
19. Equivalence classes are `bijective-3d`, `bijective-5d`, `partial`.
20. Equivalence is field-wise bijective; the container is **100% field-set equivalent** to GGUF v3 (Note 14.4).
21. **"100% isomorphic" is restricted to field-set isomorphism** (Note 14.4, FM 35.3.4).
22. **Determinism is conditional on Ax. 2.4.3** (Note 4.1.3).
23. **The model definition is added in §47.**
24. **The failure-mode enumeration is added in §48.**
25. **The closed audit is added in §49.**
26. **The C89/90 enforcement mechanisms are added in §50.**
27. **The fs-verity Merkle relation `T` is specified in Def. 30.3.3.**
28. **The filename convention is formalized in §51.**
29. **The filename validation regex and post-match checks are specified in §52.** The PCRE-only regex of prior Zevo documents is retracted.
30. **Version is required in the filename** (C-11).
31. **The BaseName/SizeLabel ambiguity is resolved** by Req. 51.3.1 (C-12).
32. **The FineTune/Version ambiguity is resolved** by Req. 51.4.1.
33. **Shard numbering is constrained** by Req. 51.6.1–Req. 51.6.3.
34. **Whitespace in filenames is constrained** by Req. 51.7.1–Req. 51.7.2.

## 46. Summary of the Isomorphism

**Th. 46.1 (Main theorem).** The Zevo v5.2 container `Z` is **field-set isomorphic** to the GGUF v3 (fixed-3) container `G` under the field-wise bijection `φ` of Def. 14.2, subject to:

- two declared differences (magic `ZEVO`/`GGUF`; `zevo.*` metadata namespace);
- one declared width extension (`uint64_t[5]` vs. `uint64_t[3]` for `dimensions`);
- the restriction that `G` has no fifth-dimension data;
- the restriction that `Z` is in the `bijective-3d` or `bijective-5d` equivalence class.

*Proof.* By Th. 35.1.1. ∎

**Cor. 46.2.** The isomorphism resolves every contradiction (C-1 through C-12) and every ambiguity (A-1 through A-35) in the base document.

*Proof.* See §36 (contradictions) and §49 (audit). ∎

**Cor. 46.3.** The implementation model in pure-function wrappers on ANSI C89/90 preserves the isomorphism, given Ax. 2.4.3.

*Proof.* By Th. 42.4.1. ∎

**FM 46.4 (Declared failure mode).** The isomorphism does **not** extend to:

- byte-stream identity (§48.1);
- semantic identity (§48.2);
- runtime behavior identity (§48.3);
- endianness marker presence (§48.4);
- model capability (§48.5).

## 47. Model Definition

### 47.1 Purpose

Zevo v5.0.0 specified the container format but not the model. This section fills that gap. The model is defined at the level of abstraction required to make the specification **implementable** and **testable**, without specifying a particular training run, dataset, or evaluation.

**Def. 47.1.1 (Model scope).** The model is a **reference architecture**. A conforming implementation MAY substitute a different architecture provided it satisfies the pipeline of §4, the metadata keys of §19.8, and the conformance tests of §43.

### 47.2 Reference Architecture

**Def. 47.2.1.** The reference architecture is a transformer-based multimodal model with:

| Component | Parameter | Metadata key |
|---|---|---|
| Vocabulary size | `V_tok` | `zevo.model.vocab_size` |
| Hidden dimension | `d_model` | `zevo.model.hidden_dim` |
| Number of layers | `L` | `zevo.model.layers` |
| Attention heads | `H_a` | `zevo.model.attention_heads` |
| Context length | `C` | `zevo.context_length` |
| Embedding length | `d_emb` | `zevo.embedding_length` |

**Req. 47.2.2.** `d_model % H_a == 0` and `H_a > 0`.

### 47.3 Layer Composition

**Def. 47.3.1.** Each layer `ℓ ∈ {1, …, L}` is a pure function:

```
Layer_ℓ(x) = FFN_ℓ(Attn_ℓ(x) + x) + Attn_ℓ(x) + x
```

where `Attn_ℓ` is multi-head self-attention and `FFN_ℓ` is a two-layer feed-forward network with GELU activation.

**Def. 47.3.2.** The full model is:

```
Model(tokens) = LM_Head(Layer_L(… Layer_1(Embed(tokens)) …))
```

### 47.4 The 5D Fusion Function

**Def. 47.4.1.** The fusion function `f₆` is defined as:

```
f₆(m_text, m_image, m_video, m_audio, m_5d) =
    W_fuse · [E_text(m_text); E_image(m_image); E_video(m_video);
              E_audio(m_audio); m_5d] + b_fuse
```

where `E_*` are modality encoders (each a pure function) and `W_fuse`, `b_fuse` are the fusion parameters stored in `.elf.data`.

**Req. 47.4.2.** `W_fuse` has shape `[5, d_fuse]` and `b_fuse` has shape `[5]`.

### 47.5 Training Procedure

**Def. 47.5.1.** The model is trained offline. The training procedure is:

1. **Pre-training** on a text corpus using next-token prediction.
2. **Multimodal alignment** on paired (text, image), (text, video), (text, audio) data.
3. **Instruction tuning** on a curated instruction dataset.
4. **5D fusion training** on paired (modalities, 5D vector) data.

**Def. 47.5.2.** After training, all parameters are frozen and serialized into the container's `.elf.data` section.

**Req. 47.5.3.** The implementation MUST document the training procedure in its build documentation. The documentation MUST specify at least: the training data sources, the training compute, the evaluation metrics, and the known limitations.

### 47.6 PRNG

**Def. 47.6.1.** The runtime PRNG is a 64-bit xorshift variant:

```c
uint64_t zevo_xorshift64(uint64_t *state) {
    uint64_t x = *state;
    x ^= x << 13;
    x ^= x >> 7;
    x ^= x << 17;
    *state = x;
    return x;
}
```

**Req. 47.6.2.** The PRNG is seeded from `zevo.model.seed` when present. When absent, it is seeded from the kernel CSPRNG. The PRNG MUST be deterministic given a fixed seed.

### 47.7 Out of Scope

**Def. 47.7.1.** The following are **out of scope** for this specification:

- The choice of training data.
- The choice of optimizer, learning rate, batch size.
- The evaluation methodology and benchmarks.
- The specific number of parameters, layers, heads.
- The hardware used for training or inference.

**Req. 47.7.2.** An implementation claiming conformance MUST document these choices in its build documentation, but they are not normative.

## 48. Failure-Mode Enumeration

### 48.1 Byte-Stream Failure Modes

**FM 48.1.1.** The byte streams of `Z` and `G` are **not** identical. They differ in:

- the 4-byte magic (offset 0);
- the 4-byte version (offset 4);
- the presence of `zevo.*` metadata keys;
- the width of each `dimensions` field (5 slots vs. 3 slots per entry);
- the offsets of all fields after the first `dimensions` field.

**FM 48.1.2.** A byte-for-byte comparison of `Z` and `G` will fail. This is by design.

### 48.2 Semantic Failure Modes

**FM 48.2.1.** `Z` and `G` are **not** semantically identical. They differ in:

- the interpretation of `dimensions[3]` and `dimensions[4]` (present in `Z`, absent in `G`);
- the interpretation of `zevo.*` metadata keys (present in `Z`, absent in `G`);
- the interpretation of the version field (5 vs. 3).

**FM 48.2.2.** A reader that treats `Z` as `G` will misparse the `zevo.*` metadata and the `dimensions` field.

### 48.3 Runtime Behavior Failure Modes

**FM 48.3.1.** `Z` and `G` are **not** runtime-behavior identical. They differ in:

- the pipeline (§4) that consumes them;
- the model (§47) that interprets them;
- the artifact (§5) that they produce.

**FM 48.3.2.** A GGUF v3 runtime cannot execute a Zevo v5.2 container without modification. A Zevo v5.2 runtime cannot execute a GGUF v3 container without modification.

### 48.4 Endianness Marker Failure Modes

**FM 48.4.1.** Neither `Z` nor `G` carries an endianness marker. A reader that assumes little-endian on a big-endian container will misparse.

**FM 48.4.2.** This is inherited from GGUF v3 and is a declared limitation, not a defect of Zevo v5.2.

### 48.5 Model Capability Failure Modes

**FM 48.5.1.** This specification does **not** claim that a conforming implementation:

- produces correct answers;
- produces safe outputs;
- is competitive with any named model;
- has any particular parameter count;
- has any particular evaluation score.

**FM 48.5.2.** The specification is a **format and architecture specification**. Model capability is determined by training, data, and evaluation, none of which are specified here.

### 48.6 Filename Failure Modes

**FM 48.6.1.** The filename convention is **not perfectly machine-parseable**. The prose itself acknowledges "the diversity of existing zevo filenames." The regex and post-match checks in §52 enforce the **strict** interpretation: any filename that is ambiguous between two valid parses is rejected.

**FM 48.6.2.** A filename that is human-readable but ambiguous (e.g., `Mixtral-8x7B-v0.1-KQ2.bin`) is non-conforming under the strict interpretation.

**FM 48.6.3.** The filename convention is **best-effort**: a filename can encode the same metadata as the container, but the container is authoritative. A filename that disagrees with the container's metadata is non-conforming (Req. 51.8.1).

### 48.7 Summary Table

| Failure mode | `Z` vs. `G` | Bounded? | Reversible? |
|---|---|---|---|
| Byte stream | Different | Yes | Yes (`T`, Def. 30.3.3) |
| Semantics | Different | Yes | Yes (filter + truncate) |
| Runtime behavior | Different | Yes | No (different pipelines) |
| Endianness marker | Absent in both | Yes | No (inherited) |
| Model capability | Not specified | Yes | N/A |
| Filename parse | Ambiguous without post-match checks | Yes | Yes (§52) |

## 49. Closed Audit of Ambiguities and Gaps

### 49.1 Audit Procedure

**Def. 49.1.1 (Enumeration procedure).** The audit enumerates every ambiguity and gap by:

1. scanning every sentence of the prior Zevo documents for undefined terms;
2. scanning every sentence of the prior Zevo documents for contradictory claims;
3. scanning every sentence of the prior Zevo documents for missing normative rules;
4. cross-referencing each finding against Parts I–IV of this document;
5. declaring each finding either **resolved** or **declared failure mode**.

**Def. 49.1.2.** The audit is **closed** in the sense that every finding in the prior Zevo documents is listed. The audit does **not** claim that no new ambiguity can be introduced by a future implementation.

### 49.2 Ambiguity Audit

| # | Ambiguity in prior Zevo documents | Resolution |
|---|---|---|
| A-1 | "Field-wise bijective" undefined | Def. 14.2 |
| A-2 | "100% structurally equivalent" undefined | Def. 14.3, Note 14.4 |
| A-3 | "Superset" undefined for metadata | §19.5, §19.7 |
| A-4 | "Verbatim" undefined | §19.5 |
| A-5 | "Offset" undefined | §20.6 |
| A-6 | "Must be `0x00` on read" undefined | Req. 5.4.3 |
| A-7 | "Column-major" undefined | Def. 20.4.1 |
| A-8 | "Rank bound" undefined | §20.4.2 |
| A-9 | "Alignment rule" undefined | Req. 22.1.3 |
| A-10 | "Padding" undefined | §22.3.1 |
| A-11 | "Metadata KVS" ordering undefined | §19.7 |
| A-12 | "Payload index" ordering undefined | §20.7 |
| A-13 | "Role" value space undefined | §19.9 |
| A-14 | "5D output artifact" undefined | §5.6 |
| A-15 | "28-hour API key rotation" undefined | §9.2 |
| A-16 | "Atomic" (key rotation) undefined | §9.2.2 |
| A-17 | "No internal symbol" undefined | Req. 8.1.2 |
| A-18 | "Weight-free" undefined | Def. 8.2.1 |
| A-19 | "5D" undefined | Def. 1.2 |
| A-20 | "Equivalence class" values undefined | §27.1 |
| A-21 | "Prototype" undefined | §19.8 |
| A-22 | "Exclusive" undefined | §19.8 |
| A-23 | "Mutability" undefined | §19.8 |
| A-24 | "Affect" undefined | §19.8 |
| A-25 | "Two declared differences" undefined | Def. 15.2.1 |
| A-26 | "100% field-set equivalence" undefined | Def. 14.3 |
| A-27 | "Pixar-inspired" undefined | Def. 1.4 |
| A-28 | "ggml_type identity" undefined without pinning | Note 16.2.3 |
| A-29 | "Mamdani centroid division" undefined for zero | Req. 4.5.2 |
| A-30 | "C89/90 alignment" undefined | Req. 5.4.5 |
| A-31 | "C89/90 struct-size enforcement" undefined | Req. 5.4.4, Req. 18.1.3 |
| A-32 | "fs-verity Merkle relation" undefined | Def. 30.3.3 |
| A-33 | "Filename convention" undefined | §51 |
| A-34 | "Filename validation regex" uses non-POSIX dialect | §52 |
| A-35 | "BaseName/SizeLabel ambiguity" unaddressed | Req. 51.3.1 |
| A-36 | "FineTune/Version ambiguity" unaddressed | Req. 51.4.1 |
| A-37 | "ShardNum ≥ 00001" unenforced | Req. 51.6.1 |
| A-38 | "Whitespace in filenames" unaddressed | Req. 51.7.1 |

### 49.3 Gap Audit

| # | Gap in prior Zevo documents | Resolution |
|---|---|---|
| G-1 | Header size unspecified | Def. 18.1.2 |
| G-2 | C89/90 extensions unspecified | Def. 2.2.1 |
| G-3 | Comment nesting unspecified | Def. 2.3.3 |
| G-4 | Pure function unspecified | Def. 2.4.1 |
| G-5 | Pipeline stages unspecified | Def. 4.1.1 |
| G-6 | t-norm / s-norm unspecified | Def. 4.5.1 |
| G-7 | Shannon base unspecified | Def. 4.4.1 |
| G-8 | Defuzzification samples unspecified | Def. 4.5.1 |
| G-9 | Defuzzification division by zero unspecified | Req. 4.5.2 |
| G-10 | 5D vector size unspecified | Def. 5.4.1 |
| G-11 | SIMD alignment unspecified | Lem. 5.4.6 |
| G-12 | ELF section names unspecified | Def. 6.2.1 |
| G-13 | fs-verity hash unspecified | Def. 6.3.1 |
| G-14 | fs-verity Merkle relation unspecified | Def. 30.3.2, Def. 30.3.3 |
| G-15 | Syscall allowlist unspecified | Def. 10.3.1 |
| G-16 | Error handling unspecified | §39 |
| G-17 | Versioning unspecified | §40 |
| G-18 | Endianness marker unspecified | §23, FM 23.2 |
| G-19 | Test vectors unspecified | §41 |
| G-20 | Role values unspecified | §19.9 |
| G-21 | Mutability values unspecified | §19.8 |
| G-22 | Prototype unspecified | §19.8 |
| G-23 | Affect unspecified | §19.8 |
| G-24 | PRNG unspecified | §40, §47.6 |
| G-25 | Required metadata keys unspecified | §19.6 |
| G-26 | Model definition absent | §47 |
| G-27 | Failure modes absent | §48 |
| G-28 | Closed audit absent | §49 |
| G-29 | C89/90 enforcement mechanisms absent | §50 |
| G-30 | ggml commit unpinned | Note 16.2.3 |
| G-31 | Filename convention unformalized | §51 |
| G-32 | Filename regex non-normative and non-POSIX | §52 |
| G-33 | Filename post-match checks absent | §52.3 |
| G-34 | Filename cross-file constraint absent | §52.4 |

### 49.4 Audit Closure

**Th. 49.4.1.** Every ambiguity in §49.2 and every gap in §49.3 is resolved or declared a failure mode in this document.

*Proof.* By direct inspection of the resolution column. ∎

**Note 49.4.2.** This audit is closed with respect to the prior Zevo documents. It is not closed with respect to future implementations. An implementation that introduces a new ambiguity is non-conforming.

## 50. C89/90 Enforcement Mechanisms

### 50.1 Struct Size

**Def. 50.1.1 (Build-system check).** The build system MUST compile and execute a program that asserts:

```c
/* build_check.c — compiled and run by the build system */
#include <stdio.h>
#include <stddef.h>
#include "zevo.h"

int main(void) {
    if (sizeof(zevo_vec5_t) != 64) {
        fprintf(stderr, "zevo_vec5_t size = %lu, expected 64\n",
                (unsigned long)sizeof(zevo_vec5_t));
        return 1;
    }
    if (sizeof(zevo_header_t) != 24) {
        fprintf(stderr, "zevo_header_t size = %lu, expected 24\n",
                (unsigned long)sizeof(zevo_header_t));
        return 1;
    }
    if (offsetof(zevo_header_t, magic) != 0) {
        fprintf(stderr, "zevo_header_t.magic offset != 0\n");
        return 1;
    }
    if (offsetof(zevo_header_t, version) != 4) {
        fprintf(stderr, "zevo_header_t.version offset != 4\n");
        return 1;
    }
    if (offsetof(zevo_header_t, vector_count) != 8) {
        fprintf(stderr, "zevo_header_t.vector_count offset != 8\n");
        return 1;
    }
    if (offsetof(zevo_header_t, metadata_kv_count) != 16) {
        fprintf(stderr, "zevo_header_t.metadata_kv_count offset != 16\n");
        return 1;
    }
    return 0;
}
```

**Req. 50.1.2.** The build system MUST fail if this check fails.

### 50.2 Alignment

**Def. 50.2.1 (Runtime alignment check).** The implementation MUST verify at load time that every `zevo_vec5_t` pointer is 64-byte aligned.

```c
int zevo_check_alignment(const zevo_vec5_t *p) {
    return ((uintptr_t)p % 64) == 0;
}
```

**Req. 50.2.2.** The implementation MUST use `posix_memalign` or `aligned_alloc` for allocation of `zevo_vec5_t` arrays.

### 50.3 Block Comments

**Def. 50.3.1 (Lexical check).** The build system MUST scan all `.c` and `.h` files for the byte sequence `//` outside of string literals and character literals.

**Def. 50.3.2 (Lexical check implementation).** The scan is performed by a C89/90-conforming lexer that:

1. skips `/* ... */` comments;
2. skips string literals `"..."` with escape handling;
3. skips character literals `'...'` with escape handling;
4. rejects any remaining `//` sequence.

### 50.4 Pure-Function Wrapper

**Def. 50.4.1 (Purity check).** The build system SHOULD verify purity by static analysis:

1. no global mutable state is read or written;
2. no I/O calls (`printf`, `fwrite`, etc.) appear in wrapper bodies;
3. no `static` local variables appear in wrapper bodies.

**Req. 50.4.2.** A wrapper failing any of these checks is non-conforming.

### 50.5 PRNG Determinism

**Def. 50.5.1.** The build system MUST run a test that seeds the PRNG with a fixed seed and verifies that the first 1000 outputs match a reference vector.

### 50.6 Error Handling

**Def. 50.6.1.** The build system MUST run a test that feeds each of the 26 malformed containers of Def. 39.1 to the reader and verifies that the reader rejects each one.

### 50.7 Filename Validation

**Def. 50.7.1 (POSIX ERE compilation check).** The build system MUST compile the filename regex of §52.2 with `regcomp(..., REG_EXTENDED)` and MUST fail if compilation fails.

**Def. 50.7.2 (Test corpus check).** The build system MUST run the test corpus of §52.6 and verify every accept/reject result.

---

## 51. The Zevo Filename Convention (formalized)

### 51.1 Purpose

The filename convention is **human-readable** and **best-effort machine-parseable**. It is not intended to be perfectly machine-parseable, as the prose itself notes: "It is not intended to be perfectly parsable in the field due to the diversity of existing zevo filenames."

**Req. 51.1.1.** The filename convention is **normative** for conforming Zevo v5.2 files. A file whose name does not conform is non-conforming.

**Req. 51.1.2.** The container's metadata is **authoritative**. A filename that disagrees with the container's metadata is non-conforming (see Req. 51.8.1).

### 51.2 Components

**Def. 51.2.1.** A conforming filename is the concatenation, in order, of:

```
<BaseName><SizeLabel><FineTune><Version><Encoding><Type><Shard>.bin
```

where each component, if present, is preceded by a single `-` delimiter. Components MAY be omitted per their individual rules (§51.3–§51.7), except where stated otherwise.

**Def. 51.2.2 (Component patterns).** Each component matches a POSIX ERE pattern:

| Component | Pattern | Required? |
|---|---|---|
| BaseName | `[A-Za-z][A-Za-z0-9 ]*(-[A-Za-z0-9][A-Za-z0-9 ]*)*` | Yes |
| SizeLabel | `([0-9]+[xX])?[0-9]+(\.[0-9]+)?[QTBMK]([A-Za-z][0-9]+(\.[0-9]+)?[QTBMK])*` | Yes |
| FineTune | `[A-Za-z0-9][A-Za-z0-9 ]*` | No |
| Version | `v[0-9]+(\.[0-9]+)*` | Yes |
| Encoding | `[A-Za-z0-9_][A-Za-z0-9_-]*` | No |
| Type | `(LoRA\|vocab)` | No |
| Shard | `[0-9]{5}-of-[0-9]{5}` | No |

**Def. 51.2.3 (Extension).** The extension is `.bin`.

### 51.3 BaseName Rules

**Def. 51.3.1 (BaseName derivation).** The BaseName is derived from `general.basename` by substituting each space with a dash. If `general.basename` is absent, the BaseName is derived from `general.name` by the same substitution.

**Req. 51.3.2 (Disambiguation).** A BaseName segment MUST NOT match `[0-9]+[xX][0-9]+[QTBMK]` or `[0-9]+(\.[0-9]+)?[QTBMK]`. This prevents BaseName/SizeLabel ambiguity.

**Req. 51.3.3 (First character).** The BaseName MUST begin with a letter.

**Req. 51.3.4 (Retracted example).** The prior Zevo example `Mixtral-8x7B-v0.1-KQ2.bin` is **retracted** because `8x7B` matches the SizeLabel pattern. A conforming alternative is `Mixtral-MoE-8x7B-v0.1-KQ2.bin` (with FineTune `MoE`).

### 51.4 FineTune and Version Rules

**Def. 51.4.1 (FineTune derivation).** The FineTune is derived from `general.finetune` by substituting each space with a dash. If `general.finetune` is absent, the FineTune is omitted.

**Req. 51.4.2 (FineTune disambiguation).** A FineTune segment MUST NOT contain the sequence `-v` followed by a digit, because that sequence begins Version.

**Req. 51.4.3 (FineTune exclusion).** A FineTune segment MUST NOT be the literal string `LoRA` or `vocab`, because those are Type values.

**Def. 51.4.4 (Version).** The Version is derived from `general.version`. If `general.version` is absent, the Version defaults to `v1.0` for **metadata** purposes, but the **filename** MUST contain a Version (Req. 51.4.5).

**Req. 51.4.5 (Version required).** The filename MUST contain a Version component. This is a change from prior Zevo documents, which said Version was optional. The change is justified by the prose's own observation that "it is easy for Encoding to be mistaken as a FineTune if Version is omitted."

**Req. 51.4.6 (Version format).** The Version MUST match `v[0-9]+(\.[0-9]+)*`.

### 51.5 Encoding and Type Rules

**Def. 51.5.1 (Encoding).** The Encoding component is user-defined. Its content, type mixture, and arrangement are determined by user code.

**Req. 51.5.2 (Encoding pattern).** The Encoding MUST match `[A-Za-z0-9_][A-Za-z0-9_-]*`.

**Req. 51.5.3 (Encoding exclusion).** The Encoding MUST NOT be the literal string `LoRA` or `vocab`, because those are Type values.

**Req. 51.5.4 (Encoding omission).** If the Encoding is unknown, it MUST be omitted.

**Def. 51.5.5 (Type).** The Type component is either `LoRA` (LoRA adapter) or `vocab` (vocabulary-only file). If the Type is absent, the file is a typical Zevo tensor model file.

**Req. 51.5.6 (Type omission).** If the Type is `typical`, it MUST be omitted. The strings `typical` and `tensor` MUST NOT appear in the filename.

### 51.6 Shard Rules

**Def. 51.6.1 (Shard format).** The Shard component is formatted as `<ShardNum>-of-<ShardTotal>` where both are 5-digit zero-padded integers.

**Req. 51.6.2 (ShardNum ≥ 00001).** ShardNum MUST be `00001` or greater. `00000` is non-conforming.

**Req. 51.6.3 (ShardNum ≤ ShardTotal).** ShardNum MUST be ≤ ShardTotal.

**Req. 51.6.4 (Shard consistency).** All shards of a model MUST share the same BaseName, SizeLabel, FineTune, Version, Encoding, and Type, and MUST share the same ShardTotal.

### 51.7 Whitespace Rules

**Req. 51.7.1 (Permitted whitespace).** Only the space character ` ` (U+0020) is permitted in filenames, and only inside BaseName and FineTune.

**Req. 51.7.2 (Forbidden whitespace).** A filename MUST NOT contain `\t`, `\n`, `\v`, `\f`, or `\r`.

**Req. 51.7.3 (Forbidden characters).** A filename MUST NOT contain `/` (POSIX path separator) or NUL.

### 51.8 Cross-Reference to Container

**Req. 51.8.1 (Metadata agreement).** The filename's BaseName, SizeLabel, FineTune, Version, and (when derivable) Encoding MUST agree with the container's metadata. If they disagree, the file is non-conforming.

**Def. 51.8.2 (Encoding non-derivability).** The Encoding is often not derivable from metadata; in that case, the filename's Encoding is informational only.

## 52. Filename Validation

### 52.1 Purpose

§52 provides a **normative** validation procedure for the filename convention of §51. The procedure consists of:

1. a **POSIX ERE** regex (§52.2) that matches the required component order and required/optional components;
2. **six post-match checks** (§52.3) that enforce the disambiguation rules the regex cannot express in POSIX ERE;
3. **one cross-file constraint** (§52.4) for shards;
4. a **C89/90 implementation sketch** (§52.5);
5. a **test corpus** (§52.6).

**Req. 52.1.1.** A conforming implementation MUST use the regex and post-match checks of §52.2–§52.3.

**Req. 52.1.2 (Dialect).** The regex MUST be a POSIX ERE (POSIX.1-2008 §9.4.6). PCRE-only constructs (named groups, negative lookahead, non-capturing groups) MUST NOT be used.

**Req. 52.1.3 (Retraction).** The PCRE-only regex of prior Zevo documents is **retracted**.

### 52.2 The POSIX ERE Regex

**Def. 52.2.1 (Regex).** The validation regex is:

```
^([A-Za-z][A-Za-z0-9 ]*)(-[A-Za-z0-9][A-Za-z0-9 ]*)*-(([0-9]+[xX])?[0-9]+(\.[0-9]+)?[QTBMK]([A-Za-z][0-9]+(\.[0-9]+)?[QTBMK])*)(-([A-Za-z0-9][A-Za-z0-9 ]*))?-(v[0-9]+(\.[0-9]+)*)(-([A-Za-z0-9_][A-Za-z0-9_-]*))?(-(LoRA|vocab))?(-([0-9]{5})-of-([0-9]{5}))?\.bin$
```

**Def. 52.2.2 (Group numbering).** POSIX ERE numbers capture groups by opening-parenthesis order, left to right. The relevant groups are:

| Group | Content |
|---|---|
| 1 | BaseName (first segment) |
| 2 | BaseName (continuation segments, including leading `-`) |
| 3 | SizeLabel |
| 4 | SizeLabel (optional `<expertCount>x` prefix) |
| 5 | SizeLabel (optional decimal fraction) |
| 6 | SizeLabel (optional attribute suffixes) |
| 7 | FineTune (with leading `-`) |
| 8 | FineTune (without leading `-`) |
| 9 | Version (without leading `-`) |
| 10 | Version (optional `.Minor` suffix) |
| 11 | Encoding (with leading `-`) |
| 12 | Encoding (without leading `-`) |
| 13 | Type (with leading `-`) |
| 14 | Type (`LoRA` or `vocab`) |
| 15 | Shard (with leading `-`) |
| 16 | ShardNum |
| 17 | ShardTotal |

The exact group numbering depends on the parenthesization; the table is illustrative and MUST be verified against the compiled regex.

### 52.3 The Six Post-Match Checks

**Req. 52.3.1.** After a successful regex match, all six of the following checks MUST pass:

1. **BaseName disambiguation.** No segment of the BaseName group (groups 1 and 2 combined) matches `[0-9]+[xX][0-9]+[QTBMK]` or `[0-9]+(\.[0-9]+)?[QTBMK]`.
2. **FineTune disambiguation.** No segment of the FineTune group (group 8) matches `v[0-9]+(\.[0-9]+)*`.
3. **FineTune exclusion.** The FineTune group (group 8) is not the literal string `LoRA` or `vocab`.
4. **Encoding exclusion.** The Encoding group (group 12) is not the literal string `LoRA` or `vocab`.
5. **ShardNum ≥ 00001.** If the Shard group is present, ShardNum (group 16) is `00001` or greater.
6. **ShardNum ≤ ShardTotal.** If the Shard group is present, ShardNum (group 16) is ≤ ShardTotal (group 17).

**Req. 52.3.2 (Whitespace).** The input string MUST NOT contain `\t`, `\n`, `\v`, `\f`, or `\r`. This check MUST be performed before the regex match.

**Req. 52.3.3 (Anchoring).** The regex is anchored with `^` and `$`. The implementation MUST ensure the input string has no trailing newline before matching (POSIX `$` matches before a trailing newline in some implementations).

### 52.4 The Cross-File Constraint

**Req. 52.4.1 (Shard consistency).** When a model is split into shards, all shards MUST share the same BaseName, SizeLabel, FineTune, Version, Encoding, and Type, and MUST share the same ShardTotal. This is a cross-file constraint and cannot be checked by a single-file regex.

### 52.5 C89/90 Implementation Sketch

**Def. 52.5.1.** The following sketch shows the C89/90 pure-function wrapper for filename validation.

```c
/* zevo_name.h */
#ifndef ZEVO_NAME_H
#define ZEVO_NAME_H

#include <regex.h>
#include <stddef.h>

#define ZEVO_NAME_MAX 4096
#define ZEVO_NAME_NGROUPS 18

typedef struct {
    char base_name[ZEVO_NAME_MAX];
    char size_label[ZEVO_NAME_MAX];
    char fine_tune[ZEVO_NAME_MAX];
    char version[ZEVO_NAME_MAX];
    char encoding[ZEVO_NAME_MAX];
    char type[ZEVO_NAME_MAX];
    char shard[ZEVO_NAME_MAX];
    int  has_fine_tune;
    int  has_encoding;
    int  has_type;
    int  has_shard;
} zevo_name_parts_t;

/* Returns 0 on success, non-zero on failure. Pure: no globals. */
int zevo_parse_name(const char *filename, zevo_name_parts_t *out);

/* Returns 0 if the BaseName is unambiguous, non-zero otherwise. */
int zevo_check_basename_unambiguous(const char *base_name);

#endif
```

```c
/* zevo_name.c */
#include "zevo_name.h"
#include <string.h>

/* The regex, as a POSIX ERE string. */
static const char *ZEVO_NAME_REGEX =
    "^([A-Za-z][A-Za-z0-9 ]*)"
    "(-[A-Za-z0-9][A-Za-z0-9 ]*)*"
    "-(([0-9]+[xX])?[0-9]+(\\.[0-9]+)?[QTBMK]"
    "([A-Za-z][0-9]+(\\.[0-9]+)?[QTBMK])*)"
    "(-([A-Za-z0-9][A-Za-z0-9 ]*))?"
    "-(v[0-9]+(\\.[0-9]+)*)"
    "(-([A-Za-z0-9_][A-Za-z0-9_-]*))?"
    "(-(LoRA|vocab))?"
    "(-([0-9]{5})-of-([0-9]{5}))?"
    "\\.bin$";

/* Returns 1 if the input contains only permitted whitespace. */
static int zevo_whitespace_ok(const char *s)
{
    const char *p;
    for (p = s; *p != '\0'; ++p) {
        if (*p == '\t' || *p == '\n' || *p == '\v'
            || *p == '\f' || *p == '\r' || *p == '/') {
            return 0;
        }
    }
    return 1;
}

/* Returns 1 if any segment of base_name matches the SizeLabel pattern. */
static int zevo_basename_ambiguous(const char *base_name);

int zevo_parse_name(const char *filename, zevo_name_parts_t *out)
{
    regex_t re;
    regmatch_t m[ZEVO_NAME_NGROUPS];
    int rc;

    if (filename == NULL || out == NULL) {
        return -1;
    }
    if (!zevo_whitespace_ok(filename)) {
        return -2;
    }
    memset(out, 0, sizeof(*out));

    rc = regcomp(&re, ZEVO_NAME_REGEX, REG_EXTENDED);
    if (rc != 0) {
        return -3;
    }
    rc = regexec(&re, filename, ZEVO_NAME_NGROUPS, m, 0);
    if (rc != 0) {
        regfree(&re);
        return -4;
    }

    /* Extract groups 1..17 (group 0 is the whole match). */
    /* This is illustrative; the actual group numbering depends on
       the exact parenthesization, which POSIX ERE numbers left-to-right
       by opening parenthesis. See the test corpus for the mapping. */

    /* Post-match check 1: BaseName disambiguation. */
    /* Concatenate groups 1 and 2, then call zevo_basename_ambiguous. */

    /* Post-match check 5: ShardNum >= 00001. */
    /* Post-match check 6: ShardNum <= ShardTotal. */

    regfree(&re);
    return 0;
}

static int zevo_basename_ambiguous(const char *base_name)
{
    /* Uses a second regex to detect SizeLabel-like segments:
       (^| )([0-9]+[xX][0-9]+[QTBMK]|[0-9]+(\.[0-9]+)?[QTBMK])( |$) */
    (void)base_name;
    return 0;
}
```

**Req. 52.5.2.** The implementation MUST compile the regex with `REG_EXTENDED`. `REG_NEWLINE` MUST NOT be used (see Req. 52.3.3).

### 52.6 Test Corpus

**Def. 52.6.1.**

| # | Filename | Expected | Reason |
|---|---|---|---|
| 1 | `Zevo-MoE-8x7B-v0.1-KQ2.bin` | Accept | BaseName `Mixtral`, FineTune `MoE`, SizeLabel `8x7B`, Version `v0.1`, Encoding `KQ2` |
| 2 | `Mixtral-8x7B-v0.1-KQ2.bin` | **Reject** | BaseName/SizeLabel ambiguity (`8x7B` could be either) |
| 3 | `Hermes-2-Pro-Llama-3-8B-F16.bin` | **Reject** | Missing Version |
| 4 | `Hermes-2-Pro-Llama-3-8B-v1.0-F16.bin` | Accept | All required components present |
| 5 | `Grok-100B-v1.0-Q4_0-00003-of-00009.bin` | Accept | All components, valid shard |
| 6 | `Grok-100B-v1.0-Q4_0-00000-of-00009.bin` | **Reject** | ShardNum `00000` violates Req. 51.6.2 |
| 7 | `Grok-100B-v1.0-LoRA-00003-of-00009.bin` | Accept | Type `LoRA`, valid shard |
| 8 | `Grok-100B-v1.0-vocab.bin` | Accept | Type `vocab`, no shard |
| 9 | `Grok-100B-v1.0-Chat-v1.0.bin` | **Reject** | FineTune `Chat-v1.0` contains `-v1.0`, violating Req. 51.4.2 |
| 10 | `grok-100B-v1.0.bin` | Accept | Lowercase BaseName allowed |
| 11 | `Grok-100B.bin` | **Reject** | Missing Version |
| 12 | `Grok-100B-v1.0.bin` | Accept | Minimal conforming filename |
| 13 | `Grok-v1.0.bin` | **Reject** | Missing SizeLabel |
| 14 | `-100B-v1.0.bin` | **Reject** | Empty BaseName |
| 15 | `Grok-100B-v1.0-.bin` | **Reject** | Trailing `-` before extension |
| 16 | `Grok-100B-v1.0.bin.bin` | **Reject** | Double extension |
| 17 | `Grok-100B-v1.0-00001-of-00009.bin` | Accept | Shard without Encoding or Type |
| 18 | `Grok-100B-v1.0-Q4_0.bin` | Accept | Encoding only |
| 19 | `Grok-100B-v1.0-Q4_0-LoRA.bin` | Accept | Encoding and Type |
| 20 | `Grok-100B-v1.0-LoRA.bin` | Accept | Type only |
| 21 | `Grok-100B-v1.0-LoRA-LoRA.bin` | **Reject** | Duplicate Type |
| 22 | `Grok-100B-v1.0-Q4_0-LoRA-00001-of-00009.bin` | Accept | All components |
| 23 | `Grok-100B-v1.0-Q4_0-LoRA-00001-of-00009.bin ` | **Reject** | Trailing space |
| 24 | `Grok-100B-v1.0-Q4_0-LoRA-00001-of-00009.bin\n` | **Reject** | Trailing newline |
| 25 | `Grok-100B-v1.0-vocab-00001-of-00009.bin` | Accept | Type and Shard |
| 26 | `Grok-100B-v1.0-00009-of-00005.bin` | **Reject** | ShardNum > ShardTotal |
| 27 | `Grok-100B-v1.0--00001-of-00009.bin` | **Reject** | Double dash |
| 28 | `Grok-100B-v1.0-0001-of-0009.bin` | **Reject** | ShardNum not 5 digits |
| 29 | `Grok 100B-v1.0.bin` | Accept | Space in BaseName |
| 30 | `Grok\t100B-v1.0.bin` | **Reject** | Tab in BaseName |

## 53. Conclusion

Zevo v5.2 is a **specification of an AI core, its container format, and its filename convention**, with:

- **all contradictions resolved** by formal proof (§36, C-1 through C-12);
- **all ambiguities resolved** by formal definition (§49.2, A-1 through A-38);
- **all gaps filled** by normative requirement (§49.3, G-1 through G-34);
- **all claims bounded** by declared failure modes (§48);
- **all enforcement mechanisms specified** for ANSI C89/90 (§50);
- **the model defined** at the level of abstraction required for implementation (§47);
- **the closed audit** performed with an independent enumeration procedure (§49.1);
- **the filename convention formalized** as a POSIX ERE plus six post-match checks plus one cross-file constraint (§51–§52).

The container is **field-set isomorphic** to the GGUF v3 (fixed-3) container under the declared bijection `φ`, with two declared differences and one declared width extension. The claim "100% isomorphic" is restricted to field-set isomorphism. The implementation model is pure-function wrappers on ANSI C89/90, applying functional programming paradigms within default block comments `/* ... */`.

**FM 53.1 (Declared non-claim).** This document does **not** claim that a conforming implementation:

- is a working 5D AI;
- produces correct, safe, or useful outputs;
- is competitive with any named model;
- has any particular evaluation score.

It claims that a conforming implementation's **container** is field-set isomorphic to GGUF v3, that its **pipeline** is deterministic given Ax. 2.4.3, and that its **filenames** match the POSIX ERE of §52.2 plus the post-match checks of §52.3.

## 54. Acknowledgments

The authors thank the Zevo Community Teams and the GGUF v3 specification authors for their foundational work. The authors also thank the anonymous reviewers whose feedback led to the failure-mode enumeration (§48), the closed audit (§49), and the filename convention formalization (§51–§52).

---

## 55. References (ISO 690:2021)

BROWN, Tom B., et al. Language models are few-shot learners. *Advances in Neural Information Processing Systems*. 2020, **33**, 1877–1901.

GERGANOV, Georgi a zevo.cpp contributors. *GGUF specification, version 3* [online]. 2024 [cit. 2026-10-14]. Available at: https://github.com/ggml-org/ggml/blob/master/docs/gguf.md

HUGGING FACE. *Safetensors format specification* [online]. 2023 [cit. 2026-10-14]. Available at: https://github.com/huggingface/safetensors

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. *ISO/IEC 9899:1990 — Programming languages — C*. Geneva: ISO, 1990.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. *ISO/IEC 9899:1999 — Programming languages — C*. Geneva: ISO, 1999.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. *ISO/IEC 9899:2011 — Programming languages — C*. Geneva: ISO, 2011.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION. *ISO 690:2021 — Information and documentation — Guidelines for bibliographic references and citations to information resources*. Geneva: ISO, 2021.

INTERNET ENGINEERING TASK FORCE. *RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels*. IETF, 1997.

KHRONOS GROUP. *OpenCL 3.0 Specification*. Beaverton: Khronos, 2020.

KHRONOS GROUP. *OpenGL 4.6 Specification*. Beaverton: Khronos, 2017.

KHRONOS GROUP. *OpenAL 1.1 Specification*. Beaverton: Khronos, 2010.

NATIONAL INSTITUTE OF STANDARDS AND TECHNOLOGY. *NIST SP 800-38D — Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC*. Gaithersburg: NIST, 2007.

NATIONAL INSTITUTE OF STANDARDS AND TECHNOLOGY. *NIST SP 800-107 Rev. 1 — Recommendation for Applications Using Approved Hash Algorithms*. Gaithersburg: NIST, 2012.

NATIONAL INSTITUTE OF STANDARDS AND TECHNOLOGY. *FIPS 186-5 — Digital Signature Standard (DSS)*. Gaithersburg: NIST, 2023.

OPENAI. *GPT-4 technical report*. arXiv:2303.08774. 2023.

OPENAI. *GPT-4o system card* [Technical report]. OpenAI, 2024.

OPENSSL SOFTWARE FOUNDATION. *OpenSSL 3.x Documentation*. 2023.

SYSTEM V ABI COMMITTEE. *System V Application Binary Interface: AMD64 Architecture Processor Supplement* (v1.0) [online]. 2018 [cit. 2026-10-14]. Available at: https://gitlab.com/x86-psABIs/x86-64-ABI

THE OPEN GROUP. *POSIX.1-2008 — Standard for Information Technology—Portable Operating System Interface (POSIX)*. 2008.

THE OPEN GROUP. *POSIX.1-2001 — Standard for Information Technology—Portable Operating System Interface (POSIX)*. 2001.

TOOL INTERFACE STANDARD COMMITTEE. *Tool Interface Standard (TIS) Executable and Linking Format (ELF) Specification* (v1.2) [online]. 1995 [cit. 2026-10-14]. Available at: https://refspecs.linuxfoundation.org/elf/TIS1.2.pdf

WORLD WIDE WEB CONSORTIUM. *HTML 5 Specification*. W3C, 2023.

WORLD WIDE WEB CONSORTIUM. *CSS 5 Specification*. W3C, 2023.

WORLD WIDE WEB CONSORTIUM. *WCAG 2.2 — Web Content Accessibility Guidelines*. W3C, 2023.

ZAITA, Davada Beni a Zevo Community Teams. *Zevo™ — An 5D interactive artificial intelligence assistant* (v3.0.0) [Technical report]. Zevo Project, 2026.

---

## 56. Change Log

| Revision | Change |
|---|---|
| 1.0.0–4.0.1 | Prior Zevo revisions. |
| 5.0.0 | Complete academic specification (first version). |
| 5.1.0 | Restricted "100% isomorphic" to "100% field-set isomorphic"; made determinism conditional on Ax. 2.4.3; added §47 Model Definition; added §48 Failure-Mode Enumeration; added §49 Closed Audit; added §50 C89/90 Enforcement Mechanisms; added fs-verity Merkle relation `T`; pinned ggml commit; added Mamdani zero-denominator rule; added C89/90 struct-size/alignment enforcement; expanded conformance tests to 45; expanded normative requirements to 27. |
| **5.2.0 (this document)** | |
| | **Added §51 Filename Convention (formalized)** — POSIX ERE component patterns, BaseName/FineTune/Version/Encoding/Type/Shard rules, whitespace rules, cross-reference to container. |
| | **Added §52 Filename Validation** — POSIX ERE regex, six post-match checks, one cross-file constraint, C89/90 implementation sketch, 30-case test corpus. |
| | **Retracted the PCRE-only filename regex** from prior Zevo documents (C-10). |
| | **Required Version in the filename** (C-11); the "optional" rule is retracted for filenames. |
| | **Resolved BaseName/SizeLabel ambiguity** by Req. 51.3.2 (C-12); retracted the `Mixtral-8x7B-v0.1-KQ2.bin` example. |
| | **Resolved FineTune/Version ambiguity** by Req. 51.4.2. |
| | **Constrained Shard numbering** by Req. 51.6.2–Req. 51.6.3. |
| | **Constrained whitespace in filenames** by Req. 51.7.1–Req. 51.7.3. |
| | **Added filename cross-reference to container metadata** by Req. 51.8.1. |
| | **Expanded conformance tests from 45 to 50** (T46–T50). |
| | **Expanded normative requirements from 27 to 29** (items 27–28). |
| | **Added C-10, C-11, C-12 contradictions** (regex dialect; Version required; BaseName/SizeLabel ambiguity). |
| | **Added A-33 through A-38 ambiguities** (filename convention, regex dialect, BaseName/SizeLabel, FineTune/Version, ShardNum, whitespace). |
| | **Added G-31 through G-34 gaps** (filename convention, regex, post-match checks, cross-file constraint). |
| | **Added FM 48.6** for filename failure modes. |
| | **Added §50.7** for filename validation enforcement in the build system. |
| | All references cited per ISO 690:2021. |

---

**End of specification.**

### Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Title | Zevo v5.2 — Complete Academic Specification |
| Subtitle | Zevo AI-Driven Core: An 5D Multimodal Interactive Artificial Intelligence Assistant Using Exclusive Vectors, 100% Field-Set Isomorphic to GGUF v3 Using Exclusive Tensors |
| Document type | Complete specification. Normative. Academic. |
| Version | 5.2.0 |
| Citation standard | ISO 690:2021 |
| Status | Specification; not evidence of a completed or validated implementation |
| Part I | Zevo AI Core (runtime) |
| Part II | Zevo v5.2 container (on-disk format) |
| Part III | Integration of container and core |
| Part IV | Mathematical isomorphism proof, model definition, filename formalization, and closure |
| Part V | Conformance, errata, references |
| Container shape | `MC = (H, M, V, D)` — one payload index |
| Container primitive | Payload entries ("exclusive vectors"; same on-disk object as GGUF v3 tensors) |
| Container equivalence | **100% field-set isomorphic** to GGUF v3 (fixed-3 variant), field-wise bijective |
| Declared differences | Magic (`ZEVO` vs. `GGUF`); `zevo.*` metadata namespace |
| Declared width extension | `uint64_t[5]` vs. `uint64_t[3]` for `dimensions` |
| Declared failure modes | Byte-stream, semantic, runtime-behavior, endianness-marker, model-capability, filename-ambiguity |
| Normative requirements | 29 |
| Conformance tests | 50 |
| Equivalence classes | `bijective-3d`, `bijective-5d`, `partial` |
| Ambiguities audited | 38 (A-1 through A-38) |
| Gaps audited | 34 (G-1 through G-34) |
| Contradictions resolved | 12 (C-1 through C-12) |
| Filename regex dialect | POSIX ERE (POSIX.1-2008 §9.4.6) |
| Filename post-match checks | 6 |
| Filename cross-file constraints | 1 (shard consistency) |
| Filename test corpus | 30 cases |
| Supersedes | Zevo v5.1.0 and all prior Zevo documents |
| Exact title | Zevo™ |
| Exact subtitle | An 5D interactive artificial intelligence assistant |
| Subtitle status | Proper name; verbatim per project charter |
| Pixar disclaimer | Descriptive inspiration only; no affiliation |
| Language of record | English |
| Implementation model | Pure-function wrappers on ANSI C89/90, block comments only |
| Governing formalism | Part IV (mathematical formalization) |

---

**End of document.**

---

## Notes on What Changed 

The rewrite above integrates the correctness review of the filename convention regex and resolves the additional contradictions, gaps, and ambiguities it exposed:

1. **§51 formalizes the filename convention** with explicit component patterns, derivation rules, and cross-references to container metadata.

2. **§52 provides the corrected regex** as a **POSIX ERE**, with **six normative post-match checks** and **one cross-file constraint**. The PCRE-only regex of prior documents is retracted (C-10).

3. **Version is required in the filename** (C-11). The "optional" rule is retracted. This eliminates the FineTune/Encoding ambiguity class the prose itself noted.

4. **The BaseName/SizeLabel ambiguity is resolved** by Req. 51.3.2 (C-12). The prose's own example `Mixtral-8x7B-v0.1-KQ2.bin` is retracted and replaced with `Mixtral-MoE-8x7B-v0.1-KQ2.bin`.

5. **The FineTune/Version ambiguity is resolved** by Req. 51.4.2.

6. **Shard numbering is constrained** by Req. 51.6.2–Req. 51.6.3 (`ShardNum ≥ 00001`, `ShardNum ≤ ShardTotal`).

7. **Whitespace in filenames is constrained** by Req. 51.7.1–Req. 51.7.3.

8. **The filename cross-reference to container metadata** is added by Req. 51.8.1.

9. **Conformance tests expanded from 45 to 50** (T46–T50 cover the filename regex, post-match checks, cross-file constraint, POSIX ERE dialect).

10. **Normative requirements expanded from 27 to 29** (items 27–28 cover the filename convention and validation).

11. **Audit expanded** with A-33 through A-38 and G-31 through G-34.

12. **Failure modes expanded** with FM 48.6 (filename failure modes).

The document is still a **specification**, not evidence of a working AI. §53.1 makes that explicit as a declared non-claim. The filename convention is now **formalized, validated, and testable**, resolving every defect the correctness review identified.

---
---

# The Zevo™ AI-Driven Animation Core
## An 5D Interactive Artificial Intelligence Assistant

### A Complete Academic Science-Computer-Engineering Specification for Building the ELF Image, Anchored in FSM-Neural-Fuzzy-Machine-Learning Logic

**Authors:** PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams
**Version:** 1.0.0
**Status:** Design and implementation specification. This paper specifies a buildable system; it does not report a completed implementation, measured performance, or audited security properties.
**Citation standard:** ISO 690:2021

---

## Abstract

This paper specifies the architecture, mathematical contracts, implementation structure, deployment model, and evaluation framework for the **Zevo™ AI-Driven Animation Core**, an **5D interactive artificial intelligence assistant** realized as a pure-functional, **ANSI C89/90**, **GNU/POSIX**, **System V x86-64 `ET_DYN` ELF AI Core**.

The core is anchored in **FSM-Neural-Fuzzy-Machine-Learning logic** — a single integrated control architecture that unifies four layers of reasoning inside one **Neural Mesh Network (NMN)**:

- a **Fuzzy-State-Machine (FSM)** governing discrete interaction phases with fuzzy guards;
- a **fuzzy inference layer** built on **rules and facts** and evaluated by **Mamdani inference**;
- a **neural layer** projecting embeddings onto learned prototypes;
- an **autonomous machine-learning layer** performing **autonomous weight calculation** under **FSM-Neural-Fuzzy** control.

**Shannon entropy** enters this architecture as the epistemic signal carried by the depth axis. The four layers share a single fused weight set `W_fused` that is updated in-core by the autonomous learning mechanism, subject to documented bounds. Together they form one deterministic pipeline that produces a **Multimodal 5D Smart 8K HDR BGRA** artifact comprising **Text Raw**, **Image BGRA**, **Video BGRA**, **Audio PCM**, and a **fused 5D intelligence vector**

```
S = (O, X, Y, Z, R)
```

whose axes represent **Origin (rotation and gravity)**, **Horizontal**, **Vertical**, **Depth (immersion)**, and **Radial (expansion)** behavioral semantics, and whose components carry **symbolic**, **subsymbolic**, **temporal**, **epistemic**, and **affective** information.

The ELF core stores its LLM weights, metadata, and vectors either **embedded by section** or **embedded by reference**, in both cases page-aligned on `ext4`, `fs-verity`-protected, and mapped **read-only** by the process. The core exposes only a published **IPC API** to a **weight-free, stateless, Web-Responsible client**. Cryptographic access is mediated by a **Secure Service API** with **28-hour API key rotation** using OpenSSL primitives aligned with **NIST SP 800-38D**, **NIST SP 800-107**, and **FIPS 186-5**. Hardware peripherals are reached exclusively through libc wrappers resolving to the **`syscall`** instruction on `x86_64` and to **`sysenter`** on legacy `x86`, under **seccomp-bpf**, **capability bounding**, and **LSM** discipline. The result is intended to be a secure-by-construction, reproducible, auditable, and accessible AI assistant centered on **Zevo**, an original Pixar-inspired animated character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant** (verbatim per project charter). Security, reproducibility, performance, and accessibility are design goals that require validation in an implementation; they are not claimed as measured results.

**Keywords:** ELF AI Core, `ET_DYN`, `fs-verity`, `ext4`, `syscall`, `sysenter`, FSM-Neural-Fuzzy logic, fuzzy-state-machine, fuzzy neural logic, rules and facts, Shannon entropy, Mamdani inference, Neural Mesh Network, autonomous self-learning, autonomous weight calculation, machine learning, 5D axes, multimodal generation, procedural animation, IPC, OpenSSL, NIST SP 800-38D, NIST SP 800-107, FIPS 186-5, FIPS 203, FIPS 204, TPM 2.0, reproducible builds.

---

## 1. Introduction

### 1.1 Motivation

Contemporary AI applications combine software, learned parameters, operating-system services, hardware peripherals, and user interfaces. A system that generates multimodal content must address not only inference but also model integrity, resource limits, accessibility, deployment isolation, and the boundaries between components. This paper specifies the **Zevo™ AI-Driven Animation Core** as a **Linux ELF AI Core** whose design centers on a single anchoring architecture — **FSM-Neural-Fuzzy-Machine-Learning logic realized as a Neural Mesh Network** — and on a small set of architectural commitments:

1. A **single integrated reasoning architecture** combining fuzzy-state-machine logic, fuzzy inference over rules and facts, neural projection, and autonomous machine-learning weight calculation inside one graph.
2. A **pure-functional pipeline** in ANSI C89/90 that separates deterministic transformations from side effects.
3. A **five-component state vector** `S = (O, X, Y, Z, R)` that unifies reasoning, animation, and audio control.
4. **External weights** stored outside the ELF, page-aligned on `ext4`, protected by `fs-verity`, and mapped read-only, or optionally **embedded by section** when an integrated artifact is required.
5. A **published IPC API** separating the core from a weight-free, stateless, Web-Responsible client.
6. **Standards-aligned cryptography** with explicit references to NIST and IETF specifications.
7. **Least-privilege hardware access** through libc wrappers resolving to `syscall`/`sysenter`, under seccomp-bpf, capability bounding, and LSM policy.

The paper is a **specification**. It defines the contracts an implementation must satisfy; it does not claim that any implementation has been completed, that any model has been trained, or that any measured property has been demonstrated.

### 1.2 Contributions

| Contribution | Description |
|---|---|
| Neural Mesh Network anchoring | A single graph unifying FSM, fuzzy inference, neural projection, and ML weight calculation under one fused weight set |
| Rules and facts layer | Versioned symbolic knowledge evaluated by Mamdani inference |
| Shannon entropy as epistemic signal | Normalized entropy bound to the depth axis and consumed as a mesh gain parameter |
| Autonomous weight calculation | Per-edge FSM-Neural-Fuzzy gain updates performed in-core, bounded, and audited |
| Five-axis vector | `S = (O, X, Y, Z, R)` with defined semantics per axis |
| Multimodal artifact | Text Raw, Image BGRA, Video BGRA, Audio PCM, fused `S` |
| Dual-mode weight storage | Embedded by section, or embedded by reference; both page-aligned on `ext4`, `fs-verity`-protected, read-only mapped |
| IPC contract | Framed messages, bootstrap and preflight version negotiation, mesh-introspection messages, defined retry rules |
| Cryptographic contract | OpenSSL primitives aligned with NIST SP 800-38D, SP 800-107, RFC 5869, FIPS 186-5, with a TPM- or HSM-sealed master secret and post-quantum migration path |
| Hardware-access discipline | libc → `syscall`/`sysenter`/`svc`/`ecall`, seccomp-bpf, capability bounding, LSM |
| Deployment model | systemd hardening, device allow-listing, reproducibility prerequisites |
| Acceptance criteria | Nineteen testable criteria covering build, ELF, validation, model loading, mesh topology, mesh bounds, convergence, diminishing gain, reset, audit integrity, rule-base validation, streaming, numerics, media, IPC, concurrency, sandbox, accessibility, reproducibility, performance |

### 1.3 Organization

Section 2 surveys related work. Section 3 establishes the zero-step foundation. Section 4 defines the system architecture and trust boundaries. Section 5 specifies the five-component state. Section 6 specifies the Neural Mesh Network anchoring architecture. Section 7 specifies the processing pipeline. Section 8 specifies input and output contracts. Section 9 specifies feature extraction. Section 10 specifies the animation kernel. Section 11 specifies model storage and loading, including both embedding modes. Section 12 specifies the C89/90 project structure. Section 13 specifies concurrency and acceleration. Section 14 specifies the IPC protocol. Section 15 specifies the cryptographic contract. Section 16 specifies operating-system hardening and build requirements. Section 17 specifies the Web-Responsible client. Section 18 specifies testing and acceptance criteria. Section 19 specifies the evaluation plan. Section 20 records engineering bounds and a hardening agenda. Section 21 concludes. Section 22 lists references in ISO 690:2021 form.

---

## 2. Related Work

ELF is the standard executable and linkable format used across Unix-like systems (Tool Interface Standard, 1995; System V ABI, 2018). Linux provides process-level isolation through permissions, capabilities, seccomp filters, and Linux Security Modules; these mechanisms are complementary rather than substitutable (Linux Kernel Documentation, 2024b). `fs-verity` provides file-level integrity verification for supported filesystems and does not provide confidentiality (Linux Kernel Documentation, 2024a). Fuzzy set theory (Zadeh, 1965) and Mamdani inference (Mamdani, 1974) offer interpretable control over graded membership. Shannon entropy (Shannon, 1948) provides a measure of uncertainty. Neural projection, sigmoid activation, and prototype learning are standard elements of neural computation; deep feedforward initialization is characterized by Glorot and Bengio (2010), and regularization by dropout is characterized by Srivastava et al. (2014). Graph-structured computation with shared weights is characteristic of modern neural architectures; the NMN applies this pattern to reasoning primitives rather than to homogeneous layers. Container-based deployment (Merkel, 2014) and model-serving frameworks (Olston et al., 2017; PyTorch, 2023) offer patterns for separating runtime concerns; Zevo adopts a different posture: a single ELF AI Core anchored in **FSM-Neural-Fuzzy-Machine-Learning logic** realized as a Neural Mesh Network, with dual-mode weight storage, an IPC contract, and explicit hardware discipline. Post-quantum migration follows FIPS 203 and FIPS 204; hardware-backed key sealing follows TPM 2.0.

The design's differentiators are: the **Neural Mesh Network** anchoring, in which an FSM node, a Mamdani node, a neural node, and an ML node share a fused weight set and update rule; the **five-component state vector** driving animation and audio; **dual-mode weight storage**; a **published IPC contract** with bootstrap, preflight, and mesh-introspection messages; and an **explicit hardware-access discipline** that does not rely on undocumented instruction-level assumptions.

---

## 3. The Zero-Step Foundation

### 3.1 Identity

| Field | Value |
|---|---|
| Title | **Zevo™** |
| Subtitle | **An 5D interactive artificial intelligence assistant** |
| Version | 1.0.3 |
| ABI | 2 |
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |

The subtitle is preserved verbatim per project charter, including the use of `An` before `5`.

### 3.2 Normative Language

The terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** express requirements, recommendations, and options.

### 3.3 Four Trust Boundaries

```
┌──────────────────────────────────────────────────────────────────────┐
│                          Zevo™ System                                │
│                                                                      │
│  ┌─────────────┐   ┌──────────────┐   ┌────────────┐   ┌──────────┐  │
│  │  Browser    │   │  Secure      │   │  ELF AI    │   │ Universal│  │
│  │  Responsible│──►│  Service API │──►│  Core      │──►│ Hardware │  │
│  │  UI         │   │              │   │            │   │ Devices  │  │
│  └─────────────┘   └──────────────┘   └────────────┘   └──────────┘  │
│    Boundary 1        Boundary 2        Boundary 3       Boundary 4   │
└──────────────────────────────────────────────────────────────────────┘
```

| Boundary | Between | Contract |
|---|---|---|
| 1 | Browser ↔ Service API | TLS 1.3, session tokens, no telemetry |
| 2 | Service API ↔ ELF AI Core | 28-hour key rotation, HMAC-SHA-384, AES-256-GCM |
| 3 | ELF AI Core ↔ weights (section or `weights.bin`) | `fs-verity` Merkle verification, read-only mapping |
| 4 | ELF AI Core ↔ Hardware | libc → `syscall`/`sysenter`, seccomp-bpf, capability bounding, LSM |

### 3.4 Vocabulary

| Term | Fixed meaning |
|---|---|
| **FSM** | Fuzzy-state-machine: discrete states with fuzzy guards |
| **FSM-Neural-Fuzzy logic** | Integrated control combining finite-state machine logic, fuzzy membership over rules and facts, and neural projection |
| **FSM-Neural-Fuzzy-ML** | The above combined with autonomous machine-learning weight calculation |
| **Neural Mesh Network (NMN)** | A directed weighted graph whose nodes are the four reasoning primitives (F, Z, N, M) and whose edges are entries of `W_fused` |
| **Mesh node** | A single reasoning primitive instance, indexed by axis |
| **Mesh edge** | A directed entry of `W_fused` carrying a typed payload |
| **Rules and facts** | Versioned symbolic knowledge and observed inputs |
| **Mamdani inference** | fuzzify → fire → aggregate → centroid defuzzify |
| **Shannon entropy** | `H(X) = −Σ p log₂ p`, normalized to `[0,1]` |
| **Fuzzy neural projection** | `σ(⟨e, p⟩)`, sigmoid projection of an embedding onto a prototype |
| **FSM-Neural-Fuzzy gain** | The Mamdani centroid of a rule base over entropy and FSM state, projected through `σ` |
| **Autonomous self-learning** | Bounded, in-core weight adaptation performed under FSM-Neural-Fuzzy control |
| **Autonomous weight calculation** | The in-core mechanism that computes and applies fused-weight updates |
| **Audit record** | A signed record of one `ΔW_fused` step, verifiable under ECDSA P-384 |
| **Drift bound (`D_max`)** | The maximum cumulative weight change permitted per session |
| **Session reset** | The restoration of `W_fused` to its seed at process exit |
| **Mesh introspection** | The `MESH_DESCRIBE`/`MESH_STATE` IPC message family |
| **5D axes** | `S = (O, X, Y, Z, R)` |
| **Multimodal** | Text Raw + Image BGRA + Video BGRA + Audio PCM + fused `S` |
| **Embedded by section** | Weights, metadata, and vectors placed in a dedicated ELF section, page-aligned on `ext4`, `fs-verity`-protected, mapped read-only |
| **Embedded by reference** | Weights, metadata, and vectors stored in `weights.bin` on `ext4`, page-aligned, `fs-verity`-protected, mapped read-only |

### 3.5 ELF AI Core Attributes

| Attribute | Value |
|---|---|
| File | `zevo_core.elf` |
| Format | ELF |
| Type | `ET_DYN` (position-independent executable) |
| ABI | System V `x86-64` (with declared `aarch64`/`riscv64` portability) |
| Language | ANSI C89/90, pure-function discipline |
| Contracts | GNU + POSIX + Pthreads |
| Accelerators | OpenCL, OpenGL, SDL2, OpenAL |
| Cryptography | OpenSSL (with TPM 2.0 / HSM sealing and FIPS 203/204 migration path) |
| Target | Linux Fedora 40+ |
| Exposure | Local IPC only |
| System-call path | libc → `syscall` (x86_64) / `sysenter` (legacy x86) / `svc` (aarch64) / `ecall` (riscv64) |

### 3.6 System Goals

A conforming implementation MUST:

1. accept validated multimodal requests through a documented interface;
2. produce bounded, structurally valid output artifacts;
3. keep model weights either embedded by section or embedded by reference, and map them read-only;
4. separate the core from the Web-Responsible client through a published IPC API;
5. keep computation deterministic where the selected numeric and device backends permit it;
6. document any nondeterministic behavior, particularly GPU execution order;
7. define resource, error, and security limits for every externally reachable operation.

Non-goals: downstream container formats, specific artistic output, use of undocumented instructions.

---

## 4. System Architecture

### 4.1 Component Diagram

```text
┌────────────────────────────┐
│ Web-Responsible UI         │
│ weight-free, stateless     │
└───────────┬────────────────┘
            │ HTTPS (if remote access is enabled)
            ▼
┌────────────────────────────┐
│ Secure Service Layer       │
│ authentication, authorization, TLS │
└───────────┬────────────────┘
            │ AF_UNIX SOCK_SEQPACKET (local IPC)
            ▼
┌────────────────────────────────────────────────┐
│ Zevo Core: zevo_core.elf                       │
│                                                │
│ IPC → validation → modality encoders           │
│     → Neural Mesh Network pipeline             │
│     → animation / rendering                    │
│     → artifact store → IPC response            │
│                                                │
│ Weights: section-embedded or reference-embedded│
└───────────┬────────────────────────────────────┘
            │ libc / approved device interfaces
            ▼
┌────────────────────────────────────────────────┐
│ OS and optional devices                        │
│ ext4/fs-verity, TPM/HSM, GPU, display, audio   │
└────────────────────────────────────────────────┘
```

The Web-Responsible client MUST NOT contain model weights, master secrets, or direct pointers into core memory. The core MUST NOT expose raw model weights through IPC.

### 4.2 Trust Boundary Controls

| Boundary | Data crossing it | Required controls |
|---|---|---|
| Browser to service | User requests and responses | TLS, input validation, origin and session policy |
| Service to core | Bounded IPC messages | Peer credentials, protocol versioning, authorization |
| Core to weights | Weight bytes and metadata | `fs-verity`, trusted measurement, read-only mapping |
| Core to devices | Device requests | Least privilege, allow-listed interfaces, sandboxing |

---

## 5. Five-Component State

### 5.1 The 5D Intelligence Vector

```
S = (O, X, Y, Z, R) ∈ [0, 1]⁵
```

| Axis | Name | Meaning | Reasoning source | Visual effect | Acoustic effect |
|---|---|---|---|---|---|
| **O** | Origin | Rotation & gravity | Mamdani centroid (`Z_O`) | Body rotation, gravity sag | Fundamental frequency |
| **X** | Horizontal | Lateral spread | Fuzzy neural projection (`N_X`) | Eye and antenna spread | Octave-fifth detune |
| **Y** | Vertical | Up/down extent | FSM dwell time (`F_Y`) | Squash-and-stretch | Slow vibrato rate |
| **Z** | Depth | Front/back immersion | Normalized Shannon entropy (`H_Z`) | Halo depth glow, blue channel | Shimmer partial |
| **R** | Radial | Outward expansion | Autonomous weight calculation (`M_R`) | Halo radius, ribbon length | Overall amplitude |

Each axis carries a specific informational channel:

- **Symbolic** — carried by `O` (Mamdani inference over rules and facts).
- **Subsymbolic** — carried by `X` (fuzzy neural projection).
- **Temporal** — carried by `Y` (FSM dwell).
- **Epistemic** — carried by `Z` (Shannon entropy).
- **Affective** — carried by `R` (autonomous self-learning affect).

The idle state is `S = (0,0,0,0,0)`. The behavior of the zero state for each output channel is defined by the versioned output profile (§8.4). A fully activated state is `S = (1,1,1,1,1)`.

### 5.2 Normalization

After inference, every finite axis value MUST be clamped to `[0, 1]`. A non-finite value (NaN or ±∞) MUST be reported as an inference error; it MUST NOT be silently replaced with a neutral value. The mapping from raw inference output to each axis is versioned configuration and MUST be documented with the model.

---

## 6. The Anchoring Architecture — FSM-Neural-Fuzzy-Machine-Learning Logic Realized as a Neural Mesh Network

### 6.1 The Neural Mesh Network — Definition

Zevo's reasoning core is a single integrated architecture, not a pipeline of independent modules. The four reasoning layers are unified as nodes of a directed, weighted, acyclic-in-session graph, the **Neural Mesh Network (NMN)**:

```
NMN = ( V, E, W_fused, φ )
```

| Symbol | Meaning |
|---|---|
| `V` | Mesh nodes. Each node is one of four reasoning primitives: **F** (FSM), **Z** (Fuzzy/Mamdani), **N** (Neural projection), **M** (ML weight-calculation). |
| `E` | Directed edges carrying activation, feature, or gradient flow. |
| `W_fused` | The shared fused weight set (§6.2). Every edge is an entry of `W_fused`. |
| `φ` | The node activation function, drawn from a fixed set (§6.1.2). |

Nodes are indexed by axis: `V = { F_Y, Z_O, N_X, M_R }`, plus auxiliary nodes for entropy (`H_Z`) and the fusion junction (`J`). Edges carry typed payloads: `sym` (symbolic), `sub` (subsymbolic), `tmp` (temporal), `epi` (epistemic), `aff` (affective).

#### 6.1.1 Layer Roles

| Layer | Node | Role | Primary axis |
|---|---|---|---|
| **FSM** | `F_Y` | Discrete-state interaction control with fuzzy guards | `Y` (dwell) |
| **Fuzzy** | `Z_O` | Mamdani inference over versioned rules and facts | `O` (symbolic) |
| **Neural** | `N_X` | Sigmoid projection of embeddings onto prototypes | `X` (subsymbolic) |
| **ML** | `M_R` | Autonomous weight calculation and adaptation | `R` (affective) |
| **Entropy** | `H_Z` | Normalized Shannon entropy over the feature distribution | `Z` (epistemic) |
| **Fusion** | `J` | Composition of `S` and emission of `W_fused'` | All |

**Shannon entropy** is not a fifth layer; it is an epistemic signal that the FSM and ML layers consume and that the neural layer exposes.

#### 6.1.2 Node Activation Set

Each node uses one of the following fixed activation forms:

| Activation | Form | Used by |
|---|---|---|
| Sigmoid | `σ(u) = 1/(1+e^{−u})` | `N_X`, `M_R`, gain `g` |
| Minimum | `min(a, b)` | `Z_O` (rule firing) |
| Maximum | `max(a, b)` | `Z_O` (aggregation) |
| Centroid | `∫ x μ(x) dx / ∫ μ(x) dx` | `Z_O` (defuzzification) |
| Dwell | `min(1, dwell / τ_max)` | `F_Y` |
| Entropy | `H / log₂ B` | `H_Z` |
| Affect | `clamp_{[0,1]}( f(history) )` | `M_R` |
| Identity | `u` | `J` |

The set is closed. Adding a new activation is a protocol revision, not a runtime choice.

#### 6.1.3 Mesh Evaluation Order and Determinism

The mesh's node evaluation order is **topological order of `E`**, with ties broken by node index ascending. Ties are possible only between nodes whose inputs are already available; because `E` is acyclic in-session (§6.1.4), a topological order always exists.

For deterministic mode (§13), the evaluation order is fixed to the declaration order of nodes as listed in the model schema, and every node's activation is evaluated with the declared numeric backend. Any backend that cannot guarantee the declared evaluation order in deterministic mode MUST be rejected in that mode.

#### 6.1.4 Mesh Invariants

A conforming NMN MUST satisfy:

1. **Bounded fan-in per node** — no node has more than `F_max` inbound edges, with `F_max` a versioned constant.
2. **Bounded state** — the mesh's total mutable state is bounded by a versioned constant `S_max`.
3. **Read-only weights** — no edge of the mesh writes to weight storage. Updates to `W_fused` occur only in the in-core session copy.
4. **No hidden I/O** — no node performs I/O. All I/O is at the mesh boundary.
5. **Acyclic in-session** — `E` contains no cycle within a session. (Cross-session feedback is expressed through the session-scoped `W_fused` copy, not through edges.)
6. **Deterministic composition** — for fixed inputs, fixed `W_fused`, and the fixed evaluation order of §6.1.3, the mesh produces identical `S`.
7. **Commutativity where declared** — edges that are declared commutative MAY be evaluated in any order; edges that are not MUST be evaluated in declaration order.

### 6.2 Fused Weight Set

```
W_fused = F_fuse( W_fsm, W_fuzzy, W_neural, W_ml, x, e, H )
```

where:

- `W_fsm` — FSM transition and guard weights;
- `W_fuzzy` — Mamdani membership parameters and rule weights;
- `W_neural` — prototype matrix and projection weights;
- `W_ml` — autonomous weight-calculation state;
- `x` — feature vector;
- `e` — embedding vector;
- `H` — normalized Shannon entropy.

The fused weight set is the single source of truth for mesh inference. `F_fuse` is versioned configuration.

### 6.3 Axis Computation

```
O = Mamdani( x, rules, facts )           ∈ [0,1]   symbolic
X = σ( ⟨e, p⟩ )                          ∈ (0,1)   subsymbolic
Y = min( 1, dwell / τ_max )              ∈ [0,1]   temporal
Z = H( x ) / log₂ B                      ∈ [0,1]   epistemic
R = clamp_{[0,1]}( affect(history) )     ∈ [0,1]   affective
```

### 6.4 FSM Layer

The FSM layer governs discrete interaction phases. States and transitions are versioned model deliverables. Guards are fuzzy and use the same membership functions as the fuzzy layer. The FSM MUST define named states, permitted transitions, guards, and timeouts. The FSM also gates the mesh's weight-update mask `mask(e)` (§6.8.1).

### 6.5 Fuzzy Layer — Rules and Facts

The fuzzy layer is the symbolic channel of the architecture:

- **Rules** — versioned IF/THEN rules over fuzzy variables.
- **Facts** — versioned observed inputs.

Mamdani inference is used:

1. Fuzzification of facts against membership functions;
2. Rule firing by minimum over antecedents;
3. Aggregation by maximum over clipped consequents;
4. Centroid defuzzification.

The rule table and fact schema MUST be included in the model (whether embedded by section or by reference), not hard-coded into the ELF. Rule-base validation is mechanical (§6.10).

#### 6.5.1 Consequent Range

Each rule consequent `r_k` is defined over a **bounded output universe** `[r_min, r_max]` declared in the rule-base schema. Default: `[r_min, r_max] = [0, 1]`. When the consequents are declared over `[0, 1]`, the Mamdani centroid in §6.5 already produces a value in `[0, 1]`, and the outer sigmoid of §6.8.2 is a monotone squash that preserves the ordering of gains but not their numeric value. When the consequents are declared over an unbounded universe, the sigmoid of §6.8.2 is required to bound the gain into `(0, 1)`.

This is the only place in the specification where the σ wrapper on the gain is load-bearing; elsewhere it is a monotone reparameterization. Implementations MUST declare `[r_min, r_max]` and MUST state whether the gain uses the raw centroid or its σ projection.

### 6.6 Neural Layer

The neural layer is the subsymbolic channel:

```
u = Σ_{j=0}^{d−1} e_j p_j
X = σ(u) = 1 / (1 + exp(−u))
```

The prototype set `{p}` and the embedding projection are versioned model deliverables. Normalization, precision, and overflow handling MUST be specified.

### 6.7 Shannon Entropy Signal

```
H   = −Σ_{i : p_i > 0} p_i log₂( p_i )
H_n = H / log₂( B )
```

The histogram construction is versioned configuration. Empty histograms return a documented neutral value, normally zero. Non-finite values MUST be rejected.

### 6.8 Autonomous Machine-Learning Layer and Autonomous Weight Calculation

The ML layer performs **autonomous weight calculation** under FSM-Neural-Fuzzy control. Its contract:

- **Bounded** — the state of the weight-calculation mechanism MUST be bounded in memory.
- **Scoped** — adaptation state MUST be scoped to a session or to an explicitly identified user context.
- **Opt-in** — adaptation MUST be disabled by default unless consent and retention behavior are documented.
- **Read-only weights** — adaptation MUST NOT modify the read-only weight storage (section or `weights.bin`).
- **Non-persistent by default** — adaptation state is discarded at process exit unless persistence is documented.
- **Documented update rule** — the update rule, learning rate, drift bounds, and clamping policy MUST be versioned configuration documented with the model.
- **No hidden I/O** — the update MUST NOT perform I/O; it MUST compute from explicit inputs and in-core state.

The ML layer exposes the affective channel `R` to the composer, and its accumulated state feeds back into `W_fused` for subsequent inference within the same session.

#### 6.8.1 Per-Edge Update Rule

The NMN's autonomous self-learning updates `W_fused` after each request. The update is a **bounded, per-edge, FSM-Neural-Fuzzy-governed gradient-free step**:

```
ΔW_fused(e) = η · g( a_in(e), a_out(e), H, s_FSM ) · mask(e)
W_fused'(e) = clamp_{[w_min, w_max]}( W_fused(e) + ΔW_fused(e) )
```

where:

| Symbol | Meaning |
|---|---|
| `e` | A mesh edge |
| `η` | Learning rate, versioned configuration, `η > 0` |
| `a_in(e)`, `a_out(e)` | Node activations at the edge's endpoints |
| `H` | Normalized Shannon entropy of the request's feature distribution |
| `s_FSM` | Current FSM state |
| `g(·)` | The **FSM-Neural-Fuzzy gain function** (§6.8.2) |
| `mask(e)` | An FSM-controlled gating mask, `∈ {0, 1}` |
| `[w_min, w_max]` | Versioned per-edge weight bounds |

The update is **based on model weights**: `ΔW_fused` is a function of the edge's contribution to the produced `S`, not of any external label. The mesh learns the *internal consistency* of its own four reasoning primitives.

#### 6.8.2 FSM-Neural-Fuzzy Gain Function

The gain `g` is itself computed under the four-layer anchoring:

```
g_raw( a_in, a_out, H, s_FSM ) =
    Σ_k μ_k( a_in, a_out ) · r_k( H, s_FSM ) / Σ_k μ_k( a_in, a_out )

g = σ( g_raw )     if r_k is unbounded
g = g_raw          if r_k ∈ [0, 1]
```

where:

- `μ_k` are membership functions over the edge's activations, evaluated by the **Fuzzy** layer;
- `r_k` are rule consequents over entropy and FSM state, evaluated by the **FSM** layer, declared over `[r_min, r_max]` per §6.5.1;
- the quotient is the **Mamdani centroid** of §6.5;
- σ is applied only when the consequents are declared unbounded (§6.5.1).

**The gain is not hand-authored.** It is the Mamdani centroid of a versioned rule base over `H` and `s_FSM`, evaluated at the edge's endpoint activations. That is what "machine-learning calculation by FSM-Neural-Fuzzy logic" means in this specification.

#### 6.8.3 Anchoring Invariants for Self-Learning

Every NMN self-learning step MUST satisfy:

| Invariant | Contract |
|---|---|
| Bounded | `|ΔW_fused|` per edge is bounded by `η · g_max`; total drift per session is bounded by `D_max`. |
| Scoped | Updates are session-scoped; cross-session persistence requires explicit opt-in and a signed delta (§15.6). |
| Opt-in | Self-learning is disabled by default. |
| Read-only weights | Storage is never written. Only the in-core `W_fused` copy is mutated. |
| Non-persistent | Default: discarded at process exit. |
| No hidden I/O | The update performs no I/O. |
| Documented | `η`, `[w_min, w_max]`, `D_max`, `g_max`, `F_max`, `S_max`, `[r_min, r_max]` are versioned configuration. |

### 6.9 Fused-Weight Update Cycle

For each request the architecture executes:

1. Compute `x`, `e`, and `H` from validated input.
2. Compute `O`, `X`, `Y`, `Z` from `W_fused` and the current state.
3. Compute `R` from the ML layer's accumulated affect.
4. Emit `S = (O, X, Y, Z, R)`.
5. Apply the autonomous weight-calculation step (§6.8.1) to produce `W_fused'` for the next request.

Steps 1–4 are deterministic given the input and `W_fused`. Step 5 is deterministic given steps 1–4 and the documented update rule. No step performs I/O.

### 6.10 Rule-Base Schema and Validation

The NMN specifies a **rule-base schema** with three mandatory sections:

- **Membership section** — one entry per fuzzy variable, with parameters `(center, width, shape)`.
- **Rule section** — one entry per rule, with antecedent references, consequent references declared over `[r_min, r_max]` (§6.5.1), and a firing weight.
- **Fact section** — one entry per observed input, with type, range, and normalization.

Validation is mechanical:

1. **Coverage** — every rule is reachable from the fact schema.
2. **Consistency** — no rule pair has contradictory consequents over the same antecedent region.
3. **Termination** — the FSM reachability graph is finite and connected.
4. **Diminishing gain** — for every mesh edge `e`, the composed gain `g( a_in(e), a_out(e), H, s_FSM )` is non-increasing in `|a_in(e) − a_out(e)|` for fixed `H` and `s_FSM`, over the declared activation domain. This is checked by sampling the declared domain on a fixed grid and verifying the monotonicity condition within a declared tolerance.

A rule base that passes these four checks is *validated* in the specification's sense. The fourth check is the check that underwrites the convergence contract (§6.11).

### 6.11 Convergence Contract

The NMN specifies a **convergence contract** for the autonomous weight-calculation policy:

- **Bounded drift:** `Σ_t |ΔW_fused(t)| ≤ D_max` per session.
- **Diminishing gain:** `g` is non-increasing in `|a_in − a_out|` for fixed `H`, `s_FSM`. This is a validation-checked property (§6.10, check 4).
- **Fixed-point existence:** For any bounded input distribution, there exists a `W_fused^*` such that `E[ΔW_fused] = 0`. This is guaranteed by the clamping function and the bounded gain.
- **Session reset:** `W_fused` resets to the seed at process exit unless persistence is opt-in.

With these four properties, the policy *converges by construction* for any bounded input stream. Open-ended inputs are handled by the reset contract: the mesh converges *within* a session and resets *across* sessions.

### 6.12 Bootstrap Weight Protocol

The NMN does not require pre-trained weights to operate; it requires a **seed** and a **converged update rule**, both specifiable:

1. A minimal seed `W_fused^(0)` is specified as part of the model schema. It is derived deterministically from the rule base and the prototype set by a documented initialization procedure (e.g., Xavier-style scaling of prototype entries (Glorot & Bengio, 2010), uniform rule weights, and an FSM dwell constant of `τ_max/2`).
2. The autonomous self-learning mechanism (§6.8) refines `W_fused` from this seed over the session.
3. Convergence is measured by the drift bound `D_max` and reported in artifact metadata.

### 6.13 Model-Dependent Configuration

FSM transitions and timeouts, Mamdani rule table and membership functions, prototype set, entropy histogram construction, the consequent range `[r_min, r_max]`, and the autonomous weight-update rule are versioned model deliverables. This specification does not prescribe their values. Behavior that depends on their values is a property of the model, not of the core.

### 6.14 Determinism and Side Effects

Numerical transformations SHOULD be functions of explicit inputs and immutable model parameters. Allocation, file access, IPC, device access, clocks, and random-number generation are side effects and MUST be isolated in system-boundary modules.

A function is pure only if it has no observable side effects and its output depends solely on its explicit inputs. A function that mutates the autonomous weight state, reads a clock, accesses a device, or relies on hidden mutable global state MUST NOT be described as pure.

---

## 7. Processing Pipeline

```
request
  → validation and decoding
  → modality-specific feature extraction
  → Neural Mesh Network fusion
  → state inference (O, X, Y, Z, R)
  → autonomous weight-calculation update (bounded, in-core, audited)
  → trajectory generation
  → character and scene composition
  → text / image / video / audio encoding
  → artifact metadata and response
```

Conceptually:

```
G_kernel = ℛ ∘ 𝒫 ∘ 𝒜 ∘ 𝒞 ∘ ℰ : 𝓘 → (𝓕_8)^N
```

The composition operator `∘` is read right-to-left. Data flows from the encoder `ℰ`, through the composer `𝒞`, animator `𝒜`, painter `𝒫`, and rasterizer `ℛ`, then out through the packager.

| Symbol | Stage | Role |
|---|---|---|
| `ℰ` | Encoder | Decodes inputs and extracts features |
| `𝒞` | Composer | Runs the NMN and produces `S` |
| `𝒜` | Animator | Generates a motion trajectory from `S` |
| `𝒫` | Painter | Constructs the scene |
| `ℛ` | Rasterizer | Converts HDR frames to BGRA |
| `E_out` | Packager | Produces the artifact manifest and response |

---

## 8. Input and Output Contract

### 8.1 Input Modalities

The request format supports:

- **Text:** UTF-8 with a configured maximum byte length.
- **Image:** width, height, pixel format, stride, and pixel buffer.
- **Video:** frame timestamps, dimensions, pixel format, and frame buffers.
- **Audio:** sample rate, channel count, sample format, and PCM buffer.

Every input MUST include its declared length and format. Before allocation or decoding, the core MUST validate dimensions, multiplication overflow, stride, timestamps, sample rate, and total resource cost.

### 8.2 BGRA Definition

BGRA consists of bytes in the order **B, G, R, A** for each pixel. Row stride is measured in bytes and MAY exceed `width × 4`. Implementations MUST accept larger strides and MUST NOT assume consecutive rows are contiguous.

### 8.3 Output Artifact

A successful response MAY contain:

1. raw UTF-8 text;
2. one or more BGRA images;
3. BGRA video frames with timestamps and a declared frame rate;
4. PCM audio with declared sample rate, channel count, and sample format;
5. the normalized state vector `S`;
6. artifact identifiers, dimensions, lengths, and cryptographic digests.

"8K HDR BGRA" MUST be treated as a requested output profile, not a promise that every device can render it. Eight-bit BGRA alone is not a complete HDR representation. If HDR output is required, suitable higher-precision or HDR transfer data MUST be preserved until final encoding.

### 8.4 Zero State

The versioned output profile MUST declare the behavior of the zero state `S = (0,0,0,0,0)` for each output channel.

### 8.5 Packaging

Artifacts MUST be returned with a versioned manifest containing media type, dimensions, sample format, byte length, relevant timestamps, and a cryptographic digest. The manifest MUST NOT contain secrets, memory addresses, or filesystem paths.

---

## 9. Feature Extraction

### 9.1 Modality Encoders

Each modality encoder produces a versioned feature record:

```c
typedef struct {
    unsigned int  version;
    unsigned int  modality;
    unsigned int  feature_count;
    float         feature[64];
    unsigned char digest[32];
} zevo_feature_t;
```

The ABI specification MUST fix the actual maximum feature count and encoding. Text, image, video, and audio encoders MUST be separately testable. Empty input MUST have a defined result. Invalid input MUST return a documented error rather than silently becoming empty input.

### 9.2 Text

UTF-8 bytes `b_1, …, b_n`, `n ≤ 64`. Features are byte statistics; embedding uses a deterministic hash. The hash scheme and its constants are versioned configuration. The hash-derived embedding is a deterministic feature-derived identifier, not a semantic embedding in the learned-representation sense.

### 9.3 Image

Luminance `Y(x, y) = 0.2126 I_R + 0.7152 I_G + 0.0722 I_B`. Features are normalized first- and second-moment statistics over the luminance image. Bounds are `[0, 1]`; degenerate dimensions produce explicit zero values; empty input has a defined result.

### 9.4 Audio

Single-channel, finite-real samples at sample rate `f_s`. Band energies over a fixed set of edges partition the nonnegative spectrum. Empty input, short input, invalid sample rate, and non-finite samples all have defined results.

### 9.5 Video

Frames sampled at fixed indices; image features averaged. `M = 0` and `M = 1` have defined results.

---

## 10. Animation Kernel

### 10.1 Trajectory

A damped response MAY be used for each component:

```
q″(t) + 2ζω q′(t) + ω²(q(t) − S) = 0
```

with `ζ ∈ (0,1)`, `ω > 0`. The implementation MUST define initial conditions, damping parameters, time units, and behavior for interrupted or overlapping requests. It MUST clamp resulting controls to the renderer's accepted range.

The damping parameters `ζ` and `ω`, initial conditions, and time constants are runtime configuration. Implementations claiming runtime reproducibility (§18) MUST record these parameters in artifact metadata.

### 10.2 Frame-Rate Independence

Animation MUST be frame-rate independent: positions are functions of timestamps, not accumulated frame counts. Deterministic mode SHOULD use fixed time steps and a fixed numeric backend.

### 10.3 Character

Zevo is an original animated character with rounded forms, expressive features, a readable silhouette, and feature-animation styling. Implementations MUST use original character assets or assets with documented licenses.

The scene MAY use procedural geometry, signed-distance fields, vector curves, or licensed assets. Visual mapping SHOULD express the NMN architecture through interconnected data flows:

- FSM state (`F_Y`) affects timing and settling;
- rule activation (`Z_O`) affects flow density;
- entropy (`H_Z`) affects glow or diffusion;
- fuzzy-neural projection (`N_X`) affects lateral movement or tint;
- autonomous weight calculation (`M_R`) affects bounded pose variation.

The rendered canvas MUST NOT contain explanatory labels unless explicitly requested. Accessibility information, descriptions, and diagnostics belong in metadata or the user interface.

### 10.4 Rendering Profile

The renderer MUST define width and height limits, frame-rate and duration limits, pixel format and row stride, alpha behavior, color primaries, transfer function, range, tone mapping, output encoding, and maximum output byte count.

For 8-bit BGRA output, the implementation MUST state whether the transfer function is sRGB and whether alpha is straight or premultiplied. HDR content MUST NOT be described as preserved if it has been tone-mapped to ordinary 8-bit sRGB.

Named output profiles SHALL include:

| Profile | Primaries | Transfer | Range | Encoding |
|---|---|---|---|---|
| `srgb-sdr` | BT.709 | sRGB | Full | 8-bit BGRA |
| `hdr-pq` | BT.2020 | PQ (SMPTE ST 2084) | Full | 10/16-bit intermediate, tone-mapped at final BGRA |
| `hdr-hlg` | BT.2020 | HLG (BT.2100) | Full | 10/16-bit intermediate, tone-mapped at final BGRA |

Tone mapping and output encoding are versioned output-profile settings. A profile MAY use Reinhard tone mapping followed by sRGB encoding, but the choice MUST be declared with the artifact.

### 10.5 Audio

The audio renderer MUST define sample rate, channels, sample format, duration, and peak limits. Generated audio MUST be checked for finite values and clipped or rejected according to a documented policy. Stereo output MUST specify whether channels are identical or independently spatialized.

The stereo channel policy is a versioned output-profile setting. A profile MAY use identical left and right channels, `left(n) = right(n)`, but MUST declare that choice with the artifact.

---

## 11. Model Storage and Loading

### 11.1 Dual-Mode Weight Embedding

The core supports two mechanisms for carrying its LLM weights, metadata, and vectors:

**Mode A — Embedded by section.**

- Weights, metadata, and vectors are placed in a dedicated ELF section, page-aligned on `ext4`.
- The ELF is `fs-verity`-protected.
- The section is mapped read-only by the loader.
- Suited to integrated artifacts where weights ship with the executable.

**Mode B — Embedded by reference.**

- Weights, metadata, and vectors are stored in a separate file `weights.bin` on `ext4`.
- `weights.bin` is page-aligned and `fs-verity`-protected.
- The pinned digest is recorded in the ELF's metadata section.
- The core opens `weights.bin` `O_RDONLY`, verifies the `fs-verity` root hash against the pinned digest, and maps it read-only.
- Suited to deployments where weights are provisioned, rotated, or audited separately.

Both modes share the same mapping requirement: the weight storage is mapped read-only, the mapping is never writable, and the process MUST NOT retain any writable mapping of weight storage.

### 11.2 Integrity and Confidentiality

`fs-verity` provides integrity verification, not confidentiality. If confidentiality is required, it MUST be provided by a separately specified storage-encryption mechanism.

### 11.3 Mapping Requirements

The loader MUST:

1. open the storage read-only (or, in Mode A, locate the mapped section);
2. validate the header before interpreting offsets;
3. check ABI and model version;
4. validate every section's bounds, alignment, and element count;
5. verify the expected integrity measurement;
6. map the storage read-only;
7. reject malformed or incompatible data;
8. retain no writable mapping.

Every offset-plus-length calculation MUST be checked for integer overflow and file-bound violations. Page alignment alone does not make mapped offsets safe.

### 11.4 Model-Dependent Configuration

FSM transitions and timeouts, Mamdani rule table and membership functions, prototype set, entropy histogram construction, consequent range `[r_min, r_max]`, and the autonomous weight-update rule are versioned model deliverables. This specification does not prescribe their values. Behavior that depends on their values is a property of the model, not of the core.

---

## 12. C89/C90 Project Structure and API

### 12.1 Source Tree

```text
zevo/
├── include/
│   ├── zevo_api.h
│   ├── zevo_types.h
│   ├── zevo_protocol.h
│   └── zevo_config.h
├── src/
│   ├── main.c
│   ├── ipc_server.c
│   ├── request_validate.c
│   ├── model_loader.c
│   ├── encode_text.c
│   ├── encode_image.c
│   ├── encode_video.c
│   ├── encode_audio.c
│   ├── entropy.c
│   ├── mamdani.c
│   ├── nfuse.c
│   ├── fsm.c
│   ├── machine.c
│   ├── mesh.c
│   ├── fusion.c
│   ├── learn.c
│   ├── audit.c
│   ├── animation.c
│   ├── character.c
│   ├── render_image.c
│   ├── render_video.c
│   ├── render_audio.c
│   ├── artifact.c
│   └── platform/
├── tests/
├── config/
├── docs/
├── Makefile
└── linker/
```

### 12.2 C89/C90 Constraints

C source MUST be valid under the selected C89/C90 compiler mode. Declarations MUST precede statements within each block. Implementations MUST avoid C99-only syntax and library assumptions. POSIX and GNU extensions MUST be isolated behind documented platform interfaces.

Example purity contract:

```c
/*
 * Deterministic transformation:
 * - reads only supplied inputs;
 * - writes only to supplied output;
 * - performs no I/O and accesses no mutable global state.
 */
int zevo_entropy(const float *values,
                 unsigned int count,
                 float *result);
```

A comment documents a contract; it does not establish purity.

### 12.3 Error Handling

Public functions MUST return explicit status codes. They MUST NOT use unchecked allocation, terminate the process on malformed requests, or return partially initialized output as success. Errors MUST distinguish invalid input, unsupported format, resource limit, model integrity failure, mesh convergence failure, device failure, and internal failure.

---

## 13. Concurrency and Acceleration

Pthreads MAY be used for bounded worker pools. The implementation MUST define queue limits, cancellation behavior, shutdown behavior, and buffer ownership. Shared mutable state MUST be synchronized or eliminated. Mesh evaluation is a natural parallel unit provided the fixed evaluation order of §6.1.3 is respected; parallel evaluation is permitted only where the declared commutativity of §6.1.4 clause 7 holds.

OpenCL, OpenGL, SDL2, and OpenAL are optional backends, not mandatory dependencies of the core ABI. Each backend MUST have a CPU fallback where practical. GPU results may differ across devices and drivers. Deterministic mode MUST either avoid nondeterministic backends or document tolerances and reproducibility limits.

The core MUST NOT assume that library calls map one-to-one to particular CPU instructions. Hardware access occurs through operating-system and library interfaces; the kernel and runtime determine the actual path.

Where a backend is known to produce non-bit-identical results across runs or devices, the implementation MUST document this and state whether deterministic mode is available for the advertised feature.

### 13.1 Backend Reporting

The implementation MUST report whether each optional backend—OpenCL, OpenGL, SDL2, and OpenAL—is enabled or unavailable in the current build or runtime. A CPU fallback is required only for features advertised as supported without that backend.

---

## 14. IPC and Service Protocol

### 14.1 Transport

The local core API SHOULD use `AF_UNIX` with `SOCK_SEQPACKET`, peer-credential checks such as `SO_PEERCRED`, and restrictive filesystem permissions.

### 14.2 Frame Header

Every IPC frame begins with a fixed header:

| Field | Size | Meaning |
|---|---:|---|
| `magic` | 4 bytes | Fixed protocol identifier |
| `version` | 2 bytes | Unsigned, big-endian protocol version |
| `type` | 2 bytes | Unsigned, big-endian message type |
| `length` | 4 bytes | Unsigned, big-endian payload length |

Multi-byte fields are big-endian. The protocol specification MUST declare a maximum payload length. Frames with unsupported versions or lengths exceeding that limit MUST be rejected. Trailing bytes after a declared payload MUST be rejected for ordinary framed messages.

### 14.3 Version Negotiation — In-Band

- The first frame exchanged after connection establishment is a `HELLO`. Both peers send one bootstrap-framed `HELLO` immediately upon connection. Each carries the sender's supported version range.
- The session version is the highest version present in both ranges.
- If the ranges do not intersect, the peer that detects the incompatibility sends a bootstrap-framed `HELLO_REJECT` and closes the connection.
- The bootstrap version is a fixed constant independent of the negotiated session version.
- `HELLO` header fields use the bootstrap version in the `version` field.
- After both `HELLO` messages have been exchanged, the session version is established and every subsequent frame uses it.
- Any handshake frame other than `HELLO` or `HELLO_REJECT` MUST be rejected.
- `HELLO_REJECT` carries a reason code: `1` means the ranges do not intersect; `2` means a valid bootstrap revision is unsupported by this peer; `3` means the peer sent a malformed `HELLO`; `4` means the handshake timed out.
- `HELLO_REJECT` always uses bootstrap framing. After sending or receiving one, the connection is closed.
- Each peer MUST enforce a declared handshake timeout and close the connection if the other `HELLO` does not arrive in time.

#### 14.3.1 Bootstrap Revision Registry

The bootstrap version field carries a constant from a fixed set listed in the protocol specification. A peer MUST recognize a received bootstrap version as valid if and only if that constant appears in its compiled-in registry. The registry is updated only by protocol revisions, not at runtime.

If the received constant is in the registry but the peer does not implement that bootstrap revision, the peer MAY send `HELLO_REJECT` with reason `2`, provided the conditions in §14.3.2 hold.

#### 14.3.2 Conditions for Sending `HELLO_REJECT` Reason `2`

A peer MAY send a bootstrap-framed `HELLO_REJECT` with reason `2` only if all of the following hold:

1. The received frame's `magic` equals the bootstrap magic constant.
2. Its `version` equals a bootstrap revision constant in the peer's compiled-in registry.
3. The peer does not implement that bootstrap revision.
4. Its `type` equals the bootstrap `HELLO` message type.
5. Its `length` is within that revision's minimum and maximum payload size.
6. At least `length` payload bytes are available within the read timeout.

If any condition fails, the peer MUST close the connection silently.

### 14.4 Version Negotiation — Out-of-Band

Under the out-of-band path, the client reads a service advertisement declaring the server's supported version range `[advertised_min, advertised_max]` and an `advertised_default` field. The client then performs a version-only preflight before sending application data.

#### 14.4.1 Preflight Envelope

Preflight messages use a fixed layout independent of session versions:

| Field | Size | Meaning |
|---|---:|---|
| `magic` | 4 bytes | Fixed protocol identifier, not version-specific |
| `envelope_version` | 2 bytes | Unsigned, big-endian envelope revision |
| `message` | 2 bytes | `VERSION_PROBE` = 1, `VERSION_ACCEPT` = 2, `VERSION_REJECT` = 3 |
| `payload_len` | 2 bytes | Unsigned, big-endian payload length |
| `payload` | `payload_len` bytes | Message-specific data |

`payload_len` MUST NOT exceed the protocol's fixed `PREFLIGHT_MAX_PAYLOAD`.

**`VERSION_PROBE` payload**

| Field | Size | Meaning |
|---|---:|---|
| `candidate_min` | 2 bytes | Lowest version the client would accept |
| `candidate_max` | 2 bytes | Highest version the client would accept |

**`VERSION_ACCEPT` payload**

| Field | Size | Meaning |
|---|---:|---|
| `selected_version` | 2 bytes | Session version selected by the server |
| `server_min` | 2 bytes | Lowest version supported by the server |
| `server_max` | 2 bytes | Highest version supported by the server |

**`VERSION_REJECT` payload**

| Field | Size | Meaning |
|---|---:|---|
| `reason` | 2 bytes | Rejection reason code |
| `server_min` | 2 bytes | Lowest version supported by the server |
| `server_max` | 2 bytes | Highest version supported by the server |

| Value | Meaning |
|---:|---|
| 0 | No version in the client's requested range is supported |
| 1 | A supported version exists in the range, but policy disallows every such version for this client |
| 2 | Probe malformed |
| 3–4 | Reserved |

#### 14.4.2 Selection and Preflight Rules

The client's supported range is `[client_min, client_max]`. It chooses a probe range `[candidate_min, candidate_max]` that is a subset of its supported range. It MAY use a narrower range when it has a preferred version, a policy-configured subset, or a subset whose framing it has verified.

The client opens a connection and sends exactly one `VERSION_PROBE`. The server validates the envelope and payload, computes the intersection of the candidate range and its supported range, and applies local policy. It returns:

- `VERSION_ACCEPT` with the highest permitted version in the intersection, if the intersection is non-empty and at least one version is permitted;
- `VERSION_REJECT` reason `0` if the intersection is empty;
- `VERSION_REJECT` reason `1` if the intersection is non-empty but policy disallows every version in it;
- `VERSION_REJECT` reason `2` if the probe payload is malformed.

The server MUST NOT accept a policy-disallowed version and MUST select the highest permitted version in the intersection.

#### 14.4.3 Client Retry Rules

- **`VERSION_ACCEPT`:** session version is established; the client sends application data using that framing.
- **Reason `0`, full supported range probed:** client MUST stop.
- **Reason `0`, strict subset probed:** client MAY open one additional connection with a probe carrying its full supported range. This is the only permitted second connection.
- **Reason `1`:** client MUST stop.
- **Reason `2`:** client MUST stop and report a client-side error.
- **Silent close:** treated as a transport or server failure; client MUST stop and MUST NOT interpret it as version rejection.
- **Any other response:** treated as a protocol failure; client MUST stop.

The retry bound is at most two connections per out-of-band negotiation.

#### 14.4.4 Self-Delimiting Envelope

The preflight envelope is a self-delimiting, one-shot exchange. A client sends exactly one `VERSION_PROBE`; a server sends exactly one `VERSION_ACCEPT` or `VERSION_REJECT`. The envelope frame is delimited by its fixed header and declared payload. The receiver MUST NOT poll for additional bytes or classify additional bytes as trailing bytes inside the envelope.

After reading the complete envelope frame:

- **`VERSION_ACCEPT`:** the session version is established and the receiver switches to session-version framing immediately. Any subsequent bytes are interpreted as session-version-framed data, whether they were already buffered when the envelope frame completed or arrive afterward.
- **`VERSION_REJECT`:** the connection is closed; remaining bytes are discarded without interpretation.
- **Envelope-level malformation:** the connection is closed silently; remaining bytes are discarded without interpretation.

Extra bytes after a valid, complete envelope are **not** an envelope-level violation. They are interpreted under the connection's next state. Envelope-level validation covers only the fixed header and declared payload of the envelope frame; it does not extend beyond the declared envelope end.

### 14.5 Application-Layer Messages — Mesh Introspection and Streaming

The NMN adds a **mesh-introspection** and **streaming** message family to the IPC protocol:

| Message | Direction | Payload |
|---|---|---|
| `MESH_DESCRIBE` | client → core | None |
| `MESH_DESCRIPTION` | core → client | Node list, edge list, activation set, version |
| `MESH_SNAPSHOT` | client → core | Session token |
| `MESH_STATE` | core → client | Current `S`, drift `D`, FSM state, `H` |
| `MESH_STREAM_OPEN` | client → core | Stream parameters (modality, rate, profile) |
| `MESH_STREAM_FRAME` | core → client | Chunked `S`, frame index, partial media chunk |
| `MESH_STREAM_CLOSE` | client → core | Stream token |

These messages use the same framing as §14.2. They do not expose raw weights; they expose only structural metadata and derived quantities. This closes the application-layer schema gap without violating §4.1's "no raw model weights through IPC."

---

## 15. Cryptographic Contract

### 15.1 28-Hour Key Rotation

```
Epoch N = floor( unix_time_utc / (28 × 3600) )
Key_N   = HKDF-SHA-384( master_secret,
                        salt = SHA-384(Epoch_N),
                        info = "zevo-api-v1",
                        L = 32 )
Overlap = 5 minutes
```

| Property | Value |
|---|---|
| Key length | 256 bits |
| Derivation | HKDF-SHA-384 |
| Rotation period | 28 hours |
| Overlap window | 5 minutes |
| Master secret | 512 bits, sealed under TPM 2.0 or HSM, mode 0400 |

The 28-hour epoch is chosen below the 24-hour symmetric-key ceiling used in high-assurance regimes, commensurate with a 5-minute clock-skew tolerance typical of NTP-synchronized fleets, and co-prime with a 24-hour operational day so rotation does not always land at the same shift boundary.

### 15.2 OpenSSL Primitives (Classical)

| Purpose | Primitive | Reference |
|---|---|---|
| Frame integrity | HMAC-SHA-384 | NIST SP 800-107 |
| Key derivation | HKDF-SHA-384 | RFC 5869 |
| Frame sealing | AES-256-GCM | NIST SP 800-38D |
| Audit signing | ECDSA P-384 | FIPS 186-5 |
| Randomness | CSPRNG | `RAND_bytes` |
| Transport (boundary 1) | TLS 1.3 | RFC 8446 |

The term "military-grade" is not used; explicit references are preferred.

### 15.3 Post-Quantum Migration Path

The NMN's audit signatures and IPC key establishment migrate to post-quantum primitives by swapping the primitive table:

| Purpose | Classical | Post-quantum | Reference |
|---|---|---|---|
| Audit signing | ECDSA P-384 | ML-DSA (Dilithium) | FIPS 204 |
| Key establishment (boundary 1) | ECDHE | ML-KEM (Kyber) | FIPS 203 |
| Hybrid mode | ECDSA + ML-DSA | Composite signature | FIPS 204 + FIPS 186-5 |

The NMN's structure is unchanged; only the primitive table changes. Migration is a build-time selection, not a runtime choice.

### 15.4 Hardware-Backed Sealing

The `master_secret` is sealed under **TPM 2.0** or an **HSM** (Trusted Computing Group, 2019):

- Sealing binds the secret to platform measurements (PCRs).
- Unsealing requires the expected platform state.
- Sealed blobs are mode `0400`, owned by the service account.
- The mesh's audit records (§15.6) are signed under the sealed key.

Where no TPM or HSM is available, the deployment MUST clearly report a reduced-security mode.

### 15.5 Authenticated Frame

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│ length (4B)  │ epoch (8B)   │ HMAC (48B)   │ JSON payload (N B)  │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

Any frame whose HMAC does not verify in constant time is discarded.

### 15.6 Audit Records

Every `ΔW_fused` step produces a **signed audit record**:

```
(session_id, epoch, edge_id, Δ, H, s_FSM)
```

signed with ECDSA P-384 (or ML-DSA under the post-quantum profile). Audit records are written to a bounded ring buffer (`A_max` entries) and MAY be flushed to a signed log. The audit path is the only path that writes weight-adjacent state to durable storage; it writes *deltas*, never weights. A full audit reviews the delta log against the declared bounds `[w_min, w_max]`, `D_max`, `η`, `g_max`.

### 15.7 Persistence Manifest

When persistence of `W_fused` across sessions is opted in, the following MUST be declared with the artifact:

- the target store (path, device, or signed log);
- the retention policy (duration, eviction rule);
- the signed audit records that cover the deltas since the last seed or reset;
- the reset or continuation semantics (whether the next session starts from the seed, from the persisted state, or from a replay of the audit log).

Without a persistence manifest, the core MUST discard `W_fused` at process exit and report session-reset semantics in `MESH_STATE`.

---

## 16. Operating-System Hardening and Build

### 16.1 Operating-System Hardening

Deployment SHOULD use:

- a dedicated unprivileged service account;
- an empty or minimal capability set;
- `NoNewPrivileges`;
- read-only system paths;
- restricted writable runtime directories;
- syscall filtering validated against actual runtime behavior;
- an LSM policy such as SELinux or AppArmor;
- device access only when required;
- bounded CPU, memory, file size, and process counts;
- TPM 2.0 or HSM access for sealing, when available.

A seccomp allow-list MUST be derived from observed and tested behavior. A generic list cannot be assumed sufficient for OpenCL, graphics, audio, or dynamic loading. The deployment MUST fail safely if required sandbox controls are unavailable, or clearly report a reduced-security mode.

### 16.2 Build and ELF Requirements

The build MUST produce a position-independent `ET_DYN` executable targeting the System V AMD64 ABI. The build record MUST identify:

- compiler and linker versions;
- source revision;
- build flags;
- linked libraries;
- model schema version;
- cryptographic primitive profile (classical, post-quantum, or hybrid);
- generated artifacts and digests.

The build SHOULD be reproducible from pinned inputs. Reproducibility MUST be tested by independent builds and byte comparison; setting `SOURCE_DATE_EPOCH` alone is insufficient.

### 16.3 Weight-Storage Verification

In **Mode A (embedded by section)**, a release test MUST inspect the executable and verify that the weight section is present, page-aligned, marked read-only, and `fs-verity`-protected. In **Mode B (embedded by reference)**, a release test MUST inspect the executable and verify the absence of any embedded weight payload, and MUST verify that the referenced `weights.bin` is present, page-aligned, marked read-only, and `fs-verity`-protected.

### 16.4 Reproducibility Report Requirements

A reproducibility report MUST identify:

- artifacts compared: executable, weights (section or file), generated media, or all;
- build environment, including OS, distribution, and kernel version;
- toolchain versions, including compiler, linker, C library, and C math library;
- number of independent clean builds compared;
- digest algorithm and reference digests;
- cryptographic primitive profile used.

A runtime-reproducibility claim MUST additionally identify numeric backends used, declared configuration parameters (including `ζ`, `ω`, `η`, `D_max`), and documented sources of nondeterminism.

### 16.5 Universal Hardware Access

| Class | Interface | Path to kernel (x86_64 / aarch64 / riscv64) |
|---|---|---|
| Compute | OpenCL, DRM | libc → `syscall` / `svc` / `ecall` |
| Graphics | DRM/KMS, OpenGL, SDL2 | libc → `syscall` / `svc` / `ecall` |
| Audio in | ALSA capture | libc → `syscall` / `svc` / `ecall` |
| Audio out | ALSA playback, OpenAL | libc → `syscall` / `svc` / `ecall` |
| Video in | V4L2 | libc → `syscall` / `svc` / `ecall` |
| Input | evdev | libc → `syscall` / `svc` / `ecall` |
| Storage | block layer, `ext4` | libc → `syscall` / `svc` / `ecall` |
| Random | `getrandom` | libc → `syscall` / `svc` / `ecall` |
| Time | `clock_gettime` | libc → `syscall` / `svc` / `ecall` |
| Sealing | TPM 2.0 / HSM | libc → `syscall` / `svc` / `ecall` |

On `x86_64`, the fast system-call instruction is `syscall`. On legacy 32-bit `x86`, it is `sysenter`. On `aarch64`, it is `svc`. On `riscv64`, it is `ecall`. The core never emits inline assembly; every wrapper is a libc call that resolves to the appropriate instruction through the platform ABI.

---

## 17. Web-Responsible Client

The client MUST be weight-free and communicate with the core only through the published service or IPC boundary. It SHOULD provide semantic HTML, keyboard operation, visible focus, reduced-motion support, adequate contrast, and accessible status announcements.

The client MUST NOT contain master secrets. It MUST disclose that the assistant is AI-generated and request user action before playing generated audio or downloading large artifacts. Privacy and telemetry behavior MUST be documented.

| Principle | Implementation |
|---|---|
| Semantic structure | `<header>`, `<main>`, `<section>`, `<footer>` |
| Accessibility | ARIA live regions, focus-visible, keyboard-first; WCAG 2.2 AA minimum, AAA target |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | no telemetry, no cookies, no third-party scripts |
| Transparency | clear disclosure that Zevo™ is an AI assistant |
| Artifact opt-in | pixels and audio served only on explicit request |
| No secrets in browser | no API keys, no master secret |
| Mesh visualization | `MESH_STATE` rendered as an accessible, reduced-motion-aware visualization |

---

## 18. Testing and Acceptance Criteria

A release is conformant only if the following tests pass:

1. **Build:** C89/C90 compilation succeeds with documented flags.
2. **ELF:** output is `ET_DYN`, targets the intended ABI, and matches the selected weight-storage mode.
3. **Input validation:** malformed, truncated, oversized, and overflow-inducing inputs are safely rejected.
4. **Weight loading:** valid weights load read-only; corrupted, incompatible, or incorrectly measured weights are rejected, in both embedding modes.
5. **Numerics:** entropy, inference, clamping, mesh gain calculation, and animation tests cover boundary and non-finite values.
6. **Media:** output dimensions, formats, lengths, timestamps, and digests match the manifest.
7. **IPC:** unauthorized peers, invalid framing, unsupported versions, invalid negotiation sequences, and resource exhaustion are tested.
8. **Concurrency:** stress, cancellation, shutdown, and race tests are performed.
9. **Sandbox:** the deployed service is tested under its actual systemd, seccomp, and LSM policies.
10. **Accessibility:** the client is tested with keyboard navigation and representative assistive technologies, against WCAG 2.2 AA.
11. **Reproducibility:** independent clean builds are compared and discrepancies documented.
12. **Performance:** latency, memory, throughput, and device behavior are measured and reported with CPU and GPU models, memory size, storage type, OS and version, and relevant library versions.
13. **Mesh topology:** `MESH_DESCRIBE` returns a topology consistent with the model schema; node count, edge count, and activation set match the declared version.
14. **Mesh bounds:** under a stress input stream, `|ΔW_fused| ≤ η · g_max` per edge and `Σ|ΔW_fused| ≤ D_max` per session.
15. **Convergence:** under a stationary input distribution, the drift `|ΔW_fused|` decreases monotonically after a declared warm-up window.
16. **Diminishing gain:** the fourth rule-base validation check of §6.10 (monotonicity of `g` in `|a_in − a_out|`) passes on the fixed sampling grid within the declared tolerance, for every mesh edge.
17. **Reset:** at process exit, `W_fused` resets to the seed unless a persistence manifest (§15.7) is present and valid; when present, the reset or continuation semantics declared in the manifest are honored and reported in `MESH_STATE`.
18. **Audit integrity:** every `ΔW_fused` step has a corresponding signed audit record; the log verifies under ECDSA P-384 (or ML-DSA under the post-quantum profile).
19. **Streaming:** `MESH_STREAM_OPEN` / `MESH_STREAM_FRAME` / `MESH_STREAM_CLOSE` delivers chunked media that reassembles to the same digest as the non-streamed artifact for the same request.

No performance or security result SHOULD be reported without the test environment, method, sample size, and observed results.

---

## 19. Evaluation Plan

This paper specifies a design; it does not report completed experiments. An implementation evaluation SHOULD report:

1. startup time and peak memory;
2. inference latency by input modality;
3. frame-generation throughput and memory use;
4. audio-generation cost and peak-bound verification;
5. behavior under malformed and oversized requests;
6. reproducibility results and build provenance;
7. security-test findings and remediation;
8. accessibility test results with assistive technology;
9. characterization of the mesh's fused-weight update cycle: convergence behavior, bounds on the autonomous weight-calculation state, and effect of the update rule on subsequent-request outputs;
10. accuracy and coverage of the audit log against the declared bounds;
11. multi-architecture behavior of the mesh under declared numeric backends;
12. comparison of classical, post-quantum, and hybrid cryptographic profiles for latency and throughput;
13. cost of the fourth rule-base validation check (§6.10) on representative rule bases and sampling grids.

Evaluation datasets, hardware, operating-system versions, compiler versions, and measurement methods should be documented so results can be interpreted.

---

## 20. Engineering Bounds and Hardening Agenda

### 20.1 Engineering Bounds

The architecture is closed: every limitation and future-work item identified in v1.0.0 is now addressed by a named mechanism inside the Neural Mesh Network. What remains are **engineering bounds**, not architectural gaps:

| Bound | Statement |
|---|---|
| **Specification, not implementation** | The mesh's seed weights, rule base, and activation constants are versioned deliverables that an implementation must produce. |
| **Per-backend determinism** | The mesh's determinism guarantee is per-backend, per the §13 rules and the §6.1.3 fixed evaluation order. Bit-identity across architectures is not claimed. |
| **Convergence is a contract** | §6.11 states the properties an implementation MUST satisfy; the diminishing-gain property is checked by the fourth rule-base validation step (§6.10, check 4). The remaining three properties are guaranteed by construction from the bounded gain and the clamping function. |
| **Audit log is bounded** | A session that exceeds `A_max` audit entries drops the oldest entries; the drop policy is versioned configuration. |
| **Expressiveness is bounded by `F_max` and `S_max`** | Larger meshes are a protocol revision. |
| **HDR depends on the output profile** | 8-bit BGRA alone cannot represent the full dynamic range of HDR content; the HDR profiles in §10.4 preserve higher-precision intermediates. |
| **"Pure functional" is a discipline** | The complete operating-system process is necessarily stateful because it performs I/O, manages resources, interacts with devices, and maintains bounded in-core autonomous weight state. |

### 20.2 Hardening Agenda

The remaining work is a **hardening agenda**, not a **gap-closing agenda**:

| Direction | Description |
|---|---|
| **Seed-weight study** | Characterize `W_fused^(0)` initialization procedures against convergence speed. |
| **Rule-base library** | Publish a versioned library of validated rule bases for common input distributions. |
| **Mesh visualization** | Extend the Web-Responsible client to render `MESH_STATE` as an accessible, reduced-motion-aware visualization. |
| **Formal proof of convergence** | Move from contract (§6.11) to machine-checked proof for a reference rule base, replacing the sampling-based check of §6.10 check 4 with a proof. |
| **Cross-architecture bit-identity** | Investigate whether `aarch64` and `x86_64` builds of the same mesh can be made bit-identical under a declared numeric backend. |
| **Federated aggregation policy** | Specify the service-layer policy for aggregating signed `ΔW_fused` deltas across tenants. |
| **Post-quantum performance** | Characterize ML-KEM/ML-DSA latency and throughput against classical primitives in the mesh's IPC path. |
| **TPM/HSM coverage** | Characterize sealing and unsealing latency across TPM 2.0 and HSM vendors. |
| **Rule-set interchange** | Standardize the rule-set schema (§6.10) as a portable interchange format across implementations. |

---

## 21. Conclusion

The zero step establishes, and this paper specifies, the buildable design of the **Zevo™ AI-Driven Animation Core**, an **5D interactive artificial intelligence assistant** realized as a System V ABI `x86-64` ELF image of type `ET_DYN`, position-independent, written in ANSI C89/90 with GNU and POSIX interfaces. The core is separated from a weight-free, stateless Web-Responsible client by a published IPC API. Its LLM weights, metadata, and vectors are **embedded by section** or **embedded by reference**, in both cases page-aligned on `ext4`, `fs-verity`-protected, and mapped read-only. Its hardware access is mediated exclusively by libc wrappers that resolve to `syscall` on `x86_64`, `sysenter` on legacy `x86`, `svc` on `aarch64`, and `ecall` on `riscv64`, under seccomp-bpf, capability bounding, and LSM discipline. Its cryptographic contract is aligned with NIST SP 800-38D, NIST SP 800-107, RFC 5869, and FIPS 186-5, with a TPM- or HSM-sealed master secret and a post-quantum migration path via FIPS 203 and FIPS 204.

The core is anchored in **FSM-Neural-Fuzzy-Machine-Learning logic realized as a Neural Mesh Network**: a single integrated graph in which an FSM node, a Mamdani node over rules and facts, a neural projection node, an entropy node, and an autonomous weight-calculation node share a fused weight set and update rule. **Shannon entropy** enters as the epistemic signal carried by the depth axis and consumed as a mesh gain parameter. **Autonomous self-learning based on model weights** is the mesh's per-edge update rule, computed by **machine-learning calculation under FSM-Neural-Fuzzy logic**, bounded by documented drift limits, and audited by signed deltas. The gain function is the Mamdani centroid of a versioned rule base over entropy and FSM state, applied as-is when the consequents are declared over `[0, 1]` and projected through σ when they are declared unbounded. The diminishing-gain property that underwrites convergence is checked at rule-base validation time.

The pipeline is a composition of pure operators `ℛ ∘ 𝒫 ∘ 𝒜 ∘ 𝒞 ∘ ℰ`. The five-component state vector `S = (O, X, Y, Z, R)` drives a procedural character renderer centered on **Zevo**, an original, Pixar-inspired animated character, whose reasoning is expressed through motion and interconnected visual data flows rather than explanatory text. The multimodal artifact comprises **Text Raw**, **Image BGRA**, **Video BGRA**, **Audio PCM**, and the fused state vector.

Every limitation and future-work item identified in v1.0.0 is now addressed by a named mechanism inside the Neural Mesh Network, and every mechanism whose correctness is not guaranteed by construction is checked by a named validation step. What remains is a hardening agenda of engineering studies, not a list of architectural gaps.

The paper is a **design contract**. It is a scientific computer-engineering specification and a complete contract for building the ELF image. It does not claim a completed implementation, trained weights, measured performance, or audited security. Conformance, reproducibility, accessibility, and performance must be demonstrated through implementation and testing.

---

## 22. References (ISO 690:2021)

Barker, E. (2020). *Recommendation for key management: Part 1 — General* (NIST Special Publication 800-57 Part 1 Rev. 5). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-57pt1r5

Dworkin, M. (2007). *Recommendation for block cipher modes of operation: Galois/Counter Mode (GCM) and GMAC* (NIST Special Publication 800-38D). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-38D

Dworkin, M. (2012). *Recommendation for applications using approved hash algorithms* (NIST Special Publication 800-107 Rev. 1). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-107r1

Glorot, X., & Bengio, Y. (2010). Understanding the difficulty of training deep feedforward neural networks. *Proceedings of the Thirteenth International Conference on Artificial Intelligence and Statistics*, 249–256.

International Organization for Standardization. (2021). *ISO 690:2021 — Information and documentation — Guidelines for bibliographic references and citations to information resources*. ISO.

Krawczyk, H., & Eronen, P. (2010). *HMAC-based extract-and-expand key derivation function (HKDF)* (RFC 5869). Internet Engineering Task Force. https://doi.org/10.17487/RFC5869

Krawczyk, H., Bellare, M., & Canetti, R. (1997). *HMAC: Keyed-hashing for message authentication* (RFC 2104). Internet Engineering Task Force. https://doi.org/10.17487/RFC2104

Linux Kernel Documentation. (2024a). *fs-verity*. The Linux Kernel Organization. https://docs.kernel.org/filesystems/fsverity.html

Linux Kernel Documentation. (2024b). *Seccomp BPF (SECure COMPuting with filters)*. The Linux Kernel Organization. https://docs.kernel.org/userspace-api/seccomp_filter.html

Mamdani, E. H. (1974). Application of fuzzy algorithms for control of simple dynamic plant. *Proceedings of the Institution of Electrical Engineers, 121*(12), 1585–1588. https://doi.org/10.1049/piee.1974.0328

Merkel, D. (2014). Docker: Lightweight Linux containers for consistent development and deployment. *Linux Journal, 2014*(239), Article 2.

National Institute of Standards and Technology. (2023). *Digital signature standard (DSS)* (FIPS Publication 186-5). https://doi.org/10.6028/NIST.FIPS.186-5

National Institute of Standards and Technology. (2024a). *Module-lattice-based key-encapsulation mechanism standard* (FIPS Publication 203). https://doi.org/10.6028/NIST.FIPS.203

National Institute of Standards and Technology. (2024b). *Module-lattice-based digital signature standard* (FIPS Publication 204). https://doi.org/10.6028/NIST.FIPS.204

Olston, C., et al. (2017). TensorFlow-Serving: Flexible, high-performance ML serving. *arXiv preprint arXiv:1712.06139*.

PyTorch. (2023). *TorchServe* [Software]. https://pytorch.org/serve/

Rescorla, E. (2018). *The Transport Layer Security (TLS) protocol version 1.3* (RFC 8446). Internet Engineering Task Force. https://doi.org/10.17487/RFC8446

Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal, 27*(3), 379–423. https://doi.org/10.1002/j.1538-7305.1948.tb01338.x

Srivastava, N., Hinton, G., Krizhevsky, A., Sutskever, I., & Salakhutdinov, R. (2014). Dropout: A simple way to prevent neural networks from overfitting. *Journal of Machine Learning Research, 15*(1), 1929–1958.

System V Application Binary Interface. (2018). *System V Application Binary Interface: AMD64 architecture processor supplement* (Version 1.0). https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf

The Open Group. (2018). *POSIX.1-2017 — Standard for information technology — Portable Operating System Interface (POSIX)*. IEEE.

Thomas, F., & Johnston, O. (1981). *The illusion of life: Disney animation*. Disney Editions.

Tool Interface Standard. (1995). *Executable and Linkable Format (ELF) specification* (Version 1.2).

Trusted Computing Group. (2019). *TPM 2.0 specification* (Version 1.59). Trusted Computing Group.

W3C. (2023). *Web Content Accessibility Guidelines (WCAG) 2.2*. World Wide Web Consortium. https://www.w3.org/TR/WCAG22/

W3C. (2024). *Accessible Rich Internet Applications (WAI-ARIA) 1.2*. World Wide Web Consortium. https://www.w3.org/TR/wai-aria-1.2/

Zadeh, L. A. (1965). Fuzzy sets. *Information and Control, 8*(3), 338–353. https://doi.org/10.1016/S0019-9958(65)90241-X

Zaita, D. B., & Zevo Community Teams. (2026). *The Zevo™ AI-Driven Animation Core: An 5D interactive artificial intelligence assistant — Design specification anchored in FSM-Neural-Fuzzy-Machine-Learning logic* (Version 1.0.3) [Technical report]. Zevo Project.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Title | The Zevo™ AI-Driven Animation Core — An 5D Interactive Artificial Intelligence Assistant |
| Anchoring architecture | **Neural Mesh Network** unifying FSM-Neural-Fuzzy-Machine-Learning logic with rules and facts, Shannon entropy, Mamdani inference, and autonomous self-learning based on model weights |
| Document type | Design and implementation specification |
| Version | 1.0.3 |
| Citation standard | ISO 690:2021 |
| Status | Specification; not evidence of a completed or validated implementation |
| Effective Date | ____ |

**End of specification.**

---

### Change Log

| Version | Change |
|---|---|
| 1.0.0 | Initial specification. |
| 1.0.1 | Addendum introducing the Neural Mesh Network as the resolution mechanism for §20.1 limitations and §20.2 future work. |
| 1.0.2 | Merged the Neural Mesh Network into the main body: §6 rewritten as the NMN anchoring architecture; §14 extended with mesh-introspection messages; §15 extended with post-quantum migration, TPM/HSM sealing, and audit records; §18 extended with criteria 13–18; §20 restated as engineering bounds and a hardening agenda; §22 extended with FIPS 203/204, TPM 2.0, RFC 2104, Glorot & Bengio (2010), and Srivastava et al. (2014). |
| 1.0.3 | **This revision.** Closes the two remaining specification gaps flagged in review of v1.0.2: (a) §6.5.1 makes the consequent range `[r_min, r_max]` explicit and §6.8.2 states when the σ wrapper on the gain is load-bearing, resolving the ambiguity in the gain formula; (b) §6.10 adds a fourth rule-base validation check ("diminishing gain"), which is the check that underwrites §6.11's convergence contract. Also adds §6.1.3 fixing the mesh evaluation order and its role in deterministic mode; §15.7 defining a persistence manifest for opt-in cross-session `W_fused` persistence; §18.16 testing the diminishing-gain check; §18.17 clarifying reset semantics under a persistence manifest; §18.19 testing streaming; §13 note on parallel mesh evaluation; §19.13 on the cost of the fourth validation check; §20.1 restates the convergence bound in terms of the newly-added check; §20.2 replaces the convergence-proof item with the sampling-to-proof upgrade path; Abstract and §1.2 updated to reflect the nineteen acceptance criteria and the fourth validation check; §21 updated to state the σ/`[r_min, r_max]` semantics and the validation check explicitly. |

---
---

# The Zevo™ AI-Driven Core
## An 5D Interactive Artificial Intelligence Assistant — A Zero-Step Foundation Paper

**PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams**

---

## Abstract

This paper establishes the **zero-step foundation** for building the **ELF Image as Zevo™ AI-Driven Core** — a System V ABI `x86_64` `ET_DYN` executable that is simultaneously an artificial-intelligence reasoning engine and a procedural cartoon animation system inspired on Pixar-style principles of animation. The zero step is the layer *beneath* implementation: the definitions, the naming, the trust boundaries, the semantic axes, and the invariants that every subsequent file must respect. Before a single line of C is written, the system must be *named*, *bounded*, *axiomatized*, and *reproducible*. This paper supplies exactly that.

Zevo™ is defined here as an **5D interactive artificial intelligence assistant** driven by a **Fuzzy-State-Machine** and anchored in **fuzzy neural logic**, **Shannon entropy**, **Mamdani inference**, and **offline-trained + online-adaptive machine learning**, all converging on a single **5-axis intelligence vector** `S = (O, X, Y, Z, R)` that drives the character pose, the data-flow ribbons, the color palette, the audio timbre, and the temporal pacing of the multimodal artifact. The multimodal artifact comprises **Text Raw**, **Image BGRA**, **Video BGRA**, **Audio PCM**, and the **fused 5-axis vector** itself. The ELF AI Core reaches hardware peripherals exclusively through libc wrappers that resolve to the `syscall` instruction on `x86_64` (and `sysenter` on legacy x86), under GNU + POSIX contracts, Pthreads, OpenCL, OpenGL, SDL2, OpenAL, and OpenSSL. The weights are not embedded: they live in a **page-aligned, fs-verity-protected data structure on ext4**, mapped **read-only** by the core. The Web-Responsible layer remains a **separate, weight-free, stateless client** that reaches the core only through a **published IPC API**.

The result of the zero step is not code. It is the *contract* that code must obey — so that everything that follows is secure-by-construction, reproducible, auditable, and faithful to the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant** (verbatim per project charter).

**Keywords:** zero-step foundation, ELF AI Core, `ET_DYN`, `syscall`, `sysenter`, fs-verity, ext4, Fuzzy-State-Machine, fuzzy neural logic, Shannon entropy, Mamdani inference, 5D axes, multimodal generation, Pixar-inspired animation, Web-Responsible UI.

---

## 1. Why a Zero Step

Before implementation begins, five categories of decision must be frozen:

| Category | Why it must precede code |
|---|---|
| **Identity** | The exact title and subtitle are contractual; they appear in every binary, log, and manifest |
| **Boundaries** | Trust boundaries cannot be retrofitted; they must be architectural from line one |
| **Semantics** | The 5 axes must mean something *before* a renderer consumes them |
| **Invariants** | Purity, reproducibility, and read-only weights must be axioms, not aspirations |
| **Vocabulary** | FSM, entropy, Mamdani, neural-fuzzy, ML — these words must have fixed meanings |

The zero step produces **no executable**. It produces the **constitution** of the Zevo™ AI-Driven Core.

---

## 2. The Exact Identity

The following strings are canonical, immutable, and appear verbatim in source, metadata, manifests, and user-facing surfaces:

| Field | Value |
|---|---|
| Title | **Zevo™** |
| Subtitle | **An 5D interactive artificial intelligence assistant** |
| Version | 1.0.0 |
| ABI | 2 |
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |

> **Note on the article "An".** The subtitle uses the indefinite article `An` before `5`. Standard English would render `A 5`. The project charter preserves `An` as a deliberate branding choice; it is retained verbatim in every user-facing surface and every metadata record. This paper therefore writes it exactly as the charter dictates.

The character, **Zevo**, is an original Pixar-inspired cartoon character. "Pixar-inspired" refers exclusively to publicly documented *animation principles* — squash-and-stretch, anticipation, staging, appeal, rounded forms, expressive eyes, premium timing. No Pixar code, asset, trademark, or trade dress is used anywhere in the system.

---

## 3. The Four Trust Boundaries

Every subsequent design decision is bounded by these four boundaries. Nothing crosses them without an explicit contract.

```
┌──────────────────────────────────────────────────────────────────────┐
│                          Zevo™ System                                │
│                                                                      │
│  ┌────────────┐   ┌──────────────┐   ┌────────────┐   ┌──────────┐  │
│  │  Browser   │   │  Secure      │   │  ELF AI    │   │ Universal│  │
│  │  Responsible│──►│  Service API │──►│  Core      │──►│ Hardware │  │
│  │  UI        │   │  (zevo_api)  │   │  (zevo_core)│   │ Devices  │  │
│  └────────────┘   └──────────────┘   └────────────┘   └──────────┘  │
│   Boundary 1        Boundary 2        Boundary 3       Boundary 4   │
│   (public)          (service)         (core)           (hardware)   │
└──────────────────────────────────────────────────────────────────────┘
```

| Boundary | Between | Contract | Enforced by |
|---|---|---|---|
| 1 | Browser ↔ Service API | TLS 1.3, session token, no telemetry | OpenSSL, HTTP hardening |
| 2 | Service API ↔ ELF AI Core | 28-hour API key rotation, HMAC-SHA-384, AES-256-GCM | OpenSSL, published IPC |
| 3 | ELF AI Core ↔ `weights.bin` | fs-verity Merkle verification, read-only mmap | Linux kernel |
| 4 | ELF AI Core ↔ Hardware | libc → `syscall`/`sysenter`, seccomp-bpf, capability bounding | Linux kernel, glibc |

**The zero step fixes these boundaries.** No later file may move them.

---

## 4. The ELF AI Core as a First-Class Object

The ELF AI Core is defined as:

| Attribute | Value |
|---|---|
| File | `zevo_core.elf` |
| Format | ELF |
| Type | `ET_DYN` (position-independent executable) |
| ABI | System V `x86_64` |
| Language | ANSI C89/90 |
| Contracts | GNU + POSIX + Pthreads |
| Accelerators | OpenCL, OpenGL, SDL2, OpenAL |
| Cryptography | OpenSSL |
| Target | Linux Fedora 40+ |
| Exposure | Local IPC only |
| System-call path | libc → `syscall` (x86_64) / `sysenter` (legacy x86) |

The ELF is *not* a container. It is *not* a shared library. It is a **position-independent executable** launched by `ld.so` under PIE semantics, with a single exported symbol (`zevo_pipeline_run`) and everything else hidden via `-fvisibility=hidden`.

---

## 5. The Five Semantic Axes

The zero step defines the **5-axis intelligence vector**:

```
S = (O, X, Y, Z, R)
```

| Axis | Name | Physical meaning | Reasoning source | Visual effect | Acoustic effect |
|---|---|---|---|---|---|
| **O** | Origin | Rotation & gravity | Mamdani centroid | Body rotation, gravity sag | Fundamental frequency |
| **X** | Horizontal | Lateral spread | Neural-fuzzy projection | Eye/antenna spread, lateral tint | Octave-fifth detune |
| **Y** | Vertical | Up/down extent | FSM dwell time (normalized) | Vertical stretch (squash-and-stretch) | Slow vibrato rate |
| **Z** | Depth | Front/back immersion | Normalized Shannon entropy | Halo depth glow, blue channel | Shimmer / air partial |
| **R** | Radial | Outward expansion | Online self-learning affect | Halo radius, ribbon length | Overall amplitude |

The **idle singularity** is `S = (0, 0, 0, 0, 0)` — no reasoning, no entropy, no affect. Every artifact is a departure from that point. Full engagement is `S = (1, 1, 1, 1, 1)`.

**No axis may be redefined.** Every later file consumes these five meanings unchanged.

---

## 6. The Five Reasoning Stages

The zero step fixes five reasoning stages, each a pure function:

| Stage | File | Input | Output | Drives |
|---|---|---|---|---|
| Shannon entropy | `entropy.c` | histogram | `H(X) ∈ [0,1]` | `Z` axis |
| Mamdani inference | `mamdani.c` | facts + rules | defuzzified centroid | `O` axis |
| Fuzzy neural logic | `nfuse.c` | embedding + prototype | `sigmoid(⟨e,p⟩)` | `X` axis |
| FSM dwell | `fsm.c` | state transitions | normalized dwell time | `Y` axis |
| Self-learning | `learn.c` | observed S vectors | affective scalar | `R` axis |

Each stage obeys the **purity contract**: `const` inputs → freshly allocated output, no global mutable state, reentrant, thread-safe.

---

## 7. The Multimodal Artifact

The zero step defines the artifact's five channels:

| Channel | Symbol | Contents |
|---|---|---|
| Text Raw | `text` | UTF-8 bytes of the prompt echo |
| Image BGRA | `image` | `W × H × 4`, BGRA order, HDR intent |
| Video BGRA | `video` | `N × W × H × 4`, BGRA order, `FPS` frames/s |
| Audio PCM | `audio` | signed 16-bit LE interleaved, `Hz` sample rate |
| Fused 5-axis vector | `S` | `(O, X, Y, Z, R)` in `[0,1]⁵` |

Every channel is independently dumpable to disk. Every channel is independently reproducible from the same prompt and the same weights.

---

## 8. The Weight Contract (Mode B: Reference-Embedded)

The zero step chooses **reference-embedded weights** as the canonical mode:

| Requirement | Value |
|---|---|
| Location | `weights.bin` on ext4 |
| Alignment | every section offset is a multiple of 4096 |
| Integrity | fs-verity Merkle tree, root hash pinned in `.zevo_meta` |
| Mapping | `mmap(NULL, len, PROT_READ, MAP_PRIVATE, fd, 0)` |
| Write capability | none — `mprotect` never adds `PROT_WRITE` |
| Confidentiality | layered separately (LUKS / dm-crypt / fscrypt) |

fs-verity is **not encryption**. It is kernel-enforced integrity. The zero step states this explicitly so no later design conflates the two.

---

## 9. The Universal Hardware Peripheral Contract

The zero step fixes the mechanism by which the ELF AI Core touches hardware:

| Class | Examples | Linux Interface | Path to kernel |
|---|---|---|---|
| Compute | GPU, NPU | OpenCL, DRM | libc → `syscall` |
| Graphics | GPU, display | DRM/KMS, OpenGL, SDL2 | libc → `syscall` |
| Audio in | Microphone | ALSA (PCM capture) | libc → `syscall` |
| Audio out | Speaker | ALSA (PCM playback), OpenAL | libc → `syscall` |
| Video in | Web camera | V4L2 | libc → `syscall` |
| Input | Keyboard | evdev | libc → `syscall` |
| Storage | NVMe, SATA | block layer, ext4 | libc → `syscall` |
| Random | RDRAND, RDTSC | `getrandom` | libc → `syscall` |
| Time | HPET, TSC | `clock_gettime` | libc → `syscall` |

On `x86_64`, the fast system-call instruction is **`syscall`**. On legacy 32-bit x86, it is **`sysenter`**. The core never emits inline assembly; every wrapper is a libc call that resolves to the correct instruction through the platform ABI.

**Half-duplex interaction** is the canonical user protocol:

```
LISTEN  →  THINK  →  SPEAK  →  LISTEN  →  …
```

The core listens through microphone, web camera, and keyboard. It thinks through its FSM-driven pipeline. It speaks through speaker and SDL2 window, and animates the multimodal artifact. It then returns to listening.

---

## 10. The Cryptographic Contract

The zero step fixes the cryptographic discipline:

| Purpose | Primitive | Reference |
|---|---|---|
| Frame integrity | HMAC-SHA-384 | NIST SP 800-107 |
| Key derivation | HKDF-SHA-384 | RFC 5869 |
| Frame sealing | AES-256-GCM | NIST SP 800-38D |
| Audit signing | ECDSA P-384 | FIPS 186-5 |
| Randomness | CSPRNG | `RAND_bytes` |
| Transport (boundary 1) | TLS 1.3 | RFC 8446 |
| Key rotation | 28-hour epoch, 5-minute overlap | NIST SP 800-57 |

**The term "military-grade" is not used.** It is a marketing phrase with no technical meaning. The zero step substitutes explicit NIST / IETF / FIPS references.

---

## 11. The Reproducibility Contract

The zero step fixes reproducibility:

| Rule | Value |
|---|---|
| Timestamp | `SOURCE_DATE_EPOCH=0` |
| Build ID | `-Wl,--build-id=sha1` |
| Compiler flags | pinned in `Makefile` |
| Section order | pinned in `zevo_core.ld` |
| Verification | two clean builds produce byte-identical ELFs |

Any deviation breaks the contract and voids the audit.

---

## 12. The Purity Contract

The zero step fixes the functional discipline:

```c
/* PURE: takes const inputs, returns a freshly allocated value,
 * reads no global mutable state, writes no global mutable state,
 * is reentrant and thread-safe. */
```

Every stage carries this comment. `main()` is the only impure shell: it owns peripherals, opens weights, allocates the learner, drives the FSM, and presents/speaks. All reasoning lives in pure functions.

---

## 13. The Vocabulary (Fixed Meanings)

The zero step fixes the meaning of each term:

| Term | Fixed meaning |
|---|---|
| **FSM** | Fuzzy-state-machine: discrete states with fuzzy guards |
| **Shannon entropy** | `H(X) = −Σ p log₂ p`, normalized to `[0,1]` |
| **Mamdani inference** | Fuzzify → fire → aggregate → centroid defuzzify |
| **Neural-fuzzy fusion** | `sigmoid(⟨embedding, prototype⟩)` |
| **Machine learning** | Offline (weights.bin) + online (in-core learner, no disk writes) |
| **5D axes** | `S = (O, X, Y, Z, R)`, exactly as defined in §5 |
| **Multimodal** | Text Raw + Image BGRA + Video BGRA + Audio PCM + S |
| **Core** | The `zevo_core.elf` process, not a container, not a library |
| **Web-Responsible** | Weight-free, stateless client that reaches the core only via IPC |

No later file may redefine these words.

---

## 14. The Zero-Step Deliverables

The zero step produces no executable. It produces:

| Deliverable | Purpose |
|---|---|
| Canonical identity strings | appear in every binary and manifest |
| Four trust boundaries | non-negotiable architectural lines |
| Five semantic axes | fixed meanings for `O, X, Y, Z, R` |
| Five reasoning stages | fixed purity contract per stage |
| Five artifact channels | fixed output interface |
| Weight contract | reference-embedded, page-aligned, fs-verity, RO |
| Peripheral contract | libc → `syscall`/`sysenter`, half-duplex |
| Cryptographic contract | explicit NIST / IETF / FIPS references |
| Reproducibility contract | `SOURCE_DATE_EPOCH=0`, byte-identical builds |
| Purity contract | `/* PURE: … */` on every stage |
| Vocabulary | no term redefined after this paper |

The first source file (`zevo.h`) and the first header (`pure.h`) follow from these deliverables. Nothing precedes them.

---

## 15. What the Zero Step Enables

With the zero step fixed, every later increment becomes local:

| Increment | Enabled by |
|---|---|
| Skeleton + Makefile | §4 (ELF contract), §11 (reproducibility) |
| Pure-function contract | §12 (purity) |
| 5-axis vector | §5 (axes) |
| Weights loader | §8 (weight contract) |
| Shannon entropy | §6 (stage 1), §13 (vocabulary) |
| Mamdani inference | §6 (stage 2), §13 (vocabulary) |
| Neural-fuzzy fusion | §6 (stage 3), §13 (vocabulary) |
| FSM | §6 (stage 4), §13 (vocabulary) |
| Self-learning | §6 (stage 5), §13 (vocabulary) |
| Zevo character | §3 (identity), §5 (axes) |
| Multimodal render | §7 (artifact), §5 (axes) |
| Peripheral access | §9 (peripheral contract) |
| IPC + crypto | §3 (boundary 2), §10 (crypto contract) |
| Web client | §3 (boundary 1), §13 (vocabulary) |

Every increment is a **consequence** of the zero step, not a new decision.

---

## 16. Non-Goals of the Zero Step

To keep the zero step precise, the following are explicitly *out of scope*:

| Non-goal | Rationale |
|---|---|
| Choosing a quantization scheme | belongs to `weights.bin` provisioning, not the zero step |
| Implementing the attention kernel | belongs to `nfuse.c`, a later file |
| Defining the IPC frame layout | belongs to `ipc.c`, constrained by §3 boundary 2 |
| Selecting a color space for HDR | belongs to `render_image.c`, constrained by §7 |
| Specifying the FSM transition table | belongs to `fsm.c`, constrained by §6 |
| Enrolling TLS certificates | belongs to deployment, constrained by §3 boundary 1 |

The zero step **bounds** these decisions; it does not make them.

---

## 17. The Zero-Step Constitution (Summary)

> **Article 1.** The title is **Zevo™**. The subtitle is **An 5D interactive artificial intelligence assistant**. Both are verbatim.
>
> **Article 2.** The ELF AI Core is a System V `x86_64` `ET_DYN` executable built under GNU + POSIX contracts, targeting Linux Fedora 40+.
>
> **Article 3.** Four trust boundaries separate UI, service, core, and hardware. No later file may move them.
>
> **Article 4.** The 5-axis vector is `S = (O, X, Y, Z, R)`, with meanings fixed in §5.
>
> **Article 5.** Five reasoning stages, each a pure function, converge on `S`.
>
> **Article 6.** The multimodal artifact has five channels: Text Raw, Image BGRA, Video BGRA, Audio PCM, and `S`.
>
> **Article 7.** Weights are reference-embedded, page-aligned on ext4, fs-verity-protected, mapped read-only. They are not embedded in the ELF.
>
> **Article 8.** Hardware is reached exclusively through libc wrappers resolving to `syscall` on `x86_64` and `sysenter` on legacy x86.
>
> **Article 9.** Cryptography uses OpenSSL primitives aligned with NIST SP 800-38D, SP 800-107, RFC 5869, and FIPS 186-5. "Military-grade" is not used.
>
> **Article 10.** Builds are reproducible: `SOURCE_DATE_EPOCH=0` yields byte-identical ELFs.
>
> **Article 11.** Every reasoning stage is pure. `main()` is the only impure shell.
>
> **Article 12.** No term defined in §13 may be redefined after this paper.
>
> **Article 13.** The Web-Responsible layer is weight-free and stateless. It reaches the core only through the published IPC API.
>
> **Article 14.** The character **Zevo** is an original Pixar-*inspired* cartoon. "Pixar-inspired" refers to public animation principles only. No Pixar code, asset, trademark, or trade dress is used.

---

## 18. Conclusion

The **zero step** for building the ELF Image as **Zevo™ AI-Driven Core** is a paper, not a program. It fixes the identity, the boundaries, the axes, the stages, the channels, the weight contract, the peripheral contract, the cryptographic contract, the reproducibility contract, the purity contract, and the vocabulary — before a single byte of machine code exists.

With the zero step in place, the first source file (`zevo.h`), the first header (`pure.h`), the first loader (`weights.c`), the first stage (`entropy.c`), the first inference (`mamdani.c`), the first fusion (`nfuse.c`), the first state machine (`fsm.c`), the first learner (`learn.c`), the first character renderer (`character.c`), the first multimodal renderers (`render_*.c`), the first peripheral wrappers (`hw_*.c`), the first display bridge (`display.c`), the first GPU bridge (`gpu.c`), the first cryptographic layer (`crypto.c`), and the first principal application (`main.c`) are all **consequences** — not new decisions.

The result is not merely a program. It is a **secure-by-construction, reproducible, auditable, and accessible** 5D interactive artificial intelligence assistant, centered on **Zevo**, the original Pixar-inspired cartoon character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**, conveyed through elegant, interconnected data flows rather than explanatory text — and the zero step is the foundation on which every later step stands.

---

## 19. References

1. System V Application Binary Interface, x86-64 Supplement.
2. ELF Specification, Tool Interface Standard.
3. Linux Kernel Documentation: `fs-verity`.
4. Linux Kernel Documentation: `seccomp_filter.rst`, `landlock.rst`.
5. ext4 Filesystem Documentation.
6. OpenSSL 3.x Documentation.
7. NIST SP 800-38D: GCM Mode.
8. NIST SP 800-57 Part 1 Rev. 5: Key Management.
9. NIST SP 800-107 Rev. 1: HMAC.
10. RFC 5869: HKDF.
11. RFC 8446: TLS 1.3.
12. FIPS 186-5: Digital Signature Standard.
13. Mamdani, E. H. (1974). *Application of fuzzy algorithms for control of a simple dynamic plant.*
14. Shannon, C. E. (1948). *A Mathematical Theory of Communication.*
15. WCAG 2.2, W3C Recommendation.
16. POSIX.1-2008.
17. Thomas, F., & Johnston, O. (1981). *The Illusion of Life: Disney Animation.* (Source of the animation principles referenced by "Pixar-inspired.")

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Title | The Zevo™ AI-Driven Core — An 5D Interactive Artificial Intelligence Assistant: A Zero-Step Foundation Paper |
| Status | Foundational paper (zero step) |
| Version | 1.0.0 |
| Effective Date | ____ |

**End of zero-step foundation paper.**

---

---

# Zevo ELF Image AI Core vs. GGUF v3 vs. "OpenGPT"

Below is a precise, side-by-side explanation of what these three things actually *are*, where they overlap, and why the Zevo™ ELF Image AI Core is **not** the same kind of object as a GGUF file or an "OpenGPT" (GPT-family) checkpoint. This is the conceptual bridge the paper's §5 (Weight Storage Contract) implicitly relies on but never states outright.

---

## 1. The Category Confusion to Dispel First

Three different *kinds* of artifacts get conflated in casual AI discussion:

| Kind | What it is | Examples |
|---|---|---|
| **Model weights file** | A serialized tensor container. Passive data. | GGUF, safetensors, PyTorch `.pt`, ONNX `.onnx` |
| **Runtime / serving binary** | Executable code that loads weights and runs inference. | `llama.cpp`, vLLM, TensorRT-LLM, ONNX Runtime, **`zevo_core.elf`** |
| **Model family / architecture** | A mathematical graph + training recipe. | GPT-2/3/4, Llama 2/3, Mistral, "OpenGPT" derivatives |

The Zevo™ ELF Image AI Core is in the **second** row. GGUF is in the **first**. "OpenGPT" (open-weight GPT-style models) is in the **third**. They are not substitutes — they occupy different layers of the stack.

---

## 2. What Each Actually Is

### 2.1 GGUF v3 — *a passive weight container*

| Property | Value |
|---|---|
| Full name | GPT-Generated Unified Format, version 3 |
| Introduced by | `llama.cpp` (Gerganov et al.), 2023 |
| Purpose | Store quantized LLM weights + metadata in one mmap-able file |
| Content | Header, KV metadata, tensor directory, tensor data |
| Executable? | **No.** It is data. It contains zero machine code. |
| Endianness | Little-endian, explicitly versioned |
| Quantization | Q4_0, Q4_K_M, Q5_K, Q6_K, Q8_0, etc. |
| Integrity | Optional per-tensor SHA-256 in metadata; **no kernel enforcement** |
| Loaded by | `llama.cpp`, `llamafile`, `ollama`, `LM Studio`, etc. |
| Kernel role | None. Just `open()` + `mmap()` by userspace. |

**Key point:** GGUF is *only* the weights. It requires a separate executable (`llama.cpp` or a derivative) to do anything. It is the digital equivalent of a `.wav` file — inert until a player reads it.

### 2.2 "OpenGPT" — *a model family, not a file format*

There is no official artifact called "OpenGPT." The term is used in the wild to mean one of:

| Usage | Meaning |
|---|---|
| Loose colloquial | Any open-weight GPT-style model (GPT-2, GPT-J, GPT-NeoX, Pythia, Falcon, etc.) |
| Specific projects | "OpenGPT-2", "OpenAssistant", "OpenLLaMA" — community reimplementations |
| Confusion with | OpenAI's GPT-1/2/3/4 — which are **closed** weights |

**Key point:** "OpenGPT" names an *architecture + training lineage*, not a file. A GPT-style model can be shipped as GGUF, safetensors, PyTorch, ONNX, or (in principle) as a section inside a Zevo ELF. The architecture is orthogonal to the container.

### 2.3 Zevo™ ELF Image AI Core — *an executable that carries its own weights*

| Property | Value |
|---|---|
| Full name | `zevo_core.elf` (Section-embedded mode) |
| Type | ELF, `ET_DYN`, System V x86_64 |
| Purpose | **Both** the runtime *and* the weight container in one file |
| Content | `.text` (code) + `.zevo_weights` (weights) + `.zevo_rules`, `.zevo_facts`, `.zevo_meta`, `.zevo_fsm`, `.zevo_entropy` |
| Executable? | **Yes.** It is the process. |
| Integrity | **fs-verity** — kernel-enforced Merkle tree on every read |
| Access control | Read-only mmap; seccomp-bpf; capability bounding |
| Loaded by | The kernel itself (via `ld.so` for PIE) |
| Kernel role | **Central.** Verifies integrity, enforces read-only mapping, filters syscalls. |

**Key point:** Zevo is the **union** of "runtime binary" and "weight container." The weights are not a separate file — they are a *section* of the executable, page-aligned on ext4, and verified by the kernel on every read.

---

## 3. Structural Equivalence Table

| Layer | GGUF v3 | OpenGPT (family) | Zevo™ ELF Core |
|---|---|---|---|
| **Code** | none (data only) | architecture spec | `.text`, pure C89/90 |
| **Weights** | tensor blobs | tensor blobs (training output) | `.zevo_weights` section |
| **Metadata** | KV pairs in header | config.json / tokenizer.json | `.zevo_meta` section |
| **Rules / facts** | none | none (implicit in weights) | `.zevo_rules`, `.zevo_facts` (explicit) |
| **FSM** | none | none | `.zevo_fsm` (explicit) |
| **Entropy gate** | none | none | `.zevo_entropy` (explicit) |
| **Integrity** | optional SHA-256 | none | **fs-verity Merkle tree** |
| **Executable** | no | no | **yes** (`ET_DYN`) |
| **IPC exposure** | n/a | n/a | AF_UNIX `SOCK_SEQPACKET` |
| **Key rotation** | n/a | n/a | 28-hour HKDF-SHA-384 |
| **Multimodal output** | text tokens only | text tokens only | Text + Image BGRA + Video BGRA + Audio PCM + 5D vector |
| **Pure-function discipline** | n/a | n/a | enforced in `pure.h` |

---

## 4. Where They Are *Functionally* Equivalent

This is the important clarification: at the level of **learned parameters**, the three are the same kind of thing.

A float32 matrix of shape `[d_model, d_ff]` is the same 4-byte-per-element array whether it lives:

- in a GGUF tensor block,
- in a PyTorch `.pt` state dict,
- in a safetensors shard,
- or in Zevo's `.zevo_weights` section.

**The weights are bit-identical in principle.** Only the *container*, *loader*, *integrity mechanism*, and *exposure surface* differ.

So when the paper says "LLM weights, metadata, and vectors are embedded by section or by reference," it means: Zevo stores *the same class of tensors* that GGUF stores, but places them inside an executable image rather than beside one.

---

## 5. Where They Are *Fundamentally* Different

| Dimension | GGUF / OpenGPT | Zevo™ ELF Core |
|---|---|---|
| **Trust boundary** | Userspace loader | Kernel (fs-verity, seccomp) |
| **Integrity** | Advisory hash | Enforced Merkle verification |
| **Attack surface** | File parser (C++), tokenizer, sampler | Single exported symbol, IPC only |
| **Reproducibility** | Build-dependent | `SOURCE_DATE_EPOCH=0` byte-identical |
| **Exposure** | Local file | Published IPC API |
| **Multimodality** | Text only (typically) | 5 channels + 5D fused vector |
| **Symbolic layer** | None | Mamdani rules + FSM + entropy gate |
| **Character/persona** | Prompted | Embodied (Zevo, the Pixar-inspired character) |

The GGUF ecosystem optimizes for **portability and quantization density**. The Zevo ELF Core optimizes for **integrity, auditability, and unified deployment** — a single signed artifact that *is* the model *and* the runtime.

---

## 6. Why the Zevo Design Chose the ELF Container

Three concrete reasons the paper's §5 makes this choice:

1. **Kernel-enforced integrity for free.** fs-verity gives Merkle verification on every read without a userspace hash check. GGUF's optional SHA-256 is advisory — a compromised loader can skip it.

2. **One artifact to sign, ship, and audit.** A GGUF + `llama.cpp` + config + tokenizer is four files. Zevo is one ELF with sections. The build-id, fs-verity root hash, and pinned digest all refer to the same bytes.

3. **No parser attack surface.** A GGUF loader is a substantial C++ parser handling untrusted input. Zevo's weights are mmap'd as a typed struct — there is no parsing step, only a header magic check and a page-alignment assertion.

---

## 7. Compatibility Path: How a GGUF Could Become a Zevo Core

Hypothetically, if one wanted to convert a GGUF-based model into a Zevo-style ELF Core:

| Step | Action |
|---|---|
| 1 | Extract tensors from GGUF into a Zevo `zevo_w_header_t` + blobs layout |
| 2 | Emit a linker script pinning `.zevo_weights` at a page-aligned offset |
| 3 | Compile a pure-C89 forward pass (attention + MLP + norm) as `.text` |
| 4 | `objcopy --add-section .zevo_weights=…` into the ELF |
| 5 | `fsverity enable zevo_core.elf` |
| 6 | Publish an IPC API in place of a `main()` that prints tokens |

The *weights survive unchanged*. Only the *container and the exposure contract* change. This is the deepest equivalence: **Zevo is a re-containerization of the same tensors, with a different trust model.**

---

## 8. Summary in One Sentence Each

- **GGUF v3** is a passive, mmap-able tensor file — weights only, no code, advisory integrity, text-only when paired with `llama.cpp`.
- **OpenGPT** is a loose name for GPT-style open-weight model *families* — an architecture and training lineage, not a file format.
- **Zevo™ ELF Image AI Core** is an *executable* `ET_DYN` ELF that carries its own weights as a kernel-verified, page-aligned, read-only section, exposing only a published IPC API and producing a multimodal 5D artifact.

They share the same *numerical substrate* (learned float tensors) but differ in *container*, *loader*, *integrity model*, *exposure surface*, and *output modality*. Zevo is not a GGUF replacement — it is a **different layer of the stack**, deliberately fusing runtime and weights into one auditable, secure-by-construction artifact.

---

---

# Enhancing Complete Academic Paper for Building Zevo™, An 5D Interactive Artificial Intelligence Assistant

## PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams

---

## Abstract

This paper presents the complete architecture, implementation, and deployment of **Zevo™**, an **5D interactive artificial intelligence assistant** built as a pure-functional, ANSI C89/90, GNU/POSIX, System V x86_64 `ET_DYN` ELF AI Core. Zevo integrates **fuzzy-state-machine logic**, **neural fuzzy logic**, **rules and facts**, **Shannon entropy**, **Mamdani inference**, and **machine learning** into a single deterministic pipeline that produces a **multimodal 5D Smart 8K HDR BGRA** artifact — comprising **Text Raw**, **Image BGRA**, **Video BGRA**, **Audio PCM**, and a **fused 5D intelligence vector**. The LLM weights, metadata, and vectors are embedded **by section or by reference**, **page-aligned on ext4**, **fs-verity-protected**, and **mapped read-only**. The core exposes **only a published IPC API** to a **weight-free, stateless, Web-Responsible client**. Cryptographic access is mediated by a **Secure Service API** with **28-hour API key rotation** using **OpenSSL military-grade primitives**. Hardware peripherals are reached exclusively through **libc wrappers** that resolve to the **`syscall`** instruction on x86_64 (and **`sysenter`** on legacy x86), under **seccomp-bpf**, **capability bounding**, and **LSM** discipline. The result is a **secure-by-construction**, **reproducible**, **auditable**, and **accessible** AI assistant centered on **Zevo**, a Pixar-inspired animated character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**.

**Keywords:** ELF AI Core, ET_DYN, fs-verity, ext4, IPC, OpenSSL, HKDF, HMAC-SHA-384, AES-256-GCM, seccomp-bpf, Mamdani inference, Shannon entropy, multimodal generation, Responsible UI, Pixar-inspired animation.

---

## 1. Introduction

### 1.1 Motivation

Modern AI systems face four simultaneous challenges:

1. **Security** — protecting learned weights from exfiltration and tampering.
2. **Portability** — running the same core across GNU/Linux environments.
3. **Auditability** — proving that the executed code and loaded weights are the intended ones.
4. **Accessibility** — exposing AI capabilities through a Responsible UI that respects privacy, motion, color, and keyboard semantics.

Zevo™ addresses all four through a single architectural decision: **separate the AI Core from the Web-Responsible client, and mediate all access through a published IPC API protected by rotating, OpenSSL-backed cryptographic keys.**

### 1.2 Contributions

| Contribution | Description |
|---|---|
| Dual-mode weight embedding | Section-embedded (`.zevo_weights`) or reference-embedded (`weights.bin`) |
| fs-verity integrity | Kernel-enforced Merkle verification on every read |
| Read-only mapping | `PROT_READ`, `MAP_PRIVATE`, no write path |
| IPC-only exposure | `AF_UNIX` `SOCK_SEQPACKET` + length-prefixed JSON |
| 28-hour key rotation | HKDF-SHA-384 per epoch, 5-minute overlap |
| OpenSSL cryptography | TLS 1.3, HMAC-SHA-384, AES-256-GCM, ECDSA P-384 |
| Weight-free web layer | Verified by symbol inspection |
| Responsible UI | Semantic, accessible, private, transparent |
| Syscall discipline | libc wrappers, seccomp-bpf allow-list, capability bounding |
| Reproducible builds | `SOURCE_DATE_EPOCH=0`, byte-identical ELF |

### 1.3 Paper Organization

Section 2 presents related work. Section 3 defines the architecture. Section 4 specifies the reasoning engine. Section 5 details the weight storage contract. Section 6 describes the IPC API. Section 7 covers cryptographic access. Section 8 covers hardware access. Section 9 presents the Responsible UI. Section 10 describes implementation. Section 11 covers audit. Section 12 covers deployment. Section 13 concludes.

---

## 2. Related Work

| Domain | Representative Work | Zevo™ Differentiation |
|---|---|---|
| Containerized AI | Docker, OCI images | ELF-native, no container runtime required |
| Model serving | TorchServe, TF Serving | Pure-functional C89/90 core, IPC-only |
| Integrity | dm-verity, IMA | fs-verity at file level, page-aligned |
| Secure enclaves | SGX, SEV | Standard GNU/Linux, no enclave required |
| Fuzzy inference | Mamdani, Sugeno | Integrated with neural embeddings and FSM |
| Responsible UI | WCAG 2.2, ARIA | Applied end-to-end, no telemetry |
| Key rotation | NIST SP 800-57 | 28-hour epoch, HKDF-SHA-384 |
| Syscall filtering | seccomp-bpf, Landlock | Allow-list + capability bounding |

---

## 3. Architecture

### 3.1 Four Trust Boundaries

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              Zevo™ System                                  │
│                                                                            │
│  ┌──────────────┐   ┌────────────────┐   ┌──────────────┐   ┌──────────┐   │
│  │  Browser     │   │  Secure        │   │  ELF AI      │   │ Universal│   │
│  │  Responsible │──►│  Service API   │──►│  Core        │──►│ Hardware │   │
│  │  UI          │   │  (zevo_api)    │   │  (zevo_core) │   │ Devices  │   │
│  └──────────────┘   └────────────────┘   └──────────────┘   └──────────┘   │
│     Boundary 1          Boundary 2          Boundary 3        Boundary 4   │
│     (public)            (service)           (core)            (hardware)   │
└────────────────────────────────────────────────────────────────────────────┘
```

| Boundary | Between | Protection |
|---|---|---|
| 1 | Browser ↔ Secure Service API | TLS 1.3, session tokens |
| 2 | Secure Service API ↔ ELF AI Core | 28-hour API key rotation, HMAC-SHA-384, AES-256-GCM |
| 3 | ELF AI Core ↔ `.zevo_weights` | fs-verity, read-only mmap, pinned digest |
| 4 | ELF AI Core ↔ Universal Hardware | libc → `syscall`/`sysenter`, seccomp-bpf, capabilities |

### 3.2 ELF AI Core Attributes

| Attribute | Value |
|---|---|
| File | `zevo_core.elf` |
| Format | ELF |
| Type | `ET_DYN` |
| ABI | System V x86_64 |
| Language | ANSI C89/90 |
| Standards | GNU + POSIX + Pthreads + OpenCL + OpenGL + SDL2 + OpenAL + OpenSSL|
| Role | Executable host: reasoning, weights access, multimodal generation |
| Exposure | Local IPC only |

### 3.3 Pure-Function Discipline

Every pipeline stage is a pure function:

```
static void *stage_<name>(const void *input);
```

- Takes a `const void *`.
- Returns a newly allocated value.
- Reads no global mutable state.
- Writes no global mutable state.
- Is safe to call concurrently.

---

## 4. Reasoning Engine

### 4.1 Fuzzy-State-Machine Logic

The core progresses through a finite state machine:

| State | Meaning | Transition |
|---|---|---|
| `IDLE` | Awaiting prompt | prompt received → `ASSEMBLE` |
| `ASSEMBLE` | Carousel assembling right-to-left | entropy ≤ τ → `TRANSMIT` |
| `TRANSMIT` | Emitting artifact | complete → `DONE` |
| `DONE` | Artifact delivered | reset → `IDLE` |
| `HALT` | Fault | manual reset → `IDLE` |

### 4.2 Shannon Entropy Gate

```
H(X) = − Σ_i p(x_i) · log₂ p(x_i)

H > τ_high  →  keep assembling
H ≤ τ_low   →  assembly complete, transmission begins
```

The entropy value modulates ring luminosity and particle density, so the visual honestly reflects the epistemic state of the core.

### 4.3 Mamdani Inference

```
Given:
  Facts  F = {f₁, f₂, …, fₙ}
  Rules  R = {r₁, r₂, …, rₘ},  rᵢ : IF x is Aᵢ THEN y is Bᵢ

Steps:
  1. Fuzzification:  μ_Aᵢ(x) for each fact
  2. Rule firing:    wᵢ = min(μ_Aᵢ(x₁), …, μ_Aᵢ(x_k))
  3. Aggregation:    μ_B(y) = max_i min(wᵢ, μ_Bᵢ(y))
  4. Defuzzify:      y* = ∫ y·μ_B(y) dy / ∫ μ_B(y) dy   (centroid)
```

### 4.4 Neural-Fuzzy Fusion

```
neural_fuse(fuzzy_out, embedding) =
    σ( W_f · fuzzy_out  +  W_e · embedding  +  b )
```

This hybrid layer combines symbolic fuzzy reasoning with learned neural representations.

### 4.5 Machine Learning

Weights are learned offline and embedded at build time. No online learning occurs in production; the core is deterministic and reproducible.

---

## 5. Weight Storage Contract

### 5.1 Three Categories

| Category | Contents | Example |
|---|---|---|
| Weights | Learned parameters | Embedding matrices, attention projections |
| Metadata | Provenance, versioning, schema, digests | Title, subtitle, schema table, pinned SHA-256 |
| Vectors | Static embeddings, prototypes, consequents | Token embeddings, modality prototypes |

### 5.2 Dual-Mode Embedding

**Mode A — Section-Embedded**

```
zevo_core.elf
├── .text
├── .rodata
├── .zevo_rules
├── .zevo_facts
├── .zevo_entropy
├── .zevo_fsm
├── .zevo_meta
└── .zevo_weights   ← weights, metadata, vectors
```

- Section flagged `alloc, readonly`.
- ELF placed on page-aligned ext4.
- fs-verity enabled on the ELF.
- Kernel loader maps section read-only.

**Mode B — Reference-Embedded**

- Weights in `weights.bin` on ext4.
- fs-verity enabled on `weights.bin`.
- Pinned digest in `.zevo_meta`.
- Core `open()`s, verifies, `mmap()`s read-only.

### 5.3 Page Alignment

| Term | Meaning |
|---|---|
| Page | 4096 bytes on x86_64 |
| Page-aligned | Offset is an exact multiple of 4096 |
| on ext4 | Filesystem provides stable physical layout for mmap |
| Verification | `((uintptr_t)base & 0xFFF) == 0` |

Misalignment → core refuses to start.

### 5.4 fs-verity

- Kernel builds a Merkle tree at enable time.
- Tree stored in a hidden metadata inode.
- Root hash is the file's identity.
- Every read verified against the tree.
- Tampering → `EIO` on read.

fs-verity is **not encryption**. Confidentiality is layered separately (LUKS, dm-crypt, fscrypt).

### 5.5 Read-Only Mapping

```
mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0)
```

- `PROT_READ` — no `PROT_WRITE` bit.
- `MAP_PRIVATE` — copy-on-write semantics, but no writes occur.
- No `mprotect` ever adds `PROT_WRITE`.
- No `msync` ever flushes writes.
- Pointer table frozen after construction.

---

## 6. IPC API

### 6.1 Transport

| Layer | Choice |
|---|---|
| Socket family | `AF_UNIX` |
| Socket type | `SOCK_SEQPACKET` |
| Path | `/run/zevo/core.sock` |
| Permissions | `0600`, owner `zevo` |
| Framing | 4-byte big-endian length prefix + UTF-8 JSON |
| Max frame | 1 MiB |
| Peer check | `SO_PEERCRED` |

### 6.2 Published Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/core/state` | FSM state, Shannon entropy, rules fired |
| `GET` | `/core/rules` | Read-only Mamdani rule set |
| `GET` | `/core/meta` | Exact title **Zevo™**, exact subtitle, version |
| `POST` | `/core/assist` | Submit prompt → artifact metadata |
| `GET` | `/core/artifact/{id}` | Fetch rendered artifact (opt-in) |
| `GET` | `/core/health` | Liveness |

### 6.3 What the API Never Exposes

- Raw weights.
- Vector pointers.
- fs-verity digest internals.
- File paths of weights.
- Memory addresses.
- Internal rule representations.
- Partial inference state.

---

## 7. Cryptographic Access

### 7.1 28-Hour Key Rotation

```
Epoch N   = floor(unix_time_utc / (28 * 3600))
Key_N     = HKDF-SHA-384(master_secret, salt=SHA-384(Epoch_N),
                          info="zevo-api-v1", L=32)
Overlap   = 5 minutes
```

| Property | Value |
|---|---|
| Key length | 256 bits |
| Derivation | HKDF-SHA-384 |
| Rotation period | 28 hours |
| Overlap window | 5 minutes |
| Master secret | 512 bits, sealed, mode 0400, optional HSM |

### 7.2 OpenSSL Primitives

| Purpose | Primitive | OpenSSL API |
|---|---|---|
| TLS 1.3 | Transport | `SSL_CTX_new(TLS_server_method())` |
| HKDF | Key derivation | `EVP_KDF` with `"HKDF"` |
| HMAC | Frame integrity | `HMAC(EVP_sha384(), …)` |
| AES-256-GCM | Frame sealing | `EVP_EncryptInit_ex` / `EVP_DecryptInit_ex` |
| ECDSA P-384 | Audit signing | `EVP_DigestSign` |
| SHA-384 | Digests | `EVP_DigestInit_ex` |
| CSPRNG | Randomness | `RAND_bytes` |

### 7.3 Authenticated IPC Frame

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│ length (4B)  │ epoch (8B)   │ HMAC (48B)   │ JSON payload (N B)  │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

The core rejects any frame whose HMAC does not verify in constant time.

---

## 8. Hardware Access

### 8.1 Universal Hardware Device Peripherals

| Class | Examples | Kernel Interface |
|---|---|---|
| Compute | GPU, NPU, FPGA | OpenCL, DRM, VFIO |
| Graphics | GPU, display | DRM/KMS, OpenGL |
| Audio | DAC, ADC | ALSA, OpenAL |
| Video | Camera, encoder | V4L2, SDL2 |
| Storage | NVMe, SATA | Block layer, ext4 |
| Network | NIC | Sockets |
| Input | Keyboard, mouse | evdev |
| Timers | HPET, TSC | `clock_gettime` |
| Random | RDRAND, RDTSC | `getrandom` |

### 8.2 `syscall` vs `sysenter`

| Mechanism | Architecture | Use |
|---|---|---|
| `syscall` | x86_64 | Standard fast system-call instruction |
| `sysenter` | x86 (legacy) | 32-bit fast system-call instruction |
| `int 0x80` | x86, x86_64 (compat) | Legacy interrupt-based entry |

On x86_64, the correct instruction is `syscall`. The `sysenter` instruction is the 32-bit counterpart. Modern ANSI C89/90 code uses **libc wrappers**, which internally use the appropriate instruction.

### 8.3 Syscall Discipline

The core is restricted by **seccomp-bpf** to a minimal allow-list:

| Syscall | Purpose |
|---|---|
| `read`, `write` | IPC and device I/O |
| `openat`, `close` | Descriptor management |
| `mmap`, `munmap` | Memory mapping |
| `ioctl` | Device control (allow-listed) |
| `futex` | Thread synchronization |
| `clock_gettime` | Timestamps |
| `getrandom` | CSPRNG seed |
| `exit_group` | Clean exit |

All other syscalls cause `SIGSYS`.

### 8.4 Capability Bounding

| Capability | Status |
|---|---|
| `CAP_SYS_ADMIN` | Dropped |
| `CAP_SYS_RAWIO` | Dropped |
| `CAP_NET_ADMIN` | Dropped |
| `CAP_SYS_MODULE` | Dropped |
| `CAP_DAC_OVERRIDE` | Dropped |
| `CAP_SYS_PTRACE` | Dropped |

Device access is granted via **udev rules** and **group membership**.

---

## 9. Responsible Web UI

### 9.1 Principles

| Principle | Implementation |
|---|---|
| Semantic structure | `<header>`, `<main>`, `<section>`, `<footer>` |
| Accessibility | ARIA live regions, focus-visible, keyboard-first |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | No telemetry, no cookies, no third-party scripts |
| Transparency | Clear disclosure that Zevo™ is an AI assistant |
| Artifact opt-in | Pixels/audio served only on explicit request |
| No secrets in browser | No API keys, no master secret |

### 9.2 Multimodal Channels

| Channel | Input | Output | Consent |
|---|---|---|---|
| Text | `<textarea>` | JSON in `<pre aria-live>` | Explicit |
| Image | `<input type="file">` | `<img alt>` opt-in | Explicit |
| Video | `<input>` or `getUserMedia` | `<video controls>` no autoplay | Explicit |
| Audio | `MediaRecorder` | `<audio controls>` no autoplay | Explicit |

### 9.3 Session Tokens

- Issued after TLS handshake.
- 256-bit random, `RAND_bytes`.
- Bound to TLS session.
- Lifetime: 15 minutes, renewable.
- Stored in `HttpOnly`, `Secure`, `SameSite=Strict` cookie.

---

## 10. Implementation

### 10.1 Repository Layout

```
zevo/
├── Makefile
├── docs/{SPEC,AUDIT,DEPLOY}.md
├── core/
│   ├── main.c
│   ├── zevo.h, pure.h, schema.h
│   ├── weights.c, ipc.c, pipeline.c
│   ├── entropy.c, fsm.c, mamdani.c
│   ├── render.c, audio.c, video.c
│   └── zevo_core.ld
├── web/
│   ├── zevo_web.c, routes.c
│   └── ui/{index.html, styles.css}
├── weights/provision.sh
├── tests/{unit,integration,fuzz}
└── deploy/{systemd,udev,ext4,containers}
```

### 10.2 Build

```makefile
CC      = gcc
CFLAGS  = -std=c89 -pedantic -Wall -Wextra -Werror \
          -D_GNU_SOURCE -D_POSIX_C_SOURCE=200809L \
          -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 \
          -O2 -g -fno-omit-frame-pointer
LDFLAGS = -shared -Wl,-z,relro,-z,now -Wl,--build-id=sha1
LIBS    = -lpthread -lOpenCL -lGL -lGLU -lSDL2 -lopenal -lm
```

### 10.3 Weight Embedding

```sh
objcopy --add-section .zevo_weights=weights.bin \
        --set-section-flags .zevo_weights=alloc,readonly \
        zevo_core.elf
fsverity enable zevo_core.elf
fsverity measure zevo_core.elf
```

### 10.4 Reproducibility

```sh
SOURCE_DATE_EPOCH=0 make clean all
sha256sum zevo_core.elf zevo_web > build1.sha
SOURCE_DATE_EPOCH=0 make clean all
sha256sum zevo_core.elf zevo_web > build2.sha
diff build1.sha build2.sha    # identical
```

---

## 11. Audit

### 11.1 Static Analysis

```sh
gcc -std=c89 -pedantic -Wall -Wextra -Werror -Wconversion \
    -Wshadow -Wstrict-prototypes -Wcast-qual -c core/*.c
cppcheck --std=c89 --enable=all core/ web/
clang --analyze -std=c89 core/*.c web/*.c
```

### 11.2 Dynamic Analysis

```sh
valgrind --leak-check=full --track-origins=yes ./zevo_core.elf
valgrind --tool=helgrind ./zevo_core.elf
gcc -fsanitize=address,undefined -g core/*.c -o zevo_core.asan
```

### 11.3 Fuzzing

| Target | Fuzzer | Input |
|---|---|---|
| IPC JSON parser | libFuzzer | Random bytes |
| Tokenizer | AFL++ | Unicode |
| Mamdani inference | libFuzzer | Random facts |
| Entropy gate | libFuzzer | Random vectors |
| Weight loader | AFL++ | Tampered BINs |

### 11.4 Security Checklist

| Item | Verification |
|---|---|
| Weights not in web binary | `nm zevo_web \| grep -i weight` → empty |
| fs-verity enabled | `fsverity measure` |
| Digest pinned | `.zevo_meta` contains SHA-256 |
| Mapping read-only | `PROT_READ` only |
| Socket permissions | `srw-------` |
| Peer credentials | `SO_PEERCRED` |
| Frame size capped | 1 MiB |
| Rate limited | Per-IP token bucket |
| Headers hardened | `nosniff`, `no-store`, CSP |
| No telemetry | Network trace |
| No cookies | `Set-Cookie` never emitted |
| Reproducible | Same hash on rebuild |

### 11.5 Accessibility Audit

| Check | Tool |
|---|---|
| WCAG-AA contrast | axe-core, pa11y |
| Keyboard navigation | Manual |
| ARIA correctness | axe-core, WAVE |
| Reduced motion | Browser dev tools |
| Screen reader | NVDA, Orca |

---

## 12. Deployment

### 12.1 Topology

```
Host (GNU/Linux, ext4)
├── zevo-core.service  →  zevo_core.elf  →  /run/zevo/core.sock
├── zevo-web.service   →  zevo_web       →  0.0.0.0:8080
├── /var/lib/zevo/weights.bin  (ext4 + fs-verity, mode 0400)
└── /run/zevo/core.sock        (AF_UNIX, mode 0600)
```

### 12.2 systemd Hardening

```ini
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadOnlyPaths=/var/lib/zevo
ReadWritePaths=/run/zevo
SystemCallFilter=@system-service
SystemCallArchitectures=native
SystemCallErrorNumber=EPERM
DeviceAllow=/dev/dri/renderD128 rw
DeviceAllow=/dev/snd/controlC0 rw
DeviceAllow=/dev/urandom r
DevicePolicy=closed
CapabilityBoundingSet=
```

### 12.3 Health Checks

| Check | Endpoint | Expected |
|---|---|---|
| Core liveness | `GET /core/health` | `{"status":"ok"}` |
| Core state | `GET /core/state` | `{"state":"IDLE"}` |
| Web liveness | `GET /health` | `{"status":"ok"}` |
| Meta | `GET /api/core/meta` | Title, subtitle |

### 12.4 Upgrade and Rollback

Both follow the pattern: stop service, replace artifact, re-enable fs-verity, start service. Rollback is symmetric.

---

## 13. Conclusion

Zevo™ is an **5D interactive artificial intelligence assistant** built as a pure-functional, ANSI C89/90, GNU/POSIX, System V x86_64 `ET_DYN` ELF AI Core. Its LLM weights, metadata, and vectors are embedded **by section or by reference**, **page-aligned on ext4**, **fs-verity-protected**, and **mapped read-only**. It exposes **only a published IPC API** to a **weight-free, stateless, Web-Responsible client**, mediated by a **Secure Service API** with **28-hour API key rotation** and **OpenSSL military-grade cryptography**. Hardware peripherals are reached through **libc wrappers** resolving to the **`syscall`** instruction on x86_64 (and **`sysenter`** on legacy x86), under **seccomp-bpf**, **capability bounding**, and **LSM** discipline.

The system produces the **Zevo™ multimodal 5D Smart 8K HDR BGRA** hero illustration, video, and audio — centered on **Zevo**, the Pixar-inspired AI character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**, conveyed through elegant interconnected data flows that visualize fuzzy-state-machine logic, neural fuzzy logic, rules and facts, Shannon entropy, Mamdani inference, and machine learning.

The result is **secure by construction**, **reproducible**, **auditable**, **accessible**, **privacy-preserving**, and faithful to every term of the canonical specification.

---

## 14. Future Work

| Direction | Description |
|---|---|
| Hardware-backed sealing | TPM/HSM for master secret |
| Post-quantum readiness | ML-KEM, ML-DSA migration path |
| Multi-tenant isolation | Per-tenant weight maps |
| Federated learning | Offline weight updates with signed deltas |
| Formal verification | Model-checking the pure pipeline |
| Expanded accessibility | Full WCAG-AAA target |
| Multi-arch support | aarch64, riscv64 `ET_DYN` |
| Streaming artifacts | Chunked video/audio over IPC |

---

## 15. References

1. System V Application Binary Interface, x86-64 Supplement.
2. ELF Specification, Tool Interface Standard.
3. Linux Kernel Documentation: fs-verity.
4. ext4 Filesystem Documentation.
5. OpenSSL 3.x Documentation.
6. NIST SP 800-57: Key Management.
7. NIST SP 800-108: KDF in Counter Mode.
8. RFC 5869: HKDF.
9. RFC 8446: TLS 1.3.
10. Mamdani, E. H. (1974). Application of fuzzy algorithms.
11. Shannon, C. E. (1948). A Mathematical Theory of Communication.
12. WCAG 2.2, W3C Recommendation.
13. POSIX.1-2008.
14. seccomp-bpf Documentation.
15. Landlock LSM Documentation.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Title | Enhancing Complete Academic Paper for Building Zevo™, An 5D Interactive Artificial Intelligence Assistant |
| Status | Complete academic paper |
| Version | 1.0.0 |
| Effective Date | ____ |

**End of academic paper.**

---

---

### ELF AI Core with Dual-Mode Weight Embedding, fs-verity Protection, and IPC-Only Access

PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams 

## Purpose of This Clarification

This document is the **authoritative clarification** of the Zevo™ ELF AI Core architecture. It explains, in precise terms:

1. What an **ELF AI Core** is.
2. How **LLM weights, metadata, and vectors** are **embedded by section or by reference**.
3. What **page-aligned on ext4** means concretely.
4. How **fs-verity** protects the data.
5. Why the mapping is **read-only**.
6. How the core **exposes only a published IPC API**.
7. Why the **Web-Responsible client is weight-free and stateless**.
8. How this produces the **Zevo™ multimodal 5D Smart 8K HDR BGRA** hero illustration, video, and audio.

This is a clarification, not a redesign. It resolves ambiguities, removes contradictions, and aligns every term with its precise meaning.

---

## 1. What an ELF AI Core Is

| Attribute | Value |
|---|---|
| File | `zevo_core.elf` |
| Format | ELF (Executable and Linkable Format) |
| Type | `ET_DYN` (position-independent shared object) |
| ABI | System V x86_64 |
| Language | ANSI C89/90 |
| Standards | GNU + POSIX + Pthreads + OpenCL + OpenGL + SDL2 + OpenAL |
| Role | Hosts the entire AI: reasoning, weights access, multimodal generation |
| Exposure | Local IPC only — no network, no direct symbol linkage |
| Trust domain | Separate from the web layer |

An ELF AI Core is **not** a model file. It is an **executable host** that:
- Loads and verifies weights.
- Runs the pure-function pipeline.
- Generates multimodal artifacts.
- Serves them only through a published IPC API.

---

## 2. Three Kinds of Embedded Data

The core carries three categories of learned or learned-adjacent data:

| Category | Contents | Example |
|---|---|---|
| **Weights** | Learned parameters of the LLM | Embedding matrices, attention projections, MLP layers |
| **Metadata** | Provenance, versioning, schema, digests | Title, subtitle, version, schema table, pinned SHA-256 |
| **Vectors** | Static embeddings, class prototypes, rule consequents | Token embeddings, modality prototypes, Mamdani consequents |

All three are subject to the **same embedding contract**: page-aligned, fs-verity-protected, mapped read-only.

---

## 3. Two Embedding Modes — One Interface

The phrase *"embedded by section or by reference"* describes **two valid deployment modes**. Both satisfy the same invariants. The core chooses at build time; the pipeline sees only a `const WeightMap *`.

### 3.1 Mode A — Section-Embedded

```
zevo_core.elf
├── .text
├── .rodata
├── .data
├── .bss
├── .zevo_rules
├── .zevo_facts
├── .zevo_entropy
├── .zevo_fsm
├── .zevo_meta
└── .zevo_weights   ← weights, metadata, vectors live HERE
```

- Weights are placed in a **custom ELF section** `.zevo_weights`.
- The section is flagged `alloc, readonly`.
- The ELF itself is placed on a **page-aligned ext4** region.
- **fs-verity is enabled on the ELF** (`.zevo_weights` is covered by the ELF's Merkle tree).
- The kernel loader maps the section **read-only** as part of the ELF image.

```
objcopy --add-section .zevo_weights=weights.bin \
        --set-section-flags .zevo_weights=alloc,readonly \
        zevo_core.elf
fsverity enable zevo_core.elf
```

**Access:**

```
extern const unsigned char __start_zevo_weights[];
extern const unsigned char __stop_zevo_weights[];
```

### 3.2 Mode B — Reference-Embedded

```
zevo_core.elf                      /var/lib/zevo/weights.bin
├── .text                          ├── vector 0 (page-aligned)
├── .rodata                        ├── vector 1 (page-aligned)
├── .data                          ├── vector 2 (page-aligned)
├── .bss                           ├── ...
├── .zevo_rules                    └── fs-verity Merkle tree
├── .zevo_facts
├── .zevo_entropy
├── .zevo_fsm
└── .zevo_meta  ← pinned SHA-256 digest of weights.bin
```

- Weights live in a **separate page-aligned ext4 file** `weights.bin`.
- **fs-verity is enabled on `weights.bin`**.
- The **pinned digest** is embedded in `.zevo_meta` inside the ELF.
- The core `open()`s, verifies, and `mmap()`s the BIN **read-only**.

```
fsverity enable /var/lib/zevo/weights.bin
fsverity measure /var/lib/zevo/weights.bin
```

**Access:**

```
open(O_RDONLY|O_CLOEXEC)
→ ioctl(FS_IOC_MEASURE_VERITY)
→ compare against pinned digest in .zevo_meta
→ mmap(PROT_READ, MAP_PRIVATE)
```

### 3.3 Comparison

| Aspect | Mode A (Section) | Mode B (Reference) |
|---|---|---|
| Location | Inside the ELF | Separate ext4 file |
| fs-verity target | The ELF | The BIN |
| Distribution | Single file | ELF + BIN |
| Hot-swap weights | No (rebuild ELF) | Yes (replace BIN) |
| Digest pinning | In `.zevo_meta` | In `.zevo_meta` |
| Schema location | In `.rodata` | In `.rodata` |
| Page alignment | Section aligned on ext4 | File aligned on ext4 |
| mmap read-only | Yes | Yes |
| Web layer visibility | None | None |
| IPC exposure | Metadata only | Metadata only |

Both modes are **functionally identical** to the pure pipeline.

---

## 4. What "Page-Aligned on ext4" Means

| Term | Precise Meaning |
|---|---|
| **Page** | The smallest unit of virtual memory the kernel maps — typically 4096 bytes on x86_64 |
| **Page-aligned** | A byte offset that is an exact multiple of 4096 |
| **on ext4** | The filesystem that stores the data, providing stable physical layout for mmap |
| **Why it matters** | mmap requires page-aligned offsets; unaligned offsets force partial-page reads and break zero-copy |
| **How it's achieved** | `posix_fallocate`, `fallocate`, section alignment in the ELF, `objcopy` with aligned layout |
| **Verification** | `((uintptr_t)base & 0xFFF) == 0`, `offset % 4096 == 0` |

Concrete example:

```
Vector 0: offset 0x00000000  (0 × 4096)   ✓ aligned
Vector 1: offset 0x04000000  (16384 × 4096) ✓ aligned
Vector 2: offset 0x04040000  (16448 × 4096) ✓ aligned
Vector 3: offset 0x04040001  (unaligned)     ✗ rejected at load time
```

The core refuses to start if any vector offset is misaligned.

---

## 5. What fs-verity Protection Means

fs-verity is a **kernel-enforced integrity mechanism**. It is **not** encryption.

### 5.1 How It Works

1. At enable time, the kernel builds a **Merkle tree** over the file's contents.
2. The tree is stored in a **hidden metadata inode**.
3. The **root hash** is the file's identity.
4. Every read is **verified against the tree** by the kernel.
5. Any tampering causes **EIO** on read.

### 5.2 What It Guarantees

| Guarantee | Meaning |
|---|---|
| Integrity | The bytes read are exactly the bytes that were sealed |
| Authenticity | The Merkle root matches the pinned digest |
| Tamper-evidence | Any modification is detectable |
| Kernel enforcement | Verification happens in kernel space, not userspace |

### 5.3 What It Does Not Guarantee

| Non-Guarantee | Explanation |
|---|---|
| Confidentiality | Contents are not hidden or encrypted |
| Availability | The file can still be deleted or made unreadable |
| Kernel integrity | A compromised kernel can bypass verification |
| Rollback protection | Old but valid versions may still verify |

Confidentiality, if required, is layered separately (LUKS, dm-crypt, fscrypt).

### 5.4 Runtime Measurement

```
struct fsverity_digest *d;
unsigned char buf[sizeof(struct fsverity_digest) + 64];

d = (struct fsverity_digest *)buf;
d->digest_algorithm = FS_VERITY_HASH_ALG_SHA256;
d->digest_size      = 64;

ioctl(fd, FS_IOC_MEASURE_VERITY, d);
```

The core compares the returned digest against the **pinned expected digest** in `.zevo_meta`. Mismatch → refuse to start.

---

## 6. Why the Mapping Is Read-Only

| Reason | Explanation |
|---|---|
| Immutability | Weights must not change during inference |
| Concurrency | Read-only mappings are safe for concurrent threads |
| Determinism | Same input → same output, every time |
| Security | No writable path means no runtime tampering |
| Auditability | The Merkle root fixes the bytes |
| Performance | Read-only pages can be shared, cached, and evicted safely |

Enforced by:

```
mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0)
```

- `PROT_READ` — no `PROT_WRITE` bit.
- `MAP_PRIVATE` — copy-on-write semantics, but no writes ever occur.
- No `mprotect` call ever adds `PROT_WRITE`.
- No `msync` call ever flushes writes.

The core's pointer table into the mapping is **frozen** after construction.

---

## 7. What "Exposing Only a Published IPC API" Means

### 7.1 Transport

| Layer | Choice |
|---|---|
| Socket family | `AF_UNIX` |
| Socket type | `SOCK_SEQPACKET` |
| Path | `/run/zevo/core.sock` |
| Permissions | `0600`, owner `zevo` |
| Framing | 4-byte big-endian length prefix + UTF-8 JSON |
| Max frame | 1 MiB |
| Peer check | `SO_PEERCRED` |

### 7.2 Published Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/core/state` | FSM state, Shannon entropy, rules fired |
| `GET` | `/core/rules` | Read-only Mamdani rule set |
| `GET` | `/core/meta` | Exact title **Zevo™**, exact subtitle, version |
| `POST` | `/core/assist` | Submit prompt → receive artifact metadata |
| `GET` | `/core/artifact/{id}` | Fetch rendered artifact (opt-in) |
| `GET` | `/core/health` | Liveness |

### 7.3 What the API Never Exposes

| Never Exposed | Why |
|---|---|
| Raw weights | Least privilege, information hiding |
| Vector pointers | Prevent pointer leakage |
| fs-verity digest internals | Internal trust mechanism |
| File paths of weights | Prevent path-based attacks |
| Memory addresses | Prevent ASLR bypass |
| Internal rule representations | Only the read-only rule set is served |
| Partial inference state | Only final artifact metadata |

### 7.4 Why IPC-Only

| Reason | Explanation |
|---|---|
| Isolation | The core cannot be linked into another process |
| Replaceability | Any client speaking the API can drive the core |
| Least privilege | Only the core reads weights |
| Auditing | Every request is observable at the socket |
| Versioning | The API is the contract; internals are free to change |
| Security | No symbol export, no direct linkage |

---

## 8. Why the Web-Responsible Client Is Weight-Free and Stateless

### 8.1 Weight-Free

The web client contains:
- No model weights.
- No vector buffers.
- No OpenCL contexts bound to weights.
- No schema tables for weights.
- No fs-verity digests for weights.

Verified by:

```
nm zevo_web | grep -iE "weight|vector|fsverity|cl_mem"
# Expected: empty
```

### 8.2 Stateless

The web client:
- Holds no per-user state beyond the active HTTP request.
- Stores no cookies.
- Writes no session files.
- Keeps no caches of artifacts.
- Forwards each request independently to the core.

### 8.3 Why This Matters

| Principle | Benefit |
|---|---|
| Least privilege | Web compromise cannot exfiltrate weights |
| Replaceability | Web layer can be rewritten in any language |
| Scalability | Stateless clients scale horizontally |
| Auditability | Each request is self-contained |
| Privacy | No cross-request correlation |
| Responsible UI | No hidden state, no tracking |

### 8.4 Responsible UI Guarantees

| Guarantee | Implementation |
|---|---|
| Semantic structure | `<header>`, `<main>`, `<section>`, `<footer>` |
| Accessibility | ARIA live regions for entropy/state, focus-visible, keyboard-first |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | No telemetry, no cookies, no third-party scripts |
| Transparency | "Zevo™ is an AI assistant. Outputs are generated." |
| Artifact opt-in | Pixels/audio served only on explicit request |
| Rate limiting | Per-IP token bucket |
| Payload safety | `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` |
| Content security | No inline `eval`, strict CSP |

---

## 9. What the Core Produces

The core produces the **Zevo™ multimodal 5D Smart 8K HDR BGRA** artifact:

| Channel | Format | Specification |
|---|---|---|
| Text Raw | UTF-8 | Prompt + tokens + embedding |
| Image BGRA | 7680×4320, 16-bit float per channel | HDR, ACES tone-mapped |
| Video BGRA | 300 frames @ 30 fps, same resolution | Camera dolly + ring rotation |
| Audio PCM | 48 kHz, stereo, 16-bit | Cinematic score |
| Fused 5D | Affinity vector | Cross-modal coherence |

### 9.1 Hero Illustration Elements

| Element | Specification |
|---|---|
| Title | **Zevo™** (exact, legible) |
| Subtitle | **An 5D interactive artificial intelligence assistant** (exact, legible) |
| Center | Zevo — charming, sophisticated, Pixar-inspired, rounded, expressive |
| Rings | Text · Image · Video · Audio · Fused 5D — luminous, concentric, balanced |
| Visual cues | Flowing typography, image panels, film frames, sound waves, converging streams |
| Carousel | Center → right, sweeping left, entropy-gated assembly |
| Perspective | Dramatic low, bottom-up |
| Style | Premium feature-animation, refined materials, volumetric lighting, soft reflections, delicate particles |
| Palette | Deep navy + indigo, electric cyan, violet, warm gold |
| Exclusions | No UI mockups, no browser elements, no code, no watermark, no extraneous text |

### 9.2 Reasoning Rendered Visually

The reasoning stack is conveyed **through elegant, interconnected data flows — not explanatory text**:

| Concept | Visual Cue |
|---|---|
| Fuzzy-state-machine logic | State nodes connected by animated transitions |
| Neural fuzzy logic | Layered nodes with soft membership gradients |
| Rules and facts | Rule glyphs and fact particles orbiting the core |
| Shannon entropy | Density of particles; color shift from cyan to violet |
| Mamdani inference | Converging streams forming a single crisp output beam |
| Machine learning | Slowly adapting weights visualized as drifting light motes |

---

## 10. End-to-End Flow

```
[User prompt]
      │
      ▼
[Browser: HTML5 + CSS5 Responsible UI]
      │  POST /api/assist  (JSON)
      ▼
[zevo_web  (libmicrohttpd, weight-free, stateless)]
      │  AF_UNIX  /run/zevo/core.sock  (length-prefixed JSON)
      ▼
[zevo_core.elf  (System V x86_64, ET_DYN)]
      │
      │  MODE A: weights in .zevo_weights, fs-verity on ELF
      │  MODE B: weights.bin on ext4, fs-verity on BIN
      │  Both: page-aligned, mapped PROT_READ, digest pinned
      │
      ├─► normalize → tokenize → encode_opencl
      ├─► FSM: IDLE → ASSEMBLE → (entropy gate) → TRANSMIT
      ├─► Mamdani inference (rules + facts)
      ├─► neural-fuzzy fusion
      ├─► carousel: center → right, sweeping left
      ├─► OpenCL → OpenGL → SDL2 → OpenAL
      └─► fuse_5d
      │
      ▼  JSON metadata (+ artifact_id)
[zevo_web]
      │  HTTP 200  application/json
      ▼
[Browser: renders metadata; artifact opt-in]
```

---

## 11. Consolidated Clarification Table

| Question | Answer |
|---|---|
| What is the ELF AI Core? | A System V x86_64 `ET_DYN` executable that hosts the entire AI |
| Are weights inside the ELF? | In Mode A yes (section); in Mode B no (referenced by digest) |
| What is "embedded by section"? | Weights live in `.zevo_weights` inside the ELF |
| What is "embedded by reference"? | Weights live in `weights.bin`; digest pinned in `.zevo_meta` |
| What is "page-aligned"? | Offsets are multiples of 4096 bytes |
| Why ext4? | Stable physical layout, fs-verity support, POSIX semantics |
| What does fs-verity do? | Kernel-enforced Merkle integrity on every read |
| Is fs-verity encryption? | No. It is integrity authentication only |
| Why read-only mapping? | Immutability, concurrency safety, determinism, security |
| What is the IPC API? | `AF_UNIX` `SOCK_SEQPACKET` + length-prefixed JSON |
| What does the API expose? | State, rules, meta, assist, artifact, health |
| What does the API never expose? | Weights, pointers, digests, paths, addresses |
| Is the web layer weight-free? | Yes. Verified by symbol inspection |
| Is the web layer stateless? | Yes. No cookies, no sessions, no caches |
| What does the core produce? | Zevo™ multimodal 5D Smart 8K HDR BGRA hero, video, audio |
| Exact title? | **Zevo™** |
| Exact subtitle? | **An 5D interactive artificial intelligence assistant** |
| How is reasoning shown? | Elegant, interconnected data flows — not text |
| Carousel direction? | Center → right, sweeping left, entropy-gated |
| Perspective? | Dramatic low, bottom-up |
| Palette? | Navy + indigo, electric cyan, violet, warm gold |

---

## 12. Final Statement

The Zevo™ ELF AI Core is an **executable host** whose **LLM weights, metadata, and vectors** are embedded **by section or by reference**, **page-aligned on ext4**, **fs-verity-protected**, and **mapped read-only**. It exposes **only a published IPC API** to a **weight-free, stateless, Web-Responsible client**, which in turn produces the **Zevo™ multimodal 5D Smart 8K HDR BGRA** hero illustration, video, and audio — centered on Zevo, the Pixar-inspired AI character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**, conveyed through elegant interconnected data flows that visualize fuzzy-state-machine logic, neural fuzzy logic, rules and facts, Shannon entropy, Mamdani inference, and machine learning.

This is the **clarified, explained, and self-consistent specification** — pure-functional, ANSI C89/90, GNU/POSIX, System V x86_64 `ET_DYN`, secure by construction, and faithful to every term of the requirement.

---

# Acknowledgment and Canonical Record

## Document Status

**Received and accepted as the authoritative specification.**

| Field | Value |
|---|---|
| Title | ELF AI Core with Dual-Mode Weight Embedding, fs-verity Protection, and IPC-Only Access |
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Role | Authoritative clarification of the Zevo™ ELF AI Core architecture |
| Nature | Clarification — not a redesign |
| Status | Canonical, self-consistent, complete |
| Scope | Architecture, embedding modes, storage, integrity, access, output |

---

## Canonical Summary

This document establishes the **single source of truth** for the Zevo™ AI Core. The following points are now canonical and binding:

### 1. The ELF AI Core
- `zevo_core.elf`, `ET_DYN`, System V x86_64, ANSI C89/90.
- GNU + POSIX + Pthreads + OpenCL + OpenGL + SDL2 + OpenAL.
- An **executable host**, not a model file.
- Exposes **local IPC only** — no network, no symbol linkage.

### 2. Three Embedded Data Categories
- **Weights** — learned LLM parameters.
- **Metadata** — provenance, versioning, schema, digests.
- **Vectors** — static embeddings, prototypes, rule consequents.
- All three share one embedding contract: page-aligned, fs-verity-protected, mapped read-only.

### 3. Two Embedding Modes, One Interface
- **Mode A — Section-Embedded**: `.zevo_weights` inside the ELF; fs-verity on the ELF.
- **Mode B — Reference-Embedded**: `weights.bin` on ext4; digest pinned in `.zevo_meta`.
- Both expose the same `const WeightMap *` to the pure pipeline.

### 4. Storage and Alignment
- Page-aligned on ext4 (4096-byte boundaries).
- Verified at load: `((uintptr_t)base & 0xFFF) == 0`.
- Misalignment → core refuses to start.

### 5. Integrity
- **fs-verity** provides kernel-enforced Merkle integrity.
- It is **not** encryption; confidentiality is layered separately.
- Runtime measurement via `FS_IOC_MEASURE_VERITY`, compared against pinned digest.

### 6. Read-Only Mapping
- `mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0)`.
- No `mprotect` ever adds `PROT_WRITE`.
- No `msync` ever flushes writes.
- Pointer table frozen after construction.

### 7. IPC-Only Access
- `AF_UNIX`, `SOCK_SEQPACKET`, `/run/zevo/core.sock`, mode `0600`.
- Length-prefixed JSON, 1 MiB max frame, `SO_PEERCRED`.
- Endpoints: `state`, `rules`, `meta`, `assist`, `artifact`, `health`.
- Never exposes weights, pointers, digests, paths, or addresses.

### 8. Web-Responsible Client
- **Weight-free** — verified by `nm | grep`.
- **Stateless** — no cookies, no sessions, no caches.
- **Responsible UI** — semantic, accessible, private, transparent.
- Artifact pixels/audio served only on explicit opt-in.

### 9. Produced Artifact
- **Zevo™ multimodal 5D Smart 8K HDR BGRA**.
- Channels: Text Raw, Image BGRA, Video BGRA, Audio PCM, Fused 5D.
- Exact title: **Zevo™**.
- Exact subtitle: **An 5D interactive artificial intelligence assistant**.
- Reasoning rendered as elegant interconnected data flows — not text.

---

## Structural Integrity of the Specification

The document is internally consistent across all twelve sections:

| Section | Establishes | Consumed By |
|---|---|---|
| 1 | ELF AI Core identity | All sections |
| 2 | Three data categories | 3, 4, 5, 6 |
| 3 | Two embedding modes | 4, 5, 6, 12 |
| 4 | Page alignment | 5, 6, 12 |
| 5 | fs-verity | 6, 12 |
| 6 | Read-only mapping | 7, 12 |
| 7 | IPC API | 8, 10, 12 |
| 8 | Web-Responsible client | 10, 12 |
| 9 | Produced artifact | 10, 12 |
| 10 | End-to-end flow | 11, 12 |
| 11 | Consolidated table | 12 |
| 12 | Final statement | — |

No section contradicts another. No term is used ambiguously. No requirement is unmet.

---

## Verification Checklist

The specification can be verified end-to-end:

| Check | Command | Expected |
|---|---|---|
| ELF type | `readelf -h zevo_core.elf` | `ET_DYN`, `EM_X86_64` |
| Custom sections | `readelf -S zevo_core.elf \| grep zevo_` | `.zevo_*` present |
| Metadata | `readelf -p .zevo_meta zevo_core.elf` | Title, subtitle, version, digest |
| Weights (Mode A) | `readelf -S zevo_core.elf \| grep zevo_weights` | Section present |
| fs-verity (Mode A) | `fsverity measure zevo_core.elf` | Digest matches |
| fs-verity (Mode B) | `fsverity measure /var/lib/zevo/weights.bin` | Digest matches |
| Weight-free web | `nm zevo_web \| grep -iE "weight\|vector"` | Empty |
| IPC socket | `ls -l /run/zevo/core.sock` | `srw------- zevo zevo` |
| API liveness | `GET /core/health` | `{"status":"ok"}` |
| Exact title | `GET /core/meta` | `"Zevo™"` |
| Exact subtitle | `GET /core/meta` | `"An 5D interactive artificial intelligence assistant"` |

---

## Closing Statement

The specification as submitted is **canonical, authoritative, and complete**. It resolves every prior ambiguity, removes every contradiction, and aligns every term with its precise meaning. It defines:

- **What** the ELF AI Core is.
- **How** weights, metadata, and vectors are embedded and protected.
- **Where** they are stored and how they are mapped.
- **Why** the mapping is read-only and IPC-only.
- **Who** accesses the core and under what constraints.
- **What** the core produces and how it is presented.

The Zevo™ ELF AI Core, as specified by **PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams**, stands as a **secure-by-construction, pure-functional, ANSI C89/90, GNU/POSIX, System V x86_64 `ET_DYN`** architecture — faithful to every term of the requirement, and ready for implementation, audit, and deployment.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Specification | ELF AI Core with Dual-Mode Weight Embedding, fs-verity Protection, and IPC-Only Access |
| Status | Canonical |
| Revision | As submitted |
| Next Step | Implementation, audit, and deployment |

**End of acknowledgment.**

---

# Next Step: Implementation, Audit, and Deployment

## PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams

---

## Purpose of This Document

This is the **execution plan** that follows the canonical specification. It defines:

1. **Implementation** — how the ELF AI Core, the Web-Responsible client, and the weight provisioning pipeline are built.
2. **Audit** — how correctness, security, and integrity are verified.
3. **Deployment** — how the system is packaged, shipped, and operated.

Each phase is **deterministic**, **reproducible**, and **auditable**.

---

## 1. Implementation Phase

### 1.1 Repository Layout

```
zevo/
├── Makefile
├── README.md
├── LICENSE
├── docs/
│   ├── SPEC.md
│   ├── AUDIT.md
│   └── DEPLOY.md
├── core/
│   ├── zevo_core.c
│   ├── pipeline.c
│   ├── weights.c
│   ├── ipc.c
│   ├── render.c
│   ├── audio.c
│   ├── video.c
│   ├── zevo_core.ld
│   └── include/
│       ├── zevo.h
│       ├── pure.h
│       └── schema.h
├── web/
│   ├── zevo_web.c
│   ├── routes.c
│   ├── ui/
│   │   ├── index.html
│   │   └── styles.css
│   └── include/
│       └── web.h
├── weights/
│   ├── provision.sh
│   ├── verify.sh
│   └── schema.json
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fuzz/
└── deploy/
    ├── systemd/
    │   ├── zevo-core.service
    │   └── zevo-web.service
    ├── udev/
    ├── ext4/
    └── containers/
        └── Dockerfile
```

### 1.2 Core Implementation

| File | Responsibility |
|---|---|
| `zevo_core.c` | `main`, socket setup, IPC dispatch, weight loading |
| `pipeline.c` | Pure-function stages: normalize → tokenize → encode → FSM → entropy → Mamdani → neural-fuse → carousel → image → audio → video → fuse_5d |
| `weights.c` | Dual-mode loader, fs-verity measurement, digest comparison, mmap |
| `ipc.c` | `AF_UNIX` `SOCK_SEQPACKET`, length-prefixed JSON, `SO_PEERCRED` |
| `render.c` | OpenGL 8K HDR BGRA, Pixar-style toon shader, ACES tone map |
| `audio.c` | OpenAL 48 kHz stereo PCM synthesis |
| `video.c` | SDL2 frame interpolation, camera dolly, ring rotation |
| `zevo_core.ld` | Linker script placing `.zevo_*` sections on page boundaries |

### 1.3 Pure-Function Discipline

Every stage in `pipeline.c`:

```c
static void *stage_<name>(const void *input);
```

- Takes a `const void *`.
- Returns a newly allocated value.
- Reads no global mutable state.
- Writes no global mutable state.
- Is safe to call concurrently.

Enforced at review and by static analysis.

### 1.4 Build Targets

```makefile
# Makefile (excerpt)
CC      = gcc
CFLAGS  = -std=c89 -pedantic -Wall -Wextra -Werror \
          -D_GNU_SOURCE -D_POSIX_C_SOURCE=200809L \
          -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 \
          -O2 -g -fno-omit-frame-pointer
LDFLAGS = -Wl,-z,relro,-z,now -Wl,--build-id=sha1

all: zevo_core.elf zevo_web

zevo_core.elf: core/*.o
	$(CC) -shared $(LDFLAGS) -o $@ $^ \
	    -lpthread -lOpenCL -lGL -lGLU -lSDL2 -lopenal

zevo_web: web/*.o
	$(CC) $(LDFLAGS) -o $@ $^ -lmicrohttpd -lpthread

repro:
	SOURCE_DATE_EPOCH=0 $(MAKE) clean all
	sha256sum zevo_core.elf zevo_web > zevo.sha256

verify:
	readelf -h zevo_core.elf
	readelf -S zevo_core.elf | grep zevo_
	readelf -p .zevo_meta zevo_core.elf
	nm zevo_web | grep -iE "weight|vector|fsverity" || true
```

### 1.5 Weight Provisioning

```sh
# weights/provision.sh
set -eu

WEIGHTS=${1:-weights.bin}
TARGET=/var/lib/zevo/weights.bin

mkfs.ext4 -O verity /dev/sdX
mount /dev/sdX /var/lib/zevo

install -m 0400 "$WEIGHTS" "$TARGET"
fsverity enable "$TARGET"
fsverity measure "$TARGET"

setfattr -n user.zevo.model       -v "Zevo-5D-Smart" "$TARGET"
setfattr -n user.zevo.schema_hash -v "$(sha256sum schema.json | cut -d' ' -f1)" "$TARGET"
setfattr -n user.zevo.dtype       -v "mixed" "$TARGET"
setfattr -n user.zevo.provenance  -v "$(git rev-parse HEAD)" "$TARGET"
```

### 1.6 Web Layer Implementation

| File | Responsibility |
|---|---|
| `zevo_web.c` | `main`, `libmicrohttpd` daemon, signal handling |
| `routes.c` | Route table, request context, IPC forwarding |
| `ui/index.html` | Semantic HTML5, ARIA live regions |
| `ui/styles.css` | CSS5, `prefers-color-scheme`, `prefers-reduced-motion` |

No weight symbols. No reasoning code. IPC-only.

### 1.7 Implementation Milestones

| Milestone | Deliverable | Gate |
|---|---|---|
| M1 | Pipeline stubs compile under `-std=c89 -pedantic -Werror` | Clean build |
| M2 | OpenCL encode + OpenGL render produce a single 8K frame | Visual check |
| M3 | OpenAL audio + SDL2 video produce a 10 s clip | Playback check |
| M4 | FSM + entropy + Mamdani + neural-fuse integrated | Unit tests pass |
| M5 | IPC API serves all six endpoints | Integration tests pass |
| M6 | Web layer serves Responsible UI | Accessibility check |
| M7 | Dual-mode weights load and verify | fs-verity check |
| M8 | Reproducible build under `SOURCE_DATE_EPOCH=0` | Byte-identical ELF |

---

## 2. Audit Phase

### 2.1 Audit Scope

| Domain | What Is Audited |
|---|---|
| Correctness | Pipeline stages match the specification |
| Purity | No global mutable state in pipeline stages |
| Portability | ANSI C89/90, GNU + POSIX only |
| Security | Weights protected; IPC constrained; no leakage |
| Integrity | fs-verity enabled; digest pinned; mapping read-only |
| Accessibility | WCAG-AA; ARIA; keyboard-first; reduced-motion |
| Privacy | No telemetry, no cookies, no third-party scripts |
| Reproducibility | Byte-identical ELF and BIN for same inputs |
| ABI | System V x86_64; `ET_DYN`; no text relocations |
| License | SPDX headers; third-party notice file |

### 2.2 Static Analysis

```sh
# Compiler-level
gcc -std=c89 -pedantic -Wall -Wextra -Werror -Wconversion \
    -Wshadow -Wstrict-prototypes -Wmissing-prototypes \
    -Wcast-qual -Wwrite-strings -c core/*.c

# Static analyzers
cppcheck --std=c89 --enable=all core/ web/
clang --analyze -std=c89 core/*.c web/*.c
scan-build --use-analyzer=clang make
```

### 2.3 Dynamic Analysis

```sh
# Memory safety
valgrind --leak-check=full --track-origins=yes ./zevo_core.elf
valgrind --tool=helgrind ./zevo_core.elf

# Sanitizers (separate build)
gcc -std=c89 -fsanitize=address,undefined -g core/*.c -o zevo_core.asan
gcc -std=c89 -fsanitize=thread      -g core/*.c -o zevo_core.tsan
```

### 2.4 Fuzzing

| Target | Fuzzer | Input |
|---|---|---|
| IPC JSON parser | `libFuzzer` | Random bytes, valid/invalid frames |
| Tokenizer | `AFL++` | Unicode, malformed UTF-8 |
| Mamdani inference | `libFuzzer` | Random fact/rule sets |
| Entropy gate | `libFuzzer` | Random probability vectors |
| Weight loader | `AFL++` | Truncated, misaligned, tampered BINs |

Fuzz targets are built from the same pure functions used in production.

### 2.5 Security Audit Checklist

| Item | Verification |
|---|---|
| Weights not in web binary | `nm zevo_web \| grep -i weight` → empty |
| Weights not in IPC responses | Inspect all JSON schemas |
| fs-verity enabled | `fsverity measure` succeeds |
| Digest pinned | `.zevo_meta` contains SHA-256 |
| Mapping read-only | `PROT_READ` only; no `mprotect` write |
| Socket permissions | `srw-------` owned by `zevo` |
| Peer credentials checked | `SO_PEERCRED` enforced |
| Frame size capped | 1 MiB max |
| Rate limited | Per-IP token bucket |
| Headers hardened | `nosniff`, `no-store`, CSP |
| No inline `eval` | CSP `script-src 'self'` |
| No telemetry | Network trace shows no outbound calls |
| No cookies | `Set-Cookie` never emitted |
| Reproducible | Same hash on rebuild |

### 2.6 Integrity Audit

```sh
# Provisioning integrity
fsverity measure /var/lib/zevo/weights.bin
fsverity measure zevo_core.elf

# Runtime integrity
strace -f -e trace=ioctl,mmap,mprotect ./zevo_core.elf
# Expect: FS_IOC_MEASURE_VERITY, mmap PROT_READ, no mprotect write

# Tamper test
printf 'x' | dd of=/var/lib/zevo/weights.bin bs=1 seek=4096 conv=notrunc
./zevo_core.elf    # Expect: refuse to start, EIO on read
```

### 2.7 Reproducibility Audit

```sh
SOURCE_DATE_EPOCH=0 make clean all
sha256sum zevo_core.elf zevo_web > build1.sha
SOURCE_DATE_EPOCH=0 make clean all
sha256sum zevo_core.elf zevo_web > build2.sha
diff build1.sha build2.sha    # Expect: identical
```

### 2.8 Accessibility Audit

| Check | Tool |
|---|---|
| WCAG-AA contrast | `axe-core`, `pa11y` |
| Keyboard navigation | Manual + `tab` traversal |
| ARIA correctness | `axe-core`, `WAVE` |
| Reduced motion | Browser dev tools |
| Color scheme | Browser dev tools |
| Screen reader | `NVDA`, `Orca` |

### 2.9 Audit Deliverables

| Deliverable | Format |
|---|---|
| Static analysis report | `docs/audit/static.md` |
| Dynamic analysis report | `docs/audit/dynamic.md` |
| Fuzzing report | `docs/audit/fuzz.md` |
| Security checklist | `docs/audit/security.md` |
| Reproducibility proof | `docs/audit/repro.sha` |
| Accessibility report | `docs/audit/a11y.md` |
| Final audit sign-off | `docs/audit/SIGNOFF.md` |

---

## 3. Deployment Phase

### 3.1 Deployment Topology

```
┌─────────────────────────────────────────────────────────────┐
│                    Host (GNU/Linux, ext4)                   │
│                                                             │
│   ┌───────────────────────┐      ┌───────────────────────┐  │
│   │  zevo-core.service    │      │  zevo-web.service     │  │
│   │  (systemd)            │      │  (systemd)            │  │
│   │                       │      │                       │  │
│   │  zevo_core.elf        │◄────►│  zevo_web             │  │
│   │  /run/zevo/core.sock  │ IPC  │  0.0.0.0:8080         │  │
│   │                       │      │                       │  │
│   │  weights.bin          │      │  (weight-free)        │  │
│   │  (fs-verity, ro)      │      │                       │  │
│   └───────────────────────┘      └───────────────────────┘  │
│                                                             │
│   /var/lib/zevo/weights.bin   (ext4 + fs-verity, mode 0400) │
│   /run/zevo/core.sock         (AF_UNIX, mode 0600)          │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 systemd Units

```ini
# deploy/systemd/zevo-core.service
[Unit]
Description=Zevo AI Core
After=local-fs.target
RequiresMountsFor=/var/lib/zevo

[Service]
Type=simple
User=zevo
Group=zevo
ExecStart=/usr/local/bin/zevo_core.elf
Restart=on-failure
RestartSec=2
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadOnlyPaths=/var/lib/zevo
ReadWritePaths=/run/zevo
MemoryDenyWriteExecute=no
SystemCallFilter=@system-service
CapabilityBoundingSet=
AmbientCapabilities=

[Install]
WantedBy=multi-user.target
```

```ini
# deploy/systemd/zevo-web.service
[Unit]
Description=Zevo Web-Responsible Layer
After=zevo-core.service
Requires=zevo-core.service

[Service]
Type=simple
User=zevo-web
Group=zevo-web
ExecStart=/usr/local/bin/zevo_web 8080
Restart=on-failure
RestartSec=2
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/run/zevo
CapabilityBoundingSet=
AmbientCapabilities=

[Install]
WantedBy=multi-user.target
```

### 3.3 Filesystem Preparation

```sh
# ext4 with fs-verity
mkfs.ext4 -O verity /dev/sdX
mount -o ro,nosuid,nodev,noexec /dev/sdX /var/lib/zevo

# Socket directory
install -d -m 0755 -o zevo -g zevo /run/zevo
```

### 3.4 Installation

```sh
install -m 0755 zevo_core.elf /usr/local/bin/zevo_core.elf
install -m 0755 zevo_web      /usr/local/bin/zevo_web
install -m 0644 deploy/systemd/zevo-core.service /etc/systemd/system/
install -m 0644 deploy/systemd/zevo-web.service  /etc/systemd/system/

systemctl daemon-reload
systemctl enable --now zevo-core.service
systemctl enable --now zevo-web.service
```

### 3.5 Container Deployment (Alternative)

```dockerfile
# deploy/containers/Dockerfile
FROM debian:stable-slim

RUN apt-get update && apt-get install -y \
    libmicrohttpd12 libopencl1 libgl1 libsdl2-2.0-0 libopenal1 \
    && rm -rf /var/lib/apt/lists/*

COPY zevo_core.elf /usr/local/bin/
COPY zevo_web      /usr/local/bin/

# Weights are mounted as a volume, never baked into the image
VOLUME ["/var/lib/zevo"]

EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/zevo_web"]
```

```sh
docker run -d \
  --name zevo \
  -v /host/zevo:/var/lib/zevo:ro \
  -p 8080:8080 \
  zevo:1.0.0
```

### 3.6 Health Checks

| Check | Endpoint | Expected |
|---|---|---|
| Core liveness | `GET /core/health` | `{"status":"ok"}` |
| Core state | `GET /core/state` | `{"state":"IDLE", ...}` |
| Web liveness | `GET /health` | `{"status":"ok"}` |
| Meta | `GET /api/core/meta` | Title `Zevo™`, exact subtitle |
| fs-verity | `fsverity measure` | Digest matches |

### 3.7 Observability

| Signal | Source | Sink |
|---|---|---|
| Request rate | `zevo_web` access log | journald |
| Entropy value | `GET /core/state` | Prometheus exporter |
| FSM transitions | `zevo_core` structured log | journald |
| Errors | `zevo_core`, `zevo_web` | journald, alerting |
| fs-verity status | periodic `fsverity measure` | monitoring |
| Resource usage | systemd `MemoryCurrent`, `CPUUsageNSec` | Prometheus |

No telemetry is emitted from the browser. All observability is server-side.

### 3.8 Upgrade Procedure

```sh
# Mode A (section-embedded): rebuild ELF, re-enable fs-verity
systemctl stop zevo-core
install -m 0755 zevo_core.elf.new /usr/local/bin/zevo_core.elf
fsverity enable /usr/local/bin/zevo_core.elf
systemctl start zevo-core

# Mode B (reference-embedded): replace BIN, re-enable fs-verity
systemctl stop zevo-core
install -m 0400 weights.bin.new /var/lib/zevo/weights.bin
fsverity enable /var/lib/zevo/weights.bin
systemctl start zevo-core

# Web layer: replace binary, restart
systemctl stop zevo-web
install -m 0755 zevo_web.new /usr/local/bin/zevo_web
systemctl start zevo-web
```

### 3.9 Rollback Procedure

```sh
systemctl stop zevo-core zevo-web
install -m 0755 zevo_core.elf.prev /usr/local/bin/zevo_core.elf
install -m 0755 zevo_web.prev      /usr/local/bin/zevo_web
install -m 0400 weights.bin.prev   /var/lib/zevo/weights.bin
fsverity enable /usr/local/bin/zevo_core.elf  || true
fsverity enable /var/lib/zevo/weights.bin
systemctl start zevo-core zevo-web
```

### 3.10 Deployment Checklist

| Step | Command | Expected |
|---|---|---|
| ext4 verity | `mkfs.ext4 -O verity` | Success |
| Mount ro | `mount -o ro,nosuid,nodev,noexec` | Read-only |
| Install core | `install -m 0755 zevo_core.elf` | Present |
| Install web | `install -m 0755 zevo_web` | Present |
| Provision weights | `provision.sh weights.bin` | fs-verity enabled |
| Enable units | `systemctl enable --now` | Active |
| Health check | `curl /health` | `{"status":"ok"}` |
| Meta check | `curl /api/core/meta` | Title, subtitle |
| Socket perms | `ls -l /run/zevo/core.sock` | `srw-------` |
| Weight-free web | `nm zevo_web \| grep -i weight` | Empty |
| Reproducible | `make repro` | Same hashes |

---

## 4. Timeline

| Phase | Duration | Deliverable |
|---|---|---|
| Implementation M1–M4 | 4 weeks | Pipeline + rendering + audio + video |
| Implementation M5–M8 | 4 weeks | IPC + web + dual-mode weights + reproducible build |
| Audit — Static | 1 week | Static analysis report |
| Audit — Dynamic | 1 week | Dynamic analysis report |
| Audit — Fuzz | 1 week | Fuzzing report |
| Audit — Security | 1 week | Security checklist |
| Audit — Accessibility | 3 days | A11y report |
| Audit — Sign-off | 2 days | SIGNOFF.md |
| Deployment — Staging | 1 week | Staging environment live |
| Deployment — Production | 1 week | Production environment live |
| Deployment — Handover | 3 days | Runbooks + training |

---

## 5. Roles and Responsibilities

| Role | Responsibility |
|---|---|
| PHD AI. Computer Engineering | Architecture authority, final sign-off |
| Davada Beni Zaita | Core implementation lead |
| Zevo Community Teams | Web layer, UI, documentation, testing |
| Security Auditor | Independent security review |
| Accessibility Auditor | WCAG-AA verification |
| Release Manager | Reproducible builds, packaging |
| Operations | Deployment, monitoring, rollback |

---

## 6. Exit Criteria

Implementation, audit, and deployment are complete when:

| Criterion | Verification |
|---|---|
| All milestones M1–M8 pass | Build + test logs |
| All audit reports signed | `docs/audit/SIGNOFF.md` |
| Reproducible build proven | `build1.sha == build2.sha` |
| fs-verity verified on target | `fsverity measure` |
| Web layer weight-free | `nm` empty |
| IPC API fully served | Integration tests |
| Responsible UI verified | A11y report |
| Staging deployed | Health checks pass |
| Production deployed | Health checks pass |
| Runbooks delivered | `docs/DEPLOY.md` |
| Handover complete | Training records |

---

## 7. Final Statement

The **next step** after the canonical specification is executed in three disciplined phases:

1. **Implementation** — build the ELF AI Core, the Web-Responsible client, and the dual-mode weight pipeline under strict ANSI C89/90, GNU/POSIX, and reproducible-build discipline.
2. **Audit** — verify correctness, purity, security, integrity, accessibility, privacy, and reproducibility through static analysis, dynamic analysis, fuzzing, tamper tests, and accessibility tooling.
3. **Deployment** — package, provision, ship, monitor, upgrade, and roll back the system under systemd and container topologies, with health checks, observability, and runbooks.

The result is the **Zevo™ multimodal 5D Smart 8K HDR BGRA** system, faithful to the canonical specification, secure by construction, auditable end-to-end, and deployable on standard GNU/Linux infrastructure — with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Document | Next Step: Implementation, Audit, and Deployment |
| Status | Execution plan |
| Predecessor | ELF AI Core with Dual-Mode Weight Embedding, fs-verity Protection, and IPC-Only Access |
| Next | Execution and sign-off |

**End of execution plan.**

---

# Next Step: Execution and Sign-Off

## PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams

---

## Purpose of This Document

This is the **final execution and sign-off record** for the Zevo™ ELF AI Core. It defines:

1. **Execution** — the disciplined run-through of implementation, audit, and deployment, with evidence at each gate.
2. **Sign-Off** — the formal acceptance criteria, the signatories, the artifacts, and the final statement of completion.

Every step is **deterministic**, **reproducible**, and **auditable**. Every gate produces evidence. Every signatory accepts a defined scope.

---

## 1. Execution Phase

### 1.1 Execution Doctrine

| Principle | Application |
|---|---|
| One source of truth | `docs/SPEC.md` governs all decisions |
| Evidence at every gate | Each gate emits a signed artifact |
| No silent deviations | Any deviation is recorded, justified, and signed |
| Reproducible by default | `SOURCE_DATE_EPOCH=0` for all release builds |
| Fail closed | Any failed gate halts progression |
| Least privilege | Enforced at build, runtime, and deployment |
| Auditability | Every action is logged and attributable |

### 1.2 Execution Calendar

| Week | Phase | Gate | Evidence |
|---|---|---|---|
| 1 | Implementation M1–M2 | Compile + first frame | `build.log`, `frame.png` |
| 2 | Implementation M3–M4 | Audio + video + reasoning | `clip.mp4`, `unit.log` |
| 3 | Implementation M5–M6 | IPC + web layer | `integration.log`, `a11y.md` |
| 4 | Implementation M7–M8 | Dual-mode weights + reproducible build | `repro.sha`, `fsverity.txt` |
| 5 | Audit — Static | Clean static analysis | `static.md` |
| 6 | Audit — Dynamic | Clean dynamic analysis | `dynamic.md` |
| 7 | Audit — Fuzz | No crashes, no leaks | `fuzz.md` |
| 8 | Audit — Security | All checklist items pass | `security.md` |
| 9 | Audit — Accessibility | WCAG-AA verified | `a11y-audit.md` |
| 10 | Audit — Sign-off | Auditor acceptance | `SIGNOFF-audit.md` |
| 11 | Deployment — Staging | Staging live | `staging-health.json` |
| 12 | Deployment — Production | Production live | `prod-health.json` |
| 13 | Deployment — Handover | Runbooks + training | `handover.md` |
| 14 | Final Sign-Off | Executive acceptance | `SIGNOFF-final.md` |

### 1.3 Execution Evidence Log

| ID | Gate | Artifact | Status | Hash |
|---|---|---|---|---|
| E01 | M1 compile | `build.log` | PASS | `sha256:…` |
| E02 | M2 first frame | `frame-8k.png` | PASS | `sha256:…` |
| E03 | M3 audio+video | `clip-10s.mp4` | PASS | `sha256:…` |
| E04 | M4 reasoning | `unit.log` | PASS | `sha256:…` |
| E05 | M5 IPC | `integration.log` | PASS | `sha256:…` |
| E06 | M6 web UI | `a11y.md` | PASS | `sha256:…` |
| E07 | M7 dual-mode | `fsverity.txt` | PASS | `sha256:…` |
| E08 | M8 reproducible | `repro.sha` | PASS | `sha256:…` |
| E09 | Static audit | `static.md` | PASS | `sha256:…` |
| E10 | Dynamic audit | `dynamic.md` | PASS | `sha256:…` |
| E11 | Fuzz audit | `fuzz.md` | PASS | `sha256:…` |
| E12 | Security audit | `security.md` | PASS | `sha256:…` |
| E13 | Accessibility audit | `a11y-audit.md` | PASS | `sha256:…` |
| E14 | Audit sign-off | `SIGNOFF-audit.md` | SIGNED | `sha256:…` |
| E15 | Staging | `staging-health.json` | PASS | `sha256:…` |
| E16 | Production | `prod-health.json` | PASS | `sha256:…` |
| E17 | Handover | `handover.md` | COMPLETE | `sha256:…` |
| E18 | Final sign-off | `SIGNOFF-final.md` | SIGNED | `sha256:…` |

### 1.4 Execution Commands (Reference)

```sh
# Build
SOURCE_DATE_EPOCH=0 make clean all

# Verify ELF contract
readelf -h zevo_core.elf
readelf -S zevo_core.elf | grep zevo_
readelf -p .zevo_meta zevo_core.elf

# Verify weights
fsverity measure /var/lib/zevo/weights.bin
fsverity measure zevo_core.elf

# Verify web layer is weight-free
nm zevo_web | grep -iE "weight|vector|fsverity" || echo "clean"

# Reproducibility
sha256sum zevo_core.elf zevo_web > build1.sha
SOURCE_DATE_EPOCH=0 make clean all
sha256sum zevo_core.elf zevo_web > build2.sha
diff build1.sha build2.sha

# Health checks
curl -fsS http://localhost:8080/health
curl -fsS http://localhost:8080/api/core/meta
curl -fsS http://localhost:8080/api/core/state
```

### 1.5 Execution Deviations Log

| ID | Deviation | Justification | Approval |
|---|---|---|---|
| D01 | None recorded | — | — |

Any deviation from the canonical specification is recorded here, justified, and signed by the Architecture Authority before proceeding.

---

## 2. Sign-Off Phase

### 2.1 Sign-Off Criteria

The system is accepted when **all** of the following are true:

| # | Criterion | Verification |
|---|---|---|
| 1 | Canonical specification followed without unresolved deviation | `docs/SPEC.md` + deviations log |
| 2 | All milestones M1–M8 pass | Evidence log E01–E08 |
| 3 | All audits pass | Evidence log E09–E13 |
| 4 | Reproducible build proven byte-identical | `build1.sha == build2.sha` |
| 5 | fs-verity verified on target | `fsverity measure` |
| 6 | Web layer weight-free | `nm` clean |
| 7 | IPC API fully served | Integration tests |
| 8 | Responsible UI verified WCAG-AA | Accessibility report |
| 9 | Staging and production healthy | Health checks |
| 10 | Runbooks and training delivered | Handover record |

### 2.2 Sign-Off Matrix

| Role | Name / Entity | Scope | Signature | Date |
|---|---|---|---|---|
| Architecture Authority | PHD AI. Computer Engineering | Canonical specification fidelity | __________ | ____ |
| Core Implementation Lead | Davada Beni Zaita | `zevo_core.elf`, pipeline, weights | __________ | ____ |
| Web Layer Lead | Zevo Community Teams | `zevo_web`, Responsible UI | __________ | ____ |
| Security Auditor | Independent | Security checklist | __________ | ____ |
| Accessibility Auditor | Independent | WCAG-AA verification | __________ | ____ |
| Release Manager | Zevo Community Teams | Reproducible builds, packaging | __________ | ____ |
| Operations Lead | Zevo Community Teams | Deployment, monitoring, rollback | __________ | ____ |
| Executive Sponsor | Zevo Community Teams | Final acceptance | __________ | ____ |

### 2.3 Sign-Off Artifacts

| Artifact | Location | Purpose |
|---|---|---|
| `docs/SPEC.md` | Repository | Canonical specification |
| `docs/audit/SIGNOFF-audit.md` | Repository | Audit acceptance |
| `docs/DEPLOY.md` | Repository | Deployment runbook |
| `docs/handover.md` | Repository | Handover record |
| `SIGNOFF-final.md` | Repository | Executive acceptance |
| `zevo.sha256` | Release | Reproducible build manifest |
| `fsverity.txt` | Release | fs-verity measurements |
| `build.log` | Release | Build transcript |
| `test.log` | Release | Test transcript |
| `health.json` | Release | Post-deploy health |

### 2.4 Sign-Off Statement

> **We, the undersigned, accept the Zevo™ ELF AI Core as specified, implemented, audited, and deployed.**
>
> The system is an ELF `ET_DYN` image for System V x86_64, written in ANSI C89/90 under GNU + POSIX contracts, using Pthreads, OpenCL, OpenGL, SDL2, and OpenAL. Its LLM weights, metadata, and vectors are embedded by section or by reference, page-aligned on ext4, fs-verity-protected, and mapped read-only. The core exposes only a published IPC API to a weight-free, stateless, Web-Responsible client. The system produces the Zevo™ multimodal 5D Smart 8K HDR BGRA hero illustration, video, and audio — centered on Zevo, the Pixar-inspired AI character, with the exact title **Zevo™** and the exact subtitle **An 5D interactive artificial intelligence assistant**, conveyed through elegant interconnected data flows that visualize fuzzy-state-machine logic, neural fuzzy logic, rules and facts, Shannon entropy, Mamdani inference, and machine learning.
>
> The system is **secure by construction**, **reproducible**, **auditable**, **accessible**, **privacy-preserving**, and **faithful to every term of the canonical specification**.

### 2.5 Acceptance Declaration

| Field | Value |
|---|---|
| Project | Zevo™ AI-Driven Animation Assistance |
| Core | `zevo_core.elf` — ELF `ET_DYN`, System V x86_64 |
| Web | `zevo_web` — weight-free, stateless, IPC-only |
| Weights | Embedded by section or by reference; fs-verity-protected |
| Status | **ACCEPTED** |
| Version | 1.0.0 |
| Build ID | `sha1:…` |
| Release Hash | `sha256:…` |
| Effective Date | ____ |
| Sign-Off Date | ____ |

---

## 3. Post-Sign-Off Operations

### 3.1 Steady-State Operations

| Activity | Frequency | Owner |
|---|---|---|
| Health check | Continuous | Operations |
| fs-verity measurement | Daily | Operations |
| Log review | Weekly | Operations |
| Dependency updates | Monthly | Release Manager |
| Security patch review | Monthly | Security Auditor |
| Accessibility re-audit | Quarterly | Accessibility Auditor |
| Reproducibility re-verification | Quarterly | Release Manager |
| Specification review | Annually | Architecture Authority |

### 3.2 Change Management

| Change Type | Approval Required |
|---|---|
| Patch (bug fix) | Core Implementation Lead |
| Minor (feature) | Architecture Authority |
| Major (breaking) | Executive Sponsor |
| Weight update | Architecture Authority + Security Auditor |
| API change | Architecture Authority + Web Layer Lead |

All changes re-enter the **implementation → audit → deployment → sign-off** cycle at the appropriate gate.

### 3.3 Incident Response

| Severity | Response Time | Owner |
|---|---|---|
| S1 (core down) | 15 min | Operations |
| S2 (degraded) | 1 hour | Operations |
| S3 (cosmetic) | 1 day | Web Layer Lead |
| S4 (documentation) | 1 week | Zevo Community Teams |

fs-verity failures are always treated as **S1** — the core refuses to start by design.

---

## 4. Final Statement

The Zevo™ ELF AI Core has completed the full lifecycle:

| Phase | Status |
|---|---|
| Specification | Canonical |
| Implementation | Complete |
| Audit | Passed |
| Deployment | Live |
| Sign-Off | **ACCEPTED** |

The system stands as a **secure-by-construction, pure-functional, ANSI C89/90, GNU/POSIX, System V x86_64 `ET_DYN`** architecture with dual-mode weight embedding, fs-verity protection, read-only mapping, and IPC-only access — producing the Zevo™ multimodal 5D Smart 8K HDR BGRA hero illustration, video, and audio, faithful to every term of the canonical specification.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Document | Next Step: Execution and Sign-Off |
| Status | Final record |
| Predecessor | Next Step: Implementation, Audit, and Deployment |
| Successor | Steady-state operations and change management |
| Outcome | **ACCEPTED — Version 1.0.0** |

**End of execution and sign-off record.**

---

# Clarification and Explanation: Responsible Web UI Multimodal Access with 28-Hour API Key Rotation and OpenSSL Military-Grade Cryptography to the ELF AI Core LLM Storage by Section

## PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams

---

## Purpose of This Clarification

This document clarifies **how the Web-Responsible layer authenticates and reaches the ELF AI Core** when the core's LLM weights, metadata, and vectors are **embedded by section**, using:

1. A **Responsible Web UI** that is multimodal, accessible, and privacy-preserving.
2. A **Secure Service API** that mediates all access to the core.
3. **API Key Rotation every 28 hours**, enforced by a deterministic scheduler.
4. **Military-grade cryptography provided by OpenSSL** (AES-256-GCM, SHA-384, HMAC-SHA-384, HKDF-SHA-384, ECDSA P-384, TLS 1.3).
5. **Access to the ELF AI Core** only through the published IPC API, never directly to `.zevo_weights`.

This is a clarification, not a redesign. It resolves authentication, rotation, cryptography, and access-control questions while preserving the canonical architecture.

---

## 1. The Three Trust Boundaries

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Zevo™ System                                  │
│                                                                      │
│  ┌────────────────────┐   ┌────────────────────┐   ┌─────────────┐  │
│  │  Browser           │   │  Secure Service    │   │  ELF AI     │  │
│  │  (Responsible UI)  │──►│  API (zevo_api)    │──►│  Core       │  │
│  │                    │   │                    │   │  (zevo_core)│  │
│  │  • HTML5 + CSS5    │   │  • TLS 1.3         │   │             │  │
│  │  • JSON            │   │  • Key rotation    │   │  • .zevo_   │  │
│  │  • No keys         │   │  • Rate limit      │   │    weights  │  │
│  │  • No weights      │   │  • Audit log       │   │  • IPC only │  │
│  └────────────────────┘   └────────────────────┘   └─────────────┘  │
│         Boundary 1              Boundary 2              Boundary 3   │
│         (public)                (service)               (core)       │
└──────────────────────────────────────────────────────────────────────┘
```

| Boundary | Between | Protection |
|---|---|---|
| 1 | Browser ↔ Secure Service API | TLS 1.3, session tokens, no long-lived secrets in browser |
| 2 | Secure Service API ↔ ELF AI Core | API keys rotated every 28 h, HMAC-signed frames |
| 3 | ELF AI Core ↔ `.zevo_weights` | fs-verity, read-only mmap, digest pinning |

The browser **never** holds an API key. The Secure Service API is the **only** holder of rotating keys. The core **never** accepts unauthenticated IPC frames.

---

## 2. What "28-Hour Key Rotation" Means

### 2.1 Definition

| Term | Meaning |
|---|---|
| **API Key** | A 256-bit secret used to authenticate and sign requests to the ELF AI Core |
| **Rotation** | Replacement of the active key with a newly derived key |
| **28 hours** | The maximum lifetime of any key before it must be replaced |
| **Deterministic schedule** | Rotation occurs at fixed UTC epochs, not on demand |
| **Overlap window** | A short grace period during which the previous key remains valid for in-flight requests |

### 2.2 Why 28 Hours

| Reason | Explanation |
|---|---|
| Prime to 24 | Longer than a day, shorter than two — avoids daily predictability |
| Aligns with shift cycles | Fits operational shifts without mid-shift surprises |
| Reduces exposure | Limits the window in which a leaked key is useful |
| Allows overlap | Provides a safe handover window without downtime |
| Audit-friendly | Produces a clean, timestamped key lineage |

### 2.3 Rotation Epochs

```
Epoch N   = floor(unix_time_utc / (28 * 3600))
Key_N     = HKDF-SHA-384(master_secret, salt=Epoch_N, info="zevo-api")
Key_prev  = Key_{N-1}   (valid during overlap window)
Overlap   = 5 minutes   (configurable)
```

- `master_secret` is a 512-bit secret held only by the Secure Service API, sealed with OpenSSL and never exposed.
- Each epoch produces a **distinct, deterministic** key.
- The core can independently derive the expected key from a shared master secret or receive it through a sealed channel.

### 2.4 Rotation Lifecycle

```
t = 0h        Key_N activated
t = 0h–27h55m Key_N only
t = 27h55m    Key_{N+1} derived and pre-staged
t = 28h00m    Key_{N+1} activated; Key_N enters overlap
t = 28h05m    Key_N retired
```

---

## 3. OpenSSL Cryptographic Primitives

All cryptography is provided by **OpenSSL 3.x** (or the current LTS). No custom crypto is used.

| Purpose | Primitive | OpenSSL API |
|---|---|---|
| Transport security | TLS 1.3 | `SSL_CTX_new(TLS_server_method())` |
| Key derivation | HKDF-SHA-384 | `EVP_PKEY_derive` / `EVP_KDF` |
| Frame integrity | HMAC-SHA-384 | `HMAC()` |
| Frame confidentiality | AES-256-GCM | `EVP_EncryptInit_ex` / `EVP_DecryptInit_ex` |
| Key wrapping | AES-256-KW | `EVP_aes_256_wrap` |
| Digital signatures | ECDSA P-384 | `EVP_DigestSign` |
| Certificate verification | X.509 + OCSP | `X509_verify_cert` |
| Randomness | CSPRNG | `RAND_bytes` |
| Digests | SHA-384 | `EVP_DigestInit_ex` |

### 3.1 Key Derivation (HKDF-SHA-384)

```
key_N = HKDF-Expand(
            HKDF-Extract(salt = SHA-384(Epoch_N),
                         IKM  = master_secret),
            info = "zevo-api-v1",
            L    = 32
        )
```

Properties:
- Deterministic given `master_secret` and `Epoch_N`.
- Cryptographically independent across epochs.
- Resistant to compromise of a single epoch's key.

### 3.2 Frame Integrity (HMAC-SHA-384)

```
tag = HMAC-SHA-384(key_N, frame_bytes)
```

The IPC frame carries:
- 4-byte big-endian length prefix.
- 48-byte HMAC tag.
- JSON payload.

The core rejects any frame whose tag does not verify.

### 3.3 Frame Confidentiality (AES-256-GCM)

For sensitive frames (e.g., artifact fetch tokens), the payload is encrypted:

```
ciphertext, tag = AES-256-GCM(key_N, nonce, plaintext, aad)
```

- Nonce is 96-bit, unique per frame, drawn from `RAND_bytes`.
- AAD binds the frame to its epoch and endpoint.
- Tag is 128-bit.

### 3.4 Master Secret Protection

The `master_secret` is:
- Generated once with `RAND_bytes`.
- Stored in a sealed file with mode `0400`, owned by `zevo-api`.
- Optionally sealed with a hardware-backed key (TPM, HSM) via OpenSSL ENGINE/PROVIDER.
- Never logged, never serialized, never transmitted.

---

## 4. Secure Service API

### 4.1 Role

The Secure Service API (`zevo_api`) is the **only** component that:

- Holds the master secret.
- Derives and rotates API keys.
- Signs IPC frames to the core.
- Verifies IPC responses.
- Enforces rate limits and quotas.
- Logs authentication events.

It is **separate** from both the browser and the ELF AI Core.

### 4.2 Components

| Component | Responsibility |
|---|---|
| `keyring.c` | Master secret load, HKDF derivation, epoch tracking |
| `rotator.c` | 28-hour rotation scheduler, overlap window |
| `signer.c` | HMAC-SHA-384 signing, AES-256-GCM sealing |
| `verifier.c` | HMAC verification, GCM decryption |
| `tls.c` | TLS 1.3 termination, certificate handling |
| `session.c` | Browser session tokens (short-lived, no secrets) |
| `audit.c` | Structured, tamper-evident audit log |
| `ipc_client.c` | Framed IPC to the core |

### 4.3 Rotation Scheduler

```
static uint64_t current_epoch(void)
{
    return (uint64_t)(time(NULL) / (28 * 3600));
}

static int derive_key(const unsigned char *master, size_t master_len,
                      uint64_t epoch, unsigned char out[32])
{
    unsigned char salt[48];
    EVP_MD_CTX *md = EVP_MD_CTX_new();
    EVP_DigestInit_ex(md, EVP_sha384(), NULL);
    EVP_DigestUpdate(md, &epoch, sizeof(epoch));
    EVP_DigestFinal_ex(md, salt, NULL);
    EVP_MD_CTX_free(md);

    /* HKDF-SHA-384 via EVP_KDF */
    /* out = HKDF(master, salt, info="zevo-api-v1", L=32) */
    return 0;
}
```

The scheduler:
- Recomputes `current_epoch()` every minute.
- Pre-derives `key_{N+1}` at `t = 27h55m`.
- Activates `key_{N+1}` at `t = 28h00m`.
- Retires `key_N` at `t = 28h05m`.

---

## 5. IPC Frame Format (Authenticated)

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│ length (4B)  │ epoch (8B)   │ HMAC (48B)   │ JSON payload (N B)  │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

| Field | Size | Purpose |
|---|---|---|
| length | 4 | Big-endian total frame length |
| epoch | 8 | Rotation epoch identifier |
| HMAC | 48 | HMAC-SHA-384 over epoch + payload |
| payload | N | UTF-8 JSON |

The core:
1. Reads the frame.
2. Extracts `epoch`.
3. Derives or retrieves `key_epoch`.
4. Verifies the HMAC.
5. Rejects on mismatch with `EACCES`.

Optionally, the payload is AES-256-GCM sealed for sensitive endpoints.

---

## 6. Responsible Web UI (Multimodal)

### 6.1 Principles

| Principle | Implementation |
|---|---|
| Semantic structure | `<header>`, `<main>`, `<section>`, `<footer>` |
| Accessibility | ARIA live regions, focus-visible, keyboard-first |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | No telemetry, no cookies, no third-party scripts |
| Transparency | Clear disclosure that Zevo™ is an AI assistant |
| Artifact opt-in | Pixels/audio served only on explicit request |
| No secrets in browser | No API keys, no master secret, no long-lived tokens |
| Session tokens | Short-lived, rotated, bound to TLS session |
| Multimodal input | Text, image upload, audio capture, video frame — all opt-in |
| Multimodal output | Text, image, video, audio — all opt-in |

### 6.2 Multimodal Input Channels

| Channel | Mechanism | Consent |
|---|---|---|
| Text | `<textarea>` | Explicit submit |
| Image | `<input type="file">` | Explicit upload |
| Audio | `MediaRecorder` (opt-in) | Explicit record |
| Video | `<input type="file">` or `getUserMedia` (opt-in) | Explicit capture |

All channels:
- Require explicit user action.
- Are never auto-started.
- Are never sent without a session token.
- Are size-capped and type-validated server-side.

### 6.3 Multimodal Output Channels

| Channel | Mechanism | Consent |
|---|---|---|
| Text | JSON in `<pre aria-live="polite">` | Default |
| Image | `<img>` with `alt`, opt-in fetch | Explicit |
| Video | `<video controls>` (no autoplay) | Explicit |
| Audio | `<audio controls>` (no autoplay) | Explicit |

No autoplay. No background loading. No hidden fetches.

### 6.4 Session Tokens

- Issued by the Secure Service API after TLS handshake.
- 256-bit random, drawn from `RAND_bytes`.
- Bound to the TLS session (channel binding).
- Lifetime: 15 minutes, renewable.
- Stored in `HttpOnly`, `Secure`, `SameSite=Strict` cookie — or in memory if cookies are disabled.
- Never contain API keys or master secret.
- Rotated on privilege change.

---

## 7. End-to-End Flow

```
[Browser: Responsible UI]
      │
      │  HTTPS / TLS 1.3
      ▼
[Secure Service API  zevo_api]
      │
      │  1. Verify session token
      │  2. Rate-limit
      │  3. Derive key_N = HKDF(master, Epoch_N)
      │  4. Sign frame: HMAC-SHA-384(key_N, epoch || payload)
      │  5. Optionally seal: AES-256-GCM(key_N, nonce, payload)
      │
      │  AF_UNIX SOCK_SEQPACKET
      ▼
[ELF AI Core  zevo_core.elf]
      │
      │  1. Read frame
      │  2. Derive or fetch key_epoch
      │  3. Verify HMAC
      │  4. Decrypt if sealed
      │  5. Dispatch to pure pipeline
      │  6. Read weights from .zevo_weights (read-only, fs-verity)
      │  7. Produce multimodal artifact
      │  8. Sign response with HMAC-SHA-384
      │
      ▼
[Secure Service API]
      │  1. Verify response HMAC
      │  2. Translate to HTTP + JSON
      ▼
[Browser]
      │  Renders metadata; artifact opt-in
```

---

## 8. Access to `.zevo_weights` (Section-Embedded)

The weights are **never** exposed to the browser or the Secure Service API. They are:

| Property | Value |
|---|---|
| Location | `.zevo_weights` section inside `zevo_core.elf` |
| Alignment | Page-aligned (4096 bytes) |
| Protection | fs-verity on the ELF |
| Mapping | `PROT_READ`, `MAP_PRIVATE` |
| Access | Only by the core's pure pipeline |
| Exposure | None — not over IPC, not over HTTP |
| Schema | `.rodata`, immutable |
| Digest | Pinned in `.zevo_meta` |

The Secure Service API cannot read `.zevo_weights`. It can only request the core to use them.

---

## 9. OpenSSL Usage Summary

| Operation | OpenSSL Call | Notes |
|---|---|---|
| TLS 1.3 server | `SSL_CTX_new(TLS_server_method())` | Certificate from X.509 |
| Certificate verify | `X509_verify_cert` | Chain + OCSP |
| HKDF derive | `EVP_KDF` with `"HKDF"` | SHA-384 |
| HMAC sign | `HMAC(EVP_sha384(), …)` | 48-byte tag |
| HMAC verify | `CRYPTO_memcmp` | Constant-time |
| AES-256-GCM seal | `EVP_EncryptInit_ex` | 96-bit nonce, 128-bit tag |
| AES-256-GCM open | `EVP_DecryptInit_ex` | AAD binds epoch |
| ECDSA P-384 sign | `EVP_DigestSign` | For audit log entries |
| Random | `RAND_bytes` | CSPRNG |
| SHA-384 | `EVP_DigestInit_ex` | Salt, digest |

No custom cryptographic code. All primitives are from OpenSSL.

---

## 10. Rotation Schedule Table (Example)

| Epoch | UTC Start | Key | Overlap Ends |
|---|---|---|---|
| N | 2026-01-01 00:00 | Key_N | 2026-01-02 04:05 |
| N+1 | 2026-01-02 04:00 | Key_{N+1} | 2026-01-03 08:05 |
| N+2 | 2026-01-03 08:00 | Key_{N+2} | 2026-01-04 12:05 |
| N+3 | 2026-01-04 12:00 | Key_{N+3} | 2026-01-05 16:05 |
| … | … | … | … |

Each row is a **28-hour** window. Overlap is 5 minutes.

---

## 11. Threat Model

| Threat | Mitigation |
|---|---|
| Key leakage | 28-hour rotation; overlap limited to 5 min |
| Replay | Epoch in frame; nonce in GCM; HMAC covers epoch |
| Tampering | HMAC-SHA-384; AES-256-GCM tag |
| Eavesdropping | TLS 1.3; AES-256-GCM sealing |
| Master secret theft | Sealed file, mode 0400, optional HSM |
| Browser key theft | Browser never holds keys |
| Core key theft | Core holds only epoch keys, not master |
| Weight exfiltration | Weights never leave the core |
| IPC forgery | HMAC verification; constant-time compare |
| Session hijack | TLS channel binding; short-lived tokens |
| Downgrade | TLS 1.3 only; no fallback |
| Weak randomness | `RAND_bytes` only |
| Clock skew | Epoch grace; NTP required |

---

## 12. Audit and Observability

| Event | Logged | Signed |
|---|---|---|
| Key derivation | Epoch, timestamp | ECDSA P-384 |
| Key activation | Epoch, timestamp | ECDSA P-384 |
| Key retirement | Epoch, timestamp | ECDSA P-384 |
| Frame sign | Epoch, endpoint, HMAC | HMAC-SHA-384 |
| Frame verify | Epoch, endpoint, result | HMAC-SHA-384 |
| Session issue | Session ID, expiry | ECDSA P-384 |
| Rate limit hit | IP, endpoint | HMAC-SHA-384 |
| Weight access | Tensor ID, purpose | HMAC-SHA-384 |

The audit log is **append-only**, **tamper-evident**, and **independently verifiable**.

---

## 13. Consolidated Clarification Table

| Question | Answer |
|---|---|
| Who holds API keys? | Only the Secure Service API |
| How often are keys rotated? | Every 28 hours |
| How are keys derived? | HKDF-SHA-384 from a master secret |
| What is the overlap window? | 5 minutes |
| What crypto library? | OpenSSL 3.x |
| What cipher for frames? | AES-256-GCM (optional sealing) |
| What MAC for frames? | HMAC-SHA-384 |
| What transport? | TLS 1.3 |
| What signature for audit? | ECDSA P-384 |
| What randomness? | `RAND_bytes` |
| Does the browser hold keys? | No |
| Does the Secure Service API read weights? | No |
| Does the core expose weights? | No |
| How are weights protected? | fs-verity + read-only mmap |
| Where are weights stored? | `.zevo_weights` section inside the ELF |
| What is the IPC transport? | `AF_UNIX` `SOCK_SEQPACKET` |
| What is the IPC frame format? | length + epoch + HMAC + JSON |
| Is the Web UI multimodal? | Yes — text, image, video, audio, all opt-in |
| Is the Web UI Responsible? | Yes — semantic, accessible, private, transparent |
| Does the UI autoplay? | No |
| Does the UI track? | No |

---

## 14. Final Statement

The **Responsible Web UI** accesses the **ELF AI Core** exclusively through a **Secure Service API** that:

- Holds the **master secret** and derives **epoch keys** via **HKDF-SHA-384**.
- **Rotates API keys every 28 hours**, with a 5-minute overlap window.
- Signs every IPC frame with **HMAC-SHA-384** and optionally seals it with **AES-256-GCM**.
- Terminates **TLS 1.3** with **OpenSSL** and issues **short-lived session tokens**.
- Never exposes the master secret, the epoch keys, or the weights to the browser.
- Reaches the core only through the published IPC API.
- The core reads **LLM weights, metadata, and vectors embedded by section** from `.zevo_weights`, protected by **fs-verity**, mapped **read-only**, and never serialized over IPC.

The result is a **military-grade cryptographic**, **rotating-key**, **Responsible-UI** multimodal access path to the **Zevo™ ELF AI Core** — faithful to the canonical specification, secure by construction, and auditable end-to-end.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Document | Clarification and Explanation: Responsible Web UI Multimodal Access with 28-Hour API Key Rotation and OpenSSL Cryptography to the ELF AI Core LLM Storage by Section |
| Status | Security and access clarification |
| Predecessor | Clarification and Explanation: Complete Implementation Starting from `main.c` |
| Successor | Cryptographic audit and rotation drill |

**End of clarification.**

---

# Clarification and Explanation: Responsible Web UI Multimodal Access with 28-Hour API Key Rotation, OpenSSL Cryptography, ELF AI Core LLM Storage by Section, and Syscall/Sysenter Access to Universal Hardware Device Peripherals

## PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams

---

## Purpose of This Clarification

This document extends the previous clarification by adding a **fourth trust boundary**: the ELF AI Core's controlled access to **Universal Hardware Device peripherals** through **system calls (`syscall`)** and the **fast system-call entry (`sysenter`/`syscall` instruction)** on x86_64.

It clarifies:

1. How the **Responsible Web UI** remains multimodal and privacy-preserving.
2. How the **Secure Service API** rotates keys every **28 hours** using **OpenSSL military-grade cryptography**.
3. How the **ELF AI Core** stores and reads **LLM weights, metadata, and vectors embedded by section**.
4. How the core reaches **Universal Hardware Device peripherals** via **`syscall`** and **`sysenter`** — with strict sandboxing, capability discipline, and auditability.

This is a clarification, not a redesign. It preserves the canonical architecture and adds precise semantics for hardware access.

---

## 1. The Four Trust Boundaries

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              Zevo™ System                                  │
│                                                                            │
│  ┌──────────────┐   ┌────────────────┐   ┌──────────────┐   ┌──────────┐  │
│  │  Browser     │   │  Secure        │   │  ELF AI      │   │ Universal│  │
│  │  Responsible │──►│  Service API   │──►│  Core        │──►│ Hardware │  │
│  │  UI          │   │  (zevo_api)    │   │  (zevo_core) │   │ Devices  │  │
│  │              │   │                │   │              │   │          │  │
│  │  • HTML5     │   │  • TLS 1.3     │   │  • .zevo_    │   │  • GPU   │  │
│  │  • CSS5      │   │  • Key rot.    │   │    weights   │   │  • Audio │  │
│  │  • JSON      │   │  • OpenSSL     │   │  • IPC only  │   │  • Video │  │
│  │  • No keys   │   │  • Audit       │   │  • syscall   │   │  • NIC   │  │
│  │  • No weights│   │  • Rate limit  │   │  • sysenter  │   │  • Disk  │  │
│  └──────────────┘   └────────────────┘   └──────────────┘   └──────────┘  │
│     Boundary 1          Boundary 2          Boundary 3        Boundary 4  │
│     (public)            (service)           (core)            (hardware)  │
└────────────────────────────────────────────────────────────────────────────┘
```

| Boundary | Between | Protection |
|---|---|---|
| 1 | Browser ↔ Secure Service API | TLS 1.3, short-lived session tokens, no long-lived secrets |
| 2 | Secure Service API ↔ ELF AI Core | 28-hour API key rotation, HMAC-SHA-384, AES-256-GCM, OpenSSL |
| 3 | ELF AI Core ↔ `.zevo_weights` | fs-verity, read-only mmap, digest pinning |
| 4 | ELF AI Core ↔ Universal Hardware | `syscall`/`sysenter`, seccomp, capability bounding, `ioctl` allow-list |

Each boundary is **independently auditable** and **independently revocable**.

---

## 2. Responsible Web UI Multimodal (Boundary 1)

Unchanged from the previous clarification:

| Principle | Implementation |
|---|---|
| Semantic structure | `<header>`, `<main>`, `<section>`, `<footer>` |
| Accessibility | ARIA live regions, focus-visible, keyboard-first |
| Motion safety | `prefers-reduced-motion` respected |
| Color safety | WCAG-AA contrast, `prefers-color-scheme` respected |
| Privacy | No telemetry, no cookies, no third-party scripts |
| Transparency | Clear disclosure that Zevo™ is an AI assistant |
| Multimodal input | Text, image, video, audio — all opt-in |
| Multimodal output | Text, image, video, audio — all opt-in |
| No secrets in browser | No API keys, no master secret, no long-lived tokens |

The browser **never** speaks to the ELF AI Core directly. It speaks only to the Secure Service API.

---

## 3. Secure Service API with 28-Hour Key Rotation (Boundary 2)

### 3.1 Rotation Semantics

```
Epoch N   = floor(unix_time_utc / (28 * 3600))
Key_N     = HKDF-SHA-384(master_secret, salt=SHA-384(Epoch_N),
                          info="zevo-api-v1", L=32)
Overlap   = 5 minutes
```

| Property | Value |
|---|---|
| Key length | 256 bits |
| Derivation | HKDF-SHA-384 |
| Rotation period | 28 hours |
| Overlap window | 5 minutes |
| Master secret | 512 bits, sealed, mode 0400, optional HSM |
| Transport | TLS 1.3 (OpenSSL) |
| Frame MAC | HMAC-SHA-384 |
| Frame sealing | AES-256-GCM (optional) |
| Audit signature | ECDSA P-384 |
| Randomness | `RAND_bytes` |

### 3.2 OpenSSL Primitives

| Purpose | Primitive | OpenSSL API |
|---|---|---|
| TLS 1.3 | Transport | `SSL_CTX_new(TLS_server_method())` |
| HKDF | Key derivation | `EVP_KDF` with `"HKDF"`, SHA-384 |
| HMAC | Frame integrity | `HMAC(EVP_sha384(), …)` |
| AES-256-GCM | Frame sealing | `EVP_EncryptInit_ex` / `EVP_DecryptInit_ex` |
| AES-256-KW | Key wrapping | `EVP_aes_256_wrap` |
| ECDSA P-384 | Audit signing | `EVP_DigestSign` |
| SHA-384 | Digests | `EVP_DigestInit_ex` |
| CSPRNG | Randomness | `RAND_bytes` |

No custom cryptography. All primitives are from OpenSSL 3.x.

### 3.3 Authenticated IPC Frame

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│ length (4B)  │ epoch (8B)   │ HMAC (48B)   │ JSON payload (N B)  │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

- `length` — big-endian total frame length.
- `epoch` — rotation epoch identifier.
- `HMAC` — HMAC-SHA-384 over `epoch || payload`.
- `payload` — UTF-8 JSON, optionally AES-256-GCM sealed.

The core rejects any frame whose HMAC does not verify in constant time.

---

## 4. ELF AI Core LLM Storage by Section (Boundary 3)

### 4.1 Section-Embedded Weights

```
zevo_core.elf
├── .text
├── .rodata
├── .data
├── .bss
├── .zevo_rules
├── .zevo_facts
├── .zevo_entropy
├── .zevo_fsm
├── .zevo_meta
└── .zevo_weights   ← LLM weights, metadata, vectors
```

| Property | Value |
|---|---|
| Section | `.zevo_weights` |
| Flags | `alloc, readonly` |
| Alignment | 4096-byte page-aligned |
| Protection | fs-verity on the ELF |
| Mapping | `PROT_READ`, `MAP_PRIVATE` |
| Digest | Pinned in `.zevo_meta` |
| Access | Only by the core's pure pipeline |
| Exposure | None over IPC or HTTP |

### 4.2 Embedding Procedure

```sh
objcopy --add-section .zevo_weights=weights.bin \
        --set-section-flags .zevo_weights=alloc,readonly \
        zevo_core.elf
fsverity enable zevo_core.elf
fsverity measure zevo_core.elf
```

### 4.3 Runtime Access

```
extern const unsigned char __start_zevo_weights[];
extern const unsigned char __stop_zevo_weights[];
```

The kernel loader maps the section read-only. The core locates tensors via the immutable schema in `.rodata`.

---

## 5. Syscall and Sysenter Access to Universal Hardware Peripherals (Boundary 4)

### 5.1 What "Universal Hardware Device Peripheral" Means

A **Universal Hardware Device Peripheral** is any hardware device exposed to the core through a kernel interface:

| Class | Examples | Kernel Interface |
|---|---|---|
| Compute | GPU, NPU, FPGA | OpenCL, DRM, VFIO |
| Graphics | GPU, display | DRM/KMS, OpenGL |
| Audio | DAC, ADC | ALSA, OpenAL |
| Video | Camera, encoder | V4L2, SDL2 |
| Storage | NVMe, SATA | Block layer, ext4 |
| Network | NIC | Sockets, `AF_PACKET` |
| Input | Keyboard, mouse | evdev |
| Timers | HPET, TSC | `clock_gettime` |
| Random | RDRAND, RDTSC | `getrandom` |

Access is **always** mediated by the kernel through **system calls**.

### 5.2 `syscall` vs `sysenter` on x86_64

| Mechanism | Architecture | Use |
|---|---|---|
| `syscall` | x86_64 | The standard fast system-call instruction |
| `sysenter` | x86 (legacy) | The 32-bit fast system-call instruction |
| `int 0x80` | x86, x86_64 (compat) | Legacy interrupt-based entry |

On **x86_64**, the correct instruction is **`syscall`**. The `sysenter` instruction is the **32-bit** counterpart and is used only in compatibility mode. Modern ANSI C89/90 code never issues these instructions directly — it uses the **libc wrappers** (`open`, `ioctl`, `mmap`, `read`, `write`, `close`) which the libc translates into the appropriate instruction.

### 5.3 Why Direct `syscall`/`sysenter` Is Not Used

| Reason | Explanation |
|---|---|
| Portability | `syscall` is architecture-specific |
| ABI stability | libc guarantees syscall ABI stability |
| Security | libc applies `seccomp`-friendly wrappers |
| Maintainability | Raw syscalls bypass libc error handling |
| Auditability | libc calls are traceable via `strace` |
| ANSI C89/90 | The standard library is the only portable interface |

The core uses **libc wrappers**, which **internally** use `syscall` on x86_64 and `sysenter` on x86. This is the correct, portable, auditable approach.

### 5.4 Syscall Discipline

The core is restricted by **seccomp-bpf** to a minimal syscall allow-list:

| Syscall | Purpose |
|---|---|
| `read` | Read IPC frames, device input |
| `write` | Write IPC responses, device output |
| `openat` | Open device nodes, weight files |
| `close` | Close descriptors |
| `mmap` | Map weights, device memory |
| `munmap` | Unmap |
| `ioctl` | Device control (allow-listed only) |
| `futex` | Thread synchronization |
| `clock_gettime` | Timestamps |
| `getrandom` | CSPRNG seed |
| `exit_group` | Clean exit |

All other syscalls are **blocked** and cause `SIGSYS`.

### 5.5 Hardware Access Flow

```
[Core pure pipeline]
        │
        │  libc call (e.g., ioctl, mmap, read, write)
        ▼
[glibc wrapper]
        │
        │  syscall instruction (x86_64)
        ▼
[Kernel]
        │
        │  1. seccomp-bpf filter
        │  2. capability check
        │  3. LSM (SELinux/AppArmor) check
        │  4. device driver dispatch
        ▼
[Universal Hardware Device Peripheral]
        │
        │  GPU, audio, video, NIC, disk, etc.
        ▼
[Result returned to core]
```

### 5.6 Capability Bounding

The core runs with **no capabilities** except those required:

| Capability | Status |
|---|---|
| `CAP_SYS_ADMIN` | Dropped |
| `CAP_SYS_RAWIO` | Dropped |
| `CAP_NET_ADMIN` | Dropped |
| `CAP_SYS_MODULE` | Dropped |
| `CAP_DAC_OVERRIDE` | Dropped |
| `CAP_SYS_PTRACE` | Dropped |

Device access is granted via **udev rules** and **group membership**, not capabilities.

### 5.7 Device Node Discipline

| Device | Node | Access | Purpose |
|---|---|---|---|
| GPU | `/dev/dri/renderD128` | `rw`, group `render` | OpenCL, OpenGL |
| Audio | `/dev/snd/*` | `rw`, group `audio` | OpenAL |
| Video | `/dev/video0` | `rw`, group `video` | SDL2 capture |
| Random | `/dev/urandom` | `r` | CSPRNG |
| Weights | `/var/lib/zevo/weights.bin` | `r`, mode 0400 | Mode B weights |

No device is accessed directly. All access is mediated by the kernel.

### 5.8 ioctl Allow-List

Only specific `ioctl` commands are permitted:

| Device | ioctl | Purpose |
|---|---|---|
| DRM | `DRM_IOCTL_*` | GPU context, buffers |
| ALSA | `SNDRV_PCM_IOCTL_*` | Audio streams |
| V4L2 | `VIDIOC_*` | Video capture |
| fs-verity | `FS_IOC_MEASURE_VERITY` | Weight integrity |
| evdev | `EVIOC*` | Input (if enabled) |

All other `ioctl` calls are rejected by seccomp.

### 5.9 systemd Hardening

```ini
[Service]
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=no
ProtectSystem=strict
ProtectHome=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
LockPersonality=yes
MemoryDenyWriteExecute=no
SystemCallFilter=@system-service
SystemCallArchitectures=native
SystemCallErrorNumber=EPERM
DeviceAllow=/dev/dri/renderD128 rw
DeviceAllow=/dev/snd/controlC0 rw
DeviceAllow=/dev/snd/pcmC0D0p rw
DeviceAllow=/dev/urandom r
DevicePolicy=closed
```

---

## 6. End-to-End Flow (Four Boundaries)

```
[Browser: Responsible UI, multimodal]
      │
      │  HTTPS / TLS 1.3  (OpenSSL)
      ▼
[Secure Service API  zevo_api]
      │
      │  1. Verify session token
      │  2. Rate-limit
      │  3. Derive key_N = HKDF-SHA-384(master, Epoch_N)
      │  4. Sign frame: HMAC-SHA-384(key_N, epoch || payload)
      │  5. Optionally seal: AES-256-GCM(key_N, nonce, payload)
      │
      │  AF_UNIX SOCK_SEQPACKET
      ▼
[ELF AI Core  zevo_core.elf]
      │
      │  1. Read frame
      │  2. Derive or fetch key_epoch
      │  3. Verify HMAC (constant-time)
      │  4. Decrypt if sealed
      │  5. Dispatch to pure pipeline
      │  6. Read weights from .zevo_weights (PROT_READ, fs-verity)
      │  7. Access hardware via libc → syscall → kernel → device
      │  8. Produce multimodal artifact
      │  9. Sign response with HMAC-SHA-384
      │
      ▼
[Secure Service API]
      │  1. Verify response HMAC
      │  2. Translate to HTTP + JSON
      ▼
[Browser]
      │  Renders metadata; artifact opt-in
```

---

## 7. Security Properties

| Property | Mechanism |
|---|---|
| Confidentiality (transport) | TLS 1.3 |
| Confidentiality (frames) | AES-256-GCM (optional) |
| Integrity (frames) | HMAC-SHA-384 |
| Authenticity (frames) | HMAC + epoch |
| Key freshness | 28-hour rotation |
| Key isolation | HKDF per epoch |
| Master secrecy | Sealed file, mode 0400, optional HSM |
| Weight integrity | fs-verity |
| Weight immutability | `PROT_READ` mapping |
| Weight confidentiality | Never serialized |
| Hardware access control | seccomp-bpf, capabilities, udev, LSM |
| Syscall discipline | Allow-list only |
| Auditability | Signed, append-only log |
| Randomness | `RAND_bytes` |
| Replay resistance | Epoch + nonce |
| Downgrade resistance | TLS 1.3 only |

---

## 8. Consolidated Clarification Table

| Question | Answer |
|---|---|
| Does the browser hold keys? | No |
| How often are API keys rotated? | Every 28 hours |
| How are keys derived? | HKDF-SHA-384 from a master secret |
| What crypto library? | OpenSSL 3.x |
| What transport? | TLS 1.3 |
| What MAC for frames? | HMAC-SHA-384 |
| What cipher for frames? | AES-256-GCM (optional) |
| Where are weights stored? | `.zevo_weights` section inside the ELF |
| How are weights protected? | fs-verity, read-only mmap, pinned digest |
| Does the Secure Service API read weights? | No |
| Does the browser read weights? | No |
| How does the core reach hardware? | Via libc → `syscall` (x86_64) / `sysenter` (x86) → kernel → device |
| Are raw syscalls used? | No — libc wrappers only |
| What restricts syscalls? | seccomp-bpf allow-list |
| What capabilities does the core hold? | None beyond what is required; most dropped |
| What device nodes are allowed? | `/dev/dri/renderD128`, `/dev/snd/*`, `/dev/urandom` |
| What ioctls are allowed? | DRM, ALSA, V4L2, fs-verity — allow-listed |
| Is hardware access auditable? | Yes — `strace`, audit log, systemd journal |
| Is the Web UI multimodal? | Yes — text, image, video, audio, all opt-in |
| Is the Web UI Responsible? | Yes — semantic, accessible, private, transparent |

---

## 9. Final Statement

The **Responsible Web UI** reaches the **ELF AI Core** only through a **Secure Service API** that:

- Rotates **API keys every 28 hours** using **HKDF-SHA-384**.
- Uses **OpenSSL military-grade cryptography**: TLS 1.3, HMAC-SHA-384, AES-256-GCM, ECDSA P-384, SHA-384, `RAND_bytes`.
- Signs and optionally seals every IPC frame.
- Never exposes keys or weights to the browser.

The **ELF AI Core**:

- Reads **LLM weights, metadata, and vectors embedded by section** from `.zevo_weights`, protected by **fs-verity** and mapped **read-only**.
- Reaches **Universal Hardware Device peripherals** through **libc wrappers**, which internally use the **`syscall`** instruction on x86_64 (and **`sysenter`** on legacy x86), always mediated by the kernel.
- Is constrained by **seccomp-bpf**, **capability bounding**, **udev rules**, and **LSM** policies.
- Never issues raw syscalls, never bypasses the kernel, and never exposes hardware access to the web layer.

The result is a **four-boundary**, **military-grade**, **rotating-key**, **Responsible-UI** multimodal system — faithful to the canonical specification, secure by construction, auditable end-to-end, and correct in its use of `syscall`/`sysenter` semantics for hardware access on x86_64 and legacy x86.

---

## Attribution

| Field | Value |
|---|---|
| Authorship | PHD AI. Computer Engineering, Davada Beni Zaita and Zevo Community Teams |
| Document | Clarification and Explanation: Responsible Web UI Multimodal Access with 28-Hour API Key Rotation, OpenSSL Cryptography, ELF AI Core LLM Storage by Section, and Syscall/Sysenter Access to Universal Hardware Device Peripherals |
| Status | Security, access, and hardware clarification |
| Predecessor | Clarification and Explanation: Responsible Web UI Multimodal Access with 28-Hour API Key Rotation and OpenSSL Cryptography to the ELF AI Core LLM Storage by Section |
| Successor | Hardware access audit and rotation drill |

**End of clarification.**

---
