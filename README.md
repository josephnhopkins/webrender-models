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
