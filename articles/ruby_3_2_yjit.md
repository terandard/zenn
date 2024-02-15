---
title: "Ruby3.2 + Rails 7.0 で YJIT 有効化によるパフォーマンスについて"
emoji: "☕"
type: "tech"
topics: ["ruby", "rails"]
published: true
published_at: 2024-02-15 12:00
publication_name: "socialplus"
---
## はじめに
こんにちは、terandard です。  

昨年(2023)のクリスマスに Ruby 3.3.0 のリリースがありました。  
リリースノートには「YJIT の大幅なパフォーマンス改善」とあるので、今年の RubyKaigi でも YJIT についての講演があるのかな？とワクワクしております。    
https://www.ruby-lang.org/ja/news/2023/12/25/ruby-3-3-0-released/
https://k0kubun.hatenablog.com/entry/ruby-3-3-yjit

弊社のサービスは昨年まで Ruby 3.1 だったため、年始に Ruby 3.2 にアップデートしました。  
YJIT も有効化したので、YJIT 有効化前後のパフォーマンス比較について共有します。


## YJIT 有効化前後のパフォーマンス確認
リリース前後の1週間のパフォーマンスを比較しました。  
グラフに関しては「点線: リリース前」「実践: リリース後」となっています。  

### リクエスト数
![](/images/ruby_3_2_yjit/request.png)
Avg 2.34k → 2.36k  
時間ごとのリクエスト数の差はほとんどありませんでした。  
従って以降の各指標は YJIT 有効化前後でほぼ同じ条件での比較となります。

### Rack Request P90 Latency
![](/images/ruby_3_2_yjit/rack_request_p90_latency.png)
👍 Avg 255ms → 220ms (**-13.7%**)  
リクエスト全体で見ると **13.7%** 高速化していることが確認できました。  
ただし一部の時間帯で不自然にレスポンスタイムが速くなっていました。(Wed 17 あたり)  
調査したところ 404 Not Found を返しているリクエストが大量にあったため、その時間帯で平均するとレスポンスが速くなっていました。  
参考までに Wed 17 以前のデータのみで比較をしたところ、 Avg 268ms → 234ms (**-12.7%**) だったため、約 **13%** 程度の改善が確認できました。

### ActiveRecord Instantiation P90 Latency
![](/images/ruby_3_2_yjit/active_record_instantiation_p90_latency.png)
👍 Avg 215μm → 171μm (**-20.4%**)  
インスタンス生成にかかる時間は改善度合いが大きいようです。  

### Action Controller P90 Latency
![](/images/ruby_3_2_yjit/action_controller_p90_latency.png)
👍 Avg 293ms → 280ms (**-4.4%**)  
コントローラ全体で見るとあまり改善が見られませんでした。  

弊社のサービスではソーシャルログインを扱っているため、認証プロバイダに対して HTTP リクエストを送信しています。  
コントローラの処理における外部リクエストの割合が多く(約50%)、Ruby の割合が少ない(約20%)ため、影響が少なかったようです。
![](/images/ruby_3_2_yjit/time_spent_in_controller.png)

### CPU 使用率
![](/images/ruby_3_2_yjit/cpu_utilization.png)
👍 Avg 3.44% → 3.04% (**-0.4%**)  
CPU 使用率は若干低下していました。  

### メモリ使用量
![](/images/ruby_3_2_yjit/memory_usage.png)
👎 Avg 306MiB → 385MiB (**+25.8%**)  
メモリ使用量は明らかに上がっていました。  
今回はサーバー上の問題は無かったのでパラメータのチューニングは行っていませんが、実行環境によっては注意が必要そうです。  
https://k0kubun.hatenablog.com/entry/tuning-yjit

## まとめ
Ruby 3.2 + Rails 7.0 の環境で YJIT を有効にし、パフォーマンスについて調査しました。  
リクエスト全体で約 **13%** の改善が確認できましたが、メモリ使用量は約 **25%** 増加しました。  
Ruby 3.3 では YJIT のパフォーマンスが更に改善されるので、Ruby 3.3 にアップデートしたらまた報告したいと思います。  
