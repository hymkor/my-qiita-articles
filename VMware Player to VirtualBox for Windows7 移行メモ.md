---
title: VMware Player to VirtualBox for Windows7 移行メモ
tags: vmware_player VirtualBox Windows:7
author: zetamatta
slide: false
---
今まで VMware Player で幸せに開発環境を維持ってきた。Workstation と違い、標準ではスナップショットが取れないけど、NHM ( http://euee.web.fc2.com/tool/nhm/nhm.html )というソフトウェアを併用すると、スナップショットが取れるので、まぁ、遜色なかった。

だが、いつの間にか、VMware Player の商業利用の制限が広がってしまった。前は仮想環境を売ってはいけない程度だったのに、今は仕事で使うこと自体ダメになってしまったようだ。

つーことで、前職で Linux 用に利用していた VirtualBox に移行したいなと思っていたんだが、今はWindows。アクティベーションで面倒なことになるのも嫌なので二の足を踏んでいた。が、英語の記事だけど、ハードウェアの変更にならずに、綺麗に移行する方法があるようなので、さっそく試してみた。

* [Migrating Windows 7 from VMWare Fusion to VirtualBox](http://blog.csanchez.org/2011/08/01/migrating-windows-7-from-vmware-fusion-to-virtualbox/)

これに従ってやってみたら、ほぼ問題なくうまくでけた。記憶と記事を頼りに振り返ってみると：

* 移行前に VMware Player 上で、ゲストOS内の Addition をアンインストールしておく
* ゲストOSをシャットダウンして、*.vmdk を VirtualBox 用に全てコピーしとく
* VirtualBox では、ゲスト OS を起動する前に以下をやっておく
  * ハードディスクを SATA から外し、IDE のプライマリに付け直す。
     * あと、IDE を PIIX4 without host I/O cache とやらにしないといけなかったらしかったが、ちゃんと設定したか覚えていない。
  * メモリは VMware の時と同じサイズに設定し直す
  * IO APIC とやらを Enable にしておく
  * 記事だと、OS設定は Windows7 64bit とあったが、自分のイメージは32bitなので、Windows7 32bit にした。

元の VM イメージにもよると思うが、自分の場合、一応、できたという事例ということで。

