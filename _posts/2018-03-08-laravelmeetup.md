---
layout: post
posted: 2018-03-08
title: Laravel Meetup Vol 10
description: Laravel Meetup Vol.10に参加してLTしてきました
---

とりあえずブログ環境を作ったので書く習慣をつくるために…

[Laravel Meetup Vol.10](https://laravel-meetup-tokyo.connpass.com/event/77823) に参加してLTしてきました。

LTの資料はこちら。

<script async class="speakerdeck-embed" data-id="3c5c6d5187cd438ebe992f6ae12e2b5e" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

時間の都合でだいぶはしょってしゃべりましたけど、言いたかったことは

- SQLiteをテストのDBとして使ってチームメンバーと共有した
- テストからドキュメントを自動生成してる

ってことです。
ドキュメントを自動生成するためにこのライブラリを作りました。
[https://github.com/shmurakami/api-doc-generator](https://github.com/shmurakami/api-doc-generator)

今のところSwaggerのYaml, apiDocのDoc commentに対応しています。

