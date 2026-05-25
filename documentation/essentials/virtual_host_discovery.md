# Virtual Host Discovery

## Introduction

Virtual host discovery explains how servers host multiple sites on a single IP and why resolving the correct hostname matters during web testing.

The server selects which site to serve based on the HTTP `Host:` header, for example:

```
Host: academy.htb
```

If the domain doesn't resolve correctly on your machine, you may:

- see a default Apache/Nginx page
- be redirected unexpectedly
- fail to reach the real site

Add an hostname entry to `/etc/hosts` to resolve the domain locally:

```bash
sudo nano /etc/hosts
```

Add a line like:

```
10.129.229.183 academy.htb
```

This forces the local machine to resolve:

```
academy.htb → 10.129.229.183
```

Then access the site using the hostname (e.g., `https://academy.htb`) so the server receives the correct `Host:` header.

