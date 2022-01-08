---
title: "React Zendesk, and components that lie"
date: 2022-01-07T22:34:48-05:00
draft: false
tags: ["2022", "technical"]
---

Zendesk is a very popular cloud-based customer service platform that lets you implement chatbots and CRM systems easily - but you probably recognize them from these buttons at the bottom-right of many websites:

![Zendesk Button](/react-zendesk/button.png#center)

If you’ve ever wanted to implement this on a React app, you’ve probably encountered [React Zendesk](https://www.npmjs.com/package/react-zendesk), which attempts to convert Zendesk's widget into a React Component. It has over 10k weekly downloads and a bunch of references online, but it is far from official.

### The Widget API

The component attempts to implement the [Zendesk Widget](https://support.zendesk.com/hc/en-us/articles/4408836216218-Using-Web-Widget-Classic-to-embed-customer-service-in-your-website), which essentially boils down to a script that appends the button to the document body (using `document.body.appendChild`). It also has functions using which you can show or hide the button. The script is fairly static - you can’t change the position of the button or place it exactly where you want. It’s always on the bottom right, and always on the top of everything else on this page. 

 This was probably designed as a feature since we generally always want the user to be able to look for help, and the user *probably* expects this button to be at the bottom-right. For the average React developer, though, this is very strange behaviour for a component.

### The React Component, but not really

You can see the actual usage [here]([https://github.com/B3nnyL/react-zendesk](https://github.com/B3nnyL/react-zendesk)), but as you would expect, the library creates a `<Zendesk />` component for you to use. Behind the scenes, though, this component implements exactly what the widget API does - it simply appends the button to the document body. The component itself doesn't control whether the button is shown - that is handled by API calls. This behavior is, therefore, closer to a function call than a traditional DOM element. You can “call” this component multiple times, in different parts of your render function, but it won’t do anything - you’re just calling the button initialization repeatedly.

 If you want the button to always be present on your website, it works well enough. However, if you place it inside a component that re-renders repeatedly, or expect it to hide/show depending on where you place it, it won’t work like you think it should - because again, you’re essentially calling a function that does the same thing every time.

 What you really want to do is treat the component *like* the widget - call it at the top of your project (i.e. `App.tsx`) and use the API function calls when you want to hide or show the component. 

In your top-most file, do:

```jsx
<Zendesk
        zendeskKey={YOUR_KEY_HERE}
        onLoaded={() => {
          ZendeskAPI("webWidget", "hide");
        }}
/>
```

On pages where you want to show the widget, do:

```jsx
useEffect(() => {
    ZendeskAPI("webWidget", "show");

    return () => {
      ZendeskAPI("webWidget", "hide");
    };
  }, []);
```

(See source [here](https://github.com/B3nnyL/react-zendesk/issues/28)) 

And voila! This should work well enough for most cases. 

However, to complicate things, the API hide/show function also has a [race condition](https://www.bennadel.com/blog/3248-the-zendesk-web-widget-appears-to-have-a-small-hide-show-race-condition.htm), since it isn’t really intended to be toggled frequently. My recommendation for that would be to avoid complicated logic that dictates whether the widget needs to be visible. Keep it simple, ideally only hiding the widget on specific pages.

### Conclusion

This is a somewhat dramatic post for what is a very niche topic that very few people would have to deal with. The main reason I’m writing this because it is an interesting case study on how abstraction can lead to misguided expectations and unintended behaviour. In hindsight, if you read the documentation of React-Zendesk, it does seem to hint towards the solution described above. The average developer, though, would install this package and attempt to use it like any other component, by calling it on every page they want the button to show up. This might work initially, but things will start to break if you start to use the Zendesk component like a regular component.

Reference: [https://github.com/B3nnyL/react-zendesk/issues/28](https://github.com/B3nnyL/react-zendesk/issues/28)