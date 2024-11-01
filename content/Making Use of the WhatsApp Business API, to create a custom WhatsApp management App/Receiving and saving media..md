---
title: "Receiving and saving media."
draft: false
tags:
---
Now let’s see the tricky part:
Since WhatsApp is an end-to-end encryption (it doesn’t save your messages or media files ) it will not be available in some sort of storage on their end.

  

It’s gonna need a specific process to get the media you’re looking for.

  

### 1- Saving the message itself

  

We are already handling getting the message in our webhook which will contain

  

```jsx

caption — String. Caption for the document, if provided.

filename — String. Name for the file on the sender's device.

sha256 — String. SHA 256 hash.

mime_type — _String. _ Mime type of the document file.

id — String. ID for the document.

```

  

These data that you have to save initially in your database so you can get the file.

  

### 2- Get the file URL

  

since you already have the necessary info about your media, you can call an API with that info to get the URL for your file

  

Be careful: this URL will be available just for the next 5 minutes from the user sending the file.

  

```jsx

const response = await axios.get(

    `https://graph.facebook.com/v21.0/${mediaId}/`,

    {

      headers: {

        Authorization: `Bearer ${ACCESS_TOKEN}`,

      },

    }

  );

```

  

You’ll have to provide the media ID that you’ve already saved in the previous step.

  

### 3- Getting the file itself

  

Now that you have already the file URL you can call a different API to get the file using the URL that you have now

  

```jsx

Const file = await axios.get(url, {

      headers: {

        Authorization: `Bearer ${ACCESS_TOKEN}`,

      },

      responseType: "arraybuffer",

    });

```

  

This way you have the file in the form of binary data

  

```jsx

const fileData = Buffer.from(file .data, 'binary').toString('base64');

```

  

Like you have the file in the form of base64 text that can be stored in your database and ready to be sent to the front whenever you like and don’t have to ask Meta to give it to you again

  

Using these steps you’re pretty much done for sending and receiving any type of message you want to send or receive from the user.