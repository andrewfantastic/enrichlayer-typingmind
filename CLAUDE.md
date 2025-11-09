# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TypingMind plugin for EnrichLayer's B2B data enrichment API. The plugin enables users to access 1.2B+ professional profiles and 50M+ company records directly from TypingMind's AI chat interface.

## Architecture

The codebase consists of a single plugin configuration file (`plugin.json`) that defines three main API functions:

1. **Person Profile Enrichment** (`enrichlayer_person_profile`)
   - Fetches LinkedIn profile data via GET request to `/api/v2/profile`
   - Returns formatted markdown with work history, education, skills
   - Cost: 1 credit (2 credits with `extra=include`)

2. **Company Profile Enrichment** (`enrichlayer_company_profile`)
   - Fetches company data via GET request to `/api/v2/company`
   - Returns formatted markdown with company size, funding, locations
   - Cost: 1 credit base (+ 1 for funding data, + 1 for extra enrichment)

3. **Person Lookup** (`enrichlayer_person_lookup`)
   - Searches for LinkedIn profiles via GET request to `/api/v2/profile/resolve`
   - Requires first_name, last_name, and company_domain parameters
   - Returns profile URL with match quality scores
   - Cost: 2 credits

### Plugin Structure

The plugin uses TypingMind's plugin format with HTTP Action implementations:
- **userSettings**: Defines API key configuration (password field)
- **pluginFunctions**: Array of three function definitions with:
  - `openaiSpec`: OpenAPI-compatible function schema for LLM tool calling
  - `httpAction`: HTTP request configuration with URL templates and headers
  - `implementationType`: "http" (proxied through TypingMind servers to avoid CORS)

### API Integration

All functions:
- Use Bearer token authentication with user-provided API key via HTTP headers
- Accept `use_cache=if-present` parameter for caching
- Use URL template syntax (e.g., `{profile_url}`, `{extra|exclude}` for defaults)
- Return raw JSON responses that the AI interprets

## Development Commands

This plugin doesn't require build steps. To test changes:

1. **Validate JSON syntax**:
   ```bash
   cat plugin.json | python3 -m json.tool > /dev/null && echo "Valid JSON"
   ```

2. **Test in TypingMind**: Import the plugin using the GitHub repository URL:
   ```
   https://github.com/andrewfantastic/enrichlayer-typingmind
   ```

## API Rate Limits

- Trial accounts: 2 requests/minute
- Standard accounts: 300 requests/minute

## Key Implementation Details

- **HTTP Action Implementation**: Uses `implementationType: "http"` to bypass CORS restrictions (EnrichLayer API doesn't support CORS for browser requests)
- All API calls use `use_cache=if-present` to optimize credit usage
- URL template syntax with defaults: `{extra|exclude}` means "use {extra} or default to 'exclude'"
- Person lookup uses `/api/v2/profile/resolve` endpoint (not `/person/lookup`)
- Profile URLs must be full LinkedIn URLs (e.g., `https://www.linkedin.com/in/username`)
- AI receives raw JSON and formats it for the user (no custom markdown transformation)
