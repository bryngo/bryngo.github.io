---
layout: post
title: Anime Watchlist
excerpt: Watch me become a weeb.
categories: [anime]
comments: true
image:
  feature: https://images2.alphacoders.com/742/742320.png
  credit: Unknown
  creditlink: https://wall.alphacoders.com/big.php?i=742320
---

## Currently Watching
#### Made in Abyss

## Wanting to Watch


## Already Watched
#### 3 - Gatstu No Lion / 3 - Gatsu No Lion 2nd Season
#### Gabriel Dropout
#### Assassination Classroom + Assassination Classroom 2nd Season
#### Steins Gate / Steins Gate 0
#### Yuru Camp
#### Kono Subarashii Sekai Ni Shukufuku Wo! / Kono Subarashii Sekai Ni Shukufuku Wo! 2
#### When the Promised Flower Blooms
#### Your Lie in April
#### SukaSuka
#### Toradora!
#### Kokoro Connect
#### 5 Centimeters per Second
#### Kotonoha No Niwa
#### Madoka Magica
#### Seishun Buta Yarou
#### Goblin Slayer
#### Iroduku: The World in Colors
#### Oregairu / Oregairu 2nd Season


{% raw %}
~~~html
{% for career in site.data.index.careers %}
      <div class="archive-title">
          <div class="archive-year">
              <strong>{{ career.date }}</strong><br> {{ career.job }} @ <a href="{{ career.link }}">{{ career.name }}</a>
          </div>
      </div>
{% endfor %}
 ~~~
{% endraw %}
