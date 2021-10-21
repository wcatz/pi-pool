# Grafana Telegram Alerts

## Prerequisites&#x20;

Prometheus data source working in Grafana & a Telegram account. To get alerts it is best to have Grafana on a vps or in a different location. This way if your internet goes down you will still be alerted. What I do is connect Grafana to a Prometheus endpoint through a Wireguard tunnel. I have a local Grafana and a remote Grafana using the same data source.

### Create a Telegram bot

Go to [https://t.me/botfather](https://t.me/botfather) in Telegram and create a new bot and give it a name and username, then copy your API access token.

> /newbot
>
> Graffana\_Alerts
>
> my\__pool\__bot

Something like that. Get creative.

Follow the t.me link to your bot. Start it and click on the three dots and 'get info'. Copy your bots username

{% hint style="info" %}
Return to Bot Father if you need a link to your bot or if you need the api token.

[https://t.me/botfather](https://t.me/botfather)
{% endhint %}

### Create a Telegram group

In Telegram you can create a group by clicking on the :pencil:symbol next to the search bar.

### Add your bot to the new group

Edit your new group, click on add members. Enter your bots name in the search bar and add your bot to the group.

### Find your bots group user id

Paste your bots API token into the url below. If you only get status ok you may have to remove and add your bot back in again. Otherwise use the image below to determine your Chat ID.

https://api.telegram.org/bot\<YOUR BOT API TOKEN>/getUpdates

![ID will start with a - sign like above](<.gitbook/assets/Screen Shot 2021-10-21 at 11.06.02 AM.png>)

## Create a notification channel in Grafana

Click the bell icon in Grafanas left hand verticle menu and choose notification channels. Choose 'Add channel'. Give it a name and choose Telegram under type. Enter the token and the chat id value you copied earlier. Click test and you should get a green 'test notification sent' and a message in your new group. If so go ahead and click save.

###
