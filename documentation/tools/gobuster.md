# Gobuster

## Introduction

Gobuster is a versatile directory, DNS, and virtual-host brute-forcing tool written in Go. It is commonly used during web application recon to discover hidden directories, files, virtual hosts and subdomains.

## Wordlists

Good wordlists are essential. Common sources:

- SecLists (`/usr/share/wordlists/dirbuster/`, `/usr/share/seclists/Discovery/Web-Content/`)

Examples:

```text
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

## Common Modes

- `dir` directory/file discovery (default)
- `dns` DNS subdomain enumeration
- `vhost` virtual host discovery

## Basic directory scan

```bash
gobuster dir -u https://example.com -w /path/to/wordlist.txt
```

Useful flags:

- `-u, --url` : target URL
- `-w, --wordlist` : wordlist path
- `-t, --threads` : number of concurrent threads (e.g., 50)
- `-x, --extensions` : file extensions to append (e.g., php,html)
- `-s, --status-codes` : comma-separated status codes to show (default 200,204,301,302,307,401,403)
- `-o, --output` : write results to a file
- `-k` : skip TLS verification (for self-signed certs)

Example with extensions and threads:

```bash
gobuster dir -u https://example.com -w common.txt -x php,html,txt -t 50 -o gobuster_results.txt
```

## Filtering results by status codes

You can limit output to certain status codes with `-s`:

```bash
gobuster dir -u https://example.com -w common.txt -s 200,301,302,403
```

Common approach: show `200` (OK) and `403` (Forbidden) since `403` can reveal interesting protected endpoints.

## Virtual host discovery

If the target hosts multiple sites on the same IP, use `vhost` mode:

```bash
gobuster vhost -u https://10.10.10.10 -w vhosts.txt -t 30 -k
```

This sets the `Host:` header to candidates from `vhosts.txt` and checks responses. Look for differences in status code, title, content length, or response body to identify valid vhosts.

## DNS subdomain enumeration

```bash
gobuster dns -d example.com -w subdomains.txt -t 50
```

Use a DNS resolver list or your own resolver with `-r` to avoid DNS blocking.

## Recursive scanning

Gobuster can be combined with recursive scripts or tools that feed discovered directories back into Gobuster. There is no built-in deep recursion; orchestration is typically done with custom scripts.

## Tips & Best Practices

- Start with a medium-sized wordlist, then iterate with larger lists if needed.
- Use `-t` to tune performance. Too many threads can overwhelm the target or your network.
- Prefer HTTPS (`https://`) when possible to avoid redirections hiding results.
- Use `-x` to find files such as `.php`, `.bak`, `.old`, `.inc`.
- Collect `Content-Length`, `Title`, and status codes to triage results quickly.
- Combine Gobuster with a browser or `curl` to verify promising findings.
- Respect engagement rules and scope while pentesting.

## Examples

Directory scan with common extensions:

```bash
gobuster dir -u https://example.com -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,txt -t 50 -o gobuster_dir.txt
```

Virtual-host discovery against an IP:

```bash
gobuster vhost -u https://10.10.10.10 -w ~/wordlists/vhosts.txt -t 40 -k -o gobuster_vhost.txt
```

DNS subdomain enumeration using a resolver:

```bash
gobuster dns -d example.com -w subdomains.txt -r 8.8.8.8:53 -t 50 -o gobuster_dns.txt
```

## Further reading

- Gobuster GitHub: https://github.com/OJ/gobuster
- SecLists: https://github.com/danielmiessler/SecLists

Apply filters and follow-up enumeration to validate discoveries.
