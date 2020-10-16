Creating a Telegram Bot
***********************

Create a Telegram Account
=========================

Go to https://web.telegram.org/ and follow the steps to create an account.

Set Up a Bot
============

1. Add https://t.me/botfather to your bots on Telegram
2. Ask the BotFather to create a bot by sending him the following message:

.. code:: text

    /newbot

3. Give the bot a name, this one does not have to be unique. Just answer the Botfather with your preferred name.
4. Give the bot a usernmae, this one has to be unique. Again jsut write it as a text message into the chat.
5. Click on the link you receive from the botfather to connect to your bot. Write down the token the Botfather sent you as well. If you lost the key, enter /mybots in the Botfather chat, click on the bot and click on "API Token"
6. Send the botfather the following message to edit your newly created bot:

.. code:: text

    /mybots

7. Click on the username for the bot you just created.
8. Click on "Edit Bot"
9. Click on "Edit Description", enter the description as a text message and send it.
10. Click "Edit Commands" and Send the following message:

.. code:: text

    plotc0 - create a current plot fot C0 and send it
    plotc1 - create two plots for C1 once avg and once sum

Congratualtions! You just successfully created your bot!

Find out your Chat ID
======================

The chat id is needed for the bot to send you messages without you requesting it beforehand. Thus he can initiate messages.

1. If not yet done, connect to your bot by:
    - Writing /mybots to the botfather
    - Clicking on the bot
    - Clicking on the name with the "@" in the message the BotFather sent you
    - Clicking the button START button
2. Send your bot a message, the content is not important
3. Enter the following link into your browser, replacing <token> with the token of your bot:

..code:: text

    https://api.telegram.org/bot<token>/getUpdates

4. Your should a json in a form like the following (the fields with *** are censored, due to privacy reasons):

.. code:: text

    {"ok":true,"result":[{"update_id":******,"message":{"message_id":2 ,"from":{"id": *** ,"is_bot":false,"first_name":"***","last_name":"***","language_code":"en"},"chat":{"id":***,"first_name":"***","last_name":"***","type":"private"},"date":1602834899,"text":"Hello"}}]}

5. If you want, you pretty print the code with the code editor of your choice, or with an online tool like https://jsonformatter.org/json-pretty-print:

.. code:: json

    {
        "ok": true,
        "result": [
            {
                "update_id": ******,
                "message": {
                    "message_id": 2,
                    "from": {
                    "id": ***,
                    "is_bot": false,
                    "first_name": "***",
                    "last_name": "***",
                    "language_code": "en"
                    },
                "chat": {
                    "id": ***,
                    "first_name": "***",
                    "last_name": "***",
                    "type": "private"
                    },
                    "date": 1602834899,
                    "text": "Hello"
                }
            }
        ]
    }

6. As you see, there are a lot of different fields, even with only one message received. We are looking for a specific result. All the results have the 3 keywords 'update_id', 'message' and 'chat'. Look for the result which has the text that you sent in the "text" field inside the "chat" struct.
7. In the concerning "chat" struct, look for the key "id". This id is the chat id, with which the bot can directly send you messages, without you requesting anything in advance. Write that one down as well.
