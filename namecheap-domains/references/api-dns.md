# Namecheap DNS API Reference

Detailed parameter reference for `namecheap.domains.dns.*` commands.

All DNS commands require the domain split into SLD and TLD parameters. For `example.com`: SLD=`example`, TLD=`com`. For `example.co.uk`: SLD=`example`, TLD=`co.uk`.

## domains.dns.setDefault

Sets domain to use Namecheap default DNS servers. Required for host record management, URL forwarding, email forwarding, dynamic DNS, and other value-added services.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| SLD | String | Yes | Second-level domain (e.g., `example`) |
| TLD | String | Yes | Top-level domain (e.g., `com`) |

### Response

Returns `DomainDNSSetDefaultResult` with `Domain` and `Updated` (true/false) attributes.

---

## domains.dns.setCustom

Sets domain to use custom DNS servers. URL forwarding, email forwarding, and dynamic DNS will not work with custom nameservers.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| SLD | String | Yes | Second-level domain |
| TLD | String | Yes | Top-level domain |
| Nameservers | String | Yes | Comma-separated list of nameservers |

### Response

Returns `DomainDNSSetCustomResult` with `Domain` and `Update` (true/false) attributes.

---

## domains.dns.getList

Gets the list of DNS servers associated with a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| SLD | String | Yes | Second-level domain |
| TLD | String | Yes | Top-level domain |

### Response

Returns `DomainDNSGetListResult` with `Domain`, `IsUsingOurDNS` attributes, and `Nameserver` child elements.

---

## domains.dns.getHosts

Retrieves DNS host record settings for a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| SLD | String | Yes | Second-level domain |
| TLD | String | Yes | Top-level domain |

### Response

Returns `host` elements with attributes:

| Attribute | Description |
|-----------|-------------|
| HostId | Unique ID for the record |
| Name | Hostname (e.g., `@`, `www`, `mail`) |
| Type | Record type: `A`, `AAAA`, `CNAME`, `MX`, `MXE`, `TXT`, `URL`, `URL301`, `FRAME` |
| Address | Record value (IP, hostname, URL, or text) |
| MXPref | MX priority (for MX records) |
| TTL | Time to live in seconds |
| AssociatedAppTitle | Associated app (if any) |
| FriendlyName | Human-readable name |
| IsActive | Whether the record is active |
| IsDDNSEnabled | Whether dynamic DNS is enabled |

---

## domains.dns.getEmailForwarding

Gets email forwarding settings for a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain name |

### Response

Returns `Forward` elements with `mailbox` (source) and `ForwardTo` (destination) attributes.

---

## domains.dns.setEmailForwarding

Sets email forwarding for a domain.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| DomainName | String | Yes | Domain name |
| MailBox1 | String | Yes | Mailbox alias (e.g., `info`) |
| ForwardTo1 | String | Yes | Destination email address |
| MailBox2 | String | No | Additional mailbox alias |
| ForwardTo2 | String | No | Additional destination |
| ... | ... | ... | Numbered pairs up to MailBoxN/ForwardToN |

Example: to forward `info@example.com` to `user@gmail.com`:
- `MailBox1=info`
- `ForwardTo1=user@gmail.com`

---

## domains.dns.setHosts

Sets DNS host records for a domain. This replaces ALL existing records — include every record you want to keep.

### Request Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| SLD | String | Yes | Second-level domain |
| TLD | String | Yes | Top-level domain |
| HostName1 | String | Yes | Hostname for record 1 (e.g., `@`, `www`) |
| RecordType1 | String | Yes | Record type: `A`, `AAAA`, `CNAME`, `MX`, `MXE`, `TXT`, `URL`, `URL301`, `FRAME` |
| Address1 | String | Yes | Record value |
| MXPref1 | Number | No | MX priority (required for MX records, default: 10) |
| TTL1 | Number | No | TTL in seconds (default: 1800) |
| HostName2 | String | No | Hostname for record 2 |
| RecordType2 | String | No | Record type for record 2 |
| Address2 | String | No | Value for record 2 |
| ... | ... | ... | Continue numbering for additional records |

### Record Types

| Type | Description | Address Format |
|------|-------------|----------------|
| A | IPv4 address | `1.2.3.4` |
| AAAA | IPv6 address | `2001:db8::1` |
| CNAME | Canonical name | `target.example.com` |
| MX | Mail exchange | `mail.example.com` (set MXPref) |
| MXE | Mail exchange (IP) | `1.2.3.4` |
| TXT | Text record | `v=spf1 include:...` |
| URL | URL redirect (302) | `http://example.com` |
| URL301 | URL redirect (301) | `http://example.com` |
| FRAME | URL frame | `http://example.com` |

### Response

Returns `DomainDNSSetHostsResult` with `Domain` and `IsSuccess` attributes.

### Common Patterns

**Basic website setup:**
```
HostName1=@       RecordType1=A       Address1=1.2.3.4       TTL1=1800
HostName2=www     RecordType2=CNAME   Address2=example.com   TTL2=1800
```

**Email setup (Google Workspace):**
```
HostName1=@   RecordType1=MX   Address1=aspmx.l.google.com        MXPref1=1    TTL1=1800
HostName2=@   RecordType2=MX   Address2=alt1.aspmx.l.google.com   MXPref2=5    TTL2=1800
HostName3=@   RecordType3=MX   Address3=alt2.aspmx.l.google.com   MXPref3=5    TTL3=1800
HostName4=@   RecordType4=TXT  Address4=v=spf1 include:_spf.google.com ~all     TTL4=1800
```

**Domain verification (TXT record):**
```
HostName1=@   RecordType1=TXT   Address1=google-site-verification=XXXXX   TTL1=1800
```
