---
title: "HackThe6ix 2022 🥇"
date: 2022-08-23T12:45:15-05:00
draft: false
tags: ["2022", "technical"]
---


Over the last weekend, I attended [HackThe6ix 22'](https://hackthe6ix.com/) . It was a hybrid event, mostly held online with a couple of events happening at the Wealthsimple Offices in Toronto. I was able to snag a ticket to the in-person events, which unfortunately also meant traveling ***3 hours*** on the GO Bus from Waterloo to Toronto 😔. Luckily, the Wealthsimple Office was a great place to meet my team, attend workshops and get some free food!

![pic1.png](../ht6/pic1.png#center)

Our team tackled Wealthsimple's challenge to create a financial product that empowered a marginalized group. The group we decided to focus on were patients struggling with Alzheimer's and Dementia. We found [several news articles](https://www.cbc.ca/news/business/senior-alzheimers-upsold-bell-products-source-1.6014904) about how such individuals often make unnecessary purchases since they aren't always fully aware of where they are. This can also potentially leave them in a vulnerable position, letting people take advantage of them. 

 And thus, **HeimWallet** was born! Our app allows a trusted guardian set an allowance that the patient can spend in a day. Any transaction that exceeds this allowance triggers a notification on the guardian (using Twillio) to either approve or deny the transaction, ensuring that no unnecessary spending happens. This allows the guardian to protect the patient's financial assets, while still giving the Alzheimer's patient some autonomy to spend their money.
 
  The frontend of the application was made using React Native, with the designs being made using Figma. Since the app was targetting older patients, we had to ensure that the interface was clean, easy to understand, and accessible. The guts of the project run on a Python (Flask) server, hosted using Heroku. We also integrated Twillio (for the SMS notifications) and Google Maps API (to identify the location of the purchase).

![pic2.png](../ht6/pic2.png#center)

I'm happy to say that after 36 grueling hours, our team ended up coming **1st Place** in the Hackathon out of over 70 submissions! 🎉 In addition to that, we also won the Wealthsimple Challenge we were originally inspired by! If you want to see more of the project, you can check out the [Devpost link here](https://devpost.com/software/heimwallet), or the [video demo link here](https://www.youtube.com/watch?v=6q0U1Z_pYIY).

![HeimWallet.png](../ht6/HeimWallet.png#center)
