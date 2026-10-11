### Assignment: Part 1

1. **Idempotency In Practice**:

````text
I've looked my bhalobasha backend APIs and found this post api ```ts
@Post('send-otp')
  @ApiOperation({ summary: 'Send OTP to phone number' })
  sendOtp(@Body() dto: SendOtpDto) {
    return this.authService.sendOtp(dto);
  }
```
is not idempotent by default. Because POST api shouldn't maintain idempotency by default, it should create multiple resources for every request.

Now to make this api idempotent I have to remove body from request and have to fix a fixed dto so that every time i get only that default data for sending otp. But that is not convenient way because others user won't be able to get otp. That's why I need idempotency free POST api for this.
````

2. **Pagination tradeoff**:

```text
For this endpoint which one is getting more row day by day and current size is 500K+ rows I will use cursor-based pagination. Because It's already 5lakhs plus rows that means it's already in high volume data. So if I offset-based pagination for this I will face duplications or skipping problem for this. So, for maintaining accuracy of data i have to use cursor-based pagination.
```

3. **Protocol choice**:

```text
For adding a live typing idicator i would choose WebSockets undoubtly. And I've already built this feature with websocket that was too accurate. I would use this because websocket built connection with service to service on top of the http without closing connection. So when clients listen each of them i can send a message each of them that one or both is typing from each other and easily i can show typing indicator in this way. So, if i choose http polling or UDP i need more time and process to show typing indicator that's why i wanna show user quicly typing or not and easy communication for all with websocket connection i can show typing indicator easily.
```

4. **TLS termination reasoning**:

```text
If we terminate session in backend server and our data goes decrypted it will be still exposed. So i think terminating TLS at a load balancer is the most secure way to keep my data encrypted.

```

5. **Status code audit**:

```text
I've audit my bhalobasha server 5 endpoints all are sending correct status code so i didn't change anything

```
