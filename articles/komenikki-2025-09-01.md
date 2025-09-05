---
title: "ProxmoxにmacOSをインストールする"
emoji: "🍎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [macOS, Proxmox, OpenCore, Hackintosh]
published: true
---

# ProxmoxにmacOSをインストールする

## はじめに

普段はLinuxサーバーやWindows環境を仮想化して使うことが多いですが、  
ふと「Proxmox上でmacOSを動かせないだろうか？」と思い立ちました。  

本記事では、Proxmox環境にmacOSをインストールする手順をまとめてみました。  
※本記事は検証・学習目的であり、ライセンスや動作保証は一切ありません。

### 対象読者
- Proxmoxの基本操作（VM作成、WebUIの使用）に慣れている方
- 仮想化技術やmacOSに興味がある中級者以上

### 前提知識
- **Proxmox**: Debianベースの仮想化プラットフォーム。VMやコンテナを管理できます。
- **VM (Virtual Machine)**: 仮想マシン。物理マシン上で動作する仮想的なコンピュータ。
- **OpenCore**: macOSのブートローダー。Hackintoshなどで使用され、macOSを非公式ハードウェアで動作させるためのツール。

### 注意事項
この方法はAppleのライセンスに違反する可能性があり、実用的な使用は推奨しません。学習・実験目的のみでお使いください。

## 環境

- Proxmox VE 8.3.0

本来は複雑なVMの設定等が必要なのですが、それを簡略化したスクリプトを作成されている方がいたので今回はこちらを使用しました。  
OSX-PROXMOX - Run macOS on ANY Computer (AMD & Intel)  
https://github.com/luchina-gabriel/OSX-PROXMOX?tab=readme-ov-file

## 手順

## 手順

1. **OSX-PROXMOXをインストールする**  
   以下のコマンドをmacOSをインストールしたいProxmoxホストShellで実行します。

   ```bash
   /bin/bash -c "$(curl -fsSL https://install.osx-proxmox.com)"
   ```

   このコマンドは、OSX-PROXMOXスクリプトをダウンロード・インストールします。  
   ダウンロード後、ランダムなキーを生成するかとmacOSの製品名を問われます。筆者はキー生成はyでmacOSの製品名はもともと書いてあった内容をそのまま入力しました。

   その後自動的に再起動しますが、その際VMやコンテナが起動していると再起動しないことがあります。再起動しない場合は、手動ですべてのVMやコンテナをシャットダウンしてください。

2. **VMの作成**  
   以下のコマンドでスクリプトを起動します。

   ```bash
   osx-setup
   ```

   このコマンドで、macOS VM作成の対話型セットアップが開始されます。  
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
   起動が終わるとVMのコンソールにmacOSの再インストール画面が表示されるのでDisk Utilityを起動します。

   以下の画像のように、macOSの再インストール画面が表示されます：

   ![macOS再インストール画面 - Disk Utilityを選択](/images/komenikki-2025-09-01/reinstall_disk.png)

   次に、Apple Inc.VirtIO Block MediaをEraseします。

   以下の画像のように、Disk Utilityでディスクを選択してEraseを実行してください：

   ![Disk Utility画面 - ディスクのErase操作](/images/komenikki-2025-09-01/disk_utility.png)

   完了後左上のリンゴマークからRestartをクリックし再起動します。

   今度はReInstallMacを選択します。

   以下の画像のように、再インストールオプションを選択してください：

   ![macOS再インストール画面 - ReInstallMacを選択](/images/komenikki-2025-09-01/reinstall_reinstall.png)

   その後は画面の指示に従いインストールを進めます。

   これにて完了です。お疲れさまでした。
---

## トラブルシューティング
- **白背景に灰色のリンゴが表示されて進まない**: CPUコア数を変えてみてください。筆者の場合、6コアから2コアや8コアに変更したら起動しました。
---

## 性能と制限

### 制限
- **ライセンス問題**: macOSはAppleのライセンスで仮想化が制限されているため、法的グレーゾーンです。実用的な使用は避け、学習目的のみに留めてください。
- **パフォーマンス**: 物理Macに比べ、仮想化によるオーバーヘッドがあるため、動作が重くなる可能性があります。
- **互換性**: 一部のハードウェアやソフトウェアで動作しない場合があります。特に、グラフィック関連の機能が制限されることが多いです。
- **サポート**: Apple公式サポート外のため、自己責任で使用してください。問題が発生してもAppleに問い合わせできません。

### 実用性について
この方法は技術的な挑戦としては魅力的ですが、実用性は非常に限定的です。日常的なmacOS使用には物理Macをおすすめします。この記事は仮想化技術の学習や実験を目的としています。

## 参考リンク

- [OSX-PROXMOX GitHub](https://github.com/luchina-gabriel/OSX-PROXMOX)

---

## まとめ

Proxmox上でmacOSを動かすのは手間もありますが、動いたときの感動は大きいです。  
実用性は限定的ですが、仮想化の仕組みを理解する良い題材になると思います。  
この記事を通じて、Proxmoxの高度な使い方やmacOSのブートプロセスについて学ぶことができます。  

興味のある方はぜひ試してみてください。ただし、上記の制限を十分に理解した上で自己責任でお試しください！

それでは、良い仮想化ライフを！