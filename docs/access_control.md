# Access control

Via ACL (Access Control Lists) you can specify which resources can be accessed within your tailnet.  It is quite a large topic in the Tailscale documention, so we will not go into too much detail here.

Let's stick to a simple example.  You have 1 server (i.e. your Raspberry running Node-RED) and multiple clients (i.e. Android smartphones, Windows 10 portable, ...).  You want the clients to be able to access the server, but the clients should not be able to access each other.  Because you want to minimize risks within your tailnet, in case one the device would be infected (before you could remove it from your tailnet).

1. Login to the admin console.
2. Create 2 custom tags for servers and clients:

   ![image](https://github.com/user-attachments/assets/35d1a210-f60c-4c6f-9545-19757fa1d347)

3. In the *"Machines"* tabsheet, click on the `...` button of a device.  And choose this menu item:

   ![image](https://github.com/user-attachments/assets/bf40c434-0500-4afa-b985-35ee7e896e1e)

4. Now select one of the 2 tags we created in the first step:

   ![image](https://github.com/user-attachments/assets/de2cb108-ed90-4901-91b8-a8a087a4f9d2)

5. After you have repeated this for every device, the tags will be clearly displayed:

   ![image](https://github.com/user-attachments/assets/71deecee-b94f-4c6a-b6ce-24a0d19bf1a2)

6. Now create an ACL list to specify that clients should only be able to access servers:

   ![image](https://github.com/user-attachments/assets/2fce6a51-7301-446b-8b55-addddbe841a4)

7. The Tailnet control servers will distribute these ACL lists to all the Tailscale agents in your tailnet.  When 2 Tailscale agents communicate with each other, they will ***both*** check whether he is allowed to talk to the other agent.
8. You will now see for example in your Android Tailscale app, that only the devices are visible to which your Android device has access.
