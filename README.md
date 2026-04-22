# claude-font-fix

Fix Claude.ai Chinese font rendering by replacing Japanese fonts with Chinese fonts.

## Problem

Claude.ai recently updated their frontend CSS font fallback chain. Chinese characters are now rendered using Japanese fonts instead of proper Chinese fonts.

Claude's own font (Anthropic Serif) only covers English and symbols. For Chinese characters, the browser walks down the fallback chain and picks the first font that can render them. The full chain (stored in --font-ui-serif and --font-serif CSS variables) is:

```
"Anthropic Serif", Georgia, "Arial Hebrew", "Noto Sans Hebrew",
"Times New Roman", Times, "Hiragino Sans"(7th), "Yu Gothic"(8th), Meiryo,
"Noto Sans CJK JP", "PingFang TC", "Microsoft JhengHei",
"Noto Sans CJK TC", "PingFang SC"(14th), "Microsoft YaHei"(15th),
"Noto Sans CJK SC", "Apple SD Gothic Neo", "Malgun Gothic",
"Noto Sans CJK KR", serif
```

Windows and Mac share the same chain, but hit different Japanese fonts depending on which are installed:

**Windows:** Yu Gothic (8th) is matched. PingFang SC is not installed, Microsoft YaHei (15th) never gets reached.

**Mac:** Hiragino Sans (7th) is matched. PingFang SC (14th) never gets reached.

Both result in Chinese text being rendered with Japanese glyph standards, causing incorrect stroke structures and poor readability for Chinese users.


## Solution

### Windows: `@font-face` hijacking

We hijack the Japanese font-face `Yu Gothic` declarations to point to proper Chinese fonts.

```css
@font-face {
  font-family: "Yu Gothic";
  src: local("Noto Serif SC");
  font-weight: 1 999;
}
@font-face {
  font-family: "Yu Gothic UI";
  src: local("Noto Serif SC");
  font-weight: 1 999;
}
```

### Mac: CSS variable override + selector override

`@font-face` hijacking doesn't work for Hiragino Sans on macOS (Chrome restricts overriding system-level fonts). Instead we insert Chinese fonts before Hiragino Sans via four layers:

1. **`:root` variable override**: covers normal (non-incognito) chat where `--font-ui-serif` and `--font-serif` are used
2. **`.font-claude-response` scoped selectors matching `:is(.standard-markdown, .progressive-markdown)`**: covers incognito chat (which hardcodes `font-family` instead of using CSS variables). The selector is depth-agnostic so it handles all four DOM variants: streaming/completed | with-thinking/no-thinking. The two markdown classes correspond to streaming (`.progressive-markdown`) and completed (`.standard-markdown`) rendering.
3. **`.font-ui.leading-normal` guard**: scopes back to UI sans-serif for thinking content. This anchor works in both streaming and completed states because Anthropic uses the same wrapper class for thinking content in both cases — position-based selectors like `.row-start-1` don't work because thinking markdown is temporarily rendered inside `.row-start-2` during streaming before the DOM restructures.
4. **Compound `.font-claude-response.standard-markdown`**: covers the Artifact/MD preview panel where both classes sit on the same element.
5. **`.katex .cjk_fallback`**: covers CJK characters inside math formulas.

The Chinese font fallback order is: **Noto Serif CJK SC** (optional, install for best results) -> **Songti SC** (macOS built-in serif) -> **PingFang SC** (fallback sans-serif). English and numbers hit `Anthropic Serif` first and are unaffected.

For the full DOM structure and selector coverage matrix, see [dom-structure.md](dom-structure.md).

## Installation

### Windows

#### 1. Install Noto Serif SC font

If you already have **Noto Serif SC** or **Source Han Serif SC** (思源宋体) installed, skip this step. They are the same typeface.

Download from [GitHub Releases](https://github.com/notofonts/noto-cjk/releases) -> **Noto Serif CJK Version 2.003** -> **All Variable TTF/OTC** (recommended) or **Language Specific OTFs Simplified Chinese**

Install the font and restart Chrome.

#### 2. Install the Chrome extension

```bash
git clone https://github.com/Azzoril/claude-font-fix.git
```
or copy `windows-font-fix/manifest.json` and `windows-font-fix/style.css` into a folder.

1. Open `chrome://extensions`
2. Enable **Developer mode** (top right)
3. Click **Load unpacked** -> select the `windows-font-fix` folder
4. Refresh Claude.ai

### Mac

The extension works out of the box with macOS built-in **Songti SC**. For better screen readability, you can optionally install **Noto Serif CJK SC** (思源宋体):

Download from [GitHub Releases](https://github.com/notofonts/noto-cjk/releases) -> **Noto Serif CJK Version 2.003** -> **All Variable TTF/OTC** (recommended) or **Language Specific OTFs Simplified Chinese**

Restart Chrome after installing.

#### Install the Chrome extension

```bash
git clone https://github.com/Azzoril/claude-font-fix.git
```

or copy `mac-font-fix/manifest.json` and `mac-font-fix/style.css` into a folder.
1. Open `chrome://extensions`
2. Enable **Developer mode** (top right)
3. Click **Load unpacked** -> select the `mac-font-fix` folder
4. Refresh Claude.ai

## Community

Published on [LINUX DO](https://linux.do) community.

## License

MIT