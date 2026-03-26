# Exp:5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
### NAME: HARI PRIYA M
### REGISTER NO : 212224240047
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm

1.Start the program.
<BR>
2.Get the frame size from the user
<BR>
3.To create the frame based on the user request.
<BR>
4.To send frames to server from the client side.
<BR>
5.If your frames reach the server it will send ACK signal to client otherwise it will send NACK signal to client.
<BR>
6.Stop the program
<BR>
## Program 
### server.py
```python
import socket
import os

HOST = "127.0.0.1"
PORT = 8080

def handle_client(conn):
    request = b""
    while True:
        chunk = conn.recv(4096)
        request += chunk
        if len(chunk) < 4096:
            break

    request_text = request.decode(errors="ignore")
    first_line = request_text.split("\r\n")[0]
    method = first_line.split(" ")[0]

    print(f"\n[SERVER] Received {method} request")

    if method == "POST":
        if "\r\n\r\n" in request_text:
            headers, body = request_text.split("\r\n\r\n", 1)
        else:
            body = ""

        with open("example.txt", "w") as f:
            f.write(body)

        print("[SERVER] File uploaded and saved as 'example.txt'")
        response = (
            "HTTP/1.1 200 OK\r\n"
            "Content-Type: text/plain\r\n"
            "\r\n"
            "File uploaded successfully!"
        )
        conn.sendall(response.encode())

    elif method == "GET":
        path = first_line.split(" ")[1].lstrip("/")
        print(f"[SERVER] Client requested file: '{path}'")

        if os.path.exists(path):
            with open(path, "rb") as f:
                file_data = f.read()
            response_headers = (
                "HTTP/1.1 200 OK\r\n"
                "Content-Type: text/plain\r\n"
                f"Content-Length: {len(file_data)}\r\n"
                "\r\n"
            )
            conn.sendall(response_headers.encode() + file_data)
            print(f"[SERVER] File '{path}' sent to client")
        else:
            response = (
                "HTTP/1.1 404 Not Found\r\n"
                "Content-Type: text/plain\r\n"
                "\r\n"
                "404 File Not Found"
            )
            conn.sendall(response.encode())
            print(f"[SERVER] File '{path}' not found")

    conn.close()

def start_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(5)
    print(f"[SERVER] Listening on {HOST}:{PORT} ...")

    while True:
        conn, addr = server_socket.accept()
        print(f"[SERVER] Connection from {addr}")
        handle_client(conn)

if __name__ == "__main__":
    start_server()

```

### client.py
```python
import socket

def send_request(host, port, request):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    s.sendall(request.encode())

    response = b""
    while True:
        data = s.recv(4096)
        if not data:
            break
        response += data

    s.close()
    return response.decode(errors="ignore")


def upload_file(host, port, filename):
    with open(filename, "rb") as file:
        file_data = file.read()

    content_length = len(file_data)

    request = (
        f"POST /upload HTTP/1.1\r\n"
        f"Host: {host}\r\n"
        f"Content-Length: {content_length}\r\n"
        f"\r\n"
    )

    request = request.encode() + file_data

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    s.sendall(request)

    response = s.recv(4096)
    s.close()

    return response.decode(errors="ignore")


def download_file(host, port, filename):
    request = f"GET /{filename} HTTP/1.1\r\nHost: {host}\r\n\r\n"

    response = send_request(host, port, request)

    parts = response.split("\r\n\r\n", 1)
    if len(parts) > 1:
        file_content = parts[1]
        with open(filename, "wb") as file:
            file.write(file_content.encode())


if __name__ == "__main__":
    host = "127.0.0.1"
    port = 8080

    upload_response = upload_file(host, port, "example.txt")
    print("Upload response:", upload_response)

    download_file(host, port, "example.txt")
    print("File downloaded successfully.")
```


## OUTPUT
<img width="1634" height="316" alt="image" src="https://github.com/user-attachments/assets/76672aab-c328-4811-a031-578d8137483b" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
