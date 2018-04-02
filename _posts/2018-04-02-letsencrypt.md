---
layout: post
posted: 2018-04-02
title: Let's Encryptのワイルドカード証明書を試す
description: Ubuntu16.0.4 Nginx環境にLet's Encryptのワイルドカード証明書を入れてみた
---

3月13日にLet's Encryptがワイルドカードをサポートするようになっていた[[参考]](https://letsencrypt.org/2017/07/06/wildcard-certificates-coming-jan-2018.html)ので試してみた。

とりあえずcertbotを更新。ワイドカードはACME v2を使っているのでcertbotの場合は0.22以上にしないといけないらしい。ACMEはよく分かってないので今度調べる。

```
sudo apt upgrade certbot
sudo certbot --version
> certbot 0.22.2
```


ワイルドカード証明書を生成するにはDNS所有権を確認する必要があり、DNSの認証が必要になります。AWSのroute53とかGoogle Cloud DNS, Cloudflareとか使ってる環境ならcertbotのDNS Pluginを使ってできるのかな。
今回の環境は別の所でドメイン管理してるのでマニュアルで対応します。
ドメイン発行のコマンドは以下。

```
sudo certbot certonly \
--manual \
--preferred-challenges dns \
--server https://acme-v02.api.letsencrypt.org/directory \
--domain *.shmrkm.com
```

`--prefereed-challenges` はdns-01じゃなくていいのかなと思ったけど、certbotのドキュメントに`dns`と書いてあったのでそれで。
このコマンドを実行すると以下のようにTXTレコードを設定するように言われます。

```
-------------------------------------------------------------------------------
Please deploy a DNS TXT record under the name
_acme-challenge.shmrkm.com with the following value:

<TXT RECORD VALUE>

Before continuing, verify the record is deployed.
-------------------------------------------------------------------------------
```

`TXT RECORD VALUE`に表示された値を `_acme-challenge.ドメイン名` のTXTレコードに設定すればOKですが、コマンド実行ごとに毎回値が変わるので毎回設定しなおす必要があります。

digでTXTレコードが更新されたことを確認し

```
$ dig TXT _acme-challenge.shmrkm.com
> <TXT RECORD VALUE>
```

certbotのコマンドを進めると認証されて証明書が生成されます。
これをnginxに適用すると無事にワイルドカード証明書が使えました。

再生成するにはこうしたら良いって言ってくれてますね。

> To obtain a new or tweaked
> version of this certificate in the future, simply run certbot
> again. To non-interactively renew *all* of your certificates, run
> "certbot renew"

最後に、支援してくれるならここから寄付してくれ！みたいなこと言われますが

Let's Encryptにかぎらず支援したいものがあっても日本からだと寄付できなくて少しもにょりますね。


# 参考

[https://certbot.eff.org/docs/using.html?highlight=domain](https://certbot.eff.org/docs/using.html?highlight=domain)

[https://laboradian.com/use-wildcard-with-letsencrypt/](https://laboradian.com/use-wildcard-with-letsencrypt/)


