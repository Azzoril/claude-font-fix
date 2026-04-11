# claude-font-fix

Fix Claude.ai Chinese font rendering on Windows by replacing Yu Gothic (Japanese font) with Noto Serif SC using @font-face hijacking.

## Problem

Claude.ai recently updated their frontend fonts. On Windows, Chinese characters are now rendered using **Yu Gothic** (Japanese font). This happens because Claude's CSS font fallback chain places Japanese fonts before Chinese fonts:
Anthropic Serif Web Text -> Georgia -> ... -> Yu Gothic (8th) -> ... -> Microsoft YaHei (15th)

The browser matches Chinese characters from left to right. Yu Gothic contains Chinese characters but renders them using Japanese glyph standards, resulting in incorrect stroke structures and poor readability for Chinese users.

## Solution

Instead of fighting CSS specificity and selectors, we hijack the `Yu Gothic` font-face declaration to point to `Noto Serif SC`:

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

The browser thinks it's using Yu Gothic, but actually loads Noto Serif SC. English/numbers hit `Anthropic Serif Web Text` first and are completely unaffected.

## Installation

### 1. Install Noto Serif SC font

If you already have **Noto Serif SC** or **Source Han Serif SC** (思源宋体) installed, skip this step. They are the same typeface.

Download from [GitHub Releases](https://github.com/notofonts/noto-cjk/releases) -> **Noto Serif CJK Version 2.003** -> **All Variable TTF/OTC** (recommended) or **Language Specific OTFs Simplified Chinese**

Install the font and restart Chrome.

### 2. Install the Chrome extension

```bash
git clone https://github.com/Azzoril/claude-font-fix.git
```
or copy `manifest.json` and `style.css` into a folder.

1. Open `chrome://extensions`
2. Enable **Developer mode** (top right)
3. Click **Load unpacked**
4. Select the cloned folder
5. Refresh Claude.ai

## License

MIT