---
title: "Process"
draft: false
tags:
---
### Making a Meta developer account

  

first of all, you have to go and make a developer account on the meta developers platform: [Registration Dialog - Meta for Developers](https://developers.facebook.com/async/registration/dialog/?src=default),

  

after waiting for your account to be approved,  you can create an application for your platform

  

Then you will be able to add the WhatsApp product to your application.

  

### Acquiring the API Token for your application

  

Then, you’ll be provided with a token to use while sending calls to the Meta API. At first, it’s going to be a temporary token, but then you can ask for a permanent one on the API setup page.

  

in **WhatsApp** > **API Setup**

  

### Setup Number list for testing

  

At first, you’ll be provided a test number you can send from until you add a real number and by adding your bill details you’ll be able to use it as the main sending number.

  

Under **Send and receive messages**, select the **To** field and choose **Manage phone number list**.

You can add any valid WhatsApp number as a recipient. The recipient number will receive a confirmation code in WhatsApp that can be used to verify the number.

  

### Send a test message

  

```jsx

Await Axios. post(

      `https://graph.facebook.com/v21.0/${FROM_PHONE_NUMBER_ID}/messages`,

      {

        messaging_product: "whatsapp",

        recipient_type: "individual",

        to: `${TO_PHONE_NUMBER}`,

        type: "text",

        text: {

          preview_url: `${MESSAEG_URL_PREVIEW}`, //optional make it false

          body: `${MESSAEG_BODY_TEXT}`,

        },

      },

      {

        headers: {

          Authorization: `Bearer ${ACCESS_TOKEN}`,

          "Content-Type": "application/JSON",

        },

      }

    );

```

  

Using a simple setup like this one you can send a message to the allowed numbers on your list.

  

To do so, you’ll be able to send messages using your application.

  

to send more types of messages [Messages - Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages)

  

## Setting the webhooks

  

Now we can send, but for us to be able to receive those messages users send to us on our custom portal, we’ll need to set up a webhook, for WhatsApp to come and send us the messages we receive.

  

Let’s start with setting up the endpoint.

  

On your backend ( suppose NodeJS )

  

You’ll have to set a simple hook that WhatsApp can call to provide you with all the messages you receive

  

```jsx

app.post("/webhook", whatsappWebhookHandler);

```

  

But for the first time, Meta wants to make sure that your endpoint is working, so you have to add this to your webhook for the first time only.

  

```jsx

app.get('/webhook', (req, res) => {

  const challenge = req.query['hub.challenge'];

  if (challenge) {

    res.status(200).send(challenge);

  } else {

    res.status(400).send('Bad Request');

  }

});

```

  
After the webhook is added like this here.
![[image.png]]
  
You can hit manage and subscribe to all the webhooks you need to receive
![[image(1).png]]
  
## Setting up the webhook handler

  

In your webhook handler you wanna make sure you’re handling all types of messages you can get

  

A generic webhook call will look something like this.

  

```jsx

{

  "object": "whatsapp_business_account",

  "entry": [{

      "id": "WHATSAPP_BUSINESS_ACCOUNT_ID",

      "changes": [{

          "value": {

              "messaging_product": "whatsapp",

              "metadata": {

                  "display_phone_number": "PHONE_NUMBER",

                  "phone_number_id": "PHONE_NUMBER_ID"

              },

              # specific Webhooks payload            

          },

          "field": "messages"

        }]

    }]

}

```

  

But of course, you’ll receive a different webhook for each of type of message ( message, reaction, media (images, docs, etc..) etc..)

  

Something will look pretty much like
  
```jsx

{

  "object": "whatsapp_business_account",

  "entry": [{

    "id": "WHATSAPP-BUSINESS-ACCOUNT-ID",

    "changes": [{

      "value": {

         "messaging_product": "WhatsApp",

         "metadata": {

           "display_phone_number": "PHONE-NUMBER",

           "phone_number_id": "PHONE-NUMBER-ID"

         },

      # Additional arrays and objects

         "contacts": [{...}]

         "errors": [{...}]

         "messages": [{...}]

         "statuses": [{...}]

      },

      "field": "messages"

    }]

  }]

}

```


Look for more details here [Webhooks - Cloud API](https://developers.facebook.com/docs/whatsapp/cloud-api/webhooks/components).

  
![[image(2).png]]
![image.png]

  

Now let’s see what that might look like in our app.

  

```jsx

export const whatsappWebhookHandler = async (

  req: Request,

  res: Response,

  next: NextFunction

) => {

  try {

    const payload = req.body;

    if (payload.object === "whatsapp_business_account") {

      if (payload.entry[0].changes[0].value.statuses) {

        // Call updateMessageStatus if it's a status update

        await updateMessageStatus(payload);

      } else if (payload.entry[0].changes[0].value.messages[0].reaction) {

         // Call updateMessageReaction if it's a reaction update

        await updateMessageReaction(payload);

      } else {

        // Call saveMessage for a new message

        await saveMessage(payload);

      }

      return res

        .status(200)

        .json({ success: true, message: "Message processed successfully" });

    }

    res

      .status(400)

      .json({ success: false, message: "Invalid webhook payload" });

  } catch (error) {

    console.error("Error processing WhatsApp webhook:", error);

    res.status(500).json({ success: false, message: "Server error" });

  }

};

```

  

In the (saveMessage, updateMessageStatus, updateMessageReaction).

  

You can implement your logic for handling the database saving of your messages and interactions.

  

Don’t forget to adapt your database to your liking to catch and save all useful data that might help you in your app.
