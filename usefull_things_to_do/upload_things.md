You may need to upload something on a machine without network.

To do it, you can open a http server on localhost :

- `python3 -m http.server 1234`

You need to get your IP :

- `ip -br a`

Then, you can wget it from the machine without network :

- `wget http://<ip_addr>:1234/<file_name>`