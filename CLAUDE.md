# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based video parsing service that removes watermarks from videos across 20+ Chinese social media platforms. The project provides three interfaces:
- HTTP API (web server)
- MCP (Model Context Protocol) server for AI assistant integration
- Go library for programmatic use

## Development Commands

### Building and Running
```bash
# Run HTTP server (default mode: HTTP + MCP SSE)
go run main.go

# Run modes
go run main.go --http          # HTTP server only
go run main.go --mcp           # MCP server with stdio transport
go run main.go --mcp-sse       # MCP server with SSE transport
go run main.go --mcp-port 9000 # Custom MCP SSE port (default: 8081)
go run main.go --both          # Both HTTP and MCP servers (default)

# Run with basic auth
export PARSE_VIDEO_USERNAME=your_username
export PARSE_VIDEO_PASSWORD=your_password
go run main.go

# Build the binary
go build -o main ./main.go
```

### Testing
```bash
# Run all tests
go test ./...

# Run specific test file
go test ./parser/douyin_test.go

# Run tests with verbose output
go test -v ./...
```

### Docker
```bash
# Build Docker image
docker build -t parse-video .

# Run container
docker run -d -p 8080:8080 parse-video

# Run with basic auth
docker run -d -p 8080:8080 -e PARSE_VIDEO_USERNAME=user -e PARSE_VIDEO_PASSWORD=pass parse-video
```

## Architecture

### Core Components

1. **Parser System** (`parser/`):
   - `parser.go`: Main entry point with URL routing logic
   - `vars.go`: Defines platform constants, interfaces, and data structures
   - Platform-specific parsers (e.g., `douyin.go`, `kuaishou.go`, `bilibili.go`, `twitter.go`)

2. **Web Server** (`main.go`):
   - Gin-based HTTP server on port 8080
   - Basic auth middleware (optional)
   - Embedded HTML templates
   - Two main endpoints: `/video/share/url/parse` and `/video/id/parse`

3. **MCP Server** (`mcp/`):
   - `server.go`: MCP server initialization and transport modes (stdio/SSE)
   - `tools.go`: 5 MCP tools for video parsing operations
   - `resources.go`: Platform information resources
   - `types.go`: Type conversion utilities

4. **Utilities** (`utils/`):
   - `utils.go`: URL extraction utilities using regex

### Key Design Patterns

- **Strategy Pattern**: Each platform has its own parser implementing `videoShareUrlParser` and `videoIdParser` interfaces
- **Factory Pattern**: `videoSourceInfoMapping` maps platform identifiers to their respective parsers
- **Interface Segregation**: Separate interfaces for share URL parsing and video ID parsing

### Data Flow

1. **Share URL Parsing**:
   - Extract URL from input string using regex
   - Match URL domain to platform in `videoSourceInfoMapping`
   - Call platform-specific `parseShareUrl()` method

2. **Video ID Parsing**:
   - Direct lookup by platform source and video ID
   - Call platform-specific `parseVideoID()` method

3. **Batch Processing**:
   - Concurrent parsing using goroutines and sync.WaitGroup
   - Thread-safe result collection with mutex

### MCP Tools

The MCP server provides 5 tools:
- `parse_video_share_url`: Parse video from share URL
- `parse_video_id`: Parse video by platform and ID
- `batch_parse_video_id`: Batch parse multiple videos
- `extract_url_from_text`: Extract URLs from text
- `get_supported_platforms`: List all supported platforms

## Platform Support

The project supports 20+ video platforms and 5 image album platforms. Each platform is defined in `vars.go` with:
- Unique source identifier (e.g., `SourceDouYin`)
- Associated domains for URL matching
- Parser implementation

Key platforms include:
- Video: 抖音, 快手, 小红书, 微博, 西瓜视频, 哔哩哔哩, Twitter/X, etc.
- Image Albums: 抖音, 快手, 小红书, 皮皮虾, 微博
- LivePhoto: 小红书

## Configuration

### Environment Variables
- `PARSE_VIDEO_USERNAME`: Basic auth username (optional)
- `PARSE_VIDEO_PASSWORD`: Basic auth password (optional)

### Dependencies
- `github.com/gin-gonic/gin`: Web framework
- `github.com/go-resty/resty/v2`: HTTP client
- `github.com/tidwall/gjson`: JSON parsing
- `github.com/PuerkitoBio/goquery`: HTML parsing
- `github.com/mark3labs/mcp-go`: MCP (Model Context Protocol) implementation

## Adding New Platforms

1. Add source constant in `vars.go`
2. Create platform parser file implementing interfaces
3. Add mapping in `videoSourceInfoMapping`
4. Write unit tests for new parser
5. Update README.md with platform support
