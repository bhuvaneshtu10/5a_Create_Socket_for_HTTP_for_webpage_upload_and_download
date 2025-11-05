# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
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

```
import socket
import threading
import time

# Simple server function to serve a static file
def start_server(host='127.0.0.1', port=8080):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(1)
    print(f"Server listening on {host}:{port}")

    while True:
        client_sock, addr = server_socket.accept()
        print(f"Connection from {addr}")
        request = client_sock.recv(1024).decode()
        print(f"Request:\n{request}")

        # Simple HTTP GET handling
        if request.startswith("GET"):
            filename = request.split(" ")[1][1:]  # Extract filename from GET /filename HTTP/1.1
            try:
                with open(filename, "rb") as f:
                    content = f.read()
                response_headers = "HTTP/1.1 200 OK\r\n"
                response_headers += f"Content-Length: {len(content)}\r\n"
                response_headers += "Content-Type: application/octet-stream\r\n"
                response_headers += "Connection: close\r\n\r\n"
                client_sock.sendall(response_headers.encode() + content)
            except FileNotFoundError:
                response = "HTTP/1.1 404 Not Found\r\nConnection: close\r\n\r\nFile not found"
                client_sock.sendall(response.encode())
        else:
            response = "HTTP/1.1 400 Bad Request\r\nConnection: close\r\n\r\n"
            client_sock.sendall(response.encode())

        client_sock.close()

# Client function to send GET request and save file
def send_request(host, port, request):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        s.sendall(request.encode())
        response = b""
        while True:
            data = s.recv(4096)
            if not data:
                break
            response += data
    return response

def download_file(host, port, filename):
    request = f"GET /{filename} HTTP/1.1\r\nHost: {host}\r\nConnection: close\r\n\r\n"
    response = send_request(host, port, request)
    headers, _, body = response.partition(b"\r\n\r\n")
    headers_text = headers.decode(errors="ignore")
    if "200 OK" not in headers_text:
        print("Error: Server did not return 200 OK")
        print(headers_text)
        return
    with open("downloaded_" + filename, "wb") as f:
        f.write(body)
    print(f"✅ File '{filename}' downloaded successfully as 'downloaded_{filename}'")

if __name__ == "__main__":
    host = "127.0.0.1"
    port = 8080
    filename = "example.txt"

    # Create a sample file to serve
    with open(filename, "w") as f:
        f.write("This is a sample file for testing.\n")

    # Start server in a separate thread
    server_thread = threading.Thread(target=start_server, args=(host, port), daemon=True)
    server_thread.start()

    # Wait a moment for server to start
    time.sleep(1)

    # Run the client to download the file
    download_file(host, port, filename)

```
## OUTPUT

<img width="783" height="246" alt="image" src="https://github.com/user-attachments/assets/722e038c-1105-4bc4-a662-b11aacec1629" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
