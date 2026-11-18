# webrender-models

Model files that the Webrender apps fetch when they are built. Each file is pinned by size and sha256 in the app that uses
it, so a build cannot ship other bytes. There is one release per export, and each release says how its file was made.
Nothing here runs at install time.

## `kokoro-durations-v1` — Kokoro 82M v1.0 with a `durations` output

| | |
|---|---|
| File | `kokoro-durations.onnx` |
| Size | 325,501,490 B |
| sha256 | `e015f89f484764c13569356de9483ad8ed750e1916b3dc529c540224d8270c89` |
| Weights | hexgrad's **Kokoro 82M v1.0**: [`hexgrad/Kokoro-82M`](https://huggingface.co/hexgrad/Kokoro-82M), `kokoro-v1_0.pth`, revision `f3ff3571` |
| Used by | Webrender Pdf Desk's read-aloud: the word highlight that follows the voice |

**What it is.** The same model as the `kokoro.onnx` in
[Lyrcaxis/KokoroSharpBinaries v2.0.0](https://github.com/Lyrcaxis/KokoroSharpBinaries/releases/tag/v2.0.0), with one
output added. Beside `audio` it returns `durations`, an int64 frame count for each input token, the two pad tokens
included. [KokoroSharp](https://github.com/Lyrcaxis/KokoroSharp) 0.8.2 and later read that output to give a timestamp for
every phoneme it speaks. The published export has no such output, so KokoroSharp's timestamps are empty with it.

**How it was made.** hexgrad's `kokoro` 0.9.4 `KModel`, with `disable_complex=True`, wrapping `forward_with_tokens`, was
exported by PyTorch 2.14.1's TorchScript exporter at opset 20. That is the same exporter, opset and graph shape as the
v2.0.0 file: 4,543 nodes and 443 initializers, op for op, plus the one output. One frame is 600 samples at 24 kHz, so
`sum(durations) × 600` equals the audio's length.

**Changes from the published weights** (Apache-2.0 §4(b)). The weight normalisation on the convolution layers is folded
into plain weights before the export (`torch.nn.utils.remove_weight_norm`, 89 modules), as in the v2.0.0 file. This is the
same function computed once rather than at every inference. No weight is retrained or quantised. The `durations` output
is the duration predictor's own rounded frame counts, which the model already computes to place its audio.

**Licence.** The weights are hexgrad's, released under the Apache License 2.0 ([`LICENSE`](LICENSE)).

## `misakisharp-2.1.1-english` — MisakiSharp 2.1.1 with only its English lexicons

| | |
|---|---|
| File | `MisakiSharp.dll` |
| Size | 20,601,344 B |
| sha256 | `ee0c948be2225f435763cc40bf39de537b7fca9446bedbb41e55b4dbf2c43c9a` |
| From | the [MisakiSharp](https://github.com/Lyrcaxis/MisakiSharp) 2.1.1 NuGet package's `lib/net8.0/MisakiSharp.dll`, 69,382,656 B, sha256 `f90ef27ee2d001b15b31270003d2310297847d2b44f7dc2665150f014f9efe0d` (source commit `4ec982c8`) |
| Used by | the Webrender apps' voice (Kokoro through KokoroSharp 0.8.4), in place of the package's copy |

**What it is.** MisakiSharp is the G2P that turns text into the phonemes Kokoro speaks. Its assembly carries 20 gzipped
lexicons for eight languages, and 69,192,039 of its 69,382,656 bytes are those. The Webrender apps speak only with
English voices (British and American), and an English reading loads only the six English lexicons, so the other 14 are
dead weight in every copy of every app.

**How it was made.** [Mono.Cecil](https://github.com/jbevain/cecil) 0.11.6 reads the package's dll, removes the 14
embedded resources that are not English, rewrites the description and informational version, and writes the assembly with
a content-derived MVID. Two runs on the same input write the same bytes. It is made from the package's dll rather than
from the source, so the code is the package's own: decompiled, the two assemblies differ only in those two attributes.

**Changes from the package** (Apache-2.0 §4(b)). Removed: `es_espeak`, `fr_espeak`, `hi_espeak`, `hi_espeak_lexicon`,
`it_espeak`, `ja_char_unk`, `ja_lexicon`, `ja_matrix_compact`, `ja_words`, `pt_espeak`, `zh_finalseg_hmm`, `zh_g2p`,
`zh_g2p_v2`, `zh_posseg_hmm`. Kept, byte for byte: `en_lexicon_gb`, `en_lexicon_us`, `en_tagger`, `en_tokenizer`,
`engb_espeak`, `enus_espeak`. The description now says the file is modified, and the informational version reads
`2.1.1+4ec982c8…+english-only`. The name and version (`MisakiSharp`, `2.1.1.0`) are unchanged, so KokoroSharp loads it
as it loads the package's. **Asked for a language other than English, it throws** (`ArgumentNullException` on the missing
lexicon's stream). It is unsigned, as the package's is.

**Licence.** MisakiSharp is Lyrcaxis's, released under the Apache License 2.0 ([`LICENSE`](LICENSE): the same text).
Its `NOTICE` file and its `LICENSE` are reproduced in this release's `MisakiSharp-2.1.1-english-NOTICES.txt`.
