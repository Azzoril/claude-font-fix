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

`@font-face` hijacking doesn't work for Hiragino Sans on macOS (Chrome restricts overriding system fonts). Instead we insert Chinese fonts before Hiragino Sans:

1. **`:root` variables**: covers normal chat where `--font-ui-serif` / `--font-serif` are used.
2. **Selectors on `.font-claude-response`**: covers incognito chat, which hardcodes `font-family` instead of reading variables. Matches `:is(.standard-markdown, .progressive-markdown)` to handle both streaming and completed rendering, with no depth constraints so it works whether or not the reply has a thinking block.
3. **`.font-ui.leading-normal` guard**: restores UI sans-serif for thinking content. Anthropic uses this wrapper class in both streaming and completed states, so one rule covers both. Position-based selectors like `.row-start-1` won't work because thinking content moves positions during streaming.
4. **`.font-claude-response.standard-markdown`**: covers the Artifact/MD preview where both classes sit on the same element.
5. **`.katex .cjk_fallback`**: covers CJK inside math formulas.

Chinese font order: **Noto Serif CJK SC** (optional), **Songti SC** (macOS built-in), **PingFang SC** (fallback). English and numbers hit `Anthropic Serif` first and are unaffected.

See [dom-structure.md](dom-structure.md) for the full DOM layout.

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