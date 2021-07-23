# sidecar container で IP address whitelisting を実現する試行錯誤

GKE Ingress、NGINX Ingress いずれも src ipaddress の制限機能はあるがあまり多くのルールを書くのに適した仕組みではない。ある程度アプリケーション側で管理する前提で複雑なルールにも対応するにはどうしたらよいかを試行錯誤してみる。

実際には GCLB -> GKE の Pod で使う想定だが、簡易的に docker compose で試す。
いくつか思いついた方法を試してみる。


## Envoy で接続元 src ip address 制限をできないか?

proxy 型 LB を想定するとして、なので src ip address は X-Forwarded-For にあるものとする。

かんたんな方法は無さそう...? できそうと思ったものについて以下所感。

### Lua + http filter

あまり調べてないが、実装が複雑になるし効率も心配。


### ext_authz External Authorization を使う

Envoy から問い合わせる authz server を gRPC か HTTP で用意する。authz server はじぶんでつくる必要がある。処理内容は自由にできるがさすがに大仰である。

参考 https://qiita.com/ryysud/items/7365352aa3a30e059c64


## nginx で reverse proxy が現実的かな...?

Envoy を外部 proxy LB と見立てて意図的に XFF header を出力させ、その手前を client IP address として allow -> deny rule がはたらくことを確認してみる。

nginx では set_real_ip_from で trusted address を指定しておく。その上で XFF を遡って最後の non-trusted address を client address として拾うように設定する。


## では、Envoy は何のためのものなのか?

https://www.envoyproxy.io/docs/envoy/latest/intro/what_is_envoy
