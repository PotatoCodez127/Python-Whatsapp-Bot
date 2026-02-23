# Build AI WhatsApp Bots with Pure Python & Google Gemini

This guide will walk you through the process of creating a WhatsApp bot using the Meta (formerly Facebook) Cloud API with pure Python and Flask. We'll also integrate webhook events to receive messages in real-time and use Google's Gemini API to generate intelligent responses. 

For more information on the structure of the Flask application, you can refer to the `app/` directory documentation.

## Prerequisites

1. A Meta developer account — If you don’t have one, you can create a Meta developer account here: https://developers.facebook.com/
2. A business app — If you don't have one, you can learn to create a business app here: https://developers.facebook.com/docs/development/create-an-app/. If you don't see an option to create a business app, select Other > Next > Business.
3. A Google AI Studio Account — To get a Google API Key for the Gemini model.
4. Familiarity with Python to follow the tutorial.

## Table of Contents

- Get Started
- Step 1: Select Phone Numbers
- Step 2: Send Messages with the API
- Step 3: Configure Webhooks to Receive Messages
- Step 4: Understanding Webhook Security
- Step 5: Integrate AI (Google Gemini) into the Application
- Step 6: Add a Production Phone Number

---

## Get Started

1. Overview & Setup: Begin your journey here: https://developers.facebook.com/docs/whatsapp/cloud-api/get-started
2. Locate Your Bots: Your bots can be found here: https://developers.facebook.com/apps/
3. WhatsApp API Documentation: Familiarize yourself with the official documentation: https://developers.facebook.com/docs/whatsapp

## Step 1: Select Phone Numbers

- Make sure WhatsApp is added to your App.
- You begin with a test number that you can use to send messages to up to 5 numbers.
- Go to API Setup and locate the test number from which you will be sending messages.
- Here, you can also add numbers to send messages to. Enter your own WhatsApp number.
- You will receive a code on your phone via WhatsApp to verify your number.

## Step 2: Send Messages with the API

1. Obtain a 24-hour access token from the API access section.
2. Create a `.env` file based on `example.env` and update the required variables. 

Creating an access token that works longer than 24 hours:
1. Create a system user at the Meta Business account level: https://business.facebook.com/settings/system-users
2. On the System Users page, configure the assets for your System User, assigning your WhatsApp app with full control. Don't forget to click the Save Changes button.
3. Now click "Generate new token" and select the app, and then choose how long the access token will be valid. You can choose 60 days or never expire.
4. Select all the permissions.
5. Confirm and copy the access token.

Update the following information in your `.env` file from the App Dashboard:

- APP_ID: <YOUR-WHATSAPP-BUSINESS-APP_ID>
- APP_SECRET: <YOUR-WHATSAPP-BUSINESS-APP_SECRET>
- RECIPIENT_WAID: <YOUR-RECIPIENT-TEST-PHONE-NUMBER> 
- VERSION: v18.0 (Or the latest version of the Meta Graph API)
- ACCESS_TOKEN: <YOUR-SYSTEM-USER-ACCESS-TOKEN> 

## Step 3: Configure Webhooks to Receive Messages

#### Start your app
- Ensure you have a python installation or virtual environment and install the requirements: 
  pip install -r requirements.txt

- Run your Flask app locally by executing run.py:
  python run.py


#### Launch ngrok

You need a static ngrok domain because Meta validates your ngrok domain and certificate.

1. Sign up for ngrok (https://ngrok.com/) for free and download the agent.
2. Authenticate your ngrok agent using your Authtoken.
3. On the ngrok dashboard, navigate to Cloud Edge > Domains and create a free static domain.
4. Start ngrok by running the following command in a terminal:
  ngrok http 8000 --domain your-domain.ngrok-free.app

5. ngrok will display a URL where your localhost application is exposed to the internet.

#### Integrate WhatsApp

In the Meta App Dashboard, go to WhatsApp > Configuration, then click the Edit button.
1. In the Callback URL field, enter the URL provided by ngrok with `/webhook` at the end (e.g., `https://your-domain.ngrok-free.app/webhook`).
2. Enter a verification token. This string is set up by you. Make sure to update this in your `.env` file as `VERIFY_TOKEN`.
3. Confirm your localhost app receives the validation GET request and logs `WEBHOOK_VERIFIED` in the terminal.
4. Back on the Configuration page, click Manage next to Webhook fields.
5. Subscribe to the "messages" field.

## Step 4: Understanding Webhook Security

The application includes built-in webhook security in `app/decorators/security.py`. 

WhatsApp signs all Event Notification payloads with a SHA256 signature included in the request's `X-Hub-Signature-256` header. The decorator generates a SHA256 signature using the payload and your app's App Secret, and compares it to verify the payload is genuinely from Meta.

## Step 5: Integrate AI (Google Gemini) into the Application

This repository is pre-configured to work with Google's Gemini models rather than standard hardcoded responses.

1. Get your API Key from Google AI Studio (https://aistudio.google.com/).
2. Add your key to the `.env` file: `GOOGLE_API_KEY=your_api_key_here`
3. The core AI logic is located in `app/services/gemini_service.py`. 
4. The message processing pipeline in `app/utils/whatsapp_utils.py` calls the Gemini service to generate dynamic, conversational responses based on the user's input.
5. Conversation histories are logged locally via SQLAlchemy (e.g., `gemini_threads_db.db` and `processed_messages_db.db`).

## Step 6: Add a Production Phone Number

When you’re ready to use your app for a production use case, you need to use your own phone number to send messages to your users.

To start sending messages to any WhatsApp number, you must add a phone number in the Meta Business settings. If you want to use a number that is already being used in the WhatsApp customer or business app, you will have to fully migrate that number to the business platform. 

**Recommendation**: For prolonged or professional purposes, using a virtual phone number service or purchasing a new SIM card for a dedicated device is advisable to separate your personal WhatsApp from the Business API.
