# Local STT command providers

Use this recipe when the host application's native STT backends do not include the desired open-source engine.

## Procedure

1. Read the engine's official README and release assets. Prefer a prebuilt binary over compiling when the project publishes a matching Windows accelerator build.
2. Place the binary and model under a stable application data directory, for example:
   `C:/Users/<user>/AppData/Local/hermes/<engine>/bin/` and `.../models/`.
3. Run the engine's `--help` command, then a real short WAV transcription. A successful process exit alone is insufficient; confirm the output and accelerator-selection log.
4. Configure Hermes' command provider with a literal executable/model path and the `{input_path}` placeholder:
   `hermes config set HERMES_LOCAL_STT_COMMAND '<exe> transcribe --model <model> --input {input_path} ...'`
5. Select it with `hermes config set stt.provider local_command` and keep `stt.enabled` true.
6. Restart Hermes before testing voice input, because provider and environment changes are loaded by the new process.

## Parakeet.cpp example

For `mudler/parakeet.cpp` on Windows, the Vulkan release binary is appropriate for an NVIDIA GPU:

```text
parakeet-cli.exe transcribe --model <model>.gguf --input {input_path} --decoder tdt
```

Use a TDT model with `--decoder tdt`; for example, the `tdt-1.1b-q6_k.gguf` model balances accuracy and size, while a smaller 0.6B quantization reduces download and memory cost. The model repository is `mudler/parakeet-cpp-gguf` on Hugging Face.

## Pitfalls

- Do not set `stt.provider` to an engine name unless Hermes has a registered plugin or command provider; unknown names produce an unavailable-provider error.
- Do not validate only with the system Python. Hermes optional dependencies may live in a managed runtime, while command engines are independent native binaries.
- Do not download multi-gigabyte weights before checking the available accelerator and model size; the wrong binary can silently fall back to CPU or fail at load time.
- Do not claim the engine is active from configuration output alone; run a real WAV probe and inspect the backend/device line.