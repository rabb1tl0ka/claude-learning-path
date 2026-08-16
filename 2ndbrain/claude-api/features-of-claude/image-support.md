## Source
- [Image support](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287778)

## Summary

- Claude's vision capabilities let a user message include images for tasks like counting objects, comparing images, or identifying content. Limits: up to 100 images per request; token cost scales with image dimensions.
- Images are sent as an **image block** inside a user message — either raw image data or a URL to a hosted image. Multiple image blocks can appear in one message.
- Simple prompts on images often produce inaccurate results — e.g. asking "how many marbles are in this image?" on a 12-marble image returned an incorrect count of 13. The same prompting techniques from earlier in the course (guidelines, analysis steps, one-shot/multi-shot examples) dramatically improve accuracy: asking Claude to count twice using different strategies and compare the two counts got the correct count of 12.
- One-shot/multi-shot prompting for images works by alternating image blocks and text blocks in the message — e.g. an example image with 11 marbles paired with a text block stating "the image above has 11 marbles," before the real target image.
- Worked real-world example: a wildfire-insurance scenario where an insurer needs to verify a homeowner has trimmed/cut trees around their house before insuring it — illustrates how detailed, structured prompting is needed for a real image-analysis use case.

## My Insights

- Found the level of prompting detail required for a real-world use case (the fire-risk assessment example) interesting.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.
