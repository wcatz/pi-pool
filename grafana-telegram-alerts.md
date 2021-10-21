# Grafana Telegram Alerts

## Prerequisites&#x20;

Prometheus data source working in Grafana & a Telegram account. To get alerts it is best to have Grafana on a Pi in another location or on a vps. This way if your internet goes down you will still be alerted.&#x20;

What I do is connect Grafana to a Prometheus endpoint through a Wireguard VPN on my core. I have a local Grafana and a remote Grafana using the same data source. One dashboard on the remote Grafana has alerts that use Telegram.

### Create a Telegram bot

Go to [https://t.me/botfather](https://t.me/botfather) in Telegram and create a new bot and give it a name and username, then copy your API access token.

> /newbot
>
> Grafana\_Alerts
>
> my\__pool\__bot

Something like that. Get creative.

### Create a Telegram group

In Telegram you can create a group by clicking on the :pencil:symbol next to the search bar. You can go back and add an image and change the name later.

### Add your bot to the new group

{% hint style="info" %}
Return to Bot Father if you need a link to your bot or if you need the api token.

[https://t.me/botfather](https://t.me/botfather)
{% endhint %}

Open botfather Follow the t.me link to your bot. Start it and click on the three dots and 'get info'. Copy your bots username.

Edit your new group, click on add members. Enter your bots name in the search bar and add your bot to the group.

### Find your groups Chat ID

Paste your bots API token into the url below. If you only get status ok you may have to remove and add your bot back in again. Otherwise use the image below to determine your Chat ID. Starts with -

https://api.telegram.org/bot\<YOUR BOT API TOKEN>/getUpdates

![ID will start with a - sign like above](<.gitbook/assets/Screen Shot 2021-10-21 at 11.06.02 AM.png>)

## Create a notification channel in Grafana

Click the bell icon in Grafanas left hand vertical menu and choose notification channels. Choose 'Add channel'. Give it a name and choose Telegram under type. Enter the token and the chat id value you copied earlier. Click test and you should get a green 'test notification sent' and a message in your new group. If so go ahead and click save.

## Add a dashboard for monitoring

Grafana Alerts are only available for 'time series' & 'graph (old)' type panels, not always how we would look at the data. Let's create a new dashboard with graph type panels we need for alerts.
