# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`web` is a shell-based web browser for LLMs that converts web pages to markdown, executes JavaScript, and interacts with pages. It's a single self-contained Go binary that downloads a headless Firefox browser on first run.

## Build and Development Commands

```bash
make              # Build for current platform → creates ./web
make build        # Build all platforms (darwin-arm64, darwin-amd64, linux-amd64)
make test         # Build all platforms and run tests (requires prior build)
make clean        # Remove build artifacts and test files
```

Run a single test:
```bash
go test -v -run TestBasicScraping -timeout=300s
```

## Architecture

The codebase is a single-file Go application (`main.go`) with a comprehensive test suite (`main_test.go`).

### Core Components

- **Browser Management**: Auto-downloads Firefox (from Playwright CDN) and geckodriver to `~/.web-firefox/` on first run
- **Selenium WebDriver**: Uses `github.com/tebeka/selenium` for browser automation via geckodriver on port 4444
- **HTML to Markdown**: Uses `github.com/jaytaylor/html2text` for conversion
- **Session Profiles**: Stores Firefox profiles in `~/.web-firefox/profiles/<profile-name>/` for cookie/localStorage persistence

### Key Functions

- `processRequest()`: Main processing pipeline - starts geckodriver, creates WebDriver, handles navigation, JS execution, form handling, and content extraction
- `ensureFirefox()`/`ensureGeckodriver()`: Platform-aware download and setup of browser dependencies
- `handleForm()`: Form filling with special handling for Phoenix LiveView (detects `[data-phx-session]`)
- `waitForSelector()`/`waitForFunction()`: Selenium wait helpers for page state

### Phoenix LiveView Support

The tool has special handling for Phoenix LiveView pages:
- Detects LiveView via `[data-phx-session]` attribute
- Waits for `.phx-connected` class before proceeding
- Tracks navigation via `phx:page-loading-start`/`phx:page-loading-stop` events
- Injects `window.__phxNavigationState` for navigation tracking

### Console Capture

Two-layer console capture:
1. Injected JavaScript captures `console.log/warn/error/info/debug` to `window.__consoleMessages`
2. Selenium `wd.Log(log.Browser)` captures browser-level errors (JS errors, network errors)

## Testing Notes

- Tests start a local HTTP server on port 9999 with mock pages
- Test profiles are created with `test-*` prefix in `~/.web-firefox/profiles/`
- Screenshots from tests are saved as `test-screenshot-*.png` and cleaned by `make clean`
