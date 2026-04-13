# claude-font-fix

Fix Claude.ai Chinese font rendering by replacing Japanese fonts with Chinese fonts using `@font-face` hijacking.

## Problem

Claude.ai recently updated their frontend CSS font fallback chain. Chinese characters are now rendered using Japanese fonts instead of proper Chinese fonts.

Claude's own font (`Anthropic Serif`) only covers English and symbols. For Chinese characters, the browser walks down the fallback chain and picks the first font that can render them. The full chain is:

`"Anthropic Serif", Georgia, "Arial Hebrew", "Noto Sans Hebrew", "Times New Roman", Times, "Hiragino Sans", "Yu Gothic", Meiryo, "Noto Sans CJK JP", "PingFang TC", "Microsoft JhengHei", "Noto Sans CJK TC", "PingFang SC", "Microsoft YaHei", "Noto Sans CJK SC", "Apple SD Gothic Neo", "Malgun Gothic", "Noto Sans CJK KR", serif`

Windows and Mac share the same chain, but hit different Japanese fonts depending on which are installed:

**Windows:** Yu Gothic (8th) is matched. PingFang SC is not installed, Microsoft YaHei (15th) never gets reached.

**Mac:** Hiragino Sans (7th) is matched. PingFang SC (14th) never gets reached.

Both result in Chinese text being rendered with Japanese glyph standards, incorrect stroke structures and poor readability for Chinese users.


## Solution

Instead of fighting CSS specificity and selectors, we hijack the Japanese font-face declarations to point to proper Chinese fonts. English/numbers hit `Anthropic Serif` first and are completely unaffected.

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