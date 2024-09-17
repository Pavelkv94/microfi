
``# Microfrontend Communication Library

This library provides two React hooks, `useHostBus` and `useMicrofrontendBus`, to facilitate communication between a host application and microfrontend applications using iframes.

## Overview

- **`useHostBus`**: Used in the host application to manage communication with registered iframes.
- **`useMicrofrontendBus`**: Used in microfrontend applications to communicate with the host.

## Installation

To install the library, add it to your project via npm or yarn:

```
npm install @pavelkv94/microfi
# or
yarn add @pavelkv94/microfi`` 
```

## `useHostBus`

### Description

`useHostBus` is a React hook that helps the host application register iframes and send actions to them.

### Usage

1.  **Initialize the Hook**
    
```
    import { useHostBus } from "@pavelkv94/microfi";
    
    const originsWhiteList = ["http://localhost:5001", "http://localhost:5002"];
    const { sendToRemote, registerIframe } = useHostBus(originsWhiteList);` 
   ```
2.  **Register Iframes**
    
    In the host app, you can register iframes and handle incoming messages using the `registerIframe` method.
    
```
    `useEffect(() => {
      const messageHandler = (event: MessageEvent) => {
        if (originsWhiteList.includes(event.origin)) {
          // Handle incoming messages from iframes
          try {
            const action: Action = JSON.parse(event.data);
    
            switch (action.type) {
              case "IFRAME-LOADED":
                registerIframe(event); // Register the iframe
                break;
              case "STATE-ACTION":
                sendToRemote(action); // Send the action to all registered iframes
                break;
              default:
                console.warn("Unhandled action type:", action.type);
            }
          } catch (error) {
            console.error("Failed to handle message:", error);
          }
        } else {
          console.warn("Blocked message from untrusted origin:", event.origin);
        }
      };
    
      window.addEventListener("message", messageHandler);
    
      return () => {
        window.removeEventListener("message", messageHandler);
      };
    }, [registerIframe, sendToRemote]);` 
   ```
3.  **Send Actions to Iframes**
    
    You can use the `sendToRemote` method to send actions to all registered iframes.
```
    
    `const sendToken = () => {
      const action: Action = {
        type: "TOKEN_CREATED",
        payload: { token: "XXX-YYYY" },
      };
    
      sendToRemote(action);
    };` 
 ```  

### Example

```

`import React, { useEffect, useState } from "react";
import { useHostBus } from "@pavelkv94/microfi";

function App() {
  const [menu, setMenu] = useState<string[]>([]);
  const originsWhiteList = ["http://localhost:5001", "http://localhost:5002"];
  const { sendToRemote, registerIframe } = useHostBus(originsWhiteList);

  useEffect(() => {
    const messageHandler = (event: MessageEvent) => {
      if (originsWhiteList.includes(event.origin)) {
        try {
          const action: Action = JSON.parse(event.data);

          switch (action.type) {
            case "MENU_SENT":
              setMenu((prev) => [...prev, ...action.payload.menu]);
              break;
            case "IFRAME-LOADED":
              registerIframe(event); // Register the iframe
              break;
            case "REDUX-ACTION":
              sendToRemote(action); // Send the action to all registered iframes
              break;
            default:
              console.warn("Unhandled action type:", action.type);
          }
        } catch (error) {
          console.error("Failed to handle message:", error);
        }
      } else {
        console.warn("Blocked message from untrusted origin:", event.origin);
      }
    };

    window.addEventListener("message", messageHandler);

    return () => {
      window.removeEventListener("message", messageHandler);
    };
  }, [registerIframe, sendToRemote]);

  return (
    <div className="App">
      <h1>HOST APP</h1>
      <ul>
        {menu.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
      <button onClick={() => sendToken()}>Send Token to all microfronts</button>
      <iframe src="http://localhost:5001/" style={{ width: "500px", height: "500px", border: "none", marginRight: "10px" }}></iframe>
      <iframe src="http://localhost:5002/" style={{ width: "500px", height: "500px", border: "none" }}></iframe>
    </div>
  );
}

export default App;` 
```
## `useMicrofrontendBus`

### Description

`useMicrofrontendBus` is a React hook used in microfrontend applications to communicate with the host.

### Usage

1.  **Initialize the Hook**
```
    
    `import { useMicrofrontendBus } from "@pavelkv94/microfi";
    
    const { registerMeInHost, sendToHost, subscribe } = useMicrofrontendBus("http://localhost:5000");` 
   ``` 
2.  **Register the Iframe**
    
    Call `registerMeInHost` to notify the host that the iframe is loaded.
    
```
    
    `useEffect(() => {
      registerMeInHost();
    }, [registerMeInHost]);` 
 ```  
3.  **Subscribe to Actions**
    
    Use the `subscribe` method to listen for specific actions from the host.
    
```
    
    `useEffect(() => {
      const unsubscribe = subscribe("TOKEN_CREATED", (action: Action) => {
        console.log("Received token:", action.payload.token);
      });
    
      return () => {
        unsubscribe(); // Clean up subscription
      };
    }, [subscribe]);` 
   ```
4.  **Send Actions to the Host**
    
    Use the `sendToHost` method to send actions to the host application.
    
```
    
    `const notifyHost = () => {
      const action: Action = {
        type: "CUSTOM_ACTION",
        payload: { data: "some data" },
      };
    
      sendToHost(action);
    };` 
```
### Example

```
`import React, { useEffect, useState } from "react";
import { useMicrofrontendBus } from "@pavelkv94/microfi";

function App() {
  const [token, setToken] = useState<string | undefined>(undefined);
  const { registerMeInHost, subscribe, sendToHost } = useMicrofrontendBus("http://localhost:5000");

  useEffect(() => {
    registerMeInHost();

    const unsubscribe = subscribe("TOKEN_CREATED", (action: Action) => {
      setToken(action.payload.token);
    });

    return () => {
      unsubscribe(); // Clean up subscription
    };
  }, [registerMeInHost, subscribe]);

  return (
    <>
      <h1>REMOTE APP</h1>
      <p>Token is {token}</p>
      <button onClick={() => sendToHost({ type: "REQUEST_TOKEN" })}>
        Request Token from Host
      </button>
    </>
  );
}

export default App;` 
```
## Summary

-   Use `useHostBus` in the host application to manage and communicate with iframes.
-   Use `useMicrofrontendBus` in microfrontend applications to communicate with the host.

Feel free to adjust the types and implementations based on your specific use case and payload structure.