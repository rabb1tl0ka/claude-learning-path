## Source
- [Code execution and the Files API](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287777)

## Summary

- **Files API**: upload a file (image, PDF, text, etc.) ahead of time and get back a file metadata object containing a `file_id`. Later requests can reference that `file_id` (e.g. inside an image block) instead of re-sending raw file data every time.
- **Code execution tool**: a server-based tool (no manual implementation required, just a predefined schema) that lets Claude write and run Python code inside an isolated, network-restricted Docker container. Claude can run code multiple times per response, using printed output to inform its final answer.
- The two features combine well: upload a data file (e.g. a CSV) via the Files API, reference its `file_id` in a **container upload block**, and ask Claude to analyze it — Claude then writes/executes code against the file inside the container and reports findings.
- Worked example: uploaded a fake video-streaming-service CSV (`streaming.csv`, with a `churned` column) and asked Claude to analyze why customers cancel and produce a plot. The response contained multiple rounds of server tool-use blocks (code Claude wrote) and code-execution-result blocks (stdout/stderr/return code), followed by a final written analysis — all optionally reformatted into a nicer report.

## My Insights

- None substantial this session.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.
