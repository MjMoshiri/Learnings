# AI in Browser (Chrome extension)

Chrome extension that calls the Hugging Face Inference API on selected text in any tab. Built to stop tab-switching to ChatGPT for small tasks like grammar fixes. Single context-menu action, configurable model.

What I took from it: extension manifest v3, background scripts vs content scripts, message passing between them, and the awkwardness of bundling a real build pipeline (webpack) into a Chrome extension.
