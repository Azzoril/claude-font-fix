```text
body.font-ui
`- #root
   |- .font-claude-response                    <- outermost wrapper for each reply
   |  `- div > div.grid.grid-rows-[auto_auto]
   |     |
   |     |- div.row-start-1                    <- thinking block (sans-serif, do not change)
   |     |  `- div > div.flex.flex-col.font-ui.leading-normal
   |     |     `- div.standard-markdown        <- markdown inside thinking block
   |     |        |- p.font-claude-response-body
   |     |        |- ol / ul
   |     |        `- ...
   |     |
   |     `- div.row-start-2                    <- main response block (serif, needs fix)
   |        `- div.row-start-1 > div
   |           `- div.standard-markdown.standard-markdown
   |              |- p.font-claude-response-body
   |              |- h1-h6
   |              |- ol / ul (including nested lists)
   |              |- blockquote > p.font-claude-response-body
   |              |- div[role="group"] (code block, monospace)
   |              |- div > table
   |              |- span.katex (inline math)
   |              |- span.katex-display (block math)
   |              |- ul.contains-task-list (task list)
   |              |- hr
   |              `- a (link)
   |
   |- div.mt-4                                 <- following thinking + response blocks (same structure)
   |  `- div.grid > .row-start-1 + .row-start-2
   |
   |- #wiggle-file-content                     <- Markdown preview panel
   |  `- div
   |     `- div.standard-markdown.font-claude-response  <- both classes on same element
   |        |- p.font-claude-response-body
   |        |- h1-h6, ol, ul, table, code, ...
   |        `- (same structure as main response)
   |
   `- iframe[src="claudeusercontent.com"]      <- React/JSX preview (cross-origin iframe, CSS cannot reach)
```