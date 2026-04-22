body.font-ui
`- #root
   |- div[data-is-streaming]                   <- "true" during streaming, "false" when done
   |- .font-claude-response                    <- outermost wrapper for each reply
   |  `- div
   |     |
   |     |- [A] No-thinking reply (flat structure)
   |     |    `- :is(.standard-markdown, .progressive-markdown)   <- main response, directly here
   |     |       |- p.font-claude-response-body
   |     |       |- h1-h6, ol/ul, blockquote, table, ...
   |     |       |- div[role="group"] (code block, monospace)
   |     |       |- span.katex (inline math)
   |     |       `- ...
   |     |
   |     `- [B] With-thinking reply (grid structure)
   |          `- div.grid.grid-rows-[auto_auto]
   |             |
   |             |- div.row-start-1                    <- thinking block (when completed)
   |             |  `- ... button (status summary)
   |             |  `- div.grid (expandable container)
   |             |     `- div.overflow-hidden
   |             |        `- div.flex.flex-col.font-ui.leading-normal    <- thinking content anchor
   |             |           `- ...
   |             |              `- .standard-markdown          <- thinking markdown
   |             |
   |             `- div.row-start-2                    <- main response block
   |                `- div.row-start-1.z-[2]           <- main response slot
   |                |  `- div
   |                |     `- div.standard-markdown.standard-markdown      <- main response (completed)
   |                |
   |                `- div.row-start-1.z-[3]           <- streaming thinking slot (TEMPORARY)
   |                   `- div.flex.flex-col.font-ui.leading-normal    <- thinking content anchor
   |                      `- ...
   |                         `- :is(.standard-markdown, .progressive-markdown)  <- thinking markdown (streaming)
   |
   |- Streaming vs completed markdown classes:
   |    - .progressive-markdown: outer wrapper used during streaming; each block
   |      (p, h3, code block, ul, etc.) is wrapped in its own <div> under it,
   |      and each inner wrapper contains a .standard-markdown.
   |    - .standard-markdown: used in completed state (direct container),
   |      and also nested inside .progressive-markdown during streaming.
   |    The selector :is(.standard-markdown, .progressive-markdown) covers both.
   |
   |- Thinking content anchor (.font-ui.leading-normal):
   |    Same class is used on the thinking wrapper in both states. The
   |    `.row-start-1` position changes between streaming (nested inside
   |    .row-start-2) and completed (direct grid child), so position-based
   |    selectors are unreliable. Anchoring on this class works for both.
   |
   |- div.mt-4                                 <- following thinking + response blocks (same structure)
   |
   |- #wiggle-file-content                     <- Markdown preview panel
   |  `- div
   |     `- div.standard-markdown.font-claude-response  <- both classes on same element
   |        |- p.font-claude-response-body
   |        |- h1-h6, ol, ul, table, code, ...
   |        `- (same structure as main response)
   |
   `- iframe[src="claudeusercontent.com"]      <- React/JSX preview (cross-origin iframe, CSS cannot reach)