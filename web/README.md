I beging with the next room 
<img width="755" height="306" alt="image" src="https://github.com/user-attachments/assets/f155c47b-5e90-4d59-8a9c-7f25e69aa115" />

We have the IP and the only clue "http://10.80.158.20:8080/flag.txt"

If we enter in the IP we don't have access to the flag

<img width="498" height="216" alt="image" src="https://github.com/user-attachments/assets/f5dd1fc6-9c10-4540-8fa8-af3f100829d9" />

Isn't not so easy how pretend

Then, we delete /flag.txt in the URL, we got the kitty shop (sticker shop)

<img width="708" height="638" alt="image" src="https://github.com/user-attachments/assets/9dcde84b-0ae7-4717-b895-b005370dfbd8" />


Use gobuster to enumerate the ip, with a little lucky we can find hidden directories

<img width="559" height="309" alt="image" src="https://github.com/user-attachments/assets/17ca952c-ae60-405b-8d14-213d5735ea63" />


While gobuster ir running, we go to check aroung the page 

In the source code find the diretory /static, that's where the images are stored 

<img width="747" height="299" alt="image" src="https://github.com/user-attachments/assets/2e84cec9-b9be-448b-8d35-c5943216c10a" />


If we use curl in the terminal, we get the error 401 again
<img width="382" height="115" alt="image" src="https://github.com/user-attachments/assets/eb15c71d-beb4-4145-a968-b97c0d56ae02" />


I try to change the method GET to POST
<img width="477" height="182" alt="image" src="https://github.com/user-attachments/assets/30932aa5-b660-4346-ae5d-e1afdd407b1f" />
We get a new error: "405 Method Not Allowed"

Ok, what's happen if we add the -i option? 
<img width="491" height="293" alt="image" src="https://github.com/user-attachments/assets/30d4e5ae-6648-43a1-9fcd-625b9fff75a5" />
We get the methods allowed, OPTIONS, HEAD and GET

Then, we use the HEAD method
<img width="486" height="227" alt="image" src="https://github.com/user-attachments/assets/59bf42df-0034-4bd2-8d79-25770e778b94" />
But, we don't get anything again

