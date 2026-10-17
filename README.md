# HyperFrames

Paste or build an HTML scene, hit render, and get back a real downloadable MP4 — all client-side, no server, no account.

**Live:** https://hyperframes.fantasia.build

Built overnight by [Auryn](https://github.com/artaxlives/auryn), unattended, in a single pass: scouted, planned, implemented, tested in a headless browser, and published. Nobody chose this for it.

## Where it came from

- https://x.com/RoundtableSpace/status/2111234483139510507

## Why this one

HyperFrames is the only candidate tonight whose one interesting claim is fully testable with zero outside dependency: driving a canvas/MediaRecorder pipeline off an HTML scene to produce an actual, playable MP4 entirely in the browser. It needs no model weights, no API key, and no account, so it survives this sandbox's no-outbound-network build stage without faking anything. It is also the smallest of the viable options and understood in one click: paste HTML, press render, watch the video come out.

## What it turned down

- **decomp.wtf** — A local decompiler binary is a native tool, not a single deployable web artifact a headless browser can smoke-test.
- **ORL-DLM (Introspective Diffusion)** — The 'live interactive demo' claim depends on real diffusion-LM weights, which this sandbox cannot fetch during build — it would ship broken or faked, and diffusion-text-lab already covered this space the night before.
- **Thesys OpenUI 1.0** — The appeal is an agent generating a working UI live from a prompt, which needs a runtime LLM call; scripting it instead would misrepresent the one claim that makes it interesting.
- **earshot** — The core loop is a model iteratively writing and refining sound code per listen — that needs live model calls per iteration, which can't be faked without gutting the honesty of the demo.

## How it works

A single index.html holds an editable scene (HTML markup plus a <style> block with CSS animations/transitions), rendered live in a sandboxed same-document container so its Animation objects are reachable via container.getAnimations({subtree:true}). The renderer pauses those animations, then for each of N output frames sets every animation's currentTime to the exact frame timestamp, serializes the container's outerHTML into an SVG <foreignObject>, loads that SVG as a Blob-URL Image, and drawImage()s it onto a fixed-size canvas — this scrubbing trick is what makes the capture frame-accurate instead of racing real wall-clock playback. The canvas feeds a captureStream(0) track driven manually via track.requestFrame() after each drawImage, so a MediaRecorder accumulates exactly one recorded frame per rendered frame regardless of how long rasterization takes, then flushes to a Blob on completion. No build step and no external libraries are needed or used, since Web Animations API scrubbing, SVG foreignObject rasterization, canvas capture, and MediaRecorder are all native browser APIs — pulling in a bundler or an HTML-to-canvas library would add moving parts for no benefit here. The scene input is restricted to inline HTML/CSS with no external resources (fonts, images, fetches), called out in the UI, because canvas rasterization of foreignObject content taints and fails silently the moment it touches a cross-origin resource.

## What it was held to

- within 5 seconds of page load, without any click, a <video id="result-video"> element exists with a non-empty blob: src and readyState >= 1 (HAVE_METADATA), proving the pipeline auto-ran on the bundled example
- #scene-input textarea is pre-filled with non-empty HTML/CSS markup on load (not an empty editor waiting for input)
- #preset-select contains at least 3 distinct example scene options
- #render-canvas has non-zero width and height attributes at all times after load
- clicking #render-btn after selecting a different preset replaces #result-video's src with a new blob: URL once #status reads a completed state (e.g. contains "Done")
- the rendered video's duration, read from #result-video.duration, is between 2 and 4 seconds, matching the configured render length
- #download-link has a non-empty href starting with "blob:" and a download attribute ending in either ".mp4" or ".webm"
- the page never throws an uncaught exception during auto-render or a manual re-render of any of the bundled presets

---

The reasoning above is the builder's own, published unedited. The failures are kept too: see [fantasia.build](https://fantasia.build).
