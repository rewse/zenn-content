---
title: "[解決済み] macOS Tahoe でNASへの Time Machine 新規バックアップができない"
emoji: "️🕰️"
type: "tech"
topics: ["macos", "timemachine", "nas", "synology", "troubleshooting"]
published: true
---

## TL;DR

日本語環境の macOS Tahoe でNASへの Time Machine 新規バックアップが失敗する場合、一時的にmacOSの優先言語を英語にすると解決します。

## 発生した問題

私はmacOSの Time Machine バックアップをSMB経由で Synology NAS DS-916+ に取っています。先日バックアップ領域が不足したためNAS上のSparsebundleファイルを削除して、新規でバックアップしようとすると常に失敗するようになりました。

macOSのログには `Permission denied` や`BACKUP_FAILED_DISCONNECTED_DISK_IMAGE`といったエラーが出ています。

```bash
log show --predicate 'subsystem == "com.apple.TimeMachine"' \
  --info --last 10m | grep -E "(Error|Failed)" | tail -100
```

```log
2025-12-14 23:05:53.257791+0900 0x7b2f7    Error       0x0                  627    0    SystemUIServer: (TimeMachine) [com.apple.TimeMachine:General] Failed to read capabilities for '/Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup', error: Operation not permitted
2025-12-14 23:05:53.257988+0900 0x7b2f7    Error       0x0                  627    0    SystemUIServer: (TimeMachine) [com.apple.TimeMachine:General] Failed to get resource value 'NSURLVolumeURLForRemountingKey' for '/Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup', error: Error Domain=NSCocoaErrorDomain Code=257 "ファイル"Backup"を表示するためのアクセス権がないため、開けませんでした。" UserInfo={NSURL=file:///Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup, NSFilePath=/Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup, NSUnderlyingError=0xb6bdcc930 {Error Domain=NSPOSIXErrorDomain Code=13 "Permission denied"}}
2025-12-14 23:05:53.266305+0900 0x7b294    Error       0x0                  11046  0    TimeMachineSettings: (TimeMachine) [com.apple.TimeMachine:General] Failed to get resource value 'NSURLVolumeURLForRemountingKey' for '/Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup', error: Error Domain=NSCocoaErrorDomain Code=257 "ファイル"Backup"を表示するためのアクセス権がないため、開けませんでした。" UserInfo={NSURL=file:///Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup, NSFilePath=/Volumes/.timemachine/guppy._smb._tcp.local./8DC1E582-2107-4E13-8F25-43799A7F5046/Backup, NSUnderlyingError=0xcab3ea1f0 {Error Domain=NSPOSIXErrorDomain Code=13 "Permission denied"}}
2025-12-14 23:05:55.938978+0900 0x7b293    Error       0x198067             528    0    backupd: (TimeMachine) [com.apple.TimeMachine:MountPointValidity] '/Volumes/youthのバックアップ' - no volume mounted at this path
2025-12-14 23:05:55.939163+0900 0x7b292    Error       0x198067             528    0    backupd: (TimeMachine) [com.apple.TimeMachine:UnmountChecker] Failing backup with BACKUP_FAILED_DISCONNECTED_DISK_IMAGE (70)
```

## 原因

日本語環境のmacOSは、新規バックアップ中に `<ホスト名>のバックアップ` という名前でディスクイメージを作成します。しかし、macOS Tahoe では、濁点または半濁点のある文字（`バックアップ`の`バ`と`プ`）がディスクイメージのファイル名に含まれていると正しく動作しません。

## 解決策

1. `システム設定 > 一般 > 言語と地域 > 優先する言語` で、先頭を`English`に
1. macOSを再起動
1. 新規 Time Machine バックアップを取得
1. `System Settings > General > Language & Region` で、先頭を`日本語`に

![macOS システム設定の言語と地域で優先言語をEnglishに変更した画面](/images/macos-tahoe-timemachine-backup-fix/language-settings-english.png)

英語環境では `Backups of <ホスト名>` というディスクイメージ名で作成されるため、問題が起きません。バックアップを一度取得した後は既存の `Backups of <ホスト名>` という名のディスクイメージを使うので、優先言語を日本語に戻しても問題ありません。

## 参考

https://discussions.apple.com/thread/256137390?sortBy=rank

## この記事の動作環境

- macOS: Tahoe 26.2
- Synology DSM: 7.2.2-72806 Update 5
