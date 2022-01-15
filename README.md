# React Web SDK
The Unum ID React Web SDK is a library for adding Unum ID functionality and UI to a React application. It helps you create PresentationRequests and share them with an Unum ID-powered mobile app.

## Installation
The React Web SDK is available via NPM/Yarn or Github

Using npm:
```
npm install @unumid/web-sdk-react
```

Using yarn:
```
yarn add @unumid/web-sdk-react
```

From Github:
```
npm install @unumid/web-sdk-react@https://github.com/UnumID/web-sdk-react.git
```

```
yarn add @unumid/web-sdk-react@https://github.com/UnumID/web-sdk-react.git
```

## Functionality
### Creating PresentationRequests
By default, the React Web SDK will create a PresentationRequest as soon as it is rendered, and will periodically regenerate the PresentationRequest (to ensure that it does not expire) until the user shares data or declines the request, or the widget is unmounted. You can use different combinations of props (see below) to choose how much control over PresentationRequest creation you want to have. For example, instead of automatically creating a PresentationRequest on load, you may want to trigger its creation based on a user interaction like a button click.

### Displaying deeplinks
PresentationRequests are shared to an Unum ID-powered mobile app a user's device via deep links. The React Web SDK determines how it displays them based on the browser's userAgent.

#### Button
On mobile browsers, the React Web SDK will default to displaying the deep link as a button, which can be tapped to open the deeplink in the mobile app.

#### QR Code
On non-mobile browsers, the React Web SDK will default to displaying the deep link as a QR code that users can scan with their mobile device.


#### Fallback options
In some situations, neither a QR code nor a button is convenient. The React Web SDK offers the following fallback options for sending the deep link to the user's device.

**push notification** Sends a push notification to the user's device. The user must have push notifications enabled for an Unum ID-powered mobile app in order to use this option. To use the default push notification service provided by Unum ID, you will need to upload the push notification credentials (Firebase Cloud Messaging and/or Apple Push Notification Service) for your app.

**sms**: Sends the user an sms containing the deeplink, which they can open on their device. You must provide the user's mobile phone number in order to use this option.

**email**: Sends the user an email containing the deeplink, which they can open on their device. You must provide The user's email address in order to use this option.

**login**: If your client application doesn't have the information required to use the desired fallback options, we can redirect them to your existing login page. You must provide the `goToLogin` prop in order to use this option.

**custom notifications** The React Web SDK sends default push, sms, and email notifications using the Unum ID SaaS. If you want to customize your notifications, you may provide the `sendPushNotification`, `sendEmail` and/or `sendSms` props. If these props are provided, they will be used instead of the default.

## API
### UnumIDWidget Component
The React Web SDK default exports a single component, `UnumIDWidget`
This component encapsulates all of the React Web SDK's functionality.


#### Props
**env** (string) _optional_: The environment to run in (`'sandbox'` or `'production'`). This determines which Unum ID SaaS environment the React Web SDK will connect to. While this prop is technically optional, it is required for most use cases and should always be provided.

**apiKey** (string) _optional_: Your Web SDK api key (obtained from Unum ID). While this prop is technically optional, it is required for most use cases and should always be provided.

**userInfo** (`UserInfo`) _optional_: Information about a logged in user of your application. The React Web SDK will use this to determine which fallback options are available.

**presentationRequest** (`PresentationRequestPostDto`) _optional_: a `PresentationRequestPostDto` object, created on your server via the [Server SDK](https://github.com/UnumID/Server-SDK-TypeScript). You may provide this prop in combination with setting `createInitialPresentationRequest` (below) to `false` for more control over when the widget should display a PresentationRequest to the user.

**createInitialPresentationRequest** (boolean) _optional_: Whether the widget should immediately call `createPresentationRequest` on load. You can combine this with the `presentationRequest` prop to gain control over when the initial PresentationRequest is created. By default, it is `false` if you provide the `presentationRequest` prop and `true` if you do not.

**createPresentationRequest** (`() => Promise<PresentationRequestPostDto> | void`) _optional_: A function which should call your Server SDK powered backend to create a PresentationRequest. If it returns a value, it is assumed that that value is a `PresentationRequestPostDto`. If it does not return a value, you must provide the response via the `presentationRequest` prop in order for the widget to display the PresentationRequest. (As in a redux application, where `createPresentationRequest` will probably be an async action creator of some sort.) The widget will call this function on an interval in order to ensure that it never displays an expired PresentationRequest. This prop is technically optional, but should always be provided unless you know that you will only ever be displaying previously existing PresentationRequests.

**sendEmail** (`(options: ExternalMessageInput) => Promise<SuccessResponse>`) _optional_: A function which takes an `EmailOptions` object and calls your backend to send a deeplink via email. You may use the `sendEmail` function from the Server SDK to send the email, or your own email provider.

**sendSms** (`(options: ExternalMessageInput) => Promise<SuccessResponse>`) _optional_: A function which takes an `SmsOptions` object and calls your backend to send a deeplink via sms. You may use the `sendSMS` function from the Server SDK to send the SMS, or your own SMS provider.

**sendPushNotification** (`options: PushNotificationOptions) => Promise<SuccessResponse>`) _optional_: A function which takes a `PushNotificationOptions` object and calls your backend to send a deeplink via push notification.

**goToLogin** (`() => void`) _optional_: A function which redirects the user to your existing login page. You should provide this if you are using Unum ID as an additional authentication factor on top of your existing login. If this prop is not provided, the login fallback option will not be available.

**userCode** (`string`) _optional_: A code which uniquely identifies a user in your system. If provided, it will allow the holder to call back to your api and enable functionality including just-in-time credential issuance.



## Examples
The simplest typical use case. It allows the SDK to handle all PresentationRequest creation, and does not provide any additional fallback options.
```jsx
import UnumIDWidget from '@unumid/web-sdk-react';

const App = () => {
  const createPresentationRequest = async () => {
    // Call your backend to create a PresentationRequest and return the response.
  };

  return (
    <UnumIDWidget
    env='sandbox'
    apiKey='my_sandbox_api_key'
    createPresentationRequest={createPresentationRequest}
  />
  );
};
```

The simplest typical use case (above), using TypeScript.
```tsx
import { FC } from 'react';

import UnumIDWidget, { PresentationRequestPostDto } from '@unumid/web-sdk-react';

const App: FC = () => {
  const createPresentationRequest = async (): Promise<PresentationRequestPostDto> => {
    // Call your backend to create a PresentationRequest and return the response.
  };

  return (
    <UnumIDWidget
      env='sandbox'
      apiKey='my_sandbox_api_key'
      createPresentationRequest={createPresentationRequest}
    />
  );
};
```

A slightly more complex use case which allows the SDK to handle PresentationRequest creation, but enables fallback options by providing user info,
and just-in-time issuance by providing a user code
```jsx
import UnumIDWidget from '@unumid/web-sdk-react';

const App = () => {
  const createPresentationRequest = async () => {
    // Call your backend to create a PresentationRequest and return the response.
  };

  return (
    <UnumIDWidget
      env='sandbox'
      apiKey='my_sandbox_api_key'
      userInfo={{
        email: 'mrplow@gmail.com', // The user's email is required to enable the email fallback.
        phone: 'KL5-5555', // The user's mobile phone number is required to enable the sms fallback.
        pushToken: {
          provider: 'FCM', // this is a token for Firebase Cloud Messaging (FCM)
          value: 'my firebase cloud messaging token' // FCM token from the user's device
        }
      }}
      createPresentationRequest={createPresentationRequest}
      userCode='my-user-code'
    />
  );
};
```

Allows your application more control over when the initial PresentationRequest is created. Enables custom email, sms, and push notification fallbacks.
```jsx
import { useState } from 'react';

import UnumIDWidget from '@unumid/web-sdk-react';

const App = () => {
  // Save the PresentationRequest in local component state.
  const [presentationRequest, setPresentationRequest] = useState();

  const createPresentationRequest = async () => {
    const options = {
      // Customizable PresentationRequest options.
      credentialRequests: [{
        type: 'LoginCredential',
        issuers: ['did:unum:5235d82e-5aac-4df4-adf2-7c6cc0cbec95']
      }],
      verifier: 'did:unum:a74fce7c-7dfa-4702-b85f-f68a854c3cfe'
    };
    // Call your backend to create a PresentationRequest and save in the component state.
    const response = await callBackend(options);
    setPresentationRequest(response);
  };

  const sendEmail = async (options) => {
    // Call your backend to send a deeplink via custom email and return the response.
  };

  const sendSms = async (options) => {
    // Call your backend to send a deeplink via custom sms and return the response.
  };

  const sendPushNotification = async (options) => {
    // Call your backend to send a deeplink via custom push notification and return the response.
  }

  const goToLogin = () => {
    // Navigate to your login page.
  };

  return (
    <UnumIDWidget
      env='sandbox'
      apiKey='my_sandbox_api_key'
      userInfo={{
        email: 'mrplow@gmail.com', // The user's email is required to enable the email fallback.
        phone: 'KL5-5555', // The user's mobile phone number is required to enable the sms fallback.
        pushToken: {
          provider: 'FCM', // this is a token for Firebase Cloud Messaging (FCM)
          value: 'my firebase cloud messaging token' // FCM token from the user's device
        }
      }}
      presentationRequest={presentationRequest} // Provide the React Web SDK with an already-created PresentationRequest.
      createInitialPresentationRequest={false} // Prevent the React Web SDK from immediately creating a new PresentationRequest on load.
      createPresentationRequest={createPresentationRequest} // We still need to provide the React Web SDK with a createPresentationRequest function so that it can create a new PresentationRequest before the current one expires.
      sendEmail={sendEmail} // Will be used instead of the default email fallback
      sendSms={sendSms} // Will be used instead of the default sms fallback
      sendPushNotification={sendPushNotification} // Will be used instead of the default push notification fallback
      goToLogin={goToLogin}

    />
  );
};
```

Applications using Redux and other similar state management libraries have some unique challenges, as side effects such as creating resources usually happen in action creator functions, which dispatch actions to the store rather than returning values. In this example, we're providing the React Web SDK with our `createPresentationRequest` action creator to call, then selecting the created PresentationRequest from the store to provide separately.
```jsx
import { useSelector } from 'react-redux';

import UnumIDWidget from '@unumid/web-sdk-react';

// Import your action creators. They have been wrapped in React hooks in this example, but your application may be different.
import { useActionCreators } from './hooks/actionCreators';

const App = () => {
  // These functions can be defined as async action creators using redux-thunk, redux-saga, or other libraries.
  const { createPresentationRequest } = useActionCreators();

  // Select a previously created PresentationRequest from state.
  const presentationRequest = useSelector(state => state.presentationRequest);

  // Select the logged in user from state.
  const loggedInUser = useSelector(state => state.loggedInUser);

  const goToLogin = () => {
    // Navigate to your login page.
  };

  return (
    <UnumIDWidget
      env='sandbox'
      apiKey='my_sandbox_api_key'
      userInfo={{
        email: 'mrplow@gmail.com', // The user's email is required to enable the email fallback.
        phone: 'KL5-5555', // The user's mobile phone number is required to enable the sms fallback.
        pushToken: {
          provider: 'FCM', // this is a token for Firebase Cloud Messaging (FCM)
          value: 'my firebase cloud messaging token' // FCM token from the user's device
        }
      }}
      presentationRequest={presentationRequest} // Provide the React Web SDK with an already-created PresentationRequest.
      createInitialPresentationRequest={true} // The React Web SDK should immediately create a PresentationRequest on load.
      createPresentationRequest={createPresentationRequest} // We still need to provide the React Web SDK with a createPresentationRequest function so that it can create a new PresentationRequest before the current one expires.
      goToLogin={goToLogin}
    />
  );
};
```

It is possible to use the sdk to simply display an existing PresentationRequest. In this example, a PresentationRequest is being gotten from the backend and displayed using the Web SDK with no fallback options or recreation on expiration. Using the Web SDK in this way will disable much of the functionality described above and is not recommended unless you're very sure this is the only use case you need to support.
```jsx
import { useState, useEffect } from 'react';
import UnumIDWidget from '@unumid/web-sdk-react';

const App = () => {
  const [presentationRequest, setPresentationRequest] = useState();

  useEffect(() => {
    const getPresentationRequest = async () => {
      // call your backend to get a presentationRequest using the server sdk
      const gottenRequest = await callBackend();
      setPresentationRequest(gottenRequest);
    }
  }, []);

  return <UnumIDWidget presentationRequest={presentationRequest}>;
}
```

## Minimum Requirements
The minimum supported version of React is v16.8.0. If you are using an older version of React, you will need to upgrade it to at least v16.8.0 in order to use the React Web SDK.

## TypeScript support
The React Web SDK is written in TypeScript and exports relevant types. Some types are also pulled from our shared types library, [`@unumid/types`](https://github.com/UnumID/types). We recommend adding `@unumid/types` as a dependency to ensure full type support between the React Web SDK and [Server SDK](https://github.com/UnumID/Server-SDK-TypeScript).

**Client reference application** is available at Git and can be cloned using 
`git clone https://github.com/UnumID/Verifier-Client-SDK-Client-Reference-App.git`. 
