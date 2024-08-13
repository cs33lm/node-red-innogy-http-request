# node-red-innogy-http-request
HTTP request nodes to control Innogy Professional Wallbox

Following nodes will
- Open the wallbox interface to request a cookie
- Login in using the provides credentials
- Submit the remote control form to start charging

## Prerequisites
Remote control must be active (if you use an RFID card - authenticate via card, then copy the corresponding contract ID from Remote Control Section via webinterface). 
Test with connected cable without card if manually submitted contract ID starts the charging process).

Please check code below and change ip addresses and credentials according to your environment.

## Limitations
Please build your own your logic around when to trigger the authentication process. I grab the cable connected status via EVCC api and store globally if authentification was triggered. 

I sometimes notice that my car stops charging even if surplus solar power is available. I added a notification (node red analyzes the EVCC api data and sends a push alert via Home Assistant) and a manual switch (Home Assistant) to re-auth. 

# Code
1. function node (grab cookie):
```
msg = {};
msg.url= "https://IP:Port";
msg.method = "GET";
return msg;
```
2. function node (login):
```
let cookie = msg.responseCookies.ecu_session.value;
msg = {};
msg.payload = {
    username: "admin",
    password: "password"
}
msg.headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
}
msg.url= "https://IP:Port/cgi_c_login";
msg.method = "POST";
msg.cookies = {
    ecu_session: cookie
}
return msg;
```
3. function node (submit LDP1-Remote Control):
```
let cookie = msg.responseCookies.ecu_session.value;
msg = {};
msg.payload = {
    "waittime": "0",
    "chargetime": "0",
    "contractID": "YOUR_RFID_HERE",
    "rfid_uid": "YOUR_RFID_HERE",
    "start_stop": "0",
    "currentByContract": "50"
}
msg.headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
}
msg.url= "https://IP:Port/cgi_c_ldp1.remote_control";
msg.method = "POST";
msg.cookies = {
    ecu_session: cookie
}
return msg;
```
4. add HTTP request nodes (method via msg.method, SSL without cert validation due to the self signed wallbox cert)

## Final result:

![nr](https://github.com/user-attachments/assets/546d7374-dd47-4b06-9051-713f175071e2)



