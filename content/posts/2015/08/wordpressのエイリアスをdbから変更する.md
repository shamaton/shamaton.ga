---
title: wordpressのエイリアスをDBから変更する
author: しゃまとん
type: post
date: 2015-08-04T15:34:59+00:00
url: /archives/105
featured_image: /wp-content/uploads/2015/08/wordpress-logo-stacked-rgb.png
categories:
  - Web関連

---
お世話になっております。  
しゃまとんです。

自分のサイトですが、wordpressを使って、shamaton.orz.hmでアクセスするとTOPページが表示されるようにしていたのですが、新たにTOPページをつくり、別のコンテンツページを簡単に用意できるようにしたかったので、

shamaton.orz.hm

↓

shamaton.orz.hm/blog

で表示されるように変更しました。  
変更するに辺り、色々なやり方があると思いますが、こちらではwordpress用に作成されたdatabaseに手を加えることによって変更する手順となります。

<!--more-->

> 注意！！
> 
> この手順ではデータベースのレコードを更新するため、動かなくなる可能性があります。  
> 実際に試す際は、データベースのバックアップ等を行ってからの作業をおすすめします。

参考にされたサイトによっては、データベースの名前やテーブル名が違う可能性がありますが、当方ではこちらのサイトを参考にwordpressを導入しました。

<a title="ブログサイト構築" href="http://centossrv.com/wordpress.shtml" target="_blank">ブログサイト構築(WordPress)</a>

手順としては、割りと単純です。  
簡単な構成であればApache上でwordpressが動いていると思います。

■ 各テーブル内に記載されたURLを置換する  
投稿した記事はテーブルに保存されており、guidやpost_contentに現在動作しているURLが記載されていたりします。それらを置換してやります。

<pre class="brush: text; gutter: true">-- wp_posts
UPDATE wp_posts SET guid = REPLACE (guid, &#039;http://shamaton.orz.hm&#039;, &#039;http://shamaton.orz.hm/blog&#039;);
UPDATE wp_posts SET post_content = REPLACE (post_content, &#039;http://shamaton.orz.hm&#039;, &#039;http://shamaton.orz.hm/blog&#039;);

-- wp_options
UPDATE wp_options SET option_value = REPLACE (option_value, &#039;http://shamaton.orz.hm&#039;, &#039;http://shamaton.orz.hm/blog/&#039;) where option_name = &#039;home&#039; OR option_name = &#039;siteurl&#039;;

-- wp_postmeta
UPDATE wp_postmeta SET meta_value = REPLACE (meta_value, &#039;http://shamaton.orz.hm&#039;, &#039;http://shamaton.orz.hm/blog/&#039;);</pre>

必要に応じてURLを入れ替えてください。

実行後は意図通りに更新されているかselectしてみると良いかと思います。  
ex.) select _from wp\_options where option\_name = &#8216;home&#8217; OR option_name = &#8216;siteurl&#8217;;_

■ apacheの設定ファイルを修正して、モジュール群を移動させる  
場合によってはこちらの作業は不要かもしれません。

当方のサイトでは記事が見つからない事象が発生したため、下記対応をしました。  
1. /etc/httpd/conf/httpd.confを編集

DocumentRoot &#8220;/var/www/wordpress&#8221;  
↓  
DocumentRoot &#8220;/var/www/html&#8221;

<Directory &#8220;/var/www/wordpress&#8221;>  
↓  
<Directory &#8220;/var/www/html&#8221;>

2. /etc/httpd/conf.d/wordpress.confを編集  
Alias /blog /var/www/html/wordpress/

3. /var/www/wordpress を /war/www/html/wordpress に移動  
mv /var/www/wordpress /var/www/html/wordpress

これで以前に記事に新しいURLから、表示できるか確認してみましょう。  
心配な方はVirtualBox等、利用して試してみてもよいかもしれません。

以上です。

■参考  
<a href="http://notnil-creative.com/blog/archives/446" target="_blank">notnil creation weblog &#8211; SQL文でドメイン名を一括置換</a>

<a href="http://www.risewill.co.jp/blog/archives/1111" target="_blank">mysqldump(DBバックアップ)の使い方</a>