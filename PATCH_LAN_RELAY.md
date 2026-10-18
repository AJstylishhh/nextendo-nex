# LAN pair-relay patch

`PublicIPOf` in `relaypair.go` normally returns empty for private (RFC1918) addresses.
That blocks pair-relay when both peers are on the same Wi-Fi (Switch + Citron).

Apply this change in `relaypair.go` function `PublicIPOf`:

```go
func PublicIPOf(c *Connection) string {
	if c == nil {
		return ""
	}
	host, _, err := net.SplitHostPort(c.RemoteAddr)
	if err != nil || host == "" {
		return ""
	}
	if isPrivateIP(host) {
		if os.Getenv("NEXTENDO_RELAY_ALLOW_PRIVATE") == "1" {
			return host
		}
		return ""
	}
	return host
}
```

Then in dying-light-nex set `NEXTENDO_RELAY_ALLOW_PRIVATE=1` (main.go v9 does this).
