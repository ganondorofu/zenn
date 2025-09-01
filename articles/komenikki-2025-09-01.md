---
title: "ProxmoxにmacOSを入れてみる"
emoji: "🍎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [macOS, Proxmox, OpenCore, Hackintosh]
published: false
---

# ProxmoxにmacOSを入れてみる

## はじめに

普段はLinuxサーバーやWindows環境を仮想化して使うことが多いですが、  
ふと「Proxmox上でmacOSを動かせないだろうか？」と思い立ちました。  

本記事では、Proxmox環境にmacOSをインストールする手順をまとめてみました。  
※本記事は検証・学習目的であり、ライセンスや動作保証は一切ありません。

## 環境

- Proxmox VE 8.3.0

本来は複雑なVMの設定等が必要なのですが、それを簡略化したスクリプトを作成されている方がいたので今回はこちらを使用しました。  
OSX-PROXMOX - Run macOS on ANY Computer (AMD & Intel)  
https://github.com/luchina-gabriel/OSX-PROXMOX?tab=readme-ov-file

## 手順の流れ

1. **OSX-PROXMOXをインストールする**  
   以下のコマンドをmacOSをインストールしたいProxmoxホストShellで実行します。

   ```bash
   /bin/bash -c "$(curl -fsSL https://install.osx-proxmox.com)"
   ```

   ダウンロード後、ランダムなキーを生成するかとmacOSの製品名を問われます。筆者はキー生成はyでmacOSの製品名はもともと書いてあった内容を入力しました。

   その後自動的に再起動しますが、その際VMやコンテナが起動していると再起動しないことがあります。再起動しない場合は、手動ですべてのVMやコンテナをシャットダウンしてください。

2. **VMの作成**  
   以下のコマンドでスクリプトを起動します。

   ```bash
   osx-setup
   ```

   実行すると以下のように表示されます。1から8までの数字でmacOSのバージョンを選択します。筆者は7のSonomaを選択しました。

   ```
   #######################################################
   ################ O S X - P R O X M O X ################
   ############### https://osx-proxmox.com ###############
   ############### version: 2025.07.23 ###################
   #######################################################

    Next VM ID: 116
    OpenCore version: 1.0.4

   Enter macOS version:
    1 - High Sierra - 10.13
    2 - Mojave - 10.14
    3 - Catalina - 10.15
    4 - Big Sur - 11
    5 - Monterey - 12
    6 - Ventura - 13
    7 - Sonoma - 14
    8 - macOS Sequoia - 15

   Additional options:
    200 - Add Proxmox VE no-subscription repo
    201 - Update OpenCore ISO
    202 - Clear all macOS recovery images
    203 - Remove Proxmox subscription notice
    204 - Add new bridge (macOS in cloud)
    205 - Customize OpenCore config.plist

    0 - Quit (or ENTER)
   ```

   選択すると以下の質問が順に表示されます。  
   VM ID (VM番号 Enterを押すことで自動生成)  
   VM Name (VMのホスト名 Enterで自動生成)  
   Disk Size (ディスクサイズ Enterで初期値)  
   Storage (どのストレージを使うか Enterで初期値)  
   CPU Cores (CPUコア数 Enterで初期値)  
   RAM (RAMサイズ Enterで初期値)  
   Download recovery image? (リカバリーイメージをダウンロードするか)

   インストール時はコア数は2、RAMは4GBを推奨します。インストール時にコア数を増やすと起動しないことがあります。筆者は最初6コアでインストールしたのですが、何度やってもインストール後起動しなかったので、コア数を2にしたら直りました。

   完了したらWebUIに戻りVMを起動します。

3. **macOSのインストール**  
   起動が終わるとmacOSの再インストール画面が表示されるのでDisk Utilityを起動します。

   ![macOS再インストール画面](/images/komenikki-2025-09-01/reinstall_disk.png)

   Apple Inc.VirtIO Block MediaをEraseします。

   ![Disk Utility画面](/images/komenikki-2025-09-01/disk_utility.png)

   完了後左上のリンゴマークからRestartをクリックし再起動します。

   今度はReInstallMacを選択します。

   ![ReInstallMac選択画面](/images/komenikki-2025-09-01/reinstall_reinstall.png)

    その後は画面の指示に従いインストールを進めます。

    これにて完了です。
    お疲れさまでした。
---

## トラブルシューティング
- **白背景に灰色のリンゴが表示されて進まない**: CPUコア数を変えてみてください。筆者の場合、6コアから2コアや8コアに変更したら起動しました。
---

## 性能と制限

### 制限
- **ライセンス問題**: macOSはAppleのライセンスで仮想化が制限されているため、法的グレーゾーン。
- **パフォーマンス**: 物理Macに比べ、仮想化によるオーバーヘッドがある。
- **互換性**: 一部のハードウェアやソフトウェアで動作しない場合あり。
- **サポート**: Apple公式サポート外のため、自己責任。

---

## 参考リンク

- [OSX-PROXMOX GitHub](https://github.com/luchina-gabriel/OSX-PROXMOX)

---

## まとめ

Proxmox上でmacOSを動かすのは手間もありますが、動いたときの感動は大きいです。  
実用性は限定的ですが、仮想化の仕組みを理解する良い題材になると思います。  
今後は、より新しいmacOSバージョンへの対応や、パフォーマンスの改善を期待しています。  
皆さんもぜひ試してみてください！

---

それでは、良い仮想化ライフを！