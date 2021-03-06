---
title: WindowsからVM上のLinuxをSSH経由で利用する開発環境の構築
author: azu
layout: post
permalink: /2011/0420/res2588/
SBM_count:
  - '00018<>1355446538<>18<>0<>0<>0<>0'
dsq_thread_id:
  - 300803230
categories:
  - インストール設定
tags:
  - Git
  - Node.js
  - VM
  - Windows
  - セキュリティ
  - 設定
  - 開発環境
---
[VirtualBox][1] or[VMWare Player][2]でLinux環境をWindows 7&#215;64に構築するメモ   
今回は[VirtualBox][1]と[Turnkey Linux core][3]を使って構築した。

**と見せかけて、最終的には[Ubuntu Server][4]使う事にしたので途中まで飛ばしていいです。**

なんでVMを使ってまでやるかというと   
WIndowsでのCUIは[Console][5]+[NYAOS][6]でコンソールとしていいのですが、node.jsなど実行できないものが出てきたので、VM上に環境を作ることにしました。   
Cygwin : 食わず嫌いでしたが、食ったら嫌いでした。   
coLinux : [64 bit][7]が非対応でした。 

必要なもの

*   [VirtualBox][1](仮想化ソフトウェア） 
*   [TurnKey Core][3](サーバー、そこら辺の便利なソフトが入ってる感じのディストリビューション) 
*   [RLogin][8](SSHクライアント) 

Turnkey Linux coreはVM向けにovf形式でも配布してるので、OVFと書かれてるリンクからturnkey-core-バージョン-lucid-x86-ovf.zipをダウンロードして使う。

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss11" border="0" alt="2011-04-20-ss11" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss11_thumb.png" width="640" height="153" />][9]

VirtualBoxを起動してメニューの仮想アプライアンスのインポートから、先ほどのovfをインポートすると自動でTurnKey Coreが仮想マシン一覧に並ぶ。(設定するのは仮想マシンの名前ぐらい）

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss8" border="0" alt="2011-04-20-ss8" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss8_thumb.png" width="240" height="170" />][10]

起動するとパスワードの設定などがあって、パスワード以外はEnter押してればいいと思う。   
設定が終わると起動して下のようなメニュー画面が表示される。

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss12" border="0" alt="2011-04-20-ss12" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss12_thumb.png" width="240" height="209" />][11]   
メニューを終了させると、CUIで操作できるけどキーやマウスの関係で扱いにくいのでSSHからアクセスして操作する。

[RLogin][8]を起動して、サーバの接続から新規追加して、プロトコロルにSSH、アドレスにはLinuxサーバーのIPアドレス、ユーザーはrootで、パスワードは最初の起動時に設定したものを入力して接続する

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss13" border="0" alt="2011-04-20-ss13" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss13_thumb.png" width="226" height="240" />][12]

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss9" border="0" alt="2011-04-20-ss9" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss9_thumb.png" width="240" height="141" />][13]

こっからはサーバー{ゲスト側(Ubuntu)}の設定

まずはrootだとあんまりよくないので、ユーザー(azuという例で)を追加、そのユーザーのパスワードを設定する。

<div>
  <pre id="codeSnippet" class="csharpcode">root@core ~<span class="rem"># useradd -m -s /bin/bash azu</span>
<span class="rem"># ユーザーazuを追加する。mオプションがないとHOMEディレクトリが追加されなかった</span>
root@core ~<span class="rem"># ls /home/     </span>
azu/
<span class="rem"># HOMEディレクトリがあるのを確認</span>
root@core ~<span class="rem"># passwd azu</span>
<span class="rem"># パスワードの設定</span>
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully</pre>
</div>

<div>
  useraddの-sオプションでログインシェルを決めないとかなり不自由な感じになります。<br /> <br />後からログインシェルを決める場合はchsh -s /bin/bash などとする。
</div>

<div>
  と、このままTurnkey Linuxを使おうと思っていたんだけど、ファイル共有でどうしても上手くGuest Additionsのインストールが上手くできないのと、なんかroot前提なような環境で他とは少し違った感じで躓く事がありそうだったので、<a href="http://www.ubuntu.com/business/server/overview">Ubuntu Server</a>に切り替えました。
</div>

<div>
  &#160;
</div>

### [**Ubuntu Server**][4]**を使って環境構築(改めて)**

<div>
  (ちょこちょログにTurnkeyが出てくるのはそのときの名残です。プロンプトの文字は無視してください)
</div>

<div>
  (Turnkey Linuxの事は忘れてください)
</div>

<div>
  必要なもの
</div>

*   [VirtualBox][1](仮想化ソフトウェア） 
*   [Ubuntu Server][4] (ゲストOS) 
*   [RLogin][8](SSHクライアント) 

<div>
  <table border="0" cellspacing="0" cellpadding="2" width="400">
    <tr>
      <td valign="top" width="200">
        <p>
          ホストOS
        </p>
      </td>
      
      <td valign="top" width="200">
        <p>
          ゲストOS
        </p>
      </td>
    </tr>
    
    <tr>
      <td valign="top" width="200">
        <p>
          Windows 7 64bit
        </p>
      </td>
      
      <td valign="top" width="200">
        <p>
          Ubuntu Server
        </p>
      </td>
    </tr>
  </table>
</div>

<div>
  で環境を作っていきます。
</div>

<div>
  <a href="http://www.ubuntu.com/business/server/overview">Ubuntu Server</a>のisoをダウンロードしてきて、新規仮想マシン作成から適当な配分で仮想マシンを作りますが、<strong>ネットワークをブリッジ接続</strong>に変更しないとUbuntu ServerのIPアドレスが10.0.2.25とかいう感じになってSSH接続できなかったので、ネットワークをブリッジ接続に変更して作成しました。
</div>

<div>
  <a href="http://efcl.info/wp-content/uploads/2011/04/image.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://efcl.info/wp-content/uploads/2011/04/image_thumb.png" width="240" height="186" /></a>
</div>

<div>
  <strong>追記</strong>: ネットワークがNATでもポートフォワーディングすればSSH接続できました(こっちの方がいいかも)
</div>

<div>
  ネットワークの設定をNATにしてから、高度の設定でポートフォワーディングにホストには任意のポート、ゲストにはUbuntu側に設定したSSHのポート番号(デフォルト22)を設定します。
</div>

<div>
  <a href="http://efcl.info/wp-content/uploads/2011/04/2011-04-23-ss1.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="2011-04-23-ss1" border="0" alt="2011-04-23-ss1" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-23-ss1_thumb.png" width="240" height="120" /></a>
</div>

<div>
  この状態で、RLoginに接続IPアドレスに127.0.0.1 or localhost で、ポート番号にはホストに設定したポート番号を入力すればSSH接続できます。<br /> <br />任意のポートだけを開く感じで使えるのでこっちの方がいい気がします。同様の方法でWebのポートである8080もポートフォワーディングに設定しました。
</div>

*   [VirtualBox + CentOSでNAT接続のポートフォワーディングを行う方法 &#8211; 大人になったら肺呼吸][14]
*   [VirtualBox4.04にCentOS5.5をインストール &#8211; sdhrの日記][15]
*   [VirtualBox 4.0.2のゲストにsshで接続してみる。 « chocokanpan BLOG][16]

&#160;

<div>
  仮想マシンを起動したUbuntu Serverのインストール画面でユーザーアカウントの作成ができるので、下のガイドに従って入力していくだけで先ほどのTurnkey Linux でのユーザーアカウント追加までと同じ事ができます。
</div>

*   [どうぐばこ » ubuntu server インストール ガイド][17] 

注意点としては*１８．サーバーソフトウェアの選択画面*でOpenSSH SERVERを選択してSSHでつなげるようにしておくと楽でいいです(スペースキーで選択チェックが入る)

忘れた場合でも

<div>
  <pre id="codeSnippet" class="csharpcode">sudo apt-get install openssh-server</pre>
</div>

<div>
  とすればいいだけなので、そこまで問題ないです。
</div>

<div>
  Ubuntu Serverはそのまま使うと<a href="http://hamamuratakuo.blog61.fc2.com/blog-entry-400.html">文字化け</a>して扱いにくいので、<a href="http://nanno.dip.jp/softlib/man/rlogin/">RLogin</a>を使ってアクセスすれば文字化け対策をしなくてもいいので、最初から<a href="http://nanno.dip.jp/softlib/man/rlogin/">RLogin</a>を使って作業します。</p> <p>
    サーバのIPアドレスは</div> <pre>ifconfig</pre>
    
    <p>
      で、わかると思います。<br /> <br />初期設定なら、インストール時に入力したアカウントとパスワードでログインできると思います。<strong></strong>
    </p>
    
    <p>
      <strong><br /> <br />SSHで鍵を使って接続</strong>
    </p>
    
    <p>
      セキュリティ的にパスワードではなく鍵でSSHをつなぐのが普通だと思うので、SSHの鍵設定をします。
    </p>
    
    <div>
      <pre id="codeSnippet" class="csharpcode">root@core ~<span class="rem"># cd /home/azu/</span>
<span class="rem"># ユーザのHOMEへ</span>
root@core /home/azu<span class="rem"># mkdir .ssh</span>
root@core /home/azu<span class="rem"># cd .ssh</span>
<span class="rem"># .sshディレクトリを作って移動</span>
root@core azu/.ssh<span class="rem"># ssh-keygen -t rsa</span>
<span class="rem"># 鍵を生成する</span>
Generating public/<span class="kwrd">private</span> rsa key pair.
Enter file <span class="kwrd">in</span> which to save the key (/root/.ssh/id_rsa): turnkey <span class="rem"># ファイル名は適当に</span>
Enter passphrase (empty <span class="kwrd">for</span> no passphrase): 
Enter same passphrase again: 
Your identification has been saved <span class="kwrd">in</span> turnkey.
Your public key has been saved <span class="kwrd">in</span> turnkey.pub.
root@core azu/.ssh<span class="rem"># ls</span>
turnkey  turnkey.pub
<span class="rem"># .sshディレクトリに秘密鍵と暗号鍵が生成される</span>
root@core azu/.ssh<span class="rem"># mv turnkey.pub authorized_keys</span>
<span class="rem"># turnkey.pub を authorized_keysにリネームする</span>
root@core azu/.ssh<span class="rem"># ls</span>
authorized_keys  turnkey</pre>
    </div>
    
    <div>
      鍵は生成して公開鍵(authorized_keys)の登録ができたので、秘密鍵をホスト側のPCに転送します。<br /> <br /><a href="http://nanno.dip.jp/softlib/man/rlogin/">RLogin</a>にファイル転機能がついてるので、.sshディレクトリにある秘密鍵(turnkey)をホスト側に移動させます。 </p> <p>
        ゲスト側(Ubuntu)に秘密鍵は置いておく必要はないので、秘密鍵(turnkey)は転送したら削除します。</div> <div>
          <a href="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss15.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="2011-04-20-ss15" border="0" alt="2011-04-20-ss15" src="http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss15_thumb.png" width="240" height="115" /></a>
        </div>
        
        <div>
          次にSSHの設定で、鍵以外でのログインはできないように/etc/ssh/sshd_configを書き換えます。
        </div>
        
        <div id="codeSnippetWrapper">
          <pre id="codeSnippet" class="csharpcode"><span class="rem"># それぞれをnoに書き換える </span>
PermitRootLogin no  
PasswordAuthentication no  
UsePAM no
# 面倒だったので一度rebootした</pre>
          
          <p>
            自分はportも22から適当なものに変更しました。
          </p>
        </div>
        
        <div>
          後はRLoginに秘密鍵を登録してSSHログインするだけです。
        </div>
        
        <div>
          SSH Identity keyに転送した秘密鍵をセットして、ポートを変えた場合はポートも任意のものに設定してから接続します。
        </div>
        
        <ul>
          <li>
            <a href="http://d.hatena.ne.jp/Fiore/20080228/1204174833">Ubuntuでsshdの設定をしてリモートから接続できるようにする &#8211; そ、そんなことないんだから！</a>
          </li>
          <li>
            <a href="http://blog.myfinder.jp/2010/09/vpsssh.html">myfinder&#8217;s blog: さくらのVPSを借りたら真っ先にやるべきssh設定</a>
          </li>
        </ul>
        
        <div>
          <strong>共有フォルダの設定</strong>
        </div>
        
        <div>
          Ubuntu Server にはGUIがないので、Guest AdditionsのインストールもCUIで行わないといけません。
        </div>
        
        <div>
          ここで結構はまりました
        </div>
        
        <ul>
          <li>
            <a href="http://amis-annex.posterous.com/virtualbox-4xguest-ubuntu-server">virtualbox 4.x/Guest ubuntu-server で共有フォルダを使う &#8211; As mind is suitable&#8230;</a>
          </li>
          <li>
            <a href="http://blog.brettalton.com/2010/04/28/installing-guest-additions-in-virtualbox-for-an-ubuntu-server-guest/">Articles • brettalton.com</a>
          </li>
        </ul>
        
        <p>
          が大変参考になった。
        </p>
        
        <p>
          まずは適当な共有フォルダを設定しておく。
        </p>
        
        <p>
          <a href="http://efcl.info/wp-content/uploads/2011/04/image1.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://efcl.info/wp-content/uploads/2011/04/image_thumb1.png" width="240" height="214" /></a>
        </p>
        
        <p>
          そして、起動してるVMのメニューにある”<em>Guest Additionsのインストール</em>”を押しておく。
        </p>
        
        <p>
          何にも起きてないように見えるが、CDドライブにCDが入った感じになる。
        </p>
        
        <pre id="codeSnippet" class="csharpcode">sudo apt-get install build-essential linux-headers-`uname -r`
sudo apt-get install xserver-xorg xserver-xorg-core
<span class="rem">#必要なものを先にインストールしておく</span>
<span class="rem">###################</span>
<span class="rem"># ここまでにGuest Additionsのインストールを押してマウントの準備が必要</span>
<span class="rem">###################</span>
mkdir /tmp/cdrom
sudo mount /dev/sr0 /tmp/cdrom
<span class="rem"># Guest Additionsのディスクをマウントする</span>
cd /tmp/cdrom
sudo bash VBoxLinuxAdditions.run  --nox11
sudo addgroup --system --quiet vboxsf
sudo usermod -a -G vboxsf azu <span class="rem"># ユーザーをvboxsfグループに加える</span>
sudo reboot
# リブート

azu@ubuntu:/media$ ls
cdrom&#160; sf_azu</pre>
        
        <div>
          自動マウントするためにはユーザーをvboxsfグループというグループに加える必要があることに注意。<br /> <br />リブートすると/media以下にsf_フォルダ名が現れてアクセスできるようになる。
        </div>
        
        <div>
          大体ここまで基礎的な環境ができあがるので、後は好きなソフトを入れていく感じになると思います。(一応スナップショットをとっておきました)
        </div>
        
        <div>
          Ubuntu Server + VirtualBox GUIありでメモリ使用量は45MBぐらい、VBoxHeadless.exeで画面表示なしで起動させると30MBになって結構メモリ使用量は少ない。<br /> <br />VirtualBoxをタスクトレイに入れて管理するには<a href="http://www.toptensoftware.com/VBoxHeadlessTray/">VBoxHeadlessTray</a>がおすすめ。ヘッドレスモードやシャットダウンとなどの操作もタスクトレイで行えるのでとてもいい。
        </div>
        
        <p>
          ついでにNode.jsの環境も作ってみる<br /> <br />Node.jsは直接入れるよりもバージョン管理するツールから入れるのがいいらしいので、naveかnvmを使う事にした。
        </p>
        
        <p>
          <a href="https://github.com/isaacs/nave">nave</a>は何かインストールが面倒だったので、<a href="https://github.com/creationix/nvm">nvm</a>を使う事にした。
        </p>
        
        <p>
          nvmは自動でnpmもインストールしてくれるので便利。
        </p>
        
        <div id="codeSnippetWrapper">
          <pre id="codeSnippet" class="csharpcode">azu@ubuntu:~$ sudo apt-get install build-essential libssl-dev git-core 
azu@ubuntu:~$ sudo apt-get install curl
<span class="rem"># 必要なものを先にインストールしておく</span>
azu@ubuntu:~$ git clone git://github.com/creationix/nvm.git ~/.nvm          
azu@ubuntu:~$ cd .nvm/
azu@ubuntu:~$ bash ./nvm.sh
azu@ubuntu:~$ nvm install latest
<span class="rem"># 最新のnodeをインストールする</span>
azu@ubuntu:~$ node -v           
v0.4.5</pre>
        </div>
        
        <p>
          毎回 bash ./nvm.sh 実行するのは手間なので。<br /> <br />.bashrc を編集して、
        </p>
        
        <div id="codeSnippetWrapper">
          <pre id="codeSnippet" class="csharpcode">. ~/.node/nvm.sh
nvm use latest</pre>
        </div>
        
        <p>
          を書き加えておきます。
        </p>
        
        <ul>
          <li>
            <a href="http://blog.summerwind.jp/archives/1464">SummerWind &#8211; Node.jsの管理はnvmで</a>
          </li>
          <li>
            <a href="http://www.atmarkit.co.jp/fwcr/rensai2/nodejs02/01.html">naveでNode.jsのバージョン管理＆イベントループ詳説（1/3）- ＠IT</a>
          </li>
          <li>
            <a href="http://d.hatena.ne.jp/t_43z/20110304/1299222231">Amazon EC2 MicroインスタンスのAmazon LinuxにNodeをインストールした &#8211; 自分の感受性くらい</a>
          </li>
        </ul>
        
        <p>
          <strong>雑記</strong>
        </p>
        
        <p>
          VMWareはスタートアップにいろんなもの生やすし、ダウンロードも登録必要で面倒なので、VMWareよりもVirtualBoxの方が好みでした。
        </p>
        
        <p>
          目標としては、<a href="http://d.hatena.ne.jp/takuya_1st/20110214/1297688680">Cygwin Here</a>みたくWindowsのエクスプローラー上のコンテキストメニューから、SSHクライアントを開いてそのときに同時に共有フォルダのsf_azu以下にある同じフォルダまで移動したいのだけど、お客様の中でよい方法をお知りな方がいらしゃったらお願いします。
        </p>
        
        <p>
          e.g)
        </p>
        
        <p>
          windows : C:UsersazuDownloads のコンテキストメニューからSSHクライアントを開く
        </p>
        
        <p>
          Ubuntu&#160;&#160; : 渡されたパスを元に /media/sf_azu/Downloads をカレントディレクトリにする
        </p>
        
        <p>
          SSHクライアントのマクロみたいので実現するのかな。
        </p>
        
        <div style="position: absolute; width: 1px; height: 1px; overflow: hidden; top: 1680px; left: -10000px" id="_mcePaste" class="mcePaste">
          Turnkey Linuxの
        </div>

 [1]: http://www.virtualbox.org/
 [2]: http://www.vmware.com/products/player/
 [3]: http://www.turnkeylinux.org/core
 [4]: http://www.ubuntu.com/business/server/overview
 [5]: http://sourceforge.net/projects/console/
 [6]: http://www.nyaos.org/
 [7]: http://colinux.wikia.com/wiki/Dashboard_for_developing_a_64_bit_coLinux
 [8]: http://nanno.dip.jp/softlib/man/rlogin/
 [9]: http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss11.png
 [10]: http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss8.png
 [11]: http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss12.png
 [12]: http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss13.png
 [13]: http://efcl.info/wp-content/uploads/2011/04/2011-04-20-ss9.png
 [14]: http://d.hatena.ne.jp/replication/20110423/1303525759
 [15]: http://d.hatena.ne.jp/sdhr/20110219/1298111849
 [16]: http://chocokanpan.net/archives/374
 [17]: http://studio-bey.chicappa.jp/dougubako/ubuntu_server/269