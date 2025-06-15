# Demo of capture ktls plain text using eBPF

## Test environment

Ubuntu 24.04 with HWE kernel (current version 6.11.0)
```
sudo install linux-generic-hwe-24.04
```

For other dependencies, please refer to [nginx](https://github.com/nginx/nginx), [openssl](https://github.com/openssl/openssl) and [libbpf-bootstrap](https://github.com/libbpf/libbpf-bootstrap) documents.

## Build

Download dependencies
```
git submodule update --init --recursive
```

Build nginx with ktls support

```
cd nginx
bash ../nginx-configure.sh
make -j10
```


Build eBPF
```
cd libbpf-bootstrap/
git apply ../ktsl-ebpf-demo.diff
cd examples/c
make sk_msg
```

## Test


Run eBPF
```
sudo ./libbpf-bootstrap/examples/c/sk_msg
```

Tail eBPF trace
```
sudo cat /sys/kernel/debug/tracing/trace_pipe
```

Run nginx
```
bash nginx-run.sh
```

Access to nginx server
```
curl -k https://www.localtest.me:8443/hello.html
```

Messages like following should show in eBPF trace, which includes the plain text HTTP header and body
```
           nginx-8808    [008] ...11  3703.188749: bpf_trace_printk: >>> sk_msg: 276 bytes from 2130706433:-724434944 to 2130706433:8443

           nginx-8808    [008] ...11  3703.188753: bpf_trace_printk: >>> sk_msg: HTTP/1.1 200 OK
Server: nginx/1.28.0
Date: Sun, 15 Jun 2025 21:59:35 GMT
Content-Type: text/html
Content-Length: 40
Last-Modified: Sat, 14 Jun 2025 23:46:46 GMT
Connection: keep-alive
ETag: "684e09e6-28"
Accept-Ranges: bytes

<html><body>hello world\!</body></html>

```