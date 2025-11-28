---
title: "[解決済み] Synology NAS で容量に余裕があるのに「No space left on device」エラーが発生"
emoji: 📦
type: tech
topics:
  - synology
  - btrfs
  - troubleshooting
  - nas
published: true
---
## TL;DR

ハードリンクを多用するバックアップツールがBtrfsファイルシステムのメタデータ領域を圧迫していました。不要なバックアップセットを削除して領域を解放することで、メタデータ領域不足による書き込みエラーを解決しました。

## 問題の発生

Synology NAS DS-916+ へのMacからの Time Machine バックアップやUbuntuからのNFSコピーがたびたび失敗するようになったので、DS-916+にSSHログインして原因調査してみました。

## 原因調査

### 容量確認

```sh
tats@guppy:/volume1$ df
Filesystem              1K-blocks       Used  Available Use% Mounted on
/dev/md0                  8191352    1436808    6635760  18% /
devtmpfs                  4042724          0    4042724   0% /dev
tmpfs                     4046564          4    4046560   1% /dev/shm
tmpfs                     4046564      24928    4021636   1% /run
tmpfs                     4046564          0    4046564   0% /sys/fs/cgroup
tmpfs                     4046564      30768    4015796   1% /tmp
/dev/loop0                  27633        768      24572   4% /tmp/SynologyAuthService
/dev/mapper/cachedev_1 1909582728  365899372 1543683356  20% /volume2
/dev/mapper/cachedev_2 3739641448 2371952676 1367688772  64% /volume1
/dev/mapper/cachedev_0 3739641448 2875991224  863650224  77% /volume3
/volume3/@Archive@     3739641448 2875991224  863650224  77% /volume3/Archive
/volume2/@homes@       1909582728  365899372 1543683356  20% /volume2/homes
tats@guppy:/volume1$ sudo touch dummy
touch: cannot touch 'dummy': No space left on device
```

バックアップ先の`/volume1`は64%しか使用していないにもかかわらず `No space left on device` エラーが起きます。`touch`コマンドは0byteのファイルを作成するにもかかわらず失敗するということは、実際には容量不足ではなさそうです。

### inode確認

こういうときはinode不足が考えられるので、inodeの使用状況を確認してみます。

```sh
tats@guppy:~$ df -i
Filesystem              Inodes IUsed   IFree IUse% Mounted on
/dev/md0                524288 43534  480754    9% /
devtmpfs               1010681   884 1009797    1% /dev
tmpfs                  1011641     2 1011639    1% /dev/shm
tmpfs                  1011641  1485 1010156    1% /run
tmpfs                  1011641    10 1011631    1% /sys/fs/cgroup
tmpfs                  1011641   150 1011491    1% /tmp
/dev/loop0                8192    10    8182    1% /tmp/SynologyAuthService
/dev/mapper/cachedev_1       0     0       0     - /volume2
/dev/mapper/cachedev_2       0     0       0     - /volume1
/dev/mapper/cachedev_0       0     0       0     - /volume3
/volume3/@Archive@           0     0       0     - /volume3/Archive
/volume2/@homes@             0     0       0     - /volume2/homes
```

`/volume1`のinode状況はうまく取得できませんでした。

### メタデータ領域の確認

Btrfsファイルシステムは`btrfs`コマンドで使用状況の詳細を確認できます。

```sh
tats@guppy:/volume1$ sudo btrfs filesystem df /volume1
Data, single: total=3.41TiB, used=2.14TiB
System, DUP: total=8.00MiB, used=96.00KiB
Metadata, DUP: total=111.50GiB, used=109.38GiB
GlobalReserve, single: total=2.00GiB, used=0.00B
```

メタデータ領域は111.50GB中109.38GB使っており、2%ほどしか空き容量がありません。どうもこれが原因のようです。

:::message
Btrfsでは、実際のファイルデータとは別に「メタデータ」という領域があります。メタデータにはファイル名 / 権限 / タイムスタンプ / ディレクトリ構造 / ハードリンクの情報などが格納されます。ハードリンクが大量にあると、同じファイルを指す複数のメタデータエントリを必要とするため、メタデータ領域を大量に消費します。
:::

## 解決方法

### バックアップセットの削除

私はUbuntuのバックアップに [Rsync time backup](https://github.com/laurent22/rsync-time-backup) というツールを使っています。このツールは各バックアップセット間で変更のないファイルはハードリンクを使用しています。そのため容量は多重で消費しないものの、inodeは100個のバックアップセットがあったら100倍使用します。これがinodeを多量に消費している可能性が高いため、3分の1程度のバックアップセットを削除してみました。その後、ファイルシステムの状況を再度確認してみます。

```sh
tats@guppy:/volume1$ sudo btrfs filesystem df /volume1
Data, single: total=3.41TiB, used=2.20TiB
System, DUP: total=8.00MiB, used=112.00KiB
Metadata, DUP: total=112.99GiB, used=76.92GiB
GlobalReserve, single: total=2.00GiB, used=0.00B
```

メタデータ領域の使用量が109.38GBから76.92GBに減り、問題なくファイルが作成できるようになりました。

## 予防策: Rsync time backup の設定変更

Rsync time backup は`--strategy`オプションで有効期限とバックアップセット数を設定できます。デフォルトは `"1:1 30:7 365:30"` で、これは「1日後は1日に1個のバックアップセットのみ維持する。30日後は7日に1個のバックアップセットのみ維持する。365日後は30日に1個のバックアップセットを維持する」の意味です。週次のバックアップセットは1年分もいらないので、これを3カ月分に変更しました。

```sh
nice -n 10 \
  tmbackup --strategy "1:1 30:7 90:30" \
  $src $dest /etc/tmbackup.exclude > /dev/null
```

:::message
`nice -n 10` は、バックアップ処理の優先度を下げて実行するコマンドです。数値が大きいほど優先度が低くなり（-20〜19の範囲）、他のプロセスへの影響を最小限に抑えながらバックグラウンドでバックアップを実行できます。
:::

## まとめ

Synology NAS で `No space left on device` エラーが発生した際は、単純な容量不足だけでなくメタデータ領域の不足も疑う必要があります。特にハードリンクを多用するバックアップツール（rsync-time-backup / duplicity / borgbackup / rdiff-backupなど）を使用している場合、大量のinodeを消費してメタデータ領域を圧迫する可能性があります。

対処法：
- `btrfs filesystem df` でメタデータ領域の使用状況を確認
- 不要なバックアップセットを削除してメタデータ領域を解放
- 定期的なバックアップセットの整理を検討

このような問題を避けるため、バックアップ戦略を見直し、保持するバックアップセット数を適切に管理することが重要です。

## この記事の動作環境

- Synology DSM: 7.2.2-72806 Update 4
