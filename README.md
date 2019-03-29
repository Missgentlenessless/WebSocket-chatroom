# WebSocket-chatroom
A chatroom based on websocket


Before the start, please make sure you have installed node.js and mongodb
This is a link to the configuration method of mongodb: 

step1 npm install

step2 Add your native IP to the file: client/containers/index.ts

step3 Add your id and password required for mongodb connection to the file: server/mongoDBHelper.ts
(You can also start the database without the id and password.It depends on your previous database configuration.)

step4 mongod --config e:/chat.conf

step5 npm test

step6 Open the browser and input http://127.0.0.1:8081/index.html

https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/socket/WebSocketSession.html
